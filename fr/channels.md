

  Aperçu

  
# Canaux de discussion

OpenClaw peut discuter avec vous sur n'importe quelle application de discussion que vous utilisez déjà. Chaque canal se connecte via la Passerelle. Le texte est pris en charge partout ; les médias et les réactions varient selon le canal.

## Canaux pris en charge

-   [BlueBubbles](./channels/bluebubbles.md) — **Recommandé pour iMessage** ; utilise l'API REST du serveur BlueBubbles macOS avec prise en charge complète des fonctionnalités (modifier, annuler l'envoi, effets, réactions, gestion de groupe — la modification est actuellement cassée sur macOS 26 Tahoe).
-   [Discord](./channels/discord.md) — API de bot Discord + Passerelle ; prend en charge les serveurs, les canaux et les messages privés.
-   [Feishu](./channels/feishu.md) — Bot Feishu/Lark via WebSocket (plugin, installé séparément).
-   [Google Chat](./channels/googlechat.md) — Application API Google Chat via webhook HTTP.
-   [iMessage (hérité)](./channels/imessage.md) — Intégration macOS héritée via CLI imsg (obsolète, utilisez BlueBubbles pour les nouvelles installations).
-   [IRC](./channels/irc.md) — Serveurs IRC classiques ; canaux + messages privés avec contrôles d'appairage/liste blanche.
-   [LINE](./channels/line.md) — Bot API de messagerie LINE (plugin, installé séparément).
-   [Matrix](./channels/matrix.md) — Protocole Matrix (plugin, installé séparément).
-   [Mattermost](./channels/mattermost.md) — API de bot + WebSocket ; canaux, groupes, messages privés (plugin, installé séparément).
-   [Microsoft Teams](./channels/msteams.md) — Bot Framework ; support entreprise (plugin, installé séparément).
-   [Nextcloud Talk](./channels/nextcloud-talk.md) — Discussion auto-hébergée via Nextcloud Talk (plugin, installé séparément).
-   [Nostr](./channels/nostr.md) — Messages privés décentralisés via NIP-04 (plugin, installé séparément).
-   [Signal](./channels/signal.md) — signal-cli ; axé sur la confidentialité.
-   [Synology Chat](./channels/synology-chat.md) — Chat Synology NAS via webhooks sortants+entrants (plugin, installé séparément).
-   [Slack](./channels/slack.md) — SDK Bolt ; applications d'espace de travail.
-   [Telegram](./channels/telegram.md) — API de bot via grammY ; prend en charge les groupes.
-   [Tlon](./channels/tlon.md) — Messagerie basée sur Urbit (plugin, installé séparément).
-   [Twitch](./channels/twitch.md) — Chat Twitch via connexion IRC (plugin, installé séparément).
-   [WebChat](./web/webchat.md) — Interface WebChat de la Passerelle via WebSocket.
-   [WhatsApp](./channels/whatsapp.md) — Le plus populaire ; utilise Baileys et nécessite un appairage par QR code.
-   [Zalo](./channels/zalo.md) — API de bot Zalo ; messagerie populaire au Vietnam (plugin, installé séparément).
-   [Zalo Personal](./channels/zalouser.md) — Compte personnel Zalo via connexion par QR code (plugin, installé séparément).

## Notes

-   Les canaux peuvent fonctionner simultanément ; configurez-en plusieurs et OpenClaw acheminera par discussion.
-   La configuration la plus rapide est généralement **Telegram** (jeton de bot simple). WhatsApp nécessite un appairage par QR code et stocke plus d'état sur le disque.
-   Le comportement des groupes varie selon le canal ; voir [Groupes](./channels/groups.md).
-   L'appairage en messages privés et les listes blanches sont appliqués pour la sécurité ; voir [Sécurité](./gateway/security.md).
-   Dépannage : [Dépannage des canaux](./channels/troubleshooting.md).
-   Les fournisseurs de modèles sont documentés séparément ; voir [Fournisseurs de modèles](./providers/models.md).

[BlueBubbles](./channels/bluebubbles.md)

---