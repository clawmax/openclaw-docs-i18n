

  CLI commands

  
# CLI Reference

This page describes the current CLI behavior. If commands change, update this doc.

## Command pages

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
-   [`plugins`](./cli/plugins.md) (plugin commands)
-   [`channels`](./cli/channels.md)
-   [`security`](./cli/security.md)
-   [`secrets`](./cli/secrets.md)
-   [`skills`](./cli/skills.md)
-   [`daemon`](./cli/daemon.md) (legacy alias for gateway service commands)
-   [`clawbot`](./cli/clawbot.md) (legacy alias namespace)
-   [`voicecall`](./cli/voicecall.md) (plugin; if installed)

## Global flags

-   `--dev`: isolate state under `~/.openclaw-dev` and shift default ports.
-   `--profile `: isolate state under `~/.openclaw-`.
-   `--no-color`: disable ANSI colors.
-   `--update`: shorthand for `openclaw update` (source installs only).
-   `-V`, `--version`, `-v`: print version and exit.

## Output styling

-   ANSI colors and progress indicators only render in TTY sessions.
-   OSC-8 hyperlinks render as clickable links in supported terminals; otherwise we fall back to plain URLs.
-   `--json` (and `--plain` where supported) disables styling for clean output.
-   `--no-color` disables ANSI styling; `NO_COLOR=1` is also respected.
-   Long-running commands show a progress indicator (OSC 9;4 when supported).

## Color palette

OpenClaw uses a lobster palette for CLI output.

