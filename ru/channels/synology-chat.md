title: "Руководство по установке и настройке плагина OpenClaw для Synology Chat"
description: "Узнайте, как установить и настроить плагин OpenClaw для Synology Chat. Отправляйте и получайте сообщения через вебхуки, настраивайте политики личных сообщений и управляйте несколькими учетными записями."
keywords: ["synology chat", "плагин openclaw", "настройка вебхука", "канал личных сообщений", "интеграция synology chat", "входящий вебхук", "исходящий вебхук", "политика лс"]
---

  Платформы обмена сообщениями

  
# Synology Chat

Статус: поддерживается через плагин в качестве канала для личных сообщений с использованием вебхуков Synology Chat. Плагин принимает входящие сообщения от исходящих вебхуков Synology Chat и отправляет ответы через входящий вебхук Synology Chat.

## Требуется плагин

Поддержка Synology Chat реализована через плагин и не входит в стандартную установку основных каналов. Установите из локальной копии репозитория:

```bash
openclaw plugins install ./extensions/synology-chat
```

Подробнее: [Плагины](../tools/plugin.md)

## Быстрая настройка

1.  Установите и включите плагин Synology Chat.
2.  В разделе интеграций Synology Chat:
    -   Создайте входящий вебхук и скопируйте его URL.
    -   Создайте исходящий вебхук со своим секретным токеном.
3.  Направьте URL исходящего вебхука на ваш шлюз OpenClaw:
    -   По умолчанию: `https://gateway-host/webhook/synology`.
    -   Или на ваш пользовательский путь `channels.synology-chat.webhookPath`.
4.  Настройте `channels.synology-chat` в OpenClaw.
5.  Перезапустите шлюз и отправьте личное сообщение боту в Synology Chat.

Минимальная конфигурация:

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      token: "synology-outgoing-token",
      incomingUrl: "https://nas.example.com/webapi/entry.cgi?api=SYNO.Chat.External&method=incoming&version=2&token=...",
      webhookPath: "/webhook/synology",
      dmPolicy: "allowlist",
      allowedUserIds: ["123456"],
      rateLimitPerMinute: 30,
      allowInsecureSsl: false,
    },
  },
}
```

## Переменные окружения

Для учетной записи по умолчанию можно использовать переменные окружения:

-   `SYNOLOGY_CHAT_TOKEN`
-   `SYNOLOGY_CHAT_INCOMING_URL`
-   `SYNOLOGY_NAS_HOST`
-   `SYNOLOGY_ALLOWED_USER_IDS` (через запятую)
-   `SYNOLOGY_RATE_LIMIT`
-   `OPENCLAW_BOT_NAME`

Значения из конфигурации имеют приоритет над переменными окружения.

## Политика ЛС и контроль доступа

-   `dmPolicy: "allowlist"` — рекомендуемое значение по умолчанию.
-   `allowedUserIds` принимает список (или строку, разделенную запятыми) идентификаторов пользователей Synology.
-   В режиме `allowlist` пустой список `allowedUserIds` считается ошибкой конфигурации, и маршрут вебхука не будет запущен (используйте `dmPolicy: "open"` для разрешения всем).
-   `dmPolicy: "open"` разрешает ЛС от любого отправителя.
-   `dmPolicy: "disabled"` блокирует ЛС.
-   Подтверждение сопряжения работает с командами:
    -   `openclaw pairing list synology-chat`
    -   `openclaw pairing approve synology-chat `

## Исходящая доставка

Используйте числовые идентификаторы пользователей Synology Chat в качестве целей. Примеры:

```bash
openclaw message send --channel synology-chat --target 123456 --text "Hello from OpenClaw"
openclaw message send --channel synology-chat --target synology-chat:123456 --text "Hello again"
```

Отправка медиафайлов поддерживается через доставку файлов по URL.

## Несколько учетных записей

Поддерживается несколько учетных записей Synology Chat в разделе `channels.synology-chat.accounts`. Каждая учетная запись может переопределять токен, URL входящего вебхука, путь вебхука, политику ЛС и ограничения.

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      accounts: {
        default: {
          token: "token-a",
          incomingUrl: "https://nas-a.example.com/...token=...",
        },
        alerts: {
          token: "token-b",
          incomingUrl: "https://nas-b.example.com/...token=...",
          webhookPath: "/webhook/synology-alerts",
          dmPolicy: "allowlist",
          allowedUserIds: ["987654"],
        },
      },
    },
  },
}
```

## Примечания по безопасности

-   Храните `token` в секрете и обновите его в случае утечки.
-   Оставляйте `allowInsecureSsl: false`, если вы явно не доверяете самоподписанному сертификату локального NAS.
-   Входящие запросы вебхуков проверяются по токену и ограничиваются по частоте для каждого отправителя.
-   Для продакшена предпочтительна политика `dmPolicy: "allowlist"`.

[Signal](./signal.md)[Slack](./slack.md)

---