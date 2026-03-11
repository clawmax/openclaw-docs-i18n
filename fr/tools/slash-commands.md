

  Compétences

  
# Commandes Slash

Les commandes sont gérées par la Gateway. La plupart des commandes doivent être envoyées sous forme de message **autonome** commençant par `/`. La commande bash réservée à l'hôte utilise `! ` (avec `/bash ` comme alias). Il existe deux systèmes liés :

-   **Commandes** : messages autonomes `/...`.
-   **Directives** : `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
    -   Les directives sont supprimées du message avant que le modèle ne le voie.
    -   Dans les messages de chat normaux (pas uniquement des directives), elles sont traitées comme des "indications en ligne" et ne **persistent pas** les paramètres de session.
    -   Dans les messages uniquement composés de directives (le message ne contient que des directives), elles persistent pour la session et répondent avec un accusé de réception.
    -   Les directives ne sont appliquées que pour les **expéditeurs autorisés**. Si `commands.allowFrom` est défini, c'est la seule liste d'autorisation utilisée ; sinon l'autorisation provient des listes d'autorisation/jumelage de canal plus `commands.useAccessGroups`. Les expéditeurs non autorisés voient les directives traitées comme du texte brut.

Il existe aussi quelques **raccourcis en ligne** (expéditeurs autorisés uniquement) : `/help`, `/commands`, `/status`, `/whoami` (`/id`). Ils s'exécutent immédiatement, sont supprimés avant que le modèle ne voie le message, et le texte restant poursuit le flux normal.

## Configuration

```json
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

-   `commands.text` (par défaut `true`) active l'analyse de `/...` dans les messages de chat.
    -   Sur les surfaces sans commandes natives (WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams), les commandes texte fonctionnent toujours même si vous définissez ceci à `false`.
