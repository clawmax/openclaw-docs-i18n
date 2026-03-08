

  配置与运维

  
# 可信代理身份验证

> ⚠️ **安全敏感功能。** 此模式将身份验证完全委托给您的反向代理。配置错误可能导致您的网关遭受未授权访问。启用前请仔细阅读本页内容。

## 使用场景

在以下情况下使用 `trusted-proxy` 身份验证模式：

-   您在**身份感知代理**（Pomerium、Caddy + OAuth、nginx + oauth2-proxy、Traefik + forward auth）后面运行 OpenClaw
-   您的代理处理所有身份验证并通过请求头传递用户身份
-   您处于 Kubernetes 或容器环境中，代理是访问网关的唯一路径
-   您遇到 WebSocket `1008 unauthorized` 错误，因为浏览器无法在 WS 负载中传递令牌

## 不应使用的情况

-   如果您的代理不对用户进行身份验证（仅作为 TLS 终结器或负载均衡器）
-   如果存在任何绕过代理访问网关的路径（防火墙漏洞、内部网络访问）
-   如果您不确定您的代理是否正确剥离/覆盖了转发的请求头
-   如果您只需要个人单用户访问（考虑使用 Tailscale Serve + loopback 以获得更简单的设置）

## 工作原理

1.  您的反向代理对用户进行身份验证（OAuth、OIDC、SAML 等）
2.  代理添加一个包含已认证用户身份的请求头（例如，`x-forwarded-user: nick@example.com`）
3.  OpenClaw 检查请求是否来自**可信代理 IP**（在 `gateway.trustedProxies` 中配置）
4.  OpenClaw 从配置的请求头中提取用户身份
5.  如果所有检查通过，则请求被授权

## 控制界面配对行为

当 `gateway.auth.mode = "trusted-proxy"` 激活且请求通过可信代理检查时，控制界面 WebSocket 会话可以在没有设备配对身份的情况下连接。这意味着：

-   在此模式下，配对不再是控制界面访问的主要关卡。
-   您的反向代理身份验证策略和 `allowUsers` 成为有效的访问控制手段。
-   请确保网关入口仅锁定到可信代理 IP（`gateway.trustedProxies` + 防火墙）。

## 配置

```json
{
  gateway: {
    // 对于同主机代理设置使用 loopback；对于远程代理主机使用 lan/custom
    bind: "loopback",

    // 关键：仅在此处添加您的代理 IP
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // 包含已认证用户身份的请求头（必需）
        userHeader: "x-forwarded-user",

        // 可选：必须存在的请求头（代理验证）
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // 可选：限制为特定用户（空 = 允许所有）
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

如果 `gateway.bind` 是 `loopback`，请在 `gateway.trustedProxies` 中包含一个环回代理地址（`127.0.0.1`、`::1` 或等效的环回 CIDR）。

### 配置参考

| 字段 | 必需 | 描述 |
| --- | --- | --- |
| `gateway.trustedProxies` | 是 | 可信代理 IP 地址数组。来自其他 IP 的请求将被拒绝。 |
| `gateway.auth.mode` | 是 | 必须为 `"trusted-proxy"` |
| `gateway.auth.trustedProxy.userHeader` | 是 | 包含已认证用户身份的请求头名称 |
| `gateway.auth.trustedProxy.requiredHeaders` | 否 | 请求必须包含的额外请求头，才能被信任 |
| `gateway.auth.trustedProxy.allowUsers` | 否 | 用户身份白名单。为空表示允许所有已认证用户。 |

## TLS 终结与 HSTS

使用一个 TLS 终结点并在那里应用 HSTS。

### 推荐模式：代理 TLS 终结

当您的反向代理为 `https://control.example.com` 处理 HTTPS 时，在该域的代理处设置 `Strict-Transport-Security`。

-   适合面向互联网的部署。
-   将证书和 HTTP 强化策略集中在一处。
-   OpenClaw 可以保持在代理后面的环回 HTTP 上。

示例请求头值：

