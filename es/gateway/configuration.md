

  Configuración y operaciones

  
# Configuración

OpenClaw lee una configuración **JSON5** opcional desde `~/.openclaw/openclaw.json`. Si el archivo falta, OpenClaw usa valores predeterminados seguros. Razones comunes para agregar una configuración:

-   Conectar canales y controlar quién puede enviar mensajes al bot
-   Establecer modelos, herramientas, sandboxing o automatización (cron, hooks)
-   Ajustar sesiones, medios, redes o UI

Consulta la [referencia completa](./configuration-reference.md) para cada campo disponible.

> **💡** **¿Nuevo en configuración?** Comienza con `openclaw onboard` para una configuración interactiva, o revisa la guía [Ejemplos de Configuración](./configuration-examples.md) para configuraciones completas para copiar y pegar.

## Configuración mínima

```
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Editar configuración

```bash
openclaw onboard       # asistente de configuración completo
openclaw configure     # asistente de configuración
```

```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```

Abre [http://127.0.0.1:18789](http://127.0.0.1:18789) y usa la pestaña **Config**. La UI de Control renderiza un formulario desde el esquema de configuración, con un editor **Raw JSON** como salida de emergencia.

Edita `~/.openclaw/openclaw.json` directamente. El Gateway observa el archivo y aplica los cambios automáticamente (ver [recarga en caliente](#config-hot-reload)).

## Validación estricta

> **⚠️** OpenClaw solo acepta configuraciones que coincidan completamente con el esquema. Claves desconocidas, tipos malformados o valores inválidos hacen que el Gateway **se niegue a iniciar**. La única excepción a nivel raíz es `$schema` (string), para que los editores puedan adjuntar metadatos de JSON Schema.

 Cuando falla la validación:

-   El Gateway no arranca
-   Solo funcionan los comandos de diagnóstico (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
-   Ejecuta `openclaw doctor` para ver los problemas exactos
-   Ejecuta `openclaw doctor --fix` (o `--yes`) para aplicar reparaciones

## Tareas comunes

Cada canal tiene su propia sección de configuración bajo `channels.`. Consulta la página dedicada del canal para los pasos de configuración:

-   [WhatsApp](../channels/whatsapp.md) — `channels.whatsapp`
-   [Telegram](../channels/telegram.md) — `channels.telegram`
-   [Discord](../channels/discord.md) — `channels.discord`
-   [Slack](../channels/slack.md) — `channels.slack`
-   [Signal](../channels/signal.md) — `channels.signal`
-   [iMessage](../channels/imessage.md) — `channels.imessage`
-   [Google Chat](../channels/googlechat.md) — `channels.googlechat`
-   [Mattermost](../channels/mattermost.md) — `channels.mattermost`
-   [MS Teams](../channels/msteams.md) — `channels.msteams`

Todos los canales comparten el mismo patrón de política de DM:

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",   // pairing | allowlist | open | disabled
      allowFrom: ["tg:123"], // solo para allowlist/open
    },
  },
}
```

Establece el modelo principal y respaldos opcionales:

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5.2"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "openai/gpt-5.2": { alias: "GPT" },
      },
    },
  },
}
```

-   `agents.defaults.models` define el catálogo de modelos y actúa como la lista de permitidos para `/model`.
-   Las referencias de modelo usan el formato `provider/model` (ej. `anthropic/claude-opus-4-6`).
-   `agents.defaults.imageMaxDimensionPx` controla la reducción de escala de imágenes de transcripción/herramienta (predeterminado `1200`); valores más bajos generalmente reducen el uso de tokens de visión en ejecuciones con muchas capturas de pantalla.
-   Consulta [CLI de Modelos](../concepts/models.md) para cambiar modelos en el chat y [Model Failover](../concepts/model-failover.md) para la rotación de autenticación y el comportamiento de respaldo.
-   Para proveedores personalizados/autoalojados, consulta [Proveedores personalizados](./configuration-reference.md#custom-providers-and-base-urls) en la referencia.

El acceso a DM se controla por canal mediante `dmPolicy`:

-   `"pairing"` (predeterminado): los remitentes desconocidos reciben un código de emparejamiento de un solo uso para aprobar
-   `"allowlist"`: solo remitentes en `allowFrom` (o el almacén de permitidos emparejado)
-   `"open"`: permitir todos los DM entrantes (requiere `allowFrom: ["*"]`)
-   `"disabled"`: ignorar todos los DM

Para grupos, usa `groupPolicy` + `groupAllowFrom` o listas de permitidos específicas del canal. Consulta la [referencia completa](./configuration-reference.md#dm-and-group-access) para detalles por canal.

Los mensajes grupales requieren **mención** por defecto. Configura patrones por agente:

```json
{
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw"],
        },
      },
    ],
  },
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

