

  Herramientas integradas

  
# Niveles de Pensamiento

## Qué hace

-   Directiva en línea en cualquier cuerpo de mensaje entrante: `/t `, `/think:`, o `/thinking `.
-   Niveles (alias): `off | minimal | low | medium | high | xhigh | adaptive`
    -   minimal → “think”
    -   low → “think hard”
    -   medium → “think harder”
    -   high → “ultrathink” (presupuesto máximo)
    -   xhigh → “ultrathink+” (solo modelos GPT-5.2 + Codex)
    -   adaptive → presupuesto de razonamiento adaptativo gestionado por el proveedor (compatible con la familia de modelos Anthropic Claude 4.6)
    -   `x-high`, `x_high`, `extra-high`, `extra high`, y `extra_high` se asignan a `xhigh`.
    -   `highest`, `max` se asignan a `high`.
-   Notas del proveedor:
    -   Los modelos Anthropic Claude 4.6 usan `adaptive` por defecto cuando no se establece un nivel de pensamiento explícito.
    -   Z.AI (`zai/*`) solo soporta pensamiento binario (`on`/`off`). Cualquier nivel que no sea `off` se trata como `on` (mapeado a `low`).
    -   Moonshot (`moonshot/*`) mapea `/think off` a `thinking: { type: "disabled" }` y cualquier nivel que no sea `off` a `thinking: { type: "enabled" }`. Cuando el pensamiento está habilitado, Moonshot solo acepta `tool_choice` `auto|none`; OpenClaw normaliza los valores incompatibles a `auto`.

## Orden de resolución

1.  Directiva en línea en el mensaje (se aplica solo a ese mensaje).
2.  Anulación de sesión (establecida enviando un mensaje que solo contiene la directiva).
3.  Valor predeterminado global (`agents.defaults.thinkingDefault` en la configuración).
4.  Respaldo: `adaptive` para modelos Anthropic Claude 4.6, `low` para otros modelos con capacidad de razonamiento, `off` en caso contrario.

## Establecer un valor predeterminado de sesión

-   Envía un mensaje que sea **solo** la directiva (se permite espacio en blanco), p. ej. `/think:medium` o `/t high`.
-   Eso se mantiene para la sesión actual (por remitente por defecto); se borra con `/think:off` o con el reinicio por inactividad de sesión.
-   Se envía una respuesta de confirmación (`Nivel de pensamiento establecido en high.` / `Pensamiento deshabilitado.`). Si el nivel no es válido (p. ej. `/thinking big`), el comando es rechazado con una pista y el estado de la sesión no se modifica.
-   Envía `/think` (o `/think:`) sin argumento para ver el nivel de pensamiento actual.

## Aplicación por agente

-   **Embedded Pi**: el nivel resuelto se pasa al entorno de ejecución del agente Pi en proceso.

## Directivas detalladas (/verbose o /v)

-   Niveles: `on` (mínimo) | `full` | `off` (predeterminado).
-   Un mensaje que solo contiene la directiva alterna el modo detallado de la sesión y responde `Registro detallado habilitado.` / `Registro detallado deshabilitado.`; los niveles no válidos devuelven una pista sin cambiar el estado.
-   `/verbose off` almacena una anulación explícita de sesión; bórrala a través de la UI de Sesiones eligiendo `inherit`.
-   La directiva en línea afecta solo a ese mensaje; de lo contrario, se aplican los valores predeterminados de sesión/globales.
-   Envía `/verbose` (o `/verbose:`) sin argumento para ver el nivel detallado actual.
-   Cuando verbose está activado, los agentes que emiten resultados estructurados de herramientas (Pi, otros agentes JSON) envían cada llamada a herramienta como su propio mensaje solo de metadatos, con el prefijo ` <nombre-herramienta>: ` cuando está disponible (ruta/comando). Estos resúmenes de herramientas se envían tan pronto como cada herramienta comienza (burbujas separadas), no como deltas de transmisión.
-   Los resúmenes de fallos de herramientas siguen siendo visibles en modo normal, pero los sufijos de detalles de error sin procesar están ocultos a menos que verbose sea `on` o `full`.
-   Cuando verbose es `full`, las salidas de las herramientas también se reenvían después de su finalización (burbuja separada, truncadas a una longitud segura). Si alternas `/verbose on|full|off` mientras una ejecución está en curso, las burbujas de herramientas posteriores respetan la nueva configuración.

## Visibilidad del razonamiento (/reasoning)

-   Niveles: `on|off|stream`.
-   Un mensaje que solo contiene la directiva alterna si se muestran los bloques de pensamiento en las respuestas.
-   Cuando está habilitado, el razonamiento se envía como un **mensaje separado** con el prefijo `Razonamiento:`.
-   `stream` (solo Telegram): transmite el razonamiento a la burbuja de borrador de Telegram mientras se genera la respuesta, luego envía la respuesta final sin el razonamiento.
-   Alias: `/reason`.
-   Envía `/reasoning` (o `/reasoning:`) sin argumento para ver el nivel de razonamiento actual.

## Relacionado

-   La documentación del modo elevado se encuentra en [Modo elevado](./elevated.md).

## Latidos

-   El cuerpo de la sonda de latido es el mensaje de latido configurado (predeterminado: `Lee HEARTBEAT.md si existe (contexto del espacio de trabajo). Síguelo estrictamente. No infieras ni repitas tareas antiguas de chats anteriores. Si no hay nada que requiera atención, responde HEARTBEAT_OK.`). Las directivas en línea en un mensaje de latido se aplican como de costumbre (pero evita cambiar los valores predeterminados de sesión desde los latidos).
-   La entrega del latido es solo la carga útil final por defecto. Para enviar también el mensaje separado `Razonamiento:` (cuando esté disponible), establece `agents.defaults.heartbeat.includeReasoning: true` o por agente `agents.list[].heartbeat.includeReasoning: true`.

## UI de chat web

-   El selector de pensamiento del chat web refleja el nivel almacenado de la sesión desde el almacén/configuración de sesión entrante cuando se carga la página.
-   Elegir otro nivel se aplica solo al siguiente mensaje (`thinkingOnce`); después de enviar, el selector vuelve al nivel de sesión almacenado.
-   Para cambiar el valor predeterminado de la sesión, envía una directiva `/think:` (como antes); el selector lo reflejará después de la siguiente recarga.

[Reacciones](./reactions.md)[Herramientas Web](./web.md)