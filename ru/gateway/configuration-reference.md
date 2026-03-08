title: "Справочник по конфигурации OpenClaw Gateway и руководство по настройкам"
description: "Полный справочник по всем полям конфигурации OpenClaw Gateway. Научитесь настраивать каналы, политики личных сообщений, переопределения моделей, а также конфигурировать WhatsApp, Telegram и Discord."
keywords: ["конфигурация openclaw", "настройка шлюза", "конфигурация каналов", "политика личных сообщений", "переопределения моделей", "бот whatsapp", "бот telegram", "бот discord"]
---

  Конфигурация и операции

  
# Справочник по конфигурации

Все поля, доступные в `~/.openclaw/openclaw.json`. Для обзора, ориентированного на задачи, см. [Конфигурация](./configuration.md). Формат конфигурации — **JSON5** (разрешены комментарии и завершающие запятые). Все поля необязательны — OpenClaw использует безопасные значения по умолчанию, если они опущены.

* * *

## Каналы

Каждый канал запускается автоматически, когда существует его раздел конфигурации (если не указано `enabled: false`).

### Доступ в ЛС и группах

Все каналы поддерживают политики для ЛС и групп:

| Политика ЛС | Поведение |
| --- | --- |
| `pairing` (по умолчанию) | Неизвестным отправителям выдается одноразовый код сопряжения; владелец должен подтвердить |
| `allowlist` | Только отправители из `allowFrom` (или из хранилища разрешенных контактов) |
| `open` | Разрешить все входящие ЛС (требует `allowFrom: ["*"]`) |
| `disabled` | Игнорировать все входящие ЛС |

| Политика групп | Поведение |
| --- | --- |
| `allowlist` (по умолчанию) | Только группы, соответствующие настроенному списку разрешений |
| `open` | Обойти списки разрешений групп (упоминание все еще применяется) |
| `disabled` | Блокировать все сообщения из групп/комнат |

> **ℹ️** `channels.defaults.groupPolicy` устанавливает значение по умолчанию, когда `groupPolicy` провайдера не задан. Коды сопряжения истекают через 1 час. Ожидающие запросы на сопряжение в ЛС ограничены **3 на канал**. Если блок провайдера полностью отсутствует (`channels.` отсутствует), политика групп во время выполнения возвращается к `allowlist` (закрытая при сбое) с предупреждением при запуске.

### Переопределения моделей для каналов

Используйте `channels.modelByChannel`, чтобы закрепить конкретные идентификаторы каналов за моделью. Значения принимают `provider/model` или настроенные псевдонимы моделей. Сопоставление канала применяется, когда у сессии еще нет переопределения модели (например, установленного через `/model`).

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

### Значения по умолчанию для каналов и heartbeat

Используйте `channels.defaults` для общего поведения политики групп и heartbeat между провайдерами:

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

-   `channels.defaults.groupPolicy`: запасная политика групп, когда `groupPolicy` на уровне провайдера не задан.
-   `channels.defaults.heartbeat.showOk`: включать статусы здоровых каналов в вывод heartbeat.
-   `channels.defaults.heartbeat.showAlerts`: включать статусы с ухудшением/ошибками в вывод heartbeat.
-   `channels.defaults.heartbeat.useIndicator`: отображать компактный вывод heartbeat в стиле индикатора.

### WhatsApp

WhatsApp работает через веб-канал шлюза (Baileys Web). Он запускается автоматически, когда существует связанная сессия.

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // синие галочки (false в режиме self-chat)
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

-   Исходящие команды по умолчанию используют аккаунт `default`, если он присутствует; иначе первый настроенный идентификатор аккаунта (отсортированный).
-   Необязательный `channels.whatsapp.defaultAccount` переопределяет этот запасной выбор аккаунта по умолчанию, когда он соответствует настроенному идентификатору аккаунта.
-   Устаревший каталог аутентификации Baileys для одного аккаунта мигрируется с помощью `openclaw doctor` в `whatsapp/default`.
-   Переопределения на аккаунт: `channels.whatsapp.accounts..sendReadReceipts`, `channels.whatsapp.accounts..dmPolicy`, `channels.whatsapp.accounts..allowFrom`.

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
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streaming: "partial", // off | partial | block | progress (default: off)
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

