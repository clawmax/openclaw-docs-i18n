

  协议与 API

  
# OpenAI 聊天补全

OpenClaw 的 Gateway 可以提供一个轻量级的 OpenAI 兼容的聊天补全端点。此端点**默认禁用**。请先在配置中启用。

-   `POST /v1/chat/completions`
-   与 Gateway 相同的端口（WS + HTTP 复用）：`http://<gateway-host>:/v1/chat/completions`

在底层，请求作为一次普通的 Gateway 代理运行执行（与 `openclaw agent` 相同的代码路径），因此路由/权限/配置与您的 Gateway 匹配。

## 认证

使用 Gateway 的认证配置。发送一个 bearer 令牌：

-   `Authorization: Bearer `

注意：

-   当 `gateway.auth.mode="token"` 时，使用 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）。
-   当 `gateway.auth.mode="password"` 时，使用 `gateway.auth.password`（或 `OPENCLAW_GATEWAY_PASSWORD`）。
-   如果配置了 `gateway.auth.rateLimit` 且认证失败次数过多，端点将返回 `429` 并附带 `Retry-After`。

## 安全边界（重要）

请将此端点视为网关实例的**完全操作员访问**接口。

-   此处的 HTTP bearer 认证不是细粒度的每用户范围模型。
-   此端点的有效 Gateway 令牌/密码应被视为所有者/操作员凭据。
-   请求通过与受信任操作员操作相同的控制平面代理路径运行。
-   此端点没有单独的非所有者/每用户工具边界；一旦调用者通过此处的 Gateway 认证，OpenClaw 就将该调用者视为此网关的受信任操作员。
-   如果目标代理策略允许敏感工具，此端点可以使用它们。
-   请仅将此端点保留在环回/Tailnet/私有入口；不要直接将其暴露在公共互联网上。

参见[安全性](./security.md)和[远程访问](./remote.md)。

## 选择代理

无需自定义请求头：将代理 ID 编码在 OpenAI 的 `model` 字段中：

-   `model: "openclaw:"`（例如：`"openclaw:main"`, `"openclaw:beta"`）
-   `model: "agent:"`（别名）

或者通过请求头指定特定的 OpenClaw 代理：

-   `x-openclaw-agent-id: `（默认：`main`）

高级用法：

-   `x-openclaw-session-key: ` 以完全控制会话路由。

## 启用端点

将 `gateway.http.endpoints.chatCompletions.enabled` 设置为 `true`：

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## 禁用端点

将 `gateway.http.endpoints.chatCompletions.enabled` 设置为 `false`：

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## 会话行为

默认情况下，该端点是**每次请求无状态**的（每次调用都会生成一个新的会话密钥）。如果请求包含一个 OpenAI `user` 字符串，Gateway 会从中派生出稳定的会话密钥，因此重复调用可以共享一个代理会话。

## 流式传输 (SSE)

设置 `stream: true` 以接收服务器发送事件 (SSE)：

-   `Content-Type: text/event-stream`
-   每个事件行格式为 `data: `
-   流以 `data: [DONE]` 结束

## 示例

非流式：

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

流式：

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```

[桥接协议](./bridge-protocol.md)[OpenResponses API](./openresponses-http-api.md)