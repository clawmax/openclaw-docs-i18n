

  ブラウザ

  
# ブラウザのトラブルシューティング

## 問題: 「Failed to start Chrome CDP on port 18800」

OpenClawのブラウザ制御サーバーがChrome/Brave/Edge/Chromiumの起動に失敗し、以下のエラーが発生します:

```json
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```

### 根本原因

Ubuntu（および多くのLinuxディストリビューション）では、デフォルトのChromiumインストールは**snapパッケージ**です。SnapのAppArmor制限が、OpenClawがブラウザプロセスを生成・監視する方法と干渉します。`apt install chromium`コマンドは、snapにリダイレクトするスタブパッケージをインストールします:

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

これは実際のブラウザではなく、単なるラッパーです。

### 解決策 1: Google Chromeをインストールする（推奨）

snapでサンドボックス化されていない公式のGoogle Chrome `.deb`パッケージをインストールします:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # 依存関係エラーがある場合
```

その後、OpenClawの設定（`~/.openclaw/openclaw.json`）を更新します:

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### 解決策 2: アタッチ専用モードでSnap Chromiumを使用する

snap Chromiumを使用する必要がある場合は、OpenClawを手動で起動したブラウザにアタッチするように設定します:

1.  設定を更新:

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2.  Chromiumを手動で起動:

```
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3.  オプションで、Chromeを自動起動するsystemdユーザーサービスを作成:

```bash
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

有効化: `systemctl --user enable --now openclaw-browser.service`

### ブラウザの動作確認

ステータスを確認:

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

ブラウジングをテスト:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

### 設定リファレンス

| オプション | 説明 | デフォルト |
| --- | --- | --- |
| `browser.enabled` | ブラウザ制御を有効にする | `true` |
| `browser.executablePath` | Chromiumベースのブラウザバイナリ（Chrome/Brave/Edge/Chromium）へのパス | 自動検出（Chromiumベースのデフォルトブラウザを優先） |
| `browser.headless` | GUIなしで実行 | `false` |
| `browser.noSandbox` | `--no-sandbox`フラグを追加（一部のLinux環境で必要） | `false` |
| `browser.attachOnly` | ブラウザを起動せず、既存のものにのみアタッチ | `false` |
| `browser.cdpPort` | Chrome DevTools Protocolポート | `18800` |

### 問題: 「Chrome extension relay is running, but no tab is connected」

`chrome`プロファイル（拡張機能リレー）を使用しています。これはOpenClawブラウザ拡張機能がライブタブに接続されていることを期待しています。修正オプション:

1.  **管理対象ブラウザを使用する:** `openclaw browser start --browser-profile openclaw`（または`browser.defaultProfile: "openclaw"`を設定）。
2.  **拡張機能リレーを使用する:** 拡張機能をインストールし、タブを開き、OpenClaw拡張機能アイコンをクリックして接続します。

注意:

-   `chrome`プロファイルは可能な場合、**システムのデフォルトChromiumブラウザ**を使用します。
-   ローカルの`openclaw`プロファイルは自動的に`cdpPort`/`cdpUrl`を割り当てます。リモートCDPの場合のみこれらを設定してください。

[Chrome拡張機能](./chrome-extension.md)[エージェント送信](./agent-send.md)