-   **Menciones de metadatos**: menciones nativas @ (WhatsApp tocar-para-mencionar, Telegram @bot, etc.)
-   **Patrones de texto**: patrones regex en `mentionPatterns`
-   Consulta la [referencia completa](./configuration-reference.md#group-chat-mention-gating) para anulaciones por canal y modo de auto-chat.

Las sesiones controlan la continuidad y el aislamiento de la conversación:

```json
{
  session: {
    dmScope: "per-channel-peer",  // recomendado para multi-usuario
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
  },
}
```

-   `dmScope`: `main` (compartido) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
-   `threadBindings`: valores predeterminados globales para el enrutamiento de sesión vinculado a hilo (Discord soporta `/focus`, `/unfocus`, `/agents`, `/session idle`, y `/session max-age`).
-   Consulta [Gestión de Sesiones](../concepts/session.md) para alcance, enlaces de identidad y política de envío.
-   Consulta la [referencia completa](./configuration-reference.md#session) para todos los campos.

Ejecuta sesiones de agente en contenedores Docker aislados:

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",  // off | non-main | all
        scope: "agent",    // session | agent | shared
      },
    },
  },
}
```

Construye la imagen primero: `scripts/sandbox-setup.sh` Consulta [Sandboxing](./sandboxing.md) para la guía completa y la [referencia completa](./configuration-reference.md#sandbox) para todas las opciones.

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
  },
}
```

-   `every`: cadena de duración (`30m`, `2h`). Establece `0m` para deshabilitar.
-   `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
-   `directPolicy`: `allow` (predeterminado) o `block` para objetivos de heartbeat estilo DM
-   Consulta [Heartbeat](./heartbeat.md) para la guía completa.

```json
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h",
    runLog: {
      maxBytes: "2mb",
      keepLines: 2000,
    },
  },
}
```

-   `sessionRetention`: poda sesiones de ejecución aisladas completadas de `sessions.json` (predeterminado `24h`; establece `false` para deshabilitar).
-   `runLog`: poda `cron/runs/.jsonl` por tamaño y líneas retenidas.
-   Consulta [Trabajos cron](../automation/cron-jobs.md) para la descripción general de la función y ejemplos de CLI.

Habilita endpoints HTTP de webhook en el Gateway:

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "main",
        deliver: true,
      },
    ],
  },
}
```

Nota de seguridad:

-   Trata todo el contenido de la carga útil del hook/webhook como entrada no confiable.
-   Mantén las banderas de omisión de contenido no seguro deshabilitadas (`hooks.gmail.allowUnsafeExternalContent`, `hooks.mappings[].allowUnsafeExternalContent`) a menos que estés haciendo depuración de alcance estricto.
-   Para agentes impulsados por hooks, prefiere niveles de modelo modernos fuertes y política de herramientas estricta (por ejemplo, solo mensajería más sandboxing donde sea posible).

Consulta la [referencia completa](./configuration-reference.md#hooks) para todas las opciones de mapeo e integración de Gmail.

Ejecuta múltiples agentes aislados con espacios de trabajo y sesiones separados:

```json
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

Consulta [Multi-Agent](../concepts/multi-agent.md) y la [referencia completa](./configuration-reference.md#multi-agent-routing) para reglas de vinculación y perfiles de acceso por agente.

Usa `$include` para organizar configuraciones grandes:

```
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/a.json5", "./clients/b.json5"],
  },
}
```

-   **Archivo único**: reemplaza el objeto contenedor
-   **Array de archivos**: se fusionan en profundidad en orden (el último gana)
-   **Claves hermanas**: se fusionan después de los includes (anulan los valores incluidos)
-   **Includes anidados**: soportados hasta 10 niveles de profundidad
-   **Rutas relativas**: resueltas relativas al archivo que incluye
-   **Manejo de errores**: errores claros para archivos faltantes, errores de análisis e includes circulares

## Recarga en caliente de configuración

El Gateway observa `~/.openclaw/openclaw.json` y aplica los cambios automáticamente — no se necesita reinicio manual para la mayoría de los ajustes.

### Modos de recarga

| Modo | Comportamiento |
| --- | --- |
| **`hybrid`** (predeterminado) | Aplica cambios seguros al instante. Reinicia automáticamente para los críticos. |
| **`hot`** | Solo aplica cambios seguros en caliente. Registra una advertencia cuando se necesita un reinicio — lo manejas tú. |
| **`restart`** | Reinicia el Gateway en cualquier cambio de configuración, seguro o no. |
| **`off`** | Desactiva la observación de archivos. Los cambios surten efecto en el próximo reinicio manual. |

```json
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Qué se aplica en caliente vs qué necesita un reinicio

La mayoría de los campos se aplican en caliente sin tiempo de inactividad. En modo `hybrid`, los cambios que requieren reinicio se manejan automáticamente.

| Categoría | Campos | ¿Se necesita reinicio? |
| --- | --- | --- |
| Canales | `channels.*`, `web` (WhatsApp) — todos los canales integrados y de extensión | No |
| Agente y modelos | `agent`, `agents`, `models`, `routing` | No |
| Automatización | `hooks`, `cron`, `agent.heartbeat` | No |
| Sesiones y mensajes | `session`, `messages` | No |
| Herramientas y medios | `tools`, `browser`, `skills`, `audio`, `talk` | No |
| UI y misceláneos | `ui`, `logging`, `identity`, `bindings` | No |
| Servidor Gateway | `gateway.*` (puerto, bind, auth, tailscale, TLS, HTTP) | **Sí** |
| Infraestructura | `discovery`, `canvasHost`, `plugins` | **Sí** |

