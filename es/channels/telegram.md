title: "Configuración y configuración del bot de Telegram para OpenClaw AI"
description: "Aprende a configurar y configurar un bot de Telegram para OpenClaw AI. Incluye configuración de token, políticas de grupo, comportamiento en tiempo de ejecución y funciones avanzadas como transmisión y comandos."
keywords: ["bot de telegram", "openclaw", "configuración de bot", "grammy", "plataforma de mensajería", "configuración de canal", "política de grupo", "token de bot"]
---

  Plataformas de mensajería

  
# Telegram

Estado: listo para producción para mensajes directos de bot + grupos a través de grammY. El modo de long polling es el predeterminado; el modo webhook es opcional.

## Configuración rápida

### Paso 1: Crear el token del bot en BotFather

Abre Telegram y chatea con **@BotFather** (confirma que el identificador sea exactamente `@BotFather`). Ejecuta `/newbot`, sigue las indicaciones y guarda el token.

### Paso 2: Configurar el token y la política de mensajes directos

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

Respaldo de variable de entorno: `TELEGRAM_BOT_TOKEN=...` (solo cuenta predeterminada). Telegram **no** usa `openclaw channels login telegram`; configura el token en config/env, luego inicia la puerta de enlace.

### Paso 3: Iniciar la puerta de enlace y aprobar el primer mensaje directo

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Los códigos de emparejamiento expiran después de 1 hora.

### Paso 4: Agregar el bot a un grupo

Agrega el bot a tu grupo, luego configura `channels.telegram.groups` y `groupPolicy` para que coincidan con tu modelo de acceso.

 

> **ℹ️** El orden de resolución del token tiene en cuenta la cuenta. En la práctica, los valores de configuración tienen prioridad sobre el respaldo de la variable de entorno, y `TELEGRAM_BOT_TOKEN` solo se aplica a la cuenta predeterminada.

## Configuraciones del lado de Telegram

Los bots de Telegram tienen **Modo de Privacidad** predeterminado, lo que limita los mensajes de grupo que reciben. Si el bot debe ver todos los mensajes del grupo, puedes:

-   desactivar el modo de privacidad mediante `/setprivacy`, o
-   hacer que el bot sea administrador del grupo.

Al cambiar el modo de privacidad, elimina y vuelve a agregar el bot en cada grupo para que Telegram aplique el cambio.

El estado de administrador se controla en la configuración del grupo de Telegram. Los bots administradores reciben todos los mensajes del grupo, lo que es útil para un comportamiento de grupo siempre activo.

-   `/setjoingroups` para permitir/denegar agregar a grupos
-   `/setprivacy` para el comportamiento de visibilidad en grupos

## Control de acceso y activación

/getUpdates"', lang: 'bash' }, { label: 'Política de grupo y listas de permitidos', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "-1001234567890": {\n          groupPolicy: "open",\n          requireMention: false,\n        },\n      },\n    },\n  },\n}', lang: 'json' }, { label: 'Comportamiento de mención', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "*": { requireMention: false },\n      },\n    },\n  },\n}', lang: 'json' }]} />

## Comportamiento en tiempo de ejecución

-   Telegram es propiedad del proceso de la puerta de enlace.
-   El enrutamiento es determinista: las respuestas entrantes de Telegram vuelven a Telegram (el modelo no elige canales).
-   Los mensajes entrantes se normalizan en el sobre de canal compartido con metadatos de respuesta y marcadores de posición de medios.
-   Las sesiones de grupo están aisladas por ID de grupo. Los temas del foro agregan `:topic:` para mantener los temas aislados.
-   Los mensajes directos pueden llevar `message_thread_id`; OpenClaw los enruta con claves de sesión conscientes del hilo y preserva el ID del hilo para las respuestas.
-   El long polling usa grammY runner con secuenciación por chat/hilo. La concurrencia general del sumidero del runner usa `agents.defaults.maxConcurrent`.
-   La API de Bot de Telegram no tiene soporte para acuses de recibo (`sendReadReceipts` no se aplica).

