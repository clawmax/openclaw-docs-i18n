title: "Guide de configuration, d'installation et d'utilisation de l'intégration OpenClaw Slack"
description: "Apprenez à installer et configurer l'intégration OpenClaw Slack. Configurez les jetons, le contrôle d'accès, les commandes, le threading et les actions pour les messages directs et les canaux."
keywords: ["intégration slack", "openclaw slack", "configuration bot slack", "configuration slack", "canaux slack", "commandes slack", "api slack", "mode socket"]
---

  Plateformes de messagerie

  
# Slack

Statut : prêt pour la production pour les messages directs + canaux via les intégrations d'app Slack. Le mode par défaut est le Socket Mode ; le mode HTTP Events API est également pris en charge.

## Installation rapide

## Modèle de jetons

-   `botToken` + `appToken` sont requis pour le Socket Mode.
-   Le mode HTTP nécessite `botToken` + `signingSecret`.
-   Les jetons de configuration écrasent le fallback d'environnement.
-   Le fallback d'environnement `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` s'applique uniquement au compte par défaut.
-   `userToken` (`xoxp-...`) est configurable uniquement (pas de fallback d'environnement) et adopte par défaut un comportement en lecture seule (`userTokenReadOnly: true`).
-   Optionnel : ajoutez `chat:write.customize` si vous souhaitez que les messages sortants utilisent l'identité de l'agent actif (personnalisation du `username` et de l'icône). `icon_emoji` utilise la syntaxe `:nom_emoji:`.

> **💡** Pour les actions/lectures de répertoire, le jeton utilisateur peut être préféré lorsqu'il est configuré. Pour les écritures, le jeton bot reste préféré ; les écritures avec jeton utilisateur ne sont autorisées que lorsque `userTokenReadOnly: false` et que le jeton bot n'est pas disponible.

## Contrôle d'accès et routage

`channels.slack.dmPolicy` contrôle l'accès aux messages directs (ancienne clé : `channels.slack.dm.policy`) :

-   `pairing` (par défaut)
-   `allowlist`
-   `open` (nécessite que `channels.slack.allowFrom` inclue `"*"` ; ancienne clé : `channels.slack.dm.allowFrom`)
-   `disabled`

Indicateurs pour les messages directs :

-   `dm.enabled` (par défaut true)
-   `channels.slack.allowFrom` (préféré)
-   `dm.allowFrom` (ancienne clé)
-   `dm.groupEnabled` (messages directs de groupe par défaut false)
-   `dm.groupChannels` (allowlist optionnelle pour les MPIM)

Précédence multi-comptes :

-   `channels.slack.accounts.default.allowFrom` s'applique uniquement au compte `default`.
-   Les comptes nommés héritent de `channels.slack.allowFrom` lorsque leur propre `allowFrom` n'est pas défini.
-   Les comptes nommés n'héritent pas de `channels.slack.accounts.default.allowFrom`.

L'appairage dans les messages directs utilise `openclaw pairing approve slack `.

`channels.slack.groupPolicy` contrôle la gestion des canaux :

-   `open`
-   `allowlist`
-   `disabled`

L'allowlist des canaux se trouve sous `channels.slack.channels`.
Note d'exécution : si `channels.slack` est complètement absent (configuration uniquement par environnement), l'exécution revient à `groupPolicy="allowlist"` et enregistre un avertissement (même si `channels.defaults.groupPolicy` est défini).
Résolution des noms/ID :

-   les entrées de l'allowlist des canaux et de l'allowlist des messages directs sont résolues au démarrage lorsque l'accès au jeton le permet
-   les entrées non résolues sont conservées telles que configurées
-   la correspondance d'autorisation entrante se fait par ID en premier par défaut ; la correspondance directe par nom d'utilisateur/slug nécessite `channels.slack.dangerouslyAllowNameMatching: true`

Les messages dans les canaux sont par défaut conditionnés par une mention.
Sources de mention :

-   mention explicite de l'application (`<@botId>`)
-   motifs regex de mention (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
-   comportement implicite de réponse au bot dans un fil

Contrôles par canal (`channels.slack.channels.<id|name>`) :

-   `requireMention`
-   `users` (allowlist)
-   `allowBots`
-   `skills`
-   `systemPrompt`
-   `tools`, `toolsBySender`
-   Format de clé `toolsBySender` : `id:`, `e164:`, `username:`, `name:`, ou le joker `"*"` (les anciennes clés non préfixées correspondent toujours uniquement à `id:`)

## Commandes et comportement slash

-   Le mode auto des commandes natives est **désactivé** pour Slack (`commands.native: "auto"` n'active pas les commandes natives Slack).
-   Activez les gestionnaires de commandes natives Slack avec `channels.slack.commands.native: true` (ou globalement `commands.native: true`).
-   Lorsque les commandes natives sont activées, enregistrez les commandes slash correspondantes dans Slack (noms `/`), avec une exception :
    -   enregistrez `/agentstatus` pour la commande status (Slack réserve `/status`)
-   Si les commandes natives ne sont pas activées, vous pouvez exécuter une seule commande slash configurée via `channels.slack.slashCommand`.
-   Les menus d'arguments natifs adaptent maintenant leur stratégie de rendu :
    -   jusqu'à 5 options : blocs de boutons
    -   6-100 options : menu de sélection statique
    -   plus de 100 options : sélection externe avec filtrage asynchrone des options lorsque les gestionnaires d'options d'interactivité sont disponibles
    -   si les valeurs d'option encodées dépassent les limites de Slack, le flux revient aux boutons
-   Pour les charges utiles d'options longues, les menus d'arguments des commandes slash utilisent une boîte de dialogue de confirmation avant d'envoyer une valeur sélectionnée.

Paramètres par défaut des commandes slash :

-   `enabled: false`
-   `name: "openclaw"`
-   `sessionPrefix: "slack:slash"`
-   `ephemeral: true`

Les sessions slash utilisent des clés isolées :

-   `agent::slack:slash:`

et acheminent toujours l'exécution de la commande vers la session de conversation cible (`CommandTargetSessionKey`).

## Threading, sessions et balises de réponse

-   Les messages directs sont routés en tant que `direct` ; les canaux en tant que `channel` ; les MPIM en tant que `group`.
-   Avec la valeur par défaut `session.dmScope=main`, les messages directs Slack sont regroupés dans la session principale de l'agent.
-   Sessions de canal : `agent::slack:channel:`.
-   Les réponses dans un fil peuvent créer des suffixes de session de fil (`:thread:`) le cas échéant.
-   `channels.slack.thread.historyScope` par défaut est `thread` ; `thread.inheritParent` par défaut est `false`.
-   `channels.slack.thread.initialHistoryLimit` contrôle le nombre de messages de fil existants récupérés lorsqu'une nouvelle session de fil démarre (par défaut `20` ; définissez `0` pour désactiver).

Contrôles du threading des réponses :

-   `channels.slack.replyToMode` : `off|first|all` (par défaut `off`)
-   `channels.slack.replyToModeByChatType` : par `direct|group|channel`
-   fallback hérité pour les discussions directes : `channels.slack.dm.replyToMode`

Les balises de réponse manuelles sont prises en charge :

-   `[[reply_to_current]]`
-   `[[reply_to:]]`

Note : `replyToMode="off"` désactive **tous** les threadings de réponse dans Slack, y compris les balises explicites `[[reply_to_*]]`. Cela diffère de Telegram, où les balises explicites sont toujours honorées en mode `"off"`. La différence reflète les modèles de threading des plateformes : les fils Slack masquent les messages du canal, tandis que les réponses Telegram restent visibles dans le flux principal de la discussion.

## Médias, fragmentation et livraison

Les pièces jointes de fichiers Slack sont téléchargées depuis des URL privées hébergées par Slack (flux de requête authentifié par jeton) et écrites dans le stockage de médias lorsque le téléchargement réussit et que les limites de taille le permettent.
La limite de taille entrante par défaut est de `20MB` sauf si elle est écrasée par `channels.slack.mediaMaxMb`.

-   les fragments de texte utilisent `channels.slack.textChunkLimit` (par défaut 4000)
-   `channels.slack.chunkMode="newline"` active la division par paragraphe en premier
-   les envois de fichiers utilisent les API d'upload de Slack et peuvent inclure des réponses dans un fil (`thread_ts`)
-   la limite de médias sortants suit `channels.slack.mediaMaxMb` lorsqu'elle est configurée ; sinon, les envois dans les canaux utilisent les valeurs par défaut par type MIME du pipeline média

Cibles explicites préférées :

-   `user:` pour les messages directs
-   `channel:` pour les canaux

Les messages directs Slack sont ouverts via les API de conversation de Slack lors de l'envoi vers des cibles utilisateur.

## Actions et portes

Les actions Slack sont contrôlées par `channels.slack.actions.*`. Groupes d'actions disponibles dans l'outillage Slack actuel :

| Groupe | Par défaut |
| --- | --- |
| messages | activé |
| reactions | activé |
| pins | activé |
| memberInfo | activé |
| emojiList | activé |

## Événements et comportement opérationnel

-   Les modifications/suppressions de messages et les diffusions dans les fils sont mappées en événements système.
-   Les événements d'ajout/suppression de réaction sont mappés en événements système.
-   Les événements d'arrivée/départ de membre, de création/renommage de canal, et d'ajout/suppression d'épinglage sont mappés en événements système.
-   Les mises à jour de statut des fils de l'assistant (pour les indicateurs "est en train d'écrire..." dans les fils) utilisent `assistant.threads.setStatus` et nécessitent la portée bot `assistant:write`.
-   `channel_id_changed` peut migrer les clés de configuration de canal lorsque `configWrites` est activé.
-   Les métadonnées de sujet/objectif du canal sont traitées comme un contexte non fiable et peuvent être injectées dans le contexte de routage.
-   Les actions de bloc et les interactions modales émettent des événements système structurés `Slack interaction: ...` avec des champs de charge utile riches :
    -   actions de bloc : valeurs sélectionnées, étiquettes, valeurs de sélecteur et métadonnées `workflow_*`
    -   événements modaux `view_submission` et `view_closed` avec métadonnées de canal routées et entrées de formulaire

## Réactions d'accusé de réception

`ackReaction` envoie un emoji d'accusé de réception pendant qu'OpenClaw traite un message entrant. Ordre de résolution :

-   `channels.slack.accounts..ackReaction`
-   `channels.slack.ackReaction`
-   `messages.ackReaction`
-   fallback sur l'emoji de l'identité de l'agent (`agents.list[].identity.emoji`, sinon ”👀”)

Notes :

-   Slack attend des codes courts (par exemple `"eyes"`).
-   Utilisez `""` pour désactiver la réaction pour le compte Slack ou globalement.

## Fallback de réaction de saisie

`typingReaction` ajoute une réaction temporaire au message Slack entrant pendant qu'OpenClaw traite une réponse, puis la supprime lorsque l'exécution se termine. C'est un fallback utile lorsque la saisie native de l'assistant Slack n'est pas disponible, notamment dans les messages directs. Ordre de résolution :

-   `channels.slack.accounts..typingReaction`
-   `channels.slack.typingReaction`

Notes :

-   Slack attend des codes courts (par exemple `"hourglass_flowing_sand"`).
-   La réaction est à "best-effort" et le nettoyage est tenté automatiquement après la fin de la réponse ou du chemin d'échec.

## Manifeste et checklist des portées

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Connecteur Slack pour OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Envoyer un message à OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "assistant:write",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

Si vous configurez `channels.slack.userToken`, les portées de lecture typiques sont :

-   `channels:history`, `groups:history`, `im:history`, `mpim:history`
-   `channels:read`, `groups:read`, `im:read`, `mpim:read`
-   `users:read`
-   `reactions:read`
-   `pins:read`
-   `emoji:read`
-   `search:read` (si vous dépendez des lectures de recherche Slack)

## Dépannage

Vérifiez, dans l'ordre :

-   `groupPolicy`
-   l'allowlist des canaux (`channels.slack.channels`)
-   `requireMention`
-   l'allowlist `users` par canal

Commandes utiles :

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

Vérifiez :

-   `channels.slack.dm.enabled`
-   `channels.slack.dmPolicy` (ou l'ancienne clé `channels.slack.dm.policy`)
-   les approbations d'appairage / les entrées de l'allowlist

```bash
openclaw pairing list slack
```

Validez les jetons bot + app et l'activation du Socket Mode dans les paramètres de l'app Slack.

Validez :

-   le secret de signature
-   le chemin du webhook
-   les URL de requête Slack (Événements + Interactivité + Commandes Slash)
-   un `webhookPath` unique par compte HTTP

Vérifiez si vous souhaitiez :

-   le mode commande native (`channels.slack.commands.native: true`) avec les commandes slash correspondantes enregistrées dans Slack
-   ou le mode commande slash unique (`channels.slack.slashCommand.enabled: true`)

Vérifiez également `commands.useAccessGroups` et les allowlists de canal/utilisateur.

## Diffusion de texte en continu

OpenClaw prend en charge la diffusion de texte native Slack via l'API Agents and AI Apps. `channels.slack.streaming` contrôle le comportement de prévisualisation en direct :

-   `off` : désactive la diffusion de prévisualisation en direct.
-   `partial` (par défaut) : remplace le texte de prévisualisation par la dernière sortie partielle.
-   `block` : ajoute des mises à jour de prévisualisation par blocs.
-   `progress` : affiche un texte d'état de progression pendant la génération, puis envoie le texte final.

`channels.slack.nativeStreaming` contrôle l'API de diffusion native de Slack (`chat.startStream` / `chat.appendStream` / `chat.stopStream`) lorsque `streaming` est `partial` (par défaut : `true`). Désactivez la diffusion native Slack (conservez le comportement de prévisualisation brouillon) :

```yaml
channels:
  slack:
    streaming: partial
    nativeStreaming: false
```

Anciennes clés :

-   `channels.slack.streamMode` (`replace | status_final | append`) est automatiquement migré vers `channels.slack.streaming`.
-   le booléen `channels.slack.streaming` est automatiquement migré vers `channels.slack.nativeStreaming`.

### Prérequis

1.  Activez **Agents and AI Apps** dans les paramètres de votre app Slack.
2.  Assurez-vous que l'app a la portée `assistant:write`.
3.  Un fil de réponse doit être disponible pour ce message. La sélection du fil suit toujours `replyToMode`.

### Comportement

-   Le premier fragment de texte démarre un flux (`chat.startStream`).
-   Les fragments de texte suivants s'ajoutent au même flux (`chat.appendStream`).
-   La fin de la réponse finalise le flux (`chat.stopStream`).
-   Les médias et charges utiles non textuelles reviennent à la livraison normale.
-   Si la diffusion échoue en cours de réponse, OpenClaw revient à la livraison normale pour les charges utiles restantes.

## Pointeurs de référence de configuration

Référence principale :

-   [Référence de configuration - Slack](../gateway/configuration-reference.md#slack) Champs Slack à fort signal :
    -   mode/auth : `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
    -   accès aux messages directs : `dm.enabled`, `dmPolicy`, `allowFrom` (anciennes clés : `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
    -   bascule de compatibilité : `dangerouslyAllowNameMatching` (break-glass ; gardez désactivé sauf besoin)
    -   accès aux canaux : `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
    -   threading/historique : `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
    -   livraison : `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `streaming`, `nativeStreaming`
    -   opérations/fonctionnalités : `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Liens connexes

-   [Appairage](./pairing.md)
-   [Routage des canaux](./channel-routing.md)
-   [Dépannage](./troubleshooting.md)
-   [Configuration](../gateway/configuration.md)
-   [Commandes slash](../tools/slash-commands.md)

[Synology Chat](./synology-chat.md)[Telegram](./telegram.md)