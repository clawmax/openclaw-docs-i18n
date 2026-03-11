

  設定と運用

  
# 複数ゲートウェイ

単一のゲートウェイで複数のメッセージング接続とエージェントを処理できるため、ほとんどのセットアップでは1つのゲートウェイを使用すべきです。より強力な分離や冗長性（例：レスキューボット）が必要な場合は、分離されたプロファイル/ポートで別々のゲートウェイを実行してください。

## 分離チェックリスト（必須）

-   `OPENCLAW_CONFIG_PATH` — インスタンスごとの設定ファイル
-   `OPENCLAW_STATE_DIR` — インスタンスごとのセッション、認証情報、キャッシュ
-   `agents.defaults.workspace` — インスタンスごとのワークスペースルート
-   `gateway.port` (または `--port`) — インスタンスごとに一意
-   派生ポート（ブラウザ/キャンバス）が重複しないこと

これらが共有されている場合、設定競合やポート競合が発生します。

## 推奨: プロファイル (--profile)

プロファイルは `OPENCLAW_STATE_DIR` + `OPENCLAW_CONFIG_PATH` を自動的にスコープし、サービス名にサフィックスを付けます。

```bash
# メイン
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# レスキュー
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

プロファイルごとのサービス:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

## レスキューボットガイド

同じホスト上で、独自の以下を持つ2番目のゲートウェイを実行します:

-   プロファイル/設定
-   ステートディレクトリ
-   ワークスペース
-   ベースポート（および派生ポート）

これにより、レスキューボットはメインボットから分離され、プライマリボットがダウンしている場合にデバッグや設定変更を適用できます。ポート間隔: 派生するブラウザ/キャンバス/CDPポートが決して衝突しないように、ベースポート間に少なくとも20ポートの間隔を空けてください。

### インストール方法（レスキューボット）

```bash
# メインボット（既存または新規、--profileパラメータなし）
# ポート18789 + Chrome CDC/Canvas/... ポートで実行
openclaw onboard
openclaw gateway install

# レスキューボット（分離プロファイル + ポート）
openclaw --profile rescue onboard
# 注意:
# - ワークスペース名はデフォルトで -rescue が付加されます
# - ポートは少なくとも 18789 + 20 ポートにする必要があります。
#   完全に異なるベースポート（例: 19789）を選択することをお勧めします。
# - オンボーディングの残りの部分は通常と同じです

# サービスをインストールする場合（オンボーディング中に自動的に行われなかった場合）
openclaw --profile rescue gateway install
```

## ポートマッピング（派生）

ベースポート = `gateway.port` (または `OPENCLAW_GATEWAY_PORT` / `--port`)。

-   ブラウザ制御サービスポート = ベース + 2 (ループバックのみ)
-   キャンバスホストはゲートウェイHTTPサーバー上で提供されます (`gateway.port` と同じポート)
-   ブラウザプロファイルCDPポートは `browser.controlPort + 9 .. + 108` から自動割り当てされます

設定や環境変数でこれらのいずれかを上書きする場合は、インスタンスごとに一意に保つ必要があります。

## ブラウザ/CDPに関する注意（よくある落とし穴）

-   複数のインスタンスで `browser.cdpUrl` を同じ値に固定**しないでください**。
-   各インスタンスは独自のブラウザ制御ポートとCDP範囲（そのゲートウェイポートから派生）が必要です。
-   明示的なCDPポートが必要な場合は、インスタンスごとに `browser.profiles..cdpPort` を設定してください。
-   リモートChrome: `browser.profiles..cdpUrl` を使用してください（プロファイルごと、インスタンスごと）。

## 手動環境変数の例

```
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```

## クイックチェック

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```

[バックグラウンド実行とプロセスツール](./background-process.md)[トラブルシューティング](./troubleshooting.md)