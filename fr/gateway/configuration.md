title: "Guide de configuration et tutoriel d'installation d'OpenClaw Gateway"
description: "Apprenez à configurer OpenClaw Gateway avec JSON5. Configurez les canaux, modèles, sessions, sandboxing, tâches cron et le routage multi-agent."
keywords: ["configuration openclaw", "installation de la passerelle", "configuration json5", "routage multi-agent", "gestion des sessions", "sandboxing", "tâches cron", "politique de dm"]
---

  Configuration et opérations

  
# Configuration

OpenClaw lit une configuration **JSON5** optionnelle depuis `~/.openclaw/openclaw.json`. Si le fichier est absent, OpenClaw utilise des valeurs par défaut sûres. Les raisons courantes d'ajouter une configuration :

-   Connecter des canaux et contrôler qui peut envoyer des messages au bot
-   Définir des modèles, outils, sandboxing ou automatisation (cron, hooks)
-   Ajuster les sessions, médias, réseau ou interface utilisateur

Consultez la [référence complète](./configuration-reference.md) pour tous les champs disponibles.

> **💡** **Nouveau avec la configuration ?** Commencez avec `openclaw onboard` pour une installation interactive, ou consultez le guide [Exemples de configuration](./configuration-examples.md) pour des configurations complètes à copier-coller.

## Configuration minimale

```
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Édition de la configuration

```bash
openclaw onboard       # assistant d'installation complet
openclaw configure     # assistant de configuration
```

```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```

Ouvrez [http://127.0.0.1:18789](http://127.0.0.1:18789) et utilisez l'onglet **Config**. L'interface de contrôle affiche un formulaire généré à partir du schéma de configuration, avec un éditeur **JSON Brut** comme échappatoire.

Éditez `~/.openclaw/openclaw.json` directement. La passerelle surveille le fichier et applique les changements automatiquement (voir [rechargement à chaud](#config-hot-reload)).

## Validation stricte

> **⚠️** OpenClaw n'accepte que les configurations qui correspondent entièrement au schéma. Les clés inconnues, les types malformés ou les valeurs invalides font que la passerelle **refuse de démarrer**. La seule exception au niveau racine est `$schema` (chaîne), permettant aux éditeurs d'attacher des métadonnées de schéma JSON.

 Lorsque la validation échoue :

-   La passerelle ne démarre pas
-   Seules les commandes de diagnostic fonctionnent (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
-   Exécutez `openclaw doctor` pour voir les problèmes exacts
-   Exécutez `openclaw doctor --fix` (ou `--yes`) pour appliquer des corrections

## Tâches courantes

Chaque canal a sa propre section de configuration sous `channels.`. Consultez la page dédiée au canal pour les étapes d'installation :

-   [WhatsApp](../channels/whatsapp.md) — `channels.whatsapp`
-   [Telegram](../channels/telegram.md) — `channels.telegram`
-   [Discord](../channels/discord.md) — `channels.discord`
-   [Slack](../channels/slack.md) — `channels.slack`
-   [Signal](../channels/signal.md) — `channels.signal`
-   [iMessage](../channels/imessage.md) — `channels.imessage`
-   [Google Chat](../channels/googlechat.md) — `channels.googlechat`
-   [Mattermost](../channels/mattermost.md) — `channels.mattermost`
-   [MS Teams](../channels/msteams.md) — `channels.msteams`

Tous les canaux partagent le même modèle de politique de DM :

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",   // pairing | allowlist | open | disabled
      allowFrom: ["tg:123"], // seulement pour allowlist/open
    },
  },
}
```

