

  Configuración y operaciones

  
# Referencia de Configuración

Cada campo disponible en `~/.openclaw/openclaw.json`. Para una visión orientada a tareas, consulta [Configuración](./configuration.md). El formato de configuración es **JSON5** (se permiten comentarios y comas finales). Todos los campos son opcionales — OpenClaw usa valores predeterminados seguros cuando se omiten.

* * *

## Canales

Cada canal se inicia automáticamente cuando existe su sección de configuración (a menos que `enabled: false`).

### Acceso a DM y grupos

Todos los canales admiten políticas de DM y políticas de grupo:

| Política de DM | Comportamiento |
| --- | --- |
| `pairing` (predeterminado) | Los remitentes desconocidos reciben un código de emparejamiento de un solo uso; el propietario debe aprobar |
| `allowlist` | Solo remitentes en `allowFrom` (o almacén de permitidos emparejado) |
| `open` | Permitir todos los DM entrantes (requiere `allowFrom: ["*"]`) |
| `disabled` | Ignorar todos los DM entrantes |

| Política de grupo | Comportamiento |
| --- | --- |
| `allowlist` (predeterminado) | Solo grupos que coincidan con la lista de permitidos configurada |
| `open` | Omitir las listas de permitidos de grupos (la mención aún aplica) |
| `disabled` | Bloquear todos los mensajes de grupo/sala |

> **ℹ️** `channels.defaults.groupPolicy` establece el valor predeterminado cuando `groupPolicy` de un proveedor no está configurado. Los códigos de emparejamiento expiran después de 1 hora. Las solicitudes de emparejamiento de DM pendientes están limitadas a **3 por canal**. Si falta por completo un bloque de proveedor (`channels.` ausente), la política de grupo en tiempo de ejecución vuelve a `allowlist` (fallo cerrado) con una advertencia de inicio.

### Anulaciones de modelo por canal

Usa `channels.modelByChannel` para asignar IDs de canal específicos a un modelo. Los valores aceptan `provider/model` o alias de modelo configurados. La asignación de canal se aplica cuando una sesión no tiene ya una anulación de modelo (por ejemplo, establecida mediante `/model`).

```json
{
  channels: {
    modelByChannel: {
      discord: {
        "123456789012345678": "anthropic/claude-opus-4-6",
      },
      slack: {
        C1234567890: "openai/gpt-4.1",
      },
      telegram: {
        "-1001234567890": "openai/gpt-4.1-mini",
        "-1001234567890:topic:99": "anthropic/claude-sonnet-4-6",
      },
    },
  },
}
```

### Valores predeterminados de canal y latido

Usa `channels.defaults` para compartir comportamiento de política de grupo y latido entre proveedores:

```json
{
  channels: {
    defaults: {
      groupPolicy: "allowlist", // open | allowlist | disabled
      heartbeat: {
        showOk: false,
        showAlerts: true,
        useIndicator: true,
      },
    },
  },
}
```

-   `channels.defaults.groupPolicy`: política de grupo de respaldo cuando `groupPolicy` a nivel de proveedor no está configurado.
-   `channels.defaults.heartbeat.showOk`: incluir estados de canal saludables en la salida del latido.
-   `channels.defaults.heartbeat.showAlerts`: incluir estados degradados/de error en la salida del latido.
-   `channels.defaults.heartbeat.useIndicator`: renderizar salida de latido compacta estilo indicador.

### WhatsApp

WhatsApp se ejecuta a través del canal web del gateway (Baileys Web). Se inicia automáticamente cuando existe una sesión vinculada.

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // ticks azules (false en modo self-chat)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

