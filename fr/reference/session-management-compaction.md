

  Internes de la compaction

  
# Plongée approfondie dans la gestion des sessions

Ce document explique comment OpenClaw gère les sessions de bout en bout :

-   **Routage des sessions** (comment les messages entrants sont mappés à une `sessionKey`)
-   **Magasin de sessions** (`sessions.json`) et ce qu'il suit
-   **Persistance des transcripts** (`*.jsonl`) et sa structure
-   **Hygiène des transcripts** (corrections spécifiques au fournisseur avant les exécutions)
-   **Limites de contexte** (fenêtre de contexte vs jetons suivis)
-   **Compaction** (manuelle + auto-compaction) et où brancher le travail pré-compaction
-   **Tâches de maintenance silencieuses** (par ex. écritures en mémoire qui ne doivent pas produire de sortie visible pour l'utilisateur)

Si vous voulez d'abord un aperçu de haut niveau, commencez par :

-   [/concepts/session](../concepts/session.md)
-   [/concepts/compaction](../concepts/compaction.md)
-   [/concepts/session-pruning](../concepts/session-pruning.md)
-   [/reference/transcript-hygiene](./transcript-hygiene.md)

* * *

## Source de vérité : la Gateway

OpenClaw est conçu autour d'un unique **processus Gateway** qui possède l'état des sessions.

-   Les interfaces utilisateur (application macOS, interface web Control UI, TUI) doivent interroger la Gateway pour les listes de sessions et les compteurs de jetons.
-   En mode distant, les fichiers de session sont sur l'hôte distant ; « vérifier vos fichiers Mac locaux » ne reflétera pas ce que la Gateway utilise.

* * *

## Deux couches de persistance

OpenClaw persiste les sessions en deux couches :

1.  **Magasin de sessions (`sessions.json`)**
    -   Map clé/valeur : `sessionKey -> SessionEntry`
    -   Petit, mutable, sûr à éditer (ou à supprimer des entrées)
    -   Suit les métadonnées de session (id de session actuelle, dernière activité, bascules, compteurs de jetons, etc.)
2.  **Transcript (`.jsonl`)**
    -   Transcript en ajout seul avec une structure arborescente (les entrées ont un `id` + `parentId`)
    -   Stocke la conversation réelle + les appels d'outils + les résumés de compaction
    -   Utilisé pour reconstruire le contexte du modèle pour les tours futurs

* * *

## Emplacements sur disque

Par agent, sur l'hôte Gateway :

-   Magasin : `~/.openclaw/agents//sessions/sessions.json`
-   Transcripts : `~/.openclaw/agents//sessions/.jsonl`
    -   Sessions de sujet Telegram : `.../-topic-.jsonl`

OpenClaw les résout via `src/config/sessions.ts`.

* * *

## Maintenance du magasin et contrôles disque

La persistance des sessions a des contrôles de maintenance automatique (`session.maintenance`) pour `sessions.json` et les artefacts de transcript :

-   `mode` : `warn` (par défaut) ou `enforce`
-   `pruneAfter` : seuil d'âge pour les entrées obsolètes (par défaut `30j`)
-   `maxEntries` : limite le nombre d'entrées dans `sessions.json` (par défaut `500`)
-   `rotateBytes` : rotation de `sessions.json` en cas de surtaille (par défaut `10mo`)
-   `resetArchiveRetention` : rétention pour les archives de transcript `*.reset.` (par défaut : identique à `pruneAfter` ; `false` désactive le nettoyage)
-   `maxDiskBytes` : budget optionnel pour le répertoire des sessions
-   `highWaterBytes` : cible optionnelle après nettoyage (par défaut `80%` de `maxDiskBytes`)

Ordre d'application pour le nettoyage du budget disque (`mode: "enforce"`) :

1.  Supprimer d'abord les artefacts de transcript archivés ou orphelins les plus anciens.
2.  Si toujours au-dessus de la cible, évincer les entrées de session les plus anciennes et leurs fichiers de transcript.
3.  Continuer jusqu'à ce que l'utilisation soit égale ou inférieure à `highWaterBytes`.

En `mode: "warn"`, OpenClaw signale les éventuelles évictions mais ne modifie pas le magasin/les fichiers. Exécutez la maintenance à la demande :

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

* * *

## Sessions cron et journaux d'exécution

Les exécutions cron isolées créent également des entrées de session/transcripts, et elles ont des contrôles de rétention dédiés :

-   `cron.sessionRetention` (par défaut `24h`) élimine les anciennes sessions d'exécution cron isolées du magasin de sessions (`false` désactive).
-   `cron.runLog.maxBytes` + `cron.runLog.keepLines` éliminent les fichiers `~/.openclaw/cron/runs/.jsonl` (par défaut : `2_000_000` octets et `2000` lignes).

* * *

## Clés de session (sessionKey)

Une `sessionKey` identifie *dans quel compartiment de conversation* vous vous trouvez (routage + isolation). Modèles courants :

-   Chat principal/direct (par agent) : `agent::` (par défaut `main`)
-   Groupe : `agent:::group:`
-   Salon/canal (Discord/Slack) : `agent:::channel:` ou `...:room:`
-   Cron : `cron:<job.id>`
-   Webhook : `hook:` (sauf si remplacé)

Les règles canoniques sont documentées sur [/concepts/session](../concepts/session.md).

* * *

## Identifiants de session (sessionId)

Chaque `sessionKey` pointe vers un `sessionId` actuel (le fichier de transcript qui poursuit la conversation). Règles empiriques :

-   **Réinitialisation** (`/new`, `/reset`) crée un nouveau `sessionId` pour cette `sessionKey`.
-   **Réinitialisation quotidienne** (par défaut 4h00 heure locale sur l'hôte gateway) crée un nouveau `sessionId` au prochain message après la limite de réinitialisation.
-   **Expiration par inactivité** (`session.reset.idleMinutes` ou hérité `session.idleMinutes`) crée un nouveau `sessionId` lorsqu'un message arrive après la fenêtre d'inactivité. Lorsque quotidienne et inactivité sont toutes deux configurées, celle qui expire en premier l'emporte.
-   **Garde-fou de bifurcation parente** (`session.parentForkMaxTokens`, par défaut `100000`) ignore la bifurcation du transcript parent lorsque la session parente est déjà trop volumineuse ; le nouveau thread démarre à neuf. Réglez à `0` pour désactiver.

Détail d'implémentation : la décision a lieu dans `initSessionState()` dans `src/auto-reply/reply/session.ts`.

* * *

## Schéma du magasin de sessions (sessions.json)

Le type de valeur du magasin est `SessionEntry` dans `src/config/sessions.ts`. Champs clés (non exhaustif) :

-   `sessionId` : identifiant du transcript actuel (le nom de fichier en est dérivé sauf si `sessionFile` est défini)
-   `updatedAt` : horodatage de la dernière activité
-   `sessionFile` : chemin de transcript explicite optionnel
-   `chatType` : `direct | group | room` (aide pour les interfaces utilisateur et la politique d'envoi)
-   `provider`, `subject`, `room`, `space`, `displayName` : métadonnées pour l'étiquetage des groupes/canaux
-   Bascules :
    -   `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
    -   `sendPolicy` (remplacement par session)
-   Sélection de modèle :
    -   `providerOverride`, `modelOverride`, `authProfileOverride`
-   Compteurs de jetons (au mieux / dépendant du fournisseur) :
    -   `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
-   `compactionCount` : combien de fois l'auto-compaction s'est terminée pour cette clé de session
-   `memoryFlushAt` : horodatage de la dernière purge de mémoire pré-compaction
-   `memoryFlushCompactionCount` : compteur de compaction lors de la dernière exécution de la purge

Le magasin est sûr à éditer, mais la Gateway fait autorité : elle peut réécrire ou réhydrater les entrées lors de l'exécution des sessions.

* * *

## Structure du transcript (\*.jsonl)

Les transcripts sont gérés par `SessionManager` de `@mariozechner/pi-coding-agent`. Le fichier est en JSONL :

-   Première ligne : en-tête de session (`type: "session"`, inclut `id`, `cwd`, `timestamp`, optionnel `parentSession`)
-   Ensuite : entrées de session avec `id` + `parentId` (arbre)

Types d'entrées notables :

-   `message` : messages utilisateur/assistant/toolResult
-   `custom_message` : messages injectés par extension qui *entrent* dans le contexte du modèle (peuvent être masqués de l'interface utilisateur)
-   `custom` : état d'extension qui n'entre *pas* dans le contexte du modèle
-   `compaction` : résumé de compaction persistant avec `firstKeptEntryId` et `tokensBefore`
-   `branch_summary` : résumé persistant lors de la navigation dans une branche d'arbre

OpenClaw ne « corrige » *pas* intentionnellement les transcripts ; la Gateway utilise `SessionManager` pour les lire/écrire.

* * *

## Fenêtres de contexte vs jetons suivis

Deux concepts différents sont importants :

1.  **Fenêtre de contexte du modèle** : plafond dur par modèle (jetons visibles pour le modèle)
2.  **Compteurs du magasin de sessions** : statistiques glissantes écrites dans `sessions.json` (utilisées pour /status et tableaux de bord)

Si vous réglez les limites :

-   La fenêtre de contexte provient du catalogue de modèles (et peut être remplacée via la configuration).
-   `contextTokens` dans le magasin est une estimation/valeur de rapport en temps d'exécution ; ne le traitez pas comme une garantie stricte.

Pour plus d'informations, voir [/token-use](./token-use.md).

* * *

## Compaction : ce que c'est

La compaction résume les conversations plus anciennes en une entrée `compaction` persistante dans le transcript et garde les messages récents intacts. Après compaction, les tours futurs voient :

-   Le résumé de compaction
-   Les messages après `firstKeptEntryId`

La compaction est **persistante** (contrairement à l'élagage de session). Voir [/concepts/session-pruning](../concepts/session-pruning.md).

* * *

## Quand l'auto-compaction se produit (runtime Pi)

Dans l'agent Pi embarqué, l'auto-compaction se déclenche dans deux cas :

1.  **Récupération de dépassement** : le modèle renvoie une erreur de dépassement de contexte → compaction → nouvelle tentative.
2.  **Maintenance de seuil** : après un tour réussi, lorsque :

`contextTokens > contextWindow - reserveTokens` Où :

-   `contextWindow` est la fenêtre de contexte du modèle
-   `reserveTokens` est la marge réservée pour les prompts + la prochaine sortie du modèle

Ce sont les sémantiques du runtime Pi (OpenClaw consomme les événements, mais Pi décide quand compacter).

* * *

## Paramètres de compaction (reserveTokens, keepRecentTokens)

Les paramètres de compaction de Pi se trouvent dans les paramètres Pi :

```json
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw applique également un plancher de sécurité pour les exécutions embarquées :

-   Si `compaction.reserveTokens < reserveTokensFloor`, OpenClaw l'augmente.
-   Le plancher par défaut est de `20000` jetons.
-   Définissez `agents.defaults.compaction.reserveTokensFloor: 0` pour désactiver le plancher.
-   S'il est déjà plus élevé, OpenClaw le laisse tel quel.

Pourquoi : laisser assez de marge pour les « tâches de maintenance » multi-tours (comme les écritures en mémoire) avant que la compaction ne devienne inévitable. Implémentation : `ensurePiCompactionReserveTokens()` dans `src/agents/pi-settings.ts` (appelée depuis `src/agents/pi-embedded-runner.ts`).

* * *

## Surfaces visibles par l'utilisateur

Vous pouvez observer la compaction et l'état des sessions via :

-   `/status` (dans n'importe quelle session de chat)
-   `openclaw status` (CLI)
-   `openclaw sessions` / `sessions --json`
-   Mode verbeux : `🧹 Auto-compaction complete` + compteur de compaction

* * *

## Tâches de maintenance silencieuses (NO\_REPLY)

OpenClaw prend en charge les tours « silencieux » pour les tâches en arrière-plan où l'utilisateur ne doit pas voir de sortie intermédiaire. Convention :

-   L'assistant commence sa sortie par `NO_REPLY` pour indiquer « ne pas délivrer de réponse à l'utilisateur ».
-   OpenClaw supprime/rétracte cela dans la couche de livraison.

Depuis `2026.1.10`, OpenClaw supprime également **le streaming de brouillon/tape en cours** lorsqu'un fragment partiel commence par `NO_REPLY`, afin que les opérations silencieuses ne fuient pas de sortie partielle en cours de tour.

* * *

## « Purge de mémoire » pré-compaction (implémentée)

Objectif : avant que l'auto-compaction ne se produise, exécuter un tour agentique silencieux qui écrit l'état durable sur disque (par ex. `memory/YYYY-MM-DD.md` dans l'espace de travail de l'agent) afin que la compaction ne puisse pas effacer le contexte critique. OpenClaw utilise l'approche de **purge pré-seuil** :

1.  Surveiller l'utilisation du contexte de session.
2.  Lorsqu'elle franchit un « seuil doux » (en dessous du seuil de compaction de Pi), exécuter une directive silencieuse « écrire la mémoire maintenant » vers l'agent.
3.  Utiliser `NO_REPLY` pour que l'utilisateur ne voie rien.

Configuration (`agents.defaults.compaction.memoryFlush`) :

-   `enabled` (par défaut : `true`)
-   `softThresholdTokens` (par défaut : `4000`)
-   `prompt` (message utilisateur pour le tour de purge)
-   `systemPrompt` (prompt système supplémentaire ajouté pour le tour de purge)

Notes :

-   Le prompt/prompt système par défaut incluent un indice `NO_REPLY` pour supprimer la livraison.
-   La purge s'exécute une fois par cycle de compaction (suivi dans `sessions.json`).
-   La purge ne s'exécute que pour les sessions Pi embarquées (les backends CLI l'ignorent).
-   La purge est ignorée lorsque l'espace de travail de la session est en lecture seule (`workspaceAccess: "ro"` ou `"none"`).
-   Voir [Mémoire](../concepts/memory.md) pour la disposition des fichiers de l'espace de travail et les modèles d'écriture.

Pi expose également un crochet `session_before_compact` dans l'API d'extension, mais la logique de purge d'OpenClaw réside aujourd'hui côté Gateway.

* * *

## Liste de dépannage

-   Clé de session incorrecte ? Commencez par [/concepts/session](../concepts/session.md) et confirmez la `sessionKey` dans `/status`.
-   Incohérence entre magasin et transcript ? Confirmez l'hôte Gateway et le chemin du magasin depuis `openclaw status`.
-   Spam de compaction ? Vérifiez :
    -   la fenêtre de contexte du modèle (trop petite)
    -   les paramètres de compaction (`reserveTokens` trop élevé pour la fenêtre du modèle peut provoquer une compaction plus précoce)
    -   l'encombrement des résultats d'outils : activez/réglez l'élagage de session
-   Fuite de tours silencieux ? Confirmez que la réponse commence par `NO_REPLY` (jeton exact) et que vous êtes sur une version qui inclut le correctif de suppression du streaming.

[Node.js](../install/node.md)[Configuration](../start/setup.md)