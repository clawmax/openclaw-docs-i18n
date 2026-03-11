

  Sessions et mémoire

  
# Outils de session

Objectif : un petit ensemble d'outils difficile à mal utiliser pour que les agents puissent lister les sessions, récupérer l'historique et envoyer vers une autre session.

## Noms des outils

-   `sessions_list`
-   `sessions_history`
-   `sessions_send`
-   `sessions_spawn`

## Modèle de clé

-   Le compartiment de chat direct principal est toujours la clé littérale `"main"` (résolue vers la clé principale de l'agent actuel).
-   Les chats de groupe utilisent `agent:::group:` ou `agent:::channel:` (passez la clé complète).
-   Les tâches cron utilisent `cron:<job.id>`.
-   Les hooks utilisent `hook:` sauf indication explicite.
-   Les sessions de nœud utilisent `node-` sauf indication explicite.

`global` et `unknown` sont des valeurs réservées et ne sont jamais listées. Si `session.scope = "global"`, nous l'aliasons en `main` pour tous les outils afin que les appelants ne voient jamais `global`.

## sessions\_list

Lister les sessions sous forme de tableau de lignes. Paramètres :

-   `kinds?: string[]` filtre : parmi `"main" | "group" | "cron" | "hook" | "node" | "other"`
-   `limit?: number` nombre maximum de lignes (par défaut : valeur serveur, limitée ex. 200)
-   `activeMinutes?: number` uniquement les sessions mises à jour dans les N dernières minutes
-   `messageLimit?: number` 0 = pas de messages (par défaut 0) ; >0 = inclure les N derniers messages

Comportement :

-   `messageLimit > 0` récupère `chat.history` par session et inclut les N derniers messages.
-   Les résultats d'outils sont filtrés dans la sortie de liste ; utilisez `sessions_history` pour les messages d'outils.
-   Lorsqu'elle s'exécute dans une session d'agent **sandboxée**, les outils de session passent par défaut en **visibilité limitée aux sessions créées** (voir ci-dessous).

Format de ligne (JSON) :

-   `key` : clé de session (chaîne)
-   `kind` : `main | group | cron | hook | node | other`
-   `channel` : `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
-   `displayName` (libellé d'affichage du groupe si disponible)
-   `updatedAt` (ms)
-   `sessionId`
-   `model`, `contextTokens`, `totalTokens`
-   `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
-   `sendPolicy` (remplacement de session si défini)
-   `lastChannel`, `lastTo`
-   `deliveryContext` (normalisé `{ channel, to, accountId }` quand disponible)
-   `transcriptPath` (chemin dérivé au mieux du répertoire de stockage + sessionId)
-   `messages?` (uniquement quand `messageLimit > 0`)

## sessions\_history

Récupérer la transcription pour une session. Paramètres :

-   `sessionKey` (obligatoire ; accepte une clé de session ou un `sessionId` de `sessions_list`)
-   `limit?: number` nombre maximum de messages (limité par le serveur)
-   `includeTools?: boolean` (par défaut false)

Comportement :

-   `includeTools=false` filtre les messages `role: "toolResult"`.
-   Renvoie un tableau de messages au format brut de transcription.
-   Lorsqu'un `sessionId` est donné, OpenClaw le résout vers la clé de session correspondante (les identifiants manquants génèrent une erreur).

## sessions\_send

Envoyer un message dans une autre session. Paramètres :

-   `sessionKey` (obligatoire ; accepte une clé de session ou un `sessionId` de `sessions_list`)
-   `message` (obligatoire)
-   `timeoutSeconds?: number` (par défaut >0 ; 0 = fire-and-forget)

Comportement :

-   `timeoutSeconds = 0` : met en file d'attente et renvoie `{ runId, status: "accepted" }`.
-   `timeoutSeconds > 0` : attend jusqu'à N secondes pour la complétion, puis renvoie `{ runId, status: "ok", reply }`.
-   Si l'attente expire : `{ runId, status: "timeout", error }`. L'exécution continue ; appelez `sessions_history` plus tard.
-   Si l'exécution échoue : `{ runId, status: "error", error }`.
-   L'annonce de livraison s'exécute après la fin de l'exécution principale et est best-effort ; `status: "ok"` ne garantit pas que l'annonce a été livrée.
-   L'attente se fait via `agent.wait` de la passerelle (côté serveur) pour que les reconnexions n'interrompent pas l'attente.
-   Le contexte de message agent-à-agent est injecté pour l'exécution principale.
-   Les messages inter-sessions sont persistés avec `message.provenance.kind = "inter_session"` afin que les lecteurs de transcription puissent distinguer les instructions d'agent routées de la saisie utilisateur externe.
-   Après la fin de l'exécution principale, OpenClaw exécute une **boucle de réponse** :
    -   Les tours 2+ alternent entre l'agent demandeur et l'agent cible.
    -   Répondez exactement `REPLY_SKIP` pour arrêter le ping‑pong.
    -   Le nombre maximum de tours est `session.agentToAgent.maxPingPongTurns` (0–5, par défaut 5).
-   Une fois la boucle terminée, OpenClaw exécute l'**étape d'annonce agent‑à‑agent** (agent cible uniquement) :
    -   Répondez exactement `ANNOUNCE_SKIP` pour rester silencieux.
    -   Toute autre réponse est envoyée au canal cible.
    -   L'étape d'annonce inclut la requête originale + la réponse du tour 1 + la dernière réponse du ping‑pong.

## Champ Channel

-   Pour les groupes, `channel` est le canal enregistré dans l'entrée de session.
-   Pour les chats directs, `channel` est mappé à partir de `lastChannel`.
-   Pour cron/hook/node, `channel` est `internal`.
-   Si manquant, `channel` est `unknown`.

## Sécurité / Politique d'envoi

Blocage basé sur des politiques par canal/type de chat (pas par identifiant de session).

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

Remplacement à l'exécution (par entrée de session) :

-   `sendPolicy: "allow" | "deny"` (non défini = hérité de la configuration)
-   Définissable via `sessions.patch` ou via `/send on|off|inherit` (message autonome) réservé au propriétaire.

Points d'application :

-   `chat.send` / `agent` (passerelle)
-   logique de livraison de réponse automatique

## sessions\_spawn

Créer une exécution de sous-agent dans une session isolée et annoncer le résultat au canal de chat du demandeur. Paramètres :

-   `task` (obligatoire)
-   `label?` (optionnel ; utilisé pour les logs/UI)
-   `agentId?` (optionnel ; créer sous un autre identifiant d'agent si autorisé)
-   `model?` (optionnel ; remplace le modèle du sous-agent ; les valeurs invalides génèrent une erreur)
-   `thinking?` (optionnel ; remplace le niveau de réflexion pour l'exécution du sous-agent)
-   `runTimeoutSeconds?` (par défaut `agents.defaults.subagents.runTimeoutSeconds` quand défini, sinon `0` ; quand défini, interrompt l'exécution du sous-agent après N secondes)
-   `thread?` (par défaut false ; demande un routage lié à un fil pour cette création quand supporté par le canal/plugin)
-   `mode?` (`run|session` ; par défaut `run`, mais par défaut `session` quand `thread=true` ; `mode="session"` nécessite `thread=true`)
-   `cleanup?` (`delete|keep`, par défaut `keep`)
-   `sandbox?` (`inherit|require`, par défaut `inherit` ; `require` rejette la création sauf si l'environnement d'exécution enfant cible est sandboxé)
-   `attachments?` (tableau optionnel de fichiers inline ; uniquement pour l'environnement d'exécution sous-agent, ACP rejette). Chaque entrée : `{ name, content, encoding?: "utf8" | "base64", mimeType? }`. Les fichiers sont matérialisés dans l'espace de travail enfant à `.openclaw/attachments//`. Renvoie un reçu avec sha256 par fichier.
-   `attachAs?` (optionnel ; `{ mountPath? }` indice réservé pour les futures implémentations de montage)

Liste d'autorisation :

-   `agents.list[].subagents.allowAgents` : liste des identifiants d'agents autorisés via `agentId` (`["*"]` pour autoriser tous). Par défaut : uniquement l'agent demandeur.
-   Garde d'héritage sandbox : si la session du demandeur est sandboxée, `sessions_spawn` rejette les cibles qui s'exécuteraient non sandboxées.

Découverte :

-   Utilisez `agents_list` pour découvrir quels identifiants d'agents sont autorisés pour `sessions_spawn`.

Comportement :

-   Démarre une nouvelle session `agent::subagent:` avec `deliver: false`.
-   Les sous-agents ont par défaut l'ensemble complet d'outils **moins les outils de session** (configurable via `tools.subagents.tools`).
-   Les sous-agents ne sont pas autorisés à appeler `sessions_spawn` (pas de création sous-agent → sous-agent).
-   Toujours non bloquant : renvoie immédiatement `{ status: "accepted", runId, childSessionKey }`.
-   Avec `thread=true`, les plugins de canal peuvent lier la livraison/le routage à une cible de fil (le support Discord est contrôlé par `session.threadBindings.*` et `channels.discord.threadBindings.*`).
-   Après achèvement, OpenClaw exécute une **étape d'annonce du sous-agent** et publie le résultat sur le canal de chat du demandeur.
    -   Si la réponse finale de l'assistant est vide, le dernier `toolResult` de l'historique du sous-agent est inclus en tant que `Result`.
-   Répondez exactement `ANNOUNCE_SKIP` pendant l'étape d'annonce pour rester silencieux.
-   Les réponses d'annonce sont normalisées en `Status`/`Result`/`Notes` ; `Status` provient du résultat d'exécution (pas du texte du modèle).
-   Les sessions de sous-agent sont automatiquement archivées après `agents.defaults.subagents.archiveAfterMinutes` (par défaut : 60).
-   Les réponses d'annonce incluent une ligne de statistiques (temps d'exécution, tokens, sessionKey/sessionId, chemin de transcription, et coût optionnel).

## Visibilité des sessions Sandbox

Les outils de session peuvent être limités en portée pour réduire l'accès inter-sessions. Comportement par défaut :

-   `tools.sessions.visibility` par défaut à `tree` (session actuelle + sessions de sous-agents créées).
-   Pour les sessions sandboxées, `agents.defaults.sandbox.sessionToolsVisibility` peut forcer la visibilité.

Configuration :

```json
{
  tools: {
    sessions: {
      // "self" | "tree" | "agent" | "all"
      // default: "tree"
      visibility: "tree",
    },
  },
  agents: {
    defaults: {
      sandbox: {
        // default: "spawned"
        sessionToolsVisibility: "spawned", // or "all"
      },
    },
  },
}
```

Notes :

-   `self` : uniquement la clé de session actuelle.
-   `tree` : session actuelle + sessions créées par la session actuelle.
-   `agent` : toute session appartenant à l'identifiant d'agent actuel.
-   `all` : toute session (l'accès inter-agent nécessite toujours `tools.agentToAgent`).
-   Quand une session est sandboxée et `sessionToolsVisibility="spawned"`, OpenClaw force la visibilité à `tree` même si vous définissez `tools.sessions.visibility="all"`.

[Élagage des sessions](./session-pruning.md)[Mémoire](./memory.md)