## Referencia de funciones

OpenClaw puede transmitir respuestas parciales en tiempo real:

-   chats directos: transmisión de borradores nativos de Telegram a través de `sendMessageDraft`
-   grupos/temas: mensaje de vista previa + `editMessageText`

Requisito:

-   `channels.telegram.streaming` es `off | partial | block | progress` (predeterminado: `partial`)
-   `progress` se asigna a `partial` en Telegram (compatible con la nomenclatura entre canales)
-   los valores heredados `channels.telegram.streamMode` y booleanos `streaming` se asignan automáticamente

Telegram habilitó `sendMessageDraft` para todos los bots en Bot API 9.5 (1 de marzo de 2026). Para respuestas solo de texto:

-   MD: OpenClaw actualiza el borrador en su lugar (sin mensaje de vista previa adicional)
-   grupo/tema: OpenClaw mantiene el mismo mensaje de vista previa y realiza una edición final en su lugar (sin segundo mensaje)

Para respuestas complejas (por ejemplo, cargas de medios), OpenClaw recurre a la entrega final normal y luego limpia el mensaje de vista previa. La transmisión de vista previa es independiente de la transmisión por bloques. Cuando la transmisión por bloques está explícitamente habilitada para Telegram, OpenClaw omite la transmisión de vista previa para evitar una doble transmisión. Si el transporte de borrador nativo no está disponible/se rechaza, OpenClaw recurre automáticamente a `sendMessage` + `editMessageText`. Flujo de razonamiento solo para Telegram:

-   `/reasoning stream` envía el razonamiento a la vista previa en vivo mientras se genera
-   la respuesta final se envía sin texto de razonamiento

El texto saliente usa Telegram `parse_mode: "HTML"`.

-   El texto tipo Markdown se renderiza a HTML seguro para Telegram.
-   El HTML crudo del modelo se escapa para reducir fallos de análisis de Telegram.
-   Si Telegram rechaza el HTML analizado, OpenClaw lo reintenta como texto plano.

Las vistas previas de enlaces están habilitadas por defecto y se pueden desactivar con `channels.telegram.linkPreview: false`.

El registro del menú de comandos de Telegram se maneja al inicio con `setMyCommands`. Comandos nativos predeterminados:

-   `commands.native: "auto"` habilita los comandos nativos para Telegram

Agrega entradas personalizadas al menú de comandos:

```json
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Copia de seguridad Git" },
        { command: "generate", description: "Crear una imagen" },
      ],
    },
  },
}
```

Reglas:

-   los nombres se normalizan (eliminar `/` inicial, minúsculas)
-   patrón válido: `a-z`, `0-9`, `_`, longitud `1..32`
-   los comandos personalizados no pueden anular los comandos nativos
-   los conflictos/duplicados se omiten y se registran

Notas:

-   los comandos personalizados son solo entradas de menú; no implementan automáticamente el comportamiento
-   los comandos de complementos/habilidades aún pueden funcionar cuando se escriben, incluso si no se muestran en el menú de Telegram

Si los comandos nativos están deshabilitados, los integrados se eliminan. Los comandos personalizados/de complementos aún pueden registrarse si están configurados. Fallo de configuración común:

-   `setMyCommands failed` generalmente significa que el DNS/HTTPS saliente a `api.telegram.org` está bloqueado.

### Comandos de emparejamiento de dispositivos (complemento device-pair)

Cuando el complemento `device-pair` está instalado:

1.  `/pair` genera un código de configuración
2.  pega el código en la aplicación de iOS
3.  `/pair approve` aprueba la solicitud pendiente más reciente

