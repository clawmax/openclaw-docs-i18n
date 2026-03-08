

  Comandos CLI

  
# message

Comando de salida único para enviar mensajes y acciones de canal (Discord/Google Chat/Slack/Mattermost (plugin)/Telegram/WhatsApp/Signal/iMessage/MS Teams).

## Uso

```bash
openclaw message <subcomando> [flags]
```

Selección de canal:

-   `--channel` requerido si hay más de un canal configurado.
-   Si hay exactamente un canal configurado, se convierte en el predeterminado.
-   Valores: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost requiere plugin)

Formatos de destino (`--target`):

-   WhatsApp: E.164 o JID de grupo
-   Telegram: id de chat o `@nombredeusuario`
-   Discord: `channel:` o `user:` (o mención `<@id>`; los ids numéricos crudos se tratan como canales)
-   Google Chat: `spaces/` o `users/`
-   Slack: `channel:` o `user:` (se acepta el id de canal crudo)
-   Mattermost (plugin): `channel:`, `user:`, o `@nombredeusuario` (los ids sueltos se tratan como canales)
-   Signal: `+E.164`, `group:`, `signal:+E.164`, `signal:group:`, o `username:`/`u:`
-   iMessage: handle, `chat_id:`, `chat_guid:`, o `chat_identifier:`
-   MS Teams: id de conversación (`19:...@thread.tacv2`) o `conversation:` o `user:<aad-object-id>`

Búsqueda por nombre:

-   Para proveedores compatibles (Discord/Slack/etc), nombres de canal como `Ayuda` o `#ayuda` se resuelven a través de la caché del directorio.
-   En caso de fallo de caché, OpenClaw intentará una búsqueda en vivo en el directorio cuando el proveedor lo admita.

## Banderas comunes

-   `--channel `
-   `--account `
-   `--target ` (canal o usuario destino para enviar/encuesta/leer/etc)
-   `--targets ` (repetir; solo para difusión)
-   `--json`
-   `--dry-run`
-   `--verbose`

## Acciones

### Núcleo

-   `send`
    -   Canales: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams
    -   Requerido: `--target`, más `--message` o `--media`
    -   Opcional: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
    -   Solo Telegram: `--buttons` (requiere `channels.telegram.capabilities.inlineButtons` para permitirlo)
    -   Solo Telegram: `--thread-id` (id de tema del foro)
    -   Solo Slack: `--thread-id` (marca de tiempo del hilo; `--reply-to` usa el mismo campo)
    -   Solo WhatsApp: `--gif-playback`
-   `poll`
    -   Canales: WhatsApp/Telegram/Discord/Matrix/MS Teams
    -   Requerido: `--target`, `--poll-question`, `--poll-option` (repetir)
    -   Opcional: `--poll-multi`
    -   Solo Discord: `--poll-duration-hours`, `--silent`, `--message`
    -   Solo Telegram: `--poll-duration-seconds` (5-600), `--silent`, `--poll-anonymous` / `--poll-public`, `--thread-id`
-   `react`
    -   Canales: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
    -   Requerido: `--message-id`, `--target`
    -   Opcional: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
    -   Nota: `--remove` requiere `--emoji` (omite `--emoji` para limpiar reacciones propias donde se admita; ver /tools/reactions)
    -   Solo WhatsApp: `--participant`, `--from-me`
    -   Reacciones en grupo de Signal: se requiere `--target-author` o `--target-author-uuid`
-   `reactions`
    -   Canales: Discord/Google Chat/Slack
    -   Requerido: `--message-id`, `--target`
    -   Opcional: `--limit`
-   `read`
    -   Canales: Discord/Slack
    -   Requerido: `--target`
    -   Opcional: `--limit`, `--before`, `--after`
    -   Solo Discord: `--around`
-   `edit`
    -   Canales: Discord/Slack
    -   Requerido: `--message-id`, `--message`, `--target`
-   `delete`
    -   Canales: Discord/Slack/Telegram
    -   Requerido: `--message-id`, `--target`
