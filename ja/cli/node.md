

  CLI コマンド

  
# node

このマシン上で `system.run` / `system.which` を公開し、ゲートウェイ WebSocket に接続する **ヘッドレスノードホスト** を実行します。

## ノードホストを使用する理由

他のマシンに完全な macOS コンパニオンアプリをインストールせずに、エージェントがネットワーク内の **他のマシンでコマンドを実行** できるようにしたい場合にノードホストを使用します。一般的なユースケース:

-   リモートの Linux/Windows マシン（ビルドサーバー、ラボマシン、NAS）でコマンドを実行する。
-   実行をゲートウェイ上で **サンドボックス化** したまま、承認された実行を他のホストに委譲する。
-   自動化や CI ノード用の軽量なヘッドレス実行ターゲットを提供する。

実行は依然としてノードホスト上の **実行承認** とエージェントごとの許可リストによって保護されるため、コマンドアクセスを範囲限定かつ明示的に保つことができます。

## ブラウザプロキシ (ゼロ設定)

ノードホストは、ノードで `browser.enabled` が無効化されていない場合、自動的にブラウザプロキシをアドバタイズします。これにより、エージェントは追加設定なしでそのノード上でブラウザ自動化を使用できます。必要に応じてノードで無効化してください:

```json
{
  nodeHost: {
    browserProxy: {
      enabled: false,
    },
  },
}
```

## 実行 (フォアグラウンド)

```bash
openclaw node run --host <gateway-host> --port 18789
```

オプション:

-   `--host `: ゲートウェイ WebSocket ホスト (デフォルト: `127.0.0.1`)
-   `--port `: ゲートウェイ WebSocket ポート (デフォルト: `18789`)
-   `--tls`: ゲートウェイ接続に TLS を使用
-   `--tls-fingerprint `: 期待される TLS 証明書フィンガープリント (sha256)
-   `--node-id `: ノード ID を上書き (ペアリングトークンをクリア)
-   `--display-name `: ノード表示名を上書き

## サービス (バックグラウンド)

ヘッドレスノードホストをユーザーサービスとしてインストールします。

```bash
openclaw node install --host <gateway-host> --port 18789
```

オプション:

-   `--host `: ゲートウェイ WebSocket ホスト (デフォルト: `127.0.0.1`)
-   `--port `: ゲートウェイ WebSocket ポート (デフォルト: `18789`)
-   `--tls`: ゲートウェイ接続に TLS を使用
-   `--tls-fingerprint `: 期待される TLS 証明書フィンガープリント (sha256)
-   `--node-id `: ノード ID を上書き (ペアリングトークンをクリア)
-   `--display-name `: ノード表示名を上書き
-   `--runtime `: サービスランタイム (`node` または `bun`)
-   `--force`: 既にインストール済みの場合も再インストール/上書き

サービスの管理:

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

フォアグラウンドのノードホスト (サービスなし) には `openclaw node run` を使用してください。サービスコマンドは機械可読な出力のために `--json` を受け付けます。

## ペアリング

最初の接続時に、ゲートウェイ上で保留中のデバイスペアリングリクエスト (`role: node`) が作成されます。以下のコマンドで承認してください:

```bash
openclaw devices list
openclaw devices approve <requestId>
```

ノードホストは、そのノード ID、トークン、表示名、およびゲートウェイ接続情報を `~/.openclaw/node.json` に保存します。

## 実行承認

`system.run` はローカルの実行承認によって制御されます:

-   `~/.openclaw/exec-approvals.json`
-   [実行承認](../tools/exec-approvals.md)
-   `openclaw approvals --node <id|name|ip>` (ゲートウェイから編集)

[models](./models.md)[nodes](./nodes.md)