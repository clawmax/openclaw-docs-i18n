title: "Guide d'installation et de configuration de l'intégration du chat Twitch avec OpenClaw"
description: "Apprenez à configurer OpenClaw pour la prise en charge du chat Twitch. Connectez votre compte bot, générez des identifiants, gérez le contrôle d'accès et résolvez les problèmes courants."
keywords: ["bot twitch", "openclaw twitch", "intégration chat twitch", "connexion irc", "jeton twitch", "configuration bot", "contrôle d'accès", "multi-compte"]
---

  Plateformes de messagerie

  
# Twitch

Prise en charge du chat Twitch via une connexion IRC. OpenClaw se connecte en tant qu'utilisateur Twitch (compte bot) pour recevoir et envoyer des messages dans les canaux.

## Plugin requis

Twitch est fourni sous forme de plugin et n'est pas inclus dans l'installation de base. Installez-le via la CLI (registre npm) :

```bash
openclaw plugins install @openclaw/twitch
```

Installation locale (lors de l'exécution depuis un dépôt git) :

```bash
openclaw plugins install ./extensions/twitch
```

Détails : [Plugins](../tools/plugin.md)

## Configuration rapide (débutant)

1.  Créez un compte Twitch dédié pour le bot (ou utilisez un compte existant).
2.  Générez les identifiants : [Générateur de jeton Twitch](https://twitchtokengenerator.com/)
    -   Sélectionnez **Bot Token**
    -   Vérifiez que les portées `chat:read` et `chat:write` sont sélectionnées
    -   Copiez l'**ID Client** et le **Jeton d'accès**
3.  Trouvez votre ID utilisateur Twitch : [https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)
4.  Configurez le jeton :
    -   Variable d'env. : `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (compte par défaut uniquement)
    -   Ou config : `channels.twitch.accessToken`
    -   Si les deux sont définis, la configuration a la priorité (la variable d'env. est un secours pour le compte par défaut uniquement).
5.  Démarrez la passerelle.

**⚠️ Important :** Ajoutez un contrôle d'accès (`allowFrom` ou `allowedRoles`) pour empêcher les utilisateurs non autorisés de déclencher le bot. `requireMention` est `true` par défaut. Configuration minimale :

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw", // Compte Twitch du bot
      accessToken: "oauth:abc123...", // Jeton d'accès OAuth (ou utilisez la variable d'env OPENCLAW_TWITCH_ACCESS_TOKEN)
      clientId: "xyz789...", // ID Client du Générateur de jeton
      channel: "vevisk", // Canal Twitch à rejoindre (requis)
      allowFrom: ["123456789"], // (recommandé) Votre ID utilisateur Twitch uniquement - obtenez-le sur https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    },
  },
}
```

## Ce que c'est

-   Un canal Twitch détenu par la Passerelle.
-   Routage déterministe : les réponses vont toujours vers Twitch.
-   Chaque compte correspond à une clé de session isolée `agent::twitch:`.
-   `username` est le compte du bot (qui s'authentifie), `channel` est la salle de discussion à rejoindre.

## Installation (détaillée)

### Générer les identifiants

Utilisez [Twitch Token Generator](https://twitchtokengenerator.com/) :

-   Sélectionnez **Bot Token**
-   Vérifiez que les portées `chat:read` et `chat:write` sont sélectionnées
-   Copiez l'**ID Client** et le **Jeton d'accès**

Aucune inscription manuelle d'application nécessaire. Les jetons expirent après plusieurs heures.

### Configurer le bot

**Variable d'environnement (compte par défaut uniquement) :**

```
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**Ou configuration :**

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
    },
  },
}
```

Si la variable d'env. et la config sont définies, la configuration a la priorité.

### Contrôle d'accès (recommandé)

```json
{
  channels: {
    twitch: {
      allowFrom: ["123456789"], // (recommandé) Votre ID utilisateur Twitch uniquement
    },
  },
}
```

Préférez `allowFrom` pour une liste d'autorisation stricte. Utilisez `allowedRoles` à la place si vous voulez un accès basé sur les rôles. **Rôles disponibles :** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`. **Pourquoi les ID utilisateur ?** Les noms d'utilisateur peuvent changer, permettant l'usurpation. Les ID utilisateur sont permanents. Trouvez votre ID utilisateur Twitch : [https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/) (Convertissez votre nom d'utilisateur Twitch en ID)

## Actualisation du jeton (optionnel)

Les jetons de [Twitch Token Generator](https://twitchtokengenerator.com/) ne peuvent pas être actualisés automatiquement - régénérez-les à l'expiration. Pour un rafraîchissement automatique, créez votre propre application Twitch sur [Console Développeur Twitch](https://dev.twitch.tv/console) et ajoutez à la configuration :

```json
{
  channels: {
    twitch: {
      clientSecret: "votre_client_secret",
      refreshToken: "votre_refresh_token",
    },
  },
}
```

Le bot rafraîchit automatiquement les jetons avant expiration et enregistre les événements de rafraîchissement.

## Prise en charge multi-compte

Utilisez `channels.twitch.accounts` avec des jetons par compte. Voir [`gateway/configuration`](../gateway/configuration.md) pour le modèle partagé. Exemple (un compte bot dans deux canaux) :

```json
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk",
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel",
        },
      },
    },
  },
}
```

**Note :** Chaque compte a besoin de son propre jeton (un jeton par canal).

## Contrôle d'accès

### Restrictions basées sur les rôles

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"],
        },
      },
    },
  },
}
```

### Liste d'autorisation par ID utilisateur (plus sécurisé)

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"],
        },
      },
    },
  },
}
```

