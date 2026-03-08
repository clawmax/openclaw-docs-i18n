title: "OpenClaw Webインターフェース、Control UI、Webhook、セキュリティ"
description: "OpenClaw GatewayのWebインターフェース、Control UI、Webhook、Tailscaleアクセスの設定方法を学びます。セキュリティ、認証、デプロイメントモードを設定します。"
keywords: ["openclaw webインターフェース", "gateway control ui", "tailscale統合", "webhook設定", "gatewayセキュリティ", "ループバックバインド", "ファンネルデプロイメント", "認証トークン"]
---

  Webインターフェース

  
# Web

Gatewayは、Gateway WebSocketと同じポートから小さな**ブラウザ用Control UI** (Vite + Lit)を提供します:

-   デフォルト: `http://:18789/`
-   オプションのプレフィックス: `gateway.controlUi.basePath`を設定 (例: `/openclaw`)

機能の詳細は[Control UI](./web/control-ui.md)にあります。このページでは、バインドモード、セキュリティ、およびWeb向けのインターフェースに焦点を当てます。

## Webhook

`hooks.enabled=true`の場合、Gatewayは同じHTTPサーバー上に小さなWebhookエンドポイントも公開します。認証とペイロードについては、[Gateway設定](./gateway/configuration.md) → `hooks`を参照してください。

## 設定 (デフォルトで有効)

Control UIは、アセットが存在する場合 (`dist/control-ui`)、**デフォルトで有効**です。設定で制御できます:

```json
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePathはオプション
  },
}
```

## Tailscaleアクセス

### 統合Serve (推奨)

Gatewayをループバック上に維持し、Tailscale Serveにプロキシさせます:

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

その後、gatewayを起動します:

```bash
openclaw gateway
```

開くURL:

-   `https:///` (または設定した`gateway.controlUi.basePath`)

### Tailnetバインド + トークン

```json
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

その後、gatewayを起動します (ループバック以外のバインドではトークンが必要):

```bash
openclaw gateway
```

開くURL:

-   `http://<tailscale-ip>:18789/` (または設定した`gateway.controlUi.basePath`)

### パブリックインターネット (Funnel)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // または OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## セキュリティに関する注意点

-   Gateway認証はデフォルトで必要です (トークン/パスワードまたはTailscale IDヘッダー)。
-   ループバック以外のバインドでも、共有トークン/パスワード (`gateway.auth`または環境変数) が**必要**です。
-   ウィザードはデフォルトでgatewayトークンを生成します (ループバック上でも)。
-   UIは`connect.params.auth.token`または`connect.params.auth.password`を送信します。
-   ループバック以外のControl UIデプロイメントでは、`gateway.controlUi.allowedOrigins`を明示的に設定してください (完全なオリジン)。これがない場合、デフォルトではgatewayの起動が拒否されます。
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true`はHostヘッダーによるオリジンフォールバックモードを有効にしますが、セキュリティを危険に低下させます。
-   Serveを使用する場合、`gateway.auth.allowTailscale`が`true`のとき、Tailscale IDヘッダーがControl UI/WebSocket認証を満たすことができます (トークン/パスワードは不要)。HTTP APIエンドポイントは引き続きトークン/パスワードを必要とします。明示的な認証情報を要求するには`gateway.auth.allowTailscale: false`を設定します。詳細は[Tailscale](./gateway/tailscale.md)および[セキュリティ](./gateway/security.md)を参照してください。このトークンレスフローは、gatewayホストが信頼されていることを前提としています。
-   `gateway.tailscale.mode: "funnel"`は`gateway.auth.mode: "password"` (共有パスワード) を必要とします。

## UIのビルド

Gatewayは`dist/control-ui`から静的ファイルを提供します。以下のコマンドでビルドします:

```bash
pnpm ui:build # 初回実行時はUIの依存関係を自動インストールします
```

[脅威モデルへの貢献](./security/CONTRIBUTING-THREAT-MODEL.md)[Control UI](./web/control-ui.md)