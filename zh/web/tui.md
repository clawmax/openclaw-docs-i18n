

  Web 界面

  
# TUI

## 快速开始

1.  启动网关。

```bash
openclaw gateway
```

2.  打开 TUI。

```bash
openclaw tui
```

3.  输入消息并按回车键。

远程网关：

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

如果您的网关使用密码认证，请使用 `--password`。

## 界面概览

-   标题栏：连接 URL、当前代理、当前会话。
-   聊天记录：用户消息、助手回复、系统通知、工具卡片。
-   状态行：连接/运行状态（连接中、运行中、流式传输、空闲、错误）。
-   页脚：连接状态 + 代理 + 会话 + 模型 + 思考/详细/推理 + 令牌计数 + 交付。
-   输入框：带自动补全的文本编辑器。

## 核心概念：代理 + 会话

-   代理是唯一的标识符（例如 `main`、`research`）。网关会暴露此列表。
-   会话属于当前代理。
-   会话键存储为 `agent::`。
    -   如果您输入 `/session main`，TUI 会将其扩展为 `agent::main`。
    -   如果您输入 `/session agent:other:main`，您将显式切换到该代理的会话。
-   会话作用域：
    -   `per-sender`（默认）：每个代理拥有多个会话。
    -   `global`：TUI 始终使用 `global` 会话（选择器可能为空）。
-   当前代理 + 会话始终显示在页脚中。

## 发送 + 交付

-   消息被发送到网关；默认情况下不向提供商交付。
-   开启交付：
    -   `/deliver on`
    -   或通过设置面板
    -   或启动时使用 `openclaw tui --deliver`

## 选择器 + 覆盖层

-   模型选择器：列出可用模型并设置会话覆盖。
-   代理选择器：选择不同的代理。
-   会话选择器：仅显示当前代理的会话。
-   设置：切换交付、工具输出展开和思考可见性。

## 键盘快捷键

-   Enter：发送消息
-   Esc：中止活动运行
-   Ctrl+C：清空输入（按两次退出）
-   Ctrl+D：退出
-   Ctrl+L：模型选择器
-   Ctrl+G：代理选择器
-   Ctrl+P：会话选择器
-   Ctrl+O：切换工具输出展开
-   Ctrl+T：切换思考可见性（重新加载历史记录）

## 斜杠命令

核心命令：

-   `/help`
-   `/status`
-   `/agent `（或 `/agents`）
-   `/session `（或 `/sessions`）
-   `/model <provider/model>`（或 `/models`）

会话控制：

-   `/think <off|minimal|low|medium|high>`
-   `/verbose <on|full|off>`
-   `/reasoning <on|off|stream>`
-   `/usage <off|tokens|full>`
-   `/elevated <on|off|ask|full>`（别名：`/elev`）
-   `/activation <mention|always>`
-   `/deliver <on|off>`

会话生命周期：

-   `/new` 或 `/reset`（重置会话）
-   `/abort`（中止活动运行）
-   `/settings`
-   `/exit`

其他网关斜杠命令（例如 `/context`）会转发到网关并显示为系统输出。请参阅[斜杠命令](../tools/slash-commands.md)。

## 本地 Shell 命令

-   在一行前加上 `!` 以在 TUI 主机上运行本地 shell 命令。
-   TUI 每个会话会提示一次以允许本地执行；拒绝则在该会话中保持 `!` 禁用。
-   命令在 TUI 工作目录的新建、非交互式 shell 中运行（无持久化的 `cd`/环境变量）。
-   本地 shell 命令在其环境中接收 `OPENCLAW_SHELL=tui-local`。
-   单独的 `!` 会作为普通消息发送；前导空格不会触发本地执行。

## 工具输出

-   工具调用显示为包含参数 + 结果的卡片。
-   Ctrl+O 在折叠/展开视图之间切换。
-   工具运行时，部分更新会流式传输到同一张卡片中。

## 历史记录 + 流式传输

-   连接时，TUI 加载最新的历史记录（默认 200 条消息）。
-   流式响应会在原位置更新，直到最终确定。
-   TUI 还会监听代理工具事件以获取更丰富的工具卡片。

## 连接详情

-   TUI 以 `mode: "tui"` 向网关注册。
-   重新连接会显示系统消息；事件间隙会在日志中显示。

## 选项

-   `--url `：网关 WebSocket URL（默认为配置或 `ws://127.0.0.1:`）
-   `--token `：网关令牌（如果需要）
-   `--password `：网关密码（如果需要）
-   `--session `：会话键（默认：`main`，或作用域为 global 时的 `global`）
-   `--deliver`：将助手回复交付给提供商（默认关闭）
-   `--thinking `：覆盖发送时的思考级别
-   `--timeout-ms `：代理超时时间（毫秒）（默认为 `agents.defaults.timeoutSeconds`）

注意：当您设置 `--url` 时，TUI 不会回退到配置或环境凭据。请显式传递 `--token` 或 `--password`。缺少显式凭据会导致错误。

## 故障排除

发送消息后无输出：

-   在 TUI 中运行 `/status` 以确认网关已连接且处于空闲/繁忙状态。
-   检查网关日志：`openclaw logs --follow`。
-   确认代理可以运行：`openclaw status` 和 `openclaw models status`。
-   如果您期望在聊天频道中看到消息，请启用交付（`/deliver on` 或 `--deliver`）。
-   `--history-limit `：要加载的历史记录条目数（默认 200）

## 连接故障排除

-   `disconnected`：确保网关正在运行，并且您的 `--url/--token/--password` 正确。
-   选择器中无代理：检查 `openclaw agents list` 和您的路由配置。
-   会话选择器为空：您可能处于全局作用域或尚未创建任何会话。

[WebChat](./webchat.md)