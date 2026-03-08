

  Plateformes de messagerie

  
# BlueBubbles

Statut : plugin intégré qui communique avec le serveur BlueBubbles macOS via HTTP. **Recommandé pour l'intégration iMessage** en raison de son API plus riche et de sa configuration plus facile par rapport à l'ancien canal imsg.

## Vue d'ensemble

-   Fonctionne sur macOS via l'application d'aide BlueBubbles ([bluebubbles.app](https://bluebubbles.app)).
-   Recommandé/testé : macOS Sequoia (15). macOS Tahoe (26) fonctionne ; l'édition est actuellement cassée sur Tahoe, et les mises à jour d'icône de groupe peuvent signaler un succès mais ne pas se synchroniser.
-   OpenClaw communique avec lui via son API REST (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
-   Les messages entrants arrivent via des webhooks ; les réponses sortantes, les indicateurs de saisie, les accusés de lecture et les tapbacks sont des appels REST.
-   Les pièces jointes et les autocollants sont ingérés en tant que médias entrants (et exposés à l'agent lorsque possible).
-   L'appairage/la liste blanche fonctionne de la même manière que les autres canaux (`/channels/pairing` etc) avec `channels.bluebubbles.allowFrom` + codes d'appairage.
-   Les réactions sont exposées comme des événements système, tout comme Slack/Telegram, afin que les agents puissent les "mentionner" avant de répondre.
-   Fonctionnalités avancées : édition, annulation d'envoi, réponse en fil, effets de message, gestion de groupe.

## Démarrage rapide

1.  Installez le serveur BlueBubbles sur votre Mac (suivez les instructions sur [bluebubbles.app/install](https://bluebubbles.app/install)).
2.  Dans la configuration de BlueBubbles, activez l'API web et définissez un mot de passe.
3.  Exécutez `openclaw onboard` et sélectionnez BlueBubbles, ou configurez manuellement :
    
    Copier
    
    ```json
    {
      channels: {
        bluebubbles: {
          enabled: true,
          serverUrl: "http://192.168.1.100:1234",
          password: "example-password",
          webhookPath: "/bluebubbles-webhook",
        },
      },
    }
    ```
    
4.  Pointez les webhooks BlueBubbles vers votre passerelle (exemple : `https://your-gateway-host:3000/bluebubbles-webhook?password=`).
5.  Démarrez la passerelle ; elle enregistrera le gestionnaire de webhook et commencera l'appairage.

Note de sécurité :

-   Définissez toujours un mot de passe pour le webhook.
-   L'authentification du webhook est toujours requise. OpenClaw rejette les requêtes de webhook BlueBubbles à moins qu'elles n'incluent un mot de passe/guid correspondant à `channels.bluebubbles.password` (par exemple `?password=` ou `x-password`), indépendamment de la topologie de boucle locale/proxy.
-   L'authentification par mot de passe est vérifiée avant la lecture/l'analyse complète des corps des webhooks.

## Garder Messages.app actif (configurations VM / sans interface)

Certaines configurations de VM macOS / toujours actives peuvent finir avec Messages.app en état "inactif" (les événements entrants s'arrêtent jusqu'à ce que l'application soit ouverte/mise au premier plan). Une solution simple consiste à **stimuler Messages toutes les 5 minutes** en utilisant un AppleScript + LaunchAgent.

### 1) Enregistrez l'AppleScript

Enregistrez ceci sous :

-   `~/Scripts/poke-messages.scpt`

Exemple de script (non interactif ; ne vole pas le focus) :

```
try
  tell application "Messages"
    if not running then
      launch
    end if

    -- Touchez l'interface de script pour garder le processus réactif.
    set _chatCount to (count of chats)
  end tell
on error
  -- Ignorez les échecs transitoires (invites de premier lancement, session verrouillée, etc.).
end try
```

### 2) Installez un LaunchAgent

Enregistrez ceci sous :

-   `~/Library/LaunchAgents/com.user.poke-messages.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>

    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript &quot;$HOME/Scripts/poke-messages.scpt&quot;</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>StartInterval</key>
    <integer>300</integer>

    <key>StandardOutPath</key>
    <string>/tmp/poke-messages.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/poke-messages.err</string>
  </dict>
</plist>
```

Notes :

-   Ceci s'exécute **toutes les 300 secondes** et **à la connexion**.
-   Le premier lancement peut déclencher des invites **d'Automatisation** macOS (`osascript` → Messages). Approuvez-les dans la même session utilisateur qui exécute le LaunchAgent.

Chargez-le :

```bash
launchctl unload ~/Library/LaunchAgents/com.user.poke-messages.plist 2>/dev/null || true
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

## Intégration

BlueBubbles est disponible dans l'assistant de configuration interactif :

```bash
openclaw onboard
```

L'assistant demande :

-   **URL du serveur** (requis) : Adresse du serveur BlueBubbles (par ex., `http://192.168.1.100:1234`)
-   **Mot de passe** (requis) : Mot de passe API des paramètres du serveur BlueBubbles
-   **Chemin du webhook** (optionnel) : Par défaut `/bluebubbles-webhook`
-   **Politique des MP** : appairage, liste blanche, ouvert ou désactivé
-   **Liste autorisée** : Numéros de téléphone, emails ou cibles de chat

Vous pouvez également ajouter BlueBubbles via CLI :

```bash
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## Contrôle d'accès (MP + groupes)

MP :

-   Par défaut : `channels.bluebubbles.dmPolicy = "pairing"`.
-   Les expéditeurs inconnus reçoivent un code d'appairage ; les messages sont ignorés jusqu'à approbation (les codes expirent après 1 heure).
-   Approuvez via :
    -   `openclaw pairing list bluebubbles`
    -   `openclaw pairing approve bluebubbles `
-   L'appairage est l'échange de jetons par défaut. Détails : [Appairage](./pairing.md)

Groupes :

-   `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (par défaut : `allowlist`).
-   `channels.bluebubbles.groupAllowFrom` contrôle qui peut déclencher dans les groupes lorsque `allowlist` est défini.

### Gestion des mentions (groupes)

BlueBubbles prend en charge la gestion des mentions pour les conversations de groupe, correspondant au comportement iMessage/WhatsApp :

-   Utilise `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`) pour détecter les mentions.
-   Lorsque `requireMention` est activé pour un groupe, l'agent ne répond que lorsqu'il est mentionné.
-   Les commandes de contrôle des expéditeurs autorisés contournent la gestion des mentions.

Configuration par groupe :

```json
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }, // valeur par défaut pour tous les groupes
        "iMessage;-;chat123": { requireMention: false }, // remplacement pour un groupe spécifique
      },
    },
  },
}
```

### Gestion des commandes

-   Les commandes de contrôle (par ex., `/config`, `/model`) nécessitent une autorisation.
-   Utilise `allowFrom` et `groupAllowFrom` pour déterminer l'autorisation des commandes.
-   Les expéditeurs autorisés peuvent exécuter des commandes de contrôle même sans mention dans les groupes.

## Indicateurs de saisie + accusés de lecture

-   **Indicateurs de saisie** : Envoyés automatiquement avant et pendant la génération de la réponse.
-   **Accusés de lecture** : Contrôlés par `channels.bluebubbles.sendReadReceipts` (par défaut : `true`).
-   **Indicateurs de saisie** : OpenClaw envoie des événements de début de saisie ; BlueBubbles efface la saisie automatiquement à l'envoi ou au timeout (l'arrêt manuel via DELETE n'est pas fiable).

```json
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false, // désactiver les accusés de lecture
    },
  },
}
```

## Actions avancées

BlueBubbles prend en charge les actions de message avancées lorsqu'elles sont activées dans la configuration :

```json
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true, // tapbacks (par défaut : true)
        edit: true, // éditer les messages envoyés (macOS 13+, cassé sur macOS 26 Tahoe)
        unsend: true, // annuler l'envoi de messages (macOS 13+)
        reply: true, // réponse en fil par GUID de message
        sendWithEffect: true, // effets de message (slam, loud, etc.)
        renameGroup: true, // renommer les conversations de groupe
        setGroupIcon: true, // définir l'icône/photo du groupe de discussion (instable sur macOS 26 Tahoe)
        addParticipant: true, // ajouter des participants aux groupes
        removeParticipant: true, // retirer des participants des groupes
        leaveGroup: true, // quitter les conversations de groupe
        sendAttachment: true, // envoyer des pièces jointes/médias
      },
    },
  },
}
```

Actions disponibles :

-   **react** : Ajouter/supprimer des réactions tapback (`messageId`, `emoji`, `remove`)
-   **edit** : Éditer un message envoyé (`messageId`, `text`)
-   **unsend** : Annuler l'envoi d'un message (`messageId`)
-   **reply** : Répondre à un message spécifique (`messageId`, `text`, `to`)
-   **sendWithEffect** : Envoyer avec un effet iMessage (`text`, `to`, `effectId`)
-   **renameGroup** : Renommer une conversation de groupe (`chatGuid`, `displayName`)
-   **setGroupIcon** : Définir l'icône/photo d'un groupe de discussion (`chatGuid`, `media`) — instable sur macOS 26 Tahoe (l'API peut renvoyer un succès mais l'icône ne se synchronise pas).
-   **addParticipant** : Ajouter quelqu'un à un groupe (`chatGuid`, `address`)
-   **removeParticipant** : Retirer quelqu'un d'un groupe (`chatGuid`, `address`)
-   **leaveGroup** : Quitter une conversation de groupe (`chatGuid`)
-   **sendAttachment** : Envoyer des médias/fichiers (`to`, `buffer`, `filename`, `asVoice`)
    -   Mémos vocaux : définissez `asVoice: true` avec un audio **MP3** ou **CAF** pour envoyer en tant que message vocal iMessage. BlueBubbles convertit MP3 → CAF lors de l'envoi de mémos vocaux.

### IDs de message (court vs complet)

OpenClaw peut exposer des IDs de message *courts* (par ex., `1`, `2`) pour économiser des jetons.

-   `MessageSid` / `ReplyToId` peuvent être des IDs courts.
-   `MessageSidFull` / `ReplyToIdFull` contiennent les IDs complets du fournisseur.
-   Les IDs courts sont en mémoire ; ils peuvent expirer au redémarrage ou à l'éviction du cache.
-   Les actions acceptent un `messageId` court ou complet, mais les IDs courts généreront une erreur s'ils ne sont plus disponibles.

Utilisez les IDs complets pour les automatisations durables et le stockage :

-   Modèles : `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
-   Contexte : `MessageSidFull` / `ReplyToIdFull` dans les charges utiles entrantes

Voir [Configuration](../gateway/configuration.md) pour les variables de modèle.

## Diffusion par blocs

Contrôlez si les réponses sont envoyées en un seul message ou diffusées par blocs :

```json
{
  channels: {
    bluebubbles: {
      blockStreaming: true, // activer la diffusion par blocs (désactivée par défaut)
    },
  },
}
```

## Médias + limites

-   Les pièces jointes entrantes sont téléchargées et stockées dans le cache média.
-   Limite média via `channels.bluebubbles.mediaMaxMb` pour les médias entrants et sortants (par défaut : 8 Mo).
-   Le texte sortant est découpé selon `channels.bluebubbles.textChunkLimit` (par défaut : 4000 caractères).

## Référence de configuration

Configuration complète : [Configuration](../gateway/configuration.md) Options du fournisseur :

-   `channels.bluebubbles.enabled` : Activer/désactiver le canal.
-   `channels.bluebubbles.serverUrl` : URL de base de l'API REST BlueBubbles.
-   `channels.bluebubbles.password` : Mot de passe API.
-   `channels.bluebubbles.webhookPath` : Chemin de l'endpoint webhook (par défaut : `/bluebubbles-webhook`).
-   `channels.bluebubbles.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : `pairing`).
-   `channels.bluebubbles.allowFrom` : Liste blanche des MP (identifiants, emails, numéros E.164, `chat_id:*`, `chat_guid:*`).
-   `channels.bluebubbles.groupPolicy` : `open | allowlist | disabled` (par défaut : `allowlist`).
-   `channels.bluebubbles.groupAllowFrom` : Liste blanche des expéditeurs de groupe.
-   `channels.bluebubbles.groups` : Configuration par groupe (`requireMention`, etc.).
-   `channels.bluebubbles.sendReadReceipts` : Envoyer des accusés de lecture (par défaut : `true`).
-   `channels.bluebubbles.blockStreaming` : Activer la diffusion par blocs (par défaut : `false` ; requis pour les réponses en diffusion).
-   `channels.bluebubbles.textChunkLimit` : Taille des morceaux sortants en caractères (par défaut : 4000).
-   `channels.bluebubbles.chunkMode` : `length` (par défaut) découpe uniquement lorsque `textChunkLimit` est dépassé ; `newline` découpe sur les lignes vides (limites de paragraphe) avant le découpage par longueur.
-   `channels.bluebubbles.mediaMaxMb` : Limite média entrant/sortant en Mo (par défaut : 8).
-   `channels.bluebubbles.mediaLocalRoots` : Liste blanche explicite des répertoires locaux absolus autorisés pour les chemins de médias locaux sortants. Les envois de chemins locaux sont refusés par défaut sauf si ceci est configuré. Remplacement par compte : `channels.bluebubbles.accounts..mediaLocalRoots`.
-   `channels.bluebubbles.historyLimit` : Nombre maximum de messages de groupe pour le contexte (0 désactive).
-   `channels.bluebubbles.dmHistoryLimit` : Limite d'historique des MP.
-   `channels.bluebubbles.actions` : Activer/désactiver des actions spécifiques.
-   `channels.bluebubbles.accounts` : Configuration multi-comptes.

Options globales connexes :

-   `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`).
-   `messages.responsePrefix`.

## Adressage / cibles de livraison

Préférez `chat_guid` pour un routage stable :

-   `chat_guid:iMessage;-;+15555550123` (préféré pour les groupes)
-   `chat_id:123`
-   `chat_identifier:...`
-   Identifiants directs : `+15555550123`, `user@example.com`
    -   Si un identifiant direct n'a pas de conversation MP existante, OpenClaw en créera une via `POST /api/v1/chat/new`. Ceci nécessite que l'API Privée BlueBubbles soit activée.

## Sécurité

-   Les requêtes de webhook sont authentifiées en comparant les paramètres de requête ou en-têtes `guid`/`password` avec `channels.bluebubbles.password`. Les requêtes provenant de `localhost` sont également acceptées.
-   Gardez le mot de passe API et l'endpoint webhook secrets (traitez-les comme des identifiants).
-   La confiance localhost signifie qu'un proxy inverse sur la même machine peut contourner involontairement le mot de passe. Si vous proxyfiez la passerelle, exigez une authentification au niveau du proxy et configurez `gateway.trustedProxies`. Voir [Sécurité de la passerelle](../gateway/security.md#reverse-proxy-configuration).
-   Activez HTTPS + règles de pare-feu sur le serveur BlueBubbles si vous l'exposez en dehors de votre LAN.

## Dépannage

-   Si les événements de saisie/lecture cessent de fonctionner, vérifiez les journaux de webhook BlueBubbles et assurez-vous que le chemin de la passerelle correspond à `channels.bluebubbles.webhookPath`.
-   Les codes d'appairage expirent après une heure ; utilisez `openclaw pairing list bluebubbles` et `openclaw pairing approve bluebubbles `.
-   Les réactions nécessitent l'API privée BlueBubbles (`POST /api/v1/message/react`) ; assurez-vous que la version du serveur l'expose.
-   L'édition/l'annulation d'envoi nécessitent macOS 13+ et une version compatible du serveur BlueBubbles. Sur macOS 26 (Tahoe), l'édition est actuellement cassée en raison de changements de l'API privée.
-   Les mises à jour d'icône de groupe peuvent être instables sur macOS 26 (Tahoe) : l'API peut renvoyer un succès mais la nouvelle icône ne se synchronise pas.
-   OpenClaw masque automatiquement les actions connues comme cassées en fonction de la version macOS du serveur BlueBubbles. Si l'édition apparaît toujours sur macOS 26 (Tahoe), désactivez-la manuellement avec `channels.bluebubbles.actions.edit=false`.
-   Pour les informations d'état/santé : `openclaw status --all` ou `openclaw status --deep`.

Pour la référence générale du flux de travail des canaux, voir [Canaux](../channels.md) et le guide [Plugins](../tools/plugin.md).

[Canaux de discussion](../channels.md)[Discord](./discord.md)