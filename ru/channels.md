

  Обзор

  
# Чат-каналы

OpenClaw может общаться с вами в любом чат-приложении, которое вы уже используете. Каждый канал подключается через Шлюз (Gateway). Текст поддерживается везде; медиа и реакции зависят от канала.

## Поддерживаемые каналы

-   [BlueBubbles](./channels/bluebubbles.md) — **Рекомендуется для iMessage**; использует REST API сервера BlueBubbles для macOS с полной поддержкой функций (редактирование, удаление, эффекты, реакции, управление группами — редактирование в настоящее время не работает на macOS 26 Tahoe).
-   [Discord](./channels/discord.md) — Discord Bot API + Gateway; поддерживает серверы, каналы и личные сообщения.
-   [Feishu](./channels/feishu.md) — Бот Feishu/Lark через WebSocket (плагин, устанавливается отдельно).
-   [Google Chat](./channels/googlechat.md) — Приложение Google Chat API через HTTP-вебхук.
-   [iMessage (устаревший)](./channels/imessage.md) — Устаревшая интеграция с macOS через CLI imsg (устарело, для новых установок используйте BlueBubbles).
-   [IRC](./channels/irc.md) — Классические IRC-серверы; каналы + личные сообщения с контролем сопряжения/белого списка.
-   [LINE](./channels/line.md) — Бот LINE Messaging API (плагин, устанавливается отдельно).
-   [Matrix](./channels/matrix.md) — Протокол Matrix (плагин, устанавливается отдельно).
-   [Mattermost](./channels/mattermost.md) — Bot API + WebSocket; каналы, группы, личные сообщения (плагин, устанавливается отдельно).
-   [Microsoft Teams](./channels/msteams.md) — Bot Framework; поддержка для предприятий (плагин, устанавливается отдельно).
-   [Nextcloud Talk](./channels/nextcloud-talk.md) — Самостоятельно размещаемый чат через Nextcloud Talk (плагин, устанавливается отдельно).
-   [Nostr](./channels/nostr.md) — Децентрализованные личные сообщения через NIP-04 (плагин, устанавливается отдельно).
-   [Signal](./channels/signal.md) — signal-cli; ориентирован на конфиденциальность.
-   [Synology Chat](./channels/synology-chat.md) — Чат Synology NAS через исходящие+входящие вебхуки (плагин, устанавливается отдельно).
-   [Slack](./channels/slack.md) — Bolt SDK; рабочие пространства (workspace apps).
-   [Telegram](./channels/telegram.md) — Bot API через grammY; поддерживает группы.
-   [Tlon](./channels/tlon.md) — Мессенджер на основе Urbit (плагин, устанавливается отдельно).
-   [Twitch](./channels/twitch.md) — Чат Twitch через IRC-соединение (плагин, устанавливается отдельно).
-   [WebChat](./web/webchat.md) — Веб-интерфейс WebChat Шлюза (Gateway) через WebSocket.
-   [WhatsApp](./channels/whatsapp.md) — Самый популярный; использует Baileys и требует сопряжения по QR-коду.
-   [Zalo](./channels/zalo.md) — Zalo Bot API; популярный мессенджер во Вьетнаме (плагин, устанавливается отдельно).
-   [Zalo Personal](./channels/zalouser.md) — Личный аккаунт Zalo через вход по QR-коду (плагин, устанавливается отдельно).

## Примечания

-   Каналы могут работать одновременно; настройте несколько, и OpenClaw будет маршрутизировать сообщения для каждого чата.
-   Самый быстрый способ настройки — обычно **Telegram** (простой токен бота). WhatsApp требует сопряжения по QR-коду и хранит больше состояния на диске.
-   Поведение в группах зависит от канала; см. [Группы](./channels/groups.md).
-   Для безопасности применяются сопряжение в личных сообщениях и белые списки; см. [Безопасность](./gateway/security.md).
-   Устранение неполадок: [Устранение неполадок каналов](./channels/troubleshooting.md).
-   Провайдеры моделей документированы отдельно; см. [Провайдеры моделей](./providers/models.md).

[BlueBubbles](./channels/bluebubbles.md)

---