

  CLI コマンド

  
# gateway

GatewayはOpenClawのWebSocketサーバーです（チャネル、ノード、セッション、フック）。このページのサブコマンドは `openclaw gateway …` の下にあります。関連ドキュメント：

-   [/gateway/bonjour](../gateway/bonjour.md)
-   [/gateway/discovery](../gateway/discovery.md)
-   [/gateway/configuration](../gateway/configuration.md)

## Gatewayの実行

ローカルのGatewayプロセスを実行します：

```bash
openclaw gateway
```

フォアグラウンドエイリアス：

```bash
openclaw gateway run
```

注意点：

-   デフォルトでは、`~/.openclaw/openclaw.json` に `gateway.mode=local` が設定されていない限り、Gatewayは起動を拒否します。アドホック/開発用の実行には `--allow-unconfigured` を使用してください。
-   認証なしでループバックを超えてバインドすることはブロックされます（安全ガードレール）。
-   `SIGUSR1` は、許可されている場合にプロセス内再起動をトリガーします（`commands.restart` はデフォルトで有効です； `commands.restart: false` を設定すると手動再起動をブロックしますが、gatewayツール/設定の適用/更新は引き続き許可されます）。
-   `SIGINT`/`SIGTERM` ハンドラはgatewayプロセスを停止しますが、カスタムターミナル状態は復元しません。CLIをTUIやraw-mode入力でラップしている場合は、終了前にターミナルを復元してください。

### オプション

-   `--port `: WebSocketポート（デフォルトは設定/環境変数から取得；通常 `18789`）。
-   `--bind <loopback|lan|tailnet|auto|custom>`: リスナーバインドモード。
-   `--auth <token|password>`: 認証モードの上書き。
-   `--token `: トークンの上書き（プロセスの `OPENCLAW_GATEWAY_TOKEN` も設定します）。
-   `--password `: パスワードの上書き（プロセスの `OPENCLAW_GATEWAY_PASSWORD` も設定します）。
-   `--tailscale <off|serve|funnel>`: Tailscale経由でGatewayを公開します。
-   `--tailscale-reset-on-exit`: シャットダウン時にTailscaleのserve/funnel設定をリセットします。
-   `--allow-unconfigured`: 設定に `gateway.mode=local` がなくてもgatewayの起動を許可します。
-   `--dev`: 開発用設定+ワークスペースが存在しない場合に作成します（BOOTSTRAP.mdをスキップします）。
-   `--reset`: 開発用設定+認証情報+セッション+ワークスペースをリセットします（`--dev` が必要です）。
-   `--force`: 起動前に選択されたポートで既存のリスナーを強制終了します。
-   `--verbose`: 詳細ログを出力します。
-   `--claude-cli-logs`: コンソールにclaude-cliのログのみを表示します（およびそのstdout/stderrを有効にします）。
-   `--ws-log <auto|full|compact>`: websocketログスタイル（デフォルト `auto`）。
-   `--compact`: `--ws-log compact` のエイリアスです。
-   `--raw-stream`: 生のモデルストリームイベントをjsonlとしてログ出力します。
-   `--raw-stream-path `: 生ストリームjsonlのパス。

## 実行中のGatewayへのクエリ

すべてのクエリコマンドはWebSocket RPCを使用します。出力モード：

-   デフォルト: 人間が読みやすい形式（TTYでは色付き）。
-   `--json`: 機械可読なJSON（スタイリング/スピナーなし）。
-   `--no-color`（または `NO_COLOR=1`）: ANSIを無効にしつつ人間向けレイアウトを維持します。

共有オプション（サポートされている場合）：

-   `--url `: Gateway WebSocket URL。
-   `--token `: Gatewayトークン。
-   `--password `: Gatewayパスワード。
-   `--timeout `: タイムアウト/予算（コマンドごとに異なります）。
-   `--expect-final`: 「最終」応答を待ちます（エージェント呼び出し）。

注意：`--url` を設定すると、CLIは設定や環境変数の認証情報にフォールバックしません。`--token` または `--password` を明示的に渡してください。明示的な認証情報がない場合はエラーになります。

### gateway health

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### gateway status

`gateway status` はGatewayサービス（launchd/systemd/schtasks）とオプションのRPCプローブを表示します。

```bash
openclaw gateway status
openclaw gateway status --json
```

オプション：

-   `--url `: プローブURLを上書きします。
-   `--token `: プローブのトークン認証。
-   `--password `: プローブのパスワード認証。
-   `--timeout `: プローブタイムアウト（デフォルト `10000`）。
-   `--no-probe`: RPCプローブをスキップします（サービスのみの表示）。
-   `--deep`: システムレベルのサービスもスキャンします。

