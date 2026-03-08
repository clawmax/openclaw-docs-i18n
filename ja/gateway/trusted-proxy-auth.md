

  設定と運用

  
# 信頼済みプロキシ認証

> ⚠️ **セキュリティに敏感な機能です。** このモードでは認証を完全にリバースプロキシに委任します。設定ミスにより、Gatewayが不正アクセスにさらされる可能性があります。有効にする前に、このページを注意深くお読みください。

## 使用する場面

以下の場合に `trusted-proxy` 認証モードを使用します:

-   **アイデンティティ対応プロキシ** (Pomerium、Caddy + OAuth、nginx + oauth2-proxy、Traefik + forward auth) の背後でOpenClawを実行している場合
-   プロキシがすべての認証を処理し、ヘッダー経由でユーザーIDを渡す場合
-   Gatewayへの唯一の経路がプロキシであるKubernetesやコンテナ環境にいる場合
-   WebSocket `1008 unauthorized` エラーが発生し、ブラウザがWSペイロードでトークンを渡せない場合

## 使用しない場面

-   プロキシがユーザー認証を行わない場合 (単なるTLSターミネータやロードバランサの場合)
-   Gatewayへの経路でプロキシをバイパスするものがある場合 (ファイアウォールの穴、内部ネットワークアクセス)
-   プロキシが転送ヘッダーを正しく削除/上書きしているか確信が持てない場合
-   個人用のシングルユーザーアクセスのみ必要な場合 (よりシンプルな設定にはTailscale Serve + ループバックを検討してください)

## 仕組み

1.  リバースプロキシがユーザーを認証します (OAuth、OIDC、SAMLなど)
2.  プロキシが認証済みユーザーIDを含むヘッダーを追加します (例: `x-forwarded-user: nick@example.com`)
3.  OpenClawがリクエストが **信頼済みプロキシIP** から来たことを確認します (`gateway.trustedProxies` で設定)
4.  OpenClawが設定されたヘッダーからユーザーIDを抽出します
5.  すべてのチェックが通ると、リクエストは認可されます

## コントロールUIペアリング動作

`gateway.auth.mode = "trusted-proxy"` が有効で、リクエストが信頼済みプロキシチェックを通過すると、コントロールUIのWebSocketセッションはデバイスペアリングIDなしで接続できます。意味するところ:

-   このモードでは、ペアリングはコントロールUIアクセスの主要なゲートではなくなります。
-   リバースプロキシの認証ポリシーと `allowUsers` が実効的なアクセス制御になります。
-   Gatewayのイングレスは信頼済みプロキシIPのみに制限してください (`gateway.trustedProxies` + ファイアウォール)。

## 設定

```json
{
  gateway: {
    // 同一ホストのプロキシ設定にはloopbackを、リモートプロキシホストにはlan/customを使用
    bind: "loopback",

    // 重要: ここにはプロキシのIPのみを追加してください
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // 認証済みユーザーIDを含むヘッダー (必須)
        userHeader: "x-forwarded-user",

        // オプション: 存在しなければならないヘッダー (プロキシ検証用)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // オプション: 特定のユーザーに制限 (空 = すべて許可)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

`gateway.bind` が `loopback` の場合、`gateway.trustedProxies` にループバックプロキシアドレス (`127.0.0.1`、`::1`、または同等のループバックCIDR) を含めてください。

### 設定リファレンス

| フィールド | 必須 | 説明 |
| --- | --- | --- |
| `gateway.trustedProxies` | はい | 信頼するプロキシIPアドレスの配列。他のIPからのリクエストは拒否されます。 |
| `gateway.auth.mode` | はい | `"trusted-proxy"` である必要があります |
| `gateway.auth.trustedProxy.userHeader` | はい | 認証済みユーザーIDを含むヘッダー名 |
| `gateway.auth.trustedProxy.requiredHeaders` | いいえ | リクエストを信頼するために存在しなければならない追加ヘッダー |
| `gateway.auth.trustedProxy.allowUsers` | いいえ | 許可するユーザーIDのリスト。空はすべての認証済みユーザーを許可します。 |

## TLS終端とHSTS

TLS終端点は1つとし、そこでHSTSを適用してください。

### 推奨パターン: プロキシでのTLS終端

リバースプロキシが `https://control.example.com` のHTTPSを処理する場合、そのドメインに対してプロキシで `Strict-Transport-Security` を設定します。

-   インターネットに公開するデプロイメントに適しています。
-   証明書とHTTP強化ポリシーを一箇所にまとめます。
-   OpenClawはプロキシ背後でループバックHTTPのままにできます。

ヘッダー値の例:

