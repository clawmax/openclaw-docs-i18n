

  Multi-agente

  
# Enrutamiento Multi-Agente

Objetivo: múltiples agentes *aislados* (espacio de trabajo + `agentDir` + sesiones separados), además de múltiples cuentas de canal (ej. dos WhatsApps) en un Gateway en ejecución. El tráfico entrante se enruta a un agente mediante vinculaciones.

## ¿Qué es "un agente"?

Un **agente** es un cerebro completamente delimitado con su propio:

-   **Espacio de trabajo** (archivos, AGENTS.md/SOUL.md/USER.md, notas locales, reglas de personalidad).
-   **Directorio de estado** (`agentDir`) para perfiles de autenticación, registro de modelos y configuración por agente.
-   **Almacén de sesiones** (historial de chat + estado de enrutamiento) bajo `~/.openclaw/agents//sessions`.

Los perfiles de autenticación son **por agente**. Cada agente lee desde su propio:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Las credenciales del agente principal **no** se comparten automáticamente. Nunca reutilices `agentDir` entre agentes (causa colisiones de autenticación/sesión). Si quieres compartir credenciales, copia `auth-profiles.json` en el `agentDir` del otro agente. Las habilidades son por agente a través de la carpeta `skills/` de cada espacio de trabajo, con habilidades compartidas disponibles desde `~/.openclaw/skills`. Consulta [Habilidades: por agente vs compartidas](../tools/skills.md#per-agent-vs-shared-skills). El Gateway puede alojar **un agente** (predeterminado) o **muchos agentes** en paralelo. **Nota sobre el espacio de trabajo:** el espacio de trabajo de cada agente es el **cwd predeterminado**, no un sandbox estricto. Las rutas relativas se resuelven dentro del espacio de trabajo, pero las rutas absolutas pueden alcanzar otras ubicaciones del host a menos que se habilite el sandboxing. Consulta [Sandboxing](../gateway/sandboxing.md).

## Rutas (mapa rápido)

-   Configuración: `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`)
-   Directorio de estado: `~/.openclaw` (o `OPENCLAW_STATE_DIR`)
-   Espacio de trabajo: `~/.openclaw/workspace` (o `~/.openclaw/workspace-`)
-   Directorio del agente: `~/.openclaw/agents//agent` (o `agents.list[].agentDir`)
-   Sesiones: `~/.openclaw/agents//sessions`

### Modo agente único (predeterminado)

Si no haces nada, OpenClaw ejecuta un solo agente:

-   `agentId` es por defecto **`main`**.
-   Las sesiones se identifican como `agent:main:`.
-   El espacio de trabajo predeterminado es `~/.openclaw/workspace` (o `~/.openclaw/workspace-` cuando se establece `OPENCLAW_PROFILE`).
-   El estado predeterminado es `~/.openclaw/agents/main/agent`.

## Asistente de agentes

Usa el asistente de agentes para añadir un nuevo agente aislado:

```bash
openclaw agents add work
```

Luego añade `bindings` (o deja que el asistente lo haga) para enrutar los mensajes entrantes. Verifica con:

```bash
openclaw agents list --bindings
```

## Inicio rápido

### Paso 1: Crear el espacio de trabajo de cada agente

Usa el asistente o crea espacios de trabajo manualmente:

```bash
openclaw agents add coding
openclaw agents add social
```

Cada agente obtiene su propio espacio de trabajo con `SOUL.md`, `AGENTS.md`, y opcionalmente `USER.md`, además de un `agentDir` dedicado y un almacén de sesiones bajo `~/.openclaw/agents/`.

### Paso 2: Crear cuentas de canal

Crea una cuenta por agente en tus canales preferidos:

-   Discord: un bot por agente, habilita Message Content Intent, copia cada token.
-   Telegram: un bot por agente a través de BotFather, copia cada token.
-   WhatsApp: vincula cada número de teléfono por cuenta.

```bash
openclaw channels login --channel whatsapp --account work
```

Consulta las guías de canales: [Discord](../channels/discord.md), [Telegram](../channels/telegram.md), [WhatsApp](../channels/whatsapp.md).

### Paso 3: Añadir agentes, cuentas y vinculaciones

Añade agentes bajo `agents.list`, cuentas de canal bajo `channels..accounts`, y conéctalas con `bindings` (ejemplos a continuación).

### Paso 4: Reiniciar y verificar

```bash
openclaw gateway restart
openclaw agents list --bindings
openclaw channels status --probe
```

## Múltiples agentes = múltiples personas, múltiples personalidades

Con **múltiples agentes**, cada `agentId` se convierte en una **personalidad completamente aislada**:

-   **Diferentes números de teléfono/cuentas** (por `accountId` de canal).
-   **Diferentes personalidades** (archivos del espacio de trabajo por agente como `AGENTS.md` y `SOUL.md`).
-   **Autenticación + sesiones separadas** (sin interferencia a menos que se habilite explícitamente).