-   `commands.native` (par défaut `"auto"`) enregistre les commandes natives.
    -   Auto : activé pour Discord/Telegram ; désactivé pour Slack (jusqu'à ce que vous ajoutiez des commandes slash) ; ignoré pour les fournisseurs sans support natif.
    -   Définissez `channels.discord.commands.native`, `channels.telegram.commands.native`, ou `channels.slack.commands.native` pour remplacer par fournisseur (booléen ou `"auto"`).
    -   `false` efface les commandes précédemment enregistrées sur Discord/Telegram au démarrage. Les commandes Slack sont gérées dans l'application Slack et ne sont pas supprimées automatiquement.
-   `commands.nativeSkills` (par défaut `"auto"`) enregistre les commandes de **compétence** de manière native quand c'est supporté.
    -   Auto : activé pour Discord/Telegram ; désactivé pour Slack (Slack nécessite de créer une commande slash par compétence).
    -   Définissez `channels.discord.commands.nativeSkills`, `channels.telegram.commands.nativeSkills`, ou `channels.slack.commands.nativeSkills` pour remplacer par fournisseur (booléen ou `"auto"`).
-   `commands.bash` (par défaut `false`) active `! ` pour exécuter des commandes shell hôte (`/bash ` est un alias ; nécessite les listes d'autorisation `tools.elevated`).
-   `commands.bashForegroundMs` (par défaut `2000`) contrôle combien de temps bash attend avant de passer en mode arrière-plan (`0` passe en arrière-plan immédiatement).
-   `commands.config` (par défaut `false`) active `/config` (lit/écrit `openclaw.json`).
-   `commands.debug` (par défaut `false`) active `/debug` (remplacements uniquement à l'exécution).
-   `commands.allowFrom` (optionnel) définit une liste d'autorisation par fournisseur pour l'autorisation des commandes. Quand configuré, c'est la seule source d'autorisation pour les commandes et directives (les listes d'autorisation/jumelage de canal et `commands.useAccessGroups` sont ignorés). Utilisez `"*"` pour une valeur par défaut globale ; les clés spécifiques au fournisseur la remplacent.
-   `commands.useAccessGroups` (par défaut `true`) applique les listes d'autorisation/politiques pour les commandes quand `commands.allowFrom` n'est pas défini.

## Liste des commandes

Texte + natif (quand activé) :

-   `/help`
-   `/commands`
-   `/skill  [entrée]` (exécuter une compétence par son nom)
-   `/status` (afficher le statut actuel ; inclut l'utilisation/quota du fournisseur pour le fournisseur de modèle actuel quand disponible)
-   `/allowlist` (lister/ajouter/supprimer des entrées de liste d'autorisation)
-   `/approve  allow-once|allow-always|deny` (résoudre les invites d'approbation d'exécution)
-   `/context [list|detail|json]` (expliquer le "contexte" ; `detail` montre la taille par fichier + par outil + par compétence + prompt système)
-   `/export-session [chemin]` (alias : `/export`) (exporter la session actuelle en HTML avec le prompt système complet)
-   `/whoami` (afficher votre identifiant d'expéditeur ; alias : `/id`)
-   `/session idle <durée|off>` (gérer l'auto-défocalisation par inactivité pour les liaisons de fil focalisées)
-   `/session max-age <durée|off>` (gérer l'auto-défocalisation par durée maximale stricte pour les liaisons de fil focalisées)
-   `/subagents list|kill|log|info|send|steer|spawn` (inspecter, contrôler, ou lancer des exécutions de sous-agents pour la session actuelle)
-   `/acp spawn|cancel|steer|close|status|set-mode|set|cwd|permissions|timeout|model|reset-options|doctor|install|sessions` (inspecter et contrôler les sessions d'exécution ACP)
-   `/agents` (lister les agents liés au fil pour cette session)
-   `/focus ` (Discord : lier ce fil, ou un nouveau fil, à une cible de session/sous-agent)
-   `/unfocus` (Discord : supprimer la liaison de fil actuelle)
-   `/kill <id|#|all>` (interrompre immédiatement un ou tous les sous-agents en cours d'exécution pour cette session ; pas de message de confirmation)
-   `/steer <id|#> ` (diriger immédiatement un sous-agent en cours d'exécution : en cours d'exécution quand possible, sinon interrompre le travail actuel et redémarrer sur le message de direction)
-   `/tell <id|#> ` (alias pour `/steer`)
-   `/config show|get|set|unset` (persister la configuration sur disque, propriétaire uniquement ; nécessite `commands.config: true`)
-   `/debug show|set|unset|reset` (remplacements à l'exécution, propriétaire uniquement ; nécessite `commands.debug: true`)
-   `/usage off|tokens|full|cost` (pied de page d'utilisation par réponse ou résumé des coûts locaux)
-   `/tts off|always|inbound|tagged|status|provider|limit|summary|audio` (contrôler la TTS ; voir [/tts](../tts.md))
    -   Discord : la commande native est `/voice` (Discord réserve `/tts`) ; le texte `/tts` fonctionne toujours.
-   `/stop`
-   `/restart`
-   `/dock-telegram` (alias : `/dock_telegram`) (basculer les réponses vers Telegram)
-   `/dock-discord` (alias : `/dock_discord`) (basculer les réponses vers Discord)
-   `/dock-slack` (alias : `/dock_slack`) (basculer les réponses vers Slack)
-   `/activation mention|always` (groupes uniquement)
-   `/send on|off|inherit` (propriétaire uniquement)
-   `/reset` ou `/new [modèle]` (indication de modèle optionnelle ; le reste est transmis)
-   `/think <off|minimal|low|medium|high|xhigh>` (choix dynamiques par modèle/fournisseur ; alias : `/thinking`, `/t`)
-   `/verbose on|full|off` (alias : `/v`)
-   `/reasoning on|off|stream` (alias : `/reason` ; quand activé, envoie un message séparé préfixé `Reasoning:` ; `stream` = brouillon Telegram uniquement)
-   `/elevated on|off|ask|full` (alias : `/elev` ; `full` ignore les approbations d'exécution)
-   `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=` (envoyer `/exec` pour afficher l'état actuel)
-   `/model ` (alias : `/models` ; ou `/` depuis `agents.defaults.models.*.alias`)
-   `/queue ` (plus des options comme `debounce:2s cap:25 drop:summarize` ; envoyer `/queue` pour voir les paramètres actuels)
-   `/bash ` (hôte uniquement ; alias pour `! ` ; nécessite `commands.bash: true` + listes d'autorisation `tools.elevated`)

Texte uniquement :

-   `/compact [instructions]` (voir [/concepts/compaction](../concepts/compaction.md))
-   `! ` (hôte uniquement ; une à la fois ; utiliser `!poll` + `!stop` pour les tâches longues)
-   `!poll` (vérifier la sortie / statut ; accepte un `sessionId` optionnel ; `/bash poll` fonctionne aussi)
-   `!stop` (arrêter la tâche bash en cours d'exécution ; accepte un `sessionId` optionnel ; `/bash stop` fonctionne aussi)

Notes :

-   Les commandes acceptent un `:` optionnel entre la commande et les arguments (ex : `/think: high`, `/send: on`, `/help:`).
-   `/new <modèle>` accepte un alias de modèle, `fournisseur/modèle`, ou un nom de fournisseur (correspondance approximative) ; si aucune correspondance, le texte est traité comme corps du message.
-   Pour une répartition complète de l'utilisation par fournisseur, utilisez `openclaw status --usage`.
-   `/allowlist add|remove` nécessite `commands.config=true` et respecte `configWrites` du canal.
-   `/usage` contrôle le pied de page d'utilisation par réponse ; `/usage cost` affiche un résumé des coûts locaux à partir des journaux de session OpenClaw.
-   `/restart` est activé par défaut ; définissez `commands.restart: false` pour le désactiver.
-   Commande native Discord uniquement : `/vc join|leave|status` contrôle les canaux vocaux (nécessite `channels.discord.voice` et les commandes natives ; non disponible en texte).
-   Les commandes de liaison de fil Discord (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`) nécessitent que les liaisons de fil effectives soient activées (`session.threadBindings.enabled` et/ou `channels.discord.threadBindings.enabled`).
-   Référence des commandes ACP et comportement d'exécution : [ACP Agents](./acp-agents.md).
-   `/verbose` est destiné au débogage et à une visibilité supplémentaire ; gardez-le **désactivé** en utilisation normale.
-   Les résumés d'échec d'outil sont toujours affichés quand c'est pertinent, mais le texte détaillé de l'échec n'est inclus que quand `/verbose` est `on` ou `full`.
-   `/reasoning` (et `/verbose`) sont risqués dans les paramètres de groupe : ils peuvent révéler un raisonnement interne ou une sortie d'outil que vous ne souhaitiez pas exposer. Préférez les laisser désactivés, surtout dans les chats de groupe.
-   **Chemin rapide :** les messages uniquement composés de commandes provenant d'expéditeurs autorisés sont traités immédiatement (contourne la file d'attente + le modèle).
-   **Porte de mention de groupe :** les messages uniquement composés de commandes provenant d'expéditeurs autorisés contournent les exigences de mention.
-   **Raccourcis en ligne (expéditeurs autorisés uniquement) :** certaines commandes fonctionnent aussi quand elles sont intégrées dans un message normal et sont supprimées avant que le modèle ne voie le texte restant.
    -   Exemple : `hey /status` déclenche une réponse de statut, et le texte restant poursuit le flux normal.
-   Actuellement : `/help`, `/commands`, `/status`, `/whoami` (`/id`).
-   Les messages uniquement composés de commandes non autorisés sont ignorés silencieusement, et les jetons `/...` en ligne sont traités comme du texte brut.
-   **Commandes de compétence :** les compétences `user-invocable` sont exposées comme commandes slash. Les noms sont assainis en `a-z0-9_` (max 32 caractères) ; les collisions reçoivent des suffixes numériques (ex : `_2`).
    -   `/skill  [entrée]` exécute une compétence par son nom (utile quand les limites de commandes natives empêchent les commandes par compétence).
    -   Par défaut, les commandes de compétence sont transmises au modèle comme une requête normale.
    -   Les compétences peuvent optionnellement déclarer `command-dispatch: tool` pour acheminer la commande directement vers un outil (déterministe, pas de modèle).
    -   Exemple : `/prose` (plugin OpenProse) — voir [OpenProse](../prose.md).
-   **Arguments des commandes natives :** Discord utilise l'autocomplétion pour les options dynamiques (et les menus de boutons quand vous omettez les arguments requis). Telegram et Slack affichent un menu de boutons quand une commande supporte des choix et que vous omettez l'argument.

## Surfaces d'utilisation (ce qui s'affiche où)

-   **Utilisation/quota du fournisseur** (exemple : "Claude 80% restant") apparaît dans `/status` pour le fournisseur de modèle actuel quand le suivi d'utilisation est activé.
-   **Jetons/coût par réponse** est contrôlé par `/usage off|tokens|full` (ajouté aux réponses normales).
-   `/model status` concerne les **modèles/auth/points de terminaison**, pas l'utilisation.

## Sélection de modèle (/model)

`/model` est implémenté comme une directive. Exemples :

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

Notes :

-   `/model` et `/model list` affichent un sélecteur compact numéroté (famille de modèle + fournisseurs disponibles).
-   Sur Discord, `/model` et `/models` ouvrent un sélecteur interactif avec des menus déroulants de fournisseur et de modèle plus une étape de soumission.
-   `/model <#>` sélectionne dans ce sélecteur (et préfère le fournisseur actuel quand c'est possible).
-   `/model status` affiche la vue détaillée, incluant le point de terminaison du fournisseur configuré (`baseUrl`) et le mode API (`api`) quand disponible.

## Remplacements de débogage

`/debug` permet de définir des remplacements de configuration **uniquement à l'exécution** (mémoire, pas disque). Propriétaire uniquement. Désactivé par défaut ; activez avec `commands.debug: true`. Exemples :

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

Notes :

-   Les remplacements s'appliquent immédiatement aux nouvelles lectures de configuration, mais n'écrivent **pas** dans `openclaw.json`.
-   Utilisez `/debug reset` pour effacer tous les remplacements et revenir à la configuration sur disque.

## Mises à jour de configuration

`/config` écrit dans votre configuration sur disque (`openclaw.json`). Propriétaire uniquement. Désactivé par défaut ; activez avec `commands.config: true`. Exemples :

```bash
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

Notes :

-   La configuration est validée avant l'écriture ; les changements invalides sont rejetés.
-   Les mises à jour `/config` persistent après les redémarrages.

## Notes sur les surfaces

-   **Commandes texte** s'exécutent dans la session de chat normale (les MPs partagent `main`, les groupes ont leur propre session).
-   **Commandes natives** utilisent des sessions isolées :
    -   Discord : `agent::discord:slash:`
    -   Slack : `agent::slack:slash:` (préfixe configurable via `channels.slack.slashCommand.sessionPrefix`)
    -   Telegram : `telegram:slash:` (cible la session de chat via `CommandTargetSessionKey`)
-   **`/stop`** cible la session de chat active pour pouvoir interrompre l'exécution en cours.
-   **Slack :** `channels.slack.slashCommand` est toujours supporté pour une seule commande de type `/openclaw`. Si vous activez `commands.native`, vous devez créer une commande slash Slack par commande intégrée (mêmes noms que `/help`). Les menus d'arguments de commande pour Slack sont livrés sous forme de boutons Block Kit éphémères.
    -   Exception native Slack : enregistrez `/agentstatus` (pas `/status`) car Slack réserve `/status`. Le texte `/status` fonctionne toujours dans les messages Slack.

[Créer des compétences](./creating-skills.md)[Compétences](./skills.md)