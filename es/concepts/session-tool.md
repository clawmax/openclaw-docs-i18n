

  Sesiones y memoria

  
# Herramientas de Sesión

Objetivo: conjunto de herramientas pequeño y difícil de usar mal para que los agentes puedan listar sesiones, obtener el historial y enviar a otra sesión.

## Nombres de las Herramientas

-   `sessions_list`
-   `sessions_history`
-   `sessions_send`
-   `sessions_spawn`

## Modelo de Clave

-   El bucket de chat directo principal es siempre la clave literal `"main"` (resuelta a la clave principal del agente actual).
-   Los chats grupales usan `agent:::group:` o `agent:::channel:` (pasa la clave completa).
-   Los trabajos cron usan `cron:<job.id>`.
-   Los hooks usan `hook:` a menos que se establezca explícitamente.
-   Las sesiones de nodo usan `node-` a menos que se establezca explícitamente.

`global` y `unknown` son valores reservados y nunca se listan. Si `session.scope = "global"`, lo aliasamos a `main` para todas las herramientas, de modo que los llamadores nunca vean `global`.

## sessions\_list

Lista sesiones como un array de filas. Parámetros:

-   `kinds?: string[]` filtro: cualquiera de `"main" | "group" | "cron" | "hook" | "node" | "other"`
-   `limit?: number` filas máximas (por defecto: valor por defecto del servidor, límite ej. 200)
-   `activeMinutes?: number` solo sesiones actualizadas dentro de N minutos
-   `messageLimit?: number` 0 = sin mensajes (por defecto 0); >0 = incluye los últimos N mensajes

Comportamiento:

-   `messageLimit > 0` obtiene `chat.history` por sesión e incluye los últimos N mensajes.
-   Los resultados de herramientas se filtran en la salida de la lista; usa `sessions_history` para mensajes de herramientas.
-   Cuando se ejecuta en una sesión de agente **en sandbox**, las herramientas de sesión por defecto tienen **visibilidad solo de sesiones generadas** (ver abajo).

Forma de la fila (JSON):

-   `key`: clave de sesión (string)
-   `kind`: `main | group | cron | hook | node | other`
-   `channel`: `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
-   `displayName` (etiqueta de visualización del grupo si está disponible)
-   `updatedAt` (ms)
-   `sessionId`
-   `model`, `contextTokens`, `totalTokens`
-   `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
-   `sendPolicy` (anulación de sesión si está establecida)
-   `lastChannel`, `lastTo`
-   `deliveryContext` (normalizado `{ channel, to, accountId }` cuando está disponible)
-   `transcriptPath` (ruta de mejor esfuerzo derivada del directorio de almacenamiento + sessionId)
-   `messages?` (solo cuando `messageLimit > 0`)

## sessions\_history

Obtiene la transcripción de una sesión. Parámetros:

-   `sessionKey` (requerido; acepta clave de sesión o `sessionId` de `sessions_list`)
-   `limit?: number` mensajes máximos (el servidor limita)
-   `includeTools?: boolean` (por defecto false)

Comportamiento:

-   `includeTools=false` filtra mensajes con `role: "toolResult"`.
-   Devuelve un array de mensajes en el formato de transcripción crudo.
-   Cuando se le da un `sessionId`, OpenClaw lo resuelve a la clave de sesión correspondiente (los ids faltantes dan error).

## sessions\_send

Envía un mensaje a otra sesión. Parámetros:

-   `sessionKey` (requerido; acepta clave de sesión o `sessionId` de `sessions_list`)
-   `message` (requerido)
-   `timeoutSeconds?: number` (por defecto >0; 0 = enviar y olvidar)

Comportamiento:

-   `timeoutSeconds = 0`: encola y devuelve `{ runId, status: "accepted" }`.
-   `timeoutSeconds > 0`: espera hasta N segundos para completarse, luego devuelve `{ runId, status: "ok", reply }`.
-   Si la espera se agota: `{ runId, status: "timeout", error }`. La ejecución continúa; llama a `sessions_history` más tarde.
-   Si la ejecución falla: `{ runId, status: "error", error }`.
-   La ejecución de anuncio de entrega se ejecuta después de que la ejecución primaria se completa y es de mejor esfuerzo; `status: "ok"` no garantiza que el anuncio se haya entregado.
-   Espera a través del gateway `agent.wait` (en el lado del servidor) para que las reconexiones no interrumpan la espera.
-   El contexto de mensaje de agente a agente se inyecta para la ejecución primaria.
-   Los mensajes entre sesiones se persisten con `message.provenance.kind = "inter_session"` para que los lectores de transcripción puedan distinguir las instrucciones de agentes enrutadas de la entrada externa del usuario.
-   Después de que la ejecución primaria se completa, OpenClaw ejecuta un **bucle de respuesta**:
    -   La ronda 2+ alterna entre el agente solicitante y el agente objetivo.
    -   Responde exactamente `REPLY_SKIP` para detener el ping‑pong.
    -   El máximo de turnos es `session.agentToAgent.maxPingPongTurns` (0–5, por defecto 5).
-   Una vez que termina el bucle, OpenClaw ejecuta el **paso de anuncio agente‑a‑agente** (solo el agente objetivo):
    -   Responde exactamente `ANNOUNCE_SKIP` para permanecer en silencio.
    -   Cualquier otra respuesta se envía al canal objetivo.
    -   El paso de anuncio incluye la solicitud original + la respuesta de la ronda‑1 + la última respuesta de ping‑pong.

## Campo Channel

-   Para grupos, `channel` es el canal registrado en la entrada de la sesión.
-   Para chats directos, `channel` se mapea desde `lastChannel`.
-   Para cron/hook/nodo, `channel` es `internal`.
-   Si falta, `channel` es `unknown`.