```json
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

-   Los comandos salientes usan por defecto la cuenta `default` si está presente; de lo contrario, la primera cuenta configurada (ordenada).
-   `channels.whatsapp.defaultAccount` opcional anula esa selección de cuenta predeterminada de respaldo cuando coincide con un ID de cuenta configurado.
-   El directorio de autenticación de cuenta única heredado de Baileys es migrado por `openclaw doctor` a `whatsapp/default`.
-   Anulaciones por cuenta: `channels.whatsapp.accounts..sendReadReceipts`, `channels.whatsapp.accounts..dmPolicy`, `channels.whatsapp.accounts..allowFrom`.

### Telegram

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Mantén las respuestas breves.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Mantente en el tema.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Respaldo Git" },
        { command: "generate", description: "Crear una imagen" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streaming: "partial", // off | partial | block | progress (predeterminado: off)
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 100,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        autoSelectFamily: true,
        dnsResultOrder: "ipv4first",
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

-   Token del bot: `channels.telegram.botToken` o `channels.telegram.tokenFile`, con `TELEGRAM_BOT_TOKEN` como respaldo para la cuenta predeterminada.
-   `channels.telegram.defaultAccount` opcional anula la selección de cuenta predeterminada cuando coincide con un ID de cuenta configurado.
-   En configuraciones multi-cuenta (2+ IDs de cuenta), establece un predeterminado explícito (`channels.telegram.defaultAccount` o `channels.telegram.accounts.default`) para evitar el enrutamiento de respaldo; `openclaw doctor` advierte cuando esto falta o es inválido.
-   `configWrites: false` bloquea escrituras de configuración iniciadas por Telegram (migraciones de ID de supergrupo, `/config set|unset`).
-   Las entradas de nivel superior `bindings[]` con `type: "acp"` configuran enlaces ACP persistentes para temas de foro (usa `chatId:topic:topicId` canónico en `match.peer.id`). La semántica de los campos se comparte en [Agentes ACP](../tools/acp-agents.md#channel-specific-settings).
-   Las vistas previas de transmisión de Telegram usan `sendMessage` + `editMessageText` (funciona en chats directos y de grupo).
-   Política de reintento: consulta [Política de reintento](../concepts/retry.md).

### Discord

```json
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "123456789012345678"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          ignoreOtherMentions: true,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Solo respuestas cortas.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      streaming: "off", // off | partial | block | progress (progress se asigna a partial en Discord)
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // opt-in para sessions_spawn({ thread: true })
      },
      voice: {
        enabled: true,
        autoJoin: [
          {
            guildId: "123456789012345678",
            channelId: "234567890123456789",
          },
        ],
        daveEncryption: true,
        decryptionFailureTolerance: 24,
        tts: {
          provider: "openai",
          openai: { voice: "alloy" },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

-   Token: `channels.discord.token`, con `DISCORD_BOT_TOKEN` como respaldo para la cuenta predeterminada.
-   `channels.discord.defaultAccount` opcional anula la selección de cuenta predeterminada cuando coincide con un ID de cuenta configurado.
-   Usa `user:` (DM) o `channel:` (canal de gremio) para destinos de entrega; los IDs numéricos simples son rechazados.
-   Los slugs de gremio están en minúsculas con espacios reemplazados por `-`; las claves de canal usan el nombre con slug (sin `#`). Prefiere IDs de gremio.
-   Los mensajes escritos por el bot son ignorados por defecto. `allowBots: true` los habilita; usa `allowBots: "mentions"` para solo aceptar mensajes de bot que mencionen al bot (los mensajes propios aún se filtran).
-   `channels.discord.guilds..ignoreOtherMentions` (y anulaciones de canal) descarta mensajes que mencionan a otro usuario o rol pero no al bot (excluyendo @everyone/@here).
-   `maxLinesPerMessage` (predeterminado 17) divide mensajes altos incluso cuando están por debajo de 2000 caracteres.
-   `channels.discord.threadBindings` controla el enrutamiento vinculado a hilos de Discord:
    -   `enabled`: anulación de Discord para características de sesión vinculadas a hilos (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`, y entrega/enrutamiento vinculado)
    -   `idleHours`: anulación de Discord para auto-desenfoque por inactividad en horas (`0` deshabilita)
    -   `maxAgeHours`: anulación de Discord para edad máxima dura en horas (`0` deshabilita)
    -   `spawnSubagentSessions`: interruptor de opt-in para `sessions_spawn({ thread: true })` creación/vinculación automática de hilos
-   Las entradas de nivel superior `bindings[]` con `type: "acp"` configuran enlaces ACP persistentes para canales e hilos (usa ID de canal/hilo en `match.peer.id`). La semántica de los campos se comparte en [Agentes ACP](../tools/acp-agents.md#channel-specific-settings).
-   `channels.discord.ui.components.accentColor` establece el color de acento para contenedores de componentes v2 de Discord.
-   `channels.discord.voice` habilita conversaciones en canales de voz de Discord y uniones automáticas opcionales + anulaciones de TTS.
-   `channels.discord.voice.daveEncryption` y `channels.discord.voice.decryptionFailureTolerance` se pasan a las opciones DAVE de `@discordjs/voice` (`true` y `24` por defecto).
-   OpenClaw además intenta recuperación de recepción de voz dejando/reuniéndose a una sesión de voz después de fallos de descifrado repetidos.
-   `channels.discord.streaming` es la clave de modo de transmisión canónica. Los valores heredados `streamMode` y booleanos `streaming` son migrados automáticamente.
-   `channels.discord.autoPresence` asigna la disponibilidad en tiempo de ejecución a la presencia del bot (saludable => online, degradado => idle, agotado => dnd) y permite anulaciones de texto de estado opcionales.
-   `channels.discord.dangerouslyAllowNameMatching` rehabilita la coincidencia de nombre/etiqueta mutable (modo de compatibilidad de último recurso).

**Modos de notificación de reacciones:** `off` (ninguna), `own` (mensajes del bot, predeterminado), `all` (todos los mensajes), `allowlist` (de `guilds..users` en todos los mensajes).

### Google Chat

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

-   JSON de cuenta de servicio: en línea (`serviceAccount`) o basado en archivo (`serviceAccountFile`).
-   También se admite SecretRef de cuenta de servicio (`serviceAccountRef`).
-   Respaldo de entorno: `GOOGLE_CHAT_SERVICE_ACCOUNT` o `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
-   Usa `spaces/` o `users/` para destinos de entrega.
-   `channels.googlechat.dangerouslyAllowNameMatching` rehabilita la coincidencia de principal de correo mutable (modo de compatibilidad de último recurso).

### Slack

```json
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Solo respuestas cortas.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      typingReaction: "hourglass_flowing_sand",
      textChunkLimit: 4000,
      chunkMode: "length",
      streaming: "partial", // off | partial | block | progress (modo de vista previa)
      nativeStreaming: true, // usar API de transmisión nativa de Slack cuando streaming=partial
      mediaMaxMb: 20,
    },
  },
}
```

-   **Modo socket** requiere tanto `botToken` como `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` para respaldo de entorno de cuenta predeterminada).
-   **Modo HTTP** requiere `botToken` más `signingSecret` (en raíz o por cuenta).
-   `configWrites: false` bloquea escrituras de configuración iniciadas por Slack.
-   `channels.slack.defaultAccount` opcional anula la selección de cuenta predeterminada cuando coincide con un ID de cuenta configurado.
-   `channels.slack.streaming` es la clave de modo de transmisión canónica. Los valores heredados `streamMode` y booleanos `streaming` son migrados automáticamente.
-   Usa `user:` (DM) o `channel:` para destinos de entrega.

**Modos de notificación de reacciones:** `off`, `own` (predeterminado), `all`, `allowlist` (de `reactionAllowlist`). **Aislamiento de sesión de hilo:** `thread.historyScope` es por hilo (predeterminado) o compartido a través del canal. `thread.inheritParent` copia la transcripción del canal padre a nuevos hilos.

-   `typingReaction` agrega una reacción temporal al mensaje entrante de Slack mientras se ejecuta una respuesta, luego la elimina al completarse. Usa un código corto de emoji de Slack como `"hourglass_flowing_sand"`.

| Grupo de acción | Predeterminado | Notas |
| --- | --- | --- |
| reactions | habilitado | Reaccionar + listar reacciones |
| messages | habilitado | Leer/enviar/editar/eliminar |
| pins | habilitado | Fijar/desfijar/listar |
| memberInfo | habilitado | Información de miembro |
| emojiList | habilitado | Lista de emojis personalizados |

### Mattermost

Mattermost se distribuye como un complemento: `openclaw plugins install @openclaw/mattermost`.

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      commands: {
        native: true, // opt-in
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // URL explícita opcional para despliegues con proxy inverso/públicos
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

Modos de chat: `oncall` (responder en mención @, predeterminado), `onmessage` (cada mensaje), `onchar` (mensajes que comienzan con prefijo desencadenante). Cuando los comandos nativos de Mattermost están habilitados:

-   `commands.callbackPath` debe ser una ruta (por ejemplo `/api/channels/mattermost/command`), no una URL completa.
-   `commands.callbackUrl` debe resolverse al endpoint del gateway de OpenClaw y ser accesible desde el servidor de Mattermost.
-   Para hosts de callback privados/tailnet/internos, Mattermost puede requerir que `ServiceSettings.AllowedUntrustedInternalConnections` incluya el host/dominio del callback. Usa valores de host/dominio, no URLs completas.
-   `channels.mattermost.configWrites`: permitir o denegar escrituras de configuración iniciadas por Mattermost.
-   `channels.mattermost.requireMention`: requerir `@mention` antes de responder en canales.
-   `channels.mattermost.defaultAccount` opcional anula la selección de cuenta predeterminada cuando coincide con un ID de cuenta configurado.

### Signal

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15555550123", // vinculación de cuenta opcional
      dmPolicy: "pairing",
      allowFrom: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      configWrites: true,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**Modos de notificación de reacciones:** `off`, `own` (predeterminado), `all`, `allowlist` (de `reactionAllowlist`).

-   `channels.signal.account`: vincula el inicio del canal a una identidad de cuenta de Signal específica.
-   `channels.signal.configWrites`: permitir o denegar escrituras de configuración iniciadas por Signal.
-   `channels.signal.defaultAccount` opcional anula la selección de cuenta predeterminada cuando coincide con un ID de cuenta configurado.

### BlueBubbles

BlueBubbles es la ruta recomendada para iMessage (respaldado por complemento, configurado bajo `channels.bluebubbles`).

```json
{
  channels: {
    bluebubbles: {
      enabled: true,
      dmPolicy: "pairing",
      // serverUrl, password, webhookPath, controles de grupo y acciones avanzadas:
      // ver /channels/bluebubbles
    },
  },
}
```

-   Rutas clave principales cubiertas aquí: `channels.bluebubbles`, `channels.bluebubbles.dmPolicy`.
-   `channels.bluebubbles.defaultAccount` opcional anula la selección de cuenta predeterminada cuando coincide con un ID de cuenta configurado.
-   La configuración completa del canal BlueBubbles está documentada en [BlueBubbles](../channels/bluebubbles.md).

### iMessage

OpenClaw ejecuta `imsg rpc` (JSON-RPC sobre stdio). No se requiere daemon ni puerto.

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      attachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      remoteAttachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

-   `channels.imessage.defaultAccount` opcional anula la selección de cuenta predeterminada cuando coincide con un ID de cuenta configurado.
-   Requiere Acceso Total al Disco a la base de datos de Mensajes.
-   Prefiere destinos `chat_id:`. Usa `imsg chats --limit 20` para listar chats.
-   `cliPath` puede apuntar a un wrapper SSH; establece `remoteHost` (`host` o `user@host`) para obtención de adjuntos SCP.
-   `attachmentRoots` y `remoteAttachmentRoots` restringen rutas de adjuntos entrantes (predeterminado: `/Users/*/Library/Messages/Attachments`).
-   SCP usa verificación estricta de clave de host, así que asegúrate de que la clave del host de retransmisión ya exista en `~/.ssh/known_hosts`.
-   `channels.imessage.configWrites`: permitir o denegar escrituras de configuración iniciadas por iMessage.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### Microsoft Teams

Microsoft Teams está respaldado por extensión y configurado bajo `channels.msteams`.

```json
{
  channels: {
    msteams: {
      enabled: true,
      configWrites: true,
      // appId, appPassword, tenantId, webhook, políticas de equipo/canal:
      // ver /channels/msteams
    },
  },
}
```

-   Rutas clave principales cubiertas aquí: `channels.msteams`, `channels.msteams.configWrites`.
-   La configuración completa de Teams (credenciales, webhook, política de DM/grupo, anulaciones por equipo/por canal) está documentada en [Microsoft Teams](../channels/msteams.md).

### IRC

IRC está respaldado por extensión y configurado bajo `channels.irc`.

```json
{
  channels: {
    irc: {
      enabled: true,
      dmPolicy: "pairing",
      configWrites: true,
      nickserv: {
        enabled: true,
        service: "NickServ",
        password: "${IRC_NICKSERV_PASSWORD}",
        register: false,
        registerEmail: "bot@example.com",
      },
    },
  },
}
```

-   Rutas clave principales cubiertas aquí: `channels.irc`, `channels.irc.dmPolicy`, `channels.irc.configWrites`, `channels.irc.nickserv.*`.
-   `channels.irc.defaultAccount` opcional anula la selección de cuenta predeterminada cuando coincide con un ID de cuenta configurado.
-   La configuración completa del canal IRC (host/puerto/TLS/canales/listas de permitidos/control de menciones) está documentada en [IRC](../channels/irc.md).

### Multi-cuenta (todos los canales)

Ejecuta múltiples cuentas por canal (cada una con su propio `accountId`):

```json
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Bot principal",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Bot de alertas",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

-   `default` se usa cuando se omite `accountId` (CLI + enrutamiento).
-   Los tokens de entorno solo aplican a la cuenta **predeterminada**.
-   Las configuraciones base del canal aplican a todas las cuentas a menos que se anulen por cuenta.
-   Usa `bindings[].match.accountId` para enrutar cada cuenta a un agente diferente.
-   Si agregas una cuenta no predeterminada mediante `openclaw channels add` (o incorporación de canal) mientras aún estás en una configuración de canal de nivel superior de cuenta única, OpenClaw mueve los valores de nivel superior de cuenta única con alcance de cuenta a `channels..accounts.default` primero para que la cuenta original siga funcionando.
-   Los enlaces existentes solo de canal (sin `accountId`) siguen coincidiendo con la cuenta predeterminada; los enlaces con alcance de cuenta siguen siendo opcionales.
-   `openclaw doctor --fix` también repara formas mixtas moviendo valores de nivel superior de cuenta única con alcance de cuenta a `accounts.default` cuando existen cuentas nombradas pero falta `default`.

### Otros canales de extensión

Muchos canales de extensión se configuran como `channels.` y están documentados en sus páginas de canal dedicadas (por ejemplo Feishu, Matrix, LINE, Nostr, Zalo, Nextcloud Talk, Synology Chat y Twitch). Consulta el índice completo de canales: [Canales](../channels.md).

### Control de mención en chats de grupo

Los mensajes de grupo por defecto **requieren mención** (mención de metadatos o patrones regex). Aplica a chats de grupo de WhatsApp, Telegram, Discord, Google Chat e iMessage. **Tipos de mención:**

-   **Menciones de metadatos**: Menciones @ nativas de la plataforma. Ignoradas en modo self-chat de WhatsApp.
-   **Patrones de texto**: Patrones regex en `agents.list[].groupChat.mentionPatterns`. Siempre verificados.
-   El control de mención se aplica solo cuando la detección es posible (menciones nativas o al menos un patrón).

```json
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` establece el valor predeterminado global. Los canales pueden anular con `channels..historyLimit` (o por cuenta). Establece `0` para deshabilitar.

#### Límites de historial de DM

```json
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

Resolución: anulación por DM → predeterminado del proveedor → sin límite (todos retenidos). Compatible: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Modo self-chat

Incluye tu propio número en `allowFrom` para habilitar el modo self-chat (ignora menciones @ nativas, solo responde a patrones de texto):

```json
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### Comandos (manejo de comandos de chat)

```json
{
  commands: {
    native: "auto", // registrar comandos nativos cuando sean compatibles
    text: true, // analizar /commands en mensajes de chat
    bash: false, // permitir ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // permitir /config
    debug: false, // permitir /debug
    restart: false, // permitir /restart + herramienta de reinicio del gateway
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

-   Los comandos de texto deben ser mensajes **independientes** con `/` al inicio.
-   `native: "auto"` activa comandos nativos para Discord/Telegram, deja Slack apagado.
-   Anular por canal: `channels.discord.commands.native` (bool o `"auto"`). `false` borra comandos registrados previamente.
-   `channels.telegram.customCommands` agrega entradas extra al menú del bot de Telegram.
-   `bash: true` habilita `! ` para shell del host. Requiere `tools.elevated.enabled` y remit