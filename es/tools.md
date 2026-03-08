title: "Guía y Referencia de Configuración de Herramientas del Agente OpenClaw"
description: "Aprende a configurar, permitir, denegar y gestionar las herramientas de primera clase del agente OpenClaw para navegador, canvas, nodos, cron y plugins con ejemplos detallados."
keywords: ["herramientas openclaw", "herramientas de agente", "configuración de herramientas", "perfiles de herramientas", "grupos de herramientas", "herramienta de navegador", "herramienta exec", "plugins de herramientas"]
---

  Descripción general

  
# Herramientas

OpenClaw expone **herramientas de primera clase del agente** para navegador, canvas, nodos y cron. Estas reemplazan las antiguas habilidades `openclaw-*`: las herramientas están tipadas, no hay shelling, y el agente debe confiar en ellas directamente.

## Deshabilitar herramientas

Puedes permitir/denegar herramientas globalmente mediante `tools.allow` / `tools.deny` en `openclaw.json` (denegar gana). Esto evita que las herramientas no permitidas se envíen a los proveedores de modelos.

```json
{
  tools: { deny: ["browser"] },
}
```

Notas:

-   La coincidencia no distingue entre mayúsculas y minúsculas.
-   Se admiten comodines `*` (`"*"` significa todas las herramientas).
-   Si `tools.allow` solo hace referencia a nombres de herramientas de plugins desconocidos o no cargados, OpenClaw registra una advertencia e ignora la lista de permitidos para que las herramientas principales sigan disponibles.

## Perfiles de herramientas (lista de permitidos base)

`tools.profile` establece una **lista de permitidos base de herramientas** antes de `tools.allow`/`tools.deny`. Anulación por agente: `agents.list[].tools.profile`. Perfiles:

-   `minimal`: solo `session_status`
-   `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
-   `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
-   `full`: sin restricción (igual que no configurado)

Ejemplo (solo mensajería por defecto, permitir también herramientas de Slack + Discord):

```json
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

Ejemplo (perfil de codificación, pero denegar exec/process en todas partes):

```json
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

Ejemplo (perfil de codificación global, agente de soporte solo de mensajería):

```json
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] },
      },
    ],
  },
}
```

## Política de herramientas específica del proveedor

Usa `tools.byProvider` para **restringir aún más** las herramientas para proveedores específicos (o un solo `provider/model`) sin cambiar tus configuraciones globales por defecto. Anulación por agente: `agents.list[].tools.byProvider`. Esto se aplica **después** del perfil base de herramientas y **antes** de las listas de permitidos/denegados, por lo que solo puede reducir el conjunto de herramientas. Las claves de proveedor aceptan `provider` (ej. `google-antigravity`) o `provider/model` (ej. `openai/gpt-5.2`). Ejemplo (mantener perfil de codificación global, pero herramientas mínimas para Google Antigravity):

```json
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

Ejemplo (lista de permitidos específica de proveedor/modelo para un endpoint inestable):

```json
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

Ejemplo (anulación específica de agente para un solo proveedor):

```json
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] },
          },
        },
      },
    ],
  },
}
```

## Grupos de herramientas (abreviaturas)

