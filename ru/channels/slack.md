

  Платформы обмена сообщениями

  
# Slack

Статус: готово к работе в продакшене для личных сообщений и каналов через интеграции приложения Slack. Режим по умолчанию — Socket Mode; также поддерживается режим HTTP Events API.

## Быстрая настройка

## Модель токенов

-   Для Socket Mode требуются `botToken` + `appToken`.
-   Для HTTP режима требуются `botToken` + `signingSecret`.
-   Токены из конфигурации переопределяют значения из переменных окружения.
-   Переменные окружения `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` используются только для аккаунта по умолчанию.
-   `userToken` (`xoxp-...`) задаётся только в конфигурации (без переменной окружения) и по умолчанию работает в режиме только для чтения (`userTokenReadOnly: true`).
-   Опционально: добавьте `chat:write.customize`, если хотите, чтобы исходящие сообщения использовали идентификатор активного агента (пользовательские `username` и иконка). Для `icon_emoji` используется синтаксис `:emoji_name:`.

> **💡** Для действий/чтения директории может быть предпочтительным использование пользовательского токена, если он настроен. Для записи предпочтительным остаётся токен бота; запись с пользовательским токеном разрешена только при `userTokenReadOnly: false` и недоступности токена бота.

## Контроль доступа и маршрутизация

`channels.slack.dmPolicy` управляет доступом к личным сообщениям (устаревшее: `channels.slack.dm.policy`):

-   `pairing` (по умолчанию)
-   `allowlist`
-   `open` (требует, чтобы `channels.slack.allowFrom` включал `"*"`; устаревшее: `channels.slack.dm.allowFrom`)
-   `disabled`

Флаги для личных сообщений:

-   `dm.enabled` (по умолчанию true)
-   `channels.slack.allowFrom` (предпочтительный)
-   `dm.allowFrom` (устаревший)
-   `dm.groupEnabled` (групповые личные сообщения по умолчанию false)
-   `dm.groupChannels` (опциональный разрешительный список MPIM)

Приоритет для нескольких аккаунтов:

-   `channels.slack.accounts.default.allowFrom` применяется только к аккаунту `default`.
-   Именованные аккаунты наследуют `channels.slack.allowFrom`, если их собственный `allowFrom` не задан.
-   Именованные аккаунты не наследуют `channels.slack.accounts.default.allowFrom`.

Связывание в личных сообщениях использует команду `openclaw pairing approve slack `.

`channels.slack.groupPolicy` управляет обработкой каналов:

-   `open`
-   `allowlist`
-   `disabled`

Разрешительный список каналов находится в `channels.slack.channels`.
Примечание для времени выполнения: если `channels.slack` полностью отсутствует (настройка только через env), среда выполнения возвращается к `groupPolicy="allowlist"` и выводит предупреждение (даже если задан `channels.defaults.groupPolicy`).
Разрешение имён/ID:

-   записи в разрешительном списке каналов и личных сообщений разрешаются при запуске, если доступ по токену позволяет
-   неразрешённые записи сохраняются в том виде, в котором сконфигурированы
-   сопоставление для входящей авторизации по умолчанию сначала по ID; прямое сопоставление по имени пользователя/слагу требует `channels.slack.dangerouslyAllowNameMatching: true`

Сообщения в каналах по умолчанию требуют упоминания.
Источники упоминания:

-   явное упоминание приложения (`<@botId>`)
-   шаблоны регулярных выражений для упоминаний (`agents.list[].groupChat.mentionPatterns`, резервный вариант `messages.groupChat.mentionPatterns`)
-   неявное поведение ответа боту в ветке обсуждения

Контроль для каждого канала (`channels.slack.channels.<id|name>`):

-   `requireMention`
-   `users` (разрешительный список)
-   `allowBots`
-   `skills`
-   `systemPrompt`
-   `tools`, `toolsBySender`
-   Формат ключа `toolsBySender`: `id:`, `e164:`, `username:`, `name:` или подстановочный знак `"*"` (устаревшие ключи без префикса всё ещё сопоставляются только с `id:`)

## Команды и поведение слэш-команд

-   Автоматический режим нативных команд **отключён** для Slack (`commands.native: "auto"` не включает нативные команды Slack).
-   Включите обработчики нативных команд Slack с помощью `channels.slack.commands.native: true` (или глобально `commands.native: true`).
-   Когда нативные команды включены, зарегистрируйте соответствующие слэш-команды в Slack (имена `/`), за одним исключением:
    -   зарегистрируйте `/agentstatus` для команды статуса (Slack резервирует `/status`)
-   Если нативные команды не включены, вы можете выполнить одну настроенную слэш-команду через `channels.slack.slashCommand`.
-   Меню аргументов нативных команд теперь адаптируют свою стратегию отображения:
    -   до 5 вариантов: блоки кнопок
    -   6-100 вариантов: статическое выпадающее меню
    -   более 100 вариантов: внешнее выпадающее меню с асинхронной фильтрацией вариантов, когда доступны обработчики опций интерактивности
    -   если закодированные значения вариантов превышают лимиты Slack, процесс возвращается к кнопкам