Más detalles: [Emparejamiento](./pairing.md#pair-via-telegram-recommended-for-ios).

Configura el alcance del teclado en línea:

```json
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

Anulación por cuenta:

```json
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

Alcances:

-   `off`
-   `dm`
-   `group`
-   `all`
-   `allowlist` (predeterminado)

El heredado `capabilities: ["inlineButtons"]` se asigna a `inlineButtons: "all"`. Ejemplo de acción de mensaje:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Elige una opción:",
  buttons: [
    [
      { text: "Sí", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancelar", callback_data: "cancel" }],
  ],
}
```

Los clics de devolución de llamada se pasan al agente como texto: `callback_data: `

Las acciones de herramienta de Telegram incluyen:

-   `sendMessage` (`to`, `content`, opcional `mediaUrl`, `replyToMessageId`, `messageThreadId`)
-   `react` (`chatId`, `messageId`, `emoji`)
-   `deleteMessage` (`chatId`, `messageId`)
-   `editMessage` (`chatId`, `messageId`, `content`)
-   `createForumTopic` (`chatId`, `name`, opcional `iconColor`, `iconCustomEmojiId`)

Las acciones de mensaje del canal exponen alias ergonómicos (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`, `topic-create`). Controles de habilitación:

-   `channels.telegram.actions.sendMessage`
-   `channels.telegram.actions.deleteMessage`
-   `channels.telegram.actions.reactions`
-   `channels.telegram.actions.sticker` (predeterminado: deshabilitado)

Nota: `edit` y `topic-create` están actualmente habilitados por defecto y no tienen interruptores `channels.telegram.actions.*` separados. Semántica de eliminación de reacciones: [/tools/reactions](../tools/reactions.md)

Telegram admite etiquetas explícitas de respuesta en hilos en la salida generada:

-   `[[reply_to_current]]` responde al mensaje desencadenante
-   `[[reply_to:]]` responde a un ID de mensaje de Telegram específico

`channels.telegram.replyToMode` controla el manejo:

-   `off` (predeterminado)
-   `first`
-   `all`

Nota: `off` deshabilita el hilo de respuesta implícito. Las etiquetas explícitas `[[reply_to_*]]` aún se respetan.

Supergrupos de foro:

-   las claves de sesión del tema agregan `:topic:`
-   las respuestas y el indicador de escritura se dirigen al hilo del tema
-   ruta de configuración del tema: `channels.telegram.groups..topics.`

Caso especial del tema general (`threadId=1`):

-   los envíos de mensajes omiten `message_thread_id` (Telegram rechaza `sendMessage(...thread_id=1)`)
-   las acciones de escritura aún incluyen `message_thread_id`

Herencia de temas: las entradas de tema heredan la configuración del grupo a menos que se anulen (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`). `agentId` es solo para el tema y no hereda de los valores predeterminados del grupo. **Enrutamiento de agente por tema**: Cada tema puede enrutarse a un agente diferente configurando `agentId` en la configuración del tema. Esto le da a cada tema su propio espacio de trabajo, memoria y sesión aislados. Ejemplo:

```json
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "1": { agentId: "main" },      // Tema general → agente principal
            "3": { agentId: "zu" },        // Tema de desarrollo → agente zu
            "5": { agentId: "coder" }      // Revisión de código → agente coder
          }
        }
      }
    }
  }
}
```

Cada tema tiene entonces su propia clave de sesión: `agent:zu:telegram:group:-1001234567890:topic:3` **Vinculación persistente de tema ACP**: Los temas del foro pueden fijar sesiones de arnés ACP a través de vinculaciones ACP tipadas de alto nivel:

-   `bindings[]` con `type: "acp"` y `match.channel: "telegram"`

Ejemplo:

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
    },
  ],
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "42": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

Actualmente, esto está limitado a temas de foro en grupos y supergrupos. **Generación de ACP vinculada a hilo desde el chat**:

-   `/acp spawn  --thread here|auto` puede vincular el tema actual de Telegram a una nueva sesión ACP.
-   Los mensajes de seguimiento del tema se enrutan directamente a la sesión ACP vinculada (no se requiere `/acp steer`).
-   OpenClaw fija el mensaje de confirmación de generación en el tema después de una vinculación exitosa.
-   Requiere `channels.telegram.threadBindings.spawnAcpSessions=true`.

El contexto de la plantilla incluye:

-   `MessageThreadId`
-   `IsForum`

Comportamiento de hilos en mensajes directos:

-   los chats privados con `message_thread_id` mantienen el enrutamiento de MD pero usan claves de sesión/objetivos de respuesta conscientes del hilo.

### Mensajes de audio

Telegram distingue notas de voz vs archivos de audio.

-   predeterminado: comportamiento de archivo de audio
-   etiqueta `[[audio_as_voice]]` en la respuesta del agente para forzar el envío como nota de voz

Ejemplo de acción de mensaje:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

### Mensajes de video

Telegram distingue archivos de video vs notas de video. Ejemplo de acción de mensaje:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

Las notas de video no admiten subtítulos; el texto del mensaje proporcionado se envía por separado.

### Pegatinas

Manejo de pegatinas entrantes:

-   WEBP estático: descargado y procesado (marcador de posición `<media:sticker>`)
-   TGS animado: omitido
-   WEBM de video: omitido

Campos de contexto de la pegatina:

-   `Sticker.emoji`
-   `Sticker.setName`
-   `Sticker.fileId`
-   `Sticker.fileUniqueId`
-   `Sticker.cachedDescription`

Archivo de caché de pegatinas:

-   `~/.openclaw/telegram/sticker-cache.json`

Las pegatinas se describen una vez (cuando es posible) y se almacenan en caché para reducir llamadas de visión repetidas. Habilita acciones de pegatinas:

```json
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