Las políticas de herramientas (global, agente, sandbox) admiten entradas `group:*` que se expanden a múltiples herramientas. Úsalas en `tools.allow` / `tools.deny`. Grupos disponibles:

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:web`: `web_search`, `web_fetch`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: todas las herramientas integradas de OpenClaw (excluye plugins de proveedor)

Ejemplo (permitir solo herramientas de archivo + navegador):

```json
{
  tools: {
    allow: ["group:fs", "browser"],
  },
}
```

## Plugins + herramientas

Los plugins pueden registrar **herramientas adicionales** (y comandos CLI) más allá del conjunto principal. Consulta [Plugins](./tools/plugin.md) para instalación + configuración, y [Skills](./tools/skills.md) para ver cómo se inyecta la guía de uso de herramientas en los prompts. Algunos plugins incluyen sus propias skills junto con las herramientas (por ejemplo, el plugin de llamadas de voz). Herramientas de plugin opcionales:

-   [Lobster](./tools/lobster.md): runtime de flujo de trabajo tipado con aprobaciones reanudables (requiere la CLI de Lobster en el host del gateway).
-   [LLM Task](./tools/llm-task.md): paso LLM solo JSON para salida estructurada de flujo de trabajo (validación de esquema opcional).
-   [Diffs](./tools/diffs.md): visor de diferencias de solo lectura y renderizador de archivos PNG o PDF para texto antes/después o parches unificados.

## Inventario de herramientas

### apply\_patch

Aplica parches estructurados en uno o más archivos. Úsalo para ediciones multi-hunk. Experimental: habilítalo mediante `tools.exec.applyPatch.enabled` (solo modelos de OpenAI). `tools.exec.applyPatch.workspaceOnly` por defecto es `true` (contenido en el espacio de trabajo). Establécelo en `false` solo si intencionalmente quieres que `apply_patch` escriba/elimine fuera del directorio del espacio de trabajo.

### exec

Ejecuta comandos de shell en el espacio de trabajo. Parámetros principales:

-   `command` (requerido)
-   `yieldMs` (auto-fondo después del tiempo de espera, por defecto 10000)
-   `background` (fondo inmediato)
-   `timeout` (segundos; mata el proceso si se excede, por defecto 1800)
-   `elevated` (bool; ejecutar en el host si el modo elevado está habilitado/permitido; solo cambia el comportamiento cuando el agente está en sandbox)
-   `host` (`sandbox | gateway | node`)
-   `security` (`deny | allowlist | full`)
-   `ask` (`off | on-miss | always`)
-   `node` (id/nombre del nodo para `host=node`)
-   ¿Necesitas un TTY real? Establece `pty: true`.

Notas:

-   Devuelve `status: "running"` con un `sessionId` cuando se pone en segundo plano.
-   Usa `process` para sondear/registrar/escribir/matar/limpiar sesiones en segundo plano.
-   Si `process` no está permitido, `exec` se ejecuta de forma sincrónica e ignora `yieldMs`/`background`.
-   `elevated` está controlado por `tools.elevated` más cualquier anulación `agents.list[].tools.elevated` (ambos deben permitir) y es un alias para `host=gateway` + `security=full`.
-   `elevated` solo cambia el comportamiento cuando el agente está en sandbox (de lo contrario es una no-operación).
-   `host=node` puede apuntar a una aplicación compañera de macOS o a un host de nodo sin interfaz gráfica (`openclaw node run`).
-   Aprobaciones y listas de permitidos de gateway/nodo: [Aprobaciones de Exec](./tools/exec-approvals.md).

### process

Gestiona sesiones de exec en segundo plano. Acciones principales:

-   `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

Notas:

-   `poll` devuelve nueva salida y estado de salida cuando se completa.
-   `log` admite `offset`/`limit` basado en líneas (omite `offset` para tomar las últimas N líneas).
-   `process` tiene alcance por agente; las sesiones de otros agentes no son visibles.

### loop-detection (guardarraíles de bucle de llamadas a herramientas)

OpenClaw rastrea el historial reciente de llamadas a herramientas y bloquea o advierte cuando detecta bucles repetitivos sin progreso. Habilítalo con `tools.loopDetection.enabled: true` (por defecto es `false`).

