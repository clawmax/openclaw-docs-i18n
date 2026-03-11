

  Команды CLI

  
# sessions

Вывод списка сохранённых сессий диалогов.

```bash
openclaw sessions
openclaw sessions --agent work
openclaw sessions --all-agents
openclaw sessions --active 120
openclaw sessions --json
```

Выбор области действия:

-   default: хранилище настроенного агента по умолчанию
-   `--agent `: одно настроенное хранилище агента
-   `--all-agents`: агрегировать все настроенные хранилища агентов
-   `--store `: явный путь к хранилищу (нельзя комбинировать с `--agent` или `--all-agents`)

Примеры JSON: `openclaw sessions --all-agents --json`:

```json
{
  "path": null,
  "stores": [
    { "agentId": "main", "path": "/home/user/.openclaw/agents/main/sessions/sessions.json" },
    { "agentId": "work", "path": "/home/user/.openclaw/agents/work/sessions/sessions.json" }
  ],
  "allAgents": true,
  "count": 2,
  "activeMinutes": null,
  "sessions": [
    { "agentId": "main", "key": "agent:main:main", "model": "gpt-5" },
    { "agentId": "work", "key": "agent:work:main", "model": "claude-opus-4-5" }
  ]
}
```

## Обслуживание очистки

Запустить обслуживание сейчас (вместо ожидания следующего цикла записи):

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --agent work --dry-run
openclaw sessions cleanup --all-agents --dry-run
openclaw sessions cleanup --enforce
openclaw sessions cleanup --enforce --active-key "agent:main:telegram:dm:123"
openclaw sessions cleanup --json
```

`openclaw sessions cleanup` использует настройки `session.maintenance` из конфигурации:

-   Примечание об области действия: `openclaw sessions cleanup` обслуживает только хранилища сессий/транскрипты. Он не удаляет логи запусков cron (`cron/runs/.jsonl`), которыми управляют параметры `cron.runLog.maxBytes` и `cron.runLog.keepLines` в [Конфигурации Cron](../automation/cron-jobs.md#configuration), объяснённые в [Обслуживании Cron](../automation/cron-jobs.md#maintenance).
-   `--dry-run`: предварительный просмотр того, сколько записей будет удалено/ограничено без записи.
    -   В текстовом режиме dry-run выводит таблицу действий по сессиям (`Action`, `Key`, `Age`, `Model`, `Flags`), чтобы вы могли видеть, что будет сохранено, а что удалено.
-   `--enforce`: применить обслуживание, даже когда `session.maintenance.mode` установлен в `warn`.
-   `--active-key `: защитить конкретный активный ключ от вытеснения по дисковому бюджету.
-   `--agent `: выполнить очистку для одного настроенного хранилища агента.
-   `--all-agents`: выполнить очистку для всех настроенных хранилищ агентов.
-   `--store `: выполнить для конкретного файла `sessions.json`.
-   `--json`: вывести сводку в формате JSON. С `--all-agents` вывод включает одну сводку на хранилище.

`openclaw sessions cleanup --all-agents --dry-run --json`:

```json
{
  "allAgents": true,
  "mode": "warn",
  "dryRun": true,
  "stores": [
    {
      "agentId": "main",
      "storePath": "/home/user/.openclaw/agents/main/sessions/sessions.json",
      "beforeCount": 120,
      "afterCount": 80,
      "pruned": 40,
      "capped": 0
    },
    {
      "agentId": "work",
      "storePath": "/home/user/.openclaw/agents/work/sessions/sessions.json",
      "beforeCount": 18,
      "afterCount": 18,
      "pruned": 0,
      "capped": 0
    }
  ]
}
```

Связанные темы:

-   Конфигурация сессий: [Справочник по конфигурации](../gateway/configuration-reference.md#session)

[security](./security.md)[setup](./setup.md)