title: "macOS OpenClaw Gateway セットアップと launchd サービス管理"
description: "openclaw CLI のインストール方法と、OpenClaw コンパニオンアプリ用に Gateway を macOS launchd サービスとして設定する方法を学びます。バージョンチェックとトラブルシューティングを含みます。"
keywords: ["macos openclaw", "gateway セットアップ", "launchd サービス", "openclaw cli", "macos コンパニオンアプリ", "ローカル gateway", "node gateway", "サービス管理"]
---

  macOS コンパニオンアプリ

  
# macOS 上の Gateway

OpenClaw.app は Node/Bun や Gateway ランタイムを同梱しなくなりました。macOS アプリは **外部の** `openclaw` CLI インストールを想定し、Gateway を子プロセスとして起動せず、Gateway を実行し続けるためにユーザーごとの launchd サービスを管理します（または、すでにローカル Gateway が実行されている場合はそれにアタッチします）。

## CLI のインストール (ローカルモードに必須)

Mac に Node 22+ が必要です。その後、グローバルに `openclaw` をインストールします:

```bash
npm install -g openclaw@<version>
```

macOS アプリの **CLI をインストール** ボタンは、npm/pnpm 経由で同じフローを実行します（Gateway ランタイムには bun は推奨されません）。

## Launchd (LaunchAgent としての Gateway)

ラベル:

-   `ai.openclaw.gateway` (または `ai.openclaw.`; レガシーな `com.openclaw.*` が残っている場合があります)

Plist の場所 (ユーザーごと):

-   `~/Library/LaunchAgents/ai.openclaw.gateway.plist` (または `~/Library/LaunchAgents/ai.openclaw..plist`)

マネージャー:

-   macOS アプリがローカルモードでの LaunchAgent のインストール/更新を所有します。
-   CLI でもインストールできます: `openclaw gateway install`。

動作:

-   「OpenClaw アクティブ」は LaunchAgent を有効/無効にします。
-   アプリを終了しても gateway は停止**しません** (launchd が生存を維持します)。
-   設定されたポートですでに Gateway が実行されている場合、アプリは新しいものを起動する代わりにそれにアタッチします。

ロギング:

-   launchd stdout/err: `/tmp/openclaw/openclaw-gateway.log`

## バージョン互換性

macOS アプリは、gateway のバージョンを自身のバージョンと照合します。互換性がない場合は、グローバル CLI をアプリのバージョンに合わせて更新してください。

## スモークチェック

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

その後:

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```

[macOS リリース](./release.md)[macOS IPC](./xpc.md)