Définissez le modèle principal et les modèles de secours optionnels :

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5.2"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "openai/gpt-5.2": { alias: "GPT" },
      },
    },
  },
}
```

-   `agents.defaults.models` définit le catalogue de modèles et sert de liste autorisée pour `/model`.
-   Les références de modèles utilisent le format `provider/model` (ex. `anthropic/claude-opus-4-6`).
-   `agents.defaults.imageMaxDimensionPx` contrôle la réduction d'échelle des images de transcription/outil (par défaut `1200`) ; des valeurs plus basses réduisent généralement l'utilisation de tokens de vision sur les exécutions riches en captures d'écran.
-   Voir [CLI des modèles](../concepts/models.md) pour changer de modèle en chat et [Basculement de modèle](../concepts/model-failover.md) pour la rotation d'authentification et le comportement de secours.
-   Pour les fournisseurs personnalisés/auto-hébergés, voir [Fournisseurs personnalisés](./configuration-reference.md#custom-providers-and-base-urls) dans la référence.

L'accès en DM est contrôlé par canal via `dmPolicy` :

-   `"pairing"` (par défaut) : les expéditeurs inconnus reçoivent un code d'appaiement unique à approuver
-   `"allowlist"` : seuls les expéditeurs dans `allowFrom` (ou le magasin d'autorisations appairé)
-   `"open"` : autoriser tous les DM entrants (nécessite `allowFrom: ["*"]`)
-   `"disabled"` : ignorer tous les DM

Pour les groupes, utilisez `groupPolicy` + `groupAllowFrom` ou des listes d'autorisation spécifiques au canal. Voir la [référence complète](./configuration-reference.md#dm-and-group-access) pour les détails par canal.

Les messages de groupe nécessitent par défaut **une mention**. Configurez les motifs par agent :

```json
{
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw"],
        },
      },
    ],
  },
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

-   **Mentions natives** : mentions @ natives (WhatsApp tap-to-mention, Telegram @bot, etc.)
-   **Motifs textuels** : motifs regex dans `mentionPatterns`
-   Voir la [référence complète](./configuration-reference.md#group-chat-mention-gating) pour les remplacements par canal et le mode self-chat.

Les sessions contrôlent la continuité et l'isolation des conversations :

```json
{
  session: {
    dmScope: "per-channel-peer",  // recommandé pour multi-utilisateur
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
  },
}
```

-   `dmScope` : `main` (partagé) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
-   `threadBindings` : valeurs par défaut globales pour le routage de session lié aux fils (Discord prend en charge `/focus`, `/unfocus`, `/agents`, `/session idle`, et `/session max-age`).
-   Voir [Gestion des sessions](../concepts/session.md) pour la portée, les liens d'identité et la politique d'envoi.
-   Voir la [référence complète](./configuration-reference.md#session) pour tous les champs.

Exécutez les sessions d'agent dans des conteneurs Docker isolés :

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",  // off | non-main | all
        scope: "agent",    // session | agent | shared
      },
    },
  },
}
```

Construisez d'abord l'image : `scripts/sandbox-setup.sh` Voir [Sandboxing](./sandboxing.md) pour le guide complet et la [référence complète](./configuration-reference.md#sandbox) pour toutes les options.

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
  },
}
```

-   `every` : chaîne de durée (`30m`, `2h`). Mettez `0m` pour désactiver.
-   `target` : `last` | `whatsapp` | `telegram` | `discord` | `none`
-   `directPolicy` : `allow` (par défaut) ou `block` pour les cibles de heartbeat de type DM
-   Voir [Heartbeat](./heartbeat.md) pour le guide complet.

```json
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h",
    runLog: {
      maxBytes: "2mb",
      keepLines: 2000,
    },
  },
}
```

-   `sessionRetention` : supprime les sessions d'exécution isolées terminées de `sessions.json` (par défaut `24h` ; mettez `false` pour désactiver).
-   `runLog` : supprime `cron/runs/.jsonl` par taille et lignes conservées.
-   Voir [Tâches cron](../automation/cron-jobs.md) pour une vue d'ensemble de la fonctionnalité et des exemples CLI.

