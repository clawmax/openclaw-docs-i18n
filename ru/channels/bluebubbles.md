

  Платформы обмена сообщениями

  
# BlueBubbles

Статус: встроенный плагин, который взаимодействует с сервером BlueBubbles для macOS по HTTP. **Рекомендуется для интеграции iMessage** из-за более богатого API и более простой настройки по сравнению с устаревшим каналом imsg.

## Обзор

-   Работает на macOS через вспомогательное приложение BlueBubbles ([bluebubbles.app](https://bluebubbles.app)).
-   Рекомендуется/протестировано: macOS Sequoia (15). macOS Tahoe (26) работает; редактирование в настоящее время не работает на Tahoe, а обновления значков групп могут сообщать об успехе, но не синхронизироваться.
-   OpenClaw взаимодействует с ним через его REST API (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
-   Входящие сообщения поступают через вебхуки; исходящие ответы, индикаторы набора, квитанции о прочтении и тапбеки — это REST-вызовы.
-   Вложения и стикеры принимаются как входящие медиа (и, по возможности, передаются агенту).
-   Сопряжение/белый список работают так же, как и в других каналах (`/channels/pairing` и т.д.) с `channels.bluebubbles.allowFrom` + кодами сопряжения.
-   Реакции отображаются как системные события, как в Slack/Telegram, чтобы агенты могли «упоминать» их перед ответом.
-   Расширенные функции: редактирование, отмена отправки, ветки ответов, эффекты сообщений, управление группами.

## Быстрый старт

1.  Установите сервер BlueBubbles на свой Mac (следуйте инструкциям на [bluebubbles.app/install](https://bluebubbles.app/install)).
2.  В конфигурации BlueBubbles включите веб-API и установите пароль.
3.  Запустите `openclaw onboard` и выберите BlueBubbles, или настройте вручную:
    
    Копировать
    
    ```json
    {
      channels: {
        bluebubbles: {
          enabled: true,
          serverUrl: "http://192.168.1.100:1234",
          password: "example-password",
          webhookPath: "/bluebubbles-webhook",
        },
      },
    }
    ```
    
4.  Направьте вебхуки BlueBubbles на ваш шлюз (пример: `https://your-gateway-host:3000/bluebubbles-webhook?password=`).
5.  Запустите шлюз; он зарегистрирует обработчик вебхуков и начнет сопряжение.

Примечание по безопасности:

-   Всегда устанавливайте пароль для вебхука.
-   Аутентификация вебхука всегда требуется. OpenClaw отклоняет запросы вебхуков от BlueBubbles, если они не содержат пароль/guid, соответствующий `channels.bluebubbles.password` (например, `?password=` или `x-password`), независимо от топологии петли/прокси.
-   Проверка аутентификации по паролю выполняется до чтения/разбора полных тел вебхуков.

## Поддержание активности Messages.app (настройки VM / безголовые)

Некоторые настройки macOS VM / постоянно работающих систем могут привести к тому, что Messages.app переходит в «режим простоя» (входящие события прекращаются, пока приложение не будет открыто/выведено на передний план). Простое решение — **«тыкать» Messages каждые 5 минут** с помощью AppleScript + LaunchAgent.

### 1) Сохраните AppleScript

Сохраните это как:

-   `~/Scripts/poke-messages.scpt`

Пример скрипта (неинтерактивный; не перехватывает фокус):

```
try
  tell application "Messages"
    if not running then
      launch
    end if

    -- Обращение к интерфейсу скриптов для поддержания отзывчивости процесса.
    set _chatCount to (count of chats)
  end tell
on error
  -- Игнорировать временные сбои (запросы при первом запуске, заблокированная сессия и т.д.).
end try
```

### 2) Установите LaunchAgent

Сохраните это как:

-   `~/Library/LaunchAgents/com.user.poke-messages.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>

    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript &quot;$HOME/Scripts/poke-messages.scpt&quot;</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>StartInterval</key>
    <integer>300</integer>

    <key>StandardOutPath</key>
    <string>/tmp/poke-messages.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/poke-messages.err</string>
  </dict>
</plist>
```

Примечания:

-   Это выполняется **каждые 300 секунд** и **при входе в систему**.
-   Первый запуск может вызвать запросы **Автоматизации** macOS (`osascript` → Messages). Одобрите их в той же пользовательской сессии, в которой запущен LaunchAgent.

Загрузите его:

```bash
launchctl unload ~/Library/LaunchAgents/com.user.poke-messages.plist 2>/dev/null || true
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

## Онбординг

BlueBubbles доступен в интерактивном мастере настройки:

```bash
openclaw onboard
```

Мастер запрашивает:

-   **URL сервера** (обязательно): Адрес сервера BlueBubbles (например, `http://192.168.1.100:1234`)
-   **Пароль** (обязательно): Пароль API из настроек сервера BlueBubbles
-   **Путь вебхука** (опционально): По умолчанию `/bluebubbles-webhook`
-   **Политика ЛС**: сопряжение, белый список, открытый или отключенный
-   **Белый список**: Номера телефонов, email или цели чатов

Вы также можете добавить BlueBubbles через CLI:

```bash
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## Контроль доступа (ЛС + группы)

ЛС:

-   По умолчанию: `channels.bluebubbles.dmPolicy = "pairing"`.
-   Неизвестные отправители получают код сопряжения; сообщения игнорируются до одобрения (коды истекают через 1 час).
-   Одобрить через:
    -   `openclaw pairing list bluebubbles`
    -   `openclaw pairing approve bluebubbles `
-   Сопряжение — это стандартный обмен токенами. Подробности: [Сопряжение](./pairing.md)

Группы:

-   `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (по умолчанию: `allowlist`).
-   `channels.bluebubbles.groupAllowFrom` определяет, кто может активировать бота в группах, когда установлен `allowlist`.

### Управление упоминаниями (группы)

BlueBubbles поддерживает управление упоминаниями для групповых чатов, соответствующее поведению iMessage/WhatsApp:

-   Использует `agents.list[].groupChat.mentionPatterns` (или `messages.groupChat.mentionPatterns`) для обнаружения упоминаний.
-   Когда для группы включен `requireMention`, агент отвечает только при упоминании.
-   Команды управления от авторизованных отправителей обходят управление упоминаниями.

Конфигурация для каждой группы:

```json
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }, // значение по умолчанию для всех групп
        "iMessage;-;chat123": { requireMention: false }, // переопределение для конкретной группы
      },
    },
  },
}
```

### Управление командами

-   Команды управления (например, `/config`, `/model`) требуют авторизации.
-   Использует `allowFrom` и `groupAllowFrom` для определения авторизации команд.
-   Авторизованные отправители могут выполнять команды управления даже без упоминания в группах.

## Индикатор набора + квитанции о прочтении

-   **Индикаторы набора**: Отправляются автоматически перед и во время генерации ответа.
-   **Квитанции о прочтении**: Управляются параметром `channels.bluebubbles.sendReadReceipts` (по умолчанию: `true`).
-   **Индикаторы набора**: OpenClaw отправляет события начала набора; BlueBubbles автоматически сбрасывает индикатор при отправке или таймауте (ручная остановка через DELETE ненадежна).

```json
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false, // отключить квитанции о прочтении
    },
  },
}
```

## Расширенные действия

BlueBubbles поддерживает расширенные действия с сообщениями при включении в конфигурации:

```json
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true, // тапбеки (по умолчанию: true)
        edit: true, // редактировать отправленные сообщения (macOS 13+, не работает на macOS 26 Tahoe)
        unsend: true, // отменить отправку сообщений (macOS 13+)
        reply: true, // ветки ответов по GUID сообщения
        sendWithEffect: true, // эффекты сообщений (slam, loud и т.д.)
        renameGroup: true, // переименовать групповые чаты
        setGroupIcon: true, // установить значок/фото группового чата (нестабильно на macOS 26 Tahoe)
        addParticipant: true, // добавить участников в группы
        removeParticipant: true, // удалить участников из групп
        leaveGroup: true, // покинуть групповые чаты
        sendAttachment: true, // отправлять вложения/медиа
      },
    },
  },
}
```

Доступные действия:

-   **react**: Добавить/удалить реакции тапбека (`messageId`, `emoji`, `remove`)
-   **edit**: Отредактировать отправленное сообщение (`messageId`, `text`)
-   **unsend**: Отменить отправку сообщения (`messageId`)
-   **reply**: Ответить на конкретное сообщение (`messageId`, `text`, `to`)
-   **sendWithEffect**: Отправить с эффектом iMessage (`text`, `to`, `effectId`)
-   **renameGroup**: Переименовать групповой чат (`chatGuid`, `displayName`)
-   **setGroupIcon**: Установить значок/фото группового чата (`chatGuid`, `media`) — нестабильно на macOS 26 (Tahoe) (API может возвращать успех, но значок не синхронизируется).
-   **addParticipant**: Добавить кого-либо в группу (`chatGuid`, `address`)
-   **removeParticipant**: Удалить кого-либо из группы (`chatGuid`, `address`)
-   **leaveGroup**: Покинуть групповой чат (`chatGuid`)
-   **sendAttachment**: Отправить медиа/файлы (`to`, `buffer`, `filename`, `asVoice`)
    -   Голосовые заметки: установите `asVoice: true` с аудио **MP3** или **CAF**, чтобы отправить как голосовое сообщение iMessage. BlueBubbles конвертирует MP3 → CAF при отправке голосовых заметок.

### ID сообщений (короткие vs полные)

OpenClaw может отображать *короткие* ID сообщений (например, `1`, `2`) для экономии токенов.

-   `MessageSid` / `ReplyToId` могут быть короткими ID.
-   `MessageSidFull` / `ReplyToIdFull` содержат полные ID от провайдера.
-   Короткие ID хранятся в памяти; они могут истечь при перезапуске или очистке кэша.
-   Действия принимают короткий или полный `messageId`, но короткие ID вызовут ошибку, если они больше не доступны.

Используйте полные ID для долговечных автоматизаций и хранения:

-   Шаблоны: `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
-   Контекст: `MessageSidFull` / `ReplyToIdFull` во входящих данных

См. [Конфигурация](../gateway/configuration.md) для переменных шаблонов.

## Потоковая передача блоками

Управляйте тем, отправляются ли ответы одним сообщением или потоково блоками:

```json
{
  channels: {
    bluebubbles: {
      blockStreaming: true, // включить потоковую передачу блоками (по умолчанию выключено)
    },
  },
}
```

## Медиа + ограничения

-   Входящие вложения загружаются и сохраняются в кэше медиа.
-   Ограничение на медиа через `channels.bluebubbles.mediaMaxMb` для входящих и исходящих медиа (по умолчанию: 8 МБ).
-   Исходящий текст разбивается на части по `channels.bluebubbles.textChunkLimit` (по умолчанию: 4000 символов).

## Справочник по конфигурации

Полная конфигурация: [Конфигурация](../gateway/configuration.md) Параметры провайдера:

-   `channels.bluebubbles.enabled`: Включить/отключить канал.
-   `channels.bluebubbles.serverUrl`: Базовый URL REST API BlueBubbles.
-   `channels.bluebubbles.password`: Пароль API.
-   `channels.bluebubbles.webhookPath`: Путь конечной точки вебхука (по умолчанию: `/bluebubbles-webhook`).
-   `channels.bluebubbles.dmPolicy`: `pairing | allowlist | open | disabled` (по умолчанию: `pairing`).
-   `channels.bluebubbles.allowFrom`: Белый список для ЛС (идентификаторы, email, номера E.164, `chat_id:*`, `chat_guid:*`).
-   `channels.bluebubbles.groupPolicy`: `open | allowlist | disabled` (по умолчанию: `allowlist`).
-   `channels.bluebubbles.groupAllowFrom`: Белый список отправителей для групп.
-   `channels.bluebubbles.groups`: Конфигурация для каждой группы (`requireMention` и т.д.).
-   `channels.bluebubbles.sendReadReceipts`: Отправлять квитанции о прочтении (по умолчанию: `true`).
-   `channels.bluebubbles.blockStreaming`: Включить потоковую передачу блоками (по умолчанию: `false`; требуется для потоковых ответов).
-   `channels.bluebubbles.textChunkLimit`: Размер чанка для исходящих сообщений в символах (по умолчанию: 4000).
-   `channels.bluebubbles.chunkMode`: `length` (по умолчанию) разбивает только при превышении `textChunkLimit`; `newline` разбивает по пустым строкам (границам абзацев) перед разбивкой по длине.
-   `channels.bluebubbles.mediaMaxMb`: Ограничение на входящие/исходящие медиа в МБ (по умолчанию: 8).
-   `channels.bluebubbles.mediaLocalRoots`: Явный белый список абсолютных локальных каталогов, разрешенных для исходящих локальных путей к медиа. Отправка по локальному пути запрещена по умолчанию, если это не настроено. Переопределение для каждого аккаунта: `channels.bluebubbles.accounts..mediaLocalRoots`.
-   `channels.bluebubbles.historyLimit`: Максимум сообщений группы для контекста (0 отключает).
-   `channels.bluebubbles.dmHistoryLimit`: Ограничение истории ЛС.
-   `channels.bluebubbles.actions`: Включить/отключить определенные действия.
-   `channels.bluebubbles.accounts`: Конфигурация нескольких аккаунтов.

Связанные глобальные параметры:

-   `agents.list[].groupChat.mentionPatterns` (или `messages.groupChat.mentionPatterns`).
-   `messages.responsePrefix`.

## Адресация / цели доставки

Предпочитайте `chat_guid` для стабильной маршрутизации:

-   `chat_guid:iMessage;-;+15555550123` (предпочтительно для групп)
-   `chat_id:123`
-   `chat_identifier:...`
-   Прямые идентификаторы: `+15555550123`, `user@example.com`
    -   Если для прямого идентификатора нет существующего чата ЛС, OpenClaw создаст его через `POST /api/v1/chat/new`. Для этого необходимо включить Private API BlueBubbles.

## Безопасность

-   Запросы вебхуков аутентифицируются путем сравнения параметров запроса или заголовков `guid`/`password` с `channels.bluebubbles.password`. Запросы с `localhost` также принимаются.
-   Храните пароль API и конечную точку вебхука в секрете (относитесь к ним как к учетным данным).
-   Доверие к localhost означает, что обратный прокси на том же хосте может непреднамеренно обойти пароль. Если вы проксируете шлюз, требуйте аутентификацию на прокси и настройте `gateway.trustedProxies`. См. [Безопасность шлюза](../gateway/security.md#reverse-proxy-configuration).
-   Включите HTTPS + правила брандмауэра на сервере BlueBubbles, если выставляете его за пределы вашей локальной сети.

## Устранение неполадок

-   Если события набора/прочтения перестают работать, проверьте логи вебхуков BlueBubbles и убедитесь, что путь шлюза соответствует `channels.bluebubbles.webhookPath`.
-   Коды сопряжения истекают через час; используйте `openclaw pairing list bluebubbles` и `openclaw pairing approve bluebubbles `.
-   Реакции требуют приватного API BlueBubbles (`POST /api/v1/message/react`); убедитесь, что версия сервера предоставляет его.
-   Редактирование/отмена отправки требуют macOS 13+ и совместимой версии сервера BlueBubbles. На macOS 26 (Tahoe) редактирование в настоящее время не работает из-за изменений в приватном API.
-   Обновления значков групп могут быть нестабильны на macOS 26 (Tahoe): API может возвращать успех, но новый значок не синхронизируется.
-   OpenClaw автоматически скрывает известные неработающие действия на основе версии macOS сервера BlueBubbles. Если редактирование все еще появляется на macOS 26 (Tahoe), отключите его вручную с помощью `channels.bluebubbles.actions.edit=false`.
-   Для информации о статусе/здоровье: `openclaw status --all` или `openclaw status --deep`.

Для общего справочника по рабочим процессам каналов см. [Каналы](../channels.md) и руководство по [Плагинам](../tools/plugin.md).

[Каналы чата](../channels.md)[Discord](./discord.md)