Esto permite que **múltiples personas** compartan un servidor Gateway mientras mantienen sus "cerebros" de IA y datos aislados.

## Un número de WhatsApp, múltiples personas (división de MD)

Puedes enrutar **MDs de WhatsApp diferentes** a diferentes agentes mientras permaneces en **una cuenta de WhatsApp**. Coincide con el E.164 del remitente (como `+15551234567`) usando `peer.kind: "direct"`. Las respuestas aún provienen del mismo número de WhatsApp (sin identidad de remitente por agente). Detalle importante: los chats directos se colapsan a la **clave de sesión principal** del agente, por lo que el aislamiento verdadero requiere **un agente por persona**. Ejemplo:

```json
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" },
    ],
  },
  bindings: [
    {
      agentId: "alex",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230001" } },
    },
    {
      agentId: "mia",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230002" } },
    },
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"],
    },
  },
}
```

Notas:

-   El control de acceso a MDs es **global por cuenta de WhatsApp** (emparejamiento/lista de permitidos), no por agente.
-   Para grupos compartidos, vincula el grupo a un agente o usa [Grupos de difusión](../channels/broadcast-groups.md).

## Reglas de enrutamiento (cómo los mensajes eligen un agente)

Las vinculaciones son **deterministas** y **gana la más específica**:

1.  Coincidencia de `peer` (id exacto de MD/grupo/canal)
2.  Coincidencia de `parentPeer` (herencia de hilo)
3.  `guildId + roles` (enrutamiento por rol de Discord)
4.  `guildId` (Discord)
5.  `teamId` (Slack)
6.  Coincidencia de `accountId` para un canal
7.  Coincidencia a nivel de canal (`accountId: "*"`)
8.  Fallback al agente predeterminado (`agents.list[].default`, si no, la primera entrada de la lista, predeterminado: `main`)

Si múltiples vinculaciones coinciden en el mismo nivel, gana la primera en el orden de configuración. Si una vinculación establece múltiples campos de coincidencia (por ejemplo `peer` + `guildId`), todos los campos especificados son requeridos (semántica `AND`). Detalle importante del alcance de la cuenta:

-   Una vinculación que omite `accountId` coincide solo con la cuenta predeterminada.
-   Usa `accountId: "*"` para un fallback a nivel de canal en todas las cuentas.
-   Si luego añades la misma vinculación para el mismo agente con un id de cuenta explícito, OpenClaw actualiza la vinculación existente solo de canal a una con alcance de cuenta en lugar de duplicarla.

## Múltiples cuentas / números de teléfono

Los canales que admiten **múltiples cuentas** (ej. WhatsApp) usan `accountId` para identificar cada inicio de sesión. Cada `accountId` puede enrutarse a un agente diferente, por lo que un servidor puede alojar múltiples números de teléfono sin mezclar sesiones. Si quieres una cuenta predeterminada a nivel de canal cuando se omite `accountId`, establece `channels..defaultAccount` (opcional). Cuando no está establecido, OpenClaw recurre a `default` si está presente, de lo contrario al primer id de cuenta configurado (ordenado). Los canales comunes que admiten este patrón incluyen:

-   `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`
-   `irc`, `line`, `googlechat`, `mattermost`, `matrix`, `nextcloud-talk`
-   `bluebubbles`, `zalo`, `zalouser`, `nostr`, `feishu`

## Conceptos

-   `agentId`: un "cerebro" (espacio de trabajo, autenticación por agente, almacén de sesiones por agente).
-   `accountId`: una instancia de cuenta de canal (ej. cuenta de WhatsApp `"personal"` vs `"biz"`).
-   `binding`: enruta mensajes entrantes a un `agentId` por `(channel, accountId, peer)` y opcionalmente ids de gremio/equipo.
-   Los chats directos se colapsan a `agent::` ("principal" por agente; `session.mainKey`).

## Ejemplos por plataforma

### Bots de Discord por agente

Cada cuenta de bot de Discord se asigna a un `accountId` único. Vincula cada cuenta a un agente y mantén listas de permitidos por bot.

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "coding", workspace: "~/.openclaw/workspace-coding" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "discord", accountId: "default" } },
    { agentId: "coding", match: { channel: "discord", accountId: "coding" } },
  ],
  channels: {
    discord: {
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN_MAIN",
          guilds: {
            "123456789012345678": {
              channels: {
                "222222222222222222": { allow: true, requireMention: false },
              },
            },
          },
        },
        coding: {
          token: "DISCORD_BOT_TOKEN_CODING",
          guilds: {
            "123456789012345678": {
              channels: {
                "333333333333333333": { allow: true, requireMention: false },
              },
            },
          },
        },
      },
    },
  },
}
```

Notas:

-   Invita a cada bot al gremio y habilita Message Content Intent.
-   Los tokens residen en `channels.discord.accounts..token` (la cuenta predeterminada puede usar `DISCORD_BOT_TOKEN`).

### Bots de Telegram por agente

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "alerts", workspace: "~/.openclaw/workspace-alerts" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "alerts", match: { channel: "telegram", accountId: "alerts" } },
  ],
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "123456:ABC...",
          dmPolicy: "pairing",
        },
        alerts: {
          botToken: "987654:XYZ...",
          dmPolicy: "allowlist",
          allowFrom: ["tg:123456789"],
        },
      },
    },
  },
}
```