```yaml
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### GatewayでのTLS終端

OpenClaw自体が直接HTTPSを提供する場合 (TLS終端プロキシなし):

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

`strictTransportSecurity` は文字列のヘッダー値、または明示的に無効にする場合は `false` を受け付けます。

### ロールアウトガイダンス

-   まず、トラフィックを検証しながら短いmax-age (例: `max-age=300`) で開始します。
-   確信が持てた後にのみ、長期間の値 (例: `max-age=31536000`) に増やします。
-   すべてのサブドメインがHTTPS対応している場合にのみ `includeSubDomains` を追加します。
-   フルドメインセットに対して意図的にプリロード要件を満たす場合にのみ、プリロードを使用します。
-   ループバックのみのローカル開発ではHSTSの恩恵はありません。

## プロキシ設定例

### Pomerium

PomeriumはIDを `x-pomerium-claim-email` (または他のクレームヘッダー) で、JWTを `x-pomerium-jwt-assertion` で渡します。

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // PomeriumのIP
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

Pomerium設定スニペット:

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

`caddy-security` プラグインを備えたCaddyはユーザー認証を行い、IDヘッダーを渡せます。

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // CaddyのIP (同一ホストの場合)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfileスニペット:

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

oauth2-proxyはユーザー認証を行い、IDを `x-auth-request-email` で渡します。

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxyのIP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

nginx設定スニペット:

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
    trustedProxies: ["172.17.0.1"], // TraefikコンテナのIP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## セキュリティチェックリスト

信頼済みプロキシ認証を有効にする前に、以下を確認してください:

-   [ ]  **プロキシが唯一の経路**: Gatewayポートはプロキシ以外のすべてからファイアウォールで遮断されている
-   [ ]  **trustedProxiesは最小限**: 実際のプロキシIPのみで、サブネット全体ではない
-   [ ]  **プロキシがヘッダーを削除**: プロキシがクライアントからの `x-forwarded-*` ヘッダーを追加ではなく上書きする
-   [ ]  **TLS終端**: プロキシがTLSを処理し、ユーザーはHTTPS経由で接続する
-   [ ]  **allowUsersが設定されている** (推奨): 認証された誰でも許可するのではなく、既知のユーザーに制限する

## セキュリティ監査

`openclaw security audit` は、信頼済みプロキシ認証を **重大** な深刻度の所見としてフラグを立てます。これは意図的です — セキュリティをプロキシ設定に委任していることを思い出させるためです。監査は以下をチェックします:

-   `trustedProxies` 設定の欠落
-   `userHeader` 設定の欠落
-   空の `allowUsers` (認証されたすべてのユーザーを許可)

## トラブルシューティング

### ”trusted\_proxy\_untrusted\_source”

リクエストが `gateway.trustedProxies` 内のIPから来ていません。以下を確認してください:

-   プロキシIPは正しいですか? (DockerコンテナIPは変更される可能性があります)
-   プロキシの前にロードバランサがありますか?
-   `docker inspect` または `kubectl get pods -o wide` を使用して実際のIPを確認してください

### ”trusted\_proxy\_user\_missing”

ユーザーヘッダーが空または欠落していました。以下を確認してください:

-   プロキシがIDヘッダーを渡すように設定されていますか?
-   ヘッダー名は正しいですか? (大文字小文字は区別されませんが、スペルは重要です)
-   ユーザーは実際にプロキシで認証されていますか?

### “trustedproxy\_missing\_header\*”

必須ヘッダーが存在しませんでした。以下を確認してください:

-   それらの特定のヘッダーに関するプロキシ設定
-   ヘッダーがチェーンのどこかで削除されていないか

### ”trusted\_proxy\_user\_not\_allowed”

ユーザーは認証されていますが、`allowUsers` に含まれていません。ユーザーを追加するか、許可リストを削除してください。

### WebSocketがまだ失敗する

プロキシが以下を確実に行っているか確認してください:

-   WebSocketアップグレードをサポートしている (`Upgrade: websocket`、`Connection: upgrade`)
-   IDヘッダーをWebSocketアップグレードリクエスト (HTTPだけでなく) で渡している
-   WebSocket接続用に別の認証パスがない

## トークン認証からの移行

トークン認証から信頼済みプロキシに移行する場合:

1.  プロキシを設定してユーザー認証を行い、ヘッダーを渡すようにする
2.  プロキシ設定を独立してテストする (ヘッダー付きcurl)
3.  OpenClaw設定を信頼済みプロキシ認証で更新する
4.  Gatewayを再起動する
5.  コントロールUIからのWebSocket接続をテストする
6.  `openclaw security audit` を実行し、所見を確認する

## 関連項目

-   [セキュリティ](./security.md) — 完全なセキュリティガイド
-   [設定](./configuration.md) — 設定リファレンス
-   [リモートアクセス](./remote.md) — その他のリモートアクセスパターン
-   [Tailscale](./tailscale.md) — テールネットのみのアクセスにはよりシンプルな代替手段

[Secrets Apply Plan Contract](./secrets-plan-contract.md)[ヘルスチェック](./health.md)