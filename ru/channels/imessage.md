

  Платформы обмена сообщениями

  
# iMessage

> **⚠️** Для новых развертываний iMessage используйте [BlueBubbles](./bluebubbles.md). Интеграция `imsg` является устаревшей и может быть удалена в будущем релизе.

 Статус: устаревшая внешняя CLI-интеграция. Шлюз запускает `imsg rpc` и обменивается данными через JSON-RPC на stdio (отдельный демон/порт не требуется). 

## Быстрая настройка

## Требования и разрешения (macOS)

-   В приложении "Сообщения" на Mac, где запущен `imsg`, должен быть выполнен вход.
-   Для контекста процесса, запускающего OpenClaw/`imsg`, требуется доступ "Полный доступ к диску" (для доступа к БД сообщений).
-   Для отправки сообщений через Messages.app требуется разрешение "Автоматизация".

> **💡** Разрешения предоставляются для каждого контекста процесса. Если шлюз работает без графического интерфейса (LaunchAgent/SSH), запустите однократную интерактивную команду в том же контексте, чтобы вызвать запросы:
> 
> Копировать
> 
> ```
> imsg chats --limit 1
> # или
> imsg send <handle> "test"
> ```

## Контроль доступа и маршрутизация

`channels.imessage.dmPolicy` управляет личными сообщениями:

-   `pairing` (по умолчанию)
-   `allowlist`
-   `open` (требует, чтобы `allowFrom` включал `"*"`)
-   `disabled`

Поле списка разрешений: `channels.imessage.allowFrom`. Элементы списка разрешений могут быть идентификаторами (handles) или целями чатов (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).

`channels.imessage.groupPolicy` управляет обработкой групп:

-   `allowlist` (по умолчанию при настройке)
-   `open`
-   `disabled`

Список разрешений отправителей в группах: `channels.imessage.groupAllowFrom`. Резервный вариант во время выполнения: если `groupAllowFrom` не задан, проверка отправителя в группе iMessage возвращается к `allowFrom`, если он доступен. Примечание времени выполнения: если `channels.imessage` полностью отсутствует, среда выполнения возвращается к `groupPolicy="allowlist"` и записывает предупреждение (даже если задан `channels.defaults.groupPolicy`). Управление упоминаниями для групп:

-   В iMessage нет встроенных метаданных об упоминаниях
-   обнаружение упоминаний использует шаблоны регулярных выражений (`agents.list[].groupChat.mentionPatterns`, резервный вариант `messages.groupChat.mentionPatterns`)
-   без настроенных шаблонов управление упоминаниями не может быть применено

Команды управления от авторизованных отправителей могут обходить управление упоминаниями в группах.

-   Личные сообщения используют прямую маршрутизацию; группы используют групповую маршрутизацию.
-   При настройке по умолчанию `session.dmScope=main` личные сообщения iMessage объединяются в основную сессию агента.
-   Групповые сессии изолированы (`agent::imessage:group:<chat_id>`).
-   Ответы маршрутизируются обратно в iMessage с использованием метаданных исходного канала/цели.

Поведение групповых тредов: Некоторые треды iMessage с несколькими участниками могут поступать с `is_group=false`. Если этот `chat_id` явно настроен в `channels.imessage.groups`, OpenClaw обрабатывает его как групповой трафик (групповое управление + изоляция групповой сессии).

## Схемы развертывания

Используйте выделенный Apple ID и пользователя macOS, чтобы трафик бота был изолирован от вашего личного профиля в "Сообщениях". Типичный процесс:

1.  Создайте/войдите под выделенным пользователем macOS.
2.  Войдите в "Сообщения" с Apple ID бота в этом пользователе.
3.  Установите `imsg` для этого пользователя.
4.  Создайте SSH-обертку, чтобы OpenClaw мог запускать `imsg` в контексте этого пользователя.
5.  Настройте `channels.imessage.accounts..cliPath` и `.dbPath` на профиль этого пользователя.

При первом запуске могут потребоваться подтверждения в графическом интерфейсе (Автоматизация + Полный доступ к диску) в сессии пользователя бота.