-   Для длинных полезных нагрузок опций меню аргументов слэш-команд используют диалог подтверждения перед отправкой выбранного значения.

Настройки слэш-команды по умолчанию:

-   `enabled: false`
-   `name: "openclaw"`
-   `sessionPrefix: "slack:slash"`
-   `ephemeral: true`

Сессии слэш-команд используют изолированные ключи:

-   `agent::slack:slash:`

и всё ещё направляют выполнение команды в сессию целевого разговора (`CommandTargetSessionKey`).

## Ветки обсуждений, сессии и теги ответов

-   Личные сообщения маршрутизируются как `direct`; каналы как `channel`; MPIM как `group`.
-   При настройке по умолчанию `session.dmScope=main` личные сообщения Slack объединяются в основную сессию агента.
-   Сессии каналов: `agent::slack:channel:`.
-   Ответы в ветке могут создавать суффиксы сессии ветки (`:thread:`), когда это применимо.
-   `channels.slack.thread.historyScope` по умолчанию `thread`; `thread.inheritParent` по умолчанию `false`.
-   `channels.slack.thread.initialHistoryLimit` контролирует, сколько существующих сообщений ветки загружается при запуске новой сессии ветки (по умолчанию `20`; установите `0` для отключения).

Контроль ветвления ответов:

-   `channels.slack.replyToMode`: `off|first|all` (по умолчанию `off`)
-   `channels.slack.replyToModeByChatType`: для каждого `direct|group|channel`
-   устаревший резервный вариант для прямых чатов: `channels.slack.dm.replyToMode`

Поддерживаются ручные теги ответов:

-   `[[reply_to_current]]`
-   `[[reply_to:]]`

Примечание: `replyToMode="off"` отключает **все** ветки ответов в Slack, включая явные теги `[[reply_to_*]]`. Это отличается от Telegram, где явные теги всё ещё учитываются в режиме `"off"`. Разница отражает модели ветвления платформ: ветки Slack скрывают сообщения от канала, в то время как ответы Telegram остаются видимыми в основном потоке чата.

## Медиа, разбиение на части и доставка

Вложения файлов Slack загружаются с приватных URL-адресов, размещённых в Slack (поток запроса с аутентификацией по токену), и записываются в хранилище медиа при успешной загрузке и соблюдении ограничений размера.
Верхний предел размера для входящих данных по умолчанию — `20MB`, если не переопределён `channels.slack.mediaMaxMb`.

-   части текста используют `channels.slack.textChunkLimit` (по умолчанию 4000)
-   `channels.slack.chunkMode="newline"` включает разбиение сначала по абзацам
-   отправка файлов использует API загрузки Slack и может включать ответы в ветке (`thread_ts`)
-   верхний предел для исходящих медиа следует `channels.slack.mediaMaxMb`, если настроено; в противном случае отправка в канал использует значения по умолчанию для типа MIME из конвейера медиа

Предпочтительные явные цели:

-   `user:` для личных сообщений
-   `channel:` для каналов

Личные сообщения Slack открываются через API разговоров Slack при отправке пользователям.

## Действия и шлюзы

Действия Slack контролируются `channels.slack.actions.*`. Доступные группы действий в текущем инструментарии Slack:

| Группа | По умолчанию |
| --- | --- |
| messages | включено |
| reactions | включено |
| pins | включено |
| memberInfo | включено |
| emojiList | включено |

## События и операционное поведение

-   Редактирование/удаление сообщений и широковещательные рассылки в ветках преобразуются в системные события.
-   События добавления/удаления реакции преобразуются в системные события.
-   События присоединения/выхода участника, создания/переименования канала и добавления/удаления закрепления преобразуются в системные события.
-   Обновления статуса ветки ассистента (для индикаторов "печатает..." в ветках) используют `assistant.threads.setStatus` и требуют области видимости бота `assistant:write`.
-   `channel_id_changed` может переносить ключи конфигурации канала при включённом `configWrites`.
-   Метаданные темы/цели канала рассматриваются как ненадёжный контекст и могут быть внедрены в контекст маршрутизации.
-   Действия с блоками и модальные взаимодействия генерируют структурированные системные события `Slack interaction: ...` с богатыми полями полезной нагрузки:
    -   действия с блоками: выбранные значения, метки, значения средства выбора и метаданные `workflow_*`
    -   события модального окна `view_submission` и `view_closed` с метаданными маршрутизированного канала и вводами формы

## Реакции подтверждения

`ackReaction` отправляет эмодзи подтверждения, пока OpenClaw обрабатывает входящее сообщение. Порядок разрешения:

-   `channels.slack.accounts..ackReaction`
-   `channels.slack.ackReaction`
-   `messages.ackReaction`
-   резервный вариант эмодзи идентификатора агента (`agents.list[].identity.emoji`, иначе ”👀”)

Примечания:

-   Slack ожидает короткие коды (например, `"eyes"`).
-   Используйте `""`, чтобы отключить реакцию для аккаунта Slack или глобально.

## Резервный вариант реакции "печатает"

