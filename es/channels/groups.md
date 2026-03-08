

  Configuración

  
# Grupos

OpenClaw trata los chats grupales de manera consistente en todas las superficies: WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Microsoft Teams, Zalo.

## Introducción para principiantes (2 minutos)

OpenClaw "vive" en tus propias cuentas de mensajería. No hay un usuario bot de WhatsApp separado. Si **tú** estás en un grupo, OpenClaw puede ver ese grupo y responder allí. Comportamiento por defecto:

-   Los grupos están restringidos (`groupPolicy: "allowlist"`).
-   Las respuestas requieren una mención a menos que desactives explícitamente el control por mención.

Traducción: los remitentes en la lista de permitidos pueden activar a OpenClaw mencionándolo.

> TL;DR
>
> -   El **acceso a DM** se controla con `*.allowFrom`.
> -   El **acceso a grupos** se controla con `*.groupPolicy` + listas de permitidos (`*.groups`, `*.groupAllowFrom`).
> -   El **disparo de respuestas** se controla con el control por mención (`requireMention`, `/activation`).

Flujo rápido (qué pasa con un mensaje de grupo):

```
groupPolicy? disabled -> descartar
groupPolicy? allowlist -> ¿grupo permitido? no -> descartar
requireMention? yes -> ¿mencionado? no -> almacenar solo para contexto
otherwise -> responder
```

![Flujo de mensajes de grupo](../images/channels-groups-flow.svg.md) Si quieres…

| Objetivo | Qué configurar |
| --- | --- |
| Permitir todos los grupos pero solo responder a @menciones | `groups: { "*": { requireMention: true } }` |
| Desactivar todas las respuestas en grupos | `groupPolicy: "disabled"` |
| Solo grupos específicos | `groups: { "<group-id>": { ... } }` (sin clave `"*"`) |
| Solo tú puedes activar en grupos | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

## Claves de sesión

-   Las sesiones de grupo usan claves de sesión `agent:::group:` (las salas/canales usan `agent:::channel:`).
-   Los temas de foro de Telegram añaden `:topic:` al id del grupo, por lo que cada tema tiene su propia sesión.
-   Los chats directos usan la sesión principal (o por remitente si está configurado).
-   Los latidos se omiten para las sesiones de grupo.

## Patrón: DMs personales + grupos públicos (agente único)

Sí — esto funciona bien si tu tráfico "personal" son **DMs** y tu tráfico "público" son **grupos**. Por qué: en modo de agente único, los DMs normalmente van a la clave de sesión **principal** (`agent:main:main`), mientras que los grupos siempre usan claves de sesión **no principales** (`agent:main::group:`). Si activas el sandboxing con `mode: "non-main"`, esas sesiones de grupo se ejecutan en Docker mientras tu sesión principal de DM permanece en el host. Esto te da un "cerebro" de agente (espacio de trabajo + memoria compartidos), pero dos posturas de ejecución:

-   **DMs**: herramientas completas (host)
-   **Grupos**: sandbox + herramientas restringidas (Docker)

> Si necesitas espacios de trabajo/personas verdaderamente separados ("personal" y "público" nunca deben mezclarse), usa un segundo agente + enlaces. Consulta [Enrutamiento Multi-Agente](../concepts/multi-agent.md).

Ejemplo (DMs en host, grupos en sandbox + solo herramientas de mensajería):

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // grupos/canales son no principales -> en sandbox
        scope: "session", // aislamiento más fuerte (un contenedor por grupo/canal)
        workspaceAccess: "none",
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        // Si allow no está vacío, todo lo demás se bloquea (deny aún gana).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"],
      },
    },
  },
}
```

¿Quieres "los grupos solo pueden ver la carpeta X" en lugar de "sin acceso al host"? Mantén `workspaceAccess: "none"` y monta solo las rutas permitidas en el sandbox:

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // hostPath:containerPath:mode
            "/home/user/FriendsShared:/data:ro",
          ],
        },
      },
    },
  },
}
```

