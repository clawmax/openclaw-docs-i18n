

  Fundamentos

  
# Bucle del Agente

Un bucle agente es la ejecución completa y "real" de un agente: recepción → ensamblaje del contexto → inferencia del modelo → ejecución de herramientas → respuestas en streaming → persistencia. Es la ruta autoritativa que convierte un mensaje en acciones y una respuesta final, manteniendo el estado de la sesión consistente. En OpenClaw, un bucle es una ejecución única y serializada por sesión que emite eventos del ciclo de vida y de streaming mientras el modelo piensa, llama a herramientas y transmite la salida. Este documento explica cómo ese bucle auténtico está cableado de principio a fin.

## Puntos de entrada

-   Gateway RPC: `agent` y `agent.wait`.
-   CLI: comando `agent`.

## Cómo funciona (a alto nivel)

1.  El RPC `agent` valida los parámetros, resuelve la sesión (sessionKey/sessionId), persiste los metadatos de la sesión, devuelve `{ runId, acceptedAt }` inmediatamente.
2.  `agentCommand` ejecuta el agente:
    -   resuelve el modelo + valores predeterminados de pensamiento/verbose
    -   carga la instantánea de habilidades
    -   llama a `runEmbeddedPiAgent` (tiempo de ejecución de pi-agent-core)
    -   emite **lifecycle end/error** si el bucle embebido no emite uno
3.  `runEmbeddedPiAgent`:
    -   serializa las ejecuciones mediante colas por sesión + globales
    -   resuelve el modelo + perfil de autenticación y construye la sesión pi
    -   se suscribe a eventos pi y transmite deltas del asistente/herramientas
    -   aplica timeout -> aborta la ejecución si se excede
    -   devuelve cargas útiles + metadatos de uso
