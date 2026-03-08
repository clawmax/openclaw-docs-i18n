

  ウェブインターフェース

  
# ダッシュボード

Gateway ダッシュボードは、デフォルトで `/` で提供されるブラウザのコントロールUIです (`gateway.controlUi.basePath` で上書き可能)。クイックオープン (ローカル Gateway):

-   [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (または [http://localhost:18789/](http://localhost:18789/))

主な参照先:

-   [コントロールUI](./control-ui.md) - 使用方法とUI機能について。
-   [Tailscale](../gateway/tailscale.md) - Serve/Funnel の自動化について。
-   [ウェブサーフェス](../web.md) - バインドモードとセキュリティ上の注意点。

認証は WebSocket ハンドシェイク時に `connect.params.auth` (トークンまたはパスワード) を介して強制されます。 [Gateway設定](../gateway/configuration.md) の `gateway.auth` を参照してください。セキュリティ上の注意: コントロールUIは **管理画面** (チャット、設定、実行承認) です。公開しないでください。UIはダッシュボードのURLトークンを現在のタブのメモリに保持し、読み込み後にURLからそれらを削除します。localhost、Tailscale Serve、またはSSHトンネルの使用を推奨します。

## 高速パス (推奨)

-   オンボーディング後、CLIは自動的にダッシュボードを開き、クリーンな (トークン化されていない) リンクを表示します。
-   いつでも再オープン: `openclaw dashboard` (リンクをコピーし、可能であればブラウザを開き、ヘッドレスの場合はSSHヒントを表示)。
-   UIが認証を要求した場合、`gateway.auth.token` (または `OPENCLAW_GATEWAY_TOKEN`) からのトークンをコントロールUI設定に貼り付けてください。

## トークンの基本 (ローカル vs リモート)

-   **Localhost**: `http://127.0.0.1:18789/` を開きます。
-   **トークンソース**: `gateway.auth.token` (または `OPENCLAW_GATEWAY_TOKEN`); `openclaw dashboard` はURLフラグメントを介して一度限りのブートストラップ用にトークンを渡すことができますが、コントロールUIはゲートウェイトークンをlocalStorageに永続化しません。
-   `gateway.auth.token` が SecretRef で管理されている場合、`openclaw dashboard` は設計上、トークン化されていないURLを表示/コピー/開きます。これにより、外部で管理されるトークンがシェルログ、クリップボード履歴、またはブラウザ起動引数に露出するのを防ぎます。
-   `gateway.auth.token` が SecretRef として設定されており、現在のシェルで未解決の場合、`openclaw dashboard` は依然としてトークン化されていないURLと、実行可能な認証設定ガイダンスを表示します。
-   **Localhost以外**: Tailscale Serve を使用します ( `gateway.auth.allowTailscale: true` の場合、コントロールUI/WebSocketはトークンレスで動作します。信頼されたゲートウェイホストを前提とします。HTTP APIは依然としてトークン/パスワードが必要です)、トークン付きのtailnetバインド、またはSSHトンネルを使用します。 [ウェブサーフェス](../web.md) を参照してください。

## 「unauthorized」 / 1008 が表示される場合

-   ゲートウェーに到達可能であることを確認してください (ローカル: `openclaw status`; リモート: SSHトンネル `ssh -N -L 18789:127.0.0.1:18789 user@host` を実行後、`http://127.0.0.1:18789/` を開く)。
-   ゲートウェイホストからトークンを取得または提供してください:
    -   プレーンテキスト設定: `openclaw config get gateway.auth.token`
    -   SecretRef管理設定: 外部シークレットプロバイダを解決するか、このシェルで `OPENCLAW_GATEWAY_TOKEN` をエクスポートし、その後 `openclaw dashboard` を再実行
    -   トークンが設定されていない: `openclaw doctor --generate-gateway-token`
-   ダッシュボード設定で、トークンを認証フィールドに貼り付け、接続してください。

[コントロールUI](./control-ui.md)[WebChat](./webchat.md)