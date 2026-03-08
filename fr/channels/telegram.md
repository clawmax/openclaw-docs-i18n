title: "Configuration et paramétrage du bot Telegram pour OpenClaw AI"
description: "Apprenez à configurer un bot Telegram pour OpenClaw AI. Inclut la configuration du token, les politiques de groupe, le comportement d'exécution et les fonctionnalités avancées comme le streaming et les commandes."
keywords: ["bot telegram", "openclaw", "configuration bot", "grammy", "plateforme de messagerie", "configuration canal", "politique de groupe", "token bot"]
---

  Plateformes de messagerie

  
# Telegram

Statut : prêt pour la production pour les DM de bot + groupes via grammY. Le long polling est le mode par défaut ; le mode webhook est optionnel.

## Configuration rapide

### Étape 1 : Créer le token du bot dans BotFather

Ouvrez Telegram et discutez avec **@BotFather** (vérifiez que le pseudo est exactement `@BotFather`). Exécutez `/newbot`, suivez les invites et sauvegardez le token.

### Étape 2 : Configurer le token et la politique DM

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

Fallback d'environnement : `TELEGRAM_BOT_TOKEN=...` (compte par défaut uniquement). Telegram n'utilise **pas** `openclaw channels login telegram` ; configurez le token dans config/env, puis démarrez la passerelle.

### Étape 3 : Démarrer la passerelle et approuver le premier DM

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Les codes d'appairage expirent après 1 heure.

### Étape 4 : Ajouter le bot à un groupe

Ajoutez le bot à votre groupe, puis définissez `channels.telegram.groups` et `groupPolicy` pour correspondre à votre modèle d'accès.

 

> **ℹ️** L'ordre de résolution du token tient compte du compte. En pratique, les valeurs de configuration priment sur le fallback d'environnement, et `TELEGRAM_BOT_TOKEN` ne s'applique qu'au compte par défaut.

## Paramètres côté Telegram

Les bots Telegram sont par défaut en **Mode de confidentialité**, ce qui limite les messages de groupe qu'ils reçoivent. Si le bot doit voir tous les messages du groupe, soit :

-   désactivez le mode de confidentialité via `/setprivacy`, soit
-   faites du bot un administrateur du groupe.

Lorsque vous basculez le mode de confidentialité, retirez et réajoutez le bot dans chaque groupe pour que Telegram applique le changement.

Le statut d'administrateur est contrôlé dans les paramètres du groupe Telegram. Les bots administrateurs reçoivent tous les messages du groupe, ce qui est utile pour un comportement de groupe toujours actif.

-   `/setjoingroups` pour autoriser/interdire les ajouts de groupe
-   `/setprivacy` pour le comportement de visibilité dans les groupes

## Contrôle d'accès et activation

/getUpdates"', lang: 'bash' }, { label: 'Politique de groupe et listes autorisées', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "-1001234567890": {\n          groupPolicy: "open",\n          requireMention: false,\n        },\n      },\n    },\n  },\n}', lang: 'json' }, { label: 'Comportement de mention', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "*": { requireMention: false },\n      },\n    },\n  },\n}', lang: 'json' }]} />

## Comportement d'exécution

-   Telegram est détenu par le processus de la passerelle.
-   Le routage est déterministe : les réponses entrantes de Telegram reviennent vers Telegram (le modèle ne choisit pas les canaux).
-   Les messages entrants sont normalisés dans l'enveloppe de canal partagée avec les métadonnées de réponse et les espaces réservés pour les médias.
-   Les sessions de groupe sont isolées par ID de groupe. Les sujets de forum ajoutent `:topic:` pour garder les sujets isolés.
-   Les messages DM peuvent porter un `message_thread_id` ; OpenClaw les route avec des clés de session sensibles au fil de discussion et préserve l'ID du fil pour les réponses.
-   Le long polling utilise grammY runner avec un séquencement par chat/par fil de discussion. La concurrence globale du récepteur runner utilise `agents.defaults.maxConcurrent`.
-   L'API Bot de Telegram ne prend pas en charge les accusés de lecture (`sendReadReceipts` ne s'applique pas).