`typingReaction` добавляет временную реакцию к входящему сообщению Slack, пока OpenClaw обрабатывает ответ, а затем удаляет её по завершении выполнения. Это полезный резервный вариант, когда нативная индикация печати ассистента Slack недоступна, особенно в личных сообщениях. Порядок разрешения:

-   `channels.slack.accounts..typingReaction`
-   `channels.slack.typingReaction`

Примечания:

-   Slack ожидает короткие коды (например, `"hourglass_flowing_sand"`).
-   Реакция выполняется по принципу best-effort, и очистка автоматически предпринимается после завершения ответа или пути сбоя.

## Манифест и контрольный список областей видимости

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "assistant:write",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

Если вы настраиваете `channels.slack.userToken`, типичные области видимости для чтения:

-   `channels:history`, `groups:history`, `im:history`, `mpim:history`
-   `channels:read`, `groups:read`, `im:read`, `mpim:read`
-   `users:read`
-   `reactions:read`
-   `pins:read`
-   `emoji:read`
-   `search:read` (если вы зависите от поиска в Slack)

## Диагностика проблем

Проверьте по порядку:

-   `groupPolicy`
-   разрешительный список каналов (`channels.slack.channels`)
-   `requireMention`
-   разрешительный список `users` для каждого канала

Полезные команды:

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

Проверьте:

-   `channels.slack.dm.enabled`
-   `channels.slack.dmPolicy` (или устаревшее `channels.slack.dm.policy`)
-   подтверждения связывания / записи в разрешительном списке

```bash
openclaw pairing list slack
```

Проверьте токены бота и приложения, а также включение Socket Mode в настройках приложения Slack.

Проверьте:

-   секрет подписи
-   путь вебхука
-   URL-адреса запросов Slack (Events + Interactivity + Slash Commands)
-   уникальный `webhookPath` для каждого HTTP-аккаунта

Убедитесь, что вы планировали:

-   режим нативных команд (`channels.slack.commands.native: true`) с соответствующими зарегистрированными слэш-командами в Slack
-   или режим одиночной слэш-команды (`channels.slack.slashCommand.enabled: true`)

Также проверьте `commands.useAccessGroups` и разрешительные списки каналов/пользователей.

## Потоковая передача текста

OpenClaw поддерживает нативную потоковую передачу текста Slack через API Agents and AI Apps. `channels.slack.streaming` управляет поведением живого предпросмотра:

-   `off`: отключить потоковую передачу живого предпросмотра.
-   `partial` (по умолчанию): заменять текст предпросмотра последним частичным выводом.
-   `block`: добавлять обновления предпросмотра частями.
-   `progress`: показывать текст статуса выполнения во время генерации, затем отправлять окончательный текст.

`channels.slack.nativeStreaming` управляет нативным потоковым API Slack (`chat.startStream` / `chat.appendStream` / `chat.stopStream`), когда `streaming` установлен в `partial` (по умолчанию: `true`). Отключите нативную потоковую передачу Slack (сохраните поведение черновика предпросмотра):

```yaml
channels:
  slack:
    streaming: partial
    nativeStreaming: false
```

Устаревшие ключи:

-   `channels.slack.streamMode` (`replace | status_final | append`) автоматически мигрирует в `channels.slack.streaming`.
-   булево значение `channels.slack.streaming` автоматически мигрирует в `channels.slack.nativeStreaming`.

### Требования

1.  Включите **Agents and AI Apps** в настройках вашего приложения Slack.
2.  Убедитесь, что у приложения есть область видимости `assistant:write`.
3.  Для этого сообщения должна быть доступна ветка ответов. Выбор ветки всё ещё следует `replyToMode`.

### Поведение

-   Первая часть текста запускает поток (`chat.startStream`).
-   Последующие части текста добавляются в тот же поток (`chat.appendStream`).
-   Конец ответа завершает поток (`chat.stopStream`).
-   Медиа и полезные нагрузки, не являющиеся текстом, возвращаются к обычной доставке.
-   Если потоковая передача прерывается в середине ответа, OpenClaw возвращается к обычной доставке для оставшихся полезных нагрузок.

## Указатели на справочник конфигурации

Основной справочник:

-   [Справочник по конфигурации - Slack](../gateway/configuration-reference.md#slack) Важные поля Slack:
    -   режим/аутентификация: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
    -   доступ к личным сообщениям: `dm.enabled`, `dmPolicy`, `allowFrom` (устаревшее: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
    -   переключатель совместимости: `dangerouslyAllowNameMatching` (аварийный; держите выключенным, если не требуется)
    -   доступ к каналам: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
    -   ветки/история: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
    -   доставка: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `streaming`, `nativeStreaming`
    -   операции/функции: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Связанные темы

-   [Связывание](./pairing.md)
-   [Маршрутизация каналов](./channel-routing.md)
-   [Диагностика проблем](./troubleshooting.md)
-   [Конфигурация](../gateway/configuration.md)
-   [Слэш-команды](../tools/slash-commands.md)

[Synology Chat](./synology-chat.md)[Telegram](./telegram.md)