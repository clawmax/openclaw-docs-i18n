title: "Guide de configuration et référence des outils de l'agent OpenClaw"
description: "Apprenez à configurer, autoriser, refuser et gérer les outils de premier ordre d'OpenClaw pour le navigateur, le canvas, les nœuds, cron et les plugins avec des exemples détaillés."
keywords: ["outils openclaw", "outils de l'agent", "configuration des outils", "profils d'outils", "groupes d'outils", "outil navigateur", "outil exec", "plugins d'outils"]
---

  Vue d'ensemble

  
# Outils

OpenClaw expose des **outils de premier ordre** pour le navigateur, le canvas, les nœuds et cron. Ils remplacent les anciennes compétences `openclaw-*` : les outils sont typés, pas de shell, et l'agent doit s'appuyer directement sur eux.

## Désactivation des outils

Vous pouvez autoriser/interdire globalement des outils via `tools.allow` / `tools.deny` dans `openclaw.json` (deny l'emporte). Cela empêche les outils non autorisés d'être envoyés aux fournisseurs de modèles.

```json
{
  tools: { deny: ["browser"] },
}
```

Notes :

-   La correspondance est insensible à la casse.
-   Les caractères génériques `*` sont pris en charge (`"*"` signifie tous les outils).
-   Si `tools.allow` ne référence que des noms d'outils de plugins inconnus ou non chargés, OpenClaw enregistre un avertissement et ignore la liste d'autorisation pour que les outils de base restent disponibles.

## Profils d'outils (liste d'autorisation de base)

`tools.profile` définit une **liste d'autorisation d'outils de base** avant `tools.allow`/`tools.deny`. Surcharge par agent : `agents.list[].tools.profile`. Profils :

-   `minimal` : `session_status` uniquement
-   `coding` : `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
-   `messaging` : `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
-   `full` : aucune restriction (identique à non défini)

Exemple (messagerie uniquement par défaut, autorise aussi les outils Slack + Discord) :

```json
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

Exemple (profil coding, mais interdit exec/process partout) :

```json
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

Exemple (profil coding global, agent support messagerie uniquement) :

```json
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] },
      },
    ],
  },
}
```

## Politique d'outils spécifique au fournisseur

Utilisez `tools.byProvider` pour **restreindre davantage** les outils pour des fournisseurs spécifiques (ou un seul `provider/model`) sans changer vos paramètres globaux par défaut. Surcharge par agent : `agents.list[].tools.byProvider`. Ceci est appliqué **après** le profil d'outil de base et **avant** les listes allow/deny, donc il ne peut que réduire l'ensemble d'outils. Les clés de fournisseur acceptent soit `provider` (ex. `google-antigravity`) soit `provider/model` (ex. `openai/gpt-5.2`). Exemple (conserver le profil coding global, mais outils minimaux pour Google Antigravity) :

```json
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

Exemple (liste d'autorisation spécifique provider/model pour un endpoint instable) :

```json
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

Exemple (surcharge spécifique à un agent pour un seul fournisseur) :

```json
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] },
          },
        },
      },
    ],
  },
}
```

## Groupes d'outils (raccourcis)

Les politiques d'outils (globales, agent, sandbox) prennent en charge les entrées `group:*` qui s'étendent à plusieurs outils. Utilisez-les dans `tools.allow` / `tools.deny`. Groupes disponibles :

-   `group:runtime` : `exec`, `bash`, `process`
-   `group:fs` : `read`, `write`, `edit`, `apply_patch`
-   `group:sessions` : `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory` : `memory_search`, `memory_get`
-   `group:web` : `web_search`, `web_fetch`
-   `group:ui` : `browser`, `canvas`
-   `group:automation` : `cron`, `gateway`
-   `group:messaging` : `message`
-   `group:nodes` : `nodes`
-   `group:openclaw` : tous les outils intégrés d'OpenClaw (exclut les plugins de fournisseur)

Exemple (autoriser uniquement les outils de fichiers + navigateur) :

```json
{
  tools: {
    allow: ["group:fs", "browser"],
  },
}
```

## Plugins + outils

Les plugins peuvent enregistrer **des outils supplémentaires** (et des commandes CLI) au-delà de l'ensemble de base. Voir [Plugins](./tools/plugin.md) pour l'installation + configuration, et [Skills](./tools/skills.md) pour savoir comment les conseils d'utilisation des outils sont injectés dans les prompts. Certains plugins fournissent leurs propres compétences (skills) avec leurs outils (par exemple, le plugin voice-call). Outils de plugins optionnels :

-   [Lobster](./tools/lobster.md) : runtime de workflow typé avec approbations reprises (nécessite le CLI Lobster sur l'hôte de la passerelle).
-   [LLM Task](./tools/llm-task.md) : étape LLM JSON uniquement pour une sortie de workflow structurée (validation de schéma optionnelle).
-   [Diffs](./tools/diffs.md) : visualiseur de diff en lecture seule et rendu de fichiers PNG ou PDF pour les textes avant/après ou les correctifs unifiés.

## Inventaire des outils

### apply\_patch

Applique des correctifs structurés sur un ou plusieurs fichiers. À utiliser pour les modifications multi-hunk. Expérimental : activez via `tools.exec.applyPatch.enabled` (modèles OpenAI uniquement). `tools.exec.applyPatch.workspaceOnly` est par défaut `true` (contenu dans l'espace de travail). Définissez-le à `false` uniquement si vous souhaitez intentionnellement que `apply_patch` écrive/supprime en dehors du répertoire de l'espace de travail.

### exec

Exécute des commandes shell dans l'espace de travail. Paramètres principaux :

-   `command` (obligatoire)
-   `yieldMs` (arrière-plan automatique après délai d'attente, par défaut 10000)
-   `background` (arrière-plan immédiat)
-   `timeout` (secondes ; tue le processus si dépassé, par défaut 1800)
-   `elevated` (bool ; exécute sur l'hôte si le mode élevé est activé/autorisé ; ne change le comportement que lorsque l'agent est en sandbox)
-   `host` (`sandbox | gateway | node`)
-   `security` (`deny | allowlist | full`)
-   `ask` (`off | on-miss | always`)
-   `node` (id/nom du nœud pour `host=node`)
-   Besoin d'un vrai TTY ? Définissez `pty: true`.

Notes :

-   Renvoie `status: "running"` avec un `sessionId` lorsqu'il est mis en arrière-plan.
-   Utilisez `process` pour interroger/enregistrer/écrire/tuer/effacer les sessions en arrière-plan.
-   Si `process` est interdit, `exec` s'exécute de manière synchrone et ignore `yieldMs`/`background`.
-   `elevated` est conditionné par `tools.elevated` plus toute surcharge `agents.list[].tools.elevated` (les deux doivent autoriser) et est un alias pour `host=gateway` + `security=full`.
-   `elevated` ne change le comportement que lorsque l'agent est en sandbox (sinon, c'est un no-op).
-   `host=node` peut cibler une application compagnon macOS ou un hôte de nœud sans interface (`openclaw node run`).
-   Approbations et listes d'autorisation de la passerelle/des nœuds : [Approbations Exec](./tools/exec-approvals.md).

### process

Gère les sessions exec en arrière-plan. Actions principales :

-   `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

Notes :

-   `poll` renvoie une nouvelle sortie et l'état de sortie lorsqu'il est terminé.
-   `log` prend en charge `offset`/`limit` basé sur les lignes (omettez `offset` pour récupérer les N dernières lignes).
-   `process` est limité par agent ; les sessions d'autres agents ne sont pas visibles.

### loop-detection (garde-fous des boucles d'appel d'outil)

OpenClaw suit l'historique récent des appels d'outils et bloque ou avertit lorsqu'il détecte des boucles répétitives sans progression. Activez avec `tools.loopDetection.enabled: true` (par défaut `false`).

```json
{
  tools: {
    loopDetection: {
      enabled: true,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      historySize: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

-   `genericRepeat` : motif d'appel répété du même outil + mêmes paramètres.
-   `knownPollNoProgress` : répétition d'outils de type poll avec des sorties identiques.
-   `pingPong` : motifs alternés `A/B/A/B` sans progression.
-   Surcharge par agent : `agents.list[].tools.loopDetection`.

### web\_search

Recherche sur le web en utilisant Perplexity, Brave, Gemini, Grok ou Kimi. Paramètres principaux :

-   `query` (obligatoire)
-   `count` (1–10 ; par défaut depuis `tools.web.search.maxResults`)

Notes :

-   Nécessite une clé API pour le fournisseur choisi (recommandé : `openclaw configure --section web`).
-   Activez via `tools.web.search.enabled`.
-   Les réponses sont mises en cache (par défaut 15 min).
-   Voir [Outils Web](./tools/web.md) pour la configuration.

### web\_fetch

Récupère et extrait le contenu lisible d'une URL (HTML → markdown/texte). Paramètres principaux :

-   `url` (obligatoire)
-   `extractMode` (`markdown` | `text`)
-   `maxChars` (tronque les pages longues)

Notes :

-   Activez via `tools.web.fetch.enabled`.
-   `maxChars` est limité par `tools.web.fetch.maxCharsCap` (par défaut 50000).
-   Les réponses sont mises en cache (par défaut 15 min).
-   Pour les sites lourds en JS, préférez l'outil navigateur.
-   Voir [Outils Web](./tools/web.md) pour la configuration.
-   Voir [Firecrawl](./tools/firecrawl.md) pour la solution de secours anti-bot optionnelle.

### browser

Contrôle le navigateur dédié géré par OpenClaw. Actions principales :

-   `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
-   `snapshot` (aria/ai)
-   `screenshot` (renvoie un bloc image + `MEDIA:`)
-   `act` (actions UI : cliquer/taper/appuyer/survoler/glisser/sélectionner/remplir/redimensionner/attendre/évaluer)
-   `navigate`, `console`, `pdf`, `upload`, `dialog`

Gestion des profils :

-   `profiles` — liste tous les profils du navigateur avec leur statut
-   `create-profile` — crée un nouveau profil avec un port auto-attribué (ou `cdpUrl`)
-   `delete-profile` — arrête le navigateur, supprime les données utilisateur, retire de la configuration (local uniquement)
-   `reset-profile` — tue le processus orphelin sur le port du profil (local uniquement)

Paramètres communs :

-   `profile` (optionnel ; par défaut `browser.defaultProfile`)
-   `target` (`sandbox` | `host` | `node`)
-   `node` (optionnel ; choisit un id/nom de nœud spécifique) Notes :
-   Nécessite `browser.enabled=true` (par défaut `true` ; définissez `false` pour désactiver).
-   Toutes les actions acceptent le paramètre optionnel `profile` pour la prise en charge multi-instances.
-   Lorsque `profile` est omis, utilise `browser.defaultProfile` (par défaut "chrome").
-   Noms de profils : uniquement minuscules alphanumériques + tirets (max 64 caractères).
-   Plage de ports : 18800-18899 (~100 profils max).
-   Les profils distants sont en attachement uniquement (pas de start/stop/reset).
-   Si un nœud capable de navigateur est connecté, l'outil peut automatiquement le router (sauf si vous épinglez `target`).
-   `snapshot` utilise par défaut `ai` lorsque Playwright est installé ; utilisez `aria` pour l'arbre d'accessibilité.
-   `snapshot` prend également en charge les options de snapshot de rôle (`interactive`, `compact`, `depth`, `selector`) qui renvoient des références comme `e12`.
-   `act` nécessite une `ref` provenant de `snapshot` (`12` numérique des snapshots AI, ou `e12` des snapshots de rôle) ; utilisez `evaluate` pour les rares besoins de sélecteur CSS.
-   Évitez `act` → `wait` par défaut ; utilisez-le uniquement dans des cas exceptionnels (aucun état UI fiable sur lequel attendre).
-   `upload` peut éventuellement passer une `ref` pour un clic automatique après armement.
-   `upload` prend également en charge `inputRef` (référence aria) ou `element` (sélecteur CSS) pour définir directement ``.

### canvas

Pilote le Canvas du nœud (présenter, évaluer, snapshot, A2UI). Actions principales :

-   `present`, `hide`, `navigate`, `eval`
-   `snapshot` (renvoie un bloc image + `MEDIA:`)
-   `a2ui_push`, `a2ui_reset`

Notes :

-   Utilise `node.invoke` de la passerelle en interne.
-   Si aucun `node` n'est fourni, l'outil en choisit un par défaut (nœud unique connecté ou nœud mac local).
-   A2UI est v0.8 uniquement (pas de `createSurface`) ; le CLI rejette le JSONL v0.9 avec des erreurs de ligne.
-   Test rapide : `openclaw nodes canvas a2ui push --node  --text "Hello from A2UI"`.

### nodes

Découvre et cible les nœuds appairés ; envoie des notifications ; capture caméra/écran. Actions principales :

-   `status`, `describe`
-   `pending`, `approve`, `reject` (appairage)
-   `notify` (macOS `system.notify`)
-   `run` (macOS `system.run`)
-   `camera_list`, `camera_snap`, `camera_clip`, `screen_record`
-   `location_get`, `notifications_list`, `notifications_action`
-   `device_status`, `device_info`, `device_permissions`, `device_health`

Notes :

-   Les commandes caméra/écran nécessitent que l'application du nœud soit au premier plan.
-   Les images renvoient des blocs image + `MEDIA:`.
-   Les vidéos renvoient `FILE:` (mp4).
-   La localisation renvoie une charge utile JSON (lat/lon/précision/horodatage).
-   `run` paramètres : tableau `command` argv ; optionnel `cwd`, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording`.

Exemple (`run`) :

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

### image

Analyse une image avec le modèle d'image configuré. Paramètres principaux :

-   `image` (chemin ou URL obligatoire)
-   `prompt` (optionnel ; par défaut "Décrivez l'image.")
-   `model` (surcharge optionnelle)
-   `maxBytesMb` (limite de taille optionnelle)

Notes :

-   Disponible uniquement lorsque `agents.defaults.imageModel` est configuré (primaire ou de secours), ou lorsqu'un modèle d'image implicite peut être déduit de votre modèle par défaut + l'authentification configurée (appariement au mieux).
-   Utilise le modèle d'image directement (indépendant du modèle de chat principal).

### pdf

Analyse un ou plusieurs documents PDF. Pour le comportement complet, les limites, la configuration et les exemples, voir [Outil PDF](./tools/pdf.md).

### message

Envoie des messages et des actions de canal sur Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams. Actions principales :

-   `send` (texte + média optionnel ; MS Teams prend également en charge `card` pour les cartes adaptatives)
-   `poll` (sondages WhatsApp/Discord/MS Teams)
-   `react` / `reactions` / `read` / `edit` / `delete`
-   `pin` / `unpin` / `list-pins`
-   `permissions`
-   `thread-create` / `thread-list` / `thread-reply`
-   `search`
-   `sticker`
-   `member-info` / `role-info`
-   `emoji-list` / `emoji-upload` / `sticker-upload`
-   `role-add` / `role-remove`
-   `channel-info` / `channel-list`
-   `voice-status`
-   `event-list` / `event-create`
-   `timeout` / `kick` / `ban`

Notes :

-   `send` route WhatsApp via la Passerelle ; les autres canaux vont directement.
-   `poll` utilise la Passerelle pour WhatsApp et MS Teams ; les sondages Discord vont directement.
-   Lorsqu'un appel d'outil message est lié à une session de chat active, les envois sont limités à la cible de cette session pour éviter les fuites inter-contextes.

### cron

Gère les tâches cron et les réveils de la Passerelle. Actions principales :

-   `status`, `list`
-   `add`, `update`, `remove`, `run`, `runs`
-   `wake` (met en file d'attente un événement système + battement de cœur immédiat optionnel)

Notes :

-   `add` attend un objet de tâche cron complet (même schéma que `cron.add` RPC).
-   `update` utilise `{ jobId, patch }` (`id` accepté pour compatibilité).

### gateway

Redémarre ou applique des mises à jour au processus Gateway en cours d'exécution (sur place). Actions principales :

-   `restart` (autorise + envoie `SIGUSR1` pour un redémarrage en cours de processus ; `openclaw gateway` redémarre sur place)
-   `config.schema.lookup` (inspecte un chemin de configuration à la fois sans charger le schéma complet dans le contexte du prompt)
-   `config.get`
-   `config.apply` (valide + écrit la configuration + redémarre + réveille)
-   `config.patch` (fusionne une mise à jour partielle + redémarre + réveille)
-   `update.run` (exécute la mise à jour + redémarre + réveille)

Notes :

-   `config.schema.lookup` attend un chemin de configuration ciblé tel que `gateway.auth` ou `agents.list.*.heartbeat`.
-   Les chemins peuvent inclure des identifiants de plugin délimités par des barres obliques lors de l'adressage de `plugins.entries.`, par exemple `plugins.entries.pack/one.config`.
-   Utilisez `delayMs` (par défaut 2000) pour éviter d'interrompre une réponse en cours.
-   `config.schema` reste disponible pour les flux internes de l'interface de contrôle et n'est pas exposé via l'outil `gateway` de l'agent.
-   `restart` est activé par défaut ; définissez `commands.restart: false` pour le désactiver.

### sessions\_list / sessions\_history / sessions\_send / sessions\_spawn / session\_status

Liste les sessions, inspecte l'historique des transcriptions ou envoie à une autre session. Paramètres principaux :

-   `sessions_list` : `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = aucun)
-   `sessions_history` : `sessionKey` (ou `sessionId`), `limit?`, `includeTools?`
-   `sessions_send` : `sessionKey` (ou `sessionId`), `message`, `timeoutSeconds?` (0 = fire-and-forget)
-   `sessions_spawn` : `task`, `label?`, `runtime?`, `agentId?`, `model?`, `thinking?`, `cwd?`, `runTimeoutSeconds?`, `thread?`, `mode?`, `cleanup?`, `sandbox?`, `streamTo?`, `attachments?`, `attachAs?`
-   `session_status` : `sessionKey?` (par défaut actuelle ; accepte `sessionId`), `model?` (`default` efface la surcharge)

Notes :

-   `main` est la clé de chat direct canonique ; global/unknown sont masqués.
-   `messageLimit > 0` récupère les N derniers messages par session (les messages d'outils sont filtrés).
-   Le ciblage des sessions est contrôlé par `tools.sessions.visibility` (par défaut `tree` : session actuelle + sessions des sous-agents créés). Si vous exécutez un agent partagé pour plusieurs utilisateurs, envisagez de définir `tools.sessions.visibility: "self"` pour empêcher la navigation inter-sessions.
-   `sessions_send` attend la finalisation lorsque `timeoutSeconds > 0`.
-   La livraison/annonce se produit après la finalisation et est au mieux ; `status: "ok"` confirme que l'exécution de l'agent est terminée, pas que l'annonce a été livrée.
-   `sessions_spawn` prend en charge `runtime: "subagent" | "acp"` (`subagent` par défaut). Pour le comportement du runtime ACP, voir [Agents ACP](./tools/acp-agents.md).
-   Pour le runtime ACP, `streamTo: "parent"` achemine les résumés de progression de l'exécution initiale vers la session du demandeur sous forme d'événements système au lieu d'une livraison directe à l'enfant.
-   `sessions_spawn` démarre une exécution de sous-agent et publie une réponse d'annonce dans le chat du demandeur.
    -   Prend en charge le mode one-shot (`mode: "run"`) et le mode persistant lié à un fil de discussion (`mode: "session"` avec `thread: true`).
    -   Si `thread: true` et `mode` est omis, le mode par défaut est `session`.
    -   `mode: "session"` nécessite `thread: true`.
    -   Si `runTimeoutSeconds` est omis, OpenClaw utilise `agents.defaults.subagents.runTimeoutSeconds` lorsqu'il est défini ; sinon le délai d'attente par défaut est `0` (pas de délai d'attente).
    -   Les flux liés à des fils de discussion Discord dépendent de `session.threadBindings.*` et `channels.discord.threadBindings.*`.
    -   Le format de réponse inclut `Status`, `Result` et des statistiques compactes.
    -   `Result` est le texte de complétion de l'assistant ; s'il est manquant, le dernier `toolResult` est utilisé comme solution de secours.
-   Les créations en mode complétion manuelle envoient d'abord directement, avec une file d'attente de secours et une nouvelle tentative en cas d'échec transitoire (`status: "ok"` signifie que l'exécution est terminée, pas que l'annonce a été livrée).
-   `sessions_spawn` prend en charge les pièces jointes de fichiers en ligne pour le runtime subagent uniquement (ACP les rejette). Chaque pièce jointe a `name`, `content` et optionnellement `encoding` (`utf8` ou `base64`) et `mimeType`. Les fichiers sont matérialisés dans l'espace de travail enfant à `.openclaw/attachments//` avec un fichier de métadonnées `.manifest.json`. L'outil renvoie un reçu avec `count`, `totalBytes`, `sha256` par fichier et `relDir`. Le contenu des pièces jointes est automatiquement masqué de la persistance des transcriptions.
    -   Configurez les limites via `tools.sessions_spawn.attachments` (`enabled`, `maxTotalBytes`, `maxFiles`, `maxFileBytes`, `retainOnSessionKeep`).
    -   `attachAs.mountPath` est un indice réservé pour les futures implémentations de montage.
-   `sessions_spawn` est non bloquant et renvoie immédiatement `status: "accepted"`.
-   Les réponses ACP `streamTo: "parent"` peuvent inclure `streamLogPath` (session-scoped `*.acp-stream.jsonl`) pour suivre l'historique de progression.
-   `sessions_send` exécute un ping‑pong de réponse (répondez `REPLY_SKIP` pour arrêter ; tours max via `session.agentToAgent.maxPingPongTurns`, 0–5).
-   Après le ping‑pong, l'agent cible exécute une **étape d'annonce** ; répondez `ANNOUNCE_SKIP` pour supprimer l'annonce.
-   Limitation sandbox : lorsque la session actuelle est en sandbox et `agents.defaults.sandbox.sessionToolsVisibility: "spawned"`, OpenClaw limite `tools.sessions.visibility` à `tree`.

### agents\_list

Liste les identifiants d'agents que la session actuelle peut cibler avec `sessions_spawn`. Notes :

-   Le résultat est restreint aux listes d'autorisation par agent (`agents.list[].subagents.allowAgents`).
-   Lorsque `["*"]` est configuré, l'outil inclut tous les agents configurés et marque `allowAny: true`.

## Paramètres (communs)

Outils soutenus par la passerelle (`canvas`, `nodes`, `cron`) :

-   `gatewayUrl` (par défaut `ws://127.0.0.1:18789`)
-   `gatewayToken` (si l'authentification est activée)
-   `timeoutMs`

Note : lorsque `gatewayUrl` est défini, incluez explicitement `gatewayToken`. Les outils n'héritent pas des informations d'identification de configuration ou d'environnement pour les surcharges, et l'absence d'informations d'identification explicites est une erreur. Outil navigateur :

-   `profile` (optionnel ; par défaut `browser.defaultProfile`)
-   `target` (`sandbox` | `host` | `node`)
-   `node` (optionnel ; épingle un id/nom de nœud spécifique)

## Flux d'agent recommandés

Automatisation du navigateur :

1.  `browser` → `status` / `start`
2.  `snapshot` (ai ou aria)
3.  `act` (cliquer/taper/appuyer)
4.  `screenshot` si vous avez besoin d'une confirmation visuelle

Rendu Canvas :

1.  `canvas` → `present`
2.  `a2ui_push` (optionnel)
3.  `snapshot`

Ciblage de nœud :

1.  `nodes` → `status`
2.  `describe` sur le nœud choisi
3.  `notify` / `run` / `camera_snap` / `screen_record`

## Sécurité

-   Évitez `system.run` direct ; utilisez `nodes` → `run` uniquement avec le consentement explicite de l'utilisateur.
-   Respectez le consentement de l'utilisateur pour la capture caméra/écran.
-   Utilisez `status/describe` pour vérifier les permissions avant d'invoquer les commandes média.

## Comment les outils sont présentés à l'agent

Les outils sont exposés via deux canaux parallèles :

1.  **Texte du prompt système** : une liste lisible par l'humain + des conseils.
2.  **Schéma d'outil** : les définitions de fonctions structurées envoyées à l'API du modèle.

Cela signifie que l'agent voit à la fois "quels outils existent" et "comment les appeler". Si un outil n'apparaît pas dans le prompt système ou le schéma, le modèle ne peut pas l'appeler.

[apply\_patch Tool](./tools/apply-patch.md)