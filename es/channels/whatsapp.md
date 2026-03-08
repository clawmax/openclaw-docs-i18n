

  Plataformas de mensajería

  
# WhatsApp

Estado: listo para producción vía WhatsApp Web (Baileys). El gateway posee la(s) sesión(es) vinculada(s).

## Configuración rápida

### Paso 1: Configurar la política de acceso de WhatsApp

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

### Paso 2: Vincular WhatsApp (QR)

```bash
openclaw channels login --channel whatsapp
```

Para una cuenta específica:

```bash
openclaw channels login --channel whatsapp --account work
```

### Paso 3: Iniciar el gateway

```bash
openclaw gateway
```

### Paso 4: Aprobar la primera solicitud de vinculación (si se usa el modo pairing)

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

Las solicitudes de vinculación expiran después de 1 hora. Las solicitudes pendientes están limitadas a 3 por canal.

 

> **ℹ️** OpenClaw recomienda ejecutar WhatsApp en un número separado cuando sea posible. (Los metadatos del canal y el flujo de incorporación están optimizados para esa configuración, pero también se admiten configuraciones con número personal.)

## Patrones de despliegue

Este es el modo operativo más limpio:

-   identidad de WhatsApp separada para OpenClaw
-   límites de listas de permitidos y enrutamiento más claros
-   menor probabilidad de confusión con chats propios

Patrón de política mínimo:

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

La incorporación admite el modo de número personal y escribe una línea base compatible con chats propios:

-   `dmPolicy: "allowlist"`
-   `allowFrom` incluye tu número personal
-   `selfChatMode: true`

En tiempo de ejecución, las protecciones de chat propio se activan con base en el número propio vinculado y `allowFrom`.

El canal de plataforma de mensajería está basado en WhatsApp Web (`Baileys`) en la arquitectura actual de canales de OpenClaw. No existe un canal de mensajería WhatsApp de Twilio separado en el registro de canales de chat integrado.

## Modelo de tiempo de ejecución

-   El gateway posee el socket de WhatsApp y el bucle de reconexión.
-   Los envíos salientes requieren un listener de WhatsApp activo para la cuenta objetivo.
-   Los chats de estado y difusión se ignoran (`@status`, `@broadcast`).
-   Los chats directos usan reglas de sesión DM (`session.dmScope`; por defecto `main` colapsa los DMs a la sesión principal del agente).
-   Las sesiones de grupo están aisladas (`agent::whatsapp:group:`).

## Control de acceso y activación

`channels.whatsapp.dmPolicy` controla el acceso a chats directos:

-   `pairing` (predeterminado)
-   `allowlist`
-   `open` (requiere que `allowFrom` incluya `"*"`)
-   `disabled`

`allowFrom` acepta números en formato E.164 (normalizados internamente).Anulación multi-cuenta: `channels.whatsapp.accounts..dmPolicy` (y `allowFrom`) tienen prioridad sobre los valores predeterminados a nivel de canal para esa cuenta.Detalles del comportamiento en tiempo de ejecución:

-   las vinculaciones se persisten en el almacén de permitidos del canal y se fusionan con `allowFrom` configurado
-   si no se configura ninguna lista de permitidos, el número propio vinculado se permite por defecto
-   los DMs salientes `fromMe` nunca se vinculan automáticamente

El acceso a grupos tiene dos capas:

1.  **Lista de permitidos de membresía de grupo** (`channels.whatsapp.groups`)
    -   si se omite `groups`, todos los grupos son elegibles
    -   si `groups` está presente, actúa como una lista de permitidos de grupo (`"*"` permitido)
2.  **Política de remitente de grupo** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
    -   `open`: se omite la lista de permitidos del remitente
    -   `allowlist`: el remitente debe coincidir con `groupAllowFrom` (o `*`)
    -   `disabled`: bloquear todas las entradas de grupo

Lista de permitidos de remitente alternativa:

-   si `groupAllowFrom` no está configurado, el tiempo de ejecución recurre a `allowFrom` cuando está disponible
-   las listas de permitidos del remitente se evalúan antes de la activación por mención/respuesta

