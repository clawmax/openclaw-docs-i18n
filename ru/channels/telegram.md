

  Платформы обмена сообщениями

  
# Telegram

Статус: готов к использованию в продакшене для личных сообщений бота и групп через grammY. Long polling — режим по умолчанию; режим webhook опционален.

## Быстрая настройка

### Шаг 1: Создание токена бота в BotFather

Откройте Telegram и напишите **@BotFather** (убедитесь, что это именно `@BotFather`). Выполните `/newbot`, следуйте подсказкам и сохраните токен.

### Шаг 2: Настройка токена и политики личных сообщений

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

Резервный вариант через переменные окружения: `TELEGRAM_BOT_TOKEN=...` (только для аккаунта по умолчанию). Telegram **не** использует `openclaw channels login telegram`; настройте токен в конфиге/env, затем запустите шлюз.

### Шаг 3: Запуск шлюза и подтверждение первого личного сообщения

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Коды связывания истекают через 1 час.

### Шаг 4: Добавление бота в группу

Добавьте бота в свою группу, затем настройте `channels.telegram.groups` и `groupPolicy` в соответствии с вашей моделью доступа.

 

> **ℹ️** Порядок разрешения токена учитывает аккаунт. На практике значения из конфига имеют приоритет над резервным вариантом из переменных окружения, а `TELEGRAM_BOT_TOKEN` применяется только к аккаунту по умолчанию.

## Настройки на стороне Telegram

Telegram-боты по умолчанию используют **Режим конфиденциальности**, который ограничивает, какие групповые сообщения они получают. Если бот должен видеть все групповые сообщения, необходимо:

-   отключить режим конфиденциальности через `/setprivacy`, или
-   сделать бота администратором группы.

При переключении режима конфиденциальности удалите и снова добавьте бота в каждую группу, чтобы Telegram применил изменения.

Статус администратора управляется в настройках группы Telegram. Боты-администраторы получают все групповые сообщения, что полезно для постоянного группового поведения.

-   `/setjoingroups` — разрешить/запретить добавление в группы
-   `/setprivacy` — для поведения видимости в группах

## Контроль доступа и активация

/getUpdates"', lang: 'bash' }, { label: 'Политика группы и белые списки', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "-1001234567890": {\n          groupPolicy: "open",\n          requireMention: false,\n        },\n      },\n    },\n  },\n}', lang: 'json' }, { label: 'Поведение упоминаний', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "*": { requireMention: false },\n      },\n    },\n  },\n}', lang: 'json' }]} />

## Поведение в runtime

-   Telegram управляется процессом шлюза.
-   Маршрутизация детерминирована: входящие сообщения из Telegram возвращаются в Telegram (модель не выбирает каналы).
-   Входящие сообщения нормализуются в общий конверт канала с метаданными ответа и заполнителями медиа.
-   Групповые сессии изолированы по ID группы. Темы форума добавляют `:topic:` для изоляции топиков.
-   Личные сообщения могут содержать `message_thread_id`; OpenClaw маршрутизирует их с использованием ключей сессии, учитывающих тред, и сохраняет ID треда для ответов.
-   Long polling использует grammY runner с последовательной обработкой для каждого чата/треда. Общая конкурентность sink runner использует `agents.defaults.maxConcurrent`.
-   Telegram Bot API не поддерживает квитанции о прочтении (`sendReadReceipts` не применяется).

## Справочник по функциям

OpenClaw может передавать частичные ответы в реальном времени:

-   личные чаты: нативный стриминг черновиков Telegram через `sendMessageDraft`
-   группы/топики: предварительное сообщение + `editMessageText`

Требование:

-   `channels.telegram.streaming` имеет значение `off | partial | block | progress` (по умолчанию: `partial`)
-   `progress` отображается в `partial` для Telegram (совместимость с кросс-канальным именованием)
-   устаревшие значения `channels.telegram.streamMode` и булево `streaming` автоматически преобразуются

Telegram включил `sendMessageDraft` для всех ботов в Bot API 9.5 (1 марта 2026). Для ответов только с текстом:

-   ЛС: OpenClaw обновляет черновик на месте (без дополнительного предварительного сообщения)
-   группа/топик: OpenClaw сохраняет то же предварительное сообщение и выполняет окончательное редактирование на месте (без второго сообщения)

