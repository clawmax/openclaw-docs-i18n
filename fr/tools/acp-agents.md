

  Coordination d'agents

  
# Agents ACP

Les sessions [Agent Client Protocol (ACP)](https://agentclientprotocol.com/) permettent à OpenClaw d'exécuter des harnais de codage externes (par exemple Pi, Claude Code, Codex, OpenCode et Gemini CLI) via un plugin backend ACP. Si vous demandez à OpenClaw en langage naturel d'"exécuter ceci dans Codex" ou de "démarrer Claude Code dans un fil de discussion", OpenClaw doit router cette requête vers le runtime ACP (et non vers le runtime natif des sous-agents).

## Flux opérateur rapide

Utilisez ceci lorsque vous voulez un runbook pratique `/acp` :

1.  Créer une session :
    -   `/acp spawn codex --mode persistent --thread auto`
2.  Travaillez dans le fil lié (ou ciblez explicitement cette clé de session).
3.  Vérifiez l'état du runtime :
    -   `/acp status`
4.  Ajustez les options d'exécution si nécessaire :
    -   `/acp model <provider/model>`
    -   `/acp permissions `
    -   `/acp timeout `
5.  Orientez une session active sans remplacer le contexte :
    -   `/acp steer tighten logging and continue`
6.  Arrêtez le travail :
    -   `/acp cancel` (arrêter le tour actuel), ou
    -   `/acp close` (fermer la session + supprimer les liaisons)

## Démarrage rapide pour les humains

Exemples de requêtes naturelles :

-   "Démarre une session Codex persistante dans un fil ici et garde-la concentrée."
-   "Exécute ceci comme une session ACP Claude Code unique et résume le résultat."
-   "Utilise Gemini CLI pour cette tâche dans un fil, puis garde les suivis dans ce même fil."

Ce qu'OpenClaw doit faire :

1.  Choisir `runtime: "acp"`.
2.  Résoudre la cible de harnais demandée (`agentId`, par exemple `codex`).
3.  Si une liaison de fil est demandée et que le canal actuel la supporte, lier la session ACP au fil.
4.  Router les messages de suivi du fil vers cette même session ACP jusqu'à ce qu'elle soit défocalisée/fermée/expirée.

## ACP versus sous-agents

Utilisez ACP lorsque vous voulez un runtime de harnais externe. Utilisez les sous-agents lorsque vous voulez des exécutions déléguées natives d'OpenClaw.

| Domaine | Session ACP | Exécution sous-agent |
| --- | --- | --- |
| Runtime | Plugin backend ACP (par exemple acpx) | Runtime natif sous-agent d'OpenClaw |
| Clé de session | `agent::acp:` | `agent::subagent:` |
| Commandes principales | `/acp ...` | `/subagents ...` |
| Outil de création | `sessions_spawn` avec `runtime:"acp"` | `sessions_spawn` (runtime par défaut) |

Voir aussi [Sous-agents](./subagents.md).

## Sessions liées à un fil (indépendantes du canal)

Lorsque les liaisons de fil sont activées pour un adaptateur de canal, les sessions ACP peuvent être liées à des fils :

-   OpenClaw lie un fil à une session ACP cible.
-   Les messages de suivi dans ce fil sont routés vers la session ACP liée.
-   La sortie ACP est délivrée au même fil.
-   La défocalisation/fermeture/archivage/expiration par inactivité ou durée maximale supprime la liaison.

Le support des liaisons de fil est spécifique à l'adaptateur. Si l'adaptateur de canal actif ne supporte pas les liaisons de fil, OpenClaw renvoie un message clair de non-support/indisponibilité. Les drapeaux de fonctionnalité requis pour ACP lié à un fil :

-   `acp.enabled=true`
-   `acp.dispatch.enabled` est activé par défaut (définir `false` pour suspendre le dispatch ACP)
-   Le drapeau de création de fil ACP de l'adaptateur de canal est activé (spécifique à l'adaptateur)
    -   Discord : `channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram : `channels.telegram.threadBindings.spawnAcpSessions=true`

### Canaux supportant les fils

-   Tout adaptateur de canal qui expose la capacité de liaison session/fil.
-   Support intégré actuel :
    -   Fils/canaux Discord
    -   Sujets Telegram (sujets de forum dans les groupes/supergroupes et sujets de MP)
-   Les canaux plugin peuvent ajouter un support via la même interface de liaison.

## Paramètres spécifiques au canal

Pour les workflows non éphémères, configurez des liaisons ACP persistantes dans les entrées `bindings[]` de haut niveau.

### Modèle de liaison

-   `bindings[].type="acp"` marque une liaison de conversation ACP persistante.
-   `bindings[].match` identifie la conversation cible :
    -   Canal ou fil Discord : `match.channel="discord"` + `match.peer.id=""`
    -   Sujet de forum Telegram : `match.channel="telegram"` + `match.peer.id=":topic:"`
-   `bindings[].agentId` est l'identifiant de l'agent OpenClaw propriétaire.
-   Les surcharges ACP optionnelles se trouvent sous `bindings[].acp` :
    -   `mode` (`persistent` ou `oneshot`)
    -   `label`
    -   `cwd`
    -   `backend`

### Valeurs par défaut du runtime par agent

Utilisez `agents.list[].runtime` pour définir les valeurs par défaut ACP une fois par agent :

-   `agents.list[].runtime.type="acp"`
-   `agents.list[].runtime.acp.agent` (identifiant du harnais, par exemple `codex` ou `claude`)
-   `agents.list[].runtime.acp.backend`
-   `agents.list[].runtime.acp.mode`
-   `agents.list[].runtime.acp.cwd`

Précédence de surcharge pour les sessions ACP liées :

1.  `bindings[].acp.*`
2.  `agents.list[].runtime.acp.*`
3.  Valeurs par défaut ACP globales (par exemple `acp.backend`)

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
      {
        id: "claude",
        runtime: {
          type: "acp",
          acp: { agent: "claude", backend: "acpx", mode: "persistent" },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
    {
      type: "acp",
      agentId: "claude",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
      acp: { cwd: "/workspace/repo-b" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "discord", accountId: "default" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "telegram", accountId: "default" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": { requireMention: false },
          },
        },
      },
    },
    telegram: {
      groups: {
        "-1001234567890": {
          topics: { "42": { requireMention: false } },
        },
      },
    },
  },
}
```

