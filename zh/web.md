

  Web 界面

  
# Web

网关通过与其 WebSocket 相同的端口提供一个小型的**浏览器控制 UI**（Vite + Lit）：

-   默认地址：`http://<主机>:18789/`
-   可选前缀：设置 `gateway.controlUi.basePath`（例如 `/openclaw`）

功能详情请见[控制 UI](./web/control-ui.md)。本页重点介绍绑定模式、安全性和面向 Web 的界面。

## Webhooks

当 `hooks.enabled=true` 时，网关还会在同一 HTTP 服务器上暴露一个小的 Webhook 端点。有关认证和有效载荷，请参见[网关配置](./gateway/configuration.md) → `hooks`。

## 配置（默认启用）

当资源文件存在时（`dist/control-ui`），控制 UI **默认启用**。您可以通过配置控制它：

```json
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath 可选
  },
}
```

## Tailscale 访问

### 集成 Serve（推荐）

将网关保持在环回地址上，让 Tailscale Serve 代理它：

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

然后启动网关：

```bash
openclaw gateway
```

访问：

-   `https:///`（或您配置的 `gateway.controlUi.basePath`）

### Tailnet 绑定 + 令牌

```json
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

然后启动网关（非环回绑定需要令牌）：

```bash
openclaw gateway
```

访问：

-   `http://<tailscale-ip>:18789/`（或您配置的 `gateway.controlUi.basePath`）

### 公共互联网（Funnel）

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // 或 OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## 安全注意事项

-   默认情况下需要网关认证（令牌/密码或 Tailscale 身份头）。
-   非环回绑定仍然**需要**共享令牌/密码（`gateway.auth` 或环境变量）。
-   向导默认会生成一个网关令牌（即使在环回地址上）。
-   UI 会发送 `connect.params.auth.token` 或 `connect.params.auth.password`。
-   对于非环回的控制 UI 部署，请明确设置 `gateway.controlUi.allowedOrigins`（完整的来源）。如果没有设置，默认情况下网关启动会被拒绝。
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` 启用 Host 头来源回退模式，但这是一种危险的安全降级。
-   使用 Serve 时，当 `gateway.auth.allowTailscale` 为 `true` 时，Tailscale 身份头可以满足控制 UI/WebSocket 的认证（无需令牌/密码）。HTTP API 端点仍然需要令牌/密码。设置 `gateway.auth.allowTailscale: false` 以要求显式凭据。请参阅 [Tailscale](./gateway/tailscale.md) 和 [安全](./gateway/security.md)。此无令牌流程假设网关主机是受信任的。
-   `gateway.tailscale.mode: "funnel"` 要求 `gateway.auth.mode: "password"`（共享密码）。

## 构建 UI

网关从 `dist/control-ui` 提供静态文件。使用以下命令构建它们：

```bash
pnpm ui:build # 首次运行时自动安装 UI 依赖
```

[贡献威胁模型](./security/CONTRIBUTING-THREAT-MODEL.md)[控制 UI](./web/control-ui.md)