## Référence des fonctionnalités

OpenClaw peut diffuser des réponses partielles en temps réel :

-   chats directs : streaming natif de brouillons Telegram via `sendMessageDraft`
-   groupes/sujets : message d'aperçu + `editMessageText`

Prérequis :

-   `channels.telegram.streaming` est `off | partial | block | progress` (par défaut : `partial`)
-   `progress` correspond à `partial` sur Telegram (compatibilité avec la nomenclature inter-canaux)
-   les valeurs héritées `channels.telegram.streamMode` et booléenne `streaming` sont automatiquement mappées

Telegram a activé `sendMessageDraft` pour tous les bots dans l'API Bot 9.5 (1er mars 2026). Pour les réponses uniquement textuelles :

-   DM : OpenClaw met à jour le brouillon en place (pas de message d'aperçu supplémentaire)
-   groupe/sujet : OpenClaw garde le même message d'aperçu et effectue une édition finale en place (pas de second message)

Pour les réponses complexes (par exemple des contenus multimédias), OpenClaw revient à une livraison finale normale puis nettoie le message d'aperçu. Le streaming d'aperçu est séparé du streaming par bloc. Lorsque le streaming par bloc est explicitement activé pour Telegram, OpenClaw ignore le flux d'aperçu pour éviter un double streaming. Si le transport natif de brouillon est indisponible/rejeté, OpenClaw revient automatiquement à `sendMessage` + `editMessageText`. Flux de raisonnement uniquement Telegram :

-   `/reasoning stream` envoie le raisonnement vers l'aperçu en direct pendant la génération
-   la réponse finale est envoyée sans le texte de raisonnement

Le texte sortant utilise `parse_mode: "HTML"` de Telegram.

-   Le texte de type Markdown est rendu en HTML sûr pour Telegram.
-   Le HTML brut du modèle est échappé pour réduire les échecs d'analyse de Telegram.
-   Si Telegram rejette le HTML analysé, OpenClaw réessaie en texte brut.

Les aperçus de lien sont activés par défaut et peuvent être désactivés avec `channels.telegram.linkPreview: false`.

L'enregistrement du menu de commandes Telegram est géré au démarrage avec `setMyCommands`. Commandes natives par défaut :

-   `commands.native: "auto"` active les commandes natives pour Telegram

Ajoutez des entrées de menu de commandes personnalisées :

```json
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Sauvegarde Git" },
        { command: "generate", description: "Créer une image" },
      ],
    },
  },
}
```

Règles :

-   les noms sont normalisés (supprimer le `/` initial, minuscules)
-   motif valide : `a-z`, `0-9`, `_`, longueur `1..32`
-   les commandes personnalisées ne peuvent pas remplacer les commandes natives
-   les conflits/duplicatas sont ignorés et journalisés

Notes :

-   les commandes personnalisées sont uniquement des entrées de menu ; elles n'implémentent pas automatiquement un comportement
-   les commandes de plugin/compétence peuvent toujours fonctionner lorsqu'elles sont tapées même si elles ne sont pas affichées dans le menu Telegram

Si les commandes natives sont désactivées, les commandes intégrées sont supprimées. Les commandes personnalisées/plugin peuvent toujours s'enregistrer si configurées. Échec de configuration courant :

-   `setMyCommands failed` signifie généralement que le DNS/HTTPS sortant vers `api.telegram.org` est bloqué.

### Commandes d'appairage d'appareil (plugin device-pair)

Lorsque le plugin `device-pair` est installé :

1.  `/pair` génère un code de configuration
2.  collez le code dans l'application iOS
3.  `/pair approve` approuve la demande en attente la plus récente

Plus de détails : [Appairage](./pairing.md#pair-via-telegram-recommended-for-ios).

Configurez la portée du clavier en ligne :

```json
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

Surcharge par compte :

```json
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

Portées :

-   `off`
-   `dm`
-   `group`
-   `all`
-   `allowlist` (par défaut)

L'héritage `capabilities: ["inlineButtons"]` correspond à `inlineButtons: "all"`. Exemple d'action de message :

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choisissez une option :",
  buttons: [
    [
      { text: "Oui", callback_data: "yes" },
      { text: "Non", callback_data: "no" },
    ],
    [{ text: "Annuler", callback_data: "cancel" }],
  ],
}
```

Les clics de rappel sont transmis à l'agent sous forme de texte : `callback_data: `

Les actions d'outil Telegram incluent :

-   `sendMessage` (`to`, `content`, optionnel `mediaUrl`, `replyToMessageId`, `messageThreadId`)
-   `react` (`chatId`, `messageId`, `emoji`)
-   `deleteMessage` (`chatId`, `messageId`)
-   `editMessage` (`chatId`, `messageId`, `content`)
-   `createForumTopic` (`chatId`, `name`, optionnel `iconColor`, `iconCustomEmojiId`)

Les actions de message de canal exposent des alias ergonomiques (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`, `topic-create`). Contrôles de restriction :

