

  Coordinación de agentes

  
# Sub-Agentes

Los sub-agentes son ejecuciones de agentes en segundo plano generadas desde una ejecución de agente existente. Se ejecutan en su propia sesión (`agent::subagent:`) y, al finalizar, **anuncian** su resultado de vuelta al canal de chat del solicitante.

## Comando de barra

Usa `/subagents` para inspeccionar o controlar las ejecuciones de sub-agentes para la **sesión actual**:

-   `/subagents list`
-   `/subagents kill <id|#|all>`
-   `/subagents log <id|#> [limit] [tools]`
-   `/subagents info <id|#>`
-   `/subagents send <id|#> `
-   `/subagents steer <id|#> `
-   `/subagents spawn   [--model ] [--thinking ]`

Controles de vinculación de hilos: Estos comandos funcionan en canales que admiten vinculaciones persistentes de hilos. Consulta **Canales compatibles con hilos** más abajo.

-   `/focus <subagent-label|session-key|session-id|session-label>`
-   `/unfocus`
-   `/agents`
-   `/session idle <duration|off>`
-   `/session max-age <duration|off>`

`/subagents info` muestra metadatos de la ejecución (estado, marcas de tiempo, id de sesión, ruta de la transcripción, limpieza).

### Comportamiento de generación

`/subagents spawn` inicia un sub-agente en segundo plano como un comando de usuario, no como un retransmisor interno, y envía una única actualización de finalización al chat del solicitante cuando la ejecución termina.

-   El comando de generación es no bloqueante; devuelve un id de ejecución inmediatamente.
-   Al completarse, el sub-agente anuncia un mensaje de resumen/resultado de vuelta al canal de chat del solicitante.
-   Para generaciones manuales, la entrega es resistente:
    -   OpenClaw intenta primero la entrega directa `agent` con una clave de idempotencia estable.
    -   Si la entrega directa falla, recurre al enrutamiento por cola.
    -   Si el enrutamiento por cola tampoco está disponible, el anuncio se reintenta con un retroceso exponencial corto antes de la renuncia final.
-   La transferencia de finalización a la sesión del solicitante es un contexto interno generado en tiempo de ejecución (no texto escrito por el usuario) e incluye:
    -   `Result` (texto de respuesta del `assistant`, o el último `toolResult` si la respuesta del asistente está vacía)
    -   `Status` (`completed successfully` / `failed` / `timed out` / `unknown`)
    -   estadísticas compactas de tiempo de ejecución/tokens
    -   una instrucción de entrega que le dice al agente solicitante que reescriba con la voz normal del asistente (no reenviar metadatos internos en bruto)
-   `--model` y `--thinking` anulan los valores predeterminados para esa ejecución específica.
-   Usa `info`/`log` para inspeccionar detalles y salida después de la finalización.
-   `/subagents spawn` es el modo de un solo uso (`mode: "run"`). Para sesiones persistentes vinculadas a hilos, usa `sessions_spawn` con `thread: true` y `mode: "session"`.
-   Para sesiones del arnés ACP (Codex, Claude Code, Gemini CLI), usa `sessions_spawn` con `runtime: "acp"` y consulta [Agentes ACP](./acp-agents.md).

Objetivos principales:

-   Paralelizar trabajos de "investigación / tarea larga / herramienta lenta" sin bloquear la ejecución principal.
-   Mantener los sub-agentes aislados por defecto (separación de sesión + aislamiento opcional).
-   Mantener la superficie de herramientas difícil de usar mal: los sub-agentes **no** obtienen herramientas de sesión por defecto.
-   Soportar profundidad de anidación configurable para patrones de orquestador.

Nota de coste: cada sub-agente tiene su **propio** contexto y uso de tokens. Para tareas pesadas o repetitivas, configura un modelo más barato para los sub-agentes y mantén tu agente principal en un modelo de mayor calidad. Puedes configurar esto mediante `agents.defaults.subagents.model` o anulaciones por agente.

## Herramienta

Usa `sessions_spawn`:

-   Inicia una ejecución de sub-agente (`deliver: false`, carril global: `subagent`)
-   Luego ejecuta un paso de anuncio y publica la respuesta del anuncio en el canal de chat del solicitante
-   Modelo predeterminado: hereda del llamador a menos que configures `agents.defaults.subagents.model` (o por agente `agents.list[].subagents.model`); un `sessions_spawn.model` explícito aún prevalece.
-   Nivel de pensamiento predeterminado: hereda del llamador a menos que configures `agents.defaults.subagents.thinking` (o por agente `agents.list[].subagents.thinking`); un `sessions_spawn.thinking` explícito aún prevalece.
-   Tiempo de espera de ejecución predeterminado: si se omite `sessions_spawn.runTimeoutSeconds`, OpenClaw usa `agents.defaults.subagents.runTimeoutSeconds` cuando está configurado; de lo contrario, recurre a `0` (sin tiempo de espera).

