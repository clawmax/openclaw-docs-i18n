

  macOS 伴侣应用

  
# macOS 上的网关

OpenClaw.app 不再捆绑 Node/Bun 或网关运行时。macOS 应用期望一个**外部**安装的 `openclaw` CLI，不会将网关作为子进程启动，而是管理一个每用户的 launchd 服务来保持网关运行（或者，如果已有本地网关在运行，则附加到该网关）。

## 安装 CLI（本地模式必需）

您的 Mac 需要 Node 22+，然后全局安装 `openclaw`：

```bash
npm install -g openclaw@<version>
```

macOS 应用的 **安装 CLI** 按钮通过 npm/pnpm 运行相同的流程（不推荐将 bun 用于网关运行时）。

## Launchd（网关作为 LaunchAgent）

标签：

-   `ai.openclaw.gateway`（或 `ai.openclaw.`；遗留的 `com.openclaw.*` 可能仍存在）

Plist 位置（每用户）：

-   `~/Library/LaunchAgents/ai.openclaw.gateway.plist`（或 `~/Library/LaunchAgents/ai.openclaw..plist`）

管理器：

-   在本地模式下，macOS 应用负责 LaunchAgent 的安装/更新。
-   CLI 也可以安装它：`openclaw gateway install`。

行为：

-   “OpenClaw 活跃” 启用/禁用 LaunchAgent。
-   应用退出**不会**停止网关（launchd 会保持其存活）。
-   如果网关已在配置的端口上运行，应用将附加到该网关，而不是启动新的网关。

日志：

-   launchd 标准输出/错误：`/tmp/openclaw/openclaw-gateway.log`

## 版本兼容性

macOS 应用会检查网关版本与其自身版本的兼容性。如果它们不兼容，请更新全局 CLI 以匹配应用版本。

## 冒烟测试

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

然后：

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```

[macOS 版本发布](./release.md)[macOS IPC](./xpc.md)