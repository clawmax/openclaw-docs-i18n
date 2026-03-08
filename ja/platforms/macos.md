title: "OpenClaw macOSアプリガイド：メニューバーコンパニオンとゲートウェイ"
description: "OpenClaw macOSメニューバーアプリを使用して、権限を管理し、ゲートウェイに接続し、Canvasや画面録画などのmacOS機能をAIエージェントに公開する方法を学びます。"
keywords: ["openclaw macos", "macos メニューバーアプリ", "ゲートウェイ接続", "macos 権限 tcc", "system.run 承認", "リモートモード ssh", "launchd 制御", "キャンバス自動化"]
---

  プラットフォーム概要

  
# macOSアプリ

macOSアプリは、OpenClawの**メニューバーコンパニオン**です。権限を所有し、ローカルでゲートウェイを管理/接続し（launchdまたは手動）、macOSの機能をノードとしてエージェントに公開します。

## 機能

-   ネイティブ通知とステータスをメニューバーに表示します。
-   TCCプロンプト（通知、アクセシビリティ、画面録画、マイク、音声認識、自動化/AppleScript）を所有します。
-   ゲートウェイ（ローカルまたはリモート）を実行または接続します。
-   macOS専用ツール（Canvas、カメラ、画面録画、`system.run`）を公開します。
-   **リモート**モード（launchd）でローカルノードホストサービスを開始し、**ローカル**モードでは停止します。
-   オプションでUI自動化のための**PeekabooBridge**をホストします。
-   グローバルCLI（`openclaw`）を要求に応じてnpm/pnpm経由でインストールします（ゲートウェイランタイムにはbunは推奨されません）。

## ローカルモード vs リモートモード

-   **ローカル**（デフォルト）：アプリは、実行中のローカルゲートウェイが存在する場合にそれに接続します。存在しない場合は、`openclaw gateway install`を介してlaunchdサービスを有効にします。
-   **リモート**：アプリはSSH/Tailscale経由でゲートウェイに接続し、ローカルプロセスを起動することはありません。アプリはローカルの**ノードホストサービス**を開始し、リモートゲートウェイがこのMacに到達できるようにします。アプリはゲートウェイを子プロセスとして生成しません。

## Launchd制御

アプリは、ユーザーごとのLaunchAgent（ラベル`ai.openclaw.gateway`、または`--profile`/`OPENCLAW_PROFILE`使用時は`ai.openclaw.`）を管理します（レガシーな`com.openclaw.*`はアンロードされます）。

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

名前付きプロファイルを実行する場合は、ラベルを`ai.openclaw.`に置き換えてください。LaunchAgentがインストールされていない場合は、アプリから有効にするか、`openclaw gateway install`を実行してください。

## ノード機能（mac）

macOSアプリは自身をノードとして提示します。一般的なコマンド：

-   Canvas: `canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
-   カメラ: `camera.snap`, `camera.clip`
-   画面: `screen.record`
-   システム: `system.run`, `system.notify`

ノードは`permissions`マップを報告するため、エージェントは許可される内容を決定できます。ノードサービス + アプリIPC：

-   ヘッドレスノードホストサービスが実行中（リモートモード）の場合、ノードとしてゲートウェイWSに接続します。
-   `system.run`は、ローカルUnixソケットを介してmacOSアプリ（UI/TCCコンテキスト）内で実行されます。プロンプトと出力はアプリ内に留まります。

図（SCI）：

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + TCC + system.run)
```

## 実行承認（system.run）

`system.run`は、macOSアプリの**実行承認**（設定 → 実行承認）によって制御されます。セキュリティ、確認、許可リストはMac上にローカルで保存されます：

```
~/.openclaw/exec-approvals.json
```

例：

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [{ "pattern": "/opt/homebrew/bin/rg" }]
    }
  }
}
```

注意点：

-   `allowlist`エントリは、解決されたバイナリパスに対するグロブパターンです。
-   シェル制御または展開構文（`&&`、`||`、`;`、`|`、```、`$`、`<`、`>`、`(`、`)`）を含む生のシェルコマンドテキストは、許可リストの不一致として扱われ、明示的な承認（またはシェルバイナリの許可リスト登録）が必要です。
-   プロンプトで「常に許可」を選択すると、そのコマンドが許可リストに追加されます。
-   `system.run`の環境変数オーバーライドはフィルタリングされ（`PATH`、`DYLD_*`、`LD_*`、`NODE_OPTIONS`、`PYTHON*`、`PERL*`、`RUBYOPT`、`SHELLOPTS`、`PS4`が削除されます）、その後アプリの環境とマージされます。
-   シェルラッパー（`bash|sh|zsh ... -c/-lc`）の場合、リクエストスコープの環境変数オーバーライドは、小さな明示的な許可リスト（`TERM`、`LANG`、`LC_*`、`COLORTERM`、`NO_COLOR`、`FORCE_COLOR`）に削減されます。
-   許可リストモードでの常に許可の決定において、既知のディスパッチラッパー（`env`、`nice`、`nohup`、`stdbuf`、`timeout`）は、ラッパーパスではなく内部実行可能ファイルのパスを保持します。安全にアンラップできない場合、許可リストエントリは自動的には保持されません。

