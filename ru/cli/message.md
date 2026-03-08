title: "Руководство по команде message в OpenClaw CLI для многоканальной отправки сообщений"
description: "Научитесь использовать команду message в OpenClaw CLI для отправки сообщений, опросов и реакций через Discord, Slack, WhatsApp, Telegram и другие платформы."
keywords: ["openclaw cli", "команда message", "многоканальная отправка сообщений", "discord бот", "автоматизация slack", "опрос в telegram", "whatsapp api", "автоматизация cli"]
---

  Команды CLI

  
# message

Единая команда для исходящих действий: отправка сообщений и действий в каналах (Discord/Google Chat/Slack/Mattermost (плагин)/Telegram/WhatsApp/Signal/iMessage/MS Teams).

## Использование

```bash
openclaw message <subcommand> [flags]
```

Выбор канала:

-   `--channel` обязателен, если настроено более одного канала.
-   Если настроен ровно один канал, он становится каналом по умолчанию.
-   Значения: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost требует плагин)

Форматы цели (`--target`):

-   WhatsApp: E.164 или групповой JID
-   Telegram: chat id или `@username`
-   Discord: `channel:` или `user:` (или упоминание `<@id>`; сырые числовые id трактуются как каналы)
-   Google Chat: `spaces/` или `users/`
-   Slack: `channel:` или `user:` (принимается сырой id канала)
-   Mattermost (плагин): `channel:`, `user:` или `@username` (голые id трактуются как каналы)
-   Signal: `+E.164`, `group:`, `signal:+E.164`, `signal:group:` или `username:`/`u:`
-   iMessage: handle, `chat_id:`, `chat_guid:` или `chat_identifier:`
-   MS Teams: id беседы (`19:...@thread.tacv2`) или `conversation:` или `user:<aad-object-id>`

Поиск по имени:

-   Для поддерживаемых провайдеров (Discord/Slack/и др.) имена каналов, такие как `Help` или `#help`, разрешаются через кэш директории.
-   При промахе кэша OpenClaw попытается выполнить живой поиск в директории, если провайдер это поддерживает.

## Общие флаги

-   `--channel `
-   `--account `
-   `--target ` (целевой канал или пользователь для send/poll/read/и т.д.)
-   `--targets ` (повторяется; только для broadcast)
-   `--json`
-   `--dry-run`
-   `--verbose`

## Действия

### Основные

-   `send`
    -   Каналы: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (плагин)/Signal/iMessage/MS Teams
    -   Обязательно: `--target`, плюс `--message` или `--media`
    -   Опционально: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
    -   Только Telegram: `--buttons` (требует `channels.telegram.capabilities.inlineButtons` для разрешения)
    -   Только Telegram: `--thread-id` (id темы форума)
    -   Только Slack: `--thread-id` (timestamp треда; `--reply-to` использует то же поле)
    -   Только WhatsApp: `--gif-playback`
-   `poll`
    -   Каналы: WhatsApp/Telegram/Discord/Matrix/MS Teams
    -   Обязательно: `--target`, `--poll-question`, `--poll-option` (повторяется)
    -   Опционально: `--poll-multi`
    -   Только Discord: `--poll-duration-hours`, `--silent`, `--message`
    -   Только Telegram: `--poll-duration-seconds` (5-600), `--silent`, `--poll-anonymous` / `--poll-public`, `--thread-id`
-   `react`
    -   Каналы: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
    -   Обязательно: `--message-id`, `--target`
    -   Опционально: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
    -   Примечание: `--remove` требует `--emoji` (опустите `--emoji` для очистки собственных реакций, где поддерживается; см. /tools/reactions)
    -   Только WhatsApp: `--participant`, `--from-me`
    -   Реакции в группах Signal: требуется `--target-author` или `--target-author-uuid`
-   `reactions`
    -   Каналы: Discord/Google Chat/Slack
    -   Обязательно: `--message-id`, `--target`
    -   Опционально: `--limit`
-   `read`
    -   Каналы: Discord/Slack
    -   Обязательно: `--target`
    -   Опционально: `--limit`, `--before`, `--after`
    -   Только Discord: `--around`