-   Токен бота: `channels.telegram.botToken` или `channels.telegram.tokenFile`, с `TELEGRAM_BOT_TOKEN` в качестве запасного варианта для аккаунта по умолчанию.
-   Необязательный `channels.telegram.defaultAccount` переопределяет выбор аккаунта по умолчанию, когда он соответствует настроенному идентификатору аккаунта.
-   В настройках с несколькими аккаунтами (2+ идентификатора аккаунта) установите явный аккаунт по умолчанию (`channels.telegram.defaultAccount` или `channels.telegram.accounts.default`), чтобы избежать запасной маршрутизации; `openclaw doctor` предупреждает, когда это отсутствует или недействительно.
-   `configWrites: false` блокирует запись конфигурации, инициированную Telegram (миграции ID супергрупп, `/config set|unset`).
-   Записи верхнего уровня `bindings[]` с `type: "acp"` настраивают постоянные привязки ACP для тем форума (используйте канонический `chatId:topic:topicId` в `match.peer.id`). Семантика полей описана в [ACP Agents](../tools/acp-agents.md#channel-specific-settings).
-   Превью потоков Telegram используют `sendMessage` + `editMessageText` (работает в личных и групповых чатах).
-   Политика повторных попыток: см. [Политика повторных попыток](../concepts/retry.md).

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
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      streaming: "off", // off | partial | block | progress (progress соответствует partial на Discord)
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
        spawnSubagentSessions: false, // опция для sessions_spawn({ thread: true })
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

-   Токен: `channels.discord.token`, с `DISCORD_BOT_TOKEN` в качестве запасного варианта для аккаунта по умолчанию.
-   Необязательный `channels.discord.defaultAccount` переопределяет выбор аккаунта по умолчанию, когда он соответствует настроенному идентификатору аккаунта.
-   Используйте `user:` (ЛС) или `channel:` (канал гильдии) для целей доставки; голые числовые ID отклоняются.
-   Слаг гильдии в нижнем регистре с заменой пробелов на `-`; ключи каналов используют имя в виде слага (без `#`). Предпочитайте ID гильдий.
-   Сообщения, созданные ботом, по умолчанию игнорируются. `allowBots: true` включает их; используйте `allowBots: "mentions"`, чтобы принимать только сообщения от ботов, которые упоминают бота (собственные сообщения все еще фильтруются).
-   `channels.discord.guilds..ignoreOtherMentions` (и переопределения каналов) отбрасывает сообщения, которые упоминают другого пользователя или роль, но не бота (исключая @everyone/@here).
-   `maxLinesPerMessage` (по умолчанию 17) разделяет длинные сообщения, даже если они меньше 2000 символов.
-   `channels.discord.threadBindings` управляет привязкой маршрутизации к тредам Discord:
    -   `enabled`: переопределение Discord для функций сессий, привязанных к тредам (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`, и привязанная доставка/маршрутизация)
    -   `idleHours`: переопределение Discord для автоотмены фокуса при бездействии в часах (`0` отключает)
    -   `maxAgeHours`: переопределение Discord для жесткого максимального возраста в часах (`0` отключает)
    -   `spawnSubagentSessions`: опция для автоматического создания/привязки треда в `sessions_spawn({ thread: true })`
-   Записи верхнего уровня `bindings[]` с `type: "acp"` настраивают постоянные привязки ACP для каналов и тредов (используйте ID канала/треда в `match.peer.id`). Семантика полей описана в [ACP Agents](../tools/acp-agents.md#channel-specific-settings).
-   `channels.discord.ui.components.accentColor` устанавливает акцентный цвет для контейнеров компонентов Discord v2.
-   `channels.discord.voice` включает разговоры в голосовых каналах Discord и опциональное авто-подключение + переопределения TTS.
-   `channels.discord.voice.daveEncryption` и `channels.discord.voice.decryptionFailureTolerance` передаются в опции DAVE `@discordjs/voice` (`true` и `24` по умолчанию).
-   OpenClaw дополнительно пытается восстановить прием голоса, покидая/возвращаясь в голосовую сессию после повторных сбоев расшифровки.
-   `channels.discord.streaming` — это канонический ключ режима потоковой передачи. Устаревшие значения `streamMode` и булево `streaming` автоматически мигрируются.
-   `channels.discord.autoPresence` сопоставляет доступность во время выполнения с присутствием бота (healthy => online, degraded => idle, exhausted => dnd) и позволяет опционально переопределять текст статуса.
-   `channels.discord.dangerouslyAllowNameMatching` повторно включает сопоставление по изменяемому имени/тегу (режим совместимости на случай аварии).

**Режимы уведомлений о реакциях:** `off` (нет), `own` (сообщения бота, по умолчанию), `all` (все сообщения), `allowlist` (от `guilds..users` на всех сообщениях).

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

-   JSON сервисного аккаунта: встроенный (`serviceAccount`) или на основе файла (`serviceAccountFile`).
-   Также поддерживается SecretRef сервисного аккаунта (`serviceAccountRef`).
-   Запасные варианты из env: `GOOGLE_CHAT_SERVICE_ACCOUNT` или `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
-   Используйте `spaces/` или `users/` для целей доставки.
-   `channels.googlechat.dangerouslyAllowNameMatching` повторно включает сопоставление по изменяемому email-принципалу (режим совместимости на случай аварии).

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
          systemPrompt: "Short answers only.",
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
      streaming: "partial", // off | partial | block | progress (preview mode)
      nativeStreaming: true, // использовать нативный API потоковой передачи Slack, когда streaming=partial
      mediaMaxMb: 20,
    },
  },
}
```

-   **Режим сокета** требует и `botToken`, и `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` для запасного варианта env аккаунта по умолчанию).
-   **HTTP-режим** требует `botToken` плюс `signingSecret` (на корневом уровне или на аккаунт).
-   `configWrites: false` блокирует запись конфигурации, инициированную Slack.
-   Необязательный `channels.slack.defaultAccount` переопределяет выбор аккаунта по умолчанию, когда он соответствует настроенному идентификатору аккаунта.
-   `channels.slack.streaming` — это канонический ключ режима потоковой передачи. Устаревшие значения `streamMode` и булево `streaming` автоматически мигрируются.
-   Используйте `user:` (ЛС) или `channel:` для целей доставки.

**Режимы уведомлений о реакциях:** `off`, `own` (по умолчанию), `all`, `allowlist` (из `reactionAllowlist`). **Изоляция сессий в тредах:** `thread.historyScope` — на тред (по умолчанию) или общий для канала. `thread.inheritParent` копирует транскрипт родительского канала в новые треды.

-   `typingReaction` добавляет временную реакцию к входящему сообщению Slack, пока выполняется ответ, затем удаляет ее по завершении. Используйте короткий код эмодзи Slack, например `"hourglass_flowing_sand"`.

| Группа действий | По умолчанию | Примечания |
| --- | --- | --- |
| reactions | enabled | Реакции + список реакций |
| messages | enabled | Чтение/отправка/редактирование/удаление |
| pins | enabled | Закрепление/открепление/список |
| memberInfo | enabled | Информация об участнике |
| emojiList | enabled | Список пользовательских эмодзи |

### Mattermost

Mattermost поставляется как плагин: `openclaw plugins install @openclaw/mattermost`.

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
        native: true, // опция
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // Опциональный явный URL для обратного прокси/публичных развертываний
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

Режимы чата: `oncall` (отвечать на @-упоминание, по умолчанию), `onmessage` (каждое сообщение), `onchar` (сообщения, начинающиеся с префикса триггера). Когда включены нативные команды Mattermost:

-   `commands.callbackPath` должен быть путем (например, `/api/channels/mattermost/command`), а не полным URL.
-   `commands.callbackUrl` должен разрешаться в конечную точку шлюза OpenClaw и быть доступным с сервера Mattermost.
-   Для приватных/tailnet/внутренних хостов обратного вызова Mattermost может требовать, чтобы `ServiceSettings.AllowedUntrustedInternalConnections` включал хост/домен обратного вызова. Используйте значения хоста/домена, а не полные URL.
-   `channels.mattermost.configWrites`: разрешать или запрещать запись конфигурации, инициированную Mattermost.
-   `channels.mattermost.requireMention`: требовать `@упоминание` перед ответом в каналах.
-   Необязательный `channels.mattermost.defaultAccount` переопределяет выбор аккаунта по умолчанию, когда он соответствует настроенному идентификатору аккаунта.

### Signal

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15555550123", // опциональная привязка аккаунта
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

**Режимы уведомлений о реакциях:** `off`, `own` (по умолчанию), `all`, `allowlist` (из `reactionAllowlist`).

-   `channels.signal.account`: привязывает запуск канала к конкретной идентичности аккаунта Signal.
-   `channels.signal.configWrites`: разрешать или запрещать запись конфигурации, инициированную Signal.
-   Необязательный `channels.signal.defaultAccount` переопределяет выбор аккаунта по умолчанию, когда он соответствует настроенному идентификатору аккаунта.

### BlueBubbles

BlueBubbles — рекомендуемый путь для iMessage (поддерживается плагином, настраивается в `channels.bluebubbles`).

```json
{
  channels: {
    bluebubbles: {
      enabled: true,
      dmPolicy: "pairing",
      // serverUrl, password, webhookPath, group controls, and advanced actions:
      // see /channels/bluebubbles
    },
  },
}
```

-   Основные пути ключей, описанные здесь: `channels.bluebubbles`, `channels.bluebubbles.dmPolicy`.
-   Необязательный `channels.bluebubbles.defaultAccount` переопределяет выбор аккаунта по умолчанию, когда он соответствует настроенному идентификатору аккаунта.
-   Полная конфигурация канала BlueBubbles документирована в [BlueBubbles](../channels/bluebubbles.md).

### iMessage

OpenClaw запускает `imsg rpc` (JSON-RPC через stdio). Демон или порт не требуются.

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

-   Необязательный `channels.imessage.defaultAccount` переопределяет выбор аккаунта по умолчанию, когда он соответствует настроенному идентификатору аккаунта.
-   Требуется полный доступ к диску для базы данных Messages.
-   Предпочитайте цели `chat_id:`. Используйте `imsg chats --limit 20` для списка чатов.
-   `cliPath` может указывать на SSH-обертку; установите `remoteHost` (`host` или `user@host`) для получения вложений через SCP.
-   `attachmentRoots` и `remoteAttachmentRoots` ограничивают пути входящих вложений (по умолчанию: `/Users/*/Library/Messages/Attachments`).
-   SCP использует строгую проверку ключа хоста, поэтому убедитесь, что ключ хоста ретранслятора уже существует в `~/.ssh/known_hosts`.
-   `channels.imessage.configWrites`: разрешать или запрещать запись конфигурации, инициированную iMessage.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### Microsoft Teams

Microsoft Teams поддерживается расширением и настраивается в `channels.msteams`.

```json
{
  channels: {
    msteams: {
      enabled: true,
      configWrites: true,
      // appId, appPassword, tenantId, webhook, team/channel policies:
      // see /channels/msteams
    },
  },
}
```

-   Основные пути ключей, описанные здесь: `channels.msteams`, `channels.msteams.configWrites`.
-   Полная конфигурация Teams (учетные данные, вебхук, политики ЛС/групп, переопределения на команду/канал) документирована в [Microsoft Teams](../channels/msteams.md).

### IRC

IRC поддерживается расширением и настраивается в `channels.irc`.

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

-   Основные пути ключей, описанные здесь: `channels.irc`, `channels.irc.dmPolicy`, `channels.irc.configWrites`, `channels.irc.nickserv.*`.
-   Необязательный `channels.irc.defaultAccount` переопределяет выбор аккаунта по умолчанию, когда он соответствует настроенному идентификатору аккаунта.
-   Полная конфигурация канала IRC (хост/порт/TLS/каналы/списки разрешений/упоминания) документирована в [IRC](../channels/irc.md).

### Мультиаккаунт (все каналы)

Запуск нескольких аккаунтов на канал (каждый со своим `accountId`):

```json
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

-   `default` используется, когда `accountId` опущен (CLI + маршрутизация).
-   Токены из env применяются только к аккаунту **по умолчанию**.
-   Базовые настройки канала применяются ко всем аккаунтам, если не переопределены на аккаунт.
-   Используйте `bindings[].match.accountId` для маршрутизации каждого аккаунта к разному агенту.
-   Если вы добавляете нестандартный аккаунт через `openclaw channels add` (или онбординг канала), пока все еще используете конфигурацию канала верхнего уровня для одного аккаунта, OpenClaw сначала перемещает значения верхнего уровня, привязанные к аккаунту, в `channels..accounts.default`, чтобы исходный аккаунт продолжал работать.
-   Существующие привязки только к каналу (без `accountId`) продолжают соответствовать аккаунту по умолчанию; привязки, привязанные к аккаунту, остаются опциональными.
-   `openclaw doctor --fix` также исправляет смешанные структуры, перемещая значения верхнего уровня, привязанные к аккаунту, в `accounts.default`, когда именованные аккаунты существуют, но `default` отсутствует.

### Другие каналы расширений

Многие каналы расширений настраиваются как `channels.` и документированы на своих страницах (например, Feishu, Matrix, LINE, Nostr, Zalo, Nextcloud Talk, Synology Chat и Twitch). См. полный индекс каналов: [Каналы](../channels.md).

### Управление упоминаниями в групповых чатах

Сообщения в группах по умолчанию **требуют упоминания** (упоминание в метаданных или шаблоны regex). Применяется к групповым чатам WhatsApp, Telegram, Discord, Google Chat и iMessage. **Типы упоминаний:**

-   **Упоминания в метаданных**: Нативные @-упоминания на платформе. Игнорируются в режиме self-chat WhatsApp.
-   **Текстовые шаблоны**: Шаблоны regex в `agents.list[].groupChat.mentionPatterns`. Проверяются всегда.
-   Управление упоминаниями применяется только тогда, когда обнаружение возможно (нативные упоминания или хотя бы один шаблон).

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

`messages.groupChat.historyLimit` устанавливает глобальное значение по умолчанию. Каналы могут переопределять с помощью `channels..historyLimit` (или на аккаунт). Установите `0` для отключения.

#### Ограничения истории ЛС

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

Приоритет: переопределение на ЛС → значение по умолчанию провайдера → без ограничений (все сохраняется). Поддерживается: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Режим self-chat

Включите свой собственный номер в `