Relacionado:

-   Claves de configuración y valores por defecto: [Configuración de Gateway](../gateway/configuration.md#agentsdefaultssandbox)
-   Depuración de por qué una herramienta está bloqueada: [Sandbox vs Política de Herramientas vs Elevado](../gateway/sandbox-vs-tool-policy-vs-elevated.md)
-   Detalles de montajes bind: [Sandboxing](../gateway/sandboxing.md#custom-bind-mounts)

## Etiquetas de visualización

-   Las etiquetas de la UI usan `displayName` cuando está disponible, formateado como `:`.
-   `#room` está reservado para salas/canales; los chats grupales usan `g-` (minúsculas, espacios -> `-`, conservar `#@+._-`).

## Política de grupo

Controla cómo se manejan los mensajes de grupo/sala por canal:

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789"], // id de usuario numérico de Telegram (el asistente puede resolver @username)
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: { channels: { help: { allow: true } } },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
    },
  },
}
```

| Política | Comportamiento |
| --- | --- |
| `"open"` | Los grupos omiten las listas de permitidos; aún se aplica el control por mención. |
| `"disabled"` | Bloquea todos los mensajes de grupo por completo. |
| `"allowlist"` | Solo permite grupos/salas que coincidan con la lista de permitidos configurada. |

Notas:

-   `groupPolicy` es independiente del control por mención (que requiere @menciones).
-   WhatsApp/Telegram/Signal/iMessage/Microsoft Teams/Zalo: usa `groupAllowFrom` (alternativa: `allowFrom` explícito).
-   Las aprobaciones de emparejamiento de DM (entradas de almacenamiento `*-allowFrom`) se aplican solo al acceso a DM; la autorización del remitente en grupos permanece explícita en las listas de permitidos de grupo.
-   Discord: la lista de permitidos usa `channels.discord.guilds..channels`.
-   Slack: la lista de permitidos usa `channels.slack.channels`.
-   Matrix: la lista de permitidos usa `channels.matrix.groups` (IDs de sala, alias o nombres). Usa `channels.matrix.groupAllowFrom` para restringir remitentes; también se admiten listas de permitidos `users` por sala.
-   Los DMs grupales se controlan por separado (`channels.discord.dm.*`, `channels.slack.dm.*`).
-   La lista de permitidos de Telegram puede coincidir con IDs de usuario (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) o nombres de usuario (`"@alice"` o `"alice"`); los prefijos no distinguen mayúsculas y minúsculas.
-   El valor por defecto es `groupPolicy: "allowlist"`; si tu lista de permitidos de grupo está vacía, los mensajes de grupo se bloquean.
-   Seguridad en tiempo de ejecución: cuando falta completamente un bloque de proveedor (`channels.` ausente), la política de grupo vuelve a un modo de fallo cerrado (normalmente `allowlist`) en lugar de heredar `channels.defaults.groupPolicy`.

Modelo mental rápido (orden de evaluación para mensajes de grupo):

1.  `groupPolicy` (open/disabled/allowlist)
2.  Listas de permitidos de grupo (`*.groups`, `*.groupAllowFrom`, lista de permitidos específica del canal)
3.  Control por mención (`requireMention`, `/activation`)

## Control por mención (por defecto)

Los mensajes de grupo requieren una mención a menos que se anule por grupo. Los valores por defecto viven por subsistema bajo `*.groups."*"`. Responder a un mensaje del bot cuenta como una mención implícita (cuando el canal admite metadatos de respuesta). Esto se aplica a Telegram, WhatsApp, Slack, Discord y Microsoft Teams.

```json
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false },
      },
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false },
      },
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50,
        },
      },
    ],
  },
}
```

Notas:

-   `mentionPatterns` son expresiones regulares que no distinguen mayúsculas y minúsculas.
-   Las superficies que proporcionan menciones explícitas aún pasan; los patrones son un respaldo.
-   Anulación por agente: `agents.list[].groupChat.mentionPatterns` (útil cuando múltiples agentes comparten un grupo).
-   El control por mención solo se aplica cuando la detección de mención es posible (menciones nativas o `mentionPatterns` están configuradas).
-   Los valores por defecto de Discord viven en `channels.discord.guilds."*"` (anulable por gremio/canal).
-   El contexto del historial de grupo se envuelve uniformemente en todos los canales y es **solo pendiente** (mensajes omitidos debido al control por mención); usa `messages.groupChat.historyLimit` para el valor por defecto global y `channels..historyLimit` (o `channels..accounts.*.historyLimit`) para anulaciones. Establece `0` para desactivar.

## Restricciones de herramientas por grupo/canal (opcional)

Algunas configuraciones de canal admiten restringir qué herramientas están disponibles **dentro de un grupo/sala/canal específico**.

-   `tools`: permitir/denegar herramientas para todo el grupo.
-   `toolsBySender`: anulaciones por remitente dentro del grupo. Usa prefijos de clave explícitos: `id:`, `e164:`, `username:`, `name:`, y el comodín `"*"`. Las claves sin prefijo heredadas aún se aceptan y coinciden solo como `id:`.

Orden de resolución (gana el más específico):

1.  Coincidencia `toolsBySender` de grupo/canal
2.  `tools` de grupo/canal
3.  Coincidencia `toolsBySender` por defecto (`"*"`)
4.  `tools` por defecto (`"*"`)

Ejemplo (Telegram):

```json
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "id:123456789": { alsoAllow: ["exec"] },
          },
        },
      },
    },
  },
}
```

Notas:

-   Las restricciones de herramientas por grupo/canal se aplican además de la política de herramientas global/del agente (deny aún gana).
-   Algunos canales usan anidamiento diferente para salas/canales (p.ej., Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`).