Parámetros de la herramienta:

-   `task` (requerido)
-   `label?` (opcional)
-   `agentId?` (opcional; generar bajo otro id de agente si está permitido)
-   `model?` (opcional; anula el modelo del sub-agente; los valores no válidos se omiten y el sub-agente se ejecuta en el modelo predeterminado con una advertencia en el resultado de la herramienta)
-   `thinking?` (opcional; anula el nivel de pensamiento para la ejecución del sub-agente)
-   `runTimeoutSeconds?` (por defecto `agents.defaults.subagents.runTimeoutSeconds` cuando está configurado, de lo contrario `0`; cuando se establece, la ejecución del sub-agente se aborta después de N segundos)
-   `thread?` (por defecto `false`; cuando es `true`, solicita la vinculación de hilo del canal para esta sesión de sub-agente)
-   `mode?` (`run|session`)
    -   el valor por defecto es `run`
    -   si `thread: true` y se omite `mode`, el valor por defecto se convierte en `session`
    -   `mode: "session"` requiere `thread: true`
-   `cleanup?` (`delete|keep`, por defecto `keep`)
-   `sandbox?` (`inherit|require`, por defecto `inherit`; `require` rechaza la generación a menos que el tiempo de ejecución del hijo objetivo esté aislado)
-   `sessions_spawn` **no** acepta parámetros de entrega por canal (`target`, `channel`, `to`, `threadId`, `replyTo`, `transport`). Para la entrega, usa `message`/`sessions_send` desde la ejecución generada.

## Sesiones vinculadas a hilos

Cuando las vinculaciones de hilos están habilitadas para un canal, un sub-agente puede permanecer vinculado a un hilo para que los mensajes de seguimiento del usuario en ese hilo sigan enrutándose a la misma sesión de sub-agente.

### Canales compatibles con hilos

