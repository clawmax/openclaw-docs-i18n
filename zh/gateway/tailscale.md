

  远程访问

  
# Tailscale

OpenClaw 可以自动为网关仪表板和 WebSocket 端口配置 Tailscale **Serve**（tailnet）或 **Funnel**（公开）。这使得网关绑定到环回地址，而由 Tailscale 提供 HTTPS、路由和（对于 Serve）身份验证头。

## 模式

-   `serve`: 仅限 tailnet 的 Serve，通过 `tailscale serve` 实现。网关保持在 `127.0.0.1`。
-   `funnel`: 通过 `tailscale funnel` 提供公开的 HTTPS。OpenClaw 需要一个共享密码。
-   `off`: 默认（无 Tailscale 自动化）。

## 身份验证

设置 `gateway.auth.mode` 来控制握手过程：

-   `token`（当设置了 `OPENCLAW_GATEWAY_TOKEN` 时的默认值）
-   `password`（通过 `OPENCLAW_GATEWAY_PASSWORD` 或配置的共享密钥）

当 `tailscale.mode = "serve"` 且 `gateway.auth.allowTailscale` 为 `true` 时，控制 UI/WebSocket 身份验证可以使用 Tailscale 身份头（`tailscale-user-login`），而无需提供令牌/密码。OpenClaw 通过本地 Tailscale 守护进程（`tailscale whois`）解析 `x-forwarded-for` 地址并与头信息匹配来验证身份，然后才接受它。OpenClaw 仅当请求来自环回地址并带有 Tailscale 的 `x-forwarded-for`、`x-forwarded-proto` 和 `x-forwarded-host` 头时，才将其视为 Serve 请求。HTTP API 端点（例如 `/v1/*`、`/tools/invoke` 和 `/api/channels/*`）仍然需要令牌/密码身份验证。这种无令牌流程假设网关主机是可信的。如果同一主机上可能运行不受信任的本地代码，请禁用 `gateway.auth.allowTailscale` 并改用令牌/密码身份验证。若要要求显式凭据，请设置 `gateway.auth.allowTailscale: false` 或强制 `gateway.auth.mode: "password"`。

## 配置示例

### 仅限 Tailnet (Serve)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

打开：`https:///`（或您配置的 `gateway.controlUi.basePath`）

### 仅限 Tailnet (绑定到 Tailnet IP)

当您希望网关直接在 Tailnet IP 上监听（不使用 Serve/Funnel）时使用此配置。

```json
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

从另一个 Tailnet 设备连接：

-   控制 UI：`http://<tailscale-ip>:18789/`
-   WebSocket：`ws://<tailscale-ip>:18789`

注意：环回地址（`http://127.0.0.1:18789`）在此模式下将**无法**工作。

### 公共互联网 (Funnel + 共享密码)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

建议使用 `OPENCLAW_GATEWAY_PASSWORD` 环境变量，而不是将密码提交到磁盘。

## CLI 示例

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## 注意事项

-   Tailscale Serve/Funnel 要求安装并登录 `tailscale` CLI。
-   `tailscale.mode: "funnel"` 除非身份验证模式为 `password`，否则拒绝启动，以避免公开暴露。
-   如果您希望 OpenClaw 在关闭时撤销 `tailscale serve` 或 `tailscale funnel` 配置，请设置 `gateway.tailscale.resetOnExit`。
-   `gateway.bind: "tailnet"` 是直接的 Tailnet 绑定（无 HTTPS，无 Serve/Funnel）。
-   `gateway.bind: "auto"` 优先使用环回地址；如果您想要仅限 Tailnet，请使用 `tailnet`。
-   Serve/Funnel 仅暴露**网关控制 UI + WS**。节点通过同一个网关 WS 端点连接，因此 Serve 可以用于节点访问。

## 浏览器控制（远程网关 + 本地浏览器）

如果您在一台机器上运行网关，但希望在另一台机器上驱动浏览器，请在浏览器机器上运行一个**节点主机**，并确保两者在同一个 tailnet 中。网关将把浏览器操作代理到节点；不需要单独的控制服务器或 Serve URL。避免将 Funnel 用于浏览器控制；将节点配对视为操作员访问。

## Tailscale 先决条件 + 限制

-   Serve 要求为您的 tailnet 启用 HTTPS；如果未启用，CLI 会提示。
-   Serve 会注入 Tailscale 身份头；Funnel 不会。
-   Funnel 需要 Tailscale v1.38.3+、MagicDNS、启用 HTTPS 以及一个 funnel 节点属性。
-   Funnel 仅支持通过 TLS 的端口 `443`、`8443` 和 `10000`。
-   在 macOS 上使用 Funnel 需要开源的 Tailscale 应用变体。

## 了解更多

-   Tailscale Serve 概述：[https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
-   `tailscale serve` 命令：[https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
-   Tailscale Funnel 概述：[https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
-   `tailscale funnel` 命令：[https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)

[远程网关设置](./remote-gateway-readme.md)[形式化验证（安全模型）](../security/formal-verification.md)