

  自动化

  
# Webhooks

网关可以对外暴露一个小的 HTTP webhook 端点，用于外部触发。

## 启用

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    // 可选：将显式的 `agentId` 路由限制在此允许列表中。
    // 省略或包含 "*" 以允许任何代理。
    // 设置为 [] 以拒绝所有显式的 `agentId` 路由。
    allowedAgentIds: ["hooks", "main"],
  },
}
```

注意：

-   当 `hooks.enabled=true` 时，`hooks.token` 是必需的。
-   `hooks.path` 默认为 `/hooks`。

## 认证

每个请求必须包含钩子令牌。建议使用请求头：

-   `Authorization: Bearer ` (推荐)
-   `x-openclaw-token: `
-   查询字符串令牌会被拒绝（`?token=...` 返回 `400`）。

## 端点

### POST /hooks/wake

负载：

```json
{ "text": "System line", "mode": "now" }
```

-   `text` **必需** (字符串)：事件的描述（例如，“收到新邮件”）。
-   `mode` 可选 (`now` | `next-heartbeat`)：是触发即时心跳（默认 `now`）还是等待下一次定期检查。

效果：

-   为 **主** 会话入队一个系统事件
-   如果 `mode=now`，触发即时心跳

### POST /hooks/agent

负载：

```json
{
  "message": "Run this",
  "name": "Email",
  "agentId": "hooks",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

-   `message` **必需** (字符串)：供代理处理的提示或消息。
-   `name` 可选 (字符串)：钩子的人类可读名称（例如，“GitHub”），用作会话摘要的前缀。
-   `agentId` 可选 (字符串)：将此钩子路由到特定代理。未知 ID 会回退到默认代理。设置后，钩子将使用已解析代理的工作空间和配置运行。
-   `sessionKey` 可选 (字符串)：用于标识代理会话的密钥。默认情况下，除非 `hooks.allowRequestSessionKey=true`，否则此字段会被拒绝。
-   `wakeMode` 可选 (`now` | `next-heartbeat`)：是触发即时心跳（默认 `now`）还是等待下一次定期检查。
-   `deliver` 可选 (布尔值)：如果为 `true`，代理的响应将被发送到消息通道。默认为 `true`。仅为心跳确认的响应会自动跳过。
-   `channel` 可选 (字符串)：用于传递的消息通道。可选值：`last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (插件), `signal`, `imessage`, `msteams`。默认为 `last`。
-   `to` 可选 (字符串)：通道的接收者标识符（例如，WhatsApp/Signal 的电话号码，Telegram 的聊天 ID，Discord/Slack/Mattermost (插件) 的频道 ID，MS Teams 的对话 ID）。默认为主会话中的最后一个接收者。
-   `model` 可选 (字符串)：模型覆盖（例如，`anthropic/claude-3-5-sonnet` 或别名）。如果有限制，必须在允许的模型列表中。
-   `thinking` 可选 (字符串)：思考级别覆盖（例如，`low`, `medium`, `high`）。
-   `timeoutSeconds` 可选 (数字)：代理运行的最大持续时间（秒）。

效果：

-   运行一个**独立的**代理回合（自己的会话密钥）
-   总是在**主**会话中发布摘要
-   如果 `wakeMode=now`，触发即时心跳

## 会话密钥策略（重大变更）

默认情况下，`/hooks/agent` 负载中的 `sessionKey` 覆盖是禁用的。

-   推荐：设置固定的 `hooks.defaultSessionKey` 并保持请求覆盖关闭。
-   可选：仅在需要时允许请求覆盖，并限制前缀。

推荐配置：

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

兼容性配置（旧版行为）：

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // 强烈推荐
  },
}
```

### POST /hooks/&lt;name&gt; (映射)

自定义钩子名称通过 `hooks.mappings` 解析（参见配置）。映射可以将任意负载转换为 `wake` 或 `agent` 操作，并可选择使用模板或代码转换。映射选项（摘要）：

-   `hooks.presets: ["gmail"]` 启用内置的 Gmail 映射。
-   `hooks.mappings` 允许你在配置中定义 `match`、`action` 和模板。
-   `hooks.transformsDir` + `transform.module` 加载 JS/TS 模块以实现自定义逻辑。
    -   `hooks.transformsDir`（如果设置）必须位于你的 OpenClaw 配置目录下的 transforms 根目录内（通常是 `~/.openclaw/hooks/transforms`）。
    -   `transform.module` 必须在有效的 transforms 目录内解析（遍历/转义路径会被拒绝）。
-   使用 `match.source` 来保持一个通用的接收端点（基于负载的路由）。
-   TS 转换需要 TS 加载器（例如 `bun` 或 `tsx`）或在运行时预编译的 `.js`。
-   在映射上设置 `deliver: true` + `channel`/`to` 以将回复路由到聊天界面（`channel` 默认为 `last`，并回退到 WhatsApp）。
-   `agentId` 将钩子路由到特定代理；未知 ID 回退到默认代理。
-   `hooks.allowedAgentIds` 限制显式的 `agentId` 路由。省略它（或包含 `*`）以允许任何代理。设置为 `[]` 以拒绝显式的 `agentId` 路由。
-   `hooks.defaultSessionKey` 设置当没有提供显式密钥时，钩子代理运行的默认会话。
-   `hooks.allowRequestSessionKey` 控制 `/hooks/agent` 负载是否可以设置 `sessionKey`（默认：`false`）。
-   `hooks.allowedSessionKeyPrefixes` 可选地限制来自请求负载和映射的显式 `sessionKey` 值。
-   `allowUnsafeExternalContent: true` 为该钩子禁用外部内容安全包装（危险；仅适用于受信任的内部来源）。
-   `openclaw webhooks gmail setup` 为 `openclaw webhooks gmail run` 写入 `hooks.gmail` 配置。完整的 Gmail 监视流程请参见 [Gmail Pub/Sub](./gmail-pubsub.md)。

## 响应

-   `/hooks/wake` 返回 `200`
-   `/hooks/agent` 返回 `200`（异步运行已接受）
-   认证失败返回 `401`
-   同一客户端多次认证失败后返回 `429`（检查 `Retry-After`）
-   无效负载返回 `400`
-   负载过大返回 `413`

## 示例

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### 使用不同的模型

在代理负载（或映射）中添加 `model` 以覆盖该次运行的模型：

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

如果你强制执行 `agents.defaults.models`，请确保覆盖模型包含在其中。

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## 安全

-   将钩子端点保持在环回、tailnet 或受信任的反向代理之后。
-   使用专用的钩子令牌；不要重用网关认证令牌。
-   同一客户端地址的重复认证失败会被限速，以减缓暴力破解尝试。
-   如果你使用多代理路由，请设置 `hooks.allowedAgentIds` 以限制显式的 `agentId` 选择。
-   除非你需要调用者选择会话，否则保持 `hooks.allowRequestSessionKey=false`。
-   如果你启用请求 `sessionKey`，请限制 `hooks.allowedSessionKeyPrefixes`（例如，`["hook:"]`）。
-   避免在 webhook 日志中包含敏感的原始负载。
-   钩子负载默认被视为不受信任，并用安全边界包装。如果你必须为特定钩子禁用此功能，请在该钩子的映射中设置 `allowUnsafeExternalContent: true`（危险）。

[自动化故障排除](./troubleshooting.md)[Gmail PubSub](./gmail-pubsub.md)