title: "Руководство и справочник по CLI-командам OpenClaw Gateway"
description: "Узнайте, как запускать, опрашивать, управлять и обнаруживать WebSocket-сервер OpenClaw Gateway с помощью CLI-команд. Включает опции, управление службой и использование RPC."
keywords: ["openclaw gateway", "websocket сервер", "cli команды", "управление шлюзом", "обнаружение шлюза", "gateway rpc", "служба шлюза", "обнаружение bonjour"]
---

  CLI команды

  
# gateway

Gateway — это WebSocket-сервер OpenClaw (каналы, узлы, сессии, хуки). Подкоманды на этой странице находятся под `openclaw gateway …`. Связанная документация:

-   [/gateway/bonjour](../gateway/bonjour.md)
-   [/gateway/discovery](../gateway/discovery.md)
-   [/gateway/configuration](../gateway/configuration.md)

## Запуск Gateway

Запустите локальный процесс Gateway:

```bash
openclaw gateway
```

Альтернатива для запуска в foreground:

```bash
openclaw gateway run
```

Примечания:

-   По умолчанию Gateway отказывается запускаться, если в `~/.openclaw/openclaw.json` не установлен `gateway.mode=local`. Используйте `--allow-unconfigured` для ad-hoc/разработческих запусков.
-   Привязка за пределами loopback без аутентификации блокируется (защитный механизм).
-   `SIGUSR1` инициирует перезапуск внутри процесса, когда это разрешено (`commands.restart` включен по умолчанию; установите `commands.restart: false`, чтобы заблокировать ручной перезапуск, при этом применение/обновление конфигурации через инструменты gateway остаётся разрешённым).
-   Обработчики `SIGINT`/`SIGTERM` останавливают процесс gateway, но не восстанавливают какое-либо пользовательское состояние терминала. Если вы оборачиваете CLI в TUI или режим raw-ввода, восстановите терминал перед выходом.

### Опции

-   `--port `: порт WebSocket (по умолчанию берётся из конфигурации/окружения; обычно `18789`).
-   `--bind <loopback|lan|tailnet|auto|custom>`: режим привязки слушателя.
-   `--auth <token|password>`: переопределение режима аутентификации.
-   `--token `: переопределение токена (также устанавливает `OPENCLAW_GATEWAY_TOKEN` для процесса).
-   `--password `: переопределение пароля (также устанавливает `OPENCLAW_GATEWAY_PASSWORD` для процесса).
-   `--tailscale <off|serve|funnel>`: предоставить доступ к Gateway через Tailscale.
-   `--tailscale-reset-on-exit`: сбросить конфигурацию Tailscale serve/funnel при завершении работы.
-   `--allow-unconfigured`: разрешить запуск gateway без `gateway.mode=local` в конфигурации.
-   `--dev`: создать dev-конфигурацию + рабочее пространство, если они отсутствуют (пропускает BOOTSTRAP.md).
-   `--reset`: сбросить dev-конфигурацию + учётные данные + сессии + рабочее пространство (требует `--dev`).
-   `--force`: завершить любой существующий слушатель на выбранном порту перед запуском.
-   `--verbose`: подробные логи.
-   `--claude-cli-logs`: показывать в консоли только логи claude-cli (и включить его stdout/stderr).
-   `--ws-log <auto|full|compact>`: стиль логов websocket (по умолчанию `auto`).
-   `--compact`: алиас для `--ws-log compact`.
-   `--raw-stream`: логировать сырые события потока модели в jsonl.
-   `--raw-stream-path `: путь к jsonl-файлу для сырого потока.

## Опрос работающего Gateway

Все команды опроса используют WebSocket RPC. Режимы вывода:

-   По умолчанию: удобочитаемый формат (с цветами в TTY).
-   `--json`: машиночитаемый JSON (без стилей/спиннера).
-   `--no-color` (или `NO_COLOR=1`): отключить ANSI-цвета, сохраняя удобочитаемый формат.

Общие опции (где поддерживаются):

-   `--url `: URL WebSocket Gateway.
-   `--token `: токен Gateway.
-   `--password `: пароль Gateway.
-   `--timeout `: таймаут/бюджет времени (зависит от команды).
-   `--expect-final`: ждать «финального» ответа (вызовы агента).

Примечание: при установке `--url` CLI не возвращается к учётным данным из конфигурации или окружения. Передавайте `--token` или `--password` явно. Отсутствие явных учётных данных является ошибкой.

### gateway health

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### gateway status

`gateway status` показывает службу Gateway (launchd/systemd/schtasks) плюс опциональный RPC-проверку.

