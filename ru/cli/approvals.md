title: "Управление разрешениями на выполнение команд OpenClaw CLI для локального шлюза и узлов"
description: "Узнайте, как получать, устанавливать и управлять разрешениями на выполнение команд для локальных хостов, шлюзов и узлов с помощью команд OpenClaw CLI и вспомогательных инструментов для списка разрешений."
keywords: ["разрешения openclaw", "разрешения на выполнение cli", "команды списка разрешений", "разрешения для узлов и хостов", "разрешения для шлюза", "управление разрешениями на выполнение", "openclaw cli", "разрешения в командной строке"]
---

  Команды CLI

  
# approvals

Управляйте разрешениями на выполнение команд для **локального хоста**, **хоста шлюза** или **хоста узла**. По умолчанию команды работают с локальным файлом разрешений на диске. Используйте `--gateway` для работы с шлюзом или `--node` для работы с конкретным узлом. Связанные темы:

-   Разрешения на выполнение: [Разрешения на выполнение](../tools/exec-approvals.md)
-   Узлы: [Узлы](../nodes.md)

## Часто используемые команды

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

## Замена разрешений из файла

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

## Вспомогательные инструменты для списка разрешений

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

## Примечания

-   `--node` использует тот же механизм разрешения, что и `openclaw nodes` (id, имя, ip или префикс id).
-   `--agent` по умолчанию имеет значение `"*"`, что применяется ко всем агентам.
-   Хост узла должен поддерживать `system.execApprovals.get/set` (приложение macOS или headless хост узла).
-   Файлы разрешений хранятся на каждом хосте по пути `~/.openclaw/exec-approvals.json`.

[agents](./agents.md)[browser](./browser.md)

---