Comportement :

-   OpenClaw s'assure que la session ACP configurée existe avant utilisation.
-   Les messages dans ce canal ou sujet sont routés vers la session ACP configurée.
-   Dans les conversations liées, `/new` et `/reset` réinitialisent la même clé de session ACP sur place.
-   Les liaisons de runtime temporaires (par exemple créées par les flux de focalisation de fil) s'appliquent toujours là où elles sont présentes.

## Démarrer les sessions ACP (interfaces)

### Depuis sessions\_spawn

Utilisez `runtime: "acp"` pour démarrer une session ACP depuis un tour d'agent ou un appel d'outil.

```json
{
  "task": "Open the repo and summarize failing tests",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
```

Notes :

-   `runtime` par défaut est `subagent`, donc définissez explicitement `runtime: "acp"` pour les sessions ACP.
-   Si `agentId` est omis, OpenClaw utilise `acp.defaultAgent` lorsqu'il est configuré.
-   `mode: "session"` nécessite `thread: true` pour maintenir une conversation liée persistante.

Détails de l'interface :

-   `task` (requis) : invite initiale envoyée à la session ACP.
-   `runtime` (requis pour ACP) : doit être `"acp"`.
-   `agentId` (optionnel) : identifiant de harnais cible ACP. Retombe sur `acp.defaultAgent` si défini.
-   `thread` (optionnel, défaut `false`) : demande le flux de liaison de fil là où il est supporté.
-   `mode` (optionnel) : `run` (unique) ou `session` (persistant).
    -   la valeur par défaut est `run`
    -   si `thread: true` et mode omis, OpenClaw peut adopter un comportement persistant par défaut selon le chemin d'exécution
    -   `mode: "session"` nécessite `thread: true`