Распространенная топология:

-   шлюз работает на Linux/VM
-   iMessage + `imsg` работает на Mac в вашей tailnet
-   обертка `cliPath` использует SSH для запуска `imsg`
-   `remoteHost` включает получение вложений через SCP

Пример:

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

Используйте SSH-ключи, чтобы и SSH, и SCP работали неинтерактивно. Убедитесь, что сначала доверен ключ хоста (например, `ssh bot@mac-mini.tailnet-1234.ts.net`), чтобы `known_hosts` был заполнен.

iMessage поддерживает конфигурацию для каждой учетной записи в `channels.imessage.accounts`. Каждая учетная запись может переопределять поля, такие как `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, настройки истории и списки разрешений корневых путей для вложений.

## Медиа, разбиение на части и цели доставки

-   прием входящих вложений опционален: `channels.imessage.includeAttachments`
-   пути к удаленным вложениям могут быть получены через SCP, если задан `remoteHost`
-   пути к вложениям должны соответствовать разрешенным корневым путям:
    -   `channels.imessage.attachmentRoots` (локальные)
    -   `channels.imessage.remoteAttachmentRoots` (режим удаленного SCP)
    -   шаблон корневого пути по умолчанию: `/Users/*/Library/Messages/Attachments`
-   SCP использует строгую проверку ключа хоста (`StrictHostKeyChecking=yes`)
-   размер исходящих медиа использует `channels.imessage.mediaMaxMb` (по умолчанию 16 МБ)

-   ограничение на части текста: `channels.imessage.textChunkLimit` (по умолчанию 4000)
-   режим разбиения: `channels.imessage.chunkMode`
    -   `length` (по умолчанию)
    -   `newline` (разделение по абзацам в первую очередь)

Предпочтительные явные цели:

-   `chat_id:123` (рекомендуется для стабильной маршрутизации)
-   `chat_guid:...`
-   `chat_identifier:...`

Также поддерживаются цели по идентификаторам (handles):

-   `imessage:+1555...`
-   `sms:+1555...`
-   `user@example.com`

```bash
imsg chats --limit 20
```

## Запись конфигурации

iMessage по умолчанию разрешает запись конфигурации, инициированную каналом (для `/config set|unset` при `commands.config: true`). Отключить:

```json
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Устранение неполадок

Проверьте двоичный файл и поддержку RPC:

```bash
imsg rpc --help
openclaw channels status --probe
```

Если проверка сообщает, что RPC не поддерживается, обновите `imsg`.

Проверьте:

-   `channels.imessage.dmPolicy`
-   `channels.imessage.allowFrom`
-   подтверждения связывания (`openclaw pairing list imessage`)

Проверьте:

-   `channels.imessage.groupPolicy`
-   `channels.imessage.groupAllowFrom`
-   поведение списка разрешений `channels.imessage.groups`
-   конфигурацию шаблонов упоминаний (`agents.list[].groupChat.mentionPatterns`)

Проверьте:

-   `channels.imessage.remoteHost`
-   `channels.imessage.remoteAttachmentRoots`
-   аутентификацию по ключу SSH/SCP с хоста шлюза
-   наличие ключа хоста в `~/.ssh/known_hosts` на хосте шлюза
-   доступность удаленного пути для чтения на Mac, где запущены "Сообщения"

Запустите снова в интерактивном терминале с графическим интерфейсом в том же контексте пользователя/сессии и подтвердите запросы:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

Убедитесь, что для контекста процесса, запускающего OpenClaw/`imsg`, предоставлены разрешения "Полный доступ к диску" и "Автоматизация".

## Указатели на справочник по конфигурации

-   [Справочник по конфигурации - iMessage](../gateway/configuration-reference.md#imessage)
-   [Конфигурация шлюза](../gateway/configuration.md)
-   [Связывание (Pairing)](./pairing.md)
-   [BlueBubbles](./bluebubbles.md)

[Google Chat](./googlechat.md)[IRC](./irc.md)