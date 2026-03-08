

  网络与发现

  
# 网络模型

大多数操作都流经网关 (`openclaw gateway`)，这是一个单一的长时运行进程，它拥有频道连接和 WebSocket 控制平面。

## 核心规则

-   建议每个主机运行一个网关。它是唯一被允许拥有 WhatsApp Web 会话的进程。对于救援机器人或严格隔离的场景，可以使用隔离的配置文件和端口运行多个网关。参见[多网关](./multiple-gateways.md)。
-   环回优先：网关 WebSocket 默认绑定到 `ws://127.0.0.1:18789`。向导默认会生成一个网关令牌，即使对于环回连接也是如此。对于 tailnet 访问，需要运行 `openclaw gateway --bind tailnet --token ...`，因为非环回绑定需要令牌。
-   节点根据需要通过 LAN、tailnet 或 SSH 连接到网关 WebSocket。传统的 TCP 桥接方式已弃用。
-   画布主机由网关 HTTP 服务器在**与网关相同的端口**（默认 `18789`）上提供：
    -   `/__openclaw__/canvas/`
    -   `/__openclaw__/a2ui/` 当配置了 `gateway.auth` 且网关绑定到环回地址之外时，这些路由受网关身份验证保护。节点客户端使用与其活动 WebSocket 会话绑定的节点范围能力 URL。参见[网关配置](./configuration.md) (`canvasHost`, `gateway`)。
-   远程使用通常通过 SSH 隧道或 tailnet VPN。参见[远程访问](./remote.md) 和[发现](./discovery.md)。

[本地模型](./local-models.md)[网关拥有的配对](./pairing.md)