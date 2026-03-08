title: "Guide d'installation et de configuration du plugin OpenClaw pour Synology Chat"
description: "Apprenez à installer et configurer le plugin OpenClaw pour Synology Chat. Envoyez et recevez des messages via des webhooks, configurez les politiques de messages privés et gérez plusieurs comptes."
keywords: ["synology chat", "plugin openclaw", "configuration webhook", "canal de messages privés", "intégration synology chat", "webhook entrant", "webhook sortant", "politique de mp"]
---

  Plateformes de messagerie

  
# Synology Chat

Statut : pris en charge via un plugin en tant que canal de messages privés utilisant les webhooks de Synology Chat. Le plugin accepte les messages entrants des webhooks sortants de Synology Chat et envoie les réponses via un webhook entrant de Synology Chat.

## Plugin requis

Synology Chat est basé sur un plugin et ne fait pas partie de l'installation par défaut des canaux principaux. Installez-le depuis un dépôt local :

```bash
openclaw plugins install ./extensions/synology-chat
```

Détails : [Plugins](../tools/plugin.md)

## Configuration rapide

1.  Installez et activez le plugin Synology Chat.
2.  Dans les intégrations de Synology Chat :
    -   Créez un webhook entrant et copiez son URL.
    -   Créez un webhook sortant avec votre jeton secret.
3.  Pointez l'URL du webhook sortant vers votre passerelle OpenClaw :
    -   `https://gateway-host/webhook/synology` par défaut.
    -   Ou votre `channels.synology-chat.webhookPath` personnalisé.
4.  Configurez `channels.synology-chat` dans OpenClaw.
5.  Redémarrez la passerelle et envoyez un MP au bot Synology Chat.

Configuration minimale :

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      token: "synology-outgoing-token",
      incomingUrl: "https://nas.example.com/webapi/entry.cgi?api=SYNO.Chat.External&method=incoming&version=2&token=...",
      webhookPath: "/webhook/synology",
      dmPolicy: "allowlist",
      allowedUserIds: ["123456"],
      rateLimitPerMinute: 30,
      allowInsecureSsl: false,
    },
  },
}
```

## Variables d'environnement

Pour le compte par défaut, vous pouvez utiliser les variables d'environnement :

-   `SYNOLOGY_CHAT_TOKEN`
-   `SYNOLOGY_CHAT_INCOMING_URL`
-   `SYNOLOGY_NAS_HOST`
-   `SYNOLOGY_ALLOWED_USER_IDS` (séparés par des virgules)
-   `SYNOLOGY_RATE_LIMIT`
-   `OPENCLAW_BOT_NAME`

Les valeurs de configuration écrasent les variables d'environnement.

## Politique de MP et contrôle d'accès

-   `dmPolicy: "allowlist"` est la valeur par défaut recommandée.
-   `allowedUserIds` accepte une liste (ou une chaîne séparée par des virgules) d'identifiants utilisateur Synology.
-   En mode `allowlist`, une liste `allowedUserIds` vide est considérée comme une mauvaise configuration et la route du webhook ne démarrera pas (utilisez `dmPolicy: "open"` pour autoriser tout le monde).
-   `dmPolicy: "open"` autorise tout expéditeur.
-   `dmPolicy: "disabled"` bloque les MP.
-   Les approbations d'appariement fonctionnent avec :
    -   `openclaw pairing list synology-chat`
    -   `openclaw pairing approve synology-chat `

## Envoi sortant

Utilisez les identifiants numériques d'utilisateur Synology Chat comme cibles. Exemples :

```bash
openclaw message send --channel synology-chat --target 123456 --text "Hello from OpenClaw"
openclaw message send --channel synology-chat --target synology-chat:123456 --text "Hello again"
```

L'envoi de médias est pris en charge par la livraison de fichiers basée sur une URL.

## Multi-comptes

Plusieurs comptes Synology Chat sont pris en charge sous `channels.synology-chat.accounts`. Chaque compte peut remplacer le jeton, l'URL entrante, le chemin du webhook, la politique de MP et les limites.

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      accounts: {
        default: {
          token: "token-a",
          incomingUrl: "https://nas-a.example.com/...token=...",
        },
        alerts: {
          token: "token-b",
          incomingUrl: "https://nas-b.example.com/...token=...",
          webhookPath: "/webhook/synology-alerts",
          dmPolicy: "allowlist",
          allowedUserIds: ["987654"],
        },
      },
    },
  },
}
```

## Notes de sécurité

-   Gardez le `token` secret et changez-le s'il est divulgué.
-   Gardez `allowInsecureSsl: false` sauf si vous faites explicitement confiance à un certificat local auto-signé du NAS.
-   Les requêtes de webhook entrantes sont vérifiées par jeton et limitées en débit par expéditeur.
-   Préférez `dmPolicy: "allowlist"` pour la production.

[Signal](./signal.md)[Slack](./slack.md)

---