Для сложных ответов (например, с медиа) OpenClaw возвращается к обычной окончательной доставке, а затем очищает предварительное сообщение. Предварительный стриминг отделен от блочного стриминга. Когда блочный стриминг явно включен для Telegram, OpenClaw пропускает предварительный поток, чтобы избежать двойного стриминга. Если нативный транспорт черновиков недоступен/отклонен, OpenClaw автоматически возвращается к `sendMessage` + `editMessageText`. Поток рассуждений только для Telegram:

-   `/reasoning stream` отправляет рассуждения в живой предпросмотр во время генерации
-   окончательный ответ отправляется без текста рассуждений

Исходящий текст использует Telegram `parse_mode: "HTML"`.

-   Текст в стиле Markdown преобразуется в безопасный для Telegram HTML.
-   Сырой HTML модели экранируется, чтобы уменьшить количество ошибок парсинга Telegram.
-   Если Telegram отклоняет разобранный HTML, OpenClaw повторяет попытку как обычный текст.

Предпросмотры ссылок включены по умолчанию и могут быть отключены с помощью `channels.telegram.linkPreview: false`.

Регистрация меню команд Telegram обрабатывается при запуске с помощью `setMyCommands`. Нативные команды по умолчанию:

-   `commands.native: "auto"` включает нативные команды для Telegram

Добавьте пользовательские пункты меню команд:

```json
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

Правила:

-   имена нормализуются (удаляется ведущий `/`, приводится к нижнему регистру)
-   допустимый шаблон: `a-z`, `0-9`, `_`, длина `1..32`
-   пользовательские команды не могут переопределять нативные команды
-   конфликты/дубликаты пропускаются и логируются

Примечания:

-   пользовательские команды — это только пункты меню; они не реализуют поведение автоматически
-   команды плагинов/навыков могут по-прежнему работать при вводе, даже если они не отображаются в меню Telegram

Если нативные команды отключены, встроенные команды удаляются. Пользовательские/плагинные команды могут по-прежнему регистрироваться, если настроены. Частая ошибка настройки:

-   `setMyCommands failed` обычно означает, что исходящий DNS/HTTPS к `api.telegram.org` заблокирован.

### Команды связывания устройств (плагин device-pair)

Когда установлен плагин `device-pair`:

1.  `/pair` генерирует код настройки
2.  вставьте код в приложение iOS
3.  `/pair approve` подтверждает последний ожидающий запрос

Подробнее: [Связывание](./pairing.md#pair-via-telegram-recommended-for-ios).

Настройте область действия встроенной клавиатуры:

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

Переопределение для каждого аккаунта:

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

Области действия:

-   `off`
-   `dm`
-   `group`
-   `all`
-   `allowlist` (по умолчанию)

Устаревший `capabilities: ["inlineButtons"]` преобразуется в `inlineButtons: "all"`. Пример действия сообщения:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

Клики по callback передаются агенту как текст: `callback_data: `

Действия инструментов Telegram включают:

-   `sendMessage` (`to`, `content`, опционально `mediaUrl`, `replyToMessageId`, `messageThreadId`)
-   `react` (`chatId`, `messageId`, `emoji`)
-   `deleteMessage` (`chatId`, `messageId`)
-   `editMessage` (`chatId`, `messageId`, `content`)
-   `createForumTopic` (`chatId`, `name`, опционально `iconColor`, `iconCustomEmojiId`)

Действия с сообщениями канала предоставляют эргономичные псевдонимы (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`, `topic-create`). Управляющие элементы:

-   `channels.telegram.actions.sendMessage`
-   `channels.telegram.actions.deleteMessage`
-   `channels.telegram.actions.reactions`
-   `channels.telegram.actions.sticker` (по умолчанию: отключено)

Примечание: `edit` и `topic-create` в настоящее время включены по умолчанию и не имеют отдельных переключателей `channels.telegram.actions.*`. Семантика удаления реакций: [/tools/reactions](../tools/reactions.md)

Telegram поддерживает явные теги ответов в тредах в сгенерированном выводе:

-   `[[reply_to_current]]` отвечает на инициирующее сообщение
-   `[[reply_to:]]` отвечает на конкретный ID сообщения Telegram

`channels.telegram.replyToMode` управляет обработкой:

-   `off` (по умолчанию)
-   `first`
-   `all`