```json
{
  tools: {
    loopDetection: {
      enabled: true,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      historySize: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

-   `genericRepeat`: patrón de llamada repetida de la misma herramienta + mismos parámetros.
-   `knownPollNoProgress`: repetición de herramientas tipo poll con salidas idénticas.
-   `pingPong`: patrones alternantes `A/B/A/B` sin progreso.
-   Anulación por agente: `agents.list[].tools.loopDetection`.

### web\_search

Busca en la web usando Perplexity, Brave, Gemini, Grok o Kimi. Parámetros principales:

-   `query` (requerido)
-   `count` (1–10; por defecto de `tools.web.search.maxResults`)

Notas:

-   Requiere una clave API para el proveedor elegido (recomendado: `openclaw configure --section web`).
-   Habilítalo mediante `tools.web.search.enabled`.
-   Las respuestas se almacenan en caché (por defecto 15 min).
-   Consulta [Herramientas web](./tools/web.md) para configuración.

### web\_fetch

Obtiene y extrae contenido legible de una URL (HTML → markdown/texto). Parámetros principales:

-   `url` (requerido)
-   `extractMode` (`markdown` | `text`)
-   `maxChars` (trunca páginas largas)

Notas:

-   Habilítalo mediante `tools.web.fetch.enabled`.
-   `maxChars` está limitado por `tools.web.fetch.maxCharsCap` (por defecto 50000).
-   Las respuestas se almacenan en caché (por defecto 15 min).
-   Para sitios con mucho JS, prefiere la herramienta del navegador.
-   Consulta [Herramientas web](./tools/web.md) para configuración.
-   Consulta [Firecrawl](./tools/firecrawl.md) para el respaldo opcional anti-bot.

### browser

Controla el navegador dedicado gestionado por OpenClaw. Acciones principales:

-   `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
-   `snapshot` (aria/ai)
-   `screenshot` (devuelve bloque de imagen + `MEDIA:`)
-   `act` (acciones de UI: click/type/press/hover/drag/select/fill/resize/wait/evaluate)
-   `navigate`, `console`, `pdf`, `upload`, `dialog`

Gestión de perfiles:

-   `profiles` — lista todos los perfiles del navegador con estado
-   `create-profile` — crea un nuevo perfil con puerto asignado automáticamente (o `cdpUrl`)
-   `delete-profile` — detiene el navegador, elimina datos de usuario, lo elimina de la configuración (solo local)
-   `reset-profile` — mata el proceso huérfano en el puerto del perfil (solo local)

Parámetros comunes:

-   `profile` (opcional; por defecto `browser.defaultProfile`)
-   `target` (`sandbox` | `host` | `node`)
-   `node` (opcional; elige un id/nombre de nodo específico) Notas:
-   Requiere `browser.enabled=true` (por defecto es `true`; establece `false` para deshabilitar).
-   Todas las acciones aceptan el parámetro opcional `profile` para soporte multi-instancia.
-   Cuando se omite `profile`, usa `browser.defaultProfile` (por defecto "chrome").
-   Nombres de perfil: solo minúsculas alfanuméricas + guiones (máx. 64 caracteres).
-   Rango de puertos: 18800-18899 (~100 perfiles máx.).
-   Los perfiles remotos son solo de adjuntar (sin start/stop/reset).
-   Si un nodo con capacidad de navegador está conectado, la herramienta puede enrutar automáticamente a él (a menos que fijes `target`).
-   `snapshot` por defecto es `ai` cuando Playwright está instalado; usa `aria` para el árbol de accesibilidad.
-   `snapshot` también admite opciones de snapshot de roles (`interactive`, `compact`, `depth`, `selector`) que devuelven referencias como `e12`.
-   `act` requiere `ref` de `snapshot` (`12` numérico de snapshots AI, o `e12` de snapshots de roles); usa `evaluate` para necesidades raras de selector CSS.
-   Evita `act` → `wait` por defecto; úsalo solo en casos excepcionales (sin estado de UI confiable para esperar).
-   `upload` puede opcionalmente pasar un `ref` para hacer clic automáticamente después de armar.
-   `upload` también admite `inputRef` (ref aria) o `element` (selector CSS) para establecer `` directamente.

### canvas

Controla el Canvas del nodo (presentar, eval, snapshot, A2UI). Acciones principales:

-   `present`, `hide`, `navigate`, `eval`
-   `snapshot` (devuelve bloque de imagen + `MEDIA:`)
-   `a2ui_push`, `a2ui_reset`

Notas:

-   Usa `node.invoke` del gateway internamente.
-   Si no se proporciona `node`, la herramienta elige uno por defecto (nodo único conectado o nodo mac local).
-   A2UI es solo v0.8 (no `createSurface`); la CLI rechaza JSONL v0.9 con errores de línea.
-   Prueba rápida: `openclaw nodes canvas a2ui push --node  --text "Hello from A2UI"`.

