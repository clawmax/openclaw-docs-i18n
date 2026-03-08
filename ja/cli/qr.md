

  CLIコマンド

  
# qr

現在のGateway設定からiOSペアリングQRとセットアップコードを生成します。

## 使用方法

```bash
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws --token '<token>'
```

## オプション

-   `--remote`: 設定から`gateway.remote.url`とリモートトークン/パスワードを使用
-   `--url `: ペイロードで使用するゲートウェイURLを上書き
-   `--public-url `: ペイロードで使用する公開URLを上書き
-   `--token `: ペイロード用のゲートウェイトークンを上書き
-   `--password `: ペイロード用のゲートウェイパスワードを上書き
-   `--setup-code-only`: セットアップコードのみを表示
-   `--no-ascii`: ASCII QRレンダリングをスキップ
-   `--json`: JSONを出力 (`setupCode`, `gatewayUrl`, `auth`, `urlSource`)

## 注意事項

-   `--token`と`--password`は同時に指定できません。
-   `--remote`を使用する場合、有効なリモート認証情報がSecretRefsとして設定されており、`--token`または`--password`を渡さない場合、コマンドはアクティブなゲートウェイスナップショットからそれらを解決します。ゲートウェイが利用できない場合、コマンドはすぐに失敗します。
-   `--remote`なしの場合、CLI認証の上書きが渡されないとき、ローカルゲートウェイ認証SecretRefsが解決されます:
    -   `gateway.auth.token`は、トークン認証が有効な場合（明示的な`gateway.auth.mode="token"`または、パスワードソースが優先されない推論モード）に解決されます。
    -   `gateway.auth.password`は、パスワード認証が有効な場合（明示的な`gateway.auth.mode="password"`または、認証/環境変数から優先トークンがない推論モード）に解決されます。
-   `gateway.auth.token`と`gateway.auth.password`の両方が（SecretRefsを含めて）設定されており、`gateway.auth.mode`が設定されていない場合、モードが明示的に設定されるまでセットアップコードの解決は失敗します。
-   ゲートウェイバージョンスキューの注意: このコマンドパスは`secrets.resolve`をサポートするゲートウェイが必要です。古いゲートウェイは不明なメソッドエラーを返します。
-   スキャン後、以下のコマンドでデバイスペアリングを承認します:
    -   `openclaw devices list`
    -   `openclaw devices approve `

[plugins](./plugins.md)[reset](./reset.md)