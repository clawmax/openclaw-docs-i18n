

  macOS 配套应用

  
# 远程控制

此流程允许 macOS 应用作为运行在其他主机（台式机/服务器）上的 OpenClaw 网关的完整远程控制器。这是应用的 **Remote over SSH**（远程运行）功能。所有功能——健康检查、语音唤醒转发和 Web Chat——都复用 *设置 → 通用* 中的同一远程 SSH 配置。

## 模式

-   **本地（此 Mac）**：所有内容都在笔记本电脑上运行。不涉及 SSH。
-   **Remote over SSH（默认）**：OpenClaw 命令在远程主机上执行。Mac 应用使用 `-o BatchMode` 加上您选择的身份/密钥以及本地端口转发来打开 SSH 连接。
-   **Remote direct (ws/wss)**：无 SSH 隧道。Mac 应用直接连接到网关 URL（例如，通过 Tailscale Serve 或公共 HTTPS 反向代理）。

## 远程传输方式

远程模式支持两种传输方式：

-   **SSH 隧道（默认）**：使用 `ssh -N -L ...` 将网关端口转发到本地主机。由于隧道是环回地址，网关将看到节点的 IP 为 `127.0.0.1`。
-   **Direct (ws/wss)**：直接连接到网关 URL。网关看到的是真实的客户端 IP。

## 远程主机前提条件

1.  安装 Node + pnpm 并构建/安装 OpenClaw CLI (`pnpm install && pnpm build && pnpm link --global`)。
2.  确保 `openclaw` 在非交互式 shell 的 PATH 中（如有需要，可将其符号链接到 `/usr/local/bin` 或 `/opt/homebrew/bin`）。
3.  启用 SSH 密钥认证。我们建议使用 **Tailscale** IP 以实现局域网外稳定的可达性。

## macOS 应用设置

1.  打开 *设置 → 通用*。
2.  在 **OpenClaw runs** 下，选择 **Remote over SSH** 并设置：
    -   **传输方式**：**SSH 隧道** 或 **Direct (ws/wss)**。
    -   **SSH 目标**：`user@host`（可选 `:端口`）。
        -   如果网关在同一局域网内并通过 Bonjour 广播，可以从发现的列表中选择它以自动填充此字段。
    -   **网关 URL**（仅限 Direct 模式）：`wss://gateway.example.ts.net`（或用于本地/局域网的 `ws://...`）。
    -   **身份文件**（高级）：您的密钥路径。
    -   **项目根目录**（高级）：用于命令的远程检出路径。
    -   **CLI 路径**（高级）：可运行的 `openclaw` 入口点/二进制文件的路径（在广播时自动填充）。
3.  点击 **测试远程连接**。成功表示远程 `openclaw status --json` 运行正确。失败通常意味着 PATH/CLI 问题；退出码 127 表示在远程未找到 CLI。
4.  健康检查和 Web Chat 现在将通过此 SSH 隧道自动运行。

## Web Chat

-   **SSH 隧道**：Web Chat 通过转发的 WebSocket 控制端口（默认 18789）连接到网关。
-   **Direct (ws/wss)**：Web Chat 直接连接到配置的网关 URL。
-   不再有单独的 WebChat HTTP 服务器。

## 权限

-   远程主机需要与本地相同的 TCC 批准（自动化、辅助功能、屏幕录制、麦克风、语音识别、通知）。在该机器上运行一次引导流程以授予权限。
-   节点通过 `node.list` / `node.describe` 广播其权限状态，以便代理知道可用的功能。

## 安全注意事项

-   建议在远程主机上绑定环回地址，并通过 SSH 或 Tailscale 连接。
-   SSH 隧道使用严格的主机密钥检查；请先信任主机密钥，使其存在于 `~/.ssh/known_hosts` 中。
-   如果将网关绑定到非环回接口，则需要令牌/密码认证。
-   请参阅 [安全](../../gateway/security.md) 和 [Tailscale](../../gateway/tailscale.md)。

## WhatsApp 登录流程（远程）

-   在远程主机上运行 `openclaw channels login --verbose`。使用手机上的 WhatsApp 扫描二维码。
-   如果认证过期，请在该主机上重新运行登录。健康检查将显示链接问题。

## 故障排除

-   **退出码 127 / 未找到**：`openclaw` 不在非登录 shell 的 PATH 中。将其添加到 `/etc/paths`、您的 shell rc 文件中，或符号链接到 `/usr/local/bin`/`/opt/homebrew/bin`。
-   **健康探测失败**：检查 SSH 可达性、PATH 以及 Baileys 是否已登录 (`openclaw status --json`)。
-   **Web Chat 卡住**：确认网关正在远程主机上运行，并且转发的端口与网关 WS 端口匹配；UI 需要健康的 WS 连接。
-   **节点 IP 显示 127.0.0.1**：使用 SSH 隧道时的预期情况。如果您希望网关看到真实的客户端 IP，请将**传输方式**切换为 **Direct (ws/wss)**。
-   **语音唤醒**：触发短语在远程模式下会自动转发；不需要单独的转发器。

## 通知声音

通过脚本使用 `openclaw` 和 `node.invoke` 为每个通知选择声音，例如：

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Remote gateway ready" --sound Glass
```

应用中不再有全局的“默认声音”开关；调用方为每个请求选择声音（或无声音）。

[macOS 权限](./permissions.md)[macOS 签名](./signing.md)