Acción de enviar pegatina:

```json
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Busca pegatinas en caché:

```json
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Las reacciones de Telegram llegan como actualizaciones `message_reaction` (separadas de la carga útil del mensaje). Cuando están habilitadas, OpenClaw pone en cola eventos del sistema como:

-   `Reacción de Telegram agregada: 👍 por Alice (@alice) en el mensaje 42`

Configuración:

-   `channels.telegram.reactionNotifications`: `off | own | all` (predeterminado: `own`)
-   `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (predeterminado: `minimal`)

Notas:

-   `own` significa solo reacciones de usuario a mensajes enviados por el bot (mejor esfuerzo a través del caché de mensajes enviados).
-   Los eventos de reacción aún respetan los controles de acceso de Telegram (`dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`); los remitentes no autorizados se descartan.
-   Telegram no proporciona IDs de hilo en las actualizaciones de reacción.
    -   los grupos que no son foro se enrutan a la sesión de chat grupal
    -   los grupos de foro se enrutan a la sesión del tema general del grupo (`:topic:1`), no al tema de origen exacto

`allowed_updates` para polling/webhook incluye `message_reaction` automáticamente.

`ackReaction` envía un emoji de acuse de recibo mientras OpenClaw está procesando un mensaje entrante. Orden de resolución:

-   `channels.telegram.accounts..ackReaction`
-   `channels.telegram.ackReaction`
-   `messages.ackReaction`
-   respaldo del emoji de identidad del agente (`agents.list[].identity.emoji`, si no ”👀”)

Notas:

-   Telegram espera emojis Unicode (por ejemplo ”👀”).
-   Usa `""` para deshabilitar la reacción para un canal o cuenta.

Las escrituras de configuración del canal están habilitadas por defecto (`configWrites !== false`). Las escrituras desencadenadas por Telegram incluyen:

-   eventos de migración de grupo (`migrate_to_chat_id`) para actualizar `channels.telegram.groups`
-   `/config set` y `/config unset` (requiere habilitación del comando)

Deshabilitar:

```json
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

Predeterminado: long polling. Modo webhook:

-   configura `channels.telegram.webhookUrl`
-   configura `channels.telegram.webhookSecret` (requerido cuando se establece la URL del webhook)
-   opcional `channels.telegram.webhookPath` (predeterminado `/telegram-webhook`)
-   opcional `channels.telegram.webhookHost` (predeterminado `127.0.0.1`)
-   opcional `channels.telegram.webhookPort` (predeterminado `8787`)

El oyente local predeterminado para el modo webhook se vincula a `127.0.0.1:8787`. Si tu endpoint público es diferente, coloca un proxy inverso al frente y apunta `webhookUrl` a la URL pública. Configura `webhookHost` (por ejemplo `0.0.0.0`) cuando necesites intencionalmente entrada externa.