-   `channels.telegram.actions.sendMessage`
-   `channels.telegram.actions.deleteMessage`
-   `channels.telegram.actions.reactions`
-   `channels.telegram.actions.sticker` (par défaut : désactivé)

Note : `edit` et `topic-create` sont actuellement activés par défaut et n'ont pas de bascules `channels.telegram.actions.*` séparées. Sémantique de suppression de réaction : [/tools/reactions](../tools/reactions.md)

Telegram prend en charge les balises de fil de réponse explicites dans la sortie générée :

-   `[[reply_to_current]]` répond au message déclencheur
-   `[[reply_to:]]` répond à un ID de message Telegram spécifique

`channels.telegram.replyToMode` contrôle le traitement :

-   `off` (par défaut)
-   `first`
-   `all`

Note : `off` désactive le fil de réponse implicite. Les balises explicites `[[reply_to_*]]` sont toujours honorées.

Supergroupes de forum :

-   les clés de session de sujet ajoutent `:topic:`
-   les réponses et l'indicateur de frappe ciblent le fil du sujet
-   chemin de configuration du sujet : `channels.telegram.groups..topics.`

Cas particulier du sujet général (`threadId=1`) :

-   les envois de message omettent `message_thread_id` (Telegram rejette `sendMessage(...thread_id=1)`)
-   les actions de frappe incluent toujours `message_thread_id`

Héritage du sujet : les entrées de sujet héritent des paramètres du groupe sauf surcharge (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`). `agentId` est spécifique au sujet et n'hérite pas des valeurs par défaut du groupe. **Routage d'agent par sujet** : Chaque sujet peut router vers un agent différent en définissant `agentId` dans la configuration du sujet. Cela donne à chaque sujet son propre espace de travail, mémoire et session isolés. Exemple :

```json
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "1": { agentId: "main" },      // Sujet général → agent principal
            "3": { agentId: "zu" },        // Sujet dev → agent zu
            "5": { agentId: "coder" }      // Revue de code → agent coder
          }
        }
      }
    }
  }
}
```

Chaque sujet a alors sa propre clé de session : `agent:zu:telegram:group:-1001234567890:topic:3` **Liaison persistante de sujet ACP** : Les sujets de forum peuvent épingler des sessions de harnais ACP via des liaisons ACP typées de haut niveau :

-   `bindings[]` avec `type: "acp"` et `match.channel: "telegram"`

Exemple :

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
    },
  ],
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "42": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

Ceci est actuellement limité aux sujets de forum dans les groupes et supergroupes. **Création de ACP lié au fil depuis le chat** :

-   `/acp spawn  --thread here|auto` peut lier le sujet Telegram actuel à une nouvelle session ACP.
-   Les messages suivants dans le sujet sont routés directement vers la session ACP liée (pas besoin de `/acp steer`).
-   OpenClaw épingle le message de confirmation de création dans le sujet après une liaison réussie.
-   Requiert `channels.telegram.threadBindings.spawnAcpSessions=true`.

Le contexte du modèle inclut :

-   `MessageThreadId`
-   `IsForum`

Comportement des fils DM :

-   les chats privés avec `message_thread_id` conservent le routage DM mais utilisent des clés de session/réponses sensibles au fil.

### Messages audio

Telegram distingue les notes vocales des fichiers audio.

-   par défaut : comportement de fichier audio
-   balise `[[audio_as_voice]]` dans la réponse de l'agent pour forcer l'envoi en note vocale

Exemple d'action de message :

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

### Messages vidéo

Telegram distingue les fichiers vidéo des notes vidéo. Exemple d'action de message :

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

Les notes vidéo ne prennent pas en charge les légendes ; le texte de message fourni est envoyé séparément.

### Autocollants

Gestion des autocollants entrants :

-   WEBP statique : téléchargé et traité (espace réservé `<media:sticker>`)
-   TGS animé : ignoré
-   WEBM vidéo : ignoré

Champs de contexte d'autocollant :

-   `Sticker.emoji`
-   `Sticker.setName`
-   `Sticker.fileId`
-   `Sticker.fileUniqueId`
-   `Sticker.cachedDescription`

Fichier de cache des autocollants :

-   `~/.openclaw/telegram/sticker-cache.json`

Les autocollants sont décrits une fois (si possible) et mis en cache pour réduire les appels de vision répétés. Activez les actions d'autocollant :

```json
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