-   Discord (actualmente el único canal compatible): admite sesiones de subagentes vinculadas a hilos persistentes (`sessions_spawn` con `thread: true`), controles manuales de hilos (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`), y claves del adaptador `channels.discord.threadBindings.enabled`, `channels.discord.threadBindings.idleHours`, `channels.discord.threadBindings.maxAgeHours`, y `channels.discord.threadBindings.spawnSubagentSessions`.

Flujo rápido:

1.  Genera con `sessions_spawn` usando `thread: true` (y opcionalmente `mode: "session"`).
2.  OpenClaw crea o vincula un hilo a ese objetivo de sesión en el canal activo.
3.  Las respuestas y mensajes de seguimiento en ese hilo se enrutan a la sesión vinculada.
4.  Usa `/session idle` para inspeccionar/actualizar la auto-desvinculación por inactividad y `/session max-age` para controlar el límite máximo.
5.  Usa `/unfocus` para desvincular manualmente.

Controles manuales:

-   `/focus ` vincula el hilo actual (o crea uno) a un objetivo de sub-agente/sesión.
-   `/unfocus` elimina la vinculación para el hilo vinculado actual.
-   `/agents` lista las ejecuciones activas y el estado de vinculación (`thread:` o `unbound`).
-   `/session idle` y `/session max-age` solo funcionan para hilos vinculados enfocados.

Interruptores de configuración:

-   Valor predeterminado global: `session.threadBindings.enabled`, `session.threadBindings.idleHours`, `session.threadBindings.maxAgeHours`
-   Las claves de anulación por canal y de auto-vinculación de generación son específicas del adaptador. Consulta **Canales compatibles con hilos** más arriba.

Consulta [Referencia de configuración](../gateway/configuration-reference.md) y [Comandos de barra](./slash-commands.md) para detalles actuales del adaptador. Lista de permitidos:

-   `agents.list[].subagents.allowAgents`: lista de ids de agentes que pueden ser objetivo mediante `agentId` (`["*"]` para permitir cualquiera). Por defecto: solo el agente solicitante.
-   Guardia de herencia de aislamiento: si la sesión del solicitante está aislada, `sessions_spawn` rechaza objetivos que se ejecutarían sin aislamiento.

Descubrimiento:

-   Usa `agents_list` para ver qué ids de agente están actualmente permitidos para `sessions_spawn`.

Auto-archivado:

-   Las sesiones de sub-agentes se archivan automáticamente después de `agents.defaults.subagents.archiveAfterMinutes` (por defecto: 60).
-   El archivo usa `sessions.delete` y renombra la transcripción a `*.deleted.` (misma carpeta).
-   `cleanup: "delete"` archiva inmediatamente después del anuncio (aún mantiene la transcripción mediante renombrado).
-   El auto-archivado es de mejor esfuerzo; los temporizadores pendientes se pierden si el gateway se reinicia.
-   `runTimeoutSeconds` **no** auto-archiva; solo detiene la ejecución. La sesión permanece hasta el auto-archivado.
-   El auto-archivado se aplica igualmente a sesiones de profundidad 1 y profundidad 2.

## Sub-Agentes Anidados

Por defecto, los sub-agentes no pueden generar sus propios sub-agentes (`maxSpawnDepth: 1`). Puedes habilitar un nivel de anidación configurando `maxSpawnDepth: 2`, lo que permite el **patrón de orquestador**: principal → sub-agente orquestador → sub-sub-agentes trabajadores.

### Cómo habilitar

```json
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // permitir que los sub-agentes generen hijos (por defecto: 1)
        maxChildrenPerAgent: 5, // máximo de hijos activos por sesión de agente (por defecto: 5)
        maxConcurrent: 8, // límite global de concurrencia en el carril (por defecto: 8)
        runTimeoutSeconds: 900, // tiempo de espera predeterminado para sessions_spawn cuando se omite (0 = sin tiempo de espera)
      },
    },
  },
}
```

### Niveles de profundidad

| Profundidad | Forma de la clave de sesión | Rol | ¿Puede generar? |
| --- | --- | --- | --- |
| 0 | `agent::main` | Agente principal | Siempre |
| 1 | `agent::subagent:` | Sub-agente (orquestador cuando se permite profundidad 2) | Solo si `maxSpawnDepth >= 2` |
| 2 | `agent::subagent::subagent:` | Sub-sub-agente (trabajador hoja) | Nunca |

### Cadena de anuncio

Los resultados fluyen de vuelta hacia arriba en la cadena:

1.  El trabajador de profundidad 2 termina → anuncia a su padre (orquestador de profundidad 1)
2.  El orquestador de profundidad 1 recibe el anuncio, sintetiza resultados, termina → anuncia al principal
3.  El agente principal recibe el anuncio y lo entrega al usuario

Cada nivel solo ve anuncios de sus hijos directos.

### Política de herramientas por profundidad

-   **Profundidad 1 (orquestador, cuando `maxSpawnDepth >= 2`)**: Obtiene `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` para que pueda gestionar a sus hijos. Otras herramientas de sesión/sistema permanecen denegadas.
-   **Profundidad 1 (hoja, cuando `maxSpawnDepth == 1`)**: Sin herramientas de sesión (comportamiento predeterminado actual).
-   **Profundidad 2 (trabajador hoja)**: Sin herramientas de sesión — `sessions_spawn` siempre está denegada en profundidad 2. No puede generar más hijos.

### Límite de generación por agente

Cada sesión de agente (en cualquier profundidad) puede tener como máximo `maxChildrenPerAgent` (por defecto: 5) hijos activos a la vez. Esto evita una proliferación descontrolada desde un solo orquestador.

### Parada en cascada

Detener un orquestador de profundidad 1 detiene automáticamente a todos sus hijos de profundidad 2:

-   `/stop` en el chat principal detiene todos los agentes de profundidad 1 y se propaga en cascada a sus hijos de profundidad 2.
-   `/subagents kill ` detiene un sub-agente específico y se propaga en cascada a sus hijos.
-   `/subagents kill all` detiene todos los sub-agentes del solicitante y se propaga en cascada.

## Autenticación

La autenticación de sub-agentes se resuelve por **id de agente**, no por tipo de sesión:

-   La clave de sesión del sub-agente es `agent::subagent:`.
-   El almacén de autenticación se carga desde el `agentDir` de ese agente.
-   Los perfiles de autenticación del agente principal se fusionan como **respaldo**; los perfiles del agente anulan los perfiles principales en conflictos.

Nota: la fusión es aditiva, por lo que los perfiles principales siempre están disponibles como respaldo. La autenticación completamente aislada por agente aún no es compatible.

## Anuncio

Los sub-agentes informan de vuelta mediante un paso de anuncio:

-   El paso de anuncio se ejecuta dentro de la sesión del sub-agente (no en la sesión del solicitante).
-   Si el sub-agente responde exactamente `ANNOUNCE_SKIP`, no se publica nada.
-   De lo contrario, la entrega depende de la profundidad del solicitante:
    -   las sesiones solicitantes de nivel superior usan una llamada `agent` de seguimiento con entrega externa (`deliver=true`)
    -   las sesiones solicitantes de subagentes anidadas reciben una inyección interna de seguimiento (`deliver=false`) para que el orquestador pueda sintetizar los resultados de los hijos en sesión
    -   si una sesión solicitante de subagente anidada ha desaparecido, OpenClaw recurre al solicitante de esa sesión cuando está disponible
-   La agregación de finalización de hijos está limitada a la ejecución del solicitante actual al construir hallazgos de finalización anidados, evitando que salidas de hijos de ejecuciones anteriores obsoletas se filtren en el anuncio actual.
-   Las respuestas de anuncio preservan el enrutamiento de hilo/tema cuando está disponible en los adaptadores de canal.
-   El contexto del anuncio se normaliza a un bloque de evento interno estable:
    -   fuente (`subagent` o `cron`)
    -   clave/id de sesión del hijo
    -   tipo de anuncio + etiqueta de tarea
    -   línea de estado derivada del resultado del tiempo de ejecución (`success`, `error`, `timeout`, o `unknown`)
    -   contenido del resultado del paso de anuncio (o `(no output)` si falta)
    -   una instrucción de seguimiento que describe cuándo responder vs. permanecer en silencio
-   `Status` no se infiere de la salida del modelo; proviene de señales de resultado del tiempo de ejecución.

Las cargas útiles de anuncio incluyen una línea de estadísticas al final (incluso cuando están envueltas):

-   Tiempo de ejecución (ej., `runtime 5m12s`)
-   Uso de tokens (entrada/salida/total)
-   Coste estimado cuando la fijación de precios del modelo está configurada (`models.providers.*.models[].cost`)
-   `sessionKey`, `sessionId`, y ruta de la transcripción (para que el agente principal pueda obtener el historial mediante `sessions_history` o inspeccionar el archivo en disco)
-   Los metadatos internos están destinados solo para orquestación; las respuestas orientadas al usuario deben reescribirse con la voz normal del asistente.

## Política de Herramientas (herramientas de sub-agentes)

Por defecto, los sub-agentes obtienen **todas las herramientas excepto las herramientas de sesión** y las herramientas del sistema:

-   `sessions_list`
-   `sessions_history`
-   `sessions_send`
-   `sessions_spawn`

Cuando `maxSpawnDepth >= 2`, los sub-agentes orquestadores de profundidad 1 reciben adicionalmente `sessions_spawn`, `subagents`, `sessions_list`, y `sessions_history` para que puedan gestionar a sus hijos. Anula mediante configuración:

```json
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny gana
        deny: ["gateway", "cron"],
        // si allow está configurado, se convierte en solo permitir (deny aún gana)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrencia

Los sub-agentes usan un carril de cola en proceso dedicado:

-   Nombre del carril: `subagent`
-   Concurrencia: `agents.defaults.subagents.maxConcurrent` (por defecto `8`)

## Detención

-   Enviar `/stop` en el chat del solicitante aborta la sesión del solicitante y detiene cualquier ejecución de sub-agente activa generada desde ella, propagándose en cascada a los hijos anidados.
-   `/subagents kill ` detiene un sub-agente específico y se propaga en cascada a sus hijos.

## Limitaciones

-   El anuncio de sub-agente es de **mejor esfuerzo**. Si el gateway se reinicia, el trabajo pendiente de "anunciar de vuelta" se pierde.
-   Los sub-agentes aún comparten los mismos recursos del proceso del gateway; trata `maxConcurrent` como una válvula de seguridad.
-   `sessions_spawn` siempre es no bloqueante: devuelve `{ status: "accepted", runId, childSessionKey }` inmediatamente.
-   El contexto del sub-agente solo inyecta `AGENTS.md` + `TOOLS.md` (sin `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, o `BOOTSTRAP.md`).
-   La profundidad máxima de anidación es 5 (rango de `maxSpawnDepth`: 1–5). Se recomienda la profundidad 2 para la mayoría de los casos de uso.
-   `maxChildrenPerAgent` limita los hijos activos por sesión (por defecto: 5, rango: 1–20).

[Envío de Agente](./agent-send.md)[Agentes ACP](./acp-agents.md)