-   `channels.telegram.textChunkLimit` predeterminado es 4000.
-   `channels.telegram.chunkMode="newline"` prefiere límites de párrafo (líneas en blanco) antes de la división por longitud.
-   `channels.telegram.mediaMaxMb` (predeterminado 100) limita el tamaño de los medios de Telegram entrantes y salientes.
-   `channels.telegram.timeoutSeconds` anula el tiempo de espera del cliente de la API de Telegram (si no se establece, se aplica el valor predeterminado de grammY).
-   el historial de contexto del grupo usa `channels.telegram.historyLimit` o `messages.groupChat.historyLimit` (predeterminado 50); `0` deshabilita.
-   controles de historial de MD:
    -   `channels.telegram.dmHistoryLimit`
    -   `channels.telegram.dms["<user_id>"].historyLimit`
-   la configuración `channels.telegram.retry` se aplica a los ayudantes de envío de Telegram (CLI/herramientas/acciones) para errores de API salientes recuperables.

El objetivo de envío CLI puede ser un ID de chat numérico o un nombre de usuario:

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

Las encuestas de Telegram usan `openclaw message poll` y admiten temas de foro:

```bash
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "¿Enviarlo?" --poll-option "Sí" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Elige una hora" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300 --poll-public
```

Banderas de encuesta solo para Telegram:

-   `--poll-duration-seconds` (5-600)
-   `--poll-anonymous`
-   `--poll-public`
-   `--thread-id` para temas de foro (o usa un objetivo `:topic:`)

Habilitación de acciones:

-   `channels.telegram.actions.sendMessage=false` deshabilita los mensajes salientes de Telegram, incluidas las encuestas
-   `channels.telegram.actions.poll=false` deshabilita la creación de encuestas de Telegram mientras deja los envíos regulares habilitados

## Solución de problemas

-   Si `requireMention=false`, el modo de privacidad de Telegram debe permitir visibilidad completa.
    -   BotFather: `/setprivacy` -> Deshabilitar
    -   luego elimina + vuelve a agregar el bot al grupo
-   `openclaw channels status` advierte cuando la configuración espera mensajes de grupo sin mención.
-   `openclaw channels status --probe` puede verificar IDs de grupo numéricos explícitos; el comodín `"*"` no se puede sondear para membresía.
-   prueba rápida de sesión: `/activation always`.

-   cuando existe `channels.telegram.groups`, el grupo debe estar listado (o incluir `"*"`)
-   verifica la membresía del bot en el grupo
-   revisa los registros: `openclaw logs --follow` para ver razones de omisión

-   autoriza tu identidad de remitente (emparejamiento y/o `allowFrom` numérico)
-   la autorización de comandos aún se aplica incluso cuando la política de grupo es `open`
-   `setMyCommands failed` generalmente indica problemas de accesibilidad DNS/HTTPS a `api.telegram.org`

-   Node 22+ + fetch/proxy personalizado puede desencadenar un comportamiento de aborto inmediato si los tipos de AbortSignal no coinciden.
-   Algunos hosts resuelven `api.telegram.org` primero a IPv6; una salida IPv6 rota puede causar fallos intermitentes de la API de Telegram.
-   Si los registros incluyen `TypeError: fetch failed` o `Network request for 'getUpdates' failed!`, OpenClaw ahora reintenta estos como errores de red recuperables.
-   En hosts VPS con salida/TLS directa inestable, enruta las llamadas a la API de Telegram a través de `channels.telegram.proxy`:

```yaml
channels:
  telegram:
    proxy: socks5://<user>:<password>@proxy-host:1080
```

-   Node 22+ tiene como predeterminado `autoSelectFamily=true` (excepto WSL2) y `dnsResultOrder=ipv4first`.
-   Si tu host es WSL2 o funciona explícitamente mejor con comportamiento solo IPv4, fuerza la selección de familia:

```yaml
channels:
  telegram:
    network:
      autoSelectFamily: false
```