### Accès basé sur les rôles (alternative)

`allowFrom` est une liste d'autorisation stricte. Lorsqu'elle est définie, seuls ces ID utilisateur sont autorisés. Si vous voulez un accès basé sur les rôles, ne définissez pas `allowFrom` et configurez plutôt `allowedRoles` :

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

### Désactiver l'exigence de @mention

Par défaut, `requireMention` est `true`. Pour désactiver et répondre à tous les messages :

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false,
        },
      },
    },
  },
}
```

## Dépannage

D'abord, exécutez les commandes de diagnostic :

```bash
openclaw doctor
openclaw channels status --probe
```

### Le bot ne répond pas aux messages

**Vérifiez le contrôle d'accès :** Assurez-vous que votre ID utilisateur est dans `allowFrom`, ou supprimez temporairement `allowFrom` et définissez `allowedRoles: ["all"]` pour tester. **Vérifiez que le bot est dans le canal :** Le bot doit rejoindre le canal spécifié dans `channel`.

### Problèmes de jeton

**"Échec de connexion" ou erreurs d'authentification :**

-   Vérifiez que `accessToken` est la valeur du jeton d'accès OAuth (commence généralement par le préfixe `oauth:`)
-   Vérifiez que le jeton a les portées `chat:read` et `chat:write`
-   Si vous utilisez le rafraîchissement de jeton, vérifiez que `clientSecret` et `refreshToken` sont définis

### Le rafraîchissement de jeton ne fonctionne pas

**Vérifiez les journaux pour les événements de rafraîchissement :**

```bash
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

Si vous voyez "token refresh disabled (no refresh token)" :

-   Assurez-vous que `clientSecret` est fourni
-   Assurez-vous que `refreshToken` est fourni

## Configuration

**Configuration du compte :**

-   `username` - Nom d'utilisateur du bot
-   `accessToken` - Jeton d'accès OAuth avec `chat:read` et `chat:write`
-   `clientId` - ID Client Twitch (depuis le Générateur de jeton ou votre app)
-   `channel` - Canal à rejoindre (requis)
-   `enabled` - Activer ce compte (par défaut : `true`)
-   `clientSecret` - Optionnel : Pour le rafraîchissement automatique de jeton
-   `refreshToken` - Optionnel : Pour le rafraîchissement automatique de jeton
-   `expiresIn` - Expiration du jeton en secondes
-   `obtainmentTimestamp` - Horodatage d'obtention du jeton
-   `allowFrom` - Liste d'autorisation d'ID utilisateur
-   `allowedRoles` - Contrôle d'accès basé sur les rôles (`"moderator" | "owner" | "vip" | "subscriber" | "all"`)
-   `requireMention` - Exiger une @mention (par défaut : `true`)

**Options du fournisseur :**

-   `channels.twitch.enabled` - Activer/désactiver le démarrage du canal
-   `channels.twitch.username` - Nom d'utilisateur du bot (config simplifiée mono-compte)
-   `channels.twitch.accessToken` - Jeton d'accès OAuth (config simplifiée mono-compte)
-   `channels.twitch.clientId` - ID Client Twitch (config simplifiée mono-compte)
-   `channels.twitch.channel` - Canal à rejoindre (config simplifiée mono-compte)
-   `channels.twitch.accounts.` - Configuration multi-compte (tous les champs de compte ci-dessus)

Exemple complet :

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

## Actions de l'outil

L'agent peut appeler `twitch` avec l'action :

-   `send` - Envoyer un message à un canal

Exemple :

```json
{
  action: "twitch",
  params: {
    message: "Hello Twitch!",
    to: "#mychannel",
  },
}
```

## Sécurité et opérations

-   **Traitez les jetons comme des mots de passe** - Ne les committez jamais dans git
-   **Utilisez le rafraîchissement automatique de jeton** pour les bots de longue durée
-   **Utilisez des listes d'autorisation par ID utilisateur** au lieu des noms d'utilisateur pour le contrôle d'accès
-   **Surveillez les journaux** pour les événements de rafraîchissement de jeton et l'état de la connexion
-   **Limitez les portées des jetons** - Demandez uniquement `chat:read` et `chat:write`
-   **Si bloqué :** Redémarrez la passerelle après avoir confirmé qu'aucun autre processus ne possède la session

## Limites

-   **500 caractères** par message (découpé automatiquement aux limites des mots)
-   Le Markdown est supprimé avant le découpage
-   Pas de limitation de débit (utilise les limites de débit intégrées de Twitch)

[Tlon](./tlon.md)[WhatsApp](./whatsapp.md)