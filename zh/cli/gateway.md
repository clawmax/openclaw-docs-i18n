

  CLI 命令

  
# gateway

Gateway 是 OpenClaw 的 WebSocket 服务器（频道、节点、会话、钩子）。本页的子命令位于 `openclaw gateway …` 下。相关文档：

-   [/gateway/bonjour](../gateway/bonjour.md)
-   [/gateway/discovery](../gateway/discovery.md)
-   [/gateway/configuration](../gateway/configuration.md)

## 运行 Gateway

运行本地 Gateway 进程：

```bash
openclaw gateway
```

前台别名：

```bash
openclaw gateway run
```

注意：

-   默认情况下，除非在 `~/.openclaw/openclaw.json` 中设置了 `gateway.mode=local`，否则 Gateway 会拒绝启动。使用 `--allow-unconfigured` 进行临时/开发运行。
-   未经身份验证绑定到环回接口之外是被阻止的（安全护栏）。
-   `SIGUSR1` 在授权时触发进程内重启（默认启用 `commands.restart`；设置 `commands.restart: false` 可阻止手动重启，而网关工具/配置应用/更新仍被允许）。
-   `SIGINT`/`SIGTERM` 处理程序会停止网关进程，但它们不会恢复任何自定义终端状态。如果您使用 TUI 或原始模式输入包装 CLI，请在退出前恢复终端。

### 选项

-   `--port `: WebSocket 端口（默认来自配置/环境；通常为 `18789`）。
-   `--bind <loopback|lan|tailnet|auto|custom>`: 监听器绑定模式。
-   `--auth <token|password>`: 身份验证模式覆盖。
-   `--token `: 令牌覆盖（同时为进程设置 `OPENCLAW_GATEWAY_TOKEN`）。
-   `--password `: 密码覆盖（同时为进程设置 `OPENCLAW_GATEWAY_PASSWORD`）。
-   `--tailscale <off|serve|funnel>`: 通过 Tailscale 暴露 Gateway。
-   `--tailscale-reset-on-exit`: 在关闭时重置 Tailscale serve/funnel 配置。
-   `--allow-unconfigured`: 允许在没有配置 `gateway.mode=local` 的情况下启动网关。
-   `--dev`: 如果缺少开发配置和工作区则创建（跳过 BOOTSTRAP.md）。
-   `--reset`: 重置开发配置 + 凭据 + 会话 + 工作区（需要 `--dev`）。
-   `--force`: 在启动前终止所选端口上的任何现有监听器。
-   `--verbose`: 详细日志。
-   `--claude-cli-logs`: 仅在控制台中显示 claude-cli 日志（并启用其 stdout/stderr）。
-   `--ws-log <auto|full|compact>`: websocket 日志样式（默认 `auto`）。
-   `--compact`: `--ws-log compact` 的别名。
-   `--raw-stream`: 将原始模型流事件记录到 jsonl。
-   `--raw-stream-path `: 原始流 jsonl 路径。

## 查询正在运行的 Gateway

所有查询命令都使用 WebSocket RPC。输出模式：

-   默认：人类可读（在 TTY 中带颜色）。
-   `--json`: 机器可读的 JSON（无样式/微调器）。
-   `--no-color`（或 `NO_COLOR=1`）：禁用 ANSI 颜色，同时保持人类可读布局。

共享选项（在支持的地方）：

-   `--url `: Gateway WebSocket URL。
-   `--token `: Gateway 令牌。
-   `--password `: Gateway 密码。
-   `--timeout `: 超时/预算（因命令而异）。
-   `--expect-final`: 等待“最终”响应（代理调用）。

注意：当您设置 `--url` 时，CLI 不会回退到配置或环境凭据。请显式传递 `--token` 或 `--password`。缺少显式凭据会导致错误。

### gateway health

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### gateway status

`gateway status` 显示 Gateway 服务（launchd/systemd/schtasks）以及一个可选的 RPC 探测。

```bash
openclaw gateway status
openclaw gateway status --json
```

选项：