-   Anulaciones de entorno (temporales):
    -   `OPENCLAW_TELEGRAM_DISABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_ENABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_DNS_RESULT_ORDER=ipv4first`
-   Valida las respuestas DNS:

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

 Más ayuda: [Solución de problemas del canal](./troubleshooting.md).

## Puntos de referencia de configuración de Telegram

Referencia principal:

-   `channels.telegram.enabled`: habilita/deshabilita el inicio del canal.
-   `channels.telegram.botToken`: token del bot (BotFather).
-   `channels.telegram.tokenFile`: lee el token desde la ruta del archivo.
-   `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (predeterminado: pairing).
-   `channels.telegram.allowFrom`: lista de permitidos de MD (IDs de usuario de Telegram numéricos). `allowlist` requiere al menos un ID de remitente. `open` requiere `"*"`. `openclaw doctor --fix` puede resolver entradas heredadas `@username` a IDs y puede recuperar entradas de lista de permitidos de archivos de almacenamiento de emparejamiento en flujos de migración de lista de permitidos.
-   `channels.telegram.actions.poll`: habilita o deshabilita la creación de encuestas de Telegram (predeterminado: habilitado; aún requiere `sendMessage`).
-   `channels.telegram.defaultTo`: objetivo de Telegram predeterminado utilizado por la CLI `--deliver` cuando no se proporciona un `--reply-to` explícito.
-   `channels.telegram.groupPolicy`: `open | allowlist | disabled` (predeterminado: allowlist).
-   `channels.telegram.groupAllowFrom`: lista de permitidos de remitentes de grupo (IDs de usuario de Telegram numéricos). `openclaw doctor --fix` puede resolver entradas heredadas `@username` a IDs. Las entradas no numéricas se ignoran en el momento de la autorización. La autorización de grupo no usa el respaldo del almacenamiento de emparejamiento de MD (`2026.2.25+`).
-   Precedencia de múltiples cuentas:
    -   Cuando se configuran dos o más IDs de cuenta, configura `channels.telegram.defaultAccount` (o incluye `channels.telegram.accounts.default`) para hacer explícito el enrutamiento predeterminado.
    -   Si no se establece ninguno, OpenClaw recurre al primer ID de cuenta normalizado y `openclaw doctor` advierte.
    -   `channels.telegram.accounts.default.allowFrom` y `channels.telegram.accounts.default.groupAllowFrom` se aplican solo a la cuenta `default`.
    -   Las cuentas con nombre heredan `channels.telegram.allowFrom` y `channels.telegram.groupAllowFrom` cuando los valores a nivel de cuenta no están establecidos.
    -   Las cuentas con nombre no heredan `channels.telegram.accounts.default.allowFrom` / `groupAllowFrom`.
-   `channels.telegram.groups`: valores predeterminados por grupo + lista de permitidos (usa `"*"` para valores predeterminados globales).
    -   `channels.telegram.groups..groupPolicy`: anulación por grupo para groupPolicy (`open | allowlist | disabled`).
    -   `channels.telegram.groups..requireMention`: habilitación de mención predeterminada.
    -   `channels.telegram.groups..skills`: filtro de habilidades (omitir = todas las habilidades, vacío = ninguna).
    -   `channels.telegram.groups..allowFrom`: anulación de la lista de permitidos de remitentes por grupo.
    -   `channels.telegram.groups..systemPrompt`: mensaje de sistema adicional para el grupo.
    -   `channels.telegram.groups..enabled`: deshabilita el grupo cuando es `false`.
    -   `channels.telegram.groups..topics..*`: anulaciones por tema (campos de grupo + `agentId` solo para tema).
    -   `channels.telegram.groups..topics..agentId`: enruta este tema a un agente específico (anula el enrutamiento a nivel de grupo y vinculaciones).
    -   `channels.telegram.groups..topics..groupPolicy`: anulación por tema para groupPolicy (`open | allowlist | disabled`).
    -   `channels.telegram.groups..topics..requireMention`: anulación por tema para requerimiento de mención.