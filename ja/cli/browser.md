

  CLI コマンド

  
# browser

OpenClaw のブラウザ制御サーバーを管理し、ブラウザアクション（タブ、スナップショット、スクリーンショット、ナビゲーション、クリック、入力）を実行します。関連項目：

-   ブラウザツール + API: [ブラウザツール](../tools/browser.md)
-   Chrome 拡張機能リレー: [Chrome 拡張機能](../tools/chrome-extension.md)

## 共通フラグ

-   `--url `: ゲートウェイ WebSocket URL（デフォルトは設定ファイルから）。
-   `--token `: ゲートウェイトークン（必要な場合）。
-   `--timeout `: リクエストタイムアウト（ミリ秒）。
-   `--browser-profile `: ブラウザプロファイルを選択（デフォルトは設定ファイルから）。
-   `--json`: 機械可読な出力（サポートされている場合）。

## クイックスタート（ローカル）

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## プロファイル

プロファイルは、名前付きのブラウザルーティング設定です。実際には：

-   `openclaw`: 専用の OpenClaw 管理 Chrome インスタンス（分離されたユーザーデータディレクトリ）を起動/接続します。
-   `chrome`: Chrome 拡張機能リレーを介して、既存の Chrome タブを制御します。

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

特定のプロファイルを使用する：

```bash
openclaw browser --browser-profile work tabs
```

## タブ

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## スナップショット / スクリーンショット / アクション

スナップショット：

```bash
openclaw browser snapshot
```

スクリーンショット：

```bash
openclaw browser screenshot
```

ナビゲート/クリック/入力（ref ベースの UI 自動化）：

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## Chrome 拡張機能リレー（ツールバーボタン経由で接続）

このモードでは、エージェントが手動で接続した既存の Chrome タブを制御できます（自動接続はしません）。アンパックされた拡張機能を安定したパスにインストールします：

```bash
openclaw browser extension install
openclaw browser extension path
```

その後、Chrome → `chrome://extensions` → 「デベロッパーモード」を有効化 → 「パッケージ化されていない拡張機能を読み込む」 → 表示されたフォルダを選択。詳細ガイド：[Chrome 拡張機能](../tools/chrome-extension.md)

## リモートブラウザ制御（ノードホストプロキシ）

ゲートウェイがブラウザとは異なるマシンで実行されている場合、Chrome/Brave/Edge/Chromium がインストールされているマシン上で **ノードホスト** を実行します。ゲートウェイはブラウザアクションをそのノードにプロキシします（別途ブラウザ制御サーバーは不要）。`gateway.nodes.browser.mode` を使用して自動ルーティングを制御し、`gateway.nodes.browser.node` を使用して複数接続されている場合に特定のノードを固定できます。セキュリティとリモート設定：[ブラウザツール](../tools/browser.md)、[リモートアクセス](../gateway/remote.md)、[Tailscale](../gateway/tailscale.md)、[セキュリティ](../gateway/security.md)

[approvals](./approvals.md)[channels](./channels.md)