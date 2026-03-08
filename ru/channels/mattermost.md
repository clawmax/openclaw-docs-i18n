title: "Руководство по установке и настройке интеграции OpenClaw с Mattermost"
description: "Узнайте, как установить и настроить плагин Mattermost для OpenClaw. Настройте токены бота, слеш-команды, режимы чата, контроль доступа и интерактивные функции."
keywords: ["mattermost", "openclaw", "интеграция чат-бота", "установка плагина", "слеш-команды", "командный мессенджер", "настройка бота", "самостоятельный хостинг чата"]
---

  Платформы обмена сообщениями

  
# Mattermost

Статус: поддерживается через плагин (токен бота + события WebSocket). Поддерживаются каналы, группы и личные сообщения. Mattermost — это платформа для командного обмена сообщениями с возможностью самостоятельного хостинга; подробности о продукте и загрузках смотрите на официальном сайте [mattermost.com](https://mattermost.com).

## Требуется плагин

Mattermost поставляется в виде плагина и не входит в базовую установку. Установите через CLI (npm registry):

```bash
openclaw plugins install @openclaw/mattermost
```

Локальная установка (при запуске из git-репозитория):

```bash
openclaw plugins install ./extensions/mattermost
```

Если вы выберете Mattermost во время настройки/первого запуска и будет обнаружена локальная копия репозитория, OpenClaw автоматически предложит путь для локальной установки. Подробнее: [Плагины](../tools/plugin.md)

## Быстрая настройка

1.  Установите плагин Mattermost.
2.  Создайте учетную запись бота Mattermost и скопируйте **токен бота**.
3.  Скопируйте **базовый URL** Mattermost (например, `https://chat.example.com`).
4.  Настройте OpenClaw и запустите шлюз.

Минимальная конфигурация:

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## Нативные слеш-команды

Нативные слеш-команды включаются опционально. При включении OpenClaw регистрирует команды `oc_*` через API Mattermost и получает POST-запросы обратного вызова на HTTP-сервере шлюза.

```json
{
  channels: {
    mattermost: {
      commands: {
        native: true,
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // Используйте, когда Mattermost не может напрямую обратиться к шлюзу (обратный прокси/публичный URL).
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
    },
  },
}
```

Примечания:

-   `native: "auto"` по умолчанию отключено для Mattermost. Установите `native: true` для включения.
-   Если `callbackUrl` не указан, OpenClaw формирует его из хоста/порта шлюза и `callbackPath`.
-   Для настройки нескольких учетных записей `commands` можно задать на верхнем уровне или в `channels.mattermost.accounts..commands` (значения для учетной записи переопределяют поля верхнего уровня).
-   Обратные вызовы команд проверяются с помощью токенов для каждой команды и завершаются ошибкой при неудачной проверке токена.
-   Требование доступности: конечная точка обратного вызова должна быть доступна с сервера Mattermost.
    -   Не указывайте `callbackUrl` как `localhost`, если только Mattermost не работает на том же хосте/сетевом пространстве имен, что и OpenClaw.
    -   Не указывайте `callbackUrl` как ваш базовый URL Mattermost, если только этот URL не проксирует запросы `/api/channels/mattermost/command` на OpenClaw.
    -   Быстрая проверка: `curl https://<gateway-host>/api/channels/mattermost/command`; GET-запрос должен вернуть `405 Method Not Allowed` от OpenClaw, а не `404`.
-   Требование к списку разрешенных исходящих подключений Mattermost:
    -   Если ваш обратный вызов указывает на приватные/Tailnet/внутренние адреса, установите в Mattermost `ServiceSettings.AllowedUntrustedInternalConnections` включение хоста/домена обратного вызова.
    -   Используйте записи хоста/домена, а не полные URL.
        -   Хорошо: `gateway.tailnet-name.ts.net`
        -   Плохо: `https://gateway.tailnet-name.ts.net`

## Переменные окружения (учетная запись по умолчанию)

Установите эти переменные на хосте шлюза, если предпочитаете использовать env vars:

-   `MATTERMOST_BOT_TOKEN=...`
-   `MATTERMOST_URL=https://chat.example.com`

Переменные окружения применяются только к **учетной записи по умолчанию** (`default`). Другие учетные записи должны использовать значения конфигурации.

## Режимы чата

Mattermost автоматически отвечает на личные сообщения. Поведение в каналах контролируется параметром `chatmode`:

-   `oncall` (по умолчанию): отвечать только при упоминании через @ в каналах.
-   `onmessage`: отвечать на каждое сообщение в канале.
-   `onchar`: отвечать, когда сообщение начинается с префикса-триггера.

Пример конфигурации:

```json
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

Примечания:

-   `onchar` также реагирует на явные упоминания через @.
-   `channels.mattermost.requireMention` учитывается для устаревших конфигураций, но предпочтительнее использовать `chatmode`.

## Контроль доступа (личные сообщения)

-   По умолчанию: `channels.mattermost.dmPolicy = "pairing"` (неизвестным отправителям выдается код сопряжения).
-   Подтверждение через:
    -   `openclaw pairing list mattermost`
    -   `openclaw pairing approve mattermost `
-   Открытые личные сообщения: `channels.mattermost.dmPolicy="open"` плюс `channels.mattermost.allowFrom=["*"]`.

## Каналы (группы)

-   По умолчанию: `channels.mattermost.groupPolicy = "allowlist"` (с упоминанием через @).
-   Разрешите отправителей с помощью `channels.mattermost.groupAllowFrom` (рекомендуются ID пользователей).
-   Сопоставление по `@username` изменчиво и включается только при `channels.mattermost.dangerouslyAllowNameMatching: true`.
-   Открытые каналы: `channels.mattermost.groupPolicy="open"` (с упоминанием через @).
-   Примечание для времени выполнения: если `channels.mattermost` полностью отсутствует, среда выполнения возвращается к `groupPolicy="allowlist"` для проверок групп (даже если установлен `channels.defaults.groupPolicy`).

## Цели для исходящей доставки

Используйте эти форматы целей с `openclaw message send` или cron/webhooks:

-   `channel:` для канала
-   `user:` для личного сообщения
-   `@username` для личного сообщения (разрешается через API Mattermost)

Простые ID рассматриваются как каналы.

## Реакции (инструмент сообщений)

-   Используйте `message action=react` с `channel=mattermost`.
-   `messageId` — это ID поста Mattermost.
-   `emoji` принимает названия, такие как `thumbsup` или `:+1:` (двоеточия необязательны).
-   Установите `remove=true` (логическое значение), чтобы удалить реакцию.
-   События добавления/удаления реакции пересылаются как системные события в маршрутизированный сеанс агента.

Примеры:

```bash
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup remove=true
```

Конфигурация:

-   `channels.mattermost.actions.reactions`: включить/отключить действия с реакциями (по умолчанию true).
-   Переопределение для учетной записи: `channels.mattermost.accounts..actions.reactions`.

## Интерактивные кнопки (инструмент сообщений)

Отправляйте сообщения с кликабельными кнопками. Когда пользователь нажимает кнопку, агент получает выбор и может ответить. Включите кнопки, добавив `inlineButtons` в возможности канала:

```json
{
  channels: {
    mattermost: {
      capabilities: ["inlineButtons"],
    },
  },
}
```

Используйте `message action=send` с параметром `buttons`. Кнопки представляют собой двумерный массив (ряды кнопок):

```bash
message action=send channel=mattermost target=channel:<channelId> buttons=[[{"text":"Да","callback_data":"yes"},{"text":"Нет","callback_data":"no"}]]
```

Поля кнопки:

-   `text` (обязательно): отображаемая метка.
-   `callback_data` (обязательно): значение, отправляемое обратно при нажатии (используется как ID действия).
-   `style` (опционально): `"default"`, `"primary"` или `"danger"`.

Когда пользователь нажимает кнопку:

1.  Все кнопки заменяются строкой подтверждения (например, ”✓ **Да** выбрано пользователем @user”).
2.  Агент получает выбор как входящее сообщение и отвечает.

Примечания:

-   Обратные вызовы кнопок используют проверку HMAC-SHA256 (автоматически, без дополнительной настройки).
-   Mattermost удаляет данные обратного вызова из ответов своего API (функция безопасности), поэтому все кнопки удаляются при нажатии — частичное удаление невозможно.
-   ID действий, содержащие дефисы или подчеркивания, автоматически очищаются (ограничение маршрутизации Mattermost).

Конфигурация:

-   `channels.mattermost.capabilities`: массив строк возможностей. Добавьте `"inlineButtons"`, чтобы включить описание инструмента кнопок в системном промпте агента.
-   `channels.mattermost.interactions.callbackBaseUrl`: опциональный внешний базовый URL для обратных вызовов кнопок (например, `https://gateway.example.com`). Используйте его, когда Mattermost не может напрямую обратиться к хосту привязки шлюза.
-   В настройках с несколькими учетными записями вы также можете установить это поле в `channels.mattermost.accounts..interactions.callbackBaseUrl`.
-   Если `interactions.callbackBaseUrl` не указан, OpenClaw формирует URL обратного вызова из `gateway.customBindHost` + `gateway.port`, затем возвращается к `http://localhost:`.
-   Правило доступности: URL обратного вызова кнопок должен быть доступен с сервера Mattermost. `localhost` работает только тогда, когда Mattermost и OpenClaw работают на одном хосте/сетевом пространстве имен.
-   Если ваша цель обратного вызова является приватной/Tailnet/внутренней, добавьте ее хост/домен в Mattermost `ServiceSettings.AllowedUntrustedInternalConnections`.

### Прямая интеграция с API (внешние скрипты)

Внешние скрипты и вебхуки могут публиковать кнопки напрямую через REST API Mattermost, вместо того чтобы использовать инструмент `message` агента. По возможности используйте `buildButtonAttachments()` из расширения; если публикуете сырой JSON, следуйте этим правилам: **Структура полезной нагрузки:**

```json
{
  channel_id: "<channelId>",
  message: "Выберите вариант:",
  props: {
    attachments: [
      {
        actions: [
          {
            id: "mybutton01", // только буквенно-цифровые символы — см. ниже
            type: "button", // обязательно, иначе нажатия игнорируются
            name: "Подтвердить", // отображаемая метка
            style: "primary", // опционально: "default", "primary", "danger"
            integration: {
              url: "https://gateway.example.com/mattermost/interactions/default",
              context: {
                action_id: "mybutton01", // должно совпадать с id кнопки (для поиска имени)
                action: "approve",
                // ... любые пользовательские поля ...
                _token: "<hmac>", // см. раздел HMAC ниже
              },
            },
          },
        ],
      },
    ],
  },
}
```

**Критические правила:**

1.  Вложения помещаются в `props.attachments`, а не в `attachments` верхнего уровня (тихо игнорируются).
2.  Каждому действию нужен `type: "button"` — без него нажатия тихо игнорируются.
3.  Каждому действию нужно поле `id` — Mattermost игнорирует действия без ID.
4.  `id` действия должен содержать **только буквенно-цифровые символы** (`[a-zA-Z0-9]`). Дефисы и подчеркивания нарушают серверную маршрутизацию действий Mattermost (возвращает 404). Удаляйте их перед использованием.
5.  `context.action_id` должен совпадать с `id` кнопки, чтобы в сообщении подтверждения отображалось имя кнопки (например, "Подтвердить"), а не сырой ID.
6.  `context.action_id` обязателен — обработчик взаимодействий вернет 400 без него.

**Генерация токена HMAC:** Шлюз проверяет нажатия кнопок с помощью HMAC-SHA256. Внешние скрипты должны генерировать токены, соответствующие логике проверки шлюза:

1.  Получите секрет из токена бота: `HMAC-SHA256(key="openclaw-mattermost-interactions", data=botToken)`
2.  Постройте объект контекста со всеми полями **кроме** `_token`.
3.  Сериализуйте с **отсортированными ключами** и **без пробелов** (шлюз использует `JSON.stringify` с отсортированными ключами, что дает компактный вывод).
4.  Подпишите: `HMAC-SHA256(key=secret, data=serializedContext)`
5.  Добавьте полученный хеш в шестнадцатеричном формате как `_token` в контекст.

Пример на Python:

```typescript
import hmac, hashlib, json

secret = hmac.new(
    b"openclaw-mattermost-interactions",
    bot_token.encode(), hashlib.sha256
).hexdigest()

ctx = {"action_id": "mybutton01", "action": "approve"}
payload = json.dumps(ctx, sort_keys=True, separators=(",", ":"))
token = hmac.new(secret.encode(), payload.encode(), hashlib.sha256).hexdigest()

context = {**ctx, "_token": token}
```

Распространенные проблемы с HMAC:

-   `json.dumps` в Python по умолчанию добавляет пробелы (`{"key": "val"}`). Используйте `separators=(",", ":")`, чтобы соответствовать компактному выводу JavaScript (`{"key":"val"}`).
-   Всегда подписывайте **все** поля контекста (кроме `_token`). Шлюз удаляет `_token`, а затем подписывает все остальное. Подпись подмножества приводит к тихому сбою проверки.
-   Используйте `sort_keys=True` — шлюз сортирует ключи перед подписанием, а Mattermost может переупорядочить поля контекста при сохранении полезной нагрузки.
-   Получайте секрет из токена бота (детерминированно), а не из случайных байтов. Секрет должен быть одинаковым в процессе создания кнопок и в шлюзе, который проверяет.

## Адаптер каталога

Плагин Mattermost включает адаптер каталога, который разрешает имена каналов и пользователей через API Mattermost. Это позволяет использовать цели `#channel-name` и `@username` в `openclaw message send` и доставках через cron/webhook. Дополнительная настройка не требуется — адаптер использует токен бота из конфигурации учетной записи.

## Несколько учетных записей

Mattermost поддерживает несколько учетных записей в `channels.mattermost.accounts`:

```json
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Основная", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Оповещения", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## Устранение неполадок

-   Нет ответов в каналах: убедитесь, что бот находится в канале и упомяните его (oncall), используйте префикс-триггер (onchar) или установите `chatmode: "onmessage"`.
-   Ошибки аутентификации: проверьте токен бота, базовый URL и включена ли учетная запись.
-   Проблемы с несколькими учетными записями: переменные окружения применяются только к учетной записи `default`.
-   Кнопки отображаются как белые рамки: возможно, агент отправляет некорректные данные кнопок. Убедитесь, что у каждой кнопки есть поля `text` и `callback_data`.
-   Кнопки отображаются, но нажатия не работают: убедитесь, что `AllowedUntrustedInternalConnections` в конфигурации сервера Mattermost включает `127.0.0.1 localhost`, и что `EnablePostActionIntegration` установлено в `true` в ServiceSettings.
-   При нажатии кнопки возвращается 404: вероятно, `id` кнопки содержит дефисы или подчеркивания. Маршрутизатор действий Mattermost ломается на небуквенно-цифровых ID. Используйте только `[a-zA-Z0-9]`.
-   В журналах шлюза `invalid _token`: несоответствие HMAC. Проверьте, что вы подписываете все поля контекста (а не подмножество), используете отсортированные ключи и компактный JSON (без пробелов). См. раздел HMAC выше.
-   В журналах шлюза `missing _token in context`: поле `_token` отсутствует в контексте кнопки. Убедитесь, что оно включено при построении полезной нагрузки интеграции.
-   В подтверждении отображается сырой ID вместо имени кнопки: `context.action_id` не совпадает с `id` кнопки. Установите оба значения одинаковыми и очищенными.
-   Агент не знает о кнопках: добавьте `capabilities: ["inlineButtons"]` в конфигурацию канала Mattermost.

[Matrix](./matrix.md)[Microsoft Teams](./msteams.md)