注意点：

-   `gateway status` は、可能な場合、プローブ認証のために設定された認証SecretRefを解決します。
-   このコマンドパスで必要な認証SecretRefが解決されていない場合、プローブ認証は失敗する可能性があります； `--token`/`--password` を明示的に渡すか、まずシークレットソースを解決してください。

### gateway probe

`gateway probe` は「すべてをデバッグする」コマンドです。常に以下をプローブします：

-   設定されたリモートゲートウェイ（設定されている場合）、および
-   **リモートが設定されていても** localhost（ループバック）。

複数のゲートウェイに到達可能な場合は、すべてを表示します。分離されたプロファイル/ポート（例：レスキューボット）を使用する場合、複数のゲートウェイがサポートされますが、ほとんどのインストールでは単一のゲートウェイを実行します。

```bash
openclaw gateway probe
openclaw gateway probe --json
```

#### SSH経由のリモート（Macアプリ同等）

macOSアプリの「SSH経由のリモート」モードは、ローカルポートフォワードを使用して、リモートゲートウェイ（ループバックのみにバインドされている可能性があります）を `ws://127.0.0.1:` で到達可能にします。CLI同等コマンド：

```bash
openclaw gateway probe --ssh user@gateway-host
```

オプション：

-   `--ssh `: `user@host` または `user@host:port`（ポートのデフォルトは `22`）。
-   `--ssh-identity `: アイデンティティファイル。
-   `--ssh-auto`: 最初に発見されたゲートウェイホストをSSHターゲットとして選択します（LAN/WABのみ）。

設定（オプション、デフォルトとして使用）：

-   `gateway.remote.sshTarget`
-   `gateway.remote.sshIdentity`

### gateway call &lt;method&gt;

低レベルRPCヘルパー。

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

## Gatewayサービスの管理

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

注意点：

-   `gateway install` は `--port`、`--runtime`、`--token`、`--force`、`--json` をサポートします。
-   トークン認証にトークンが必要で、`gateway.auth.token` がSecretRefで管理されている場合、`gateway install` はSecretRefが解決可能であることを検証しますが、解決されたトークンをサービス環境メタデータに永続化しません。
-   トークン認証にトークンが必要で、設定されたトークンSecretRefが解決されていない場合、インストールは閉じて失敗し、フォールバックのプレーンテキストを永続化しません。
-   推論された認証モードでは、シェル専用の `OPENCLAW_GATEWAY_PASSWORD`/`CLAWDBOT_GATEWAY_PASSWORD` はインストールのトークン要件を緩和しません；管理対象サービスをインストールする場合は、耐久性のある設定（`gateway.auth.password` または設定 `env`）を使用してください。
-   `gateway.auth.token` と `gateway.auth.password` の両方が設定されていて、`gateway.auth.mode` が設定されていない場合、モードが明示的に設定されるまでインストールはブロックされます。
-   ライフサイクルコマンドはスクリプティング用に `--json` を受け入れます。

## ゲートウェイの発見（Bonjour）

`gateway discover` はGatewayビーコン（`_openclaw-gw._tcp`）をスキャンします。

-   マルチキャストDNS-SD: `local.`
-   ユニキャストDNS-SD（Wide-Area Bonjour）: ドメイン（例：`openclaw.internal.`）を選択し、スプリットDNS + DNSサーバーを設定します；詳細は [/gateway/bonjour](../gateway/bonjour.md) を参照してください。

Bonjour発見が有効（デフォルト）なゲートウェイのみがビーコンをアドバタイズします。Wide-Area発見レコードには以下が含まれます（TXT）：

-   `role`（ゲートウェイの役割ヒント）
-   `transport`（トランスポートヒント、例：`gateway`）
-   `gatewayPort`（WebSocketポート、通常 `18789`）
-   `sshPort`（SSHポート；存在しない場合はデフォルト `22`）
-   `tailnetDns`（MagicDNSホスト名、利用可能な場合）
-   `gatewayTls` / `gatewayTlsSha256`（TLS有効 + 証明書フィンガープリント）
-   `cliPath`（リモートインストール用のオプションヒント）

### gateway discover

```bash
openclaw gateway discover
```

オプション：

-   `--timeout `: コマンドごとのタイムアウト（ブラウズ/解決）；デフォルト `2000`。
-   `--json`: 機械可読な出力（スタイリング/スピナーも無効になります）。

例：

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```

[doctor](./doctor.md)[health](./health.md)