## ディープリンク

アプリはローカルアクション用に`openclaw://` URLスキームを登録します。

### openclaw://agent

ゲートウェイの`agent`リクエストをトリガーします。

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

クエリパラメータ：

-   `message`（必須）
-   `sessionKey`（オプション）
-   `thinking`（オプション）
-   `deliver` / `to` / `channel`（オプション）
-   `timeoutSeconds`（オプション）
-   `key`（オプション、無人モードキー）

安全性：

-   `key`なしの場合、アプリは確認を求めます。
-   `key`なしの場合、アプリは確認プロンプト用に短いメッセージ制限を適用し、`deliver` / `to` / `channel`を無視します。
-   有効な`key`がある場合、実行は無人モードになります（個人の自動化を想定）。

## オンボーディングフロー（典型的）

1.  **OpenClaw.app**をインストールして起動します。
2.  権限チェックリスト（TCCプロンプト）を完了します。
3.  **ローカル**モードがアクティブで、ゲートウェイが実行中であることを確認します。
4.  ターミナルアクセスが必要な場合はCLIをインストールします。

## 状態ディレクトリの配置（macOS）

OpenClawの状態ディレクトリをiCloudや他のクラウド同期フォルダに置かないでください。同期されたパスは遅延を引き起こし、セッションや認証情報のファイルロック/同期競合を時々引き起こす可能性があります。以下のようなローカルで非同期の状態パスを推奨します：

```
OPENCLAW_STATE_DIR=~/.openclaw
```

`openclaw doctor`が以下の下に状態を検出した場合：

-   `~/Library/Mobile Documents/com~apple~CloudDocs/...`
-   `~/Library/CloudStorage/...`

警告を表示し、ローカルパスに戻すことを推奨します。

## ビルド＆開発ワークフロー（ネイティブ）

-   `cd apps/macos && swift build`
-   `swift run OpenClaw`（またはXcode）
-   アプリのパッケージ化：`scripts/package-mac-app.sh`

## ゲートウェイ接続のデバッグ（macOS CLI）

デバッグCLIを使用して、アプリを起動せずに、macOSアプリが使用するのと同じゲートウェイWebSocketハンドシェイクおよびディスカバリロジックを実行します。

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

接続オプション：

-   `--url <ws://host:port>`：設定を上書き
-   `--mode <local|remote>`：設定から解決（デフォルト：設定またはローカル）
-   `--probe`：新規のヘルスプローブを強制
-   `--timeout <ms>`：リクエストタイムアウト（デフォルト：`15000`）
-   `--json`：差分用の構造化出力

ディスカバリオプション：

-   `--include-local`：「ローカル」としてフィルタリングされるゲートウェイを含める
-   `--timeout <ms>`：全体のディスカバリウィンドウ（デフォルト：`2000`）
-   `--json`：差分用の構造化出力

ヒント：`openclaw gateway discover --json`と比較して、macOSアプリのディスカバリパイプライン（NWBrowser + tailnet DNS-SDフォールバック）がNode CLIの`dns-sd`ベースのディスカバリと異なるかどうかを確認してください。

## リモート接続の仕組み（SSHトンネル）

macOSアプリが**リモート**モードで実行されるとき、ローカルUIコンポーネントがリモートゲートウェイと通信できるように、あたかもlocalhost上にあるかのようにSSHトンネルを開きます。

### コントロールトンネル（ゲートウェイWebSocketポート）

-   **目的：** ヘルスチェック、ステータス、Webチャット、設定、およびその他のコントロールプレーン呼び出し。
-   **ローカルポート：** ゲートウェイポート（デフォルト`18789`）、常に安定。
-   **リモートポート：** リモートホスト上の同じゲートウェイポート。
-   **動作：** ランダムなローカルポートは使用せず、アプリは既存の正常なトンネルを再利用するか、必要に応じて再起動します。
-   **SSH形状：** `ssh -N -L <local>:127.0.0.1:<remote>`（BatchMode + ExitOnForwardFailure + keepaliveオプション付き）。
-   **IP報告：** SSHトンネルはループバックを使用するため、ゲートウェイはノードIPを`127.0.0.1`として認識します。実際のクライアントIPを表示したい場合は、**Direct (ws/wss)** トランスポートを使用してください（[macOSリモートアクセス](/platforms/mac/remote)を参照）。

セットアップ手順については、[macOSリモートアクセス](/platforms/mac/remote)を参照してください。プロトコルの詳細については、[ゲートウェイプロトコル](/gateway/protocol)を参照してください。

## 関連ドキュメント

-   [ゲートウェイ運用マニュアル](/gateway)
-   [ゲートウェイ（macOS）](/platforms/mac/bundled-gateway)
-   [macOS権限](/platforms/mac/permissions)
-   [Canvas](/platforms/mac/canvas)

[プラットフォーム](/platforms)[Linuxアプリ](/platforms/linux)