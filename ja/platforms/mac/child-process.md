

  macOSコンパニオンアプリ

  
# Gatewayライフサイクル

macOSアプリは、デフォルトで**launchdを介してGatewayを管理**し、Gatewayを子プロセスとして生成しません。まず、設定されたポートで既に実行中のGatewayへのアタッチを試みます。到達可能なGatewayがない場合、外部の`openclaw` CLI（組み込みランタイムなし）を介してlaunchdサービスを有効にします。これにより、ログイン時の信頼性の高い自動起動とクラッシュ時の再起動が実現します。子プロセスモード（アプリによって直接生成されるGateway）は、現在**使用されていません**。UIとのより緊密な結合が必要な場合は、ターミナルでGatewayを手動で実行してください。

## デフォルトの動作 (launchd)

-   アプリは、ユーザーごとのLaunchAgentを`ai.openclaw.gateway`というラベルでインストールします（`--profile`/`OPENCLAW_PROFILE`を使用する場合は`ai.openclaw.`。従来の`com.openclaw.*`もサポートされています）。
-   ローカルモードが有効な場合、アプリはLaunchAgentがロードされていることを確認し、必要に応じてGatewayを起動します。
-   ログはlaunchdのgatewayログパスに書き込まれます（デバッグ設定で確認可能）。

一般的なコマンド:

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

名前付きプロファイルを実行している場合は、ラベルを`ai.openclaw.`に置き換えてください。

## 未署名の開発ビルド

`scripts/restart-mac.sh --no-sign`は、署名鍵を持たない場合の高速なローカルビルド用です。launchdが未署名のリレーバイナリを指すのを防ぐために、このスクリプトは以下のことを行います:

-   `~/.openclaw/disable-launchagent`を書き込みます。

`scripts/restart-mac.sh`の署名済み実行は、このマーカーが存在する場合、このオーバーライドをクリアします。手動でリセットするには:

```bash
rm ~/.openclaw/disable-launchagent
```

## アタッチ専用モード

macOSアプリが**launchdを決してインストールまたは管理しない**ように強制するには、`--attach-only`（または`--no-launchd`）を付けて起動します。これにより`~/.openclaw/disable-launchagent`が設定され、アプリは既に実行中のGatewayにのみアタッチします。同じ動作はデバッグ設定で切り替えることもできます。

## リモートモード

リモートモードでは、ローカルのGatewayは起動されません。アプリはSSHトンネルを介してリモートホストに接続し、そのトンネルを経由して通信します。

## launchdを推奨する理由

-   ログイン時の自動起動。
-   組み込みの再起動/KeepAliveセマンティクス。
-   予測可能なログと監視。

真の子プロセスモードが再び必要になった場合、それは別個の、明示的な開発専用モードとして文書化されるべきです。

[Canvas](./canvas.md)[ヘルスチェック](./health.md)

---