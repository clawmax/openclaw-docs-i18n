

  リモートアクセス

  
# Tailscale

OpenClawは、GatewayダッシュボードとWebSocketポートに対してTailscale **Serve**（tailnet）または**Funnel**（公開）を自動設定できます。これにより、Gatewayはループバックにバインドされたまま、TailscaleがHTTPS、ルーティング、および（Serveの場合）アイデンティティヘッダーを提供します。

## モード

-   `serve`: `tailscale serve`によるTailnet専用のServe。ゲートウェイは`127.0.0.1`のままです。
-   `funnel`: `tailscale funnel`による公開HTTPS。OpenClawは共有パスワードを必要とします。
-   `off`: デフォルト（Tailscale自動化なし）。

## 認証

`gateway.auth.mode`を設定してハンドシェイクを制御します：

-   `token`（`OPENCLAW_GATEWAY_TOKEN`が設定されている場合のデフォルト）
-   `password`（`OPENCLAW_GATEWAY_PASSWORD`または設定による共有シークレット）

`tailscale.mode = "serve"`かつ`gateway.auth.allowTailscale`が`true`の場合、Control UI/WebSocket認証は、トークン/パスワードを提供せずにTailscaleアイデンティティヘッダー（`tailscale-user-login`）を使用できます。OpenClawは、`x-forwarded-for`アドレスをローカルのTailscaleデーモン（`tailscale whois`）経由で解決し、ヘッダーと一致することを確認してから受け入れます。OpenClawは、リクエストがTailscaleの`x-forwarded-for`、`x-forwarded-proto`、`x-forwarded-host`ヘッダーと共にループバックから到着した場合にのみ、Serveとして扱います。HTTP APIエンドポイント（例：`/v1/*`、`/tools/invoke`、`/api/channels/*`）は引き続きトークン/パスワード認証を必要とします。このトークンレスフローは、ゲートウェイホストが信頼されていることを前提としています。信頼できないローカルコードが同じホストで実行される可能性がある場合は、`gateway.auth.allowTailscale`を無効にし、代わりにトークン/パスワード認証を要求してください。明示的な認証情報を要求するには、`gateway.auth.allowTailscale: false`を設定するか、`gateway.auth.mode: "password"`を強制します。

## 設定例

### Tailnet専用（Serve）

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

開く：`https:///`（または設定した`gateway.controlUi.basePath`）

### Tailnet専用（Tailnet IPにバインド）

GatewayをTailnet IPで直接リッスンさせたい場合（Serve/Funnelなし）に使用します。

```json
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

別のTailnetデバイスから接続：

-   Control UI: `http://<tailscale-ip>:18789/`
-   WebSocket: `ws://<tailscale-ip>:18789`

注：このモードではループバック（`http://127.0.0.1:18789`）は動作**しません**。

### 公開インターネット（Funnel + 共有パスワード）

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

パスワードをディスクに保存するよりも`OPENCLAW_GATEWAY_PASSWORD`の使用を推奨します。

## CLI例

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## 注意事項

-   Tailscale Serve/Funnelには、`tailscale` CLIがインストールされ、ログインされている必要があります。
-   `tailscale.mode: "funnel"`は、公開露出を避けるため、認証モードが`password`でない限り起動を拒否します。
-   シャットダウン時に`tailscale serve`または`tailscale funnel`の設定を元に戻したい場合は、`gateway.tailscale.resetOnExit`を設定してください。
-   `gateway.bind: "tailnet"`は直接のTailnetバインドです（HTTPSなし、Serve/Funnelなし）。
-   `gateway.bind: "auto"`はループバックを優先します；Tailnet専用にしたい場合は`tailnet`を使用してください。
-   Serve/Funnelは**Gateway control UI + WS**のみを公開します。ノードは同じGateway WSエンドポイント経由で接続するため、Serveはノードアクセスにも機能します。

## ブラウザ制御（リモートGateway + ローカルブラウザ）

Gatewayを1台のマシンで実行し、別のマシン上のブラウザを操作したい場合は、ブラウザマシン上で**ノードホスト**を実行し、両方を同じtailnet上に維持してください。Gatewayはブラウザの操作をノードにプロキシします；別個の制御サーバーやServe URLは必要ありません。ブラウザ制御にはFunnelを使用しないでください；ノードペアリングはオペレーターアクセスと同様に扱います。

## Tailscaleの前提条件と制限

-   ServeにはtailnetでHTTPSが有効になっている必要があります；CLIは不足している場合にプロンプトを表示します。
-   ServeはTailscaleアイデンティティヘッダーを注入します；Funnelは注入しません。
-   FunnelにはTailscale v1.38.3以上、MagicDNS、HTTPS有効化、およびfunnelノード属性が必要です。
-   FunnelはTLS経由のポート`443`、`8443`、`10000`のみをサポートします。
-   macOSでのFunnelには、オープンソースのTailscaleアプリバリアントが必要です。

## 詳細情報

-   Tailscale Serve概要: [https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
-   `tailscale serve`コマンド: [https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
-   Tailscale Funnel概要: [https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
-   `tailscale funnel`コマンド: [https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)

[リモートGateway設定](./remote-gateway-readme.md)[形式検証（セキュリティモデル）](../security/formal-verification.md)