```bash
openclaw gateway status
openclaw gateway status --json
```

Опции:

-   `--url `: переопределить URL для проверки.
-   `--token `: токен для аутентификации при проверке.
-   `--password `: пароль для аутентификации при проверке.
-   `--timeout `: таймаут проверки (по умолчанию `10000`).
-   `--no-probe`: пропустить RPC-проверку (только просмотр службы).
-   `--deep`: сканировать также системные службы.

Примечания:

-   `gateway status` разрешает настроенные SecretRef для аутентификации при проверке, когда это возможно.
-   Если требуемый SecretRef для аутентификации не разрешён в пути выполнения этой команды, аутентификация при проверке может завершиться неудачей; передайте `--token`/`--password` явно или разрешите источник секрета сначала.

### gateway probe

`gateway probe` — это команда «отладки всего». Она всегда проверяет:

-   ваш настроенный удалённый gateway (если задан), и
-   localhost (loopback) **даже если настроен удалённый**.

Если доступно несколько gateway, она выводит все из них. Несколько gateway поддерживаются при использовании изолированных профилей/портов (например, rescue bot), но большинство установок всё ещё запускают один gateway.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

#### Удалённый доступ через SSH (аналогично Mac-приложению)

Режим «Remote over SSH» в macOS-приложении использует локальный проброс портов, чтобы удалённый gateway (который может быть привязан только к loopback) стал доступен по `ws://127.0.0.1:`. Эквивалент в CLI:

```bash
openclaw gateway probe --ssh user@gateway-host
```

Опции:

-   `--ssh `: `user@host` или `user@host:port` (порт по умолчанию `22`).
-   `--ssh-identity `: файл идентификации.
-   `--ssh-auto`: выбрать первый обнаруженный хост gateway в качестве цели SSH (только LAN/WAB).

Конфигурация (опционально, используется как значения по умолчанию):

-   `gateway.remote.sshTarget`
-   `gateway.remote.sshIdentity`

### gateway call &lt;method&gt;

Низкоуровневый помощник для RPC.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

## Управление службой Gateway

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Примечания:

-   `gateway install` поддерживает `--port`, `--runtime`, `--token`, `--force`, `--json`.
-   Когда для аутентификации по токену требуется токен и `gateway.auth.token` управляется через SecretRef, `gateway install` проверяет, что SecretRef может быть разрешён, но не сохраняет разрешённый токен в метаданных окружения службы.
-   Если для аутентификации по токену требуется токен и настроенный SecretRef токена не разрешён, установка завершается неудачей, а не сохраняет запасной вариант в открытом виде.
-   В выведенном режиме аутентификации, переменные окружения только для оболочки `OPENCLAW_GATEWAY_PASSWORD`/`CLAWDBOT_GATEWAY_PASSWORD` не ослабляют требования к токену при установке; используйте постоянную конфигурацию (`gateway.auth.password` или `env` в конфигурации) при установке управляемой службы.
-   Если настроены и `gateway.auth.token`, и `gateway.auth.password`, а `gateway.auth.mode` не установлен, установка блокируется до явного задания режима.
-   Команды управления жизненным циклом принимают `--json` для скриптования.

## Обнаружение gateway (Bonjour)

`gateway discover` сканирует маячки Gateway (`_openclaw-gw._tcp`).

-   Multicast DNS-SD: `local.`
-   Unicast DNS-SD (Wide-Area Bonjour): выберите домен (пример: `openclaw.internal.`) и настройте split DNS + DNS-сервер; см. [/gateway/bonjour](../gateway/bonjour.md)

Только gateway с включённым обнаружением Bonjour (по умолчанию) анонсируют маячок. Записи Wide-Area discovery включают (TXT):

-   `role` (подсказка о роли gateway)
-   `transport` (подсказка о транспорте, напр. `gateway`)
-   `gatewayPort` (порт WebSocket, обычно `18789`)
-   `sshPort` (порт SSH; по умолчанию `22`, если не указан)
-   `tailnetDns` (имя хоста MagicDNS, когда доступно)
-   `gatewayTls` / `gatewayTlsSha256` (TLS включён + отпечаток сертификата)
-   `cliPath` (опциональная подсказка для удалённых установок)

### gateway discover

```bash
openclaw gateway discover
```

Опции:

-   `--timeout `: таймаут на команду (просмотр/разрешение); по умолчанию `2000`.
-   `--json`: машиночитаемый вывод (также отключает стили/спиннер).

Примеры:

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```

[doctor](./doctor.md)[health](./health.md)