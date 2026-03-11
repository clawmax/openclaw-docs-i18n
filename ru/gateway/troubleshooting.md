

  Конфигурация и эксплуатация

  
# Устранение неполадок

Эта страница — подробное руководство. Начните с [/help/troubleshooting](../help/troubleshooting.md), если сначала хотите пройти быструю схему первичной диагностики.

## Лестница команд

Выполните эти команды первыми, в указанном порядке:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Ожидаемые признаки исправной работы:

-   `openclaw gateway status` показывает `Runtime: running` и `RPC probe: ok`.
-   `openclaw doctor` не сообщает о критических проблемах с конфигурацией или службой.
-   `openclaw channels status --probe` показывает подключённые/готовые каналы.

## Anthropic 429: требуется дополнительное использование для длинного контекста

Используйте это, когда в логах/ошибках встречается: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`.

```bash
openclaw logs --follow
openclaw models status
openclaw config get agents.defaults.models
```

Ищите:

-   У выбранной модели Anthropic Opus/Sonnet установлен параметр `params.context1m: true`.
-   Текущие учётные данные Anthropic не подходят для использования длинного контекста.
-   Запросы завершаются ошибкой только при длинных сессиях/запусках моделей, требующих пути к бета-версии с 1M контекстом.

Варианты исправления:

1.  Отключите `context1m` для этой модели, чтобы вернуться к обычному окну контекста.
2.  Используйте ключ API Anthropic с биллингом или включите Anthropic Extra Usage в учётной записи подписки.
3.  Настройте резервные модели, чтобы запуски продолжались, когда запросы Anthropic на длинный контекст отклоняются.

Связанные разделы:

-   [/providers/anthropic](../providers/anthropic.md)
-   [/reference/token-use](../reference/token-use.md)
-   [/help/faq#why-am-i-seeing-http-429-ratelimiterror-from-anthropic](../help/faq.md#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)

## Нет ответов

Если каналы работают, но никто не отвечает, проверьте маршрутизацию и политики перед повторным подключением чего-либо.

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw config get channels
openclaw logs --follow
```

Ищите:

-   Ожидающее сопряжение для отправителей в личных сообщениях.
-   Ограничения на упоминания в группах (`requireMention`, `mentionPatterns`).
-   Несоответствия в списках разрешённых каналов/групп.

Распространённые признаки:

-   `drop guild message (mention required` → сообщение в группе игнорируется до упоминания.
-   `pairing request` → отправителю требуется одобрение.
-   `blocked` / `allowlist` → отправитель/канал был отфильтрован политикой.

Связанные разделы:

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)
-   [/channels/groups](../channels/groups.md)

## Подключение к панели управления (UI)

Когда панель управления/UI управления не подключается, проверьте URL, режим аутентификации и предположения о безопасном контексте.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

Ищите:

-   Правильный URL для проверки и URL панели управления.
-   Несоответствие режима аутентификации/токена между клиентом и шлюзом.
-   Использование HTTP там, где требуется идентификация устройства.

Распространённые признаки:

-   `device identity required` → небезопасный контекст или отсутствует аутентификация устройства.
-   `device nonce required` / `device nonce mismatch` → клиент не завершает поток аутентификации устройства на основе запроса (`connect.challenge` + `device.nonce`).
-   `device signature invalid` / `device signature expired` → клиент подписал неверные данные (или устаревшую метку времени) для текущего рукопожатия.
-   `unauthorized` / цикл переподключения → несоответствие токена/пароля.
-   `gateway connect failed:` → неверный хост/порт/целевой URL.

Проверка миграции на аутентификацию устройства v2:

```bash
openclaw --version
openclaw doctor
openclaw gateway status
```

Если в логах видны ошибки nonce/подписи, обновите подключающийся клиент и убедитесь, что он:

1.  ожидает `connect.challenge`
2.  подписывает данные, привязанные к запросу
3.  отправляет `connect.params.device.nonce` с тем же одноразовым номером (nonce) из запроса

Связанные разделы:

-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/remote](./remote.md)

## Служба шлюза не запущена

Используйте это, когда служба установлена, но процесс не остаётся запущенным.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

Ищите:

-   `Runtime: stopped` с указаниями на причину завершения.
-   Несоответствие конфигурации службы (`Config (cli)` vs `Config (service)`).
-   Конфликты портов/слушателей.

Распространённые признаки:

-   `Gateway start blocked: set gateway.mode=local` → локальный режим шлюза не включен. Исправление: установите `gateway.mode="local"` в вашей конфигурации (или выполните `openclaw configure`). Если вы запускаете OpenClaw через Podman от имени выделенного пользователя `openclaw`, конфигурация находится по пути `~openclaw/.openclaw/openclaw.json`.
-   `refusing to bind gateway ... without auth` → привязка не к loopback-адресу без токена/пароля.
-   `another gateway instance is already listening` / `EADDRINUSE` → конфликт портов.

Связанные разделы:

-   [/gateway/background-process](./background-process.md)
-   [/gateway/configuration](./configuration.md)
-   [/gateway/doctor](./doctor.md)

## Канал подключён, но сообщения не передаются

Если состояние канала — подключён, но поток сообщений не работает, сосредоточьтесь на политиках, разрешениях и специфичных для канала правилах доставки.

