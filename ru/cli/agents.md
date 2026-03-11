

  Команды CLI

  
# agents

Управление изолированными агентами (рабочее пространство + аутентификация + маршрутизация). Связанные темы:

-   Многопользовательская маршрутизация: [Многопользовательская маршрутизация](../concepts/multi-agent.md)
-   Рабочее пространство агента: [Рабочее пространство агента](../concepts/agent-workspace.md)

## Примеры

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents bindings
openclaw agents bind --agent work --bind telegram:ops
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Привязки маршрутизации

Используйте привязки маршрутизации, чтобы закрепить входящий трафик канала за конкретным агентом. Список привязок:

```bash
openclaw agents bindings
openclaw agents bindings --agent work
openclaw agents bindings --json
```

Добавление привязок:

```bash
openclaw agents bind --agent work --bind telegram:ops --bind discord:guild-a
```

Если вы опустите `accountId` (`--bind `), OpenClaw разрешит его из настроек канала по умолчанию и хуков настройки плагина, когда это возможно.

### Поведение области действия привязки

-   Привязка без `accountId` соответствует только учетной записи канала по умолчанию.
-   `accountId: "*"` является резервным вариантом для всего канала (все учетные записи) и менее специфична, чем явная привязка к учетной записи.
-   Если у того же агента уже есть соответствующая привязка канала без `accountId`, и вы позже привяжете с явным или разрешенным `accountId`, OpenClaw обновит существующую привязку на месте вместо добавления дубликата.

Пример:

```bash
# начальная привязка только к каналу
openclaw agents bind --agent work --bind telegram

# последующее обновление до привязки с областью действия учетной записи
openclaw agents bind --agent work --bind telegram:ops
```

После обновления маршрутизация для этой привязки ограничена областью `telegram:ops`. Если вам также нужна маршрутизация для учетной записи по умолчанию, добавьте ее явно (например, `--bind telegram:default`). Удаление привязок:

```bash
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents unbind --agent work --all
```

## Файлы идентификации

Каждое рабочее пространство агента может включать файл `IDENTITY.md` в корне рабочего пространства:

-   Пример пути: `~/.openclaw/workspace/IDENTITY.md`
-   `set-identity --from-identity` читает из корня рабочего пространства (или явно указанного `--identity-file`)

Пути к аватарам разрешаются относительно корня рабочего пространства.

## Установка идентификатора

`set-identity` записывает поля в `agents.list[].identity`:

-   `name`
-   `theme`
-   `emoji`
-   `avatar` (путь относительно рабочего пространства, http(s) URL или data URI)

Загрузка из `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Явное переопределение полей:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Пример конфигурации:

```json
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```

[agent](./agent.md)[approvals](./approvals.md)