```yaml
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### 网关 TLS 终结

如果 OpenClaw 自身直接提供 HTTPS（没有 TLS 终结代理），请设置：

```json
{
  gateway: {
    tls: { enabled: true },
    http: {
      securityHeaders: {
        strictTransportSecurity: "max-age=31536000; includeSubDomains",
      },
    },
  },
}
```

`strictTransportSecurity` 接受一个字符串请求头值，或 `false` 来显式禁用。

### 部署指南

-   首先设置一个较短的 max-age（例如 `max-age=300`）以验证流量。
-   仅在信心充足后，再增加到长期值（例如 `max-age=31536000`）。
-   仅当每个子域都准备好支持 HTTPS 时，才添加 `includeSubDomains`。
-   仅当您有意为您的完整域名集满足预加载要求时，才使用预加载。
-   仅限环回的本地开发无法从 HSTS 中受益。

## 代理设置示例

### Pomerium

Pomerium 在 `x-pomerium-claim-email`（或其他声明请求头）中传递身份，并在 `x-pomerium-jwt-assertion` 中传递 JWT。

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium 的 IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

Pomerium 配置片段：

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy with OAuth

带有 `caddy-security` 插件的 Caddy 可以对用户进行身份验证并传递身份请求头。

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy 的 IP（如果在同一主机上）
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile 片段：

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

oauth2-proxy 对用户进行身份验证，并在 `x-auth-request-email` 中传递身份。

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

nginx 配置片段：

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik with Forward Auth

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // Traefik 容器 IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## 安全检查清单

启用可信代理身份验证前，请验证：

-   [ ]  **代理是唯一路径**：网关端口已通过防火墙屏蔽，仅允许您的代理访问
-   [ ]  **trustedProxies 最小化**：仅包含您的实际代理 IP，而非整个子网
-   [ ]  **代理剥离请求头**：您的代理覆盖（而非追加）来自客户端的 `x-forwarded-*` 请求头
-   [ ]  **TLS 终结**：您的代理处理 TLS；用户通过 HTTPS 连接
-   [ ]  **已设置 allowUsers**（推荐）：限制为已知用户，而不是允许任何已认证用户

## 安全审计

`openclaw security audit` 会将可信代理身份验证标记为**严重**级别的发现项。这是有意为之——提醒您正在将安全性委托给您的代理设置。审计检查以下内容：

-   缺少 `trustedProxies` 配置
-   缺少 `userHeader` 配置
-   `allowUsers` 为空（允许任何已认证用户）

## 故障排除

### ”trusted_proxy_untrusted_source”

请求并非来自 `gateway.trustedProxies` 中的 IP。检查：

-   代理 IP 是否正确？（Docker 容器 IP 可能会变化）
-   您的代理前面是否有负载均衡器？
-   使用 `docker inspect` 或 `kubectl get pods -o wide` 查找实际 IP

### ”trusted_proxy_user_missing”

用户请求头为空或缺失。检查：

-   您的代理是否配置为传递身份请求头？
-   请求头名称是否正确？（不区分大小写，但拼写很重要）
-   用户是否确实在代理处通过了身份验证？

### “trustedproxy_missing_header*”

缺少必需的请求头。检查：

-   您的代理配置中是否包含这些特定请求头
-   请求头是否在链路的某个环节被剥离

### ”trusted_proxy_user_not_allowed”

用户已通过身份验证，但不在 `allowUsers` 中。请将其添加至列表或移除白名单。

### WebSocket 仍然失败

确保您的代理：

-   支持 WebSocket 升级（`Upgrade: websocket`、`Connection: upgrade`）
-   在 WebSocket 升级请求（不仅仅是 HTTP）上传递身份请求头
-   没有为 WebSocket 连接设置单独的身份验证路径

## 从令牌身份验证迁移

如果您正从令牌身份验证迁移到可信代理：

1.  配置您的代理以对用户进行身份验证并传递请求头
2.  独立测试代理设置（使用 curl 带请求头）
3.  使用可信代理身份验证更新 OpenClaw 配置
4.  重启网关
5.  测试来自控制界面的 WebSocket 连接
6.  运行 `openclaw security audit` 并查看发现项

## 相关链接

-   [安全性](./security.md) — 完整安全指南
-   [配置](./configuration.md) — 配置参考
-   [远程访问](./remote.md) — 其他远程访问模式
-   [Tailscale](./tailscale.md) — 仅限 tailnet 访问的更简单替代方案

[Secrets Apply Plan Contract](./secrets-plan-contract.md)[健康检查](./health.md)