-   `pin` / `unpin`
    -   Canales: Discord/Slack
    -   Requerido: `--message-id`, `--target`
-   `pins` (listar)
    -   Canales: Discord/Slack
    -   Requerido: `--target`
-   `permissions`
    -   Canales: Discord
    -   Requerido: `--target`
-   `search`
    -   Canales: Discord
    -   Requerido: `--guild-id`, `--query`
    -   Opcional: `--channel-id`, `--channel-ids` (repetir), `--author-id`, `--author-ids` (repetir), `--limit`

### Hilos

-   `thread create`
    -   Canales: Discord
    -   Requerido: `--thread-name`, `--target` (id del canal)
    -   Opcional: `--message-id`, `--message`, `--auto-archive-min`
-   `thread list`
    -   Canales: Discord
    -   Requerido: `--guild-id`
    -   Opcional: `--channel-id`, `--include-archived`, `--before`, `--limit`
-   `thread reply`
    -   Canales: Discord
    -   Requerido: `--target` (id del hilo), `--message`
    -   Opcional: `--media`, `--reply-to`

### Emojis

-   `emoji list`
    -   Discord: `--guild-id`
    -   Slack: sin banderas extra
-   `emoji upload`
    -   Canales: Discord
    -   Requerido: `--guild-id`, `--emoji-name`, `--media`
    -   Opcional: `--role-ids` (repetir)

### Stickers

-   `sticker send`
    -   Canales: Discord
    -   Requerido: `--target`, `--sticker-id` (repetir)
    -   Opcional: `--message`
-   `sticker upload`
    -   Canales: Discord
    -   Requerido: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

### Roles / Canales / Miembros / Voz

-   `role info` (Discord): `--guild-id`
-   `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
-   `channel info` (Discord): `--target`
-   `channel list` (Discord): `--guild-id`
-   `member info` (Discord/Slack): `--user-id` (+ `--guild-id` para Discord)
-   `voice status` (Discord): `--guild-id`, `--user-id`

### Eventos

-   `event list` (Discord): `--guild-id`
-   `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
    -   Opcional: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

### Moderación (Discord)

-   `timeout`: `--guild-id`, `--user-id` (opcional `--duration-min` o `--until`; omite ambos para limpiar el tiempo de espera)
-   `kick`: `--guild-id`, `--user-id` (+ `--reason`)
-   `ban`: `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
    -   `timeout` también admite `--reason`

### Difusión

-   `broadcast`
    -   Canales: cualquier canal configurado; usa `--channel all` para apuntar a todos los proveedores
    -   Requerido: `--targets` (repetir)
    -   Opcional: `--message`, `--media`, `--dry-run`

## Ejemplos

Enviar una respuesta en Discord:

```bash
openclaw message send --channel discord \
  --target channel:123 --message "hola" --reply-to 456
```

Enviar un mensaje de Discord con componentes:

```bash
openclaw message send --channel discord \
  --target channel:123 --message "Elige:" \
  --components '{"text":"Choose a path","blocks":[{"type":"actions","buttons":[{"label":"Approve","style":"success"},{"label":"Decline","style":"danger"}]}]}'
```

Consulta [Componentes de Discord](../channels/discord.md#interactive-components) para el esquema completo. Crear una encuesta en Discord:

```bash
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "¿Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Crear una encuesta en Telegram (cierre automático en 2 minutos):

```bash
openclaw message poll --channel telegram \
  --target @mychat \
  --poll-question "¿Almuerzo?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-duration-seconds 120 --silent
```

Enviar un mensaje proactivo en Teams:

```bash
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hola"
```

Crear una encuesta en Teams:

```bash
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "¿Almuerzo?" \
  --poll-option Pizza --poll-option Sushi
```

Reaccionar en Slack:

```bash
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

Reaccionar en un grupo de Signal:

```bash
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Enviar botones en línea de Telegram:

```bash
openclaw message send --channel telegram --target @mychat --message "Elige:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```

[memory](./memory.md)[models](./models.md)