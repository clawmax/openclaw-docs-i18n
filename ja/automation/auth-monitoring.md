

  自動化

  
# 認証監視

OpenClawは、OAuthの有効期限の健全性を `openclaw models status` で公開しています。これを自動化とアラートに使用してください。スクリプトは、電話ワークフロー向けのオプションの追加機能です。

## 推奨: CLIチェック (ポータブル)

```bash
openclaw models status --check
```

終了コード:

-   `0`: OK
-   `1`: 認証情報が期限切れまたは欠落
-   `2`: まもなく期限切れ (24時間以内)

これはcron/systemdで動作し、追加のスクリプトを必要としません。

## オプションスクリプト (運用 / 電話ワークフロー)

これらは `scripts/` ディレクトリにあり、**オプション**です。ゲートウェイホストへのSSHアクセスを想定しており、systemd + Termux向けに調整されています。

-   `scripts/claude-auth-status.sh` は、信頼できる情報源として `openclaw models status --json` を使用するようになりました (CLIが利用できない場合は直接ファイル読み取りにフォールバック)。タイマー用に `openclaw` を `PATH` に保持してください。
-   `scripts/auth-monitor.sh`: cron/systemdタイマーのターゲット。アラート (ntfyまたは電話) を送信します。
-   `scripts/systemd/openclaw-auth-monitor.{service,timer}`: systemdユーザータイマー。
-   `scripts/claude-auth-status.sh`: Claude Code + OpenClaw認証チェッカー (full/json/simple)。
-   `scripts/mobile-reauth.sh`: SSH経由のガイド付き再認証フロー。
-   `scripts/termux-quick-auth.sh`: ワンタップウィジェットステータス + 認証URLを開く。
-   `scripts/termux-auth-widget.sh`: 完全なガイド付きウィジェットフロー。
-   `scripts/termux-sync-widget.sh`: Claude Code認証情報 → OpenClawへの同期。

電話の自動化やsystemdタイマーが必要ない場合は、これらのスクリプトはスキップしてください。

[ポーリング](./poll.md)[ノード](../nodes.md)