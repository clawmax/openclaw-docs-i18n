

  协议与 API

  
# Bridge 协议

Bridge 协议是一种**旧版**节点传输协议（TCP JSONL）。新的节点客户端应改用统一的 Gateway WebSocket 协议。如果您正在构建操作器或节点客户端，请使用 [Gateway 协议](./protocol.md)。**注意：** 当前的 OpenClaw 构建版本已不再包含 TCP bridge 监听器；本文档仅作历史参考。旧版的 `bridge.*` 配置键已不再是配置模式的一部分。

## 为何两者并存

-   **安全边界**：bridge 暴露一个小的允许列表，而非完整的网关 API 接口。
-   **配对与节点身份**：节点准入由网关管理，并与每个节点的令牌绑定。
-   **发现用户体验**：节点可以通过局域网上的 Bonjour 发现网关，或通过 tailnet 直接连接。
-   **环回 WS**：完整的 WS 控制平面保持本地化，除非通过 SSH 隧道传输。

## 传输

-   TCP，每行一个 JSON 对象（JSONL）。
-   可选 TLS（当 `bridge.tls.enabled` 为 true 时）。
-   旧版默认监听端口为 `18790`（当前构建版本不启动 TCP bridge）。

启用 TLS 时，发现 TXT 记录包含 `bridgeTls=1` 以及作为非机密提示的 `bridgeTlsSha256`。请注意，Bonjour/mDNS TXT 记录未经身份验证；客户端不得将广告的指纹视为权威的固定值，除非有明确的用户意图或其他带外验证。

## 握手与配对

1.  客户端发送包含节点元数据和令牌（如果已配对）的 `hello`。
2.  如果未配对，网关回复 `error`（`NOT_PAIRED`/`UNAUTHORIZED`）。
3.  客户端发送 `pair-request`。
4.  网关等待批准，然后发送 `pair-ok` 和 `hello-ok`。

`hello-ok` 返回 `serverName`，并可能包含 `canvasHostUrl`。

## 帧

客户端 → 网关：

-   `req` / `res`：作用域内的网关 RPC（聊天、会话、配置、健康、voicewake、skills.bins）
-   `event`：节点信号（语音转录、代理请求、聊天订阅、执行生命周期）

网关 → 客户端：

-   `invoke` / `invoke-res`：节点命令（`canvas.*`、`camera.*`、`screen.record`、`location.get`、`sms.send`）
-   `event`：已订阅会话的聊天更新
-   `ping` / `pong`：保活

旧版允许列表强制执行位于 `src/gateway/server-bridge.ts`（已移除）。

## 执行生命周期事件

节点可以发出 `exec.finished` 或 `exec.denied` 事件来展示 system.run 活动。这些事件在网关中被映射为系统事件。（旧版节点可能仍会发出 `exec.started`。）有效载荷字段（除非注明，否则均为可选）：

-   `sessionKey`（必需）：接收系统事件的代理会话。
-   `runId`：用于分组的唯一执行 ID。
-   `command`：原始或格式化的命令字符串。
-   `exitCode`、`timedOut`、`success`、`output`：完成详情（仅限 finished 事件）。
-   `reason`：拒绝原因（仅限 denied 事件）。

## Tailnet 使用

-   将 bridge 绑定到 tailnet IP：在 `~/.openclaw/openclaw.json` 中设置 `bridge.bind: "tailnet"`。
-   客户端通过 MagicDNS 名称或 tailnet IP 连接。
-   Bonjour **不**跨网络工作；需要时请使用手动主机/端口或广域 DNS‑SD。

## 版本控制

Bridge 目前为**隐式 v1**（无最小/最大版本协商）。预期保持向后兼容；在进行任何破坏性更改之前，应添加 bridge 协议版本字段。

[Gateway 协议](./protocol.md)[OpenAI 聊天补全](./openai-http-api.md)

---