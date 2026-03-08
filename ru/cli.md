title: "Справочник по OpenClaw CLI: Команды, флаги и руководство по использованию"
description: "Полный справочник по командам OpenClaw CLI, глобальным флагам, стилизации вывода и использованию. Научитесь настраивать, конфигурировать и управлять агентами, плагинами, безопасностью и памятью."
keywords: ["openclaw cli", "команды cli", "справочник cli", "команды агента", "настройка cli", "конфигурация cli", "инструменты разработчика", "интерфейс командной строки"]
---

  Команды CLI

  
# Справочник CLI

На этой странице описано текущее поведение CLI. Если команды изменятся, обновите эту документацию.

## Страницы команд

-   [`setup`](./cli/setup.md)
-   [`onboard`](./cli/onboard.md)
-   [`configure`](./cli/configure.md)
-   [`config`](./cli/config.md)
-   [`completion`](./cli/completion.md)
-   [`doctor`](./cli/doctor.md)
-   [`dashboard`](./cli/dashboard.md)
-   [`reset`](./cli/reset.md)
-   [`uninstall`](./cli/uninstall.md)
-   [`update`](./cli/update.md)
-   [`message`](./cli/message.md)
-   [`agent`](./cli/agent.md)
-   [`agents`](./cli/agents.md)
-   [`acp`](./cli/acp.md)
-   [`status`](./cli/status.md)
-   [`health`](./cli/health.md)
-   [`sessions`](./cli/sessions.md)
-   [`gateway`](./cli/gateway.md)
-   [`logs`](./cli/logs.md)
-   [`system`](./cli/system.md)
-   [`models`](./cli/models.md)
-   [`memory`](./cli/memory.md)
-   [`directory`](./cli/directory.md)
-   [`nodes`](./cli/nodes.md)
-   [`devices`](./cli/devices.md)
-   [`node`](./cli/node.md)
-   [`approvals`](./cli/approvals.md)
-   [`sandbox`](./cli/sandbox.md)
-   [`tui`](./cli/tui.md)
-   [`browser`](./cli/browser.md)
-   [`cron`](./cli/cron.md)
-   [`dns`](./cli/dns.md)
-   [`docs`](./cli/docs.md)
-   [`hooks`](./cli/hooks.md)
-   [`webhooks`](./cli/webhooks.md)
-   [`pairing`](./cli/pairing.md)
-   [`qr`](./cli/qr.md)
-   [`plugins`](./cli/plugins.md) (команды плагинов)
-   [`channels`](./cli/channels.md)
-   [`security`](./cli/security.md)
-   [`secrets`](./cli/secrets.md)
-   [`skills`](./cli/skills.md)
-   [`daemon`](./cli/daemon.md) (устаревший псевдоним для команд службы gateway)
-   [`clawbot`](./cli/clawbot.md) (устаревшее пространство имён)
-   [`voicecall`](./cli/voicecall.md) (плагин; если установлен)

## Глобальные флаги

-   `--dev`: изолировать состояние в `~/.openclaw-dev` и сдвинуть порты по умолчанию.
-   `--profile `: изолировать состояние в `~/.openclaw-`.
-   `--no-color`: отключить цвета ANSI.
-   `--update`: сокращение для `openclaw update` (только для установок из исходного кода).
-   `-V`, `--version`, `-v`: вывести версию и выйти.

## Стилизация вывода

-   Цвета ANSI и индикаторы прогресса отображаются только в TTY-сессиях.
-   Гиперссылки OSC-8 отображаются как кликабельные ссылки в поддерживающих терминалах; в противном случае мы возвращаемся к простым URL.
-   `--json` (и `--plain`, где поддерживается) отключает стилизацию для чистого вывода.
-   `--no-color` отключает стилизацию ANSI; также учитывается `NO_COLOR=1`.
-   Долго выполняющиеся команды показывают индикатор прогресса (OSC 9;4, когда поддерживается).

## Цветовая палитра

OpenClaw использует палитру "lobster" для вывода CLI.