-   `cwd` (optionnel) : répertoire de travail d'exécution demandé (validé par la politique backend/runtime).
-   `label` (optionnel) : libellé orienté opérateur utilisé dans le texte de session/bannière.
-   `streamTo` (optionnel) : `"parent"` diffuse les résumés de progression de l'exécution ACP initiale vers la session du demandeur sous forme d'événements système.
    -   Lorsqu'il est disponible, les réponses acceptées incluent `streamLogPath` pointant vers un journal JSONL limité à la session (`.acp-stream.jsonl`) que vous pouvez suivre pour l'historique complet de relais.

## Compatibilité du bac à sable

Les sessions ACP s'exécutent actuellement sur le runtime hôte, pas à l'intérieur du bac à sable OpenClaw. Limitations actuelles :

-   Si la session du demandeur est dans le bac à sable, les créations ACP sont bloquées.
    -   Erreur : `Sandboxed sessions cannot spawn ACP sessions because runtime="acp" runs on the host. Use runtime="subagent" from sandboxed sessions.`
-   `sessions_spawn` avec `runtime: "acp"` ne supporte pas `sandbox: "require"`.
    -   Erreur : `sessions_spawn sandbox="require" is unsupported for runtime="acp" because ACP sessions run outside the sandbox. Use runtime="subagent" or sandbox="inherit".`

Utilisez `runtime: "subagent"` lorsque vous avez besoin d'une exécution forcée dans le bac à sable.

### Depuis la commande /acp

Utilisez `/acp spawn` pour un contrôle opérateur explicite depuis le chat si nécessaire.

```bash
/acp spawn codex --mode persistent --thread auto
/acp spawn codex --mode oneshot --thread off
/acp spawn codex --thread here
```

Drapeaux clés :

-   `--mode persistent|oneshot`
-   `--thread auto|here|off`
-   `--cwd <absolute-path>`
-   `--label `

Voir [Commandes Slash](./slash-commands.md).

## Résolution de la cible de session

La plupart des actions `/acp` acceptent une cible de session optionnelle (`session-key`, `session-id` ou `session-label`). Ordre de résolution :

1.  Argument cible explicite (ou `--session` pour `/acp steer`)
    -   essaie la clé
    -   puis l'identifiant de session au format UUID
    -   puis le libellé
2.  Liaison de fil actuelle (si cette conversation/fil est lié à une session ACP)
3.  Retombée sur la session du demandeur actuelle

Si aucune cible n'est résolue, OpenClaw renvoie une erreur claire (`Unable to resolve session target: ...`).

## Modes de création de fil

`/acp spawn` supporte `--thread auto|here|off`.

| Mode | Comportement |
| --- | --- |
| `auto` | Dans un fil actif : lie ce fil. En dehors d'un fil : crée/lie un fil enfant lorsque supporté. |
| `here` | Exige un fil actif actuel ; échoue si ce n'est pas le cas. |
| `off` | Aucune liaison. La session démarre sans liaison. |

Notes :