Notas:

-   Crea un bot por agente con BotFather y copia cada token.
-   Los tokens residen en `channels.telegram.accounts..botToken` (la cuenta predeterminada puede usar `TELEGRAM_BOT_TOKEN`).

### Números de WhatsApp por agente

Vincula cada cuenta antes de iniciar el gateway:

```bash
openclaw channels login --channel whatsapp --account personal
openclaw channels login --channel whatsapp --account biz
```

`~/.openclaw/openclaw.json` (JSON5):

```json
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Enrutamiento determinista: gana la primera coincidencia (la más específica primero).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Anulación por peer opcional (ejemplo: enviar un grupo específico al agente de trabajo).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Desactivado por defecto: la mensajería entre agentes debe habilitarse explícitamente + estar en lista de permitidos.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Anulación opcional. Predeterminado: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Anulación opcional. Predeterminado: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

## Ejemplo: Chat diario de WhatsApp + Trabajo profundo en Telegram

Divide por canal: enruta WhatsApp a un agente rápido para el día a día y Telegram a un agente Opus.

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

Notas:

-   Si tienes múltiples cuentas para un canal, añade `accountId` a la vinculación (por ejemplo `{ channel: "whatsapp", accountId: "personal" }`).
-   Para enrutar un solo MD/grupo a Opus mientras mantienes el resto en chat, añade una vinculación `match.peer` para ese peer; las coincidencias de peer siempre ganan sobre las reglas a nivel de canal.

## Ejemplo: mismo canal, un peer a Opus

Mantén WhatsApp en el agente rápido, pero enruta un MD a Opus:

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    {
      agentId: "opus",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551234567" } },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
  ],
}
```

Las vinculaciones de peer siempre ganan, así que mantenlas por encima de la regla a nivel de canal.

## Agente familiar vinculado a un grupo de WhatsApp

Vincula un agente familiar dedicado a un solo grupo de WhatsApp, con control por mención y una política de herramientas más estricta:

```json
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"],
        },
        sandbox: {
          mode: "all",
          scope: "agent",
        },
        tools: {
          allow: [
            "exec",
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"],
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" },
      },
    },
  ],
}
```

Notas:

-   Las listas de permitir/denegar herramientas son **herramientas**, no habilidades. Si una habilidad necesita ejecutar un binario, asegúrate de que `exec` esté permitido y el binario exista en el sandbox.
-   Para un control más estricto, establece `agents.list[].groupChat.mentionPatterns` y mantén las listas de permitidos de grupos habilitadas para el canal.

## Configuración de Sandbox y Herramientas por Agente

A partir de v2026.1.6, cada agente puede tener sus propias restricciones de sandbox y herramientas:

```json
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // Sin sandbox para el agente personal
        },
        // Sin restricciones de herramientas - todas las herramientas disponibles
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Siempre en sandbox
          scope: "agent",  // Un contenedor por agente
          docker: {
            // Configuración única opcional después de la creación del contenedor
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Solo herramienta read
          deny: ["exec", "write", "edit", "apply_patch"],    // Denegar otras
        },
      },
    ],
  },
}
```

Nota: `setupCommand` reside bajo `sandbox.docker` y se ejecuta una vez en la creación del contenedor. Las anulaciones `sandbox.docker.*` por agente se ignoran cuando el alcance resuelto es `"shared"`. **Beneficios:**

-   **Aislamiento de seguridad**: Restringe herramientas para agentes no confiables
-   **Control de recursos**: Sandbox de agentes específicos mientras mantienes otros en el host
-   **Políticas flexibles**: Diferentes permisos por agente

Nota: `tools.elevated` es **global** y basado en el remitente; no es configurable por agente. Si necesitas límites por agente, usa `agents.list[].tools` para denegar `exec`. Para dirigirse a grupos, usa `agents.list[].groupChat.mentionPatterns` para que las @menciones se asignen claramente al agente deseado. Consulta [Sandbox y Herramientas Multi-Agente](../tools/multi-agent-sandbox-tools.md) para ejemplos detallados.

[Compaction](./compaction.md)[Presence](./presence.md)