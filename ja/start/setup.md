title: "OpenClaw 開発者セットアップガイド：安定版および最先端ワークフロー"
description: "OpenClaw の開発環境構築方法を学びます。安定版 macOS アプリと最先端 Gateway ワークフローのための、設定や認証情報の保存を含むステップバイステップの手順に従います。"
keywords: ["openclaw セットアップ", "開発者セットアップ", "gateway 設定", "macos アプリ", "カスタマイズ戦略", "認証情報保存", "最先端ワークフロー", "pnpm gateway"]
---

  開発者セットアップ

  
# セットアップ

> **ℹ️** 初めてセットアップする場合は、[はじめに](./getting-started.md)から始めてください。ウィザードの詳細については、[オンボーディングウィザード](./wizard.md)を参照してください。

 最終更新日: 2026-01-01

## 概要

-   **カスタマイズはリポジトリ外に保存:** `~/.openclaw/workspace` (ワークスペース) + `~/.openclaw/openclaw.json` (設定)。
-   **安定版ワークフロー:** macOS アプリをインストールし、バンドルされた Gateway を実行させます。
-   **最先端ワークフロー:** `pnpm gateway:watch` で自分で Gateway を実行し、macOS アプリをローカルモードで接続させます。

## 前提条件 (ソースから)

-   Node `>=22`
-   `pnpm`
-   Docker (オプション; コンテナ化セットアップ/e2e のみ — [Docker](../install/docker.md) を参照)

## カスタマイズ戦略 (更新で問題が起きないように)

「100% 自分用にカスタマイズ」*かつ* 簡単な更新を望む場合は、カスタマイズを以下に保存してください:

-   **設定:** `~/.openclaw/openclaw.json` (JSON/JSON5 風)
-   **ワークスペース:** `~/.openclaw/workspace` (スキル、プロンプト、メモリ; プライベート git リポジトリにします)

一度だけブートストラップを実行:

```bash
openclaw setup
```

このリポジトリ内からは、ローカル CLI エントリを使用:

```bash
openclaw setup
```

グローバルインストールがまだない場合は、`pnpm openclaw setup` 経由で実行してください。

## このリポジトリから Gateway を実行

`pnpm build` の後、パッケージ化された CLI を直接実行できます:

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## 安定版ワークフロー (macOS アプリ優先)

1.  **OpenClaw.app** をインストール + 起動 (メニューバー)。
2.  オンボーディング/権限チェックリストを完了 (TCC プロンプト)。
3.  Gateway が **ローカル** で実行中であることを確認 (アプリが管理します)。
4.  サーフェスをリンク (例: WhatsApp):

```bash
openclaw channels login
```

5.  動作確認:

```bash
openclaw health
```

ビルドでオンボーディングが利用できない場合:

-   `openclaw setup` を実行し、次に `openclaw channels login`、その後 Gateway を手動で起動 (`openclaw gateway`)。

## 最先端ワークフロー (ターミナル内の Gateway)

目標: TypeScript Gateway で作業し、ホットリロードを有効にし、macOS アプリ UI を接続したままにする。

### 0) (オプション) ソースから macOS アプリも実行

macOS アプリも最先端で使用したい場合:

```
./scripts/restart-mac.sh
```

### 1) 開発用 Gateway を起動

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` は Gateway をウォッチモードで実行し、TypeScript の変更時にリロードします。

### 2) macOS アプリを実行中の Gateway に向ける

**OpenClaw.app** 内で:

-   接続モード: **ローカル** アプリは設定されたポートで実行中の Gateway に接続します。

### 3) 確認

-   アプリ内の Gateway ステータスは **「既存の gateway を使用中 …」** と表示されるはずです。
-   または CLI 経由で:

```bash
openclaw health
```

### よくある落とし穴

-   **ポートが間違っている:** Gateway WS のデフォルトは `ws://127.0.0.1:18789`; アプリと CLI で同じポートを使用してください。
-   **状態の保存場所:**
    -   認証情報: `~/.openclaw/credentials/`
    -   セッション: `~/.openclaw/agents//sessions/`
    -   ログ: `/tmp/openclaw/`

## 認証情報保存マップ

認証のデバッグやバックアップ対象を決定する際に使用してください:

-   **WhatsApp**: `~/.openclaw/credentials/whatsapp//creds.json`
-   **Telegram ボットトークン**: 設定/環境変数 または `channels.telegram.tokenFile`
-   **Discord ボットトークン**: 設定/環境変数 または SecretRef (env/file/exec プロバイダ)
-   **Slack トークン**: 設定/環境変数 (`channels.slack.*`)
-   **ペアリング許可リスト**:
    -   `~/.openclaw/credentials/-allowFrom.json` (デフォルトアカウント)
    -   `~/.openclaw/credentials/--allowFrom.json` (非デフォルトアカウント)
-   **モデル認証プロファイル**: `~/.openclaw/agents//agent/auth-profiles.json`
-   **ファイルベースのシークレットペイロード (オプション)**: `~/.openclaw/secrets.json`
-   **レガシー OAuth インポート**: `~/.openclaw/credentials/oauth.json` 詳細: [セキュリティ](../gateway/security.md#credential-storage-map)。

## 更新 (セットアップを壊さずに)

-   `~/.openclaw/workspace` と `~/.openclaw/` を「あなたのもの」として保持し、個人用のプロンプトや設定を `openclaw` リポジトリに入れないでください。
-   ソースの更新: `git pull` + `pnpm install` (ロックファイルが変更された場合) + `pnpm gateway:watch` の使用を継続。

## Linux (systemd ユーザーサービス)

Linux インストールでは systemd **ユーザー** サービスを使用します。デフォルトでは、systemd はログアウト/アイドル時にユーザーサービスを停止し、Gateway を終了させます。オンボーディングはあなたのために lingering を有効にしようとします (sudo を要求する場合があります)。それでもオフの場合は、以下を実行:

```bash
sudo loginctl enable-linger $USER
```

常時稼働またはマルチユーザーサーバーの場合は、ユーザーサービスの代わりに **システム** サービスを検討してください (lingering は不要です)。systemd に関する注意事項は [Gateway 運用マニュアル](../gateway.md) を参照してください。

## 関連ドキュメント

-   [Gateway 運用マニュアル](../gateway.md) (フラグ、監視、ポート)
-   [Gateway 設定](../gateway/configuration.md) (設定スキーマ + 例)
-   [Discord](../channels/discord.md) と [Telegram](../channels/telegram.md) (返信タグ + replyToMode 設定)
-   [OpenClaw アシスタントセットアップ](./openclaw.md)
-   [macOS アプリ](../platforms/macos.md) (gateway ライフサイクル)

[セッション管理詳細解説](../reference/session-management-compaction.md)[Pi 開発ワークフロー](../pi-dev.md)

---