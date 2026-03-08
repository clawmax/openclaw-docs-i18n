

  メンテナンス

  
# 更新

OpenClaw は急速に進化しています（「1.0」以前）。更新はインフラのデプロイのように扱ってください：更新 → チェック実行 → 再起動（または `openclaw update` を使用。これは再起動も行います）→ 検証。

## 推奨: ウェブサイトインストーラーを再実行する（インプレースアップグレード）

**推奨される** 更新パスは、ウェブサイトからインストーラーを再実行することです。既存のインストールを検出し、インプレースでアップグレードし、必要に応じて `openclaw doctor` を実行します。

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

注意点:

-   オンボーディングウィザードを再実行したくない場合は、`--no-onboard` を追加してください。
-   **ソースインストール** の場合は、以下を使用します:
    
    コピー
    
    ```bash
    curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --no-onboard
    ```
    
    インストーラーは、リポジトリがクリーンな場合 **のみ** `git pull --rebase` を実行します。
-   **グローバルインストール** の場合、スクリプトは内部で `npm install -g openclaw@latest` を使用します。
-   レガシーノート: `clawdbot` は互換性シムとして引き続き利用可能です。

## 更新前に

-   インストール方法を把握する: **グローバル** (npm/pnpm) か **ソースから** (git clone) か。
-   Gateway の実行方法を把握する: **フォアグラウンドターミナル** か **監視サービス** (launchd/systemd) か。
-   カスタマイズ内容のスナップショットを取る:
    -   設定: `~/.openclaw/openclaw.json`
    -   認証情報: `~/.openclaw/credentials/`
    -   ワークスペース: `~/.openclaw/workspace`

## 更新（グローバルインストール）

グローバルインストール（いずれか一つを選択）:

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

Gateway ランタイムには Bun は **推奨しません**（WhatsApp/Telegram のバグ）。更新チャネルを切り替えるには（git + npm インストール）:

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

特定のタグ/バージョンを一度だけインストールするには `--tag <dist-tag|version>` を使用します。チャネルの意味とリリースノートについては [開発チャネル](./development-channels.md) を参照してください。注意: npm インストールでは、Gateway は起動時に更新ヒントをログに記録します（現在のチャネルタグをチェック）。`update.checkOnStart: false` で無効化できます。

### コア自動アップデーター（オプション）

自動アップデーターは **デフォルトでオフ** であり、コア Gateway 機能です（プラグインではありません）。

```json
{
  "update": {
    "channel": "stable",
    "auto": {
      "enabled": true,
      "stableDelayHours": 6,
      "stableJitterHours": 12,
      "betaCheckIntervalHours": 1
    }
  }
}
```

動作:

-   `stable`: 新しいバージョンが検出されると、OpenClaw は `stableDelayHours` 待機し、その後 `stableJitterHours` 内でインストールごとに決定論的なジッターを適用します（ロールアウトを分散）。
-   `beta`: `betaCheckIntervalHours` の間隔（デフォルト: 1時間ごと）でチェックし、更新が利用可能なときに適用します。
-   `dev`: 自動適用なし；手動で `openclaw update` を使用します。

自動化を有効にする前に、`openclaw update --dry-run` を使用して更新アクションをプレビューします。その後:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

注意点:

-   Gateway がサービスとして実行されている場合、PID を kill するよりも `openclaw gateway restart` が推奨されます。
-   特定のバージョンに固定している場合は、以下の「ロールバック / 固定」を参照してください。

## 更新（openclaw update）

**ソースインストール** (git checkout) の場合は、以下を推奨します:

```bash
openclaw update
```

これは安全な更新フローを実行します:

-   クリーンなワークツリーを必要とします。
-   選択されたチャネル（タグまたはブランチ）に切り替えます。
-   設定されたアップストリームに対してフェッチ + リベースを実行します（dev チャネル）。
-   依存関係をインストールし、ビルドし、Control UI をビルドし、`openclaw doctor` を実行します。
-   デフォルトで Gateway を再起動します（`--no-restart` でスキップ可能）。

**npm/pnpm** 経由でインストールした場合（git メタデータなし）、`openclaw update` はパッケージマネージャー経由での更新を試みます。インストールを検出できない場合は、代わりに「更新（グローバルインストール）」を使用してください。

