

  配置与运维

  
# 可信代理认证

> ⚠️ **安全敏感功能。** 此模式将认证完全委托给你的反向代理。配置不当可能导致网关遭受未授权访问。启用前请务必仔细阅读本文档。

## 适用场景

在以下情况下使用 `trusted-proxy` 认证模式：

-   你在**身份感知代理**（Pomerium、Caddy + OAuth、nginx + oauth2-proxy、Traefik + forward auth）后面运行 OpenClaw
-   你的代理负责处理所有认证，并通过请求头传递用户身份
-   你处于 Kubernetes 或容器环境中，代理是访问网关的唯一入口
-   你遇到 WebSocket `1008 unauthorized` 错误，因为浏览器无法在 WebSocket 消息体中传递令牌

## 不适用场景

-   如果你的代理不进行用户认证（仅作为 TLS 终止器或负载均衡器）
-   如果存在任何绕过代理直接访问网关的路径（防火墙漏洞、内网直连）
-   如果你不确定代理是否正确清除/覆盖了转发的请求头
-   如果你只需要个人单用户访问（考虑使用 Tailscale Serve + 本地回环更简单）

## 工作原理

1.  反向代理对用户进行认证（OAuth、OIDC、SAML 等）
2.  代理添加包含已认证用户身份的请求头（例如 `x-forwarded-user: nick@example.com`）
3.  OpenClaw 检查请求是否来自**可信代理 IP**（在 `gateway.trustedProxies` 中配置）
4.  OpenClaw 从配置的请求头中提取用户身份
5.  所有检查通过后，请求被授权

## 控制界面配对行为

当 `gateway.auth.mode = "trusted-proxy"` 生效且请求通过可信代理检查时，控制界面的 WebSocket 会话可以在没有设备配对身份的情况下连接。这意味着：

-   在此模式下，配对不再是控制界面访问的主要关卡。
-   你的反向代理认证策略和 `allowUsers` 成为实际的访问控制手段。
-   请确保网关入口仅对可信代理 IP 开放（`gateway.trustedProxies` + 防火墙）。

## 配置

```json
{
  gateway: {
    // 同主机代理配置使用 loopback；远程代理主机使用 lan/custom
    bind: "loopback",

    // 关键：仅在此处添加你的代理 IP
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // 包含已认证用户身份的请求头（必填）
        userHeader: "x-forwarded-user",

        // 可选：必须存在的请求头（用于验证代理）
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // 可选：限制为特定用户（留空表示允许所有）
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

如果 `gateway.bind` 为 `loopback`，需要在 `gateway.trustedProxies` 中包含一个本地回环代理地址（`127.0.0.1`、`::1` 或等效的回环 CIDR）。

### 配置字段参考

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `gateway.trustedProxies` | 是 | 可信代理 IP 地址数组。来自其他 IP 的请求会被拒绝。 |
| `gateway.auth.mode` | 是 | 必须为 `"trusted-proxy"` |
| `gateway.auth.trustedProxy.userHeader` | 是 | 包含已认证用户身份的请求头名称 |
| `gateway.auth.trustedProxy.requiredHeaders` | 否 | 请求必须携带的额外请求头，才能被信任 |
| `gateway.auth.trustedProxy.allowUsers` | 否 | 用户身份白名单。留空表示允许所有已认证用户。 |

## TLS 终止与 HSTS

使用单一 TLS 终止点并在该处应用 HSTS 策略。

### 推荐模式：代理 TLS 终止

当反向代理为 `https://control.example.com` 处理 HTTPS 时，在代理处为该域名设置 `Strict-Transport-Security`。

-   适合面向公网的部署。
-   证书和 HTTP 安全策略集中管理。
-   OpenClaw 可以保持在代理后面的本地 HTTP 上运行。

示例响应头：

