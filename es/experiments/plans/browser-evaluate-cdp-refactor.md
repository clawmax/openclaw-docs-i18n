

  Experimentos

  
# Refactorización de CDP para Browser Evaluate

## Contexto

`act:evaluate` ejecuta el JavaScript proporcionado por el usuario en la página. Actualmente se ejecuta a través de Playwright (`page.evaluate` o `locator.evaluate`). Playwright serializa los comandos de CDP por página, por lo que una evaluación atascada o de larga duración puede bloquear la cola de comandos de la página y hacer que cada acción posterior en esa pestaña parezca "atascada". El PR #13498 añade una red de seguridad pragmática (evaluación acotada, propagación de aborto y recuperación de mejor esfuerzo). Este documento describe una refactorización más amplia que hace que `act:evaluate` esté inherentemente aislado de Playwright, de modo que una evaluación atascada no pueda bloquear las operaciones normales de Playwright.

## Objetivos

-   `act:evaluate` no puede bloquear permanentemente acciones posteriores del navegador en la misma pestaña.
-   Los tiempos de espera son la única fuente de verdad de extremo a extremo, de modo que quien llama puede confiar en un presupuesto.
-   El aborto y el tiempo de espera se tratan de la misma manera en el despacho HTTP y en proceso.
-   Se admite el direccionamiento de elementos para evaluate sin cambiar todo fuera de Playwright.
-   Mantener la compatibilidad con versiones anteriores para los llamadores y cargas útiles existentes.

## No objetivos

-   Reemplazar todas las acciones del navegador (clic, escribir, esperar, etc.) con implementaciones de CDP.
-   Eliminar la red de seguridad existente introducida en el PR #13498 (sigue siendo un respaldo útil).
-   Introducir nuevas capacidades inseguras más allá de la puerta existente `browser.evaluateEnabled`.
-   Añadir aislamiento de procesos (proceso/hilo de trabajo) para evaluate. Si aún vemos estados de bloqueo difíciles de recuperar después de esta refactorización, esa es una idea para seguir.

## Arquitectura Actual (Por Qué Se Atasca)

A alto nivel:

-   Los llamadores envían `act:evaluate` al servicio de control del navegador.
-   El manejador de ruta llama a Playwright para ejecutar el JavaScript.
-   Playwright serializa los comandos de la página, por lo que una evaluación que nunca termina bloquea la cola.
-   Una cola atascada significa que las operaciones posteriores de clic/escritura/espera en la pestaña pueden parecer colgadas.

## Arquitectura Propuesta

### 1. Propagación del Plazo Límite

Introducir un único concepto de presupuesto y derivar todo de él:

-   El llamador establece `timeoutMs` (o un plazo límite en el futuro).
-   El tiempo de espera de la solicitud externa, la lógica del manejador de ruta y el presupuesto de ejecución dentro de la página utilizan el mismo presupuesto, con un pequeño margen donde sea necesario para la sobrecarga de serialización.
-   El aborto se propaga como una `AbortSignal` en todas partes para que la cancelación sea consistente.

Dirección de implementación:

-   Añadir un pequeño ayudante (por ejemplo `createBudget({ timeoutMs, signal })`) que devuelva:
    -   `signal`: la AbortSignal vinculada
    -   `deadlineAtMs`: plazo límite absoluto
    -   `remainingMs()`: presupuesto restante para operaciones secundarias
-   Usar este ayudante en:
    -   `src/browser/client-fetch.ts` (despacho HTTP y en proceso)
    -   `src/node-host/runner.ts` (ruta de proxy)
    -   implementaciones de acciones del navegador (Playwright y CDP)

### 2. Motor de Evaluación Separado (Ruta CDP)

Añadir una implementación de evaluación basada en CDP que no comparta la cola de comandos por página de Playwright. La propiedad clave es que el transporte de evaluación es una conexión WebSocket separada y una sesión de CDP separada adjunta al objetivo. Dirección de implementación:

-   Nuevo módulo, por ejemplo `src/browser/cdp-evaluate.ts`, que:
    -   Se conecta al endpoint de CDP configurado (socket a nivel de navegador).
    -   Usa `Target.attachToTarget({ targetId, flatten: true })` para obtener un `sessionId`.
    -   Ejecuta:
        -   `Runtime.evaluate` para evaluación a nivel de página, o
        -   `DOM.resolveNode` más `Runtime.callFunctionOn` para evaluación de elemento.
    -   En tiempo de espera o aborto:
        -   Envía `Runtime.terminateExecution` de mejor esfuerzo para la sesión.
        -   Cierra el WebSocket y devuelve un error claro.