```bash
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

Ищите:

-   Политика для личных сообщений (`pairing`, `allowlist`, `open`, `disabled`).
-   Списки разрешённых групп и требования к упоминаниям.
-   Отсутствующие разрешения/области действия API канала.

Распространённые признаки:

-   `mention required` → сообщение проигнорировано политикой упоминаний в группе.
-   `pairing` / следы ожидания одобрения → отправитель не утверждён.
-   `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → проблема с аутентификацией/разрешениями канала.

Связанные разделы:

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/whatsapp](../channels/whatsapp.md)
-   [/channels/telegram](../channels/telegram.md)
-   [/channels/discord](../channels/discord.md)

## Cron и доставка сердцебиения

Если cron или сердцебиение не сработали или не были доставлены, сначала проверьте состояние планировщика, затем цель доставки.

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

Ищите:

-   Cron включён и присутствует следующее время пробуждения.
-   Статус истории запусков заданий (`ok`, `skipped`, `error`).
-   Причины пропуска сердцебиения (`quiet-hours`, `requests-in-flight`, `alerts-disabled`).

Распространённые признаки:

-   `cron: scheduler disabled; jobs will not run automatically` → cron отключён.
-   `cron: timer tick failed` → сбой такта планировщика; проверьте ошибки файлов/логов/среды выполнения.
-   `heartbeat skipped` с `reason=quiet-hours` → вне окна активных часов.
-   `heartbeat: unknown accountId` → неверный идентификатор учётной записи для цели доставки сердцебиения.
-   `heartbeat skipped` с `reason=dm-blocked` → цель сердцебиения определена как назначение в стиле личных сообщений, в то время как `agents.defaults.heartbeat.directPolicy` (или переопределение для конкретного агента) установлено в `block`.

Связанные разделы:

-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/automation/cron-jobs](../automation/cron-jobs.md)
-   [/gateway/heartbeat](./heartbeat.md)

## Сопряжённый узел: сбой инструментов

Если узел сопряжён, но инструменты не работают, изолируйте состояние переднего плана, разрешений и утверждения.

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

Ищите:

-   Узел в сети с ожидаемыми возможностями.
-   Предоставленные разрешения ОС для камеры/микрофона/геолокации/экрана.
-   Состояние утверждений на выполнение и списка разрешённых команд.

Распространённые признаки:

-   `NODE_BACKGROUND_UNAVAILABLE` → приложение узла должно быть на переднем плане.
-   `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → отсутствует разрешение ОС.
-   `SYSTEM_RUN_DENIED: approval required` → ожидание утверждения на выполнение.
-   `SYSTEM_RUN_DENIED: allowlist miss` → команда заблокирована списком разрешённых.

Связанные разделы:

-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/nodes/index](../nodes/index.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

## Сбой инструмента браузера

Используйте это, когда действия инструмента браузера завершаются ошибкой, даже если сам шлюз работает исправно.

```bash
openclaw browser status
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

Ищите:

-   Действительный путь к исполняемому файлу браузера.
-   Доступность профиля CDP.
-   Подключение вкладки ретранслятора расширения для `profile="chrome"`.

Распространённые признаки:

-   `Failed to start Chrome CDP on port` → не удалось запустить процесс браузера.
-   `browser.executablePath not found` → настроенный путь недействителен.
-   `Chrome extension relay is running, but no tab is connected` → ретранслятор расширения не подключён.
-   `Browser attachOnly is enabled ... not reachable` → у профиля только для подключения нет доступной цели.

Связанные разделы:

-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)
-   [/tools/browser](../tools/browser.md)

## Если вы обновились и что-то внезапно сломалось

Большинство поломок после обновления — это расхождение конфигурации или более строгие настройки по умолчанию, которые теперь применяются.

### 1) Изменилось поведение аутентификации и переопределения URL

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

Что проверить:

-   Если `gateway.mode=remote`, вызовы CLI могут нацеливаться на удалённый шлюз, в то время как ваша локальная служба в порядке.
-   Явные вызовы с `--url` не возвращаются к сохранённым учётным данным.

Распространённые признаки:

-   `gateway connect failed:` → неверная целевая URL-адреса.
-   `unauthorized` → конечная точка доступна, но неверная аутентификация.

### 2) Защитные ограничения на привязку и аутентификацию стали строже

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

Что проверить:

-   Привязки не к loopback-адресам (`lan`, `tailnet`, `custom`) требуют настройки аутентификации.
-   Старые ключи, такие как `gateway.token`, не заменяют `gateway.auth.token`.

Распространённые признаки:

-   `refusing to bind gateway ... without auth` → несоответствие привязки и аутентификации.
-   `RPC probe: failed` при работающем runtime → шлюз жив, но недоступен с текущей аутентификацией/URL.

### 3) Изменилось состояние сопряжения и идентификации устройств

```bash
openclaw devices list
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
openclaw doctor
```

Что проверить:

-   Ожидающие утверждения устройств для панели управления/узлов.
-   Ожидающие утверждения сопряжения для личных сообщений после изменения политик или идентификаторов.

Распространённые признаки:

-   `device identity required` → аутентификация устройства не пройдена.
-   `pairing required` → отправитель/устройство должны быть утверждены.

Если конфигурация службы и среда выполнения всё ещё расходятся после проверок, переустановите метаданные службы из того же каталога профиля/состояния:

```bash
openclaw gateway install --force
openclaw gateway restart
```

Связанные разделы:

-   [/gateway/pairing](./pairing.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/background-process](./background-process.md)

[Несколько шлюзов](./multiple-gateways.md)[Безопасность](./security.md)