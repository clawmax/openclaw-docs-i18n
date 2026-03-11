

  Команды CLI

  
# directory

Поиск в директории для каналов, которые это поддерживают (контакты/участники, группы и "я").

## Общие флаги

-   `--channel `: id/алиас канала (обязателен при настройке нескольких каналов; определяется автоматически, если настроен только один)
-   `--account `: id аккаунта (по умолчанию: канал по умолчанию)
-   `--json`: вывод в формате JSON

## Примечания

-   Команда `directory` предназначена для помощи в поиске ID, которые можно вставить в другие команды (особенно `openclaw message send --target ...`).
-   Для многих каналов результаты основаны на конфигурации (белые списки / настроенные группы), а не на живой директории провайдера.
-   Вывод по умолчанию — `id` (и иногда `name`), разделённые табуляцией; используйте `--json` для скриптов.

## Использование результатов с message send

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## Форматы ID (по каналам)

-   WhatsApp: `+15551234567` (личные сообщения), `1234567890-1234567890@g.us` (группа)
-   Telegram: `@username` или числовой id чата; группы имеют числовые id
-   Slack: `user:U…` и `channel:C…`
-   Discord: `user:` и `channel:`
-   Matrix (плагин): `user:@user:server`, `room:!roomId:server` или `#alias:server`
-   Microsoft Teams (плагин): `user:` и `conversation:`
-   Zalo (плагин): id пользователя (Bot API)
-   Zalo Personal / `zalouser` (плагин): id беседы (личные/групповые) из `zca` (`me`, `friend list`, `group list`)

## Себя ("me")

```bash
openclaw directory self --channel zalouser
```

## Участники (контакты/пользователи)

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```

## Группы

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```

[devices](./devices.md)[dns](./dns.md)