Nota: si no existe ningún bloque `channels.whatsapp` en absoluto, la política de grupo alternativa en tiempo de ejecución es `allowlist` (con un registro de advertencia), incluso si `channels.defaults.groupPolicy` está configurado.

Las respuestas en grupo requieren mención por defecto.La detección de menciones incluye:

-   menciones explícitas de WhatsApp a la identidad del bot
-   patrones de expresión regular de mención configurados (`agents.list[].groupChat.mentionPatterns`, alternativa `messages.groupChat.mentionPatterns`)
-   detección implícita de respuesta-al-bot (el remitente de la respuesta coincide con la identidad del bot)

Nota de seguridad:

-   la cita/respuesta solo satisface la compuerta de mención; **no** otorga autorización de remitente
-   con `groupPolicy: "allowlist"`, los remitentes no permitidos aún se bloquean incluso si responden a un mensaje de un usuario permitido

Comando de activación a nivel de sesión:

-   `/activation mention`
-   `/activation always`

`activation` actualiza el estado de la sesión (no la configuración global). Está controlado por el propietario.

## Comportamiento de número personal y chat propio

Cuando el número propio vinculado también está presente en `allowFrom`, se activan las salvaguardas de chat propio de WhatsApp:

-   omitir recibos de lectura para turnos de chat propio
-   ignorar el comportamiento de activación automática por mención-JID que de otro modo se enviaría a ti mismo
-   si `messages.responsePrefix` no está configurado, las respuestas de chat propio usan por defecto `[{identity.name}]` o `[openclaw]`

## Normalización de mensajes y contexto

Los mensajes entrantes de WhatsApp se envuelven en el sobre de entrada compartido.Si existe una respuesta citada, el contexto se agrega en esta forma:

```json
[Respondiendo a <remitente> id:<stanzaId>]
<cuerpo citado o marcador de posición de medios>
[/Respondiendo]
```

Los campos de metadatos de respuesta también se completan cuando están disponibles (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, JID/E.164 del remitente).

Los mensajes entrantes que solo contienen medios se normalizan con marcadores de posición como:

-   `<media:image>`
-   `<media:video>`
-   `<media:audio>`
-   `<media:document>`
-   `<media:sticker>`

Las cargas útiles de ubicación y contacto se normalizan en contexto textual antes del enrutamiento.

Para grupos, los mensajes no procesados pueden almacenarse en búfer e inyectarse como contexto cuando finalmente se activa el bot.

-   límite predeterminado: `50`
-   configuración: `channels.whatsapp.historyLimit`
-   alternativa: `messages.groupChat.historyLimit`
-   `0` desactiva

Marcadores de inyección:

-   `[Mensajes del chat desde tu última respuesta - para contexto]`
-   `[Mensaje actual - responde a esto]`

Los recibos de lectura están habilitados por defecto para los mensajes entrantes de WhatsApp aceptados.Desactivar globalmente:

```json
{
  channels: {
    whatsapp: {
      sendReadReceipts: false,
    },
  },
}
```

Anulación por cuenta:

```json
{
  channels: {
    whatsapp: {
      accounts: {
        work: {
          sendReadReceipts: false,
        },
      },
    },
  },
}
```

Los turnos de chat propio omiten los recibos de lectura incluso cuando están habilitados globalmente.

## Entrega, fragmentación y medios

-   límite de fragmento predeterminado: `channels.whatsapp.textChunkLimit = 4000`
-   `channels.whatsapp.chunkMode = "length" | "newline"`
-   el modo `newline` prefiere límites de párrafo (líneas en blanco), luego recurre a fragmentación segura por longitud

-   admite cargas útiles de imagen, video, audio (nota de voz PTT) y documento
-   `audio/ogg` se reescribe a `audio/ogg; codecs=opus` para compatibilidad con notas de voz
-   la reproducción de GIF animados es compatible mediante `gifPlayback: true` en envíos de video
-   los subtítulos se aplican al primer elemento multimedia al enviar cargas útiles de respuesta multi-media
-   la fuente de medios puede ser HTTP(S), `file://` o rutas locales

