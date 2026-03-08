

  CLI 命令

  
# qr

从您当前的网关配置生成 iOS 配对二维码和设置码。

## 用法

```bash
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws --token '<token>'
```

## 选项

-   `--remote`: 使用配置中的 `gateway.remote.url` 加上远程令牌/密码
-   `--url `: 覆盖 payload 中使用的网关 URL
-   `--public-url `: 覆盖 payload 中使用的公共 URL
-   `--token `: 覆盖 payload 中使用的网关令牌
-   `--password `: 覆盖 payload 中使用的网关密码
-   `--setup-code-only`: 仅打印设置码
-   `--no-ascii`: 跳过 ASCII 二维码渲染
-   `--json`: 输出 JSON (`setupCode`, `gatewayUrl`, `auth`, `urlSource`)

## 说明

-   `--token` 和 `--password` 是互斥的。
-   使用 `--remote` 时，如果实际有效的远程凭证已配置为 SecretRefs 且您未传递 `--token` 或 `--password`，命令将从活动网关快照中解析它们。如果网关不可用，命令将快速失败。
-   不使用 `--remote` 时，当未传递 CLI 身份验证覆盖时，将解析本地网关身份验证 SecretRefs：
    -   当令牌身份验证可以胜出时（显式设置 `gateway.auth.mode="token"` 或推断出的模式中无密码来源胜出），解析 `gateway.auth.token`。
    -   当密码身份验证可以胜出时（显式设置 `gateway.auth.mode="password"` 或推断出的模式中无来自身份验证/环境的获胜令牌），解析 `gateway.auth.password`。
-   如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password`（包括 SecretRefs）且 `gateway.auth.mode` 未设置，则在模式被显式设置之前，设置码解析将失败。
-   网关版本偏差说明：此命令路径需要支持 `secrets.resolve` 的网关；较旧的网关将返回未知方法错误。
-   扫描后，使用以下命令批准设备配对：
    -   `openclaw devices list`
    -   `openclaw devices approve `

[插件](./plugins.md)[重置](./reset.md)