-   `--url `: 覆盖探测 URL。
-   `--token `: 探测的令牌身份验证。
-   `--password `: 探测的密码身份验证。
-   `--timeout `: 探测超时（默认 `10000`）。
-   `--no-probe`: 跳过 RPC 探测（仅服务视图）。
-   `--deep`: 同时扫描系统级服务。

注意：

-   `gateway status` 在可能的情况下会解析配置的身份验证 SecretRefs 以用于探测身份验证。
-   如果在此命令路径中所需的身份验证 SecretRef 未解析，探测身份验证可能会失败；请显式传递 `--token`/`--password` 或先解析密钥源。

### gateway probe

`gateway probe` 是“调试一切”命令。它总是探测：

-   您配置的远程网关（如果已设置），以及
-   本地主机（环回）**即使配置了远程网关**。

如果多个网关可达，它会打印所有网关。当您使用隔离的配置文件/端口（例如，救援机器人）时，支持多个网关，但大多数安装仍然运行单个网关。

```bash
openclaw gateway probe
openclaw gateway probe --json
```

#### 通过 SSH 远程（Mac 应用对等）

macOS 应用的“通过 SSH 远程”模式使用本地端口转发，使远程网关（可能仅绑定到环回）在 `ws://127.0.0.1:` 变得可达。CLI 等效命令：

```bash
openclaw gateway probe --ssh user@gateway-host
```

选项：

-   `--ssh `: `user@host` 或 `user@host:port`（端口默认为 `22`）。
-   `--ssh-identity `: 身份文件。
-   `--ssh-auto`: 选择第一个发现的网关主机作为 SSH 目标（仅限 LAN/WAB）。

配置（可选，用作默认值）：

-   `gateway.remote.sshTarget`
-   `gateway.remote.sshIdentity`

### gateway call &lt;method&gt;

低级 RPC 助手。

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

## 管理 Gateway 服务

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

注意：

-   `gateway install` 支持 `--port`、`--runtime`、`--token`、`--force`、`--json`。
-   当令牌身份验证需要令牌且 `gateway.auth.token` 由 SecretRef 管理时，`gateway install` 会验证 SecretRef 是否可解析，但不会将解析后的令牌持久化到服务环境元数据中。
-   如果令牌身份验证需要令牌且配置的令牌 SecretRef 未解析，安装会失败关闭，而不是持久化回退的明文。
-   在推断的身份验证模式下，仅限 shell 的 `OPENCLAW_GATEWAY_PASSWORD`/`CLAWDBOT_GATEWAY_PASSWORD` 不会放宽安装令牌要求；在安装托管服务时，请使用持久配置（`gateway.auth.password` 或配置 `env`）。
-   如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password` 且 `gateway.auth.mode` 未设置，则在显式设置模式之前，安装会被阻止。
-   生命周期命令接受 `--json` 用于脚本编写。

## 发现网关（Bonjour）

`gateway discover` 扫描 Gateway 信标（`_openclaw-gw._tcp`）。

-   多播 DNS-SD：`local.`
-   单播 DNS-SD（广域 Bonjour）：选择一个域（例如：`openclaw.internal.`）并设置拆分 DNS + DNS 服务器；参见 [/gateway/bonjour](../gateway/bonjour.md)

只有启用了 Bonjour 发现（默认）的网关才会广播信标。广域发现记录包括（TXT）：

-   `role`（网关角色提示）
-   `transport`（传输提示，例如 `gateway`）
-   `gatewayPort`（WebSocket 端口，通常为 `18789`）
-   `sshPort`（SSH 端口；如果不存在则默认为 `22`）
-   `tailnetDns`（MagicDNS 主机名，可用时）
-   `gatewayTls` / `gatewayTlsSha256`（启用 TLS + 证书指纹）
-   `cliPath`（远程安装的可选提示）

### gateway discover

```bash
openclaw gateway discover
```

选项：

-   `--timeout `: 每个命令的超时（浏览/解析）；默认 `2000`。
-   `--json`: 机器可读输出（同时禁用样式/微调器）。

示例：

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```

[doctor](./doctor.md)[health](./health.md)

---