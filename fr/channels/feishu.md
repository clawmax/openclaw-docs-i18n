

  Plateformes de messagerie

  
# Feishu

Feishu (Lark) est une plateforme de chat d'équipe utilisée par les entreprises pour la messagerie et la collaboration. Ce plugin connecte OpenClaw à un bot Feishu/Lark en utilisant l'abonnement aux événements WebSocket de la plateforme, permettant ainsi de recevoir des messages sans exposer d'URL de webhook publique.

* * *

## Plugin requis

Installez le plugin Feishu :

```bash
openclaw plugins install @openclaw/feishu
```

Installation locale (lors de l'exécution depuis un dépôt git) :

```bash
openclaw plugins install ./extensions/feishu
```

* * *

## Démarrage rapide

Il existe deux façons d'ajouter le canal Feishu :

### Méthode 1 : assistant de configuration (recommandé)

Si vous venez d'installer OpenClaw, exécutez l'assistant :

```bash
openclaw onboard
```

L'assistant vous guide à travers :

1.  La création d'une application Feishu et la collecte des identifiants
2.  La configuration des identifiants de l'application dans OpenClaw
3.  Le démarrage de la passerelle

✅ **Après la configuration**, vérifiez l'état de la passerelle :

-   `openclaw gateway status`
-   `openclaw logs --follow`

### Méthode 2 : configuration par CLI

Si vous avez déjà terminé l'installation initiale, ajoutez le canal via la CLI :

```bash
openclaw channels add
```

Choisissez **Feishu**, puis entrez l'ID d'application et le Secret d'application. ✅ **Après la configuration**, gérez la passerelle :

-   `openclaw gateway status`
-   `openclaw gateway restart`
-   `openclaw logs --follow`

* * *

## Étape 1 : Créer une application Feishu

### 1\. Ouvrir la plateforme ouverte Feishu

Visitez la [Plateforme ouverte Feishu](https://open.feishu.cn/app) et connectez-vous. Les locataires Lark (globaux) doivent utiliser [https://open.larksuite.com/app](https://open.larksuite.com/app) et définir `domain: "lark"` dans la configuration Feishu.

### 2\. Créer une application

1.  Cliquez sur **Créer une application d'entreprise**
2.  Remplissez le nom et la description de l'application
3.  Choisissez une icône pour l'application

![Créer une application d'entreprise](../images/channels-feishu-step2-create-app.png.md)

### 3\. Copier les identifiants

Dans **Informations d'identification et informations de base**, copiez :

-   **ID d'application** (format : `cli_xxx`)
-   **Secret d'application**

❗ **Important :** gardez le Secret d'application privé. ![Obtenir les identifiants](../images/channels-feishu-step3-credentials.png.md)

### 4\. Configurer les autorisations

Dans **Autorisations**, cliquez sur **Importation par lot** et collez :

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:read",
      "cardkit:card:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

![Configurer les autorisations](../images/channels-feishu-step4-permissions.png.md)

### 5\. Activer la capacité de bot

Dans **Capacités de l'application** > **Bot** :

1.  Activez la capacité de bot
2.  Définissez le nom du bot

![Activer la capacité de bot](../images/channels-feishu-step5-bot-capability.png.md)

### 6\. Configurer l'abonnement aux événements

⚠️ **Important :** avant de configurer l'abonnement aux événements, assurez-vous que :

1.  Vous avez déjà exécuté `openclaw channels add` pour Feishu
2.  La passerelle est en cours d'exécution (`openclaw gateway status`)

Dans **Abonnement aux événements** :

1.  Choisissez **Utiliser une connexion longue pour recevoir les événements** (WebSocket)
2.  Ajoutez l'événement : `im.message.receive_v1`

⚠️ Si la passerelle n'est pas en cours d'exécution, la configuration de la connexion longue peut échouer à être enregistrée. ![Configurer l'abonnement aux événements](../images/channels-feishu-step6-event-subscription.png.md)

### 7\. Publier l'application

1.  Créez une version dans **Gestion des versions et publication**
2.  Soumettez pour examen et publiez
3.  Attendez l'approbation de l'administrateur (les applications d'entreprise sont généralement approuvées automatiquement)

* * *

## Étape 2 : Configurer OpenClaw

### Configurer avec l'assistant (recommandé)

```bash
openclaw channels add
```

Choisissez **Feishu** et collez votre ID d'application et votre Secret d'application.

### Configurer via le fichier de configuration

Modifiez `~/.openclaw/openclaw.json` :

```json
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Mon assistant IA",
        },
      },
    },
  },
}
```

Si vous utilisez `connectionMode: "webhook"`, définissez `verificationToken`. Le serveur webhook Feishu se lie à `127.0.0.1` par défaut ; définissez `webhookHost` uniquement si vous avez intentionnellement besoin d'une adresse de liaison différente.

#### Jeton de vérification (mode webhook)

Lorsque vous utilisez le mode webhook, définissez `channels.feishu.verificationToken` dans votre configuration. Pour obtenir la valeur :

1.  Dans la Plateforme ouverte Feishu, ouvrez votre application
2.  Allez dans **Développement** → **Événements et rappels** (开发配置 → 事件与回调)
3.  Ouvrez l'onglet **Chiffrement** (加密策略)
4.  Copiez le **Jeton de vérification**

![Emplacement du jeton de vérification](../images/channels-feishu-verification-token.png.md)

### Configurer via les variables d'environnement

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Domaine Lark (global)

Si votre locataire est sur Lark (international), définissez le domaine sur `lark` (ou une chaîne de domaine complète). Vous pouvez le définir au niveau `channels.feishu.domain` ou par compte (`channels.feishu.accounts..domain`).

```json
{
  channels: {
    feishu: {
      domain: "lark",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```

### Indicateurs d'optimisation des quotas

Vous pouvez réduire l'utilisation de l'API Feishu avec deux indicateurs optionnels :

-   `typingIndicator` (par défaut `true`) : lorsqu'il est `false`, ignore les appels de réaction de frappe.
-   `resolveSenderNames` (par défaut `true`) : lorsqu'il est `false`, ignore les appels de recherche de profil de l'expéditeur.

Définissez-les au niveau supérieur ou par compte :

```json
{
  channels: {
    feishu: {
      typingIndicator: false,
      resolveSenderNames: false,
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          typingIndicator: true,
          resolveSenderNames: false,
        },
      },
    },
  },
}
```

* * *

## Étape 3 : Démarrer + tester

### 1\. Démarrer la passerelle

```bash
openclaw gateway
```

### 2\. Envoyer un message de test

Dans Feishu, trouvez votre bot et envoyez un message.

### 3\. Approuver l'appairage

Par défaut, le bot répond avec un code d'appairage. Approuvez-le :

```bash
openclaw pairing approve feishu <CODE>
```

Après approbation, vous pouvez discuter normalement.

* * *

## Aperçu

-   **Canal de bot Feishu** : Bot Feishu géré par la passerelle
-   **Routage déterministe** : les réponses reviennent toujours à Feishu
-   **Isolation des sessions** : les messages directs partagent une session principale ; les groupes sont isolés
-   **Connexion WebSocket** : connexion longue via le SDK Feishu, aucune URL publique nécessaire

* * *

## Contrôle d'accès

### Messages directs

-   **Par défaut** : `dmPolicy: "pairing"` (les utilisateurs inconnus reçoivent un code d'appairage)
-   **Approuver l'appairage** :
    
    Copier
    
    ```bash
    openclaw pairing list feishu
    openclaw pairing approve feishu <CODE>
    ```
    
-   **Mode liste d'autorisation** : définissez `channels.feishu.allowFrom` avec les ID ouverts autorisés

### Discussions de groupe

**1\. Politique de groupe** (`channels.feishu.groupPolicy`) :

-   `"open"` = autoriser tout le monde dans les groupes (par défaut)
-   `"allowlist"` = n'autoriser que `groupAllowFrom`
-   `"disabled"` = désactiver les messages de groupe

**2\. Exigence de mention** (`channels.feishu.groups.<chat_id>.requireMention`) :

-   `true` = exiger une mention @ (par défaut)
-   `false` = répondre sans mentions

* * *

## Exemples de configuration de groupe

### Autoriser tous les groupes, exiger une mention @ (par défaut)

```json
{
  channels: {
    feishu: {
      groupPolicy: "open",
      // Par défaut requireMention: true
    },
  },
}
```

### Autoriser tous les groupes, aucune mention @ requise

```json
{
  channels: {
    feishu: {
      groups: {
        oc_xxx: { requireMention: false },
      },
    },
  },
}
```

### Autoriser uniquement des groupes spécifiques

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      // Les ID de groupe Feishu (chat_id) ressemblent à : oc_xxx
      groupAllowFrom: ["oc_xxx", "oc_yyy"],
    },
  },
}
```

### Restreindre les expéditeurs autorisés à envoyer des messages dans un groupe (liste d'autorisation des expéditeurs)

En plus d'autoriser le groupe lui-même, **tous les messages** dans ce groupe sont filtrés par l'open_id de l'expéditeur : seuls les utilisateurs listés dans `groups.<chat_id>.allowFrom` voient leurs messages traités ; les messages des autres membres sont ignorés (il s'agit d'un filtrage complet au niveau de l'expéditeur, pas seulement pour les commandes de contrôle comme /reset ou /new).

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["oc_xxx"],
      groups: {
        oc_xxx: {
          // Les ID utilisateur Feishu (open_id) ressemblent à : ou_xxx
          allowFrom: ["ou_user1", "ou_user2"],
        },
      },
    },
  },
}
```

* * *

## Obtenir les ID de groupe/utilisateur

### ID de groupe (chat_id)

Les ID de groupe ressemblent à `oc_xxx`. **Méthode 1 (recommandée)**

1.  Démarrez la passerelle et mentionnez @ le bot dans le groupe
2.  Exécutez `openclaw logs --follow` et cherchez `chat_id`

**Méthode 2** Utilisez le débogueur de l'API Feishu pour lister les discussions de groupe.

### ID utilisateur (open_id)

Les ID utilisateur ressemblent à `ou_xxx`. **Méthode 1 (recommandée)**

1.  Démarrez la passerelle et envoyez un message direct au bot
2.  Exécutez `openclaw logs --follow` et cherchez `open_id`

**Méthode 2** Vérifiez les demandes d'appairage pour les ID ouverts des utilisateurs :

```bash
openclaw pairing list feishu
```

* * *

## Commandes courantes

| Commande | Description |
| --- | --- |
| `/status` | Afficher l'état du bot |
| `/reset` | Réinitialiser la session |
| `/model` | Afficher/changer de modèle |

> Remarque : Feishu ne prend pas encore en charge les menus de commandes natifs, donc les commandes doivent être envoyées sous forme de texte.

## Commandes de gestion de la passerelle

| Commande | Description |
| --- | --- |
| `openclaw gateway status` | Afficher l'état de la passerelle |
| `openclaw gateway install` | Installer/démarrer le service de passerelle |
| `openclaw gateway stop` | Arrêter le service de passerelle |
| `openclaw gateway restart` | Redémarrer le service de passerelle |
| `openclaw logs --follow` | Suivre les journaux de la passerelle |

* * *

## Dépannage

### Le bot ne répond pas dans les discussions de groupe

1.  Assurez-vous que le bot est ajouté au groupe
2.  Assurez-vous de mentionner @ le bot (comportement par défaut)
3.  Vérifiez que `groupPolicy` n'est pas défini sur `"disabled"`
4.  Consultez les journaux : `openclaw logs --follow`

### Le bot ne reçoit pas de messages

1.  Assurez-vous que l'application est publiée et approuvée
2.  Assurez-vous que l'abonnement aux événements inclut `im.message.receive_v1`
3.  Assurez-vous que la **connexion longue** est activée
4.  Assurez-vous que les autorisations de l'application sont complètes
5.  Assurez-vous que la passerelle est en cours d'exécution : `openclaw gateway status`
6.  Consultez les journaux : `openclaw logs --follow`

### Fuite du Secret d'application

1.  Réinitialisez le Secret d'application dans la Plateforme ouverte Feishu
2.  Mettez à jour le Secret d'application dans votre configuration
3.  Redémarrez la passerelle

### Échecs d'envoi de messages

1.  Assurez-vous que l'application a l'autorisation `im:message:send_as_bot`
2.  Assurez-vous que l'application est publiée
3.  Consultez les journaux pour des erreurs détaillées

* * *

## Configuration avancée

### Comptes multiples

```json
{
  channels: {
    feishu: {
      defaultAccount: "main",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Bot principal",
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          botName: "Bot de secours",
          enabled: false,
        },
      },
    },
  },
}
```

`defaultAccount` contrôle quel compte Feishu est utilisé lorsque les API sortantes ne spécifient pas explicitement un `accountId`.

### Limites de messages

-   `textChunkLimit` : taille des fragments de texte sortants (par défaut : 2000 caractères)
-   `mediaMaxMb` : limite de téléchargement/téléversement des médias (par défaut : 30 Mo)

### Diffusion en continu (streaming)

Feishu prend en charge les réponses en continu via des cartes interactives. Lorsqu'elle est activée, le bot met à jour une carte au fur et à mesure qu'il génère du texte.

```json
{
  channels: {
    feishu: {
      streaming: true, // activer la sortie de carte en continu (par défaut true)
      blockStreaming: true, // activer la diffusion en continu au niveau des blocs (par défaut true)
    },
  },
}
```

Définissez `streaming: false` pour attendre la réponse complète avant d'envoyer.

### Routage multi-agent

Utilisez `bindings` pour router les messages directs ou les groupes Feishu vers différents agents.

```json
{
  agents: {
    list: [
      { id: "main" },
      {
        id: "clawd-fan",
        workspace: "/home/user/clawd-fan",
        agentDir: "/home/user/.openclaw/agents/clawd-fan/agent",
      },
      {
        id: "clawd-xi",
        workspace: "/home/user/clawd-xi",
        agentDir: "/home/user/.openclaw/agents/clawd-xi/agent",
      },
    ],
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "clawd-fan",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_yyy" },
      },
    },
    {
      agentId: "clawd-xi",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

Champs de routage :

-   `match.channel` : `"feishu"`
-   `match.peer.kind` : `"direct"` ou `"group"`
-   `match.peer.id` : ID ouvert utilisateur (`ou_xxx`) ou ID de groupe (`oc_xxx`)

Voir [Obtenir les ID de groupe/utilisateur](#get-groupuser-ids) pour des conseils de recherche.

* * *

## Référence de configuration

Configuration complète : [Configuration de la passerelle](../gateway/configuration.md) Options clés :

| Paramètre | Description | Par défaut |
| --- | --- | --- |
| `channels.feishu.enabled` | Activer/désactiver le canal | `true` |
| `channels.feishu.domain` | Domaine de l'API (`feishu` ou `lark`) | `feishu` |
| `channels.feishu.connectionMode` | Mode de transport des événements | `websocket` |
| `channels.feishu.defaultAccount` | ID de compte par défaut pour le routage sortant | `default` |
| `channels.feishu.verificationToken` | Requis pour le mode webhook | \- |
| `channels.feishu.webhookPath` | Chemin de la route webhook | `/feishu/events` |
| `channels.feishu.webhookHost` | Hôte de liaison du webhook | `127.0.0.1` |
| `channels.feishu.webhookPort` | Port de liaison du webhook | `3000` |
| `channels.feishu.accounts..appId` | ID d'application | \- |
| `channels.feishu.accounts..appSecret` | Secret d'application | \- |
| `channels.feishu.accounts..domain` | Remplacement du domaine API par compte | `feishu` |
| `channels.feishu.dmPolicy` | Politique pour les messages directs | `pairing` |
| `channels.feishu.allowFrom` | Liste d'autorisation pour les messages directs (liste d'open_id) | \- |
| `channels.feishu.groupPolicy` | Politique pour les groupes | `open` |
| `channels.feishu.groupAllowFrom` | Liste d'autorisation pour les groupes | \- |
| `channels.feishu.groups.<chat_id>.requireMention` | Exiger une mention @ | `true` |
| `channels.feishu.groups.<chat_id>.enabled` | Activer le groupe | `true` |
| `channels.feishu.textChunkLimit` | Taille des fragments de message | `2000` |
| `channels.feishu.mediaMaxMb` | Limite de taille des médias | `30` |
| `channels.feishu.streaming` | Activer la sortie de carte en continu | `true` |
| `channels.feishu.blockStreaming` | Activer la diffusion en continu au niveau des blocs | `true` |

* * *

## Référence de dmPolicy

| Valeur | Comportement |
| --- | --- |
| `"pairing"` | **Par défaut.** Les utilisateurs inconnus reçoivent un code d'appairage ; doivent être approuvés |
| `"allowlist"` | Seuls les utilisateurs dans `allowFrom` peuvent discuter |
| `"open"` | Autoriser tous les utilisateurs (nécessite `"*"` dans allowFrom) |
| `"disabled"` | Désactiver les messages directs |

* * *

## Types de messages pris en charge

### Réception

-   ✅ Texte
-   ✅ Texte enrichi (publication)
-   ✅ Images
-   ✅ Fichiers
-   ✅ Audio
-   ✅ Vidéo
-   ✅ Autocollants

### Envoi

-   ✅ Texte
-   ✅ Images
-   ✅ Fichiers
-   ✅ Audio
-   ⚠️ Texte enrichi (prise en charge partielle)

[Discord](./discord.md)[Google Chat](./googlechat.md)