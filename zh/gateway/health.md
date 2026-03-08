

  配置与操作

  
# 健康检查

无需猜测，快速验证通道连接性的简短指南。

## 快速检查

-   `openclaw status` — 本地摘要：网关可达性/模式、更新提示、已链接通道认证时间、会话及近期活动。
-   `openclaw status --all` — 完整的本地诊断（只读、彩色输出，可安全粘贴用于调试）。
-   `openclaw status --deep` — 同时探测正在运行的网关（在支持时进行逐通道探测）。
-   `openclaw health --json` — 向正在运行的网关请求完整的健康快照（仅限 WebSocket；无直接 Baileys 套接字连接）。
-   在 WhatsApp/WebChat 中发送 `/status` 作为独立消息，以获取状态回复而无需调用智能体。
-   日志：查看 `/tmp/openclaw/openclaw-*.log` 并过滤 `web-heartbeat`、`web-reconnect`、`web-auto-reply`、`web-inbound`。

## 深度诊断

-   磁盘上的凭据：`ls -l ~/.openclaw/credentials/whatsapp//creds.json`（修改时间应为近期）。
-   会话存储：`ls -l ~/.openclaw/agents//sessions/sessions.json`（路径可在配置中覆盖）。数量和近期收件人信息可通过 `status` 命令查看。
-   重新链接流程：当日志中出现状态码 409–515 或 `loggedOut` 时，执行 `openclaw channels logout && openclaw channels login --verbose`。（注意：QR 登录流程在配对后遇到状态 515 时会自动重启一次。）

## 当出现故障时

-   `logged out` 或状态码 409–515 → 使用 `openclaw channels logout` 然后 `openclaw channels login` 重新链接。
-   网关不可达 → 启动它：`openclaw gateway --port 18789`（如果端口被占用，使用 `--force`）。
-   无入站消息 → 确认已链接的手机在线且发送者被允许（`channels.whatsapp.allowFrom`）；对于群组聊天，确保允许列表和提及规则匹配（`channels.whatsapp.groups`、`agents.list[].groupChat.mentionPatterns`）。

## 专用的“健康”命令

`openclaw health --json` 向正在运行的网关请求其健康快照（CLI 不直接连接通道套接字）。它会报告可用的已链接凭据/认证时间、逐通道探测摘要、会话存储摘要以及探测持续时间。如果网关不可达或探测失败/超时，命令将以非零状态退出。使用 `--timeout ` 来覆盖默认的 10 秒超时设置。

[可信代理认证](./trusted-proxy-auth.md)[心跳机制](./heartbeat.md)

---