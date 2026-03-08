

  协议与 API

  
# 工具调用 API

OpenClaw 网关公开了一个简单的 HTTP 端点，用于直接调用单个工具。该端点始终启用，但受网关认证和工具策略控制。

-   `POST /tools/invoke`
-   与网关（WS + HTTP 多路复用）使用相同端口：`http://<gateway-host>:/tools/invoke`

默认最大负载大小为 2 MB。

## 认证

使用网关的认证配置。发送 Bearer 令牌：

-   `Authorization: Bearer `

注意：

-   当 `gateway.auth.mode="token"` 时，使用 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）。
-   当 `gateway.auth.mode="password"` 时，使用 `gateway.auth.password`（或 `OPENCLAW_GATEWAY_PASSWORD`）。
-   如果配置了 `gateway.auth.rateLimit` 且认证失败次数过多，端点将返回 `429` 并附带 `Retry-After`。

## 请求体

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

字段说明：

-   `tool` (字符串，必需)：要调用的工具名称。
-   `action` (字符串，可选)：如果工具模式支持 `action` 且参数负载中省略了它，则映射到 args 中。
-   `args` (对象，可选)：工具特定的参数。
-   `sessionKey` (字符串，可选)：目标会话键。如果省略或为 `"main"`，网关将使用配置的主会话键（遵循 `session.mainKey` 和默认代理，或全局作用域中的 `global`）。
-   `dryRun` (布尔值，可选)：保留供将来使用；目前被忽略。

## 策略与路由行为

工具的可用性通过网关代理使用的相同策略链进行过滤：

-   `tools.profile` / `tools.byProvider.profile`
-   `tools.allow` / `tools.byProvider.allow`
-   `agents..tools.allow` / `agents..tools.byProvider.allow`
-   组策略（如果会话键映射到组或频道）
-   子代理策略（当使用子代理会话键调用时）

如果策略不允许某个工具，端点将返回 **404**。网关 HTTP 默认还应用一个硬性拒绝列表（即使会话策略允许该工具）：

-   `sessions_spawn`
-   `sessions_send`
-   `gateway`
-   `whatsapp_login`

您可以通过 `gateway.tools` 自定义此拒绝列表：

```json
{
  gateway: {
    tools: {
      // 通过 HTTP /tools/invoke 额外阻止的工具
      deny: ["browser"],
      // 从默认拒绝列表中移除工具
      allow: ["gateway"],
    },
  },
}
```

为了帮助组策略解析上下文，您可以选择性地设置：

-   `x-openclaw-message-channel: ` (例如：`slack`, `telegram`)
-   `x-openclaw-account-id: ` (当存在多个账户时)

## 响应

-   `200` → `{ ok: true, result }`
-   `400` → `{ ok: false, error: { type, message } }` (无效请求或工具输入错误)
-   `401` → 未授权
-   `429` → 认证速率限制（设置了 `Retry-After`）
-   `404` → 工具不可用（未找到或未在允许列表中）
-   `405` → 方法不允许
-   `500` → `{ ok: false, error: { type, message } }` (意外的工具执行错误；消息已脱敏)

## 示例

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```

[OpenResponses API](./openresponses-http-api.md)[CLI 后端](./cli-backends.md)