Примечание: `off` отключает неявное создание тредов ответов. Явные теги `[[reply_to_*]]` по-прежнему учитываются.

Супергруппы-форумы:

-   ключи сессии топика дополняются `:topic:`
-   ответы и индикатор набора нацелены на тред топика
-   путь конфигурации топика: `channels.telegram.groups..topics.`

Особый случай общего топика (`threadId=1`):

-   отправка сообщений опускает `message_thread_id` (Telegram отклоняет `sendMessage(...thread_id=1)`)
-   действия с индикатором набора все еще включают `message_thread_id`

Наследование настроек топика: записи топика наследуют настройки группы, если не переопределены (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`). `agentId` является уникальным для топика и не наследуется от настроек группы по умолчанию.**Маршрутизация агента для каждого топика**: Каждый топик может быть направлен к другому агенту, установив `agentId` в конфиге топика. Это дает каждому топику свою изолированную рабочую область, память и сессию. Пример:

```json
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "1": { agentId: "main" },      // Общий топик → основной агент
            "3": { agentId: "zu" },        // Топик разработки → агент zu
            "5": { agentId: "coder" }      // Ревью кода → агент coder
          }
        }
      }
    }
  }
}
```

Каждый топик затем имеет свой собственный ключ сессии: `agent:zu:telegram:group:-1001234567890:topic:3`**Постоянная привязка топика ACP**: Темы форума могут закреплять сессии ACP harness через привязки ACP верхнего уровня:

-   `bindings[]` с `type: "acp"` и `match.channel: "telegram"`

Пример:

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

В настоящее время это ограничено темами форума в группах и супергруппах.**Запуск ACP, привязанного к треду, из чата**:

-   `/acp spawn  --thread here|auto` может привязать текущий топик Telegram к новой сессии ACP.
-   Последующие сообщения в топике направляются непосредственно к привязанной сессии ACP (не требуется `/acp steer`).
-   OpenClaw закрепляет сообщение подтверждения запуска в топике после успешной привязки.
-   Требуется `channels.telegram.threadBindings.spawnAcpSessions=true`.

Контекст шаблона включает:

-   `MessageThreadId`
-   `IsForum`

Поведение тредов в личных сообщениях:

-   приватные чаты с `message_thread_id` сохраняют маршрутизацию ЛС, но используют ключи сессии и цели ответов, учитывающие тред.

### Голосовые сообщения

Telegram различает голосовые заметки и аудиофайлы.

-   по умолчанию: поведение аудиофайла
-   тег `[[audio_as_voice]]` в ответе агента для принудительной отправки как голосовой заметки

Пример действия сообщения:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

### Видеосообщения

Telegram различает видеофайлы и видеозаметки. Пример действия сообщения:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

Видеозаметки не поддерживают подписи; предоставленный текст сообщения отправляется отдельно.

### Стикеры

Обработка входящих стикеров:

-   статические WEBP: скачиваются и обрабатываются (заполнитель `<media:sticker>`)
-   анимированные TGS: пропускаются
-   видео WEBM: пропускаются

Поля контекста стикера:

-   `Sticker.emoji`
-   `Sticker.setName`
-   `Sticker.fileId`
-   `Sticker.fileUniqueId`
-   `Sticker.cachedDescription`

Файл кэша стикеров:

-   `~/.openclaw/telegram/sticker-cache.json`

Стикеры описываются один раз (когда возможно) и кэшируются, чтобы уменьшить повторные вызовы vision. Включите действия со стикерами:

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

Действие отправки стикера:

```json
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Поиск кэшированных стикеров:

```json
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Реакции Telegram приходят как обновления `message_reaction` (отдельно от полезной нагрузки сообщения). Когда включено, OpenClaw ставит в очередь системные события, такие как:

-   `Telegram reaction added: 👍 by Alice (@alice) on msg 42`

Конфигурация:

-   `channels.telegram.reactionNotifications`: `off | own | all` (по умолчанию: `own`)
-   `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (по умолчанию: `minimal`)

Примечания:

-   `own` означает реакции пользователей только на сообщения, отправленные ботом (best-effort через кэш отправленных сообщений).
-   События реакций по-прежнему соблюдают контроль доступа Telegram (`dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`); неавторизованные отправители отбрасываются.
-   Telegram не предоставляет ID тредов в обновлениях реакций.
    -   не-форум группы направляются в групповую сессию чата
    -   группы-форумы направляются в сессию общего топика группы (`:topic:1`), а не в точный исходный топик