Action d'envoi d'autocollant :

```json
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Recherchez les autocollants en cache :

```json
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Les réactions Telegram arrivent sous forme de mises à jour `message_reaction` (séparées du contenu du message). Lorsqu'elles sont activées, OpenClaw met en file d'attente des événements système comme :

-   `Réaction Telegram ajoutée : 👍 par Alice (@alice) sur msg 42`

Configuration :

-   `channels.telegram.reactionNotifications` : `off | own | all` (par défaut : `own`)
-   `channels.telegram.reactionLevel` : `off | ack | minimal | extensive` (par défaut : `minimal`)

Notes :

-   `own` signifie uniquement les réactions des utilisateurs aux messages envoyés par le bot (au mieux via le cache des messages envoyés).
-   Les événements de réaction respectent toujours les contrôles d'accès Telegram (`dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`) ; les expéditeurs non autorisés sont ignorés.
-   Telegram ne fournit pas d'ID de fil dans les mises à jour de réaction.
    -   les groupes non-forum sont routés vers la session de chat de groupe
    -   les groupes forum sont routés vers la session du sujet général du groupe (`:topic:1`), pas vers le sujet d'origine exact

`allowed_updates` pour le polling/webhook inclut automatiquement `message_reaction`.

`ackReaction` envoie un émoji d'accusé de réception pendant qu'OpenClaw traite un message entrant. Ordre de résolution :

-   `channels.telegram.accounts..ackReaction`
-   `channels.telegram.ackReaction`
-   `messages.ackReaction`
-   fallback d'émoji d'identité de l'agent (`agents.list[].identity.emoji`, sinon ”👀”)

Notes :

-   Telegram attend un émoji unicode (par exemple ”👀”).
-   Utilisez `""` pour désactiver la réaction pour un canal ou un compte.

Les écritures de configuration de canal sont activées par défaut (`configWrites !== false`). Les écritures déclenchées par Telegram incluent :

