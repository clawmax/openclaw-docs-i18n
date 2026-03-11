

  Автоматизация

  
# Опросы

## Поддерживаемые каналы

-   Telegram
-   WhatsApp (веб-канал)
-   Discord
-   MS Teams (Adaptive Cards)

## CLI

```bash
# Telegram
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "Ship it?" --poll-option "Yes" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Pick a time" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300

# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

Опции:

-   `--channel`: `whatsapp` (по умолчанию), `telegram`, `discord` или `msteams`
-   `--poll-multi`: разрешить выбор нескольких вариантов
-   `--poll-duration-hours`: только для Discord (по умолчанию 24, если не указано)
-   `--poll-duration-seconds`: только для Telegram (5-600 секунд)
-   `--poll-anonymous` / `--poll-public`: видимость опроса, только для Telegram

## Gateway RPC

Метод: `poll` Параметры:

-   `to` (строка, обязательный)
-   `question` (строка, обязательный)
-   `options` (массив строк, обязательный)
-   `maxSelections` (число, опционально)
-   `durationHours` (число, опционально)
-   `durationSeconds` (число, опционально, только для Telegram)
-   `isAnonymous` (логический, опционально, только для Telegram)
-   `channel` (строка, опционально, по умолчанию: `whatsapp`)
-   `idempotencyKey` (строка, обязательный)

## Различия между каналами

-   Telegram: 2-10 вариантов. Поддерживает темы форума через `threadId` или цели с `:topic:`. Использует `durationSeconds` вместо `durationHours`, ограничено 5-600 секундами. Поддерживает анонимные и публичные опросы.
-   WhatsApp: 2-12 вариантов, `maxSelections` должен быть в пределах количества вариантов, игнорирует `durationHours`.
-   Discord: 2-10 вариантов, `durationHours` ограничено 1-768 часами (по умолчанию 24). `maxSelections > 1` включает множественный выбор; Discord не поддерживает строгое ограничение на количество выборов.
-   MS Teams: Опросы в виде Adaptive Cards (управляются OpenClaw). Нет нативного API для опросов; `durationHours` игнорируется.

## Инструмент Агента (Message)

Используйте инструмент `message` с действием `poll` (`to`, `pollQuestion`, `pollOption`, опционально `pollMulti`, `pollDurationHours`, `channel`). Для Telegram инструмент также принимает `pollDurationSeconds`, `pollAnonymous` и `pollPublic`. Используйте `action: "poll"` для создания опроса. Поля опроса, переданные с `action: "send"`, отклоняются. Примечание: в Discord нет режима «выбрать ровно N»; `pollMulti` соответствует множественному выбору. Опросы в Teams отображаются как Adaptive Cards и требуют, чтобы шлюз оставался онлайн для записи голосов в `~/.openclaw/msteams-polls.json`.

[Gmail PubSub](./gmail-pubsub.md)[Мониторинг аутентификации](./auth-monitoring.md)

---