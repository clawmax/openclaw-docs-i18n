title: "Guide de configuration de l'intégration Microsoft Teams pour OpenClaw AI"
description: "Apprenez à installer et configurer le plugin Microsoft Teams pour OpenClaw AI. Configurez Azure Bot, gérez le contrôle d'accès et activez la messagerie dans les messages privés, groupes et canaux."
keywords: ["intégration microsoft teams", "plugin openclaw msteams", "configuration azure bot", "configuration chatbot teams", "canaux openclaw", "webhook teams", "portail développeur teams", "tunnel de développement local"]
updated: """""2026-01-21"""""
---

  Plateformes de messagerie

  
# Microsoft Teams

> “Vous qui entrez, laissez toute espérance.”

Mise à jour : 2026-01-21 Statut : le texte et les pièces jointes en MP sont pris en charge ; l'envoi de fichiers dans les canaux/groupes nécessite `sharePointSiteId` + permissions Graph (voir [Envoi de fichiers dans les conversations de groupe](#sending-files-in-group-chats)). Les sondages sont envoyés via des cartes adaptatives.

## Plugin requis

Microsoft Teams est fourni sous forme de plugin et n'est pas inclus dans l'installation de base. **Changement cassant (2026.1.15) :** MS Teams a été déplacé hors du cœur. Si vous l'utilisez, vous devez installer le plugin. Explication : permet de garder les installations de base plus légères et laisse les dépendances MS Teams se mettre à jour indépendamment. Installation via CLI (registre npm) :

```bash
openclaw plugins install @openclaw/msteams
```

