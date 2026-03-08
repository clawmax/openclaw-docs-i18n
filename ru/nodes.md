title: "Руководство по узлам OpenClaw для медиаустройств и удаленных хостов"
description: "Узнайте, как настроить и управлять узлами OpenClaw для скриншотов, камеры, геолокации, SMS и удаленного выполнения команд. Включает сопряжение, команды CLI и устранение неполадок."
keywords: ["узлы openclaw", "сопряжение устройств", "удаленный хост узла", "снимок canvas", "команды камеры", "узлы android", "system.run", "протокол шлюза"]
---

  Медиа и устройства

  
# Узлы

**Узел** — это вспомогательное устройство (macOS/iOS/Android/безголовое), которое подключается к **WebSocket** шлюза (тот же порт, что и у операторов) с `role: "node"` и предоставляет поверхность команд (например, `canvas.*`, `camera.*`, `device.*`, `notifications.*`, `system.*`) через `node.invoke`. Подробности протокола: [Протокол шлюза](./gateway/protocol.md). Устаревший транспорт: [Протокол моста](./gateway/bridge-protocol.md) (TCP JSONL; устарел/удален для текущих узлов). macOS также может работать в **режиме узла**: приложение в строке меню подключается к WS-серверу шлюза и предоставляет свои локальные команды canvas/camera как узел (так что `openclaw nodes …` работает на этом Mac). Примечания:

-   Узлы — это **периферийные устройства**, а не шлюзы. Они не запускают службу шлюза.
-   Сообщения Telegram/WhatsApp и т.д. попадают на **шлюз**, а не на узлы.
-   Руководство по устранению неполадок: [/nodes/troubleshooting](./nodes/troubleshooting.md)

## Сопряжение + статус

**WS-узлы используют сопряжение устройств.** Узлы представляют идентификатор устройства во время `connect`; шлюз создает запрос на сопряжение устройства для `role: node`. Подтвердите через CLI устройств (или UI). Быстрый CLI:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

Примечания:

-   `nodes status` отмечает узел как **сопряженный**, когда роль сопряжения устройства включает `node`.
-   `node.pair.*` (CLI: `openclaw nodes pending/approve/reject`) — это отдельное хранилище сопряжения узлов, принадлежащее шлюзу; оно **не** управляет рукопожатием WS `connect`.

## Удаленный хост узла (system.run)

Используйте **хост узла**, когда ваш шлюз работает на одной машине, а вы хотите выполнять команды на другой. Модель по-прежнему общается с **шлюзом**; шлюз перенаправляет вызовы `exec` на **хост узла**, когда выбран `host=node`.

### Что где работает

-   **Хост шлюза**: получает сообщения, запускает модель, маршрутизирует вызовы инструментов.
-   **Хост узла**: выполняет `system.run`/`system.which` на машине узла.
-   **Подтверждения**: применяются на хосте узла через `~/.openclaw/exec-approvals.json`.

### Запуск хоста узла (в фоновом режиме)

На машине узла:

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

### Удаленный шлюз через SSH-туннель (привязка к loopback)

Если шлюз привязан к loopback (`gateway.bind=loopback`, по умолчанию в локальном режиме), удаленные хосты узлов не могут подключиться напрямую. Создайте SSH-туннель и укажите хосту узла на локальный конец туннеля. Пример (хост узла -> хост шлюза):

```bash
# Терминал A (держать запущенным): проброс локального порта 18790 -> шлюз 127.0.0.1:18789
ssh -N -L 18790:127.0.0.1:18789 user@gateway-host

# Терминал B: экспортируйте токен шлюза и подключитесь через туннель
export OPENCLAW_GATEWAY_TOKEN="<gateway-token>"
openclaw node run --host 127.0.0.1 --port 18790 --display-name "Build Node"
```

Примечания:

-   Токен — это `gateway.auth.token` из конфигурации шлюза (`~/.openclaw/openclaw.json` на хосте шлюза).
-   `openclaw node run` читает `OPENCLAW_GATEWAY_TOKEN` для аутентификации.

### Запуск хоста узла (служба)

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

### Сопряжение + имя

На хосте шлюза:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw nodes status
```

Варианты именования:

-   `--display-name` в `openclaw node run` / `openclaw node install` (сохраняется в `~/.openclaw/node.json` на узле).
-   `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (переопределение на шлюзе).