-   Sur les surfaces sans liaison de fil, le comportement par défaut est effectivement `off`.
-   La création liée à un fil nécessite le support de la politique de canal :
    -   Discord : `channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram : `channels.telegram.threadBindings.spawnAcpSessions=true`

## Contrôles ACP

Famille de commandes disponible :

-   `/acp spawn`
-   `/acp cancel`
-   `/acp steer`
-   `/acp close`
-   `/acp status`
-   `/acp set-mode`
-   `/acp set`
-   `/acp cwd`
-   `/acp permissions`
-   `/acp timeout`
-   `/acp model`
-   `/acp reset-options`
-   `/acp sessions`
-   `/acp doctor`
-   `/acp install`

`/acp status` montre les options d'exécution effectives et, lorsqu'ils sont disponibles, les identifiants de session au niveau du runtime et au niveau du backend. Certains contrôles dépendent des capacités du backend. Si un backend ne supporte pas un contrôle, OpenClaw renvoie une erreur claire de contrôle non supporté.

## Livre de recettes des commandes ACP

| Commande | Ce qu'elle fait | Exemple |
| --- | --- | --- |
| `/acp spawn` | Créer une session ACP ; liaison de fil optionnelle. | `/acp spawn codex --mode persistent --thread auto --cwd /repo` |
| `/acp cancel` | Annuler le tour en cours pour la session cible. | `/acp cancel agent:codex:acp:` |
| `/acp steer` | Envoyer une instruction d'orientation à la session en cours. | `/acp steer --session support inbox prioritize failing tests` |
| `/acp close` | Fermer la session et dissocier les cibles de fil. | `/acp close` |
| `/acp status` | Afficher le backend, le mode, l'état, les options d'exécution, les capacités. | `/acp status` |
| `/acp set-mode` | Définir le mode d'exécution pour la session cible. | `/acp set-mode plan` |
| `/acp set` | Écriture générique d'option de configuration d'exécution. | `/acp set model openai/gpt-5.2` |
| `/acp cwd` | Définir la surcharge du répertoire de travail d'exécution. | `/acp cwd /Users/user/Projects/repo` |
| `/acp permissions` | Définir le profil de politique d'approbation. | `/acp permissions strict` |
| `/acp timeout` | Définir le délai d'exécution (secondes). | `/acp timeout 120` |
| `/acp model` | Définir la surcharge du modèle d'exécution. | `/acp model anthropic/claude-opus-4-5` |
| `/acp reset-options` | Supprimer les surcharges d'options d'exécution de la session. | `/acp reset-options` |
| `/acp sessions` | Lister les sessions ACP récentes depuis le stockage. | `/acp sessions` |
| `/acp doctor` | Santé du backend, capacités, correctifs actionnables. | `/acp doctor` |
| `/acp install` | Afficher les étapes d'installation et d'activation déterministes. | `/acp install` |

## Mappage des options d'exécution

`/acp` a des commandes pratiques et un setter générique. Opérations équivalentes :

-   `/acp model ` correspond à la clé de configuration d'exécution `model`.
-   `/acp permissions ` correspond à la clé de configuration d'exécution `approval_policy`.
-   `/acp timeout ` correspond à la clé de configuration d'exécution `timeout`.
-   `/acp cwd ` met à jour directement la surcharge cwd d'exécution.
-   `/acp set  ` est le chemin générique.
    -   Cas spécial : `key=cwd` utilise le chemin de surcharge cwd.
-   `/acp reset-options` efface toutes les surcharges d'exécution pour la session cible.

## Support des harnais acpx (actuel)

Alias de harnais intégrés acpx actuels :

-   `pi`
-   `claude`
-   `codex`
-   `opencode`
-   `gemini`
-   `kimi`

Lorsqu'OpenClaw utilise le backend acpx, préférez ces valeurs pour `agentId` sauf si votre configuration acpx définit des alias d'agents personnalisés. L'utilisation directe de la CLI acpx peut également cibler des adaptateurs arbitraires via `--agent `, mais cette échappatoire brute est une fonctionnalité de la CLI acpx (et non le chemin normal `agentId` d'OpenClaw).

## Configuration requise

Base ACP principale :

```json
{
  acp: {
    enabled: true,
    // Optionnel. La valeur par défaut est true ; définir false pour suspendre le dispatch ACP tout en gardant les contrôles /acp.
    dispatch: { enabled: true },
    backend: "acpx",
    defaultAgent: "codex",
    allowedAgents: ["pi", "claude", "codex", "opencode", "gemini", "kimi"],
    maxConcurrentSessions: 8,
    stream: {
      coalesceIdleMs: 300,
      maxChunkChars: 1200,
    },
    runtime: {
      ttlMinutes: 120,
    },
  },
}
```

La configuration des liaisons de fil est spécifique à l'adaptateur de canal. Exemple pour Discord :

```json
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        spawnAcpSessions: true,
      },
    },
  },
}
```

Si la création ACP liée à un fil ne fonctionne pas, vérifiez d'abord le drapeau de fonctionnalité de l'adaptateur :

-   Discord : `channels.discord.threadBindings.spawnAcpSessions=true`

Voir [Référence de configuration](../gateway/configuration-reference.md).

## Configuration du plugin pour le backend acpx

Installer et activer le plugin :

```bash
openclaw plugins install acpx
openclaw config set plugins.entries.acpx.enabled true
```

Installation locale dans l'espace de travail pendant le développement :

```bash
openclaw plugins install ./extensions/acpx
```

Puis vérifiez la santé du backend :

```bash
/acp doctor
```

### Configuration de la commande et de la version acpx

Par défaut, le plugin acpx (publié sous `@openclaw/acpx`) utilise le binaire épinglé local au plugin :

1.  La commande par défaut est `extensions/acpx/node_modules/.bin/acpx`.
2.  La version attendue par défaut est l'épingle de l'extension.
3.  Le démarrage enregistre immédiatement le backend ACP comme non prêt.
4.  Une tâche d'assurance en arrière-plan vérifie `acpx --version`.
5.  Si le binaire local au plugin est manquant ou incompatible, il exécute : `npm install --omit=dev --no-save acpx@` et revérifie.

Vous pouvez remplacer la commande/version dans la configuration du plugin :

```json
{
  "plugins": {
    "entries": {
      "acpx": {
        "enabled": true,
        "config": {
          "command": "../acpx/dist/cli.js",
          "expectedVersion": "any"
        }
      }
    }
  }
}
```

Notes :

-   `command` accepte un chemin absolu, un chemin relatif ou un nom de commande (`acpx`).
-   Les chemins relatifs sont résolus depuis le répertoire de l'espace de travail OpenClaw.
-   `expectedVersion: "any"` désactive la correspondance stricte de version.
-   Lorsque `command` pointe vers un binaire/chemin personnalisé, l'installation automatique locale au plugin est désactivée.
-   Le démarrage d'OpenClaw reste non bloquant pendant que la vérification de santé du backend s'exécute.

Voir [Plugins](./plugin.md).

## Configuration des permissions

Les sessions ACP s'exécutent de manière non interactive — il n'y a pas de TTY pour approuver ou refuser les invites de permission d'écriture de fichier et d'exécution de shell. Le plugin acpx fournit deux clés de configuration qui contrôlent la gestion des permissions :

### permissionMode

Contrôle quelles opérations l'agent de harnais peut effectuer sans invite.

| Valeur | Comportement |
| --- | --- |
| `approve-all` | Approuve automatiquement toutes les écritures de fichiers et commandes shell. |
| `approve-reads` | Approuve automatiquement les lectures uniquement ; les écritures et exécutions nécessitent des invites. |
| `deny-all` | Refuse toutes les invites de permission. |

### nonInteractivePermissions

Contrôle ce qui se passe lorsqu'une invite de permission serait affichée mais qu'aucun TTY interactif n'est disponible (ce qui est toujours le cas pour les sessions ACP).

| Valeur | Comportement |
| --- | --- |
| `fail` | Abandonne la session avec `AcpRuntimeError`. **(par défaut)** |
| `deny` | Refuse silencieusement la permission et continue (dégradation gracieuse). |

### Configuration

Définir via la configuration du plugin :

```bash
openclaw config set plugins.entries.acpx.config.permissionMode approve-all
openclaw config set plugins.entries.acpx.config.nonInteractivePermissions fail
```

Redémarrez la passerelle après avoir modifié ces valeurs.

> **Important :** OpenClaw utilise actuellement par défaut `permissionMode=approve-reads` et `nonInteractivePermissions=fail`. Dans les sessions ACP non interactives, toute écriture ou exécution qui déclenche une invite de permission peut échouer avec `AcpRuntimeError: Permission prompt unavailable in non-interactive mode`. Si vous devez restreindre les permissions, définissez `nonInteractivePermissions` sur `deny` pour que les sessions se dégradent gracieusement au lieu de planter.

## Dépannage

| Symptôme | Cause probable | Solution |
| --- | --- | --- |
| `ACP runtime backend is not configured` | Plugin backend manquant ou désactivé. | Installez et activez le plugin backend, puis exécutez `/acp doctor`. |
| `ACP is disabled by policy (acp.enabled=false)` | ACP globalement désactivé. | Définissez `acp.enabled=true`. |
| `ACP dispatch is disabled by policy (acp.dispatch.enabled=false)` | Dispatch depuis les messages de fil normaux désactivé. | Définissez `acp.dispatch.enabled=true`. |
| `ACP agent "" is not allowed by policy` | Agent non dans la liste autorisée. | Utilisez un `agentId` autorisé ou mettez à jour `acp.allowedAgents`. |
| `Unable to resolve session target: ...` | Jeton clé/id/libellé incorrect. | Exécutez `/acp sessions`, copiez la clé/libellé exact, réessayez. |
| `--thread here requires running /acp spawn inside an active ... thread` | `--thread here` utilisé en dehors d'un contexte de fil. | Déplacez-vous vers le fil cible ou utilisez `--thread auto`/`off`. |
| `Only <user-id> can rebind this thread.` | Un autre utilisateur possède la liaison de fil. | Reliez en tant que propriétaire ou utilisez un fil différent. |
| `Thread bindings are unavailable for .` | L'adaptateur manque de capacité de liaison de fil. | Utilisez `--thread off` ou déplacez-vous vers un adaptateur/canal supporté. |
| `Sandboxed sessions cannot spawn ACP sessions ...` | Le runtime ACP est côté hôte ; la session du demandeur est dans le bac à sable. | Utilisez `runtime="subagent"` depuis les sessions dans le bac à sable, ou exécutez la création ACP depuis une session non bac à sable. |
| `sessions_spawn sandbox="require" is unsupported for runtime="acp" ...` | `sandbox="require"` demandé pour le runtime ACP. | Utilisez `runtime="subagent"` pour le bac à sable obligatoire, ou utilisez ACP avec `sandbox="inherit"` depuis une session non bac à sable. |
| Métadonnées ACP manquantes pour la session liée | Métadonnées de session ACP obsolètes/supprimées. | Recréez avec `/acp spawn`, puis reliez/focalisez le fil. |
| `AcpRuntimeError: Permission prompt unavailable in non-interactive mode` | `permissionMode` bloque les écritures/exécutions dans la session ACP non interactive. | Définissez `plugins.entries.acpx.config.permissionMode` sur `approve-all` et redémarrez la passerelle. Voir [Configuration des permissions](#permission-configuration). |
| La session ACP échoue tôt avec peu de sortie | Les invites de permission sont bloquées par `permissionMode`/`nonInteractivePermissions`. | Vérifiez les journaux de la passerelle pour `AcpRuntimeError`. Pour des permissions complètes, définissez `permissionMode=approve-all` ; pour une dégradation gracieuse, définissez `nonInteractivePermissions=deny`. |
| La session ACP reste indéfiniment bloquée après avoir terminé le travail | Le processus de harnais est terminé mais la session ACP n'a pas signalé la fin. | Surveillez avec `ps aux \| grep acpx` ; tuez manuellement les processus obsolètes. |

[Sous-Agents](./subagents.md)[Bac à sable et outils multi-agents](./multi-agent-sandbox-tools.md)