-   `accent` (#FF5A2D): headings, labels, primary highlights.
-   `accentBright` (#FF7A3D): command names, emphasis.
-   `accentDim` (#D14A22): secondary highlight text.
-   `info` (#FF8A5B): informational values.
-   `success` (#2FBF71): success states.
-   `warn` (#FFB020): warnings, fallbacks, attention.
-   `error` (#E23D2D): errors, failures.
-   `muted` (#8B7F77): de-emphasis, metadata.

Palette source of truth: `src/terminal/palette.ts` (aka “lobster seam”).

## Command tree

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

Note: plugins can add additional top-level commands (for example `openclaw voicecall`).

## Security

-   `openclaw security audit` — audit config + local state for common security foot-guns.
-   `openclaw security audit --deep` — best-effort live Gateway probe.
-   `openclaw security audit --fix` — tighten safe defaults and chmod state/config.

## Secrets

-   `openclaw secrets reload` — re-resolve refs and atomically swap the runtime snapshot.
-   `openclaw secrets audit` — scan for plaintext residues, unresolved refs, and precedence drift.
-   `openclaw secrets configure` — interactive helper for provider setup + SecretRef mapping + preflight/apply.
-   `openclaw secrets apply --from <plan.json>` — apply a previously generated plan (`--dry-run` supported).

## Plugins

Manage extensions and their config:

-   `openclaw plugins list` — discover plugins (use `--json` for machine output).
-   `openclaw plugins info ` — show details for a plugin.
-   `openclaw plugins install <path|.tgz|npm-spec>` — install a plugin (or add a plugin path to `plugins.load.paths`).
-   `openclaw plugins enable ` / `disable ` — toggle `plugins.entries..enabled`.
-   `openclaw plugins doctor` — report plugin load errors.

Most plugin changes require a gateway restart. See [/plugin](./tools/plugin.md).

## Memory

Vector search over `MEMORY.md` + `memory/*.md`:

-   `openclaw memory status` — show index stats.
-   `openclaw memory index` — reindex memory files.
-   `openclaw memory search ""` (or `--query ""`) — semantic search over memory.

## Chat slash commands

Chat messages support `/...` commands (text and native). See [/tools/slash-commands](./tools/slash-commands.md). Highlights:

-   `/status` for quick diagnostics.
-   `/config` for persisted config changes.
-   `/debug` for runtime-only config overrides (memory, not disk; requires `commands.debug: true`).

## Setup + onboarding

### setup

Initialize config + workspace. Options:

-   `--workspace `: agent workspace path (default `~/.openclaw/workspace`).
-   `--wizard`: run the onboarding wizard.
-   `--non-interactive`: run wizard without prompts.
-   `--mode <local|remote>`: wizard mode.
-   `--remote-url `: remote Gateway URL.
-   `--remote-token `: remote Gateway token.

Wizard auto-runs when any wizard flags are present (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

### onboard

Interactive wizard to set up gateway, workspace, and skills. Options:

-   `--workspace `
-   `--reset` (reset config + credentials + sessions before wizard)
-   `--reset-scope <config|config+creds+sessions|full>` (default `config+creds+sessions`; use `full` to also remove workspace)
-   `--non-interactive`
-   `--mode <local|remote>`
-   `--flow <quickstart|advanced|manual>` (manual is an alias for advanced)
-   `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|moonshot-api-key-cn|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|mistral-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|custom-api-key|skip>`
-   `--token-provider ` (non-interactive; used with `--auth-choice token`)
-   `--token ` (non-interactive; used with `--auth-choice token`)
-   `--token-profile-id ` (non-interactive; default: `:manual`)
-   `--token-expires-in ` (non-interactive; e.g. `365d`, `12h`)
-   `--secret-input-mode <plaintext|ref>` (default `plaintext`; use `ref` to store provider default env refs instead of plaintext keys)
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
-   `--custom-base-url ` (non-interactive; used with `--auth-choice custom-api-key`)
-   `--custom-model-id ` (non-interactive; used with `--auth-choice custom-api-key`)
-   `--custom-api-key ` (non-interactive; optional; used with `--auth-choice custom-api-key`; falls back to `CUSTOM_API_KEY` when omitted)
-   `--custom-provider-id ` (non-interactive; optional custom provider id)
-   `--custom-compatibility <openai|anthropic>` (non-interactive; optional; default `openai`)
-   `--gateway-port `
-   `--gateway-bind <loopback|lan|tailnet|auto|custom>`
-   `--gateway-auth <token|password>`
-   `--gateway-token `
-   `--gateway-token-ref-env ` (non-interactive; store `gateway.auth.token` as an env SecretRef; requires that env var to be set; cannot be combined with `--gateway-token`)
-   `--gateway-password `
-   `--remote-url `
-   `--remote-token `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--install-daemon`
-   `--no-install-daemon` (alias: `--skip-daemon`)
-   `--daemon-runtime <node|bun>`
-   `--skip-channels`
-   `--skip-skills`
-   `--skip-health`
-   `--skip-ui`
-   `--node-manager <npm|pnpm|bun>` (pnpm recommended; bun not recommended for Gateway runtime)
-   `--json`

### configure

Interactive configuration wizard (models, channels, skills, gateway).

### config

Non-interactive config helpers (get/set/unset/file/validate). Running `openclaw config` with no subcommand launches the wizard. Subcommands:

-   `config get `: print a config value (dot/bracket path).
-   `config set  `: set a value (JSON5 or raw string).
-   `config unset `: remove a value.
-   `config file`: print the active config file path.
-   `config validate`: validate the current config against the schema without starting the gateway.
-   `config validate --json`: emit machine-readable JSON output.

### doctor

Health checks + quick fixes (config + gateway + legacy services). Options:

-   `--no-workspace-suggestions`: disable workspace memory hints.
-   `--yes`: accept defaults without prompting (headless).
-   `--non-interactive`: skip prompts; apply safe migrations only.
-   `--deep`: scan system services for extra gateway installs.

## Channel helpers

### channels

Manage chat channel accounts (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams). Subcommands:

-   `channels list`: show configured channels and auth profiles.
-   `channels status`: check gateway reachability and channel health (`--probe` runs extra checks; use `openclaw health` or `openclaw status --deep` for gateway health probes).
-   Tip: `channels status` prints warnings with suggested fixes when it can detect common misconfigurations (then points you to `openclaw doctor`).
-   `channels logs`: show recent channel logs from the gateway log file.
-   `channels add`: wizard-style setup when no flags are passed; flags switch to non-interactive mode.
    -   When adding a non-default account to a channel still using single-account top-level config, OpenClaw moves account-scoped values into `channels..accounts.default` before writing the new account.
    -   Non-interactive `channels add` does not auto-create/upgrade bindings; channel-only bindings continue to match the default account.
-   `channels remove`: disable by default; pass `--delete` to remove config entries without prompts.
-   `channels login`: interactive channel login (WhatsApp Web only).
-   `channels logout`: log out of a channel session (if supported).

Common options:

-   `--channel `: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
-   `--account `: channel account id (default `default`)
-   `--name `: display name for the account

`channels login` options:

-   `--channel ` (default `whatsapp`; supports `whatsapp`/`web`)
-   `--account `
-   `--verbose`

`channels logout` options:

-   `--channel ` (default `whatsapp`)
-   `--account `

`channels list` options:

-   `--no-usage`: skip model provider usage/quota snapshots (OAuth/API-backed only).
-   `--json`: output JSON (includes usage unless `--no-usage` is set).

`channels logs` options:

-   `--channel <name|all>` (default `all`)
-   `--lines ` (default `200`)
-   `--json`

More detail: [/concepts/oauth](./concepts/oauth.md) Examples:

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

### skills

List and inspect available skills plus readiness info. Subcommands:

-   `skills list`: list skills (default when no subcommand).
-   `skills info `: show details for one skill.
-   `skills check`: summary of ready vs missing requirements.

Options:

-   `--eligible`: show only ready skills.
-   `--json`: output JSON (no styling).
-   `-v`, `--verbose`: include missing requirements detail.

Tip: use `npx clawhub` to search, install, and sync skills.

### pairing

Approve DM pairing requests across channels. Subcommands:

-   `pairing list [channel] [--channel ] [--account ] [--json]`
-   `pairing approve   [--account ] [--notify]`
-   `pairing approve --channel  [--account ]  [--notify]`

### devices

Manage gateway device pairing entries and per-role device tokens. Subcommands:

-   `devices list [--json]`
-   `devices approve [requestId] [--latest]`
-   `devices reject `
-   `devices remove `
-   `devices clear --yes [--pending]`
-   `devices rotate --device  --role  [--scope <scope...>]`
-   `devices revoke --device  --role `

### webhooks gmail

Gmail Pub/Sub hook setup + runner. See [/automation/gmail-pubsub](./automation/gmail-pubsub.md). Subcommands:

-   `webhooks gmail setup` (requires `--account `; supports `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
-   `webhooks gmail run` (runtime overrides for the same flags)

### dns setup

Wide-area discovery DNS helper (CoreDNS + Tailscale). See [/gateway/discovery](./gateway/discovery.md). Options:

-   `--apply`: install/update CoreDNS config (requires sudo; macOS only).

## Messaging + agent

### message

Unified outbound messaging + channel actions. See: [/cli/message](./cli/message.md) Subcommands:

-   `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
-   `message thread <create|list|reply>`
-   `message emoji <list|upload>`
-   `message sticker <send|upload>`
-   `message role <info|add|remove>`
-   `message channel <info|list>`
-   `message member info`
-   `message voice status`
-   `message event <list|create>`

Examples:

-   `openclaw message send --target +15555550123 --message "Hi"`
-   `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

### agent

Run one agent turn via the Gateway (or `--local` embedded). Required:

-   `--message `

Options:

-   `--to ` (for session key and optional delivery)
-   `--session-id `
-   `--thinking <off|minimal|low|medium|high|xhigh>` (GPT-5.2 + Codex models only)
-   `--verbose <on|full|off>`
-   `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
-   `--local`
-   `--deliver`
-   `--json`
-   `--timeout `

### agents

Manage isolated agents (workspaces + auth + routing).

#### agents list

List configured agents. Options:

-   `--json`
-   `--bindings`

#### agents add \[name\]

Add a new isolated agent. Runs the guided wizard unless flags (or `--non-interactive`) are passed; `--workspace` is required in non-interactive mode. Options:

-   `--workspace `
-   `--model `
-   `--agent-dir `
-   `--bind <channel[:accountId]>` (repeatable)
-   `--non-interactive`
-   `--json`

Binding specs use `channel[:accountId]`. When `accountId` is omitted, OpenClaw may resolve account scope via channel defaults/plugin hooks; otherwise it is a channel binding without explicit account scope.

#### agents bindings

List routing bindings. Options:

-   `--agent `
-   `--json`

#### agents bind

Add routing bindings for an agent. Options:

-   `--agent `
-   `--bind <channel[:accountId]>` (repeatable)
-   `--json`

#### agents unbind

Remove routing bindings for an agent. Options:

-   `--agent `
-   `--bind <channel[:accountId]>` (repeatable)
-   `--all`
-   `--json`

#### agents delete &lt;id&gt;

Delete an agent and prune its workspace + state. Options:

-   `--force`
-   `--json`

### acp

Run the ACP bridge that connects IDEs to the Gateway. See [`acp`](./cli/acp.md) for full options and examples.

### status

Show linked session health and recent recipients. Options:

-   `--json`
-   `--all` (full diagnosis; read-only, pasteable)
-   `--deep` (probe channels)
-   `--usage` (show model provider usage/quota)
-   `--timeout `
-   `--verbose`
-   `--debug` (alias for `--verbose`)

Notes:

-   Overview includes Gateway + node host service status when available.

### Usage tracking

OpenClaw can surface provider usage/quota when OAuth/API creds are available. Surfaces:

-   `/status` (adds a short provider usage line when available)
-   `openclaw status --usage` (prints full provider breakdown)
-   macOS menu bar (Usage section under Context)

Notes:

-   Data comes directly from provider usage endpoints (no estimates).
-   Providers: Anthropic, GitHub Copilot, OpenAI Codex OAuth, plus Gemini CLI/Antigravity when those provider plugins are enabled.
-   If no matching credentials exist, usage is hidden.
-   Details: see [Usage tracking](./concepts/usage-tracking.md).

### health

Fetch health from the running Gateway. Options:

-   `--json`
-   `--timeout `
-   `--verbose`

### sessions

List stored conversation sessions. Options:

-   `--json`
-   `--verbose`
-   `--store `
-   `--active `

## Reset / Uninstall

### reset

Reset local config/state (keeps the CLI installed). Options:

-   `--scope <config|config+creds+sessions|full>`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

Notes:

-   `--non-interactive` requires `--scope` and `--yes`.

### uninstall

Uninstall the gateway service + local data (CLI remains). Options:

-   `--service`
-   `--state`
-   `--workspace`
-   `--app`
-   `--all`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

Notes:

-   `--non-interactive` requires `--yes` and explicit scopes (or `--all`).

## Gateway

### gateway

Run the WebSocket Gateway. Options:

-   `--port `
-   `--bind <loopback|tailnet|lan|auto|custom>`
-   `--token `
-   `--auth <token|password>`
-   `--password `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--allow-unconfigured`
-   `--dev`
-   `--reset` (reset dev config + credentials + sessions + workspace)
-   `--force` (kill existing listener on port)
-   `--verbose`
-   `--claude-cli-logs`
-   `--ws-log <auto|full|compact>`
-   `--compact` (alias for `--ws-log compact`)
-   `--raw-stream`
-   `--raw-stream-path `

### gateway service

Manage the Gateway service (launchd/systemd/schtasks). Subcommands:

-   `gateway status` (probes the Gateway RPC by default)
-   `gateway install` (service install)
-   `gateway uninstall`
-   `gateway start`
-   `gateway stop`
-   `gateway restart`

Notes:

-   `gateway status` probes the Gateway RPC by default using the service’s resolved port/config (override with `--url/--token/--password`).
-   `gateway status` supports `--no-probe`, `--deep`, and `--json` for scripting.
-   `gateway status` also surfaces legacy or extra gateway services when it can detect them (`--deep` adds system-level scans). Profile-named OpenClaw services are treated as first-class and aren’t flagged as “extra”.
-   `gateway status` prints which config path the CLI uses vs which config the service likely uses (service env), plus the resolved probe target URL.
-   `gateway install|uninstall|start|stop|restart` support `--json` for scripting (default output stays human-friendly).
-   `gateway install` defaults to Node runtime; bun is **not recommended** (WhatsApp/Telegram bugs).
-   `gateway install` options: `--port`, `--runtime`, `--token`, `--force`, `--json`.

### logs

Tail Gateway file logs via RPC. Notes:

-   TTY sessions render a colorized, structured view; non-TTY falls back to plain text.
-   `--json` emits line-delimited JSON (one log event per line).

Examples:

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

### gateway &lt;subcommand&gt;

Gateway CLI helpers (use `--url`, `--token`, `--password`, `--timeout`, `--expect-final` for RPC subcommands). When you pass `--url`, the CLI does not auto-apply config or environment credentials. Include `--token` or `--password` explicitly. Missing explicit credentials is an error. Subcommands:

-   `gateway call  [--params ]`
-   `gateway health`
-   `gateway status`
-   `gateway probe`
-   `gateway discover`
-   `gateway install|uninstall|start|stop|restart`
-   `gateway run`

Common RPCs:

-   `config.apply` (validate + write config + restart + wake)
-   `config.patch` (merge a partial update + restart + wake)
-   `update.run` (run update + restart + wake)

Tip: when calling `config.set`/`config.apply`/`config.patch` directly, pass `baseHash` from `config.get` if a config already exists.

## Models

See [/concepts/models](./concepts/models.md) for fallback behavior and scanning strategy. Anthropic setup-token (supported):

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

Policy note: this is technical compatibility. Anthropic has blocked some subscription usage outside Claude Code in the past; verify current Anthropic terms before relying on setup-token in production.

### models (root)

`openclaw models` is an alias for `models status`. Root options:

-   `--status-json` (alias for `models status --json`)
-   `--status-plain` (alias for `models status --plain`)

### models list

Options:

-   `--all`
-   `--local`
-   `--provider `
-   `--json`
-   `--plain`

### models status

Options:

-   `--json`
-   `--plain`
-   `--check` (exit 1=expired/missing, 2=expiring)
-   `--probe` (live probe of configured auth profiles)
-   `--probe-provider `
-   `--probe-profile ` (repeat or comma-separated)
-   `--probe-timeout `
-   `--probe-concurrency `
-   `--probe-max-tokens `

Always includes the auth overview and OAuth expiry status for profiles in the auth store. `--probe` runs live requests (may consume tokens and trigger rate limits).

### models set &lt;model&gt;

Set `agents.defaults.model.primary`.

### models set-image &lt;model&gt;

Set `agents.defaults.imageModel.primary`.

### models aliases list|add|remove

Options:

-   `list`: `--json`, `--plain`
-   `add  `
-   `remove `

### models fallbacks list|add|remove|clear

Options:

-   `list`: `--json`, `--plain`
-   `add `
-   `remove `
-   `clear`

### models image-fallbacks list|add|remove|clear

Options:

-   `list`: `--json`, `--plain`
-   `add `
-   `remove `
-   `clear`

### models scan

Options:

-   `--min-params `
-   `--max-age-days `
-   `--provider `
-   `--max-candidates `
-   `--timeout `
-   `--concurrency `
-   `--no-probe`
-   `--yes`
-   `--no-input`
-   `--set-default`
-   `--set-image`
-   `--json`

### models auth add|setup-token|paste-token

Options:

-   `add`: interactive auth helper
-   `setup-token`: `--provider ` (default `anthropic`), `--yes`
-   `paste-token`: `--provider `, `--profile-id `, `--expires-in `

### models auth order get|set|clear

Options:

-   `get`: `--provider `, `--agent `, `--json`
-   `set`: `--provider `, `--agent `, `<profileIds...>`
-   `clear`: `--provider `, `--agent `

## System

### system event

Enqueue a system event and optionally trigger a heartbeat (Gateway RPC). Required:

-   `--text `

Options:

-   `--mode <now|next-heartbeat>`
-   `--json`
-   `--url`, `--token`, `--timeout`, `--expect-final`

### system heartbeat last|enable|disable

Heartbeat controls (Gateway RPC). Options:

-   `--json`
-   `--url`, `--token`, `--timeout`, `--expect-final`

### system presence

List system presence entries (Gateway RPC). Options:

-   `--json`
-   `--url`, `--token`, `--timeout`, `--expect-final`

## Cron

Manage scheduled jobs (Gateway RPC). See [/automation/cron-jobs](./automation/cron-jobs.md). Subcommands:

-   `cron status [--json]`
-   `cron list [--all] [--json]` (table output by default; use `--json` for raw)
-   `cron add` (alias: `create`; requires `--name` and exactly one of `--at` | `--every` | `--cron`, and exactly one payload of `--system-event` | `--message`)
-   `cron edit ` (patch fields)
-   `cron rm ` (aliases: `remove`, `delete`)
-   `cron enable `
-   `cron disable `
-   `cron runs --id  [--limit ]`
-   `cron run  [--force]`

All `cron` commands accept `--url`, `--token`, `--timeout`, `--expect-final`.

## Node host

`node` runs a **headless node host** or manages it as a background service. See [`openclaw node`](./cli/node.md). Subcommands:

-   `node run --host <gateway-host> --port 18789`
-   `node status`
-   `node install [--host <gateway-host>] [--port ] [--tls] [--tls-fingerprint ] [--node-id ] [--display-name ] [--runtime <node|bun>] [--force]`
-   `node uninstall`
-   `node stop`
-   `node restart`

## Nodes

`nodes` talks to the Gateway and targets paired nodes. See [/nodes](./nodes.md). Common options:

-   `--url`, `--token`, `--timeout`, `--json`

Subcommands:

-   `nodes status [--connected] [--last-connected ]`
-   `nodes describe --node <id|name|ip>`
-   `nodes list [--connected] [--last-connected ]`
-   `nodes pending`
-   `nodes approve `
-   `nodes reject `
-   `nodes rename --node <id|name|ip> --name `
-   `nodes invoke --node <id|name|ip> --command  [--params ] [--invoke-timeout ] [--idempotency-key ]`
-   `nodes run --node <id|name|ip> [--cwd ] [--env KEY=VAL] [--command-timeout ] [--needs-screen-recording] [--invoke-timeout ] <command...>` (mac node or headless node host)
-   `nodes notify --node <id|name|ip> [--title ] [--body ] [--sound ] [--priority <passive|active|timeSensitive>] [--delivery <system|overlay|auto>] [--invoke-timeout ]` (mac only)

Camera:

-   `nodes camera list --node <id|name|ip>`
-   `nodes camera snap --node <id|name|ip> [--facing front|back|both] [--device-id ] [--max-width ] [--quality <0-1>] [--delay-ms ] [--invoke-timeout ]`
-   `nodes camera clip --node <id|name|ip> [--facing front|back] [--device-id ] [--duration <ms|10s|1m>] [--no-audio] [--invoke-timeout ]`

Canvas + screen:

-   `nodes canvas snapshot --node <id|name|ip> [--format png|jpg|jpeg] [--max-width ] [--quality <0-1>] [--invoke-timeout ]`
-   `nodes canvas present --node <id|name|ip> [--target ] [--x ] [--y ] [--width ] [--height ] [--invoke-timeout ]`
-   `nodes canvas hide --node <id|name|ip> [--invoke-timeout ]`
-   `nodes canvas navigate  --node <id|name|ip> [--invoke-timeout ]`
-   `nodes canvas eval [] --node <id|name|ip> [--js ] [--invoke-timeout ]`
-   `nodes canvas a2ui push --node <id|name|ip> (--jsonl  | --text ) [--invoke-timeout ]`
-   `nodes canvas a2ui reset --node <id|name|ip> [--invoke-timeout ]`
-   `nodes screen record --node <id|name|ip> [--screen ] [--duration <ms|10s>] [--fps ] [--no-audio] [--out ] [--invoke-timeout ]`

Location:

-   `nodes location get --node <id|name|ip> [--max-age ] [--accuracy <coarse|balanced|precise>] [--location-timeout ] [--invoke-timeout ]`

## Browser

Browser control CLI (dedicated Chrome/Brave/Edge/Chromium). See [`openclaw browser`](./cli/browser.md) and the [Browser tool](./tools/browser.md). Common options:

-   `--url`, `--token`, `--timeout`, `--json`
-   `--browser-profile `

Manage:

-   `browser status`
-   `browser start`
-   `browser stop`
-   `browser reset-profile`
-   `browser tabs`
-   `browser open `
-   `browser focus `
-   `browser close [targetId]`
-   `browser profiles`
-   `browser create-profile --name  [--color ] [--cdp-url ]`
-   `browser delete-profile --name `

Inspect:

-   `browser screenshot [targetId] [--full-page] [--ref ] [--element ] [--type png|jpeg]`
-   `browser snapshot [--format aria|ai] [--target-id ] [--limit ] [--interactive] [--compact] [--depth ] [--selector ] [--out ]`

Actions:

-   `browser navigate  [--target-id ]`
-   `browser resize   [--target-id ]`
-   `browser click  [--double] [--button <left|right|middle>] [--modifiers ] [--target-id ]`
-   `browser type   [--submit] [--slowly] [--target-id ]`
-   `browser press  [--target-id ]`
-   `browser hover  [--target-id ]`
-   `browser drag   [--target-id ]`
-   `browser select  <values...> [--target-id ]`
-   `browser upload <paths...> [--ref ] [--input-ref ] [--element ] [--target-id ] [--timeout-ms ]`
-   `browser fill [--fields ] [--fields-file ] [--target-id ]`
-   `browser dialog --accept|--dismiss [--prompt ] [--target-id ] [--timeout-ms ]`
-   `browser wait [--time ] [--text ] [--text-gone ] [--target-id ]`
-   `browser evaluate --fn  [--ref ] [--target-id ]`
-   `browser console [--level <error|warn|info>] [--target-id ]`
-   `browser pdf [--target-id ]`

## Docs search

### docs \[query...\]

Search the live docs index.

## TUI

### tui

Open the terminal UI connected to the Gateway. Options:

-   `--url `
-   `--token `
-   `--password `
-   `--session `
-   `--deliver`
-   `--thinking `
-   `--message `
-   `--timeout-ms ` (defaults to `agents.defaults.timeoutSeconds`)
-   `--history-limit `

[acp](./cli/acp.md)