## Listas de permitidos de grupo

Cuando se configuran `channels.whatsapp.groups`, `channels.telegram.groups` o `channels.imessage.groups`, las claves actúan como una lista de permitidos de grupo. Usa `"*"` para permitir todos los grupos mientras aún estableces el comportamiento de mención por defecto. Intenciones comunes (copiar/pegar):

1.  Desactivar todas las respuestas en grupos

```json
{
  channels: { whatsapp: { groupPolicy: "disabled" } },
}
```

2.  Permitir solo grupos específicos (WhatsApp)

```json
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false },
      },
    },
  },
}
```

3.  Permitir todos los grupos pero requerir mención (explícito)

```json
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

4.  Solo el propietario puede activar en grupos (WhatsApp)

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## Activación (solo propietario)

Los propietarios de grupos pueden alternar la activación por grupo:

-   `/activation mention`
-   `/activation always`

El propietario se determina por `channels.whatsapp.allowFrom` (o el E.164 propio del bot cuando no está establecido). Envía el comando como un mensaje independiente. Otras superficies actualmente ignoran `/activation`.

## Campos de contexto

Las cargas útiles entrantes de grupo establecen:

-   `ChatType=group`
-   `GroupSubject` (si se conoce)
-   `GroupMembers` (si se conoce)
-   `WasMentioned` (resultado del control por mención)
-   Los temas de foro de Telegram también incluyen `MessageThreadId` e `IsForum`.

El mensaje del sistema del agente incluye una introducción de grupo en el primer turno de una nueva sesión de grupo. Recuerda al modelo que responda como un humano, evite tablas Markdown y evite escribir secuencias literales `\n`.

## Específicos de iMessage

-   Prefiere `chat_id:` al enrutar o en listas de permitidos.
-   Listar chats: `imsg chats --limit 20`.
-   Las respuestas de grupo siempre vuelven al mismo `chat_id`.

## Específicos de WhatsApp

Consulta [Mensajes de grupo](./group-messages.md) para el comportamiento exclusivo de WhatsApp (inyección de historial, detalles del manejo de menciones).

[Mensajes de Grupo](./group-messages.md)[Grupos de Difusión](./broadcast-groups.md)

---