Installation locale (lors d'une exécution depuis un dépôt git) :

```bash
openclaw plugins install ./extensions/msteams
```

Si vous choisissez Teams pendant la configuration/onboarding et qu'un dépôt git est détecté, OpenClaw proposera automatiquement le chemin d'installation local. Détails : [Plugins](../tools/plugin.md)

## Configuration rapide (débutant)

1.  Installez le plugin Microsoft Teams.
2.  Créez un **Azure Bot** (ID d'application + secret client + ID de locataire).
3.  Configurez OpenClaw avec ces identifiants.
4.  Exposez `/api/messages` (port 3978 par défaut) via une URL publique ou un tunnel.
5.  Installez le package d'application Teams et démarrez la passerelle.

Configuration minimale :

```json
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

Note : les conversations de groupe sont bloquées par défaut (`channels.msteams.groupPolicy: "allowlist"`). Pour autoriser les réponses de groupe, définissez `channels.msteams.groupAllowFrom` (ou utilisez `groupPolicy: "open"` pour autoriser tout membre, avec mention obligatoire).

## Objectifs

-   Parler à OpenClaw via les MP Teams, les conversations de groupe ou les canaux.
-   Garder le routage déterministe : les réponses retournent toujours sur le canal d'origine.
-   Comportement par défaut sécurisé pour les canaux (mentions requises sauf configuration contraire).

## Écritures de configuration

Par défaut, Microsoft Teams est autorisé à écrire des mises à jour de configuration déclenchées par `/config set|unset` (nécessite `commands.config: true`). Désactivez avec :

```json
{
  channels: { msteams: { configWrites: false } },
}
```

## Contrôle d'accès (MP + groupes)

**Accès en MP**

-   Par défaut : `channels.msteams.dmPolicy = "pairing"`. Les expéditeurs inconnus sont ignorés jusqu'à approbation.
-   `channels.msteams.allowFrom` doit utiliser des ID d'objet AAD stables.
-   Les UPNs/noms d'affichage sont mutables ; la correspondance directe est désactivée par défaut et n'est activée qu'avec `channels.msteams.dangerouslyAllowNameMatching: true`.
-   L'assistant peut résoudre les noms en ID via Microsoft Graph lorsque les identifiants le permettent.

**Accès aux groupes**

-   Par défaut : `channels.msteams.groupPolicy = "allowlist"` (bloqué sauf si vous ajoutez `groupAllowFrom`). Utilisez `channels.defaults.groupPolicy` pour remplacer la valeur par défaut si non définie.
-   `channels.msteams.groupAllowFrom` contrôle quels expéditeurs peuvent déclencher des actions dans les conversations de groupe/canaux (reprend `channels.msteams.allowFrom`).
-   Définissez `groupPolicy: "open"` pour autoriser tout membre (toujours avec mention obligatoire par défaut).
-   Pour n'autoriser **aucun canal**, définissez `channels.msteams.groupPolicy: "disabled"`.

Exemple :

```json
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**Liste d'autorisation Teams + canaux**

-   Limitez les réponses de groupe/canal en listant les équipes et canaux sous `channels.msteams.teams`.
-   Les clés peuvent être des ID d'équipe ou des noms ; les clés de canal peuvent être des ID de conversation ou des noms.
-   Lorsque `groupPolicy="allowlist"` et qu'une liste d'autorisation d'équipes est présente, seules les équipes/canaux listés sont acceptés (avec mention obligatoire).
-   L'assistant de configuration accepte les entrées `Équipe/Canal` et les stocke pour vous.
-   Au démarrage, OpenClaw résout les noms d'équipe/canal et les noms de la liste d'autorisation d'utilisateurs en ID (lorsque les permissions Graph le permettent) et journalise le mapping ; les entrées non résolues sont conservées telles quelles.

Exemple :

```json
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "Mon Équipe": {
          channels: {
            Général: { requireMention: true },
          },
        },
      },
    },
  },
}
```

## Fonctionnement

1.  Installez le plugin Microsoft Teams.
2.  Créez un **Azure Bot** (ID d'application + secret + ID de locataire).
3.  Construisez un **package d'application Teams** qui référence le bot et inclut les permissions RSC ci-dessous.
4.  Téléchargez/installez l'application Teams dans une équipe (ou dans le scope personnel pour les MP).
5.  Configurez `msteams` dans `~/.openclaw/openclaw.json` (ou variables d'environnement) et démarrez la passerelle.
6.  La passerelle écoute le trafic webhook Bot Framework sur `/api/messages` par défaut.

## Configuration d'Azure Bot (Prérequis)

Avant de configurer OpenClaw, vous devez créer une ressource Azure Bot.

### Étape 1 : Créer un Azure Bot

1.  Allez sur [Créer un Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2.  Remplissez l'onglet **Informations de base** :
    
    | Champ | Valeur |
    | --- | --- |
    | **Identifiant du bot** | Le nom de votre bot, p. ex. `openclaw-msteams` (doit être unique) |
    | **Abonnement** | Sélectionnez votre abonnement Azure |
    | **Groupe de ressources** | Créer nouveau ou utiliser existant |
    | **Niveau tarifaire** | **Gratuit** pour dev/test |
    | **Type d'application** | **Locataire unique** (recommandé - voir note ci-dessous) |
    | **Type de création** | **Créer un nouvel ID d'application Microsoft** |
    

> **Avis de dépréciation :** La création de nouveaux bots multi-locataires a été dépréciée après le 2025-07-31. Utilisez **Locataire unique** pour les nouveaux bots.

3.  Cliquez sur **Vérifier + créer** → **Créer** (attendez ~1-2 minutes)

### Étape 2 : Obtenir les identifiants

1.  Allez dans votre ressource Azure Bot → **Configuration**
2.  Copiez **ID d'application Microsoft** → c'est votre `appId`
3.  Cliquez sur **Gérer le mot de passe** → allez dans l'Inscription d'application
4.  Sous **Certificats et secrets** → **Nouveau secret client** → copiez la **Valeur** → c'est votre `appPassword`
5.  Allez dans **Vue d'ensemble** → copiez **ID de répertoire (locataire)** → c'est votre `tenantId`

### Étape 3 : Configurer le point de terminaison de messagerie

1.  Dans Azure Bot → **Configuration**
2.  Définissez **Point de terminaison de messagerie** sur votre URL de webhook :
    -   Production : `https://votre-domaine.com/api/messages`
    -   Développement local : Utilisez un tunnel (voir [Développement local](#local-development-tunneling) ci-dessous)

### Étape 4 : Activer le canal Teams

1.  Dans Azure Bot → **Canaux**
2.  Cliquez sur **Microsoft Teams** → Configurer → Enregistrer
3.  Acceptez les Conditions d'utilisation

## Développement local (Tunnel)

Teams ne peut pas atteindre `localhost`. Utilisez un tunnel pour le développement local : **Option A : ngrok**

```bash
ngrok http 3978
# Copiez l'URL https, p. ex. https://abc123.ngrok.io
# Définissez le point de terminaison de messagerie sur : https://abc123.ngrok.io/api/messages
```

**Option B : Tailscale Funnel**

```bash
tailscale funnel 3978
# Utilisez votre URL Tailscale Funnel comme point de terminaison de messagerie
```

## Portail Développeur Teams (Alternative)

Au lieu de créer manuellement un manifeste ZIP, vous pouvez utiliser le [Portail Développeur Teams](https://dev.teams.microsoft.com/apps) :

1.  Cliquez sur **\+ Nouvelle application**
2.  Remplissez les informations de base (nom, description, info développeur)
3.  Allez dans **Fonctionnalités de l'application** → **Bot**
4.  Sélectionnez **Saisir un ID de bot manuellement** et collez votre ID d'application Azure Bot
5.  Cochez les scopes : **Personnel**, **Équipe**, **Conversation de groupe**
6.  Cliquez sur **Distribuer** → **Télécharger le package d'application**
7.  Dans Teams : **Applications** → **Gérer vos applications** → **Télécharger une application personnalisée** → sélectionnez le ZIP

C'est souvent plus simple que d'éditer manuellement les manifestes JSON.

## Tester le bot

**Option A : Chat Web Azure (vérifier d'abord le webhook)**

1.  Dans le Portail Azure → votre ressource Azure Bot → **Tester dans le chat Web**
2.  Envoyez un message - vous devriez voir une réponse
3.  Cela confirme que votre point de terminaison webhook fonctionne avant la configuration Teams

**Option B : Teams (après installation de l'application)**

1.  Installez l'application Teams (sideload ou catalogue de l'organisation)
2.  Trouvez le bot dans Teams et envoyez un MP
3.  Vérifiez les journaux de la passerelle pour l'activité entrante

## Configuration (texte uniquement minimal)

1.  **Installez le plugin Microsoft Teams**
    -   Depuis npm : `openclaw plugins install @openclaw/msteams`
    -   Depuis un dépôt local : `openclaw plugins install ./extensions/msteams`
2.  **Enregistrement du bot**
    -   Créez un Azure Bot (voir ci-dessus) et notez :
        -   ID d'application
        -   Secret client (Mot de passe d'application)
        -   ID de locataire (locataire unique)
3.  **Manifeste d'application Teams**
    -   Incluez une entrée `bot` avec `botId = `.
    -   Scopes : `personal`, `team`, `groupChat`.
    -   `supportsFiles: true` (requis pour la gestion des fichiers dans le scope personnel).
    -   Ajoutez les permissions RSC (ci-dessous).
    -   Créez des icônes : `outline.png` (32x32) et `color.png` (192x192).
    -   Zippez les trois fichiers ensemble : `manifest.json`, `outline.png`, `color.png`.
4.  **Configurez OpenClaw**
    
    Copier
    
    ```json
    {
      "msteams": {
        "enabled": true,
        "appId": "<APP_ID>",
        "appPassword": "<APP_PASSWORD>",
        "tenantId": "<TENANT_ID>",
        "webhook": { "port": 3978, "path": "/api/messages" }
      }
    }
    ```
    
    Vous pouvez aussi utiliser des variables d'environnement au lieu des clés de configuration :
    -   `MSTEAMS_APP_ID`
    -   `MSTEAMS_APP_PASSWORD`
    -   `MSTEAMS_TENANT_ID`
5.  **Point de terminaison du bot**
    -   Définissez le Point de terminaison de messagerie d'Azure Bot sur :
        -   `https://<hôte>:3978/api/messages` (ou votre chemin/port choisi).
6.  **Exécutez la passerelle**
    -   Le canal Teams démarre automatiquement lorsque le plugin est installé et que la configuration `msteams` existe avec des identifiants.

## Contexte historique

-   `channels.msteams.historyLimit` contrôle combien de messages récents de canal/groupe sont inclus dans l'invite.
-   Reprend `messages.groupChat.historyLimit`. Définissez `0` pour désactiver (par défaut 50).
-   L'historique des MP peut être limité avec `channels.msteams.dmHistoryLimit` (tours utilisateur). Remplacements par utilisateur : `channels.msteams.dms["<user_id>"].historyLimit`.

## Permissions RSC Teams actuelles (Manifeste)

Ce sont les **permissions resourceSpecific existantes** dans notre manifeste d'application Teams. Elles s'appliquent uniquement au sein de l'équipe/conversation où l'application est installée. **Pour les canaux (scope équipe) :**

-   `ChannelMessage.Read.Group` (Application) - recevoir tous les messages de canal sans @mention
-   `ChannelMessage.Send.Group` (Application)
-   `Member.Read.Group` (Application)
-   `Owner.Read.Group` (Application)
-   `ChannelSettings.Read.Group` (Application)
-   `TeamMember.Read.Group` (Application)
-   `TeamSettings.Read.Group` (Application)

**Pour les conversations de groupe :**

-   `ChatMessage.Read.Chat` (Application) - recevoir tous les messages de conversation de groupe sans @mention

## Exemple de manifeste Teams (édité)

Exemple minimal et valide avec les champs requis. Remplacez les ID et URL.

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Votre Organisation",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw dans Teams", "full": "OpenClaw dans Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

### Mises en garde sur le manifeste (champs obligatoires)

-   `bots[].botId` **doit** correspondre à l'ID d'application Azure Bot.
-   `webApplicationInfo.id` **doit** correspondre à l'ID d'application Azure Bot.
-   `bots[].scopes` doit inclure les surfaces que vous prévoyez d'utiliser (`personal`, `team`, `groupChat`).
-   `bots[].supportsFiles: true` est requis pour la gestion des fichiers dans le scope personnel.
-   `authorization.permissions.resourceSpecific` doit inclure la lecture/envoi de canal si vous voulez du trafic de canal.

### Mettre à jour une application existante

Pour mettre à jour une application Teams déjà installée (p. ex. pour ajouter des permissions RSC) :

1.  Mettez à jour votre `manifest.json` avec les nouveaux paramètres
2.  **Incrémentez le champ `version`** (p. ex. `1.0.0` → `1.1.0`)
3.  **Re-zip** le manifeste avec les icônes (`manifest.json`, `outline.png`, `color.png`)
4.  Téléchargez le nouveau zip :
    -   **Option A (Centre d'administration Teams) :** Centre d'administration Teams → Applications Teams → Gérer les applications → trouvez votre application → Télécharger une nouvelle version
    -   **Option B (Sideload) :** Dans Teams → Applications → Gérer vos applications → Télécharger une application personnalisée
5.  **Pour les canaux d'équipe :** Réinstallez l'application dans chaque équipe pour que les nouvelles permissions prennent effet
6.  **Quittez complètement et relancez Teams** (pas seulement fermer la fenêtre) pour effacer les métadonnées d'application en cache

## Capacités : RSC uniquement vs Graph

### Avec Teams RSC uniquement (application installée, pas de permissions Graph API)

Fonctionne :

-   Lire le contenu **texte** des messages de canal.
-   Envoyer le contenu **texte** des messages de canal.
-   Recevoir les pièces jointes de **fichiers en MP**.

Ne fonctionne PAS :

-   Les **contenus d'image ou de fichier** des canaux/groupes (la charge utile ne contient qu'un stub HTML).
-   Télécharger les pièces jointes stockées dans SharePoint/OneDrive.
-   Lire l'historique des messages (au-delà de l'événement webhook en direct).

### Avec Teams RSC + permissions d'application Microsoft Graph

Ajoute :

-   Télécharger les contenus hébergés (images collées dans les messages).
-   Télécharger les pièces jointes de fichiers stockées dans SharePoint/OneDrive.
-   Lire l'historique des messages de canal/conversation via Graph.

### RSC vs Graph API

| Capacité | Permissions RSC | Graph API |
| --- | --- | --- |
| **Messages en temps réel** | Oui (via webhook) | Non (sondage uniquement) |
| **Messages historiques** | Non | Oui (peut interroger l'historique) |
| **Complexité de configuration** | Manifeste d'application uniquement | Nécessite consentement admin + flux de jeton |
| **Fonctionne hors ligne** | Non (doit être en cours d'exécution) | Oui (interroger à tout moment) |

**Conclusion :** RSC est pour l'écoute en temps réel ; Graph API est pour l'accès historique. Pour rattraper les messages manqués hors ligne, vous avez besoin de Graph API avec `ChannelMessage.Read.All` (nécessite consentement admin).

## Médias et historique activés par Graph (requis pour les canaux)

Si vous avez besoin d'images/fichiers dans les **canaux** ou voulez récupérer l'**historique des messages**, vous devez activer les permissions Microsoft Graph et accorder le consentement administrateur.

1.  Dans Entra ID (Azure AD) **Inscription d'application**, ajoutez les **permissions d'application** Microsoft Graph :
    -   `ChannelMessage.Read.All` (pièces jointes de canal + historique)
    -   `Chat.Read.All` ou `ChatMessage.Read.All` (conversations de groupe)
2.  **Accordez le consentement administrateur** pour le locataire.
3.  Augmentez la **version du manifeste** de l'application Teams, retéléchargez, et **réinstallez l'application dans Teams**.
4.  **Quittez complètement et relancez Teams** pour effacer les métadonnées d'application en cache.

**Permission supplémentaire pour les mentions d'utilisateur :** Les mentions @ d'utilisateur fonctionnent nativement pour les utilisateurs dans la conversation. Cependant, si vous voulez rechercher dynamiquement et mentionner des utilisateurs qui **ne sont pas dans la conversation actuelle**, ajoutez la permission `User.Read.All` (Application) et accordez le consentement administrateur.

## Limitations connues

### Délais d'attente du webhook

Teams délivre les messages via un webhook HTTP. Si le traitement prend trop de temps (p. ex. réponses LLM lentes), vous pouvez voir :

-   Délais d'attente de la passerelle
-   Teams retente le message (provoquant des doublons)
-   Réponses perdues

OpenClaw gère cela en répondant rapidement et en envoyant des réponses de manière proactive, mais les réponses très lentes peuvent encore causer des problèmes.

### Formatage

Le markdown Teams est plus limité que Slack ou Discord :

-   Le formatage de base fonctionne : **gras**, *italique*, `code`, liens
-   Le markdown complexe (tableaux, listes imbriquées) peut ne pas s'afficher correctement
-   Les cartes adaptatives sont prises en charge pour les sondages et l'envoi de cartes arbitraires (voir ci-dessous)

## Configuration

Paramètres clés (voir `/gateway/configuration` pour les modèles de canal partagés) :

-   `channels.msteams.enabled` : activer/désactiver le canal.
-   `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId` : identifiants du bot.
-   `channels.msteams.webhook.port` (par défaut `3978`)
-   `channels.msteams.webhook.path` (par défaut `/api/messages`)
-   `channels.msteams.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : pairing)
-   `channels.msteams.allowFrom` : liste d'autorisation MP (ID d'objet AAD recommandés). L'assistant résout les noms en ID pendant la configuration lorsque l'accès Graph est disponible.
-   `channels.msteams.dangerouslyAllowNameMatching` : interrupteur de secours pour réactiver la correspondance avec les UPN/noms d'affichage mutables.
-   `channels.msteams.textChunkLimit` : taille des morceaux de texte sortants.
-   `channels.msteams.chunkMode` : `length` (par défaut) ou `newline` pour diviser sur les lignes vides (limites de paragraphe) avant le découpage par longueur.
-   `channels.msteams.mediaAllowHosts` : liste d'autorisation pour les hôtes de pièces jointes entrantes (par défaut les domaines Microsoft/Teams).
-   `channels.msteams.mediaAuthAllowHosts` : liste d'autorisation pour attacher des en-têtes Authorization lors des nouvelles tentatives de média (par défaut les hôtes Graph + Bot Framework).
-   `channels.msteams.requireMention` : exiger une @mention dans les canaux/groupes (par défaut true).
-   `channels.msteams.replyStyle` : `thread | top-level` (voir [Style de réponse](#reply-style-threads-vs-posts)).
-   `channels.msteams.teams..replyStyle` : remplacement par équipe.
-   `channels.msteams.teams..requireMention` : remplacement par équipe.
-   `channels.msteams.teams..tools` : remplacements de politique d'outils par équipe par défaut (`allow`/`deny`/`alsoAllow`) utilisés lorsqu'un remplacement de canal est manquant.
-   `channels.msteams.teams..toolsBySender` : remplacements de politique d'outils par expéditeur par équipe par défaut (support du caractère générique `"*"`).
-   `channels.msteams.teams..channels..replyStyle` : remplacement par canal.
-   `channels.msteams.teams..channels..requireMention` : remplacement par canal.
-   `channels.msteams.teams..channels..tools` : remplacements de politique d'outils par canal (`allow`/`deny`/`alsoAllow`).
-   `channels.msteams.teams..channels..toolsBySender` : remplacements de politique d'outils par expéditeur par canal (support du caractère générique `"*"`).
-   Les clés `toolsBySender` doivent utiliser des préfixes explicites : `id:`, `e164:`, `username:`, `name:` (les clés non préfixées héritées correspondent toujours uniquement à `id:`).
-   `channels.msteams.sharePointSiteId` : ID de site SharePoint pour les téléversements de fichiers dans les conversations de groupe/canaux (voir [Envoi de fichiers dans les conversations de groupe](#sending-files-in-group-chats)).

## Routage et Sessions

-   Les clés de session suivent le format d'agent standard (voir [/concepts/session](../concepts/session.md)) :
    -   Les messages directs partagent la session principale (`agent::`).
    -   Les messages de canal/groupe utilisent l'ID de conversation :
        -   `agent::msteams:channel:`
        -   `agent::msteams:group:`

## Style de réponse : Threads vs Posts

Teams a récemment introduit deux styles d'interface utilisateur de canal sur le même modèle de données sous-jacent :

| Style | Description | `replyStyle` recommandé |
| --- | --- | --- |
| **Posts** (classique) | Les messages apparaissent comme des cartes avec des réponses en filigrane en dessous | `thread` (par défaut) |
| **Threads** (style Slack) | Les messages s'écoulent linéairement, plus comme Slack | `top-level` |

**Le problème :** L'API Teams n'expose pas le style d'interface utilisateur utilisé par un canal. Si vous utilisez le mauvais `replyStyle` :

-   `thread` dans un canal de style Threads → les réponses apparaissent imbriquées bizarrement
-   `top-level` dans un canal de style Posts → les réponses apparaissent comme des posts de premier niveau séparés au lieu d'être dans le fil

**Solution :** Configurez `replyStyle` par canal en fonction de la configuration du canal :

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

## Pièces jointes et Images

**Limitations actuelles :**

-   **MP :** Les images et pièces jointes fonctionnent via les API de fichiers du bot Teams.
-   **Canaux/groupes :** Les pièces jointes résident dans le stockage M365 (SharePoint/OneDrive). La charge utile du webhook ne contient qu'un stub HTML, pas les octets réels du fichier. **Les permissions Graph API sont requises** pour télécharger les pièces jointes de canal.

Sans permissions Graph, les messages de canal avec images seront reçus en texte uniquement (le contenu de l'image n'est pas accessible au bot). Par défaut, OpenClaw ne télécharge les médias que depuis les noms d'hôte Microsoft/Teams. Remplacez avec `channels.msteams.mediaAllowHosts` (utilisez `["*"]` pour autoriser tout hôte). Les en-têtes Authorization ne sont attachés que pour les hôtes dans `channels.msteams.mediaAuthAllowHosts` (par défaut les hôtes Graph + Bot Framework). Gardez cette liste stricte (évitez les suffixes multi-locataires).

## Envoi de fichiers dans les conversations de groupe

Les bots peuvent envoyer des fichiers en MP en utilisant le flux FileConsentCard (intégré). Cependant, **l'envoi de fichiers dans les conversations de groupe/canaux** nécessite une configuration supplémentaire :

| Contexte | Comment les fichiers sont envoyés | Configuration nécessaire |
| --- | --- | --- |
| **MP** | FileConsentCard → l'utilisateur accepte → le bot téléverse | Fonctionne nativement |
| **Conversations de groupe/canaux** | Téléversement vers SharePoint → lien de partage | Requiert `sharePointSiteId` + permissions Graph |
| **Images (tout contexte)** | Encodage Base64 en ligne | Fonctionne nativement |

### Pourquoi les conversations de groupe ont besoin de SharePoint

Les bots n'ont pas de lecteur OneDrive personnel (le point de terminaison Graph API `/me/drive` ne fonctionne pas pour les identités d'application). Pour envoyer des fichiers dans les conversations de groupe/canaux, le bot téléverse vers un **site SharePoint** et crée un lien de partage.

### Configuration

1.  **Ajoutez des permissions Graph API** dans Entra ID (Azure AD) → Inscription d'application :
    -   `Sites.ReadWrite.All` (Application) - téléverser des fichiers vers SharePoint
    -   `Chat.Read.All` (Application) - optionnel, active les liens de partage par utilisateur
2.  **Accordez le consentement administrateur** pour le locataire.
3.  **Obtenez votre ID de site SharePoint :**
    
    Copier
    
    ```bash
    # Via Graph Explorer ou curl avec un jeton valide :
    curl -H "Authorization: Bearer $TOKEN" \
      "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"
    
    # Exemple : pour un site à "contoso.sharepoint.com/sites/BotFiles"
    curl -H "Authorization: Bearer $TOKEN" \
      "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"
    
    # La réponse inclut : "id": "contoso.sharepoint.com,guid1,guid2"
    ```
    
4.  **Configurez OpenClaw :**
    
    Copier
    
    ```json
    {
      channels: {
        msteams: {
          // ... autre configuration ...
          sharePointSiteId: "contoso.sharepoint.com,guid1,guid2",
        },
      },
    }
    ```
    

### Comportement de partage

| Permission | Comportement de partage |
| --- | --- |
| `Sites.ReadWrite.All` uniquement | Lien de partage organisationnel (toute personne dans l'organisation peut accéder) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | Lien de partage par utilisateur (seuls les membres de la conversation peuvent accéder) |

Le partage par utilisateur est plus sécurisé car seuls les participants à la conversation peuvent accéder au fichier. Si la permission `Chat.Read.All` est manquante, le bot revient au partage organisationnel.

### Comportement de repli

| Scénario | Résultat |
| --- | --- |
| Conversation de groupe + fichier + `sharePointSiteId` configuré | Téléversement vers SharePoint, envoi du lien de partage |
| Conversation de groupe + fichier