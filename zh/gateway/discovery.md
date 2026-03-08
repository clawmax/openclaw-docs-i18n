

  网络与发现

  
# 发现与传输

OpenClaw 有两个表面上看起来相似但本质不同的问题：

1.  **操作员远程控制**：macOS 菜单栏应用控制运行在其他地方的网关。
2.  **节点配对**：iOS/Android（以及未来的节点）发现网关并进行安全配对。

设计目标是将所有网络发现/通告功能保留在**节点网关** (`openclaw gateway`) 中，并使客户端（mac 应用、iOS）作为消费者。

## 术语

-   **网关**：一个长期运行的网关进程，拥有状态（会话、配对、节点注册表）并运行通道。大多数设置在每个主机上使用一个；也可以进行隔离的多网关设置。
-   **网关 WS（控制平面）**：默认在 `127.0.0.1:18789` 上的 WebSocket 端点；可以通过 `gateway.bind` 绑定到 LAN/tailnet。
-   **直接 WS 传输**：面向 LAN/tailnet 的网关 WS 端点（无需 SSH）。
-   **SSH 传输（回退）**：通过 SSH 转发 `127.0.0.1:18789` 进行远程控制。
-   **传统 TCP 桥接（已弃用/移除）**：较旧的节点传输（参见[桥接协议](./bridge-protocol.md)）；不再为发现而通告。

协议详情：

-   [网关协议](./protocol.md)
-   [桥接协议（旧版）](./bridge-protocol.md)

## 为何同时保留“直接”和 SSH 传输

-   **直接 WS** 在同一网络和 tailnet 内提供最佳用户体验：
    -   通过 Bonjour 在 LAN 上自动发现
    -   配对令牌 + ACL 由网关管理
    -   无需 shell 访问；协议接口可以保持精简且可审计
-   **SSH** 仍然是通用的回退方案：
    -   在任何有 SSH 访问权限的地方都有效（即使跨越无关网络）
    -   能应对组播/mDNS 问题
    -   除了 SSH 外，不需要开放任何新的入站端口

## 发现输入（客户端如何获知网关位置）

### 1) Bonjour / mDNS（仅限 LAN）

Bonjour 是尽力而为的，不能跨网络工作。它仅用于“同一 LAN”的便利性。目标方向：

-   **网关** 通过 Bonjour 通告其 WS 端点。
-   客户端浏览并显示“选择网关”列表，然后存储选定的端点。

故障排除和信标详情：[Bonjour](./bonjour.md)。

#### 服务信标详情

-   服务类型：
    -   `_openclaw-gw._tcp`（网关传输信标）
-   TXT 键（非机密）：
    -   `role=gateway`
    -   `lanHost=.local`
    -   `sshPort=22`（或通告的任何端口）
    -   `gatewayPort=18789`（网关 WS + HTTP）
    -   `gatewayTls=1`（仅在启用 TLS 时）
    -   `gatewayTlsSha256=`（仅在启用 TLS 且指纹可用时）
    -   `canvasPort=`（画布主机端口；当前在启用画布主机时与 `gatewayPort` 相同）
    -   `cliPath=`（可选；可运行的 `openclaw` 入口点或二进制文件的绝对路径）
    -   `tailnetDns=`（可选提示；当 Tailscale 可用时自动检测）

安全注意事项：

-   Bonjour/mDNS TXT 记录是**未经认证的**。客户端必须仅将 TXT 值视为用户体验提示。
-   路由（主机/端口）应优先使用**已解析的服务端点**（SRV + A/AAAA），而不是 TXT 提供的 `lanHost`、`tailnetDns` 或 `gatewayPort`。
-   TLS 固定绝不允许通告的 `gatewayTlsSha256` 覆盖先前存储的固定值。
-   iOS/Android 节点应将基于发现的直接连接视为**仅限 TLS**，并在存储首次固定之前要求明确的“信任此指纹”确认（带外验证）。

禁用/覆盖：

-   `OPENCLAW_DISABLE_BONJOUR=1` 禁用通告。
-   `~/.openclaw/openclaw.json` 中的 `gateway.bind` 控制网关绑定模式。
-   `OPENCLAW_SSH_PORT` 覆盖 TXT 中通告的 SSH 端口（默认为 22）。
-   `OPENCLAW_TAILNET_DNS` 发布 `tailnetDns` 提示（MagicDNS）。
-   `OPENCLAW_CLI_PATH` 覆盖通告的 CLI 路径。

### 2) Tailnet（跨网络）

对于伦敦/维也纳风格的设置，Bonjour 将不起作用。推荐的“直接”目标是：

-   Tailscale MagicDNS 名称（首选）或稳定的 tailnet IP。

如果网关能检测到它在 Tailscale 下运行，它会发布 `tailnetDns` 作为给客户端的可选提示（包括广域网信标）。

### 3) 手动 / SSH 目标

当没有直接路由（或直接连接被禁用）时，客户端始终可以通过 SSH 转发环回网关端口来连接。参见[远程访问](./remote.md)。

## 传输选择（客户端策略）

推荐的客户端行为：

1.  如果已配置配对的直接端点且可访问，则使用它。
2.  否则，如果 Bonjour 在 LAN 上发现网关，则提供一个一键“使用此网关”的选择，并将其保存为直接端点。
3.  否则，如果配置了 tailnet DNS/IP，则尝试直接连接。
4.  否则，回退到 SSH。

## 配对 + 认证（直接传输）

网关是节点/客户端准入的真相来源。

-   配对请求在网关中创建/批准/拒绝（参见[网关配对](./pairing.md)）。
-   网关强制执行：
    -   认证（令牌 / 密钥对）
    -   作用域/ACL（网关并非每个方法的原始代理）
    -   速率限制

## 各组件职责

-   **网关**：通告发现信标，拥有配对决策权，并托管 WS 端点。
-   **macOS 应用**：帮助您选择网关，显示配对提示，并仅将 SSH 用作回退方案。
-   **iOS/Android 节点**：将浏览 Bonjour 作为一种便利方式，并连接到配对的网关 WS。

[网关拥有的配对](./pairing.md)[Bonjour 发现](./bonjour.md)