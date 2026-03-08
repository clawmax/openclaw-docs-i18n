

  Overview

  
# Chat Channels

OpenClaw can talk to you on any chat app you already use. Each channel connects via the Gateway. Text is supported everywhere; media and reactions vary by channel.

## Supported channels

-   [BlueBubbles](./channels/bluebubbles.md) — **Recommended for iMessage**; uses the BlueBubbles macOS server REST API with full feature support (edit, unsend, effects, reactions, group management — edit currently broken on macOS 26 Tahoe).
-   [Discord](./channels/discord.md) — Discord Bot API + Gateway; supports servers, channels, and DMs.
-   [Feishu](./channels/feishu.md) — Feishu/Lark bot via WebSocket (plugin, installed separately).
-   [Google Chat](./channels/googlechat.md) — Google Chat API app via HTTP webhook.
-   [iMessage (legacy)](./channels/imessage.md) — Legacy macOS integration via imsg CLI (deprecated, use BlueBubbles for new setups).
-   [IRC](./channels/irc.md) — Classic IRC servers; channels + DMs with pairing/allowlist controls.
-   [LINE](./channels/line.md) — LINE Messaging API bot (plugin, installed separately).
-   [Matrix](./channels/matrix.md) — Matrix protocol (plugin, installed separately).
-   [Mattermost](./channels/mattermost.md) — Bot API + WebSocket; channels, groups, DMs (plugin, installed separately).
-   [Microsoft Teams](./channels/msteams.md) — Bot Framework; enterprise support (plugin, installed separately).
-   [Nextcloud Talk](./channels/nextcloud-talk.md) — Self-hosted chat via Nextcloud Talk (plugin, installed separately).
-   [Nostr](./channels/nostr.md) — Decentralized DMs via NIP-04 (plugin, installed separately).
-   [Signal](./channels/signal.md) — signal-cli; privacy-focused.
-   [Synology Chat](./channels/synology-chat.md) — Synology NAS Chat via outgoing+incoming webhooks (plugin, installed separately).
-   [Slack](./channels/slack.md) — Bolt SDK; workspace apps.
-   [Telegram](./channels/telegram.md) — Bot API via grammY; supports groups.
-   [Tlon](./channels/tlon.md) — Urbit-based messenger (plugin, installed separately).
-   [Twitch](./channels/twitch.md) — Twitch chat via IRC connection (plugin, installed separately).
-   [WebChat](./web/webchat.md) — Gateway WebChat UI over WebSocket.
-   [WhatsApp](./channels/whatsapp.md) — Most popular; uses Baileys and requires QR pairing.
-   [Zalo](./channels/zalo.md) — Zalo Bot API; Vietnam’s popular messenger (plugin, installed separately).
-   [Zalo Personal](./channels/zalouser.md) — Zalo personal account via QR login (plugin, installed separately).

## Notes

-   Channels can run simultaneously; configure multiple and OpenClaw will route per chat.
-   Fastest setup is usually **Telegram** (simple bot token). WhatsApp requires QR pairing and stores more state on disk.
-   Group behavior varies by channel; see [Groups](./channels/groups.md).
-   DM pairing and allowlists are enforced for safety; see [Security](./gateway/security.md).
-   Troubleshooting: [Channel troubleshooting](./channels/troubleshooting.md).
-   Model providers are documented separately; see [Model Providers](./providers/models.md).

[BlueBubbles](./channels/bluebubbles.md)
