title: "OpenClaw AI GatewayとCLIのアンインストール方法"
description: "OpenClaw AIを完全にアンインストールする方法を学びましょう。macOS、Linux、Windowsでの簡単なCLIメソッドまたは手動サービス削除のステップバイステップガイドに従ってください。"
keywords: ["openclaw アンインストール", "openclaw 削除", "gateway サービス削除", "openclaw cli アンインストール", "openclaw 削除", "openclaw クリーンアップ", "アンインストールガイド", "サービス削除"]
---

  メンテナンス

  
# アンインストール

2つの方法:

-   **簡単な方法** `openclaw` がまだインストールされている場合。
-   **手動サービス削除** CLIが削除されているがサービスがまだ実行されている場合。

## 簡単な方法 (CLIがインストール済み)

推奨: 組み込みのアンインストーラーを使用:

```bash
openclaw uninstall
```

非対話型 (自動化 / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

手動手順 (同じ結果):

1.  ゲートウェイサービスを停止:

```bash
openclaw gateway stop
```

2.  ゲートウェイサービスをアンインストール (launchd/systemd/schtasks):

```bash
openclaw gateway uninstall
```

3.  状態 + 設定を削除:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

`OPENCLAW_CONFIG_PATH` を状態ディレクトリ外のカスタムロケーションに設定した場合は、そのファイルも削除してください。

4.  ワークスペースを削除 (オプション、エージェントファイルを削除):

```bash
rm -rf ~/.openclaw/workspace
```

5.  CLIインストールを削除 (使用した方法を選択):

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6.  macOSアプリをインストールした場合:

```bash
rm -rf /Applications/OpenClaw.app
```

注意:

-   プロファイル (`--profile` / `OPENCLAW_PROFILE`) を使用した場合、各状態ディレクトリに対してステップ3を繰り返してください (デフォルトは `~/.openclaw-`)。
-   リモートモードでは、状態ディレクトリは**ゲートウェイホスト**上に存在するため、ステップ1-4もそこで実行してください。

## 手動サービス削除 (CLIがインストールされていない)

ゲートウェイサービスが実行され続けているが `openclaw` が見つからない場合に使用してください。

### macOS (launchd)

デフォルトのラベルは `ai.openclaw.gateway` (または `ai.openclaw.`; レガシーな `com.openclaw.*` がまだ存在する可能性あり):

```bash
launchctl bootout gui/$UID/ai.openclaw.gateway
rm -f ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

プロファイルを使用した場合は、ラベルとplist名を `ai.openclaw.` に置き換えてください。存在する場合は、レガシーな `com.openclaw.*` plistも削除してください。

### Linux (systemd ユニット)

デフォルトのユニット名は `openclaw-gateway.service` (または `openclaw-gateway-.service`):

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

### Windows (Scheduled Task)

デフォルトのタスク名は `OpenClaw Gateway` (または `OpenClaw Gateway ()`)。タスクスクリプトは状態ディレクトリの下にあります。

```bash
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

プロファイルを使用した場合は、一致するタスク名と `~\.openclaw-\gateway.cmd` を削除してください。

## 通常インストール vs ソースチェックアウト

### 通常インストール (install.sh / npm / pnpm / bun)

`https://openclaw.ai/install.sh` または `install.ps1` を使用した場合、CLIは `npm install -g openclaw@latest` でインストールされています。`npm rm -g openclaw` (またはその方法でインストールした場合は `pnpm remove -g` / `bun remove -g`) で削除してください。

### ソースチェックアウト (git clone)

リポジトリチェックアウト (`git clone` + `openclaw ...` / `bun run openclaw ...`) から実行している場合:

1.  リポジトリを削除する**前に**ゲートウェイサービスをアンインストールしてください (上記の簡単な方法または手動サービス削除を使用)。
2.  リポジトリディレクトリを削除。
3.  上記のように状態 + ワークスペースを削除。

[移行ガイド](./migrating.md)[VPSホスティング](../vps.md)

---