-   límite de guardado de medios entrantes: `channels.whatsapp.mediaMaxMb` (predeterminado `50`)
-   límite de envío de medios salientes: `channels.whatsapp.mediaMaxMb` (predeterminado `50`)
-   las anulaciones por cuenta usan `channels.whatsapp.accounts..mediaMaxMb`
-   las imágenes se optimizan automáticamente (redimensionamiento/ajuste de calidad) para ajustarse a los límites
-   en caso de fallo de envío de medios, la alternativa del primer elemento envía una advertencia de texto en lugar de descartar la respuesta en silencio

## Reacciones de acuse de recibo

WhatsApp admite reacciones de acuse de recibo inmediatas en la recepción entrante mediante `channels.whatsapp.ackReaction`.

```json
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

Notas de comportamiento:

-   se envía inmediatamente después de aceptar la entrada (antes de la respuesta)
-   los fallos se registran pero no bloquean la entrega normal de la respuesta
-   el modo de grupo `mentions` reacciona en turnos activados por mención; la activación de grupo `always` actúa como omisión para esta verificación
-   WhatsApp usa `channels.whatsapp.ackReaction` (el legado `messages.ackReaction` no se usa aquí)

## Multi-cuenta y credenciales

-   los ids de cuenta provienen de `channels.whatsapp.accounts`
-   selección de cuenta predeterminada: `default` si está presente, de lo contrario el primer id de cuenta configurado (ordenado)
-   los ids de cuenta se normalizan internamente para búsqueda

-   ruta de autenticación actual: `~/.openclaw/credentials/whatsapp//creds.json`
-   archivo de respaldo: `creds.json.bak`
-   la autenticación predeterminada anterior en `~/.openclaw/credentials/` aún se reconoce/migra para flujos de cuenta predeterminada

`openclaw channels logout --channel whatsapp [--account ]` borra el estado de autenticación de WhatsApp para esa cuenta.En directorios de autenticación anteriores, se preserva `oauth.json` mientras se eliminan los archivos de autenticación de Baileys.

## Herramientas, acciones y escrituras de configuración

-   El soporte de herramientas del agente incluye la acción de reacción de WhatsApp (`react`).
-   Compuertas de acción:
    -   `channels.whatsapp.actions.reactions`
    -   `channels.whatsapp.actions.polls`
-   Las escrituras de configuración iniciadas por el canal están habilitadas por defecto (desactivar mediante `channels.whatsapp.configWrites=false`).

## Solución de problemas

Síntoma: el estado del canal reporta "no vinculado".Solución:

```bash
openclaw channels login --channel whatsapp
openclaw channels status
```

Síntoma: cuenta vinculada con desconexiones repetidas o intentos de reconexión.Solución:

```bash
openclaw doctor
openclaw logs --follow
```

Si es necesario, volver a vincular con `channels login`.

Los envíos salientes fallan rápidamente cuando no existe un listener de gateway activo para la cuenta objetivo.Asegúrate de que el gateway esté en ejecución y la cuenta esté vinculada.

Verificar en este orden:

-   `groupPolicy`
-   `groupAllowFrom` / `allowFrom`
-   entradas de la lista de permitidos `groups`
-   compuerta de mención (`requireMention` + patrones de mención)
-   claves duplicadas en `openclaw.json` (JSON5): las entradas posteriores anulan las anteriores, así que mantén un solo `groupPolicy` por alcance

El tiempo de ejecución del gateway de WhatsApp debe usar Node. Bun se marca como incompatible para una operación estable del gateway de WhatsApp/Telegram.

## Puntos de referencia de configuración

Referencia principal:

-   [Referencia de configuración - WhatsApp](../gateway/configuration-reference.md#whatsapp)

Campos de WhatsApp de alta señal:

-   acceso: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
-   entrega: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
-   multi-cuenta: `accounts..enabled`, `accounts..authDir`, anulaciones a nivel de cuenta
-   operaciones: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
-   comportamiento de sesión: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms..historyLimit`

## Relacionado

-   [Vinculación (Pairing)](./pairing.md)
-   [Enrutamiento de canales](./channel-routing.md)
-   [Enrutamiento multi-agente](../concepts/multi-agent.md)
-   [Solución de problemas](./troubleshooting.md)

[Twitch](./twitch.md)[Zalo](./zalo.md)