Activez les points de terminaison HTTP webhook sur la passerelle :

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "main",
        deliver: true,
      },
    ],
  },
}
```

Note de sécurité :

-   Traitez tout le contenu des charges utiles de hook/webhook comme une entrée non fiable.
-   Gardez les drapeaux de contournement de contenu non sécurisé désactivés (`hooks.gmail.allowUnsafeExternalContent`, `hooks.mappings[].allowUnsafeExternalContent`) sauf pour du débogage étroitement ciblé.
-   Pour les agents pilotés par hook, préférez des niveaux de modèles modernes robustes et une politique d'outils stricte (par exemple, uniquement la messagerie plus le sandboxing lorsque possible).

Voir la [référence complète](./configuration-reference.md#hooks) pour toutes les options de mappage et l'intégration Gmail.

Exécutez plusieurs agents isolés avec des espaces de travail et sessions séparés :

```json
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

Voir [Multi-Agent](../concepts/multi-agent.md) et la [référence complète](./configuration-reference.md#multi-agent-routing) pour les règles de liaison et les profils d'accès par agent.

Utilisez `$include` pour organiser de grandes configurations :

```
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/a.json5", "./clients/b.json5"],
  },
}
```

-   **Fichier unique** : remplace l'objet conteneur
-   **Tableau de fichiers** : fusion profonde dans l'ordre (le dernier l'emporte)
-   **Clés sœurs** : fusionnées après les inclusions (écrasent les valeurs incluses)
-   **Inclusions imbriquées** : supportées jusqu'à 10 niveaux de profondeur
-   **Chemins relatifs** : résolus par rapport au fichier d'inclusion
-   **Gestion des erreurs** : erreurs claires pour les fichiers manquants, erreurs d'analyse et inclusions circulaires

## Rechargement à chaud de la configuration

La passerelle surveille `~/.openclaw/openclaw.json` et applique les changements automatiquement — aucun redémarrage manuel n'est nécessaire pour la plupart des paramètres.

### Modes de rechargement

| Mode | Comportement |
| --- | --- |
| **`hybrid`** (par défaut) | Applique à chaud les changements sûrs instantanément. Redémarre automatiquement pour les changements critiques. |
| **`hot`** | Applique à chaud uniquement les changements sûrs. Enregistre un avertissement lorsqu'un redémarrage est nécessaire — vous le gérez. |
| **`restart`** | Redémarre la passerelle à chaque changement de configuration, sûr ou non. |
| **`off`** | Désactive la surveillance de fichier. Les changements prennent effet au prochain redémarrage manuel. |