-   événements de migration de groupe (`migrate_to_chat_id`) pour mettre à jour `channels.telegram.groups`
-   `/config set` et `/config unset` (nécessite l'activation des commandes)

Désactiver :

```json
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

Par défaut : long polling. Mode webhook :

-   définissez `channels.telegram.webhookUrl`
-   définissez `channels.telegram.webhookSecret` (requis lorsque l'URL webhook est définie)
-   optionnel `channels.telegram.webhookPath` (par défaut `/telegram-webhook`)
-   optionnel `channels.telegram.webhookHost` (par défaut `127.0.0.1`)
-   optionnel `channels.telegram.webhookPort` (par défaut `8787`)

L'écouteur local par défaut pour le mode webhook se lie à `127.0.0.1:8787`. Si votre point de terminaison public diffère, placez un proxy inverse devant et pointez `webhookUrl` vers l'URL publique. Définissez `webhookHost` (par exemple `0.0.0.0`) lorsque vous avez intentionnellement besoin d'un accès externe.

-   `channels.telegram.textChunkLimit` par défaut est 4000.
-   `channels.telegram.chunkMode="newline"` préfère les limites de paragraphe (lignes vides) avant la division par longueur.
-   `channels.telegram.mediaMaxMb` (par défaut 100) limite la taille des médias Telegram entrants et sortants.
-   `channels.telegram.timeoutSeconds` remplace le délai d'attente du client API Telegram (si non défini, la valeur par défaut de grammY s'applique).
-   l'historique de contexte de groupe utilise `channels.telegram.historyLimit` ou `messages.groupChat.historyLimit` (par défaut 50) ; `0` désactive.
-   contrôles d'historique DM :
    -   `channels.telegram.dmHistoryLimit`
    -   `channels.telegram.dms["<user_id>"].historyLimit`
-   la configuration `channels.telegram.retry` s'applique aux helpers d'envoi Telegram (CLI/outils/actions) pour les erreurs API sortantes récupérables.

La cible d'envoi CLI peut être un ID de chat numérique ou un nom d'utilisateur :

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

Les sondages Telegram utilisent `openclaw message poll` et prennent en charge les sujets de forum :

```bash
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "On le livre ?" --poll-option "Oui" --poll-option "Non"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Choisissez une heure" --poll-option "10h" --poll-option "14h" \
  --poll-duration-seconds 300 --poll-public
```

Drapeaux de sondage spécifiques à Telegram :

-   `--poll-duration-seconds` (5-600)
-   `--poll-anonymous`
-   `--poll-public`
-   `--thread-id` pour les sujets de forum (ou utilisez une cible `:topic:`)

Restriction d'action :

-   `channels.telegram.actions.sendMessage=false` désactive les messages Telegram sortants, y compris les sondages
-   `channels.telegram.actions.poll=false` désactive la création de sondages Telegram tout en laissant les envois normaux activés

## Dépannage

-   Si `requireMention=false`, le mode de confidentialité de Telegram doit autoriser une visibilité complète.
    -   BotFather : `/setprivacy` -> Désactiver
    -   puis retirez + réajoutez le bot au groupe
-   `openclaw channels status` avertit lorsque la configuration attend des messages de groupe non mentionnés.
-   `openclaw channels status --probe` peut vérifier les ID de groupe numériques explicites ; le caractère générique `"*"` ne peut pas être sondé pour l'appartenance.
-   test de session rapide : `/activation always`.

-   lorsque `channels.telegram.groups` existe, le groupe doit être listé (ou inclure `"*"`)
-   vérifiez l'appartenance du bot au groupe
-   consultez les journaux : `openclaw logs --follow` pour les raisons d'ignor

-   autorisez votre identité d'expéditeur (appairage et/ou `allowFrom` numérique)
-   l'autorisation de commande s'applique toujours même lorsque la politique de groupe est `open`
-   `setMyCommands failed` indique généralement des problèmes d'accessibilité DNS/HTTPS vers `api.telegram.org`

-   Node 22+ + fetch/proxy personnalisé peut déclencher un comportement d'abandon immédiat si les types AbortSignal ne correspondent pas.
-   Certains hôtes résolvent `api.telegram.org` d'abord en IPv6 ; une sortie IPv6 défectueuse peut causer des échecs intermittents de l'API Telegram.
-   Si les journaux incluent `TypeError: fetch failed` ou `Network request for 'getUpdates' failed!`, OpenClaw retente maintenant ces erreurs réseau récupérables.
-   Sur les hôtes VPS avec une sortie/TLS directe instable, routez les appels API Telegram via `channels.telegram.proxy` :

```yaml
channels:
  telegram:
    proxy: socks5://<user>:<password>@proxy-host:1080
```

-   Node 22+ utilise par défaut `autoSelectFamily=true` (sauf WSL2) et `dnsResultOrder=ipv4first`.
-   Si votre hôte est WSL2 ou fonctionne explicitement mieux avec un comportement IPv4 uniquement, forcez la sélection de famille :

```yaml
channels:
  telegram:
    network:
      autoSelectFamily: false
```

-   Surcharges d'environnement (temporaires) :
    -   `OPENCLAW_TELEGRAM_DISABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_ENABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_DNS_RESULT_ORDER=ipv4first`
-   Validez les réponses DNS :

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

 Plus d'aide : [Dépannage des canaux](./troubleshooting.md).

## Pointeurs de référence de configuration Telegram

Référence principale :

-   `channels.telegram.enabled` : activer/désactiver le démarrage du canal.
-   `channels.telegram.botToken` : token du bot (BotFather).
-   `channels.telegram.tokenFile` : lire le token depuis un chemin de fichier.
-   `channels.telegram.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : pairing).
-   `channels.telegram.allowFrom` : liste autorisée DM (ID utilisateur Telegram numériques). `allowlist` nécessite au moins un ID d'expéditeur. `open` nécessite `"*"`. `openclaw doctor --fix` peut résoudre les entrées héritées `@username` en ID et peut récupérer les entrées de liste autorisée à partir des fichiers de stockage d'appairage dans les flux de migration de liste autorisée.
-   `channels.telegram.actions.poll` : activer ou désactiver la création de sondages Telegram (par défaut : activé ; nécessite toujours `sendMessage`).
-   `channels.telegram.defaultTo` : cible Telegram par défaut utilisée par la CLI `--deliver` lorsqu'aucun `--reply-to` explicite n'est fourni.
-   `channels.telegram.groupPolicy` : `open | allowlist | disabled` (par défaut : allowlist).
-   `channels.telegram.groupAllowFrom` : liste autorisée des expéditeurs de groupe (ID utilisateur Telegram numériques). `openclaw doctor --fix` peut résoudre les entrées héritées `@username` en ID. Les entrées non numériques sont ignorées au moment de l'authentification. L'authentification de groupe n'utilise pas le fallback de stockage d'appairage DM (`2026.2.25+`).
-   Prépondérance multi-compte :
    -   Lorsque deux ID de compte ou plus sont configurés, définissez `channels.telegram.defaultAccount` (ou incluez `channels.telegram.accounts.default`) pour rendre le routage par défaut explicite.
    -   Si aucun n'est défini, OpenClaw revient au premier ID de compte normalisé et `openclaw doctor` avertit.
    -   `channels.telegram.accounts.default.allowFrom` et `channels.telegram.accounts.default.groupAllowFrom` s'appliquent uniquement au compte `default`.
    -   Les comptes nommés héritent de `channels.telegram.allowFrom` et `channels.telegram.groupAllowFrom` lorsque les valeurs au niveau du compte ne sont pas définies.
    -   Les comptes nommés n'héritent pas de `channels.telegram.accounts.default.allowFrom` / `groupAllowFrom`.
-   `channels.telegram.groups` : valeurs par défaut par groupe + liste autorisée (utilisez `"*"` pour les valeurs par défaut globales).
    -   `channels.telegram.groups..groupPolicy` : remplacement par groupe pour groupPolicy (`open | allowlist | disabled`).
    -   `channels.telegram.groups..requireMention` : restriction de mention par défaut.
    -   `channels.telegram.groups..skills` : filtre de compétences (omis = toutes les compétences, vide = aucune).
    -   `channels.telegram.groups..allowFrom` : remplacement de liste autorisée des expéditeurs par groupe.
    -   `channels.telegram.groups..systemPrompt` : invite système supplémentaire pour le groupe.
    -   `channels.telegram.groups..enabled` : désactiver le groupe lorsque `false`.
    -   `channels.telegram.groups..topics..*` : remplacements par sujet (champs de groupe + `agentId` spécifique au sujet).
    -   `channels.telegram.groups..topics..agentId` : router ce sujet vers un agent spécifique (remplace le routage au niveau du groupe et des liaisons).
    -   `channels.telegram.groups..topics..groupPolicy` : remplacement par sujet pour groupPolicy (`open | allowlist | disabled`).
    -   `ch