-   `edit`
    -   Каналы: Discord/Slack
    -   Обязательно: `--message-id`, `--message`, `--target`
-   `delete`
    -   Каналы: Discord/Slack/Telegram
    -   Обязательно: `--message-id`, `--target`
-   `pin` / `unpin`
    -   Каналы: Discord/Slack
    -   Обязательно: `--message-id`, `--target`
-   `pins` (список)
    -   Каналы: Discord/Slack
    -   Обязательно: `--target`
-   `permissions`
    -   Каналы: Discord
    -   Обязательно: `--target`
-   `search`
    -   Каналы: Discord
    -   Обязательно: `--guild-id`, `--query`
    -   Опционально: `--channel-id`, `--channel-ids` (повторяется), `--author-id`, `--author-ids` (повторяется), `--limit`

### Треды

-   `thread create`
    -   Каналы: Discord
    -   Обязательно: `--thread-name`, `--target` (id канала)
    -   Опционально: `--message-id`, `--message`, `--auto-archive-min`
-   `thread list`
    -   Каналы: Discord
    -   Обязательно: `--guild-id`
    -   Опционально: `--channel-id`, `--include-archived`, `--before`, `--limit`
-   `thread reply`
    -   Каналы: Discord
    -   Обязательно: `--target` (id треда), `--message`
    -   Опционально: `--media`, `--reply-to`

### Эмодзи

-   `emoji list`
    -   Discord: `--guild-id`
    -   Slack: без дополнительных флагов
-   `emoji upload`
    -   Каналы: Discord
    -   Обязательно: `--guild-id`, `--emoji-name`, `--media`
    -   Опционально: `--role-ids` (повторяется)

### Стикеры

-   `sticker send`
    -   Каналы: Discord
    -   Обязательно: `--target`, `--sticker-id` (повторяется)
    -   Опционально: `--message`
-   `sticker upload`
    -   Каналы: Discord
    -   Обязательно: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

### Роли / Каналы / Участники / Голос

-   `role info` (Discord): `--guild-id`
-   `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
-   `channel info` (Discord): `--target`
-   `channel list` (Discord): `--guild-id`
-   `member info` (Discord/Slack): `--user-id` (+ `--guild-id` для Discord)
-   `voice status` (Discord): `--guild-id`, `--user-id`

### События

-   `event list` (Discord): `--guild-id`
-   `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
    -   Опционально: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

### Модерация (Discord)

-   `timeout`: `--guild-id`, `--user-id` (опционально `--duration-min` или `--until`; опустите оба, чтобы снять тайм-аут)
-   `kick`: `--guild-id`, `--user-id` (+ `--reason`)
-   `ban`: `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
    -   `timeout` также поддерживает `--reason`

### Рассылка

-   `broadcast`
    -   Каналы: любой настроенный канал; используйте `--channel all` для таргетинга всех провайдеров
    -   Обязательно: `--targets` (повторяется)
    -   Опционально: `--message`, `--media`, `--dry-run`

## Примеры

Отправить ответ в Discord:

```bash
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

Отправить сообщение в Discord с компонентами:

```bash
openclaw message send --channel discord \
  --target channel:123 --message "Choose:" \
  --components '{"text":"Choose a path","blocks":[{"type":"actions","buttons":[{"label":"Approve","style":"success"},{"label":"Decline","style":"danger"}]}]}'
```

См. [Компоненты Discord](../channels/discord.md#interactive-components) для полной схемы. Создать опрос в Discord:

```bash
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Создать опрос в Telegram (автозакрытие через 2 минуты):

```bash
openclaw message poll --channel telegram \
  --target @mychat \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-duration-seconds 120 --silent
```

Отправить проактивное сообщение в Teams:

```bash
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Создать опрос в Teams:

```bash
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

Поставить реакцию в Slack:

```bash
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

Поставить реакцию в группе Signal:

```bash
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Отправить встроенные кнопки в Telegram:

```bash
openclaw message send --channel telegram --target @mychat --message "Choose:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```

[memory](./memory.md)[models](./models.md)