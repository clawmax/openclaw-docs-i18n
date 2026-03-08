

  网络与发现

  
# Bonjour 发现

OpenClaw 使用 Bonjour (mDNS / DNS‑SD) 作为一种**仅限于局域网的便利方式**来发现活跃的网关（WebSocket 端点）。它是尽力而为的，并**不能**替代基于 SSH 或 Tailnet 的连接。

## 通过 Tailscale 的广域网 Bonjour（单播 DNS‑SD）

如果节点和网关位于不同的网络，组播 mDNS 将无法跨越网络边界。您可以通过切换到通过 Tailscale 的**单播 DNS‑SD**（“广域网 Bonjour”）来保持相同的发现用户体验。高级步骤：

1.  在网关主机上运行一个 DNS 服务器（可通过 Tailnet 访问）。
2.  在专用区域（例如：`openclaw.internal.`）下发布 `_openclaw-gw._tcp` 的 DNS‑SD 记录。
3.  配置 Tailscale **拆分 DNS**，使您选择的域名通过该 DNS 服务器为客户端（包括 iOS）解析。

OpenClaw 支持任何发现域名；`openclaw.internal.` 只是一个示例。iOS/Android 节点会同时浏览 `local.` 和您配置的广域网域名。

### 网关配置（推荐）

```json
{
  gateway: { bind: "tailnet" }, // 仅限 tailnet（推荐）
  discovery: { wideArea: { enabled: true } }, // 启用广域网 DNS-SD 发布
}
```

### 一次性 DNS 服务器设置（网关主机）

```bash
openclaw dns setup --apply
```

这将安装 CoreDNS 并将其配置为：

-   仅在网关的 Tailscale 接口上监听端口 53
-   从 `~/.openclaw/dns/.db` 为您选择的域名（例如：`openclaw.internal.`）提供服务

从连接到 tailnet 的机器进行验证：

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### Tailscale DNS 设置

在 Tailscale 管理控制台中：

-   添加一个指向网关 tailnet IP 的域名服务器（UDP/TCP 53）。
-   添加拆分 DNS，使您的发现域名使用该域名服务器。

一旦客户端接受 tailnet DNS，iOS 节点就可以在您的发现域名中浏览 `_openclaw-gw._tcp`，而无需组播。

### 网关监听器安全性（推荐）

网关 WS 端口（默认 `18789`）默认绑定到环回地址。对于局域网/tailnet 访问，请显式绑定并保持身份验证启用。对于仅限 tailnet 的设置：

-   在 `~/.openclaw/openclaw.json` 中设置 `gateway.bind: "tailnet"`。
-   重启网关（或重启 macOS 菜单栏应用）。

## 广告内容

只有网关会广告 `_openclaw-gw._tcp`。

## 服务类型

-   `_openclaw-gw._tcp` — 网关传输信标（由 macOS/iOS/Android 节点使用）。

## TXT 键（非机密提示）

网关广告小的非机密提示，以使 UI 流程更方便：

-   `role=gateway`
-   `displayName=<友好名称>`
-   `lanHost=<主机名>.local`
-   `gatewayPort=<端口>`（网关 WS + HTTP）
-   `gatewayTls=1`（仅在启用 TLS 时）
-   `gatewayTlsSha256=`（仅在启用 TLS 且指纹可用时）
-   `canvasPort=<端口>`（仅在启用画布主机时；目前与 `gatewayPort` 相同）
-   `sshPort=<端口>`（未覆盖时默认为 22）
-   `transport=gateway`
-   `cliPath=`（可选；可运行的 `openclaw` 入口点的绝对路径）
-   `tailnetDns=`（Tailnet 可用时的可选提示）

安全说明：

-   Bonjour/mDNS TXT 记录是**未经身份验证的**。客户端不得将 TXT 视为权威路由信息。
-   客户端应使用解析出的服务端点（SRV + A/AAAA）进行路由。将 `lanHost`、`tailnetDns`、`gatewayPort` 和 `gatewayTlsSha256` 仅视为提示。
-   TLS 固定绝不允许广告的 `gatewayTlsSha256` 覆盖先前存储的固定值。
-   iOS/Android 节点应将基于发现的直接连接视为**仅限 TLS**，并在信任首次指纹之前需要明确的用户确认。

## 在 macOS 上调试

有用的内置工具：

-   浏览实例：
    
    复制
    
    ```bash
    dns-sd -B _openclaw-gw._tcp local.
    ```
    
-   解析一个实例（替换 `<实例>`）：
    
    复制
    
    ```bash
    dns-sd -L "<instance>" _openclaw-gw._tcp local.
    ```
    

如果浏览有效但解析失败，通常是遇到了 LAN 策略或 mDNS 解析器问题。

## 在网关日志中调试

网关会写入一个滚动日志文件（启动时打印为 `gateway log file: ...`）。查找 `bonjour:` 行，特别是：

-   `bonjour: advertise failed ...`
-   `bonjour: ... name conflict resolved` / `hostname conflict resolved`
-   `bonjour: watchdog detected non-announced service ...`

## 在 iOS 节点上调试

iOS 节点使用 `NWBrowser` 来发现 `_openclaw-gw._tcp`。要捕获日志：

-   设置 → 网关 → 高级 → **发现调试日志**
-   设置 → 网关 → 高级 → **发现日志** → 重现 → **复制**

日志包含浏览器状态转换和结果集更改。

## 常见故障模式

-   **Bonjour 无法跨网络**：使用 Tailnet 或 SSH。
-   **组播被阻止**：某些 Wi‑Fi 网络禁用了 mDNS。
-   **睡眠 / 接口变动**：macOS 可能会暂时丢弃 mDNS 结果；请重试。
-   **浏览有效但解析失败**：保持机器名称简单（避免表情符号或标点符号），然后重启网关。服务实例名称派生自主机名，因此过于复杂的名称可能会使某些解析器混淆。

## 转义的实例名称（\\032）

Bonjour/DNS‑SD 通常将服务实例名称中的字节转义为十进制 `\DDD` 序列（例如，空格变为 `\032`）。

-   这在协议级别是正常的。
-   UI 应解码以供显示（iOS 使用 `BonjourEscapes.decode`）。

## 禁用 / 配置

-   `OPENCLAW_DISABLE_BONJOUR=1` 禁用广告（旧版：`OPENCLAW_DISABLE_BONJOUR`）。
-   `~/.openclaw/openclaw.json` 中的 `gateway.bind` 控制网关绑定模式。
-   `OPENCLAW_SSH_PORT` 覆盖 TXT 中广告的 SSH 端口（旧版：`OPENCLAW_SSH_PORT`）。
-   `OPENCLAW_TAILNET_DNS` 在 TXT 中发布 MagicDNS 提示（旧版：`OPENCLAW_TAILNET_DNS`）。
-   `OPENCLAW_CLI_PATH` 覆盖广告的 CLI 路径（旧版：`OPENCLAW_CLI_PATH`）。

## 相关文档

-   发现策略和传输选择：[发现](./discovery.md)
-   节点配对 + 批准：[网关配对](./pairing.md)

[发现与传输](./discovery.md)[远程访问](./remote.md)