`allowed_updates` для polling/webhook автоматически включают `message_reaction`.

`ackReaction` отправляет эмодзи подтверждения, пока OpenClaw обрабатывает входящее сообщение. Порядок разрешения:

-   `channels.telegram.accounts..ackReaction`
-   `channels.telegram.ackReaction`
-   `messages.ackReaction`
-   резервный вариант эмодзи идентичности агента (`agents.list[].identity.emoji`, иначе ”👀”)

Примечания:

-   Telegram ожидает эмодзи в формате Unicode (например, ”👀”).
-   Используйте `""`, чтобы отключить реакцию для канала или аккаунта.

Запись конфигурации канала включена по умолчанию (`configWrites !== false`). Записи, инициированные Telegram, включают:

-   события миграции группы (`migrate_to_chat_id`) для обновления `channels.telegram.groups`
-   `/config set` и `/config unset` (требует включения команд)

Отключить:

```json
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

По умолчанию: long polling. Режим webhook:

-   установите `channels.telegram.webhookUrl`
-   установите `channels.telegram.webhookSecret` (обязательно при установке webhook URL)
-   опционально `channels.telegram.webhookPath` (по умолчанию `/telegram-webhook`)
-   опционально `channels.telegram.webhookHost` (по умолчанию `127.0.0.1`)
-   опционально `channels.telegram.webhookPort` (по умолчанию `8787`)

Локальный слушатель по умолчанию для режима webhook привязывается к `127.0.0.1:8787`. Если ваш публичный endpoint отличается, разместите обратный прокси перед ним и укажите `webhookUrl` на публичный URL. Установите `webhookHost` (например, `0.0.0.0`), когда вам намеренно нужен внешний вход.

-   `channels.telegram.textChunkLimit` по умолчанию равен 4000.
-   `channels.telegram.chunkMode="newline"` предпочитает границы абзацев (пустые строки) перед разделением по длине.
-   `channels.telegram.mediaMaxMb` (по умолчанию 100) ограничивает размер входящих и исходящих медиа в Telegram.
-   `channels.telegram.timeoutSeconds` переопределяет таймаут клиента Telegram API (если не установлено, применяется значение по умолчанию grammY).
-   история группового контекста использует `channels.telegram.historyLimit` или `messages.groupChat.historyLimit` (по умолчанию 50); `0` отключает.
-   управление историей личных сообщений:
    -   `channels.telegram.dmHistoryLimit`
    -   `channels.telegram.dms["<user_id>"].historyLimit`
-   конфигурация `channels.telegram.retry` применяется к вспомогательным функциям отправки Telegram (CLI/инструменты/действия) для исправимых ошибок исходящего API.

Цель отправки CLI может быть числовым ID чата или именем пользователя:

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

Опросы Telegram используют `openclaw message poll` и поддерживают темы форума:

```bash
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "Ship it?" --poll-option "Yes" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Pick a time" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300 --poll-public
```

Флаги опросов только для Telegram:

-   `--poll-duration-seconds` (5-600)
-   `--poll-anonymous`
-   `--poll-public`
-   `--thread-id` для тем форума (или используйте цель с `:topic:`)

Управление действиями:

-   `channels.telegram.actions.sendMessage=false` отключает исходящие сообщения Telegram, включая опросы
-   `channels.telegram.actions.poll=false` отключает создание опросов Telegram, оставляя обычную отправку включенной

## Устранение неполадок

-   Если `requireMention=false`, режим конфиденциальности Telegram должен разрешать полную видимость.
    -   BotFather: `/setprivacy` -> Отключить
    -   затем удалите + снова добавьте бота в группу
-   `openclaw channels status` предупреждает, когда конфиг ожидает групповых сообщений без упоминания.
-   `openclaw channels status --probe` может проверить явные числовые ID групп; подстановочный знак `"*"` не может быть проверен на членство.
-   быстрый тест сессии: `/activation always`.

-   когда существует `channels.telegram.groups`, группа должна быть указана (или включать `"*"`)
-   проверьте членство бота в группе
-   проверьте логи: `openclaw logs --follow` на предмет причин пропуска

-   авторизуйте вашу личность отправителя (связывание и/или числовой `allowFrom`)
-   авторизация команд все еще применяется, даже если политика группы `open`
-   `setMyCommands failed` обычно указывает на проблемы с доступностью DNS/HTTPS к `api.telegram.org`

-   Node 22+ + пользовательский fetch/прокси может вызывать немедленное поведение прерывания, если типы AbortSignal не совпадают.
-   Некоторые хосты разрешают `api.telegram.org` сначала в IPv6; сломанный исходящий IPv6 может вызывать периодические сбои Telegram API.
-   Если логи содержат `TypeError: fetch failed` или `Network request for 'getUpdates' failed!`, OpenClaw теперь повторяет эти попытки как исправимые сетевые ошибки.
-   На VPS-хостах с нестабильным прямым исходящим трафиком/TLS направляйте вызовы Telegram API через `channels.telegram.proxy`:

```yaml
channels:
  telegram:
    proxy: socks5://<user>:<password>@proxy-host:1080