Notas:

-   Esto aún ejecuta JavaScript en la página, por lo que la terminación puede tener efectos secundarios. La ventaja es que no bloquea la cola de Playwright y es cancelable en la capa de transporte matando la sesión de CDP.

### 3. Historia de Ref (Direccionamiento de Elementos Sin Una Reescribir Completa)

La parte difícil es el direccionamiento de elementos. CDP necesita un manipulador del DOM o un `backendDOMNodeId`, mientras que hoy la mayoría de las acciones del navegador usan localizadores de Playwright basados en refs de instantáneas. Enfoque recomendado: mantener los refs existentes, pero adjuntar un id opcional que CDP pueda resolver.

#### 3.1 Extender la Información de Ref Almacenada

Extender los metadatos de ref de rol almacenados para incluir opcionalmente un id de CDP:

-   Hoy: `{ role, name, nth }`
-   Propuesto: `{ role, name, nth, backendDOMNodeId?: number }`

Esto mantiene todas las acciones basadas en Playwright existentes funcionando y permite que CDP evaluate acepte el mismo valor `ref` cuando `backendDOMNodeId` esté disponible.

#### 3.2 Poblar backendDOMNodeId en el Momento de la Instantánea

Al producir una instantánea de rol:

1.  Generar el mapa de refs de rol existente como hoy (rol, nombre, nth).
2.  Obtener el árbol AX a través de CDP (`Accessibility.getFullAXTree`) y calcular un mapa paralelo de `(rol, nombre, nth) -> backendDOMNodeId` usando las mismas reglas de manejo de duplicados.
3.  Fusionar el id de vuelta en la información de ref almacenada para la pestaña actual.

Si el mapeo falla para un ref, dejar `backendDOMNodeId` indefinido. Esto hace que la característica sea de mejor esfuerzo y segura para implementar.

#### 3.3 Comportamiento de Evaluate Con Ref

En `act:evaluate`:

-   Si `ref` está presente y tiene `backendDOMNodeId`, ejecutar evaluación de elemento a través de CDP.
-   Si `ref` está presente pero no tiene `backendDOMNodeId`, recurrir a la ruta de Playwright (con la red de seguridad).

Escotilla de escape opcional:

-   Extender la forma de la solicitud para aceptar `backendDOMNodeId` directamente para llamadores avanzados (y para depuración), manteniendo `ref` como la interfaz principal.

### 4. Mantener Una Ruta de Recuperación de Último Recurso

Incluso con CDP evaluate, hay otras formas de bloquear una pestaña o una conexión. Mantener los mecanismos de recuperación existentes (terminar ejecución + desconectar Playwright) como último recurso para:

-   llamadores heredados
-   entornos donde el adjunto de CDP está bloqueado
-   casos extremos inesperados de Playwright

## Plan de Implementación (Iteración Única)

### Entregables

-   Un motor de evaluación basado en CDP que se ejecuta fuera de la cola de comandos por página de Playwright.
-   Un único presupuesto de tiempo de espera/aborto de extremo a extremo usado consistentemente por llamadores y manejadores.
-   Metadatos de ref que pueden llevar opcionalmente `backendDOMNodeId` para evaluación de elemento.
-   `act:evaluate` prefiere el motor CDP cuando es posible y recurre a Playwright cuando no.
-   Pruebas que demuestren que una evaluación atascada no bloquea acciones posteriores.
-   Registros/métricas que hagan visibles los fallos y las rutas de respaldo.

### Lista de Verificación de Implementación

1.  Añadir un ayudante "presupuesto" compartido para vincular `timeoutMs` + `AbortSignal` ascendente en:
    -   una única `AbortSignal`
    -   un plazo límite absoluto
    -   un ayudante `remainingMs()` para operaciones posteriores
2.  Actualizar todas las rutas de llamada para usar ese ayudante, de modo que `timeoutMs` signifique lo mismo en todas partes:
    -   `src/browser/client-fetch.ts` (despacho HTTP y en proceso)
    -   `src/node-host/runner.ts` (ruta de proxy de nodo)
    -   envoltorios CLI que llaman a `/act` (añadir `--timeout-ms` a `browser evaluate`)