### Белый список команд

Подтверждения выполнения **привязаны к хосту узла**. Добавьте записи в белый список с шлюза:

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

Подтверждения хранятся на хосте узла в `~/.openclaw/exec-approvals.json`.

### Направление exec на узел

Настройте значения по умолчанию (конфигурация шлюза):

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

Или на сессию:

```bash
/exec host=node security=allowlist node=<id-or-name>
```

После настройки любой вызов `exec` с `host=node` выполняется на хосте узла (с учетом белого списка/подтверждений узла). Связанное:

-   [CLI хоста узла](./cli/node.md)
-   [Инструмент Exec](./tools/exec.md)
-   [Подтверждения Exec](./tools/exec-approvals.md)

## Вызов команд

Низкоуровневый (сырой RPC):

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

Существуют высокоуровневые помощники для распространенных рабочих процессов «дать агенту вложение MEDIA».

## Скриншоты (снимки canvas)

Если узел отображает Canvas (WebView), `canvas.snapshot` возвращает `{ format, base64 }`. Помощник CLI (записывает во временный файл и выводит `MEDIA:<путь>`):

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

### Управление Canvas

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

Примечания:

-   `canvas present` принимает URL или пути к локальным файлам (`--target`), а также опционально `--x/--y/--width/--height` для позиционирования.
-   `canvas eval` принимает встроенный JS (`--js`) или позиционный аргумент.

### A2UI (Canvas)

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

Примечания:

-   Поддерживается только A2UI v0.8 JSONL (v0.9/createSurface отклоняется).

## Фото + видео (камера узла)

Фото (`jpg`):

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # по умолчанию: обе стороны (2 строки MEDIA)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

Видеоклипы (`mp4`):

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

Примечания:

-   Узел должен быть **на переднем плане** для `canvas.*` и `camera.*` (вызовы в фоне возвращают `NODE_BACKGROUND_UNAVAILABLE`).
-   Длительность клипа ограничена (в настоящее время `<= 60s`), чтобы избежать слишком больших полезных нагрузок base64.
-   Android запросит разрешения `CAMERA`/`RECORD_AUDIO`, когда это возможно; отклоненные разрешения завершатся ошибкой `*_PERMISSION_REQUIRED`.

## Записи экрана (узлы)

Узлы предоставляют `screen.record` (mp4). Пример:

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Примечания:

-   `screen.record` требует, чтобы приложение узла было на переднем плане.
-   Android покажет системное окно захвата экрана перед записью.
-   Записи экрана ограничены до `<= 60s`.
-   `--no-audio` отключает захват микрофона (поддерживается на iOS/Android; macOS использует системный захват звука).
-   Используйте `--screen ` для выбора дисплея, когда доступно несколько экранов.

## Геолокация (узлы)

Узлы предоставляют `location.get`, когда геолокация включена в настройках. Помощник CLI:

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

Примечания:

-   Геолокация **по умолчанию отключена**.
-   «Всегда» требует системного разрешения; фоновое получение — по возможности.
-   Ответ включает широту/долготу, точность (метры) и временную метку.

## SMS (узлы Android)

Узлы Android могут предоставлять `sms.send`, когда пользователь предоставляет разрешение **SMS** и устройство поддерживает телефонию. Низкоуровневый вызов:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

Примечания:

-   Запрос разрешения должен быть принят на устройстве Android, прежде чем возможность будет объявлена.
-   Устройства только с Wi-Fi без телефонии не будут объявлять `sms.send`.

## Команды устройства Android и персональных данных

Узлы Android могут объявлять дополнительные семейства команд, когда включены соответствующие возможности. Доступные семейства:

-   `device.status`, `device.info`, `device.permissions`, `device.health`
-   `notifications.list`, `notifications.actions`
-   `photos.latest`
-   `contacts.search`, `contacts.add`
-   `calendar.events`, `calendar.add`
-   `motion.activity`, `motion.pedometer`
-   `app.update`