```json
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Ce qui s'applique à chaud vs ce qui nécessite un redémarrage

La plupart des champs s'appliquent à chaud sans interruption. En mode `hybrid`, les changements nécessitant un redémarrage sont gérés automatiquement.

| Catégorie | Champs | Redémarrage nécessaire ? |
| --- | --- | --- |
| Canaux | `channels.*`, `web` (WhatsApp) — tous les canaux intégrés et d'extension | Non |
| Agent & modèles | `agent`, `agents`, `models`, `routing` | Non |
| Automatisation | `hooks`, `cron`, `agent.heartbeat` | Non |
| Sessions & messages | `session`, `messages` | Non |
| Outils & médias | `tools`, `browser`, `skills`, `audio`, `talk` | Non |
| UI & divers | `ui`, `logging`, `identity`, `bindings` | Non |
| Serveur de passerelle | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP) | **Oui** |
| Infrastructure | `discovery`, `canvasHost`, `plugins` | **Oui** |

> **ℹ️** `gateway.reload` et `gateway.remote` sont des exceptions — les modifier ne déclenche **pas** de redémarrage.

## RPC de configuration (mises à jour programmatiques)

> **ℹ️** Les RPC d'écriture du plan de contrôle (`config.apply`, `config.patch`, `update.run`) sont limités à **3 requêtes par 60 secondes** par `deviceId+clientIp`. Lorsque limité, le RPC retourne `UNAVAILABLE` avec `retryAfterMs`.

 

Valide + écrit la configuration complète et redémarre la passerelle en une étape.

`config.apply` remplace la **configuration entière**. Utilisez `config.patch` pour des mises à jour partielles, ou `openclaw config set` pour des clés uniques.

Paramètres :

-   `raw` (chaîne) — charge utile JSON5 pour toute la configuration
-   `baseHash` (optionnel) — hash de configuration depuis `config.get` (requis si la configuration existe)
-   `sessionKey` (optionnel) — clé de session pour le ping de réveil post-redémarrage
-   `note` (optionnel) — note pour le sentinelle de redémarrage
-   `restartDelayMs` (optionnel) — délai avant redémarrage (par défaut 2000)

Les demandes de redémarrage sont fusionnées lorsqu'une est déjà en attente/en cours, et un délai de 30 secondes s'applique entre les cycles de redémarrage.

```bash
openclaw gateway call config.get --params '{}'  # capturer payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
  "baseHash": "<hash>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123"
}'
```

Fusionne une mise à jour partielle dans la configuration existante (sémantique de correctif de fusion JSON) :

-   Les objets fusionnent récursivement
-   `null` supprime une clé
-   Les tableaux se remplacent

Paramètres :

-   `raw` (chaîne) — JSON5 avec uniquement les clés à modifier
-   `baseHash` (requis) — hash de configuration depuis `config.get`
-   `sessionKey`, `note`, `restartDelayMs` — identiques à `config.apply`

Le comportement de redémarrage correspond à `config.apply` : redémarrages en attente fusionnés plus un délai de 30 secondes entre les cycles de redémarrage.

```bash
openclaw gateway call config.patch --params '{
  "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
  "baseHash": "<hash>"
}'
```

## Variables d'environnement

OpenClaw lit les variables d'env depuis le processus parent plus :

-   `.env` depuis le répertoire de travail courant (si présent)
-   `~/.openclaw/.env` (repli global)

Aucun fichier ne remplace les variables d'env existantes. Vous pouvez également définir des variables d'env en ligne dans la configuration :

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

Si activé et que les clés attendues ne sont pas définies, OpenClaw exécute votre shell de connexion et importe uniquement les clés manquantes :

```json
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Variable d'env équivalente : `OPENCLAW_LOAD_SHELL_ENV=1`

 

Référencez les variables d'env dans toute valeur de chaîne de configuration avec `${VAR_NAME}` :

```json
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Règles :

-   Seuls les noms en majuscules sont reconnus : `[A-Z_][A-Z0-9_]*`
-   Les variables manquantes/vides génèrent une erreur au chargement
-   Échappez avec `$${VAR}` pour une sortie littérale
-   Fonctionne dans les fichiers `$include`
-   Substitution en ligne : `"${BASE}/v1"` → `"https://api.example.com/v1"`

 

Pour les champs qui prennent en charge les objets SecretRef, vous pouvez utiliser :

```json
{
  models: {
    providers: {
      openai: { apiKey: { source: "env", provider: "default", id: "OPENAI_API_KEY" } },
    },
  },
  skills: {
    entries: {
      "nano-banana-pro": {
        apiKey: {
          source: "file",
          provider: "filemain",
          id: "/skills/entries/nano-banana-pro/apiKey",
        },
      },
    },
  },
  channels: {
    googlechat: {
      serviceAccountRef: {
        source: "exec",
        provider: "vault",
        id: "channels/googlechat/serviceAccount",
      },
    },
  },
}
```

Les détails de SecretRef (y compris `secrets.providers` pour `env`/`file`/`exec`) sont dans [Gestion des secrets](./secrets.md). Les chemins d'accès aux identifiants pris en charge sont listés dans [Surface d'identifiants SecretRef](../reference/secretref-credential-surface.md).

 Voir [Environnement](../help/environment.md) pour la préséance et les sources complètes.

## Référence complète

Pour la référence complète champ par champ, voir **[Référence de configuration](./configuration-reference.md)**.

* * *

*Liés : [Exemples de configuration](./configuration-examples.md) · [Référence de configuration](./configuration-reference.md) · [Doctor](./doctor.md)*

[Runbook de la passerelle](../gateway.md)[Référence de configuration](./configuration-reference.md)