## Seguridad / Política de Envío

Bloqueo basado en políticas por canal/tipo de chat (no por id de sesión).

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

Anulación en tiempo de ejecución (por entrada de sesión):

-   `sendPolicy: "allow" | "deny"` (no establecido = heredar configuración)
-   Se puede establecer mediante `sessions.patch` o mediante `/send on|off|inherit` (solo propietario, mensaje independiente).

Puntos de aplicación:

-   `chat.send` / `agent` (gateway)
-   lógica de entrega de respuesta automática

## sessions\_spawn

Genera una ejecución de subagente en una sesión aislada y anuncia el resultado de vuelta al canal de chat del solicitante. Parámetros:

-   `task` (requerido)
-   `label?` (opcional; usado para logs/UI)
-   `agentId?` (opcional; generar bajo otro id de agente si está permitido)
-   `model?` (opcional; anula el modelo del subagente; valores inválidos dan error)
-   `thinking?` (opcional; anula el nivel de pensamiento para la ejecución del subagente)
-   `runTimeoutSeconds?` (por defecto `agents.defaults.subagents.runTimeoutSeconds` cuando está establecido, de lo contrario `0`; cuando está establecido, aborta la ejecución del subagente después de N segundos)
-   `thread?` (por defecto false; solicita enrutamiento ligado a hilo para este spawn cuando el canal/plugin lo soporta)
-   `mode?` (`run|session`; por defecto `run`, pero por defecto `session` cuando `thread=true`; `mode="session"` requiere `thread=true`)
-   `cleanup?` (`delete|keep`, por defecto `keep`)
-   `sandbox?` (`inherit|require`, por defecto `inherit`; `require` rechaza el spawn a menos que el entorno de ejecución hijo objetivo esté en sandbox)
-   `attachments?` (array opcional de archivos en línea; solo entorno de ejecución de subagente, ACP rechaza). Cada entrada: `{ name, content, encoding?: "utf8" | "base64", mimeType? }`. Los archivos se materializan en el espacio de trabajo hijo en `.openclaw/attachments//`. Devuelve un recibo con sha256 por archivo.
-   `attachAs?` (opcional; `{ mountPath? }` pista reservada para futuras implementaciones de montaje)

Lista de permitidos:

-   `agents.list[].subagents.allowAgents`: lista de ids de agentes permitidos a través de `agentId` (`["*"]` para permitir cualquiera). Por defecto: solo el agente solicitante.
-   Guardia de herencia de sandbox: si la sesión del solicitante está en sandbox, `sessions_spawn` rechaza objetivos que se ejecutarían sin sandbox.

Descubrimiento:

-   Usa `agents_list` para descubrir qué ids de agente están permitidos para `sessions_spawn`.

Comportamiento:

-   Inicia una nueva sesión `agent::subagent:` con `deliver: false`.
-   Los subagentes tienen por defecto el conjunto completo de herramientas **menos las herramientas de sesión** (configurable a través de `tools.subagents.tools`).
-   Los subagentes no pueden llamar a `sessions_spawn` (no se permite generación de subagente → subagente).
-   Siempre no bloqueante: devuelve `{ status: "accepted", runId, childSessionKey }` inmediatamente.
-   Con `thread=true`, los plugins de canal pueden vincular la entrega/enrutamiento a un objetivo de hilo (el soporte de Discord está controlado por `session.threadBindings.*` y `channels.discord.threadBindings.*`).
-   Después de la finalización, OpenClaw ejecuta un **paso de anuncio de subagente** y publica el resultado en el canal de chat del solicitante.
    -   Si la respuesta final del asistente está vacía, el último `toolResult` del historial del subagente se incluye como `Result`.
-   Responde exactamente `ANNOUNCE_SKIP` durante el paso de anuncio para permanecer en silencio.
-   Las respuestas de anuncio se normalizan a `Status`/`Result`/`Notes`; `Status` proviene del resultado del entorno de ejecución (no del texto del modelo).
-   Las sesiones de subagente se archivan automáticamente después de `agents.defaults.subagents.archiveAfterMinutes` (por defecto: 60).
-   Las respuestas de anuncio incluyen una línea de estadísticas (tiempo de ejecución, tokens, sessionKey/sessionId, ruta de transcripción y costo opcional).

## Visibilidad de Sesión en Sandbox

Las herramientas de sesión pueden tener un alcance para reducir el acceso entre sesiones. Comportamiento por defecto:

-   `tools.sessions.visibility` por defecto es `tree` (sesión actual + sesiones de subagentes generadas).
-   Para sesiones en sandbox, `agents.defaults.sandbox.sessionToolsVisibility` puede limitar forzosamente la visibilidad.

Configuración:

```json
{
  tools: {
    sessions: {
      // "self" | "tree" | "agent" | "all"
      // default: "tree"
      visibility: "tree",
    },
  },
  agents: {
    defaults: {
      sandbox: {
        // default: "spawned"
        sessionToolsVisibility: "spawned", // or "all"
      },
    },
  },
}
```

Notas:

-   `self`: solo la clave de sesión actual.
-   `tree`: sesión actual + sesiones generadas por la sesión actual.
-   `agent`: cualquier sesión perteneciente al id del agente actual.
-   `all`: cualquier sesión (el acceso entre agentes aún requiere `tools.agentToAgent`).
-   Cuando una sesión está en sandbox y `sessionToolsVisibility="spawned"`, OpenClaw limita la visibilidad a `tree` incluso si configuras `tools.sessions.visibility="all"`.

[Poda de Sesiones](./session-pruning.md)[Memoria](./memory.md)