### nodes

Descubre y apunta a nodos emparejados; envía notificaciones; captura cámara/pantalla. Acciones principales:

-   `status`, `describe`
-   `pending`, `approve`, `reject` (emparejamiento)
-   `notify` (macOS `system.notify`)
-   `run` (macOS `system.run`)
-   `camera_list`, `camera_snap`, `camera_clip`, `screen_record`
-   `location_get`, `notifications_list`, `notifications_action`
-   `device_status`, `device_info`, `device_permissions`, `device_health`

Notas:

-   Los comandos de cámara/pantalla requieren que la aplicación del nodo esté en primer plano.
-   Las imágenes devuelven bloques de imagen + `MEDIA:`.
-   Los videos devuelven `FILE:` (mp4).
-   La ubicación devuelve un payload JSON (lat/lon/precisión/marca de tiempo).
-   Parámetros de `run`: array `command` argv; opcional `cwd`, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording`.

Ejemplo (`run`):

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

### image

Analiza una imagen con el modelo de imagen configurado. Parámetros principales:

-   `image` (ruta o URL requerida)
-   `prompt` (opcional; por defecto "Describe la imagen.")
-   `model` (anulación opcional)
-   `maxBytesMb` (límite de tamaño opcional)

Notas:

-   Solo disponible cuando `agents.defaults.imageModel` está configurado (primario o respaldos), o cuando se puede inferir un modelo de imagen implícito de tu modelo por defecto + autenticación configurada (emparejamiento de mejor esfuerzo).
-   Usa el modelo de imagen directamente (independiente del modelo de chat principal).

### pdf

Analiza uno o más documentos PDF. Para comportamiento completo, límites, configuración y ejemplos, consulta [Herramienta PDF](./tools/pdf.md).

### message

Envía mensajes y acciones de canal a través de Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams. Acciones principales:

-   `send` (texto + medios opcionales; MS Teams también admite `card` para Adaptive Cards)
-   `poll` (encuestas de WhatsApp/Discord/MS Teams)
-   `react` / `reactions` / `read` / `edit` / `delete`
-   `pin` / `unpin` / `list-pins`
-   `permissions`
-   `thread-create` / `thread-list` / `thread-reply`
-   `search`
-   `sticker`
-   `member-info` / `role-info`
-   `emoji-list` / `emoji-upload` / `sticker-upload`
-   `role-add` / `role-remove`
-   `channel-info` / `channel-list`
-   `voice-status`
-   `event-list` / `event-create`
-   `timeout` / `kick` / `ban`

Notas:

-   `send` enruta WhatsApp a través del Gateway; otros canales van directo.
-   `poll` usa el Gateway para WhatsApp y MS Teams; las encuestas de Discord van directo.
-   Cuando una llamada a herramienta de mensaje está vinculada a una sesión de chat activa, los envíos están restringidos al objetivo de esa sesión para evitar fugas entre contextos.

### cron

Gestiona trabajos cron y activaciones del Gateway. Acciones principales:

-   `status`, `list`
-   `add`, `update`, `remove`, `run`, `runs`
-   `wake` (encola evento del sistema + latido inmediato opcional)

Notas:

-   `add` espera un objeto de trabajo cron completo (mismo esquema que `cron.add` RPC).
-   `update` usa `{ jobId, patch }` (`id` aceptado por compatibilidad).

### gateway

Reinicia o aplica actualizaciones al proceso Gateway en ejecución (en el lugar). Acciones principales:

-   `restart` (autoriza + envía `SIGUSR1` para reinicio en proceso; `openclaw gateway` reinicia en el lugar)
-   `config.schema.lookup` (inspecciona una ruta de configuración a la vez sin cargar el esquema completo en el contexto del prompt)
-   `config.get`
-   `config.apply` (valida + escribe configuración + reinicia + activa)
-   `config.patch` (fusiona actualización parcial + reinicia + activa)
-   `update.run` (ejecuta actualización + reinicia + activa)

Notas:

-   `config.schema.lookup` espera una ruta de configuración específica como `gateway.auth` o `agents.list.*.heartbeat`.
-   Las rutas pueden incluir ids de plugin delimitados por barra cuando se dirigen a `plugins.entries.`, por ejemplo `plugins.entries.pack/one.config`.
-   Usa `delayMs` (por defecto 2000) para evitar interrumpir una respuesta en curso.
-   `config.schema` permanece disponible para flujos internos de la UI de Control y no se expone a través de la herramienta `gateway` del agente.
-   `restart` está habilitado por defecto; establece `commands.restart: false` para deshabilitarlo.

### sessions\_list / sessions\_history / sessions\_send / sessions\_spawn / session\_status

Lista sesiones, inspecciona historial de transcripción o envía a otra sesión. Parámetros principales:

-   `sessions_list`: `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = ninguno)
-   `sessions_history`: `sessionKey` (o `sessionId`), `limit?`, `includeTools?`
-   `sessions_send`: `sessionKey` (o `sessionId`), `message`, `timeoutSeconds?` (0 = disparar y olvidar)
-   `sessions_spawn`: `task`, `label?`, `runtime?`, `agentId?`, `model?`, `thinking?`, `cwd?`, `runTimeoutSeconds?`, `thread?`, `mode?`, `cleanup?`, `sandbox?`, `streamTo?`, `attachments?`, `attachAs?`
-   `session_status`: `sessionKey?` (por defecto actual; acepta `sessionId`), `model?` (`default` borra la anulación)

