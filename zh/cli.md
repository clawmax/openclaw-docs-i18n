

  CLI 命令

  
# CLI 参考

本页描述当前 CLI 的行为。如果命令发生变化，请更新此文档。

## 命令页面

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
-   [`plugins`](./cli/plugins.md) (插件命令)
-   [`channels`](./cli/channels.md)
-   [`security`](./cli/security.md)
-   [`secrets`](./cli/secrets.md)
-   [`skills`](./cli/skills.md)
-   [`daemon`](./cli/daemon.md) (网关服务命令的旧别名)
-   [`clawbot`](./cli/clawbot.md) (旧别名命名空间)
-   [`voicecall`](./cli/voicecall.md) (插件；如果已安装)

## 全局标志

-   `--dev`: 将状态隔离在 `~/.openclaw-dev` 下，并移动默认端口。
-   `--profile `: 将状态隔离在 `~/.openclaw-` 下。
-   `--no-color`: 禁用 ANSI 颜色。
-   `--update`: `openclaw update` 的简写（仅限源码安装）。
-   `-V`, `--version`, `-v`: 打印版本并退出。

## 输出样式

-   ANSI 颜色和进度指示器仅在 TTY 会话中渲染。
-   OSC-8 超链接在支持的终端中渲染为可点击链接；否则回退到纯文本 URL。
-   `--json`（以及支持的 `--plain`）禁用样式以获得干净输出。
-   `--no-color` 禁用 ANSI 样式；同时也会遵守 `NO_COLOR=1`。
-   长时间运行的命令会显示进度指示器（支持时使用 OSC 9;4）。

## 调色板

OpenClaw 使用龙虾调色板进行 CLI 输出。