-   `accent` (#FF5A2D): заголовки, метки, основные выделения.
-   `accentBright` (#FF7A3D): имена команд, акценты.
-   `accentDim` (#D14A22): второстепенный выделенный текст.
-   `info` (#FF8A5B): информационные значения.
-   `success` (#2FBF71): состояния успеха.
-   `warn` (#FFB020): предупреждения, запасные варианты, внимание.
-   `error` (#E23D2D): ошибки, сбои.
-   `muted` (#8B7F77): снижение акцента, метаданные.

Источник истины для палитры: `src/terminal/palette.ts` (также известна как "lobster seam").

## Дерево команд

```bash
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  completion
  doctor
  dashboard
  security
    audit
  secrets
    reload
    migrate
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  directory
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  daemon
    status
    install
    uninstall
    start
    stop
    restart
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  qr
  clawbot
    qr
  docs
  dns
    setup
  tui
```

Примечание: плагины могут добавлять дополнительные команды верхнего уровня (например, `openclaw voicecall`).

## Безопасность

-   `openclaw security audit` — аудит конфигурации и локального состояния на предмет распространённых проблем безопасности.
-   `openclaw security audit --deep` — проверка работоспособности Gateway в реальном времени (best-effort).
-   `openclaw security audit --fix` — ужесточение безопасных настроек по умолчанию и chmod для состояния/конфигурации.

## Секреты

-   `openclaw secrets reload` — повторное разрешение ссылок и атомарная замена снимка состояния среды выполнения.
-   `openclaw secrets audit` — поиск остатков в открытом виде, неразрешённых ссылок и отклонений в приоритетах.
-   `openclaw secrets configure` — интерактивный помощник для настройки провайдера + сопоставления SecretRef + предварительная проверка/применение.
-   `openclaw secrets apply --from <plan.json>` — применить ранее сгенерированный план (поддерживается `--dry-run`).

## Плагины

Управление расширениями и их конфигурацией:

-   `openclaw plugins list` — обнаружение плагинов (используйте `--json` для машинного вывода).
-   `openclaw plugins info ` — показать детали для плагина.
-   `openclaw plugins install <path|.tgz|npm-spec>` — установить плагин (или добавить путь к плагину в `plugins.load.paths`).
-   `openclaw plugins enable ` / `disable ` — переключить `plugins.entries..enabled`.
-   `openclaw plugins doctor` — сообщить об ошибках загрузки плагинов.

Большинство изменений плагинов требуют перезапуска gateway. См. [/plugin](./tools/plugin.md).

## Память

Векторный поиск по `MEMORY.md` + `memory/*.md`:

-   `openclaw memory status` — показать статистику индекса.
-   `openclaw memory index` — переиндексировать файлы памяти.
-   `openclaw memory search ""` (или `--query ""`) — семантический поиск по памяти.

## Слэш-команды в чате

Сообщения в чате поддерживают команды `/...` (текстовые и нативные). См. [/tools/slash-commands](./tools/slash-commands.md). Основное:

-   `/status` для быстрой диагностики.
-   `/config` для сохранённых изменений конфигурации.
-   `/debug` для переопределений конфигурации только во время выполнения (в памяти, не на диске; требует `commands.debug: true`).

## Настройка + первоначальная настройка

### setup

Инициализировать конфигурацию и рабочее пространство. Опции:

-   `--workspace `: путь к рабочему пространству агента (по умолчанию `~/.openclaw/workspace`).
-   `--wizard`: запустить мастер первоначальной настройки.
-   `--non-interactive`: запустить мастер без запросов.
-   `--mode <local|remote>`: режим мастера.
-   `--remote-url `: URL удалённого Gateway.
-   `--remote-token `: токен удалённого Gateway.

Мастер запускается автоматически, когда присутствуют любые флаги мастера (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

### onboard

Интерактивный мастер для настройки gateway, рабочего пространства и навыков. Опции:

-   `--workspace `
-   `--reset` (сбросить конфигурацию + учётные данные + сессии перед запуском мастера)
-   `--reset-scope <config|config+creds+sessions|full>` (по умолчанию `config+creds+sessions`; используйте `full` для также удаления рабочего пространства)
-   `--non-interactive`
-   `--mode <local|remote>`
-   `--flow <quickstart|advanced|manual>` (manual — это псевдоним для advanced)
-   `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|moonshot-api-key-cn|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|mistral-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|custom-api-key|skip>`
-   `--token-provider ` (неинтерактивно; используется с `--auth-choice token`)
-   `--token ` (неинтерактивно; используется с `--auth-choice token`)
-   `--token-profile-id ` (неинтерактивно; по умолчанию: `:manual`)
-   `--token-expires-in ` (неинтерактивно; например, `365d`, `12h`)
-   `--secret-input-mode <plaintext|ref>` (по умолчанию `plaintext`; используйте `ref` для хранения ссылок на переменные окружения провайдера по умолчанию вместо ключей в открытом виде)
-   `--anthropic-api-key `
-   `--openai-api-key `
-   `--mistral-api-key `
-   `--openrouter-api-key `
-   `--ai-gateway-api-key `
-   `--moonshot-api-key `
-   `--kimi-code-api-key `
-   `--gemini-api-key `
-   `--zai-api-key `
-   `--minimax-api-key `
-   `--opencode-zen-api-key `
-   `--custom-base-url ` (неинтерактивно; используется с `--auth-choice custom-api-key`)
-   `--custom-model-id ` (неинтерактивно; используется с `--auth-choice custom-api-key`)
-   `--custom-api-key ` (неинтерактивно; опционально; используется с `--auth-choice custom-api-key`; переходит к `CUSTOM_API_KEY`, если опущено)
-   `--custom-provider-id ` (неинтерактивно; опциональный идентификатор пользовательского провайдера)
-   `--custom-compatibility <openai|anthropic>` (неинтерактивно; опционально; по умолчанию `openai`)
-   `--gateway-port `
-   `--gateway-bind <loopback|lan|tailnet|auto|custom>`
-   `--gateway-auth <token|password>`
-   `--gateway-token `
-   `--gateway-token-ref-env ` (неинтерактивно; сохранить `gateway.auth.token` как env SecretRef; требует, чтобы эта переменная окружения была установлена; нельзя комбинировать с `--gateway-token`)
-   `--gateway-password `
-   `--remote-url `
-   `--remote-token `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--install-daemon`
-   `--no-install-daemon` (псевдоним: `--skip-daemon`)
-   `--daemon-runtime <node|bun>`
-   `--skip-channels`
-   `--skip-skills`
-   `--skip-health`
-   `--skip-ui`
-   `--node-manager <npm|pnpm|bun>` (рекомендуется pnpm; bun не рекомендуется для среды выполнения Gateway)
-   `--json`

### configure

Интерактивный мастер конфигурации (модели, каналы, навыки, gateway).

### config

Неинтерактивные помощники конфигурации (get/set/unset/file/validate). Запуск `openclaw config` без подкоманды запускает мастер. Подкоманды:

-   `config get `: вывести значение конфигурации (путь с точками/скобками).
-   `config set  `: установить значение (JSON5 или сырая строка).
-   `config unset `: удалить значение.
-   `config file`: вывести путь к активному файлу конфигурации.
-   `config validate`: проверить текущую конфигурацию на соответствие схеме без запуска gateway.
-   `config validate --json`: вывести машинно-читаемый JSON.

### doctor

Проверки работоспособности + быстрые исправления (конфигурация + gateway + устаревшие службы). Опции:

-   `--no-workspace-suggestions`: отключить подсказки по памяти рабочего пространства.
-   `--yes`: принимать значения по умолчанию без запросов (headless).
-   `--non-interactive`: пропустить запросы; применять только безопасные миграции.
-   `--deep`: сканировать системные службы на наличие дополнительных установок gateway.

## Помощники по каналам

### channels

Управление учётными записями каналов чата (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (плагин)/Signal/iMessage/MS Teams). Подкоманды:

-   `channels list`: показать настроенные каналы и профили аутентификации.
-   `channels status`: проверить доступность gateway и работоспособность каналов (`--probe` выполняет дополнительные проверки; используйте `openclaw health` или `openclaw status --deep` для проверок работоспособности gateway).
-   Совет: `channels status` выводит предупреждения с предлагаемыми исправлениями, когда может обнаружить распространённые ошибки конфигурации (затем указывает на `openclaw doctor`).
-   `channels logs`: показать недавние логи каналов из файла логов gateway.
-   `channels add`: настройка в стиле мастера, если флаги не переданы; флаги переключают в неинтерактивный режим.
    -   При добавлении нестандартной учётной записи в канал, который всё ещё использует конфигурацию верхнего уровня для одной учётной записи, OpenClaw перемещает значения, ограниченные учётной записью, в `channels..accounts.default` перед записью новой учётной записи.
    -   Неинтерактивный `channels add` не создаёт/обновляет привязки автоматически; привязки только для канала продолжают соответствовать учётной записи по умолчанию.
-   `channels remove`: отключить по умолчанию; передайте `--delete`, чтобы удалить записи конфигурации без запросов.
-   `channels login`: интерактивный вход в канал (только WhatsApp Web).
-   `channels logout`: выйти из сессии канала (если поддерживается).

Общие опции:

-   `--channel `: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
-   `--account `: идентификатор учётной записи канала (по умолчанию `default`)
-   `--name `: отображаемое имя для учётной записи

Опции `channels login`:

-   `--channel ` (по умолчанию `whatsapp`; поддерживает `whatsapp`/`web`)
-   `--account `
-   `--verbose`

Опции `channels logout`:

-   `--channel ` (по умолчанию `whatsapp`)
-   `--account `

Опции `channels list`:

-   `--no-usage`: пропустить снимки использования/квот провайдеров моделей (только OAuth/API-backed).
-   `--json`: вывод JSON (включает использование, если не установлен `--no-usage`).

Опции `channels logs`:

-   `--channel <name|all>` (по умолчанию `all`)
-   `--lines ` (по умолчанию `200`)
-   `--json`

Подробнее: [/concepts/oauth](./concepts/oauth.md) Примеры:

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

### skills

Список и проверка доступных навыков, а также информации об их готовности. Подкоманды:

-   `skills list`: список навыков (по умолчанию, если подкоманда не указана).
-   `skills info `: показать детали для одного навыка.
-   `skills check`: сводка о готовых и отсутствующих требованиях.

Опции:

-   `--eligible`: показывать только готовые навыки.
-   `--json`: вывод JSON (без стилизации).
-   `-v`, `--verbose`: включать детали отсутствующих требований.

Совет: используйте `npx clawhub` для поиска, установки и синхронизации навыков.

### pairing

Утверждение запросов на прямое подключение (DM) через каналы. Подкоманды:

-   `pairing list [channel] [--channel ] [--account ] [--json]`
-   `pairing approve   [--account ] [--notify]`
-   `pairing approve --channel  [--account ]  [--notify]`

### devices

Управление записями сопряжения устройств gateway и токенами устройств по ролям. Подкоманды:

-   `devices list [--json]`
-   `devices approve [requestId] [--latest]`
-   `devices reject `
-   `devices remove `
-   `devices clear --yes [--pending]`
-   `devices rotate --device  --role  [--scope <scope...>]`
-   `devices revoke --device  --role `

### webhooks gmail

Настройка и запуск хука Gmail Pub/Sub. См. [/automation/gmail-pubsub](./automation/gmail-pubsub.md). Подкоманды:

-   `webhooks gmail setup` (требует `--account `; поддерживает `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
-   `webhooks gmail run` (переопределения во время выполнения для тех же флагов)

### dns setup

Помощник по DNS для широковещательного обнаружения (CoreDNS + Tailscale). См. [/gateway/discovery](./gateway/discovery.md). Опции:

-   `--apply`: установить/обновить конфигурацию CoreDNS (требует sudo; только macOS).

## Обмен сообщениями + агент

### message

Унифицированная отправка сообщений + действия с каналами. См.: [/cli/message](./cli/message.md) Подкоманды:

-   `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
-   `message thread <create|list|reply>`
-   `message emoji <list|upload>`
-   `message sticker <send|upload>`
-   `message role <info|add|remove>`
-   `message channel <info|list>`
-   `message member info`
-   `message voice status`
-   `message event <list|create>`

Примеры:

-   `openclaw message send --target +15555550123 --message "Привет"`
-   `openclaw message poll --channel discord --target channel:123 --poll-question "Закуска?" --poll-option Pizza --poll-option Sushi`

### agent

Выполнить один ход агента через Gateway (или встроенный `--local`). Обязательно:

-   `--message `

Опции:

-   `--to ` (для ключа сессии и опциональной доставки)
-   `--session-id `
-   `--thinking <off|minimal|low|medium|high|xhigh>` (только модели GPT-5.2 + Codex)
-   `--verbose <on|full|off>`
-   `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
-   `--local`
-   `--deliver`
-   `--json`
-   `--timeout `

### agents

Управление изолированными агентами (рабочие пространства + аутентификация + маршрутизация).

#### agents list

Список настроенных агентов. Опции:

-   `--json`
-   `--bindings`

#### agents add \[name\]

Добавить нового изолированного агента. Запускает управляемый мастер, если не переданы флаги (или `--non-interactive`); `--workspace` требуется в неинтерактивном режиме. Опции:

-   `--workspace `
-   `--model `
-   `--agent-dir `
-   `--bind <channel[:accountId]>` (повторяемый)
-   `--non-interactive`
-   `--json`

Спецификации привязок используют `channel[:accountId]`. Когда `accountId` опущен, OpenClaw может разрешить область учётной записи через значения по умолчанию канала/хуки плагинов; в противном случае это привязка канала без явной области учётной записи.

#### agents bindings

Список привязок маршрутизации. Опции:

-   `--agent `
-   `--json`

#### agents bind

Добавить привязки маршрутизации для агента. Опции:

-   `--agent `
-   `--bind <channel[:accountId]>` (повторяемый)
-   `--json`

#### agents unbind

Удалить привязки маршрутизации для агента. Опции:

-   `--agent `
-   `--bind <channel[:accountId]>` (повторяемый)
-   `--all`
-   `--json`

#### agents delete &lt;id&gt;

Удалить агента и очистить его рабочее пространство + состояние. Опции:

-   `--force`
-   `--json`

### acp

Запустить мост ACP, который соединяет IDE с Gateway. См. [`acp`](./cli/acp.md) для полного списка опций и примеров.

### status

Показать работоспособность связанных сессий и недавних получателей. Опции:

-   `--json`
-   `--all` (полная диагностика; только для чтения, можно вставить)
-   `--deep` (проверка каналов)
-   `--usage` (показать использование/квоты провайдеров моделей)
-   `--timeout `
-   `--verbose`
-   `--debug` (псевдоним для `--verbose`)

Примечания:

-   Обзор включает статус Gateway + службы узла, когда они доступны.

### Отслеживание использования

OpenClaw может показывать использование/квоты провайдеров, когда доступны учётные данные OAuth/API. Показывает:

-   `/status` (добавляет короткую строку использования провайдера, когда доступно)
-   `openclaw status --usage` (выводит полную разбивку по провайдерам)
-   Строка меню macOS (раздел Usage в Context)

Примечания:

-   Данные поступают напрямую от конечных точек использования провайдеров (без оценок).
-   Провайдеры: Anthropic, GitHub Copilot, OpenAI Codex OAuth, а также Gemini CLI/Antigravity, когда эти плагины провайдеров включены.
-   Если нет соответствующих учётных данных, использование скрыто.
-   Подробности: см. [Отслеживание использования](./concepts/usage-tracking.md).

### health

Получить информацию о работоспособности от запущенного Gateway. Опции:

-   `--json`
-   `--timeout `
-   `--verbose`

### sessions

Список сохранённых сессий разговоров. Опции:

-   `--json`
-   `--verbose`
-   `--store `
-   `--active `

## Сброс / Удаление

### reset

Сбросить локальную конфигурацию/состояние (CLI остаётся установленным). Опции:

-   `--scope <config|config+creds+sessions|full>`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

Примечания:

-   `--non-interactive` требует `--scope` и `--yes`.

### uninstall

Удалить службу gateway + локальные данные (CLI остаётся). Опции:

-   `--service`
-   `--state`
-   `--workspace`
-   `--app`
-   `--all`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

Примечания:

-   `--non-interactive` требует `--yes` и явных областей (или `--all`).

## Gateway

### gateway

Запустить WebSocket Gateway. Опции:

-   `--port `
-   `--bind <loopback|tailnet|lan|auto|custom>`
-   `--token `
-   `--auth <token|password>`
-   `--password `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--allow-unconfigured`
-   `--dev`
-   `--reset` (сбросить dev-конфигурацию + учётные данные + сессии + рабочее пространство)
-   `--force` (убить существующий слушатель на порту)
-   `--verbose`
-   `--claude-cli-logs`
-   `--ws-log <auto|full|compact>`
-   `--compact` (псевдоним для `--ws-log compact`)
-   `--raw-stream`
-   `--raw-stream-path `

### gateway service

Управление службой Gateway (launchd/systemd/schtasks). Подкоманды:

-   `gateway status` (по умолчанию проверяет RPC Gateway)
-   `gateway install` (установка службы)
-   `gateway uninstall`
-   `gateway start`
-   `gateway stop`
-   `gateway restart`

Примечания:

-   `gateway status` по умолчанию проверяет RPC Gateway, используя разрешённый порт/конфигурацию службы (переопределить с помощью `--url/--token/--password`).
-   `gateway status` поддерживает `--no-probe`, `--deep` и `--json` для скриптов.
-   `gateway status` также показывает устаревшие или дополнительные службы gateway, когда может их обнаружить (`--deep` добавляет сканирование на уровне системы). Службы OpenClaw с именем профиля рассматриваются как первоклассные и не помечаются как "дополнительные".
-   `gateway status` выводит, какой путь конфигурации использует CLI, и какую конфигурацию, вероятно, использует служба (env службы), а также разрешённый целевой URL для проверки.
-   `gateway install|uninstall|start|stop|restart` поддерживают `--json` для скриптов (вывод по умолчанию остаётся удобным для человека).
-   `gateway install` по умолчанию использует среду выполнения Node; bun **не рекомендуется** (ошибки WhatsApp/Telegram).
-   Опции `gateway install`: `--port`, `--runtime`, `--token`, `--force`, `--json`.

### logs

Просмотр файловых логов Gateway через RPC. Примечания:

-   TTY-сессии отображают цветное структурированное представление; не-TTY переходит к простому тексту.
-   `--json` выводит построчный JSON (одно событие лога на строку).

Пример