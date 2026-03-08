

  Web 界面

  
# 控制界面

控制界面是一个由网关提供的小型 **Vite + Lit** 单页应用：

-   默认地址：`http://:18789/`
-   可选前缀：设置 `gateway.controlUi.basePath`（例如 `/openclaw`）

它**直接与同一端口上的网关 WebSocket** 通信。

## 快速打开（本地）

如果网关在同一台计算机上运行，请打开：

-   [http://127.0.0.1:18789/](http://127.0.0.1:18789/)（或 [http://localhost:18789/](http://localhost:18789/)）

如果页面加载失败，请先启动网关：`openclaw gateway`。认证信息在 WebSocket 握手期间通过以下方式提供：

-   `connect.params.auth.token`
-   `connect.params.auth.password` 仪表板设置面板允许您存储令牌；密码不会被持久化。默认情况下，入门向导会生成一个网关令牌，因此在首次连接时将其粘贴到此处。

## 设备配对（首次连接）

当您从新的浏览器或设备连接到控制界面时，网关需要**一次性配对批准**——即使您在同一 Tailnet 上且设置了 `gateway.auth.allowTailscale: true`。这是一项防止未授权访问的安全措施。**您将看到：**“已断开连接 (1008)：需要配对”**批准设备：**

```bash
# 列出待处理的请求
openclaw devices list

# 通过请求 ID 批准
openclaw devices approve <requestId>
```

一旦批准，设备将被记住，除非您使用 `openclaw devices revoke --device  --role ` 撤销，否则不需要重新批准。有关令牌轮换和撤销，请参阅[设备 CLI](../cli/devices.md)。**注意：**

-   本地连接（`127.0.0.1`）会自动批准。
-   远程连接（局域网、Tailnet 等）需要明确批准。
-   每个浏览器配置文件会生成唯一的设备 ID，因此切换浏览器或清除浏览器数据将需要重新配对。

## 语言支持

控制界面可以在首次加载时根据您的浏览器语言环境进行本地化，之后您可以在“访问”卡片中的语言选择器中覆盖它。

-   支持的语言环境：`en`、`zh-CN`、`zh-TW`、`pt-BR`、`de`、`es`
-   非英语翻译在浏览器中延迟加载。
-   所选语言环境保存在浏览器存储中，并在未来访问时重用。
-   缺少的翻译键会回退到英语。

## 当前功能

-   通过网关 WS 与模型聊天（`chat.history`、`chat.send`、`chat.abort`、`chat.inject`）
-   在聊天中流式传输工具调用 + 实时工具输出卡片（智能体事件）
-   频道：WhatsApp/Telegram/Discord/Slack + 插件频道（Mattermost 等）状态 + QR 登录 + 每频道配置（`channels.status`、`web.login.*`、`config.patch`）
-   实例：在线列表 + 刷新（`system-presence`）
-   会话：列表 + 每个会话的思考/详细覆盖（`sessions.list`、`sessions.patch`）
-   定时任务：列表/添加/编辑/运行/启用/禁用 + 运行历史（`cron.*`）
-   技能：状态、启用/禁用、安装、API 密钥更新（`skills.*`）
-   节点：列表 + 能力（`node.list`）
-   执行批准：编辑网关或节点允许列表 + 为 `exec host=gateway/node` 询问策略（`exec.approvals.*`）
-   配置：查看/编辑 `~/.openclaw/openclaw.json`（`config.get`、`config.set`）
-   配置：应用 + 带验证的重启（`config.apply`）并唤醒最后活跃的会话
-   配置写入包含基础哈希保护，以防止覆盖并发编辑
-   配置模式 + 表单渲染（`config.schema`，包括插件 + 频道模式）；原始 JSON 编辑器仍然可用
-   调试：状态/健康/模型快照 + 事件日志 + 手动 RPC 调用（`status`、`health`、`models.list`）
-   日志：网关文件日志的实时跟踪，带过滤/导出（`logs.tail`）
-   更新：运行包/git 更新 + 重启（`update.run`）并附带重启报告

定时任务面板说明：

-   对于隔离任务，交付默认设置为通知摘要。如果您希望仅内部运行，可以切换到无通知。
-   当选择通知时，会显示频道/目标字段。
-   Webhook 模式使用 `delivery.mode = "webhook"`，并将 `delivery.to` 设置为有效的 HTTP(S) webhook URL。
-   对于主会话任务，可使用 webhook 和无通知交付模式。
-   高级编辑控件包括运行后删除、清除智能体覆盖、定时任务精确/交错选项、智能体模型/思考覆盖以及尽力交付切换。
-   表单验证是内联的，带有字段级错误；无效值会禁用保存按钮，直到修复。
-   设置 `cron.webhookToken` 以发送专用的承载令牌，如果省略，则 webhook 将在没有认证头的情况下发送。
-   已弃用的回退：存储的遗留任务如果带有 `notify: true`，在迁移前仍可使用 `cron.webhook`。

## 聊天行为

-   `chat.send` 是**非阻塞的**：它会立即确认并返回 `{ runId, status: "started" }`，响应通过 `chat` 事件流式传输。
-   使用相同的 `idempotencyKey` 重新发送，在运行时会返回 `{ status: "in_flight" }`，完成后返回 `{ status: "ok" }`。
-   `chat.history` 响应的大小受限于 UI 安全。当转录条目过大时，网关可能会截断长文本字段、省略繁重的元数据块，并用占位符替换过大的消息（`[chat.history omitted: message too large]`）。
-   `chat.inject` 将助手备注附加到会话转录中，并广播 `chat` 事件以进行仅 UI 更新（无智能体运行，无频道交付）。
-   停止：
    -   点击**停止**（调用 `chat.abort`）
    -   输入 `/stop`（或独立的停止短语如 `stop`、`stop action`、`stop run`、`stop openclaw`、`please stop`）以在带外中止
    -   `chat.abort` 支持 `{ sessionKey }`（无 `runId`）以中止该会话的所有活跃运行
-   中止部分保留：
    -   当运行被中止时，部分助手文本仍可在 UI 中显示
    -   当存在缓冲输出时，网关会将中止的部分助手文本持久化到转录历史中
    -   持久化的条目包含中止元数据，以便转录消费者能够区分中止部分与正常完成输出

## Tailnet 访问（推荐）

### 集成 Tailscale Serve（首选）

将网关保持在环回地址，让 Tailscale Serve 通过 HTTPS 代理它：

```bash
openclaw gateway --tailscale serve
```

打开：

-   `https:///`（或您配置的 `gateway.controlUi.basePath`）

默认情况下，当 `gateway.auth.allowTailscale` 为 `true` 时，控制界面/WebSocket Serve 请求可以通过 Tailscale 身份头（`tailscale-user-login`）进行身份验证。OpenClaw 通过使用 `tailscale whois` 解析 `x-forwarded-for` 地址并与头匹配来验证身份，并且仅当请求通过 Tailscale 的 `x-forwarded-*` 头到达环回地址时才接受这些请求。如果您希望即使对于 Serve 流量也要求令牌/密码，请设置 `gateway.auth.allowTailscale: false`（或强制 `gateway.auth.mode: "password"`）。无令牌的 Serve 身份验证假定网关主机是可信的。如果该主机上可能运行不受信任的本地代码，则要求令牌/密码身份验证。

### 绑定到 tailnet + 令牌

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

然后打开：

-   `http://<tailscale-ip>:18789/`（或您配置的 `gateway.controlUi.basePath`）

将令牌粘贴到 UI 设置中（作为 `connect.params.auth.token` 发送）。

## 不安全的 HTTP

如果您通过纯 HTTP（`http://<lan-ip>` 或 `http://<tailscale-ip>`）打开仪表板，浏览器将在**非安全上下文**中运行并阻止 WebCrypto。默认情况下，OpenClaw **会阻止**没有设备身份的控制界面连接。**推荐修复方法：** 使用 HTTPS（Tailscale Serve）或在本地打开 UI：

-   `https:///`（Serve）
-   `http://127.0.0.1:18789/`（在网关主机上）

**不安全身份验证切换行为：**

```json
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`allowInsecureAuth` 不会绕过控制界面设备身份或配对检查。**仅用于紧急情况：**

```json
{
  gateway: {
    controlUi: { dangerouslyDisableDeviceAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`dangerouslyDisableDeviceAuth` 会禁用控制界面设备身份检查，这是一个严重的安全降级。紧急使用后请尽快恢复。有关 HTTPS 设置指导，请参阅 [Tailscale](../gateway/tailscale.md)。

## 构建 UI

网关从 `dist/control-ui` 提供静态文件。使用以下命令构建它们：

```bash
pnpm ui:build # 首次运行时自动安装 UI 依赖
```

可选的绝对基础路径（当您需要固定的资源 URL 时）：

```
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

对于本地开发（单独的开发服务器）：

```bash
pnpm ui:dev # 首次运行时自动安装 UI 依赖
```

然后将 UI 指向您的网关 WS URL（例如 `ws://127.0.0.1:18789`）。

## 调试/测试：开发服务器 + 远程网关

控制界面是静态文件；WebSocket 目标是可配置的，可以与 HTTP 源不同。当您希望在本地运行 Vite 开发服务器但网关运行在其他地方时，这很方便。

1.  启动 UI 开发服务器：`pnpm ui:dev`
2.  打开类似以下的 URL：

```
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

可选的一次性身份验证（如果需要）：

```
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789#token=<gateway-token>
```

注意：

-   `gatewayUrl` 在加载后存储在 localStorage 中，并从 URL 中移除。
-   `token` 被导入到当前标签页的内存中，并从 URL 中剥离；它不存储在 localStorage 中。
-   `password` 仅保存在内存中。
-   当设置了 `gatewayUrl` 时，UI 不会回退到配置或环境凭据。请明确提供 `token`（或 `password`）。缺少明确凭据会导致错误。
-   当网关位于 TLS 之后时（Tailscale Serve、HTTPS 代理等），使用 `wss://`。
-   `gatewayUrl` 仅在最顶层窗口（非嵌入）中被接受，以防止点击劫持。
-   非环回控制界面部署必须明确设置 `gateway.controlUi.allowedOrigins`（完整来源）。这包括远程开发设置。
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` 启用 Host 头来源回退模式，但这是一种危险的安全模式。

示例：

```json
{
  gateway: {
    controlUi: {
      allowedOrigins: ["http://localhost:5173"],
    },
  },
}
```

远程访问设置详情：[远程访问](../gateway/remote.md)。

[Web](../web.md)[仪表板](./dashboard.md)