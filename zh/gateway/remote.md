

  远程访问

  
# 远程访问

本仓库通过在一个专用主机（桌面/服务器）上运行单个网关（主节点）并让客户端连接到它，来支持“通过 SSH 远程访问”。

-   对于**操作员（您 / macOS 应用）**：SSH 隧道是通用的后备方案。
-   对于**节点（iOS/Android 及未来设备）**：连接到**网关 WebSocket**（根据需要，通过 LAN/tailnet 或 SSH 隧道）。

## 核心理念

-   网关 WebSocket 绑定到您配置端口（默认为 18789）的**环回地址**。
-   对于远程使用，您需要通过 SSH 转发该环回端口（或使用 tailnet/VPN 并减少隧道使用）。

## 常见的 VPN/tailnet 设置（代理所在位置）

将**网关主机**视为“代理所在之处”。它拥有会话、身份验证配置文件、通道和状态。您的笔记本电脑/台式机（以及节点）连接到该主机。

### 1) 在您的 tailnet 中始终在线的网关（VPS 或家庭服务器）

在持久性主机上运行网关，并通过 **Tailscale** 或 SSH 访问它。

-   **最佳用户体验：** 保持 `gateway.bind: "loopback"` 并使用 **Tailscale Serve** 来提供控制界面。
-   **后备方案：** 保持环回地址 + 从任何需要访问的机器进行 SSH 隧道连接。
-   **示例：** [exe.dev](../install/exe-dev.md)（简易虚拟机）或 [Hetzner](../install/hetzner.md)（生产环境 VPS）。

当您的笔记本电脑经常休眠，但您希望代理始终在线时，这是理想的选择。

### 2) 家庭台式机运行网关，笔记本电脑作为远程控制器

笔记本电脑**不**运行代理。它远程连接：

-   使用 macOS 应用的**通过 SSH 远程**模式（设置 → 通用 → “OpenClaw 运行位置”）。
-   应用会打开并管理隧道，因此 WebChat 和健康检查“直接可用”。

操作手册：[macOS 远程访问](../platforms/mac/remote.md)。

### 3) 笔记本电脑运行网关，从其他机器进行远程访问

保持网关本地运行，但安全地暴露它：

-   从其他机器通过 SSH 隧道连接到笔记本电脑，或者
-   使用 Tailscale Serve 提供控制界面，并保持网关仅限环回地址访问。

指南：[Tailscale](./tailscale.md) 和 [Web 概览](../web.md)。

## 命令流（什么在哪里运行）

一个网关服务拥有状态和通道。节点是外围设备。流程示例（Telegram → 节点）：

-   Telegram 消息到达**网关**。
-   网关运行**代理**并决定是否调用节点工具。
-   网关通过网关 WebSocket（`node.*` RPC）调用**节点**。
-   节点返回结果；网关回复回 Telegram。

注意：

-   **节点不运行网关服务。** 除非您有意运行隔离的配置文件（参见[多个网关](./multiple-gateways.md)），否则每个主机只应运行一个网关。
-   macOS 应用的“节点模式”只是通过网关 WebSocket 连接的节点客户端。

## SSH 隧道（CLI + 工具）

创建到远程网关 WebSocket 的本地隧道：

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

隧道建立后：

-   `openclaw health` 和 `openclaw status --deep` 现在可以通过 `ws://127.0.0.1:18789` 访问远程网关。
-   `openclaw gateway {status,health,send,agent,call}` 在需要时也可以通过 `--url` 指定转发的 URL 作为目标。

注意：将 `18789` 替换为您配置的 `gateway.port`（或 `--port`/`OPENCLAW_GATEWAY_PORT`）。注意：当您传递 `--url` 时，CLI 不会回退到配置文件或环境变量中的凭据。请明确包含 `--token` 或 `--password`。缺少显式凭据会导致错误。

## CLI 远程默认设置

您可以持久化一个远程目标，以便 CLI 命令默认使用它：

```json
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token",
    },
  },
}
```

当网关仅限环回地址访问时，请将 URL 保持为 `ws://127.0.0.1:18789` 并首先打开 SSH 隧道。

## 凭据优先级

网关调用/探测的凭据解析现在遵循一个共享约定：

-   显式凭据（`--token`、`--password` 或工具 `gatewayToken`）始终优先。
-   本地模式默认值：
    -   token: `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token` -> `gateway.remote.token`
    -   password: `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.auth.password` -> `gateway.remote.password`
-   远程模式默认值：
    -   token: `gateway.remote.token` -> `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token`
    -   password: `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.remote.password` -> `gateway.auth.password`
-   远程探测/状态令牌检查默认是严格的：当目标为远程模式时，它们仅使用 `gateway.remote.token`（没有本地令牌回退）。
-   遗留的 `CLAWDBOT_GATEWAY_*` 环境变量仅由兼容性调用路径使用；探测/状态/身份验证解析仅使用 `OPENCLAW_GATEWAY_*`。

## 通过 SSH 的聊天界面

WebChat 不再使用单独的 HTTP 端口。SwiftUI 聊天界面直接连接到网关 WebSocket。

-   通过 SSH 转发 `18789`（见上文），然后将客户端连接到 `ws://127.0.0.1:18789`。
-   在 macOS 上，建议使用应用的“通过 SSH 远程”模式，该模式会自动管理隧道。

## macOS 应用“通过 SSH 远程”

macOS 菜单栏应用可以端到端驱动相同的设置（远程状态检查、WebChat 和语音唤醒转发）。操作手册：[macOS 远程访问](../platforms/mac/remote.md)。

## 安全规则（远程/VPN）

简短版本：**保持网关仅限环回地址访问**，除非您确定需要绑定到其他地址。

-   **环回地址 + SSH/Tailscale Serve** 是最安全的默认设置（无公共暴露）。
-   明文 `ws://` 默认仅限环回地址访问。对于受信任的私有网络，可在客户端进程上设置 `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` 作为应急方案。
-   **非环回地址绑定**（`lan`/`tailnet`/`custom`，或当环回地址不可用时使用 `auto`）必须使用身份验证令牌/密码。
-   `gateway.remote.token` / `.password` 是客户端凭据来源。它们本身**不**配置服务器身份验证。
-   当 `gateway.auth.*` 未设置时，本地调用路径可以使用 `gateway.remote.*` 作为回退。
-   `gateway.remote.tlsFingerprint` 在使用 `wss://` 时固定远程 TLS 证书。
-   **Tailscale Serve** 可以通过身份标头对控制界面/WebSocket 流量进行身份验证，当 `gateway.auth.allowTailscale: true` 时；HTTP API 端点仍然需要令牌/密码身份验证。这种无令牌流程假设网关主机是受信任的。如果您希望在所有地方都使用令牌/密码，请将其设置为 `false`。
-   将浏览器控制视为操作员访问：仅限 tailnet + 有意的节点配对。

深入探讨：[安全](./security.md)。

[Bonjour 发现](./bonjour.md)[远程网关设置](./remote-gateway-readme.md)