```

-   Node 22+ по умолчанию использует `autoSelectFamily=true` (кроме WSL2) и `dnsResultOrder=ipv4first`.
-   Если ваш хост — WSL2 или явно лучше работает с поведением только IPv4, принудительно выберите семейство:

```yaml
channels:
  telegram:
    network:
      autoSelectFamily: false
```

-   Переопределения переменных окружения (временные):
    -   `OPENCLAW_TELEGRAM_DISABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_ENABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_DNS_RESULT_ORDER=ipv4first`
-   Проверьте ответы DNS:

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

 Дополнительная помощь: [Устранение неполадок каналов](./troubleshooting.md).

## Указатели на справочник конфигурации Telegram

Основной справочник:

-   `channels.telegram.enabled`: включить/отключить запуск канала.
-   `channels.telegram.botToken`: токен бота (BotFather).
-   `channels.telegram.tokenFile`: читать токен из пути к файлу.
-   `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (по умолчанию: pairing).
-   `channels.telegram.allowFrom`: белый список личных сообщений (числовые ID пользователей Telegram). `allowlist` требует хотя бы одного ID отправителя. `open` требует `"*"`. `openclaw doctor --fix` может разрешить устаревшие записи `@username` в ID и может восстановить записи белого списка из файлов хранилища связывания при миграции на белый список.
-   `channels.telegram.actions.poll`: включить или отключить создание опросов Telegram (по умолчанию: включено; все еще требует `sendMessage`).
-   `channels.telegram.defaultTo`: цель Telegram по умолчанию, используемая CLI `--deliver`, когда явный `--reply-to` не предоставлен.
-   `channels.telegram.groupPolicy`: `open | allowlist | disabled` (по умолчанию: allowlist).
-   `channels.telegram.groupAllowFrom`: белый список отправителей группы (числовые ID пользователей Telegram). `openclaw doctor --fix` может разрешить устаревшие записи `@username` в ID. Нечисловые записи игнорируются во время авторизации. Авторизация группы не использует резервный вариант хранилища связывания личных сообщений (`2026.2.25+`).
-   Приоритет нескольких аккаунтов:
    -   Когда настроены два или более ID аккаунта, установите `channels.telegram.defaultAccount` (или включите `channels.telegram.accounts.default`), чтобы сделать маршрутизацию по умолчанию явной.
    -   Если ни то, ни другое не установлено, OpenClaw возвращается к первому нормализованному ID аккаунта, и `openclaw doctor` предупреждает.
    -   `channels.telegram.accounts.default.allowFrom` и `channels.telegram.accounts.default.groupAllowFrom` применяются только к аккаунту `default`.
    -   Именованные аккаунты наследуют `channels.telegram.allowFrom` и `channels.telegram.groupAllowFrom`, когда значения на уровне аккаунта не установлены.
    -   Именованные аккаунты не наследуют `channels.telegram.accounts.default.allowFrom` / `groupAllowFrom`.
-   `channels.telegram.groups`: настройки по умолчанию для каждой группы + белый список (используйте `"*"` для глобальных настроек по умолчанию).
    -   `channels.telegram.groups..groupPolicy`: переопределение groupPolicy для каждой группы (`open | allowlist | disabled`).
    -   `channels.telegram.groups..requireMention`: управление упоминаниями по умолчанию.
    -   `channels.telegram.groups..skills`: фильтр навыков (опустить = все навыки, пусто = ни одного).