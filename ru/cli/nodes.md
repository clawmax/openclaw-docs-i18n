

  Команды CLI

  
# nodes

Управляйте подключенными узлами (устройствами) и вызывайте их возможности. Связанные разделы:

-   Обзор узлов: [Узлы](../nodes.md)
-   Камера: [Узлы камеры](../nodes/camera.md)
-   Изображения: [Узлы изображений](../nodes/images.md)

Общие опции:

-   `--url`, `--token`, `--timeout`, `--json`

## Часто используемые команды

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` выводит таблицы ожидающих/подключенных узлов. Строки подключенных узлов включают время с последнего подключения (Last Connect). Используйте `--connected`, чтобы показывать только узлы, подключенные в данный момент. Используйте `--last-connected ` для фильтрации узлов, которые подключались в течение указанного периода (например, `24h`, `7d`).

## Вызов / выполнение

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Флаги для invoke:

-   `--params `: Строка JSON-объекта (по умолчанию `{}`).
-   `--invoke-timeout `: Таймаут вызова узла (по умолчанию `15000`).
-   `--idempotency-key `: Опциональный ключ идемпотентности.

### Стандартное поведение в стиле exec

`nodes run` повторяет поведение exec модели (значения по умолчанию + утверждения):

-   Читает `tools.exec.*` (плюс переопределения из `agents.list[].tools.exec.*`).
-   Использует утверждения exec (`exec.approval.request`) перед вызовом `system.run`.
-   `--node` можно опустить, если задан `tools.exec.node`.
-   Требуется узел, который предоставляет `system.run` (приложение-компаньон macOS или headless-хост узла).

Флаги:

-   `--cwd `: Рабочий каталог.
-   `--env <key=val>`: Переопределение переменной окружения (можно повторять). Примечание: хосты узлов игнорируют переопределения `PATH` (и `tools.exec.pathPrepend` не применяется к хостам узлов).
-   `--command-timeout `: Таймаут команды.
-   `--invoke-timeout `: Таймаут вызова узла (по умолчанию `30000`).
-   `--needs-screen-recording`: Требовать разрешение на запись экрана.
-   `--raw `: Выполнить строку shell (`/bin/sh -lc` или `cmd.exe /c`). В режиме разрешённого списка на хостах узлов Windows выполнение команд в оболочке `cmd.exe /c` требует утверждения (одного только наличия записи в разрешённом списке недостаточно для автоматического разрешения формы с оболочкой).
-   `--agent `: Утверждения/разрешённые списки в рамках агента (по умолчанию используется настроенный агент).
-   `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: Переопределения.

[node](./node.md)[onboard](./onboard.md)

---