-   `accent` (#FF5A2D): 标题、标签、主要高亮。
-   `accentBright` (#FF7A3D): 命令名称、强调。
-   `accentDim` (#D14A22): 次要高亮文本。
-   `info` (#FF8A5B): 信息性数值。
-   `success` (#2FBF71): 成功状态。
-   `warn` (#FFB020): 警告、回退、注意。
-   `error` (#E23D2D): 错误、失败。
-   `muted` (#8B7F77): 弱化、元数据。

调色板源文件：`src/terminal/palette.ts`（又称“龙虾接缝”）。

## 命令树

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

注意：插件可以添加额外的顶级命令（例如 `openclaw voicecall`）。

## 安全

-   `openclaw security audit` — 审计配置 + 本地状态，查找常见的安全隐患。
-   `openclaw security audit --deep` — 尽力对网关进行实时探测。
-   `openclaw security audit --fix` — 收紧安全默认值并对状态/配置进行 chmod。

## 密钥

-   `openclaw secrets reload` — 重新解析引用并以原子方式交换运行时快照。
-   `openclaw secrets audit` — 扫描明文残留、未解析的引用和优先级漂移。
-   `openclaw secrets configure` — 用于提供程序设置 + SecretRef 映射 + 预检/应用的交互式助手。
-   `openclaw secrets apply --from <plan.json>` — 应用先前生成的计划（支持 `--dry-run`）。

## 插件

管理扩展及其配置：

-   `openclaw plugins list` — 发现插件（使用 `--json` 获取机器输出）。
-   `openclaw plugins info ` — 显示插件的详细信息。
-   `openclaw plugins install <path|.tgz|npm-spec>` — 安装插件（或将插件路径添加到 `plugins.load.paths`）。
-   `openclaw plugins enable ` / `disable ` — 切换 `plugins.entries..enabled`。
-   `openclaw plugins doctor` — 报告插件加载错误。

大多数插件更改需要重启网关。参见 [/plugin](./tools/plugin.md)。

## 记忆

对 `MEMORY.md` + `memory/*.md` 进行向量搜索：

-   `openclaw memory status` — 显示索引统计信息。
-   `openclaw memory index` — 重新索引记忆文件。
-   `openclaw memory search ""`（或 `--query ""`）— 对记忆进行语义搜索。

## 聊天斜杠命令

聊天消息支持 `/...` 命令（文本和原生）。参见 [/tools/slash-commands](./tools/slash-commands.md)。亮点：

-   `/status` 用于快速诊断。
-   `/config` 用于持久化配置更改。
-   `/debug` 用于仅运行时配置覆盖（内存中，非磁盘；需要 `commands.debug: true`）。

## 设置 + 入门

### setup

初始化配置 + 工作区。选项：

-   `--workspace `: 代理工作区路径（默认 `~/.openclaw/workspace`）。
-   `--wizard`: 运行入门向导。
-   `--non-interactive`: 运行向导时不提示。
-   `--mode <local|remote>`: 向导模式。
-   `--remote-url `: 远程网关 URL。
-   `--remote-token `: 远程网关令牌。

当存在任何向导标志时（`--non-interactive`、`--mode`、`--remote-url`、`--remote-token`），向导会自动运行。

### onboard

交互式向导，用于设置网关、工作区和技能。选项：

-   `--workspace `
-   `--reset`（在向导运行前重置配置 + 凭据 + 会话）
-   `--reset-scope <config|config+creds+sessions|full>`（默认 `config+creds+sessions`；使用 `full` 也会删除工作区）
-   `--non-interactive`
-   `--mode <local|remote>`
-   `--flow <quickstart|advanced|manual>`（manual 是 advanced 的别名）
-   `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|moonshot-api-key-cn|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|mistral-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|custom-api-key|skip>`
-   `--token-provider `（非交互式；与 `--auth-choice token` 一起使用）
-   `--token `（非交互式；与 `--auth-choice token` 一起使用）
-   `--token-profile-id `（非交互式；默认：`:manual`）
-   `--token-expires-in `（非交互式；例如 `365d`、`12h`）
-   `--secret-input-mode <plaintext|ref>`（默认 `plaintext`；使用 `ref` 将提供程序默认环境引用存储为密钥，而非明文）
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
-   `--custom-base-url `（非交互式；与 `--auth-choice custom-api-key` 一起使用）
-   `--custom-model-id `（非交互式；与 `--auth-choice custom-api-key` 一起使用）
-   `--custom-api-key `（非交互式；可选；与 `--auth-choice custom-api-key` 一起使用；省略时回退到 `CUSTOM_API_KEY`）
-   `--custom-provider-id `（非交互式；可选的自定义提供程序 ID）
-   `--custom-compatibility <openai|anthropic>`（非交互式；可选；默认 `openai`）
-   `--gateway-port `
-   `--gateway-bind <loopback|lan|tailnet|auto|custom>`
-   `--gateway-auth <token|password>`
-   `--gateway-token `
-   `--gateway-token-ref-env `（非交互式；将 `gateway.auth.token` 存储为环境变量 SecretRef；需要设置该环境变量；不能与 `--gateway-token` 组合使用）
-   `--gateway-password `
-   `--remote-url `
-   `--remote-token `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--install-daemon`
-   `--no-install-daemon`（别名：`--skip-daemon`）
-   `--daemon-runtime <node|bun>`
-   `--skip-channels`
-   `--skip-skills`
-   `--skip-health`
-   `--skip-ui`
-   `--node-manager <npm|pnpm|bun>`（推荐 pnpm；不推荐 bun 作为网关运行时）
-   `--json`

### configure

交互式配置向导（模型、频道、技能、网关）。

### config

非交互式配置助手（get/set/unset/file/validate）。不带子命令运行 `openclaw config` 会启动向导。子命令：

-   `config get `: 打印配置值（点/括号路径）。
-   `config set  `: 设置值（JSON5 或原始字符串）。
-   `config unset `: 移除值。
-   `config file`: 打印活动配置文件路径。
-   `config validate`: 验证当前配置是否符合模式，而不启动网关。
-   `config validate --json`: 发出机器可读的 JSON 输出。

### doctor

健康检查 + 快速修复（配置 + 网关 + 旧版服务）。选项：

-   `--no-workspace-suggestions`: 禁用工作区记忆提示。
-   `--yes`: 接受默认值而不提示（无头模式）。
-   `--non-interactive`: 跳过提示；仅应用安全迁移。
-   `--deep`: 扫描系统服务以查找额外的网关安装。

## 频道助手

### channels

管理聊天频道账户（WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (插件)/Signal/iMessage/MS Teams）。子命令：

-   `channels list`: 显示已配置的频道和身份验证配置文件。
-   `channels status`: 检查网关可达性和频道健康状况（`--probe` 运行额外检查；使用 `openclaw health` 或 `openclaw status --deep` 进行网关健康探测）。
-   提示：`channels status` 在检测到常见配置错误时会打印警告并提供建议的修复方法（然后引导你使用 `openclaw doctor`）。
-   `channels logs`: 显示网关日志文件中的近期频道日志。
-   `channels add`: 未传递标志时以向导模式设置；标志切换到非交互模式。
    -   当向仍使用单账户顶级配置的频道添加非默认账户时，OpenClaw 会在写入新账户之前，将账户作用域的值移动到 `channels..accounts.default` 中。
    -   非交互式 `channels add` 不会自动创建/升级绑定；仅频道绑定继续匹配默认账户。
-   `channels remove`: 默认禁用；传递 `--delete` 可在不提示的情况下移除配置条目。
-   `channels login`: 交互式频道登录（仅限 WhatsApp Web）。
-   `channels logout`: 登出频道会话（如果支持）。

通用选项：

-   `--channel `: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
-   `--account `: 频道账户 ID（默认 `default`）
-   `--name `: 账户的显示名称

`channels login` 选项：

-   `--channel `（默认 `whatsapp`；支持 `whatsapp`/`web`）
-   `--account `
-   `--verbose`

`channels logout` 选项：

-   `--channel `（默认 `whatsapp`）
-   `--account `

`channels list` 选项：

-   `--no-usage`: 跳过模型提供程序使用量/配额快照（仅限 OAuth/API 支持）。
-   `--json`: 输出 JSON（除非设置了 `--no-usage`，否则包含使用量）。

`channels logs` 选项：

-   `--channel <name|all>`（默认 `all`）
-   `--lines `（默认 `200`）
-   `--json`

更多详情：[/concepts/oauth](./concepts/oauth.md) 示例：

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

### skills

列出并检查可用技能及其就绪信息。子命令：

-   `skills list`: 列出技能（无子命令时的默认行为）。
-   `skills info `: 显示一个技能的详细信息。
-   `skills check`: 就绪与缺失要求的摘要。

选项：

-   `--eligible`: 仅显示就绪的技能。
-   `--json`: 输出 JSON（无样式）。
-   `-v`, `--verbose`: 包含缺失要求的详细信息。

提示：使用 `npx clawhub` 来搜索、安装和同步技能。

### pairing

跨频道批准私信配对请求。子命令：

-   `pairing list [channel] [--channel ] [--account ] [--json]`
-   `pairing approve   [--account ] [--notify]`
-   `pairing approve --channel  [--account ]  [--notify]`

### devices

管理网关设备配对条目和按角色的设备令牌。子命令：

-   `devices list [--json]`
-   `devices approve [requestId] [--latest]`
-   `devices reject `
-   `devices remove `
-   `devices clear --yes [--pending]`
-   `devices rotate --device  --role  [--scope <scope...>]`
-   `devices revoke --device  --role `

### webhooks gmail

Gmail Pub/Sub 钩子设置 + 运行器。参见 [/automation/gmail-pubsub](./automation/gmail-pubsub.md)。子命令：

-   `webhooks gmail setup`（需要 `--account `；支持 `--project`、`--topic`、`--subscription`、`--label`、`--hook-url`、`--hook-token`、`--push-token`、`--bind`、`--port`、`--path`、`--include-body`、`--max-bytes`、`--renew-minutes`、`--tailscale`、`--tailscale-path`、`--tailscale-target`、`--push-endpoint`、`--json`）
-   `webhooks gmail run`（相同标志的运行时覆盖）

### dns setup

广域发现 DNS 助手（CoreDNS + Tailscale）。参见 [/gateway/discovery](./gateway/discovery.md)。选项：

-   `--apply`: 安装/更新 CoreDNS 配置（需要 sudo；仅限 macOS）。

## 消息 + 代理

### message

统一的外发消息 + 频道操作。参见：[/cli/message](./cli/message.md) 子命令：

-   `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
-   `message thread <create|list|reply>`
-   `message emoji <list|upload>`
-   `message sticker <send|upload>`
-   `message role <info|add|remove>`
-   `message channel <info|list>`
-   `message member info`
-   `message voice status`
-   `message event <list|create>`

示例：

-   `openclaw message send --target +15555550123 --message "Hi"`
-   `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

### agent

通过网关（或 `--local` 嵌入式）运行一个代理轮次。必需：

-   `--message `

选项：

-   `--to `（用于会话密钥和可选传递）
-   `--session-id `
-   `--thinking <off|minimal|low|medium|high|xhigh>`（仅限 GPT-5.2 + Codex 模型）
-   `--verbose <on|full|off>`
-   `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
-   `--local`
-   `--deliver`
-   `--json`
-   `--timeout `

### agents

管理隔离的代理（工作区 + 身份验证 + 路由）。

#### agents list

列出已配置的代理。选项：

-   `--json`
-   `--bindings`

#### agents add \[name\]

添加新的隔离代理。除非传递标志（或 `--non-interactive`），否则运行引导式向导；在非交互模式下需要 `--workspace`。选项：

-   `--workspace `
-   `--model `
-   `--agent-dir `
-   `--bind <channel[:accountId]>`（可重复）
-   `--non-interactive`
-   `--json`

绑定规范使用 `channel[:accountId]`。当省略 `accountId` 时，OpenClaw 可能通过频道默认值/插件钩子解析账户作用域；否则它是一个没有显式账户作用域的频道绑定。

#### agents bindings

列出路由绑定。选项：

-   `--agent `
-   `--json`

#### agents bind

为代理添加路由绑定。选项：

-   `--agent `
-   `--bind <channel[:accountId]>`（可重复）
-   `--json`

#### agents unbind

移除代理的路由绑定。选项：

-   `--agent `
-   `--bind <channel[:accountId]>`（可重复）
-   `--all`
-   `--json`

#### agents delete &lt;id&gt;

删除代理并清理其工作区 + 状态。选项：

-   `--force`
-   `--json`

### acp

运行连接 IDE 到网关的 ACP 桥接器。完整选项和示例参见 [`acp`](./cli/acp.md)。

### status

显示链接的会话健康状况和最近的收件人。选项：

-   `--json`
-   `--all`（完整诊断；只读，可粘贴）
-   `--deep`（探测频道）
-   `--usage`（显示模型提供程序使用量/配额）
-   `--timeout `
-   `--verbose`
-   `--debug`（`--verbose` 的别名）

注意：

-   概览包括网关 + 节点主机服务状态（如果可用）。

### 使用量跟踪

当 OAuth/API 凭据可用时，OpenClaw 可以显示提供程序使用量/配额。显示位置：

-   `/status`（可用时添加一行简短的提供程序使用量信息）
-   `openclaw status --usage`（打印完整的提供程序细分）
-   macOS 菜单栏（上下文下的使用量部分）

注意：

-   数据直接来自提供程序使用量端点（非估算）。
-   提供程序：Anthropic、GitHub Copilot、OpenAI Codex OAuth，以及启用这些提供程序插件时的 Gemini CLI/Antigravity。
-   如果没有匹配的凭据，则隐藏使用量。
-   详情：参见[使用量跟踪](./concepts/usage-tracking.md)。

### health

从正在运行的网关获取健康状态。选项：

-   `--json`
-   `--timeout `
-   `--verbose`

### sessions

列出存储的对话会话。选项：

-   `--json`
-   `--verbose`
-   `--store `
-   `--active `

## 重置 / 卸载

### reset

重置本地配置/状态（保留 CLI 安装）。选项：

-   `--scope <config|config+creds+sessions|full>`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

注意：

-   `--non-interactive` 需要 `--scope` 和 `--yes`。

### uninstall

卸载网关服务 + 本地数据（CLI 保留）。选项：

-   `--service`
-   `--state`
-   `--workspace`
-   `--app`
-   `--all`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

注意：

-   `--non-interactive` 需要 `--yes` 和显式的作用域（或 `--all`）。

## 网关

### gateway

运行 WebSocket 网关。选项：

-   `--port `
-   `--bind <loopback|tailnet|lan|auto|custom>`
-   `--token `
-   `--auth <token|password>`
-   `--password `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--allow-unconfigured`
-   `--dev`
-   `--reset`（重置开发配置 + 凭据 + 会话 + 工作区）
-   `--force`（终止端口上的现有监听器）
-   `--verbose`
-   `--claude-cli-logs`
-   `--ws-log <auto|full|compact>`
-   `--compact`（`--ws-log compact` 的别名）
-   `--raw-stream`
-   `--raw-stream-path `

### gateway service

管理网关服务（launchd/systemd/schtasks）。子命令：

-   `gateway status`（默认探测网关 RPC）
-   `gateway install`（服务安装）
-   `gateway uninstall`
-   `gateway start`
-   `gateway stop`
-   `gateway restart`

注意：

-   `gateway status` 默认使用服务的解析端口/配置探测网关 RPC（使用 `--url/--token/--password` 覆盖）。
-   `gateway status` 支持 `--no-probe`、`--deep` 和 `--json` 用于脚本编写。
-   `gateway status` 在检测到旧版或额外网关服务时也会显示它们（`--deep` 添加系统级扫描）。以配置文件命名的 OpenClaw 服务被视为一等公民，不会被标记为“额外”。
-   `gateway status` 打印 CLI 使用的配置路径与服务可能使用的配置（服务环境变量），以及解析后的探测目标 URL。
-   `gateway install|uninstall|start|stop|restart` 支持 `--json` 用于脚本编写（默认输出保持人性化）。
-   `gateway install` 默认为 Node 运行时；**不推荐** bun（WhatsApp/Telegram 错误）。
-   `gateway install` 选项：`--port`、`--runtime`、`--token`、`--force`、`--json`。

### logs

通过 RPC 跟踪网关文件日志。注意：

-   TTY 会话渲染带颜色的结构化视图；非 TTY 回退到纯文本。
-   `--json` 发出行分隔的 JSON（每行一个日志事件）。

示例：

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

### gateway &lt;subcommand&gt;

网关 CLI 助手（对 RPC 子命令使用 `--url`、`--token`、`--password`、`--timeout`、`--expect-final`）。传递 `--url` 时，CLI 不会自动应用配置或环境凭据。请显式包含 `--token` 或 `--password`。缺少显式凭据会导致错误。子命令：

-   `gateway call  [--params ]`
-   `gateway health`
-   `gateway status`
-   `gateway probe`
-   `gateway discover`
-   `gateway install|uninstall|start|stop|restart`
-   `gateway run`

常见 RPC：

-   `config.apply`（验证 + 写入配置 + 重启 + 唤醒）
-   `config.patch`（合并部分更新 + 重启 + 唤醒）
-   `update.run`（运行更新 + 重启 + 唤醒）

提示：直接调用 `config.set`/`config.apply`/`config.patch` 时，如果配置已存在，请传递来自 `config.get` 的 `baseHash`。

## 模型

有关回退行为和扫描策略，请参见 [/concepts/models](./concepts/models.md)。Anthropic setup-token（支持）：

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

政策说明：这是技术兼容性。Anthropic 过去曾阻止在 Claude Code 之外的一些订阅使用；在生产中依赖 setup-token 之前，请验证当前的 Anthropic 条款。

### models (根命令)

`openclaw models` 是 `models status` 的别名。根命令选项：

-   `--status-json`（`models status --json` 的别名）
-   `--status-plain`（`models status --plain` 的别名）

### models list

选项：

-   `--all`
-   `--local`
-   `--provider `
-   `--json`
-   `--plain`

### models status

选项：

-   `--json`
-   `--plain`
-   `--check`（退出 1=过期/缺失，2=即将过期）
-   `--probe`（对已配置的身份验证配置文件进行实时探测）
-   `--probe-provider `
-   `--probe-profile `（重复或逗号分隔）
-   `--probe-timeout `
-   `--probe-concurrency `
-   `--probe-max-tokens `

始终包含身份验证概览和身份验证存储中配置文件的 OAuth 过期状态。`--probe` 运行实时请求（可能消耗令牌并触发速率限制）。

### models set &lt;model&gt;

设置 `agents.defaults.model.primary`。

### models set-image &lt;model&gt;

设置 `agents.defaults.imageModel.primary`。

### models aliases list|add|remove

选项：

-   `list`: `--json`, `--plain`
-   `add  `
-   `remove `

### models fallbacks list|add|remove|clear

选项：

-   `list`: `--json`, `--plain`
-   `add `
-   `remove `
-   `clear`

### models image-fallbacks list|add|remove|clear

选项：

-   `list`: `--json`, `--plain`
-   `add `
-   `remove `
-   `clear`

### models scan

选项：

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

选项：

-   `add`: 交互式身份验证助手
-   `setup-token`: `--provider `（默认 `anthropic`），`--yes`
-   `paste-token`: `--provider `，`--profile-id `，`--expires-in `

### models auth order get|set|clear

选项：

-   `get`: `--provider `，`--agent `，`--json`
-   `set`: `--provider `，`--agent `，`&lt