```yaml
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### 网关 TLS 终止

如果 OpenClaw 自身直接提供 HTTPS 服务（没有 TLS 终止代理），请设置：

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

`strictTransportSecurity` 接受字符串形式的响应头值，或 `false` 显式禁用。

### 部署建议

-   先使用较短的 max-age（如 `max-age=300`）验证流量正常。
-   确认无误后再增加到长期值（如 `max-age=31536000`）。
-   仅当所有子域名都支持 HTTPS 时才添加 `includeSubDomains`。
-   仅当你有意满足预加载要求时才使用 preload。
-   仅本地回环的开发环境无法从 HSTS 中获益。

## 代理配置示例

### Pomerium

Pomerium 通过 `x-pomerium-claim-email`（或其他声明请求头）传递身份，并在 `x-pomerium-jwt-assertion` 中传递 JWT。

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

### Caddy + OAuth

安装 `caddy-security` 插件的 Caddy 可以认证用户并传递身份请求头。

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy 的 IP（如果在同一主机）
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile 配置片段：

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

oauth2-proxy 认证用户后通过 `x-auth-request-email` 传递身份。

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy 的 IP
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

### Traefik + Forward Auth

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

启用可信代理认证前，请逐项确认：

-   [ ]  **代理是唯一入口**：网关端口已设置防火墙，仅允许代理访问
-   [ ]  **trustedProxies 精简**：仅包含实际的代理 IP，不要填整个子网
-   [ ]  **代理覆盖请求头**：你的代理会覆盖（而非追加）来自客户端的 `x-forwarded-*` 请求头
-   [ ]  **TLS 终止**：代理处理 TLS；用户通过 HTTPS 连接
-   [ ]  **设置 allowUsers**（推荐）：限制为已知用户，而非允许所有已认证用户

## 安全审计

`openclaw security audit` 会将可信代理认证标记为**严重**级别的发现。这是有意为之——提醒你正在将安全委托给代理配置。审计会检查：

-   缺少 `trustedProxies` 配置
-   缺少 `userHeader` 配置
-   `allowUsers` 为空（允许任何已认证用户）

## 故障排除

### "trusted_proxy_untrusted_source"

请求并非来自 `gateway.trustedProxies` 中的 IP。检查：

-   代理 IP 是否正确？（Docker 容器 IP 可能变化）
-   代理前面是否有负载均衡器？
-   使用 `docker inspect` 或 `kubectl get pods -o wide` 查看实际 IP

### "trusted_proxy_user_missing"

用户请求头为空或缺失。检查：

-   代理是否配置了传递身份请求头？
-   请求头名称是否正确？（不区分大小写，但拼写必须正确）
-   用户在代理处是否确实已通过认证？

### "trustedproxy_missing_header*"

缺少必需的请求头。检查：

-   代理配置中是否包含这些特定请求头
-   请求头是否在链路某处被清除

### "trusted_proxy_user_not_allowed"

用户已认证但不在 `allowUsers` 列表中。请添加该用户或移除白名单限制。

### WebSocket 仍然失败

确保你的代理：

-   支持 WebSocket 升级（`Upgrade: websocket`、`Connection: upgrade`）
-   在 WebSocket 升级请求上也传递身份请求头（不仅是普通 HTTP）
-   没有为 WebSocket 连接设置单独的认证路径

## 从令牌认证迁移

如果从令牌认证切换到可信代理认证：

1.  配置代理认证用户并传递请求头
2.  独立测试代理配置（用 curl 携带请求头测试）
3.  更新 OpenClaw 配置为可信代理认证
4.  重启网关
5.  从控制界面测试 WebSocket 连接
6.  运行 `openclaw security audit` 查看审计结果

## 相关文档

-   [安全](./security.md) — 完整安全指南
-   [配置](./configuration.md) — 配置参考
-   [远程访问](./remote.md) — 其他远程访问模式
-   [Tailscale](./tailscale.md) — 仅限 tailnet 访问的更简单替代方案

[Secrets Apply Plan Contract](./secrets-plan-contract.md)[健康检查](./health.md)