4.  `subscribeEmbeddedPiSession` conecta los eventos de pi-agent-core con el stream `agent` de OpenClaw:
    -   eventos de herramienta => `stream: "tool"`
    -   deltas del asistente => `stream: "assistant"`
    -   eventos del ciclo de vida => `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5.  `agent.wait` usa `waitForAgentJob`:
    -   espera por **lifecycle end/error** para el `runId`
    -   devuelve `{ status: ok|error|timeout, startedAt, endedAt, error? }`

## Colas + concurrencia

-   Las ejecuciones se serializan por clave de sesión (carril de sesión) y opcionalmente a través de un carril global.
-   Esto evita carreras de herramientas/sesión y mantiene el historial de sesiones consistente.
-   Los canales de mensajería pueden elegir modos de cola (collect/steer/followup) que alimentan este sistema de carriles. Ver [Cola de Comandos](./queue.md).

## Preparación de sesión + espacio de trabajo

-   El espacio de trabajo se resuelve y crea; las ejecuciones en sandbox pueden redirigirse a una raíz de espacio de trabajo sandbox.
-   Las habilidades se cargan (o reutilizan de una instantánea) y se inyectan en el entorno y el prompt.
-   Los archivos de arranque/contexto se resuelven y se inyectan en el informe del prompt del sistema.
-   Se adquiere un bloqueo de escritura de sesión; `SessionManager` se abre y prepara antes del streaming.

## Ensamblaje del prompt + prompt del sistema

-   El prompt del sistema se construye a partir del prompt base de OpenClaw, el prompt de habilidades, el contexto de arranque y las anulaciones por ejecución.
-   Se aplican límites específicos del modelo y se reservan tokens para compactación.
-   Ver [Prompt del Sistema](./system-prompt.md) para lo que ve el modelo.

## Puntos de enlace (donde puedes interceptar)

OpenClaw tiene dos sistemas de hooks:

-   **Hooks internos** (hooks de Gateway): scripts dirigidos por eventos para comandos y eventos del ciclo de vida.
-   **Hooks de plugins**: puntos de extensión dentro del ciclo de vida del agente/herramienta y la pipeline de gateway.

### Hooks internos (Hooks de Gateway)

-   **`agent:bootstrap`**: se ejecuta mientras se construyen los archivos de arranque antes de que se finalice el prompt del sistema. Úsalo para agregar/eliminar archivos de contexto de arranque.
-   **Hooks de comandos**: `/new`, `/reset`, `/stop`, y otros eventos de comandos (ver documento de Hooks).

Ver [Hooks](../automation/hooks.md) para configuración y ejemplos.

### Hooks de plugins (ciclo de vida del agente + gateway)

Estos se ejecutan dentro del bucle del agente o la pipeline de gateway:

-   **`before_model_resolve`**: se ejecuta pre-sesión (sin `messages`) para anular determinísticamente el proveedor/modelo antes de la resolución del modelo.
-   **`before_prompt_build`**: se ejecuta después de cargar la sesión (con `messages`) para inyectar `prependContext`, `systemPrompt`, `prependSystemContext`, o `appendSystemContext` antes del envío del prompt. Usa `prependContext` para texto dinámico por turno y los campos de contexto del sistema para orientación estable que debe estar en el espacio del prompt del sistema.
-   **`before_agent_start`**: hook de compatibilidad heredado que puede ejecutarse en cualquiera de las fases; prefiere los hooks explícitos anteriores.
-   **`agent_end`**: inspecciona la lista final de mensajes y los metadatos de ejecución después de la finalización.
-   **`before_compaction` / `after_compaction`**: observa o anota ciclos de compactación.
-   **`before_tool_call` / `after_tool_call`**: intercepta parámetros/resultados de herramientas.
-   **`tool_result_persist`**: transforma sincrónicamente los resultados de las herramientas antes de que se escriban en la transcripción de la sesión.
-   **`message_received` / `message_sending` / `message_sent`**: hooks de mensajes entrantes + salientes.
-   **`session_start` / `session_end`**: límites del ciclo de vida de la sesión.
-   **`gateway_start` / `gateway_stop`**: eventos del ciclo de vida del gateway.

Ver [Plugins](../tools/plugin.md#plugin-hooks) para la API de hooks y detalles de registro.

## Streaming + respuestas parciales

-   Los deltas del asistente se transmiten desde pi-agent-core y se emiten como eventos `assistant`.
-   El streaming de bloques puede emitir respuestas parciales ya sea en `text_end` o `message_end`.
-   El streaming de razonamiento puede emitirse como un stream separado o como respuestas de bloque.
-   Ver [Streaming](./streaming.md) para el comportamiento de fragmentación y respuesta de bloques.

## Ejecución de herramientas + herramientas de mensajería

-   Los eventos de inicio/actualización/fin de herramienta se emiten en el stream `tool`.
-   Los resultados de las herramientas se sanean por tamaño y cargas útiles de imagen antes de registrarse/emitirse.
-   Los envíos de herramientas de mensajería se rastrean para suprimir confirmaciones duplicadas del asistente.

## Moldeado de respuesta + supresión

-   Las cargas útiles finales se ensamblan a partir de:
    -   texto del asistente (y razonamiento opcional)
    -   resúmenes de herramientas en línea (cuando verbose + permitido)
    -   texto de error del asistente cuando el modelo falla
-   `NO_REPLY` se trata como un token silencioso y se filtra de las cargas útiles salientes.
-   Los duplicados de herramientas de mensajería se eliminan de la lista final de cargas útiles.
-   Si no quedan cargas útiles representables y una herramienta falló, se emite una respuesta de error de herramienta de respaldo (a menos que una herramienta de mensajería ya haya enviado una respuesta visible para el usuario).

## Compactación + reintentos

-   La auto-compactación emite eventos de stream `compaction` y puede desencadenar un reintento.
-   En el reintento, los búferes en memoria y los resúmenes de herramientas se reinician para evitar salida duplicada.
-   Ver [Compactación](./compaction.md) para la pipeline de compactación.

## Streams de eventos (actualmente)

-   `lifecycle`: emitido por `subscribeEmbeddedPiSession` (y como respaldo por `agentCommand`)
-   `assistant`: deltas transmitidos desde pi-agent-core
-   `tool`: eventos de herramientas transmitidos desde pi-agent-core

## Manejo de canales de chat

-   Los deltas del asistente se almacenan en búfer en mensajes `delta` del chat.
-   Se emite un `final` del chat en **lifecycle end/error**.

## Timeouts

-   `agent.wait` predeterminado: 30s (solo la espera). El parámetro `timeoutMs` lo anula.
-   Tiempo de ejecución del agente: `agents.defaults.timeoutSeconds` predeterminado 600s; aplicado en el temporizador de aborto de `runEmbeddedPiAgent`.

## Dónde las cosas pueden terminar antes de tiempo

-   Timeout del agente (abortar)
-   AbortSignal (cancelar)
-   Desconexión del Gateway o timeout del RPC
-   Timeout de `agent.wait` (solo espera, no detiene al agente)

[Runtime del Agente](./agent.md)[Prompt del Sistema](./system-prompt.md)