Notas:

-   `main` es la clave de chat directo canónica; global/desconocido están ocultos.
-   `messageLimit > 0` obtiene los últimos N mensajes por sesión (mensajes de herramientas filtrados).
-   El direccionamiento de sesiones está controlado por `tools.sessions.visibility` (por defecto `tree`: sesión actual + sesiones de subagentes generadas). Si ejecutas un agente compartido para múltiples usuarios, considera establecer `tools.sessions.visibility: "self"` para evitar la navegación entre sesiones.
-   `sessions_send` espera la finalización cuando `timeoutSeconds > 0`.
-   La entrega/anuncio ocurre después de la finalización y es de mejor esfuerzo; `status: "ok"` confirma que la ejecución del agente terminó, no que el anuncio fue entregado.
-   `sessions_spawn` admite `runtime: "subagent" | "acp"` (`subagent` por defecto). Para el comportamiento del runtime ACP, consulta [Agentes ACP](./tools/acp-agents.md).
-   Para el runtime ACP, `streamTo: "parent"` enruta resúmenes de progreso de la ejecución inicial de vuelta a la sesión solicitante como eventos del sistema en lugar de entrega directa al hijo.
-   `sessions_spawn` inicia una ejecución de sub-agente y publica una respuesta de anuncio de vuelta al chat del solicitante.
    -   Admite modo de una sola vez (`mode: "run"`) y modo persistente vinculado a hilo (`mode: "session"` con `thread: true`).
    -   Si `thread: true` y se omite `mode`, el modo por defecto es `session`.
    -   `mode: "session"` requiere `thread: true`.
    -   Si se omite `runTimeoutSeconds`, OpenClaw usa `agents.defaults.subagents.runTimeoutSeconds` cuando está configurado; de lo contrario, el tiempo de espera por defecto es `0` (sin tiempo de espera).
    -   Los flujos vinculados a hilos de Discord dependen de `session.threadBindings.*` y `channels.discord.threadBindings.*`.
    -   El formato de respuesta incluye `Status`, `Result` y estadísticas compactas.
    -   `Result` es el texto de finalización del asistente; si falta, se usa el último `toolResult` como respaldo.
-   Los spawns en modo de finalización manual envían directamente primero, con respaldo de cola y reintento en fallos transitorios (`status: "ok"` significa que la ejecución terminó, no que el anuncio fue entregado).
-   `sessions_spawn` admite archivos adjuntos en línea solo para el runtime de subagente (ACP los rechaza). Cada adjunto tiene `name`, `content` y opcional `encoding` (`utf8` o `base64`) y `mimeType`. Los archivos se materializan en el espacio de trabajo hijo en `.openclaw/attachments//` con un archivo de metadatos `.manifest.json`. La herramienta devuelve un recibo con `count`, `totalBytes`, `sha256` por archivo y `relDir`. El contenido del adjunto se redacta automáticamente de la persistencia de la transcripción.
    -   Configura límites mediante `tools.sessions_spawn.attachments` (`enabled`, `maxTotalBytes`, `maxFiles`, `maxFileBytes`, `retainOnSessionKeep`).
    -   `attachAs.mountPath` es una pista reservada para futuras implementaciones de montaje.
-   `sessions_spawn` es no bloqueante y devuelve `status: "accepted"` inmediatamente.
-   Las respuestas de ACP `streamTo: "parent"` pueden incluir `streamLogPath` (sesión con alcance `*.acp-stream.jsonl`) para seguir el historial de progreso.
-   `sessions_send` ejecuta un ping‑pong de respuesta (responde `REPLY_SKIP` para detener; turnos máx. mediante `session.agentToAgent.maxPingPongTurns`, 0–5).
-   Después del ping‑pong, el agente objetivo ejecuta un **paso de anuncio**; responde `ANNOUNCE_SKIP` para suprimir el anuncio.
-   Limitación de sandbox: cuando la sesión actual está en sandbox y `agents.defaults.sandbox.sessionToolsVisibility: "spawned"`, OpenClaw limita `tools.sessions.visibility` a `tree`.

### agents\_list

Lista los ids de agentes que la sesión actual puede apuntar con `sessions_spawn`. Notas:

-   El resultado está restringido a las listas de permitidos por agente (`agents.list[].subagents.allowAgents`).
-   Cuando se configura `["*"]`, la herramienta incluye todos los agentes configurados y marca `allowAny: true`.

## Parámetros (comunes)

Herramientas respaldadas por Gateway (`canvas`, `nodes`, `cron`):

-   `gatewayUrl` (por defecto `ws://127.0.0.1:18789`)
-   `gatewayToken` (si la autenticación está habilitada)
-   `timeoutMs`

Nota: cuando se establece `gatewayUrl`, incluye `gatewayToken` explícitamente. Las herramientas no heredan credenciales de configuración o entorno para anulaciones, y la falta de credenciales explícitas es un error. Herramienta del navegador:

-   `profile` (opcional; por defecto `browser.defaultProfile`)
-   `target` (`sandbox` | `host` | `node`)
-   `node` (opcional; fija un id/nombre de nodo específico)

## Flujos de agente recomendados

Automatización del navegador:

1.  `browser` → `status` / `start`
2.  `snapshot` (ai o aria)
3.  `act` (click/type/press)
4.  `screenshot` si necesitas confirmación visual

Renderizado de Canvas:

1.  `canvas` → `present`
2.  `a2ui_push` (opcional)
3.  `snapshot`

Direccionamiento de nodos:

1.  `nodes` → `status`
2.  `describe` en el nodo elegido
3.  `notify` / `run` / `camera_snap` / `screen_record`

## Seguridad

-   Evita `system.run` directo; usa `nodes` → `run` solo con consentimiento explícito del usuario.
-   Respeta el consentimiento del usuario para captura de cámara/pantalla.
-   Usa `status/describe` para asegurar permisos antes de invocar comandos de medios.

## Cómo se presentan las herramientas al agente

Las herramientas se exponen en dos canales paralelos:

1.  **Texto del prompt del sistema**: una lista legible por humanos + guía.
2.  **Esquema de herramientas**: las definiciones de funciones estructuradas enviadas a la API del modelo.

Eso significa que el agente ve tanto "qué herramientas existen" como "cómo llamarlas". Si una herramienta no aparece en el prompt del sistema o en el esquema, el modelo no puede llamarla.

[apply\_patch Tool](./tools/apply-patch.md)

---