## 更新（Control UI / RPC）

Control UI には **更新 & 再起動** (RPC: `update.run`) があります。これは以下を行います:

1.  `openclaw update` と同じソース更新フローを実行します（git checkout のみ）。
2.  構造化されたレポート（stdout/stderr の末尾）を含む再起動センチネルを書き込みます。
3.  Gateway を再起動し、最後のアクティブセッションにレポートを ping します。

リベースが失敗した場合、Gateway は更新を適用せずに中止し、再起動します。

## 更新（ソースから）

リポジトリのチェックアウトから: 推奨:

```bash
openclaw update
```

手動（ほぼ同等）:

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # 初回実行時に UI の依存関係を自動インストール
openclaw doctor
openclaw health
```

注意点:

-   `pnpm build` は、パッケージ化された `openclaw` バイナリ ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) を実行する場合や、Node で `dist/` を実行する場合に重要です。
-   グローバルインストールなしでリポジトリのチェックアウトから実行する場合、CLI コマンドには `pnpm openclaw ...` を使用してください。
-   TypeScript から直接実行する場合 (`pnpm openclaw ...`)、リビルドは通常不要ですが、**設定マイグレーションは依然として適用されます** → doctor を実行してください。
-   グローバルインストールと git インストールの切り替えは簡単です: もう一方の方法でインストールし、`openclaw doctor` を実行して Gateway サービスのエントリーポイントが現在のインストールに書き換えられるようにします。

## 常に実行: openclaw doctor

Doctor は「安全な更新」コマンドです。意図的に地味です: 修復 + マイグレーション + 警告。注意: **ソースインストール** (git checkout) の場合、`openclaw doctor` は最初に `openclaw update` を実行するよう提案します。典型的な動作:

-   非推奨の設定キー / レガシー設定ファイルの場所をマイグレート。
-   DM ポリシーを監査し、リスクの高い「オープン」設定について警告。
-   Gateway の健全性をチェックし、再起動を提案できます。
-   古い Gateway サービス（launchd/systemd; レガシー schtasks）を検出し、現在の OpenClaw サービスにマイグレート。
-   Linux では、systemd ユーザーリンガリングを確実にします（Gateway がログアウト後も生存するため）。

詳細: [Doctor](../gateway/doctor.md)

## Gateway の起動 / 停止 / 再起動

CLI（OS に関係なく動作）:

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

監視サービスを使用している場合:

-   macOS launchd (アプリバンドル LaunchAgent): `launchctl kickstart -k gui/$UID/ai.openclaw.gateway` (`ai.openclaw.` を使用; レガシー `com.openclaw.*` も動作)
-   Linux systemd ユーザーサービス: `systemctl --user restart openclaw-gateway[-].service`
-   Windows (WSL2): `systemctl --user restart openclaw-gateway[-].service`
    -   `launchctl`/`systemctl` はサービスがインストールされている場合のみ動作します；それ以外の場合は `openclaw gateway install` を実行してください。

ランブック + 正確なサービスラベル: [Gateway ランブック](../gateway.md)

## ロールバック / 固定（問題が発生した場合）

### 固定（グローバルインストール）

動作が確認されているバージョンをインストールします（`` を最後に動作したバージョンに置き換えてください）:

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

ヒント: 現在公開されているバージョンを確認するには、`npm view openclaw version` を実行します。その後、再起動 + doctor を再実行:

```bash
openclaw doctor
openclaw gateway restart
```

### 日付による固定（ソース）

日付からコミットを選択します（例: 「2026-01-01 時点の main の状態」）:

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

その後、依存関係を再インストール + 再起動:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

後で最新に戻したい場合:

```bash
git checkout main
git pull
```

## 行き詰まったら

-   `openclaw doctor` を再度実行し、出力を注意深く読んでください（修正方法を教えてくれることが多いです）。
-   確認: [トラブルシューティング](../gateway/troubleshooting.md)
-   Discord で質問: [https://discord.gg/clawd](https://discord.gg/clawd)

[Bun (実験的)](./bun.md)[移行ガイド](./migrating.md)