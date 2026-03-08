

  概览

  
# 聊天通道

OpenClaw 可以在您已使用的任何聊天应用上与您对话。每个通道都通过网关连接。文本在所有通道都受支持；媒体和消息反应功能因通道而异。

## 支持的通道

-   [BlueBubbles](./channels/bluebubbles.md) — **iMessage 推荐方案**；使用 BlueBubbles macOS 服务器 REST API，支持完整功能（编辑、撤回、效果、反应、群组管理 — 编辑功能目前在 macOS 26 Tahoe 上已损坏）。
-   [Discord](./channels/discord.md) — Discord 机器人 API + 网关；支持服务器、频道和私信。
-   [Feishu](./channels/feishu.md) — 通过 WebSocket 连接的飞书/Lark 机器人（插件，需单独安装）。
-   [Google Chat](./channels/googlechat.md) — 通过 HTTP webhook 连接的 Google Chat API 应用。
-   [iMessage (旧版)](./channels/imessage.md) — 通过 imsg CLI 的旧版 macOS 集成（已弃用，新设置请使用 BlueBubbles）。
-   [IRC](./channels/irc.md) — 经典 IRC 服务器；支持频道和私信，带有配对/白名单控制。
-   [LINE](./channels/line.md) — LINE Messaging API 机器人（插件，需单独安装）。
-   [Matrix](./channels/matrix.md) — Matrix 协议（插件，需单独安装）。
-   [Mattermost](./channels/mattermost.md) — 机器人 API + WebSocket；支持频道、群组、私信（插件，需单独安装）。
-   [Microsoft Teams](./channels/msteams.md) — Bot Framework；支持企业功能（插件，需单独安装）。
-   [Nextcloud Talk](./channels/nextcloud-talk.md) — 通过 Nextcloud Talk 的自托管聊天（插件，需单独安装）。
-   [Nostr](./channels/nostr.md) — 通过 NIP-04 的去中心化私信（插件，需单独安装）。
-   [Signal](./channels/signal.md) — signal-cli；注重隐私。
-   [Synology Chat](./channels/synology-chat.md) — 通过出站+入站 webhook 连接的 Synology NAS Chat（插件，需单独安装）。
-   [Slack](./channels/slack.md) — Bolt SDK；支持工作区应用。
-   [Telegram](./channels/telegram.md) — 通过 grammY 的 Bot API；支持群组。
-   [Tlon](./channels/tlon.md) — 基于 Urbit 的通讯工具（插件，需单独安装）。
-   [Twitch](./channels/twitch.md) — 通过 IRC 连接的 Twitch 聊天（插件，需单独安装）。
-   [WebChat](./web/webchat.md) — 基于 WebSocket 的网关 WebChat 用户界面。
-   [WhatsApp](./channels/whatsapp.md) — 最受欢迎；使用 Baileys 并需要二维码配对。
-   [Zalo](./channels/zalo.md) — Zalo Bot API；越南流行的通讯工具（插件，需单独安装）。
-   [Zalo Personal](./channels/zalouser.md) — 通过二维码登录的 Zalo 个人账户（插件，需单独安装）。

## 注意事项

-   通道可以同时运行；配置多个通道后，OpenClaw 将按聊天进行路由。
-   最快的设置通常是 **Telegram**（简单的机器人令牌）。WhatsApp 需要二维码配对并在磁盘上存储更多状态。
-   群组行为因通道而异；请参阅[群组](./channels/groups.md)。
-   出于安全考虑，强制执行私信配对和白名单；请参阅[安全](./gateway/security.md)。
-   故障排除：[通道故障排除](./channels/troubleshooting.md)。
-   模型提供商的文档是分开的；请参阅[模型提供商](./providers/models.md)。

[BlueBubbles](./channels/bluebubbles.md)

---