3.  Implementar `src/browser/cdp-evaluate.ts`:
    -   conectar al socket de CDP a nivel de navegador
    -   `Target.attachToTarget` para obtener un `sessionId`
    -   ejecutar `Runtime.evaluate` para evaluación de página
    -   ejecutar `DOM.resolveNode` + `Runtime.callFunctionOn` para evaluación de elemento
    -   en tiempo de espera/aborto: `Runtime.terminateExecution` de mejor esfuerzo y luego cerrar el socket
4.  Extender los metadatos de ref de rol almacenados para incluir opcionalmente `backendDOMNodeId`:
    -   mantener el comportamiento existente `{ role, name, nth }` para acciones de Playwright
    -   añadir `backendDOMNodeId?: number` para direccionamiento de elementos en CDP
5.  Poblar `backendDOMNodeId` durante la creación de la instantánea (mejor esfuerzo):
    -   obtener el árbol AX a través de CDP (`Accessibility.getFullAXTree`)
    -   calcular `(rol, nombre, nth) -> backendDOMNodeId` y fusionar en el mapa de ref almacenado
    -   si el mapeo es ambiguo o falta, dejar el id indefinido
6.  Actualizar el enrutamiento de `act:evaluate`:
    -   si no hay `ref`: siempre usar CDP evaluate
    -   si `ref` se resuelve a un `backendDOMNodeId`: usar evaluación de elemento en CDP
    -   de lo contrario: recurrir a Playwright evaluate (todavía acotado y abortable)
7.  Mantener la ruta de recuperación de "último recurso" existente como respaldo, no como ruta predeterminada.
8.  Añadir pruebas:
    -   una evaluación atascada agota el tiempo de espera dentro del presupuesto y el siguiente clic/escritura tiene éxito
    -   el aborto cancela la evaluación (desconexión del cliente o tiempo de espera) y desbloquea acciones posteriores
    -   los fallos de mapeo recurren limpiamente a Playwright
9.  Añadir observabilidad:
    -   contadores de duración y tiempo de espera de la evaluación
    -   uso de terminateExecution
    -   tasa de respaldo (CDP -> Playwright) y razones

### Criterios de Aceptación

-   Un `act:evaluate` deliberadamente colgado regresa dentro del presupuesto del llamador y no bloquea la pestaña para acciones posteriores.
-   `timeoutMs` se comporta consistentemente en CLI, herramienta de agente, proxy de nodo y llamadas en proceso.
-   Si `ref` puede mapearse a `backendDOMNodeId`, la evaluación de elemento usa CDP; de lo contrario, la ruta de respaldo sigue estando acotada y es recuperable.

## Plan de Pruebas

-   Pruebas unitarias:
    -   lógica de coincidencia `(rol, nombre, nth)` entre refs de rol y nodos del árbol AX.
    -   comportamiento del ayudante de presupuesto (margen, matemática del tiempo restante).
-   Pruebas de integración:
    -   el tiempo de espera de CDP evaluate regresa dentro del presupuesto y no bloquea la siguiente acción.
    -   el aborto cancela la evaluación y desencadena la terminación de mejor esfuerzo.
-   Pruebas de contrato:
    -   Asegurar que `BrowserActRequest` y `BrowserActResponse` sigan siendo compatibles.

## Riesgos y Mitigaciones

-   El mapeo es imperfecto:
    -   Mitigación: mapeo de mejor esfuerzo, respaldo a Playwright evaluate y añadir herramientas de depuración.
-   `Runtime.terminateExecution` tiene efectos secundarios:
    -   Mitigación: usar solo en tiempo de espera/aborto y documentar el comportamiento en los errores.
-   Sobrecarga adicional:
    -   Mitigación: obtener el árbol AX solo cuando se solicitan instantáneas, almacenar en caché por objetivo y mantener la sesión de CDP de corta duración.
-   Limitaciones del relé de extensión:
    -   Mitigación: usar APIs de adjunto a nivel de navegador cuando los sockets por página no estén disponibles, y mantener la ruta actual de Playwright como respaldo.

## Preguntas Abiertas

-   ¿Debería el nuevo motor ser configurable como `playwright`, `cdp` o `auto`?
-   ¿Queremos exponer un nuevo formato "nodeRef" para usuarios avanzados, o mantener solo `ref`?
-   ¿Cómo deberían participar las instantáneas de marco y las instantáneas con ámbito de selector en el mapeo AX?

[Plan de Refactorización de Streaming Unificado de Runtime](./acp-unified-streaming-refactor.md)[Plan de Gateway de OpenResponses](./openresponses-gateway.md)