> **ℹ️** `gateway.reload` y `gateway.remote` son excepciones — cambiarlos **no** desencadena un reinicio.

## RPC de configuración (actualizaciones programáticas)

> **ℹ️** Los RPC de escritura del plano de control (`config.apply`, `config.patch`, `update.run`) están limitados a **3 solicitudes por 60 segundos** por `deviceId+clientIp`. Cuando se limita, el RPC devuelve `UNAVAILABLE` con `retryAfterMs`.

 

Valida + escribe la configuración completa y reinicia el Gateway en un solo paso.

`config.apply` reemplaza la **configuración completa**. Usa `config.patch` para actualizaciones parciales, o `openclaw config set` para claves individuales.

Parámetros:

-   `raw` (string) — carga útil JSON5 para toda la configuración
-   `baseHash` (opcional) — hash de configuración de `config.get` (requerido cuando existe la configuración)
-   `sessionKey` (opcional) — clave de sesión para el ping de reactivación posterior al reinicio
-   `note` (opcional) — nota para el centinela de reinicio
-   `restartDelayMs` (opcional) — retraso antes del reinicio (predeterminado 2000)

Las solicitudes de reinicio se combinan mientras una ya está pendiente/en vuelo, y se aplica un enfriamiento de 30 segundos entre ciclos de reinicio.

```bash
openclaw gateway call config.get --params '{}'  # capturar payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
  "baseHash": "<hash>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123"
}'
```

Fusiona una actualización parcial en la configuración existente (semántica de parche de fusión JSON):

-   Los objetos se fusionan recursivamente
-   `null` elimina una clave
-   Los arrays se reemplazan

Parámetros:

-   `raw` (string) — JSON5 con solo las claves a cambiar
-   `baseHash` (requerido) — hash de configuración de `config.get`
-   `sessionKey`, `note`, `restartDelayMs` — igual que `config.apply`

El comportamiento de reinicio coincide con `config.apply`: reinicios pendientes combinados más un enfriamiento de 30 segundos entre ciclos de reinicio.

```bash
openclaw gateway call config.patch --params '{
  "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
  "baseHash": "<hash>"
}'
```

## Variables de entorno

OpenClaw lee variables de entorno del proceso padre más:

-   `.env` desde el directorio de trabajo actual (si está presente)
-   `~/.openclaw/.env` (respaldo global)

Ningún archivo anula las variables de entorno existentes. También puedes establecer variables de entorno en línea en la configuración:

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

Si está habilitado y las claves esperadas no están configuradas, OpenClaw ejecuta tu shell de inicio de sesión e importa solo las claves faltantes:

```json
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Variable de entorno equivalente: `OPENCLAW_LOAD_SHELL_ENV=1`

 

Referencia variables de entorno en cualquier valor de cadena de configuración con `${VAR_NAME}`:

```json
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Reglas:

-   Solo nombres en mayúsculas coinciden: `[A-Z_][A-Z0-9_]*`
-   Las variables faltantes/vacías lanzan un error al momento de carga
-   Escapa con `$${VAR}` para salida literal
-   Funciona dentro de archivos `$include`
-   Sustitución en línea: `"${BASE}/v1"` → `"https://api.example.com/v1"`

 

Para campos que admiten objetos SecretRef, puedes usar:

```json
{
  models: {
    providers: {
      openai: { apiKey: { source: "env", provider: "default", id: "OPENAI_API_KEY" } },
    },
  },
  skills: {
    entries: {
      "nano-banana-pro": {
        apiKey: {
          source: "file",
          provider: "filemain",
          id: "/skills/entries/nano-banana-pro/apiKey",
        },
      },
    },
  },
  channels: {
    googlechat: {
      serviceAccountRef: {
        source: "exec",
        provider: "vault",
        id: "channels/googlechat/serviceAccount",
      },
    },
  },
}
```

Los detalles de SecretRef (incluyendo `secrets.providers` para `env`/`file`/`exec`) están en [Gestión de Secretos](./secrets.md). Las rutas de credenciales admitidas se enumeran en [Superficie de Credenciales SecretRef](../reference/secretref-credential-surface.md).

 Consulta [Entorno](../help/environment.md) para la precedencia completa y fuentes.

## Referencia completa

Para la referencia completa campo por campo, consulta **[Referencia de Configuración](./configuration-reference.md)**.

* * *

*Relacionado: [Ejemplos de Configuración](./configuration-examples.md) · [Referencia de Configuración](./configuration-reference.md) · [Doctor](./doctor.md)*

[Runbook del Gateway](../gateway.md)[Referencia de Configuración](./configuration-reference.md)