Примеры вызовов:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command device.status --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command notifications.list --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command photos.latest --params '{"limit":1}'
```

Примечания:

-   Команды движения зависят от доступных датчиков.
-   `app.update` зависит от разрешений и политик среды выполнения узла.

## Системные команды (хост узла / mac узел)

Узел macOS предоставляет `system.run`, `system.notify` и `system.execApprovals.get/set`. Безголовый хост узла предоставляет `system.run`, `system.which` и `system.execApprovals.get/set`. Примеры:

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Примечания:

-   `system.run` возвращает stdout/stderr/код выхода в полезной нагрузке.
-   `system.notify` учитывает состояние разрешения на уведомления в приложении macOS.
-   Неизвестные метаданные `platform` / `deviceFamily` узла используют консервативный белый список по умолчанию, исключающий `system.run` и `system.which`. Если вам намеренно нужны эти команды для неизвестной платформы, добавьте их явно через `gateway.nodes.allowCommands`.
-   `system.run` поддерживает `--cwd`, `--env KEY=VAL`, `--command-timeout` и `--needs-screen-recording`.
-   Для оболочек (`bash|sh|zsh ... -c/-lc`) значения `--env` в области запроса сокращаются до явного белого списка (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
-   Для решений «разрешить всегда» в режиме белого списка известные обертки диспетчеризации (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) сохраняют пути к внутренним исполняемым файлам вместо путей обертки. Если развертывание небезопасно, запись в белый список не сохраняется автоматически.
-   На хостах узлов Windows в режиме белого списка запуск через оболочку `cmd.exe /c` требует подтверждения (одной записи в белый список недостаточно для автоматического разрешения формы обертки).
-   `system.notify` поддерживает `--priority <passive|active|timeSensitive>` и `--delivery <system|overlay|auto>`.
-   Хосты узлов игнорируют переопределения `PATH` и удаляют опасные ключи запуска/оболочки (`DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`). Если вам нужны дополнительные записи PATH, настройте окружение службы хоста узла (или установите инструменты в стандартные места) вместо передачи `PATH` через `--env`.
-   В режиме узла macOS `system.run` контролируется подтверждениями выполнения в приложении macOS (Настройки → Подтверждения выполнения). Спросить/белый список/полный ведут себя так же, как на безголовом хосте узла; отклоненные запросы возвращают `SYSTEM_RUN_DENIED`.
-   На безголовом хосте узла `system.run` контролируется подтверждениями выполнения (`~/.openclaw/exec-approvals.json`).

## Привязка exec к узлу

Когда доступно несколько узлов, вы можете привязать exec к конкретному узлу. Это устанавливает узел по умолчанию для `exec host=node` (и может быть переопределено для каждого агента). Глобальное значение по умолчанию:

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

Переопределение для агента:

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Сбросьте, чтобы разрешить любой узел:

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

## Карта разрешений

Узлы могут включать карту `permissions` в `node.list` / `node.describe`, с ключами по имени разрешения (например, `screenRecording`, `accessibility`) и логическими значениями (`true` = предоставлено).

## Безголовый хост узла (кроссплатформенный)

OpenClaw может запускать **безголовый хост узла** (без UI), который подключается к WebSocket шлюза и предоставляет `system.run` / `system.which`. Это полезно на Linux/Windows или для запуска минимального узла вместе с сервером. Запустите его:

```bash
openclaw node run --host <gateway-host> --port 18789
```

Примечания:

-   Сопряжение все еще требуется (шлюз покажет запрос на сопряжение устройства).
-   Хост узла хранит свой идентификатор узла, токен, отображаемое имя и информацию о подключении к шлюзу в `~/.openclaw/node.json`.
-   Подтверждения выполнения применяются локально через `~/.openclaw/exec-approvals.json` (см. [Подтверждения Exec](./tools/exec-approvals.md)).
-   На macOS безголовый хост узла по умолчанию выполняет `system.run` локально. Установите `OPENCLAW_NODE_EXEC_HOST=app`, чтобы маршрутизировать `system.run` через хост выполнения приложения-компаньона; добавьте `OPENCLAW_NODE_EXEC_FALLBACK=0`, чтобы требовать хост приложения и завершаться с ошибкой, если он недоступен.
-   Добавьте `--tls` / `--tls-fingerprint`, когда WS шлюза использует TLS.

## Режим узла Mac

-   Приложение macOS в строке меню подключается к WS-серверу шлюза как узел (так что `openclaw nodes …` работает на этом Mac).
-   В удаленном режиме приложение открывает SSH-туннель для порта шлюза и подключается к `localhost`.

[Мониторинг аутентификации](./automation/auth-monitoring.md)[Устранение неполадок узлов](./nodes/troubleshooting.md)