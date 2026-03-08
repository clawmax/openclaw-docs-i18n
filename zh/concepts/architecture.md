

  基础

  
# 网关架构

最后更新：2026-01-22

## 概述

-   一个长期运行的**网关**拥有所有消息界面（通过 Baileys 的 WhatsApp、通过 grammY 的 Telegram、Slack、Discord、Signal、iMessage、WebChat）。
-   控制平面客户端（macOS 应用、CLI、Web UI、自动化程序）通过 **WebSocket** 连接到网关，连接到配置的绑定主机（默认 `127.0.0.1:18789`）。
-   **节点**（macOS/iOS/Android/无头设备）也通过 **WebSocket** 连接，但需声明 `role: node` 并附带明确的 caps/commands。
-   每个主机一个网关；这是唯一打开 WhatsApp 会话的地方。
-   **画布主机**由网关 HTTP 服务器提供，位于：
    -   `/__openclaw__/canvas/`（代理可编辑的 HTML/CSS/JS）
    -   `/__openclaw__/a2ui/`（A2UI 主机）它使用与网关相同的端口（默认 `18789`）。

## 组件与流程

### 网关（守护进程）

-   维护提供商连接。
-   暴露类型化的 WS API（请求、响应、服务器推送事件）。
-   根据 JSON Schema 验证入站帧。
-   发出事件，如 `agent`、`chat`、`presence`、`health`、`heartbeat`、`cron`。

### 客户端（mac 应用 / CLI / Web 管理界面）

-   每个客户端一个 WS 连接。
-   发送请求（`health`、`status`、`send`、`agent`、`system-presence`）。
-   订阅事件（`tick`、`agent`、`presence`、`shutdown`）。

### 节点（macOS / iOS / Android / 无头设备）

-   连接到**同一个 WS 服务器**，并附带 `role: node`。
-   在 `connect` 中提供设备身份；配对是**基于设备**的（角色 `node`），批准信息存储在设备配对存储中。
-   暴露命令，如 `canvas.*`、`camera.*`、`screen.record`、`location.get`。

协议详情：

-   [网关协议](../gateway/protocol.md)

### WebChat

-   使用网关 WS API 获取聊天记录和发送消息的静态 UI。
-   在远程设置中，通过与其他客户端相同的 SSH/Tailscale 隧道连接。

## 连接生命周期（单个客户端）

## 有线协议（摘要）

-   传输：WebSocket，文本帧携带 JSON 负载。
-   第一帧**必须**是 `connect`。
-   握手后：
    -   请求：`{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
    -   事件：`{type:"event", event, payload, seq?, stateVersion?}`
-   如果设置了 `OPENCLAW_GATEWAY_TOKEN`（或 `--token`），则 `connect.params.auth.token` 必须匹配，否则套接字关闭。
-   具有副作用的命令（`send`、`agent`）需要幂等键以安全重试；服务器维护一个短期的去重缓存。
-   节点必须在 `connect` 中包含 `role: "node"` 以及 caps/commands/permissions。

## 配对 + 本地信任

-   所有 WS 客户端（操作员 + 节点）在 `connect` 时都包含一个**设备身份**。
-   新的设备 ID 需要配对批准；网关会为后续连接颁发**设备令牌**。
-   **本地**连接（环回地址或网关主机自身的 tailnet 地址）可以自动批准，以保持同主机用户体验流畅。
-   所有连接必须对 `connect.challenge` 随机数进行签名。
-   签名负载 `v3` 也绑定 `platform` + `deviceFamily`；网关在重新连接时固定已配对的元数据，并在元数据更改时要求重新配对。
-   **非本地**连接仍需要明确批准。
-   网关认证（`gateway.auth.*`）仍然适用于**所有**连接，无论是本地还是远程。

详情：[网关协议](../gateway/protocol.md)、[配对](../channels/pairing.md)、[安全](../gateway/security.md)。

## 协议类型与代码生成

-   TypeBox 模式定义协议。
-   JSON Schema 从这些模式生成。
-   Swift 模型从 JSON Schema 生成。

## 远程访问

-   首选：Tailscale 或 VPN。
-   备选：SSH 隧道
    
    复制
    
    ```bash
    ssh -N -L 18789:127.0.0.1:18789 user@host
    ```
    
-   相同的握手 + 认证令牌通过隧道应用。
-   在远程设置中，可以为 WS 启用 TLS + 可选固定。

## 运维快照

-   启动：`openclaw gateway`（前台运行，日志输出到 stdout）。
-   健康检查：通过 WS 发送 `health`（也包含在 `hello-ok` 中）。
-   监管：使用 launchd/systemd 实现自动重启。

## 不变式

-   每个主机上恰好有一个网关控制一个 Baileys 会话。
-   握手是强制性的；任何非 JSON 或非 connect 的第一帧都会导致硬关闭。
-   事件不重放；客户端必须在出现间隙时刷新。

[Pi 集成架构](../pi.md)[代理运行时](./agent.md)