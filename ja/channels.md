

  概要

  
# チャットチャネル

OpenClaw は、あなたが普段使っているあらゆるチャットアプリで会話できます。各チャネルは Gateway を介して接続されます。テキストはすべてのチャネルでサポートされています。メディアやリアクションのサポートはチャネルによって異なります。

## サポートされているチャネル

-   [BlueBubbles](./channels/bluebubbles.md) — **iMessage に推奨**。BlueBubbles macOS サーバーの REST API を使用し、フル機能をサポート（編集、送信取り消し、エフェクト、リアクション、グループ管理 — 編集機能は現在 macOS 26 Tahoe で不具合あり）。
-   [Discord](./channels/discord.md) — Discord Bot API + Gateway。サーバー、チャンネル、DM をサポート。
-   [Feishu](./channels/feishu.md) — WebSocket 経由の Feishu/Lark ボット（プラグイン、別途インストール必要）。
-   [Google Chat](./channels/googlechat.md) — HTTP ウェブフック経由の Google Chat API アプリ。
-   [iMessage (レガシー)](./channels/imessage.md) — imsg CLI 経由のレガシー macOS 統合（非推奨、新規セットアップには BlueBubbles を使用）。
-   [IRC](./channels/irc.md) — クラシック IRC サーバー。ペアリング/許可リスト制御付きのチャンネル + DM。
-   [LINE](./channels/line.md) — LINE Messaging API ボット（プラグイン、別途インストール必要）。
-   [Matrix](./channels/matrix.md) — Matrix プロトコル（プラグイン、別途インストール必要）。
-   [Mattermost](./channels/mattermost.md) — Bot API + WebSocket。チャンネル、グループ、DM（プラグイン、別途インストール必要）。
-   [Microsoft Teams](./channels/msteams.md) — Bot Framework。エンタープライズサポート（プラグイン、別途インストール必要）。
-   [Nextcloud Talk](./channels/nextcloud-talk.md) — Nextcloud Talk 経由のセルフホスト型チャット（プラグイン、別途インストール必要）。
-   [Nostr](./channels/nostr.md) — NIP-04 経由の分散型 DM（プラグイン、別途インストール必要）。
-   [Signal](./channels/signal.md) — signal-cli。プライバシー重視。
-   [Synology Chat](./channels/synology-chat.md) — 送信+受信ウェブフック経由の Synology NAS Chat（プラグイン、別途インストール必要）。
-   [Slack](./channels/slack.md) — Bolt SDK。ワークスペースアプリ。
-   [Telegram](./channels/telegram.md) — grammY 経由の Bot API。グループをサポート。
-   [Tlon](./channels/tlon.md) — Urbit ベースのメッセンジャー（プラグイン、別途インストール必要）。
-   [Twitch](./channels/twitch.md) — IRC 接続経由の Twitch チャット（プラグイン、別途インストール必要）。
-   [WebChat](./web/webchat.md) — WebSocket 経由の Gateway WebChat UI。
-   [WhatsApp](./channels/whatsapp.md) — 最も人気。Baileys を使用し、QR ペアリングが必要。
-   [Zalo](./channels/zalo.md) — Zalo Bot API。ベトナムで人気のメッセンジャー（プラグイン、別途インストール必要）。
-   [Zalo Personal](./channels/zalouser.md) — QR ログイン経由の Zalo 個人アカウント（プラグイン、別途インストール必要）。

## 注意事項

-   チャネルは同時に実行可能です。複数設定すると、OpenClaw はチャットごとにルーティングします。
-   最も簡単なセットアップは通常 **Telegram**（シンプルなボットトークン）です。WhatsApp は QR ペアリングが必要で、より多くの状態をディスクに保存します。
-   グループの動作はチャネルによって異なります。[グループ](./channels/groups.md)を参照してください。
-   安全性のため、DM ペアリングと許可リストが適用されます。[セキュリティ](./gateway/security.md)を参照してください。
-   トラブルシューティング: [チャネルトラブルシューティング](./channels/troubleshooting.md)。
-   モデルプロバイダーは別途ドキュメント化されています。[モデルプロバイダー](./providers/models.md)を参照してください。

[BlueBubbles](./channels/bluebubbles.md)

---