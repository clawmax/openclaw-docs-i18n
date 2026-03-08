title: "Routage et Configuration Multi-Agents dans OpenClaw Gateway"
description: "Apprenez à configurer plusieurs agents IA isolés avec des espaces de travail, des comptes de canaux et des règles de routage distincts dans OpenClaw pour gérer des personas et des sessions distinctes."
keywords: ["routage multi-agent", "agents openclaw", "agents isolés", "liaisons de canaux", "espace de travail agent", "gestion de session", "routage whatsapp", "agents bot discord"]
---

  Multi-agent

  
# Routage Multi-Agent

Objectif : plusieurs agents *isolés* (espace de travail + `agentDir` + sessions séparés), plus plusieurs comptes de canaux (par ex. deux WhatsApp) dans une seule Gateway en cours d'exécution. Les messages entrants sont routés vers un agent via des liaisons (bindings).

## Qu'est-ce qu'« un agent » ?

Un **agent** est un cerveau entièrement délimité avec ses propres :

-   **Espace de travail** (fichiers, AGENTS.md/SOUL.md/USER.md, notes locales, règles de persona).
-   **Répertoire d'état** (`agentDir`) pour les profils d'authentification, le registre de modèles et la configuration par agent.
-   **Stockage de session** (historique de chat + état de routage) sous `~/.openclaw/agents//sessions`.

Les profils d'authentification sont **par agent**. Chaque agent lit depuis son propre :

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Les identifiants principaux de l'agent **ne sont pas** partagés automatiquement. Ne réutilisez jamais `agentDir` entre agents (cela provoque des collisions d'authentification/session). Si vous souhaitez partager des identifiants, copiez `auth-profiles.json` dans le `agentDir` de l'autre agent. Les compétences (skills) sont par agent via le dossier `skills/` de chaque espace de travail, avec des compétences partagées disponibles depuis `~/.openclaw/skills`. Voir [Compétences : par agent vs partagées](../tools/skills.md#per-agent-vs-shared-skills). La Gateway peut héberger **un agent** (par défaut) ou **plusieurs agents** côte à côte. **Note sur l'espace de travail :** l'espace de travail de chaque agent est le **cwd par défaut**, pas un bac à sable (sandbox) strict. Les chemins relatifs se résolvent à l'intérieur de l'espace de travail, mais les chemins absolus peuvent atteindre d'autres emplacements de l'hôte à moins que le bac à sable ne soit activé. Voir [Bac à sable (Sandboxing)](../gateway/sandboxing.md).

## Chemins (carte rapide)

-   Config : `~/.openclaw/openclaw.json` (ou `OPENCLAW_CONFIG_PATH`)
-   Répertoire d'état : `~/.openclaw` (ou `OPENCLAW_STATE_DIR`)
-   Espace de travail : `~/.openclaw/workspace` (ou `~/.openclaw/workspace-`)
-   Répertoire agent : `~/.openclaw/agents//agent` (ou `agents.list[].agentDir`)
-   Sessions : `~/.openclaw/agents//sessions`

### Mode mono-agent (par défaut)

Si vous ne faites rien, OpenClaw exécute un seul agent :

-   `agentId` vaut par défaut **`main`**.
-   Les sessions sont clés comme `agent:main:`.
-   L'espace de travail par défaut est `~/.openclaw/workspace` (ou `~/.openclaw/workspace-` lorsque `OPENCLAW_PROFILE` est défini).
-   L'état par défaut est `~/.openclaw/agents/main/agent`.

## Assistant agent

Utilisez l'assistant agent pour ajouter un nouvel agent isolé :

```bash
openclaw agents add work
```

Ensuite, ajoutez des `bindings` (ou laissez l'assistant le faire) pour router les messages entrants. Vérifiez avec :

```bash
openclaw agents list --bindings
```

## Démarrage rapide

### Étape 1 : Créer chaque espace de travail d'agent

Utilisez l'assistant ou créez les espaces de travail manuellement :

```bash
openclaw agents add coding
openclaw agents add social
```

Chaque agent obtient son propre espace de travail avec `SOUL.md`, `AGENTS.md`, et optionnellement `USER.md`, plus un `agentDir` dédié et un stockage de session sous `~/.openclaw/agents/`.

### Étape 2 : Créer des comptes de canaux

Créez un compte par agent sur vos canaux préférés :

-   Discord : un bot par agent, activez l'intention Message Content, copiez chaque token.
-   Telegram : un bot par agent via BotFather, copiez chaque token.
-   WhatsApp : liez chaque numéro de téléphone par compte.

```bash
openclaw channels login --channel whatsapp --account work
```

Voir les guides des canaux : [Discord](../channels/discord.md), [Telegram](../channels/telegram.md), [WhatsApp](../channels/whatsapp.md).

### Étape 3 : Ajouter agents, comptes et liaisons

Ajoutez les agents sous `agents.list`, les comptes de canaux sous `channels..accounts`, et connectez-les avec des `bindings` (exemples ci-dessous).

### Étape 4 : Redémarrer et vérifier

```bash
openclaw gateway restart
openclaw agents list --bindings
openclaw channels status --probe
```

## Agents multiples = personnes multiples, personnalités multiples

Avec **plusieurs agents**, chaque `agentId` devient une **persona entièrement isolée** :

-   **Numéros de téléphone/comptes différents** (par `accountId` de canal).
-   **Personnalités différentes** (fichiers d'espace de travail par agent comme `AGENTS.md` et `SOUL.md`).
-   **Authentification + sessions séparées** (pas d'interférence croisée sauf si explicitement activé).

Cela permet à **plusieurs personnes** de partager un serveur Gateway tout en gardant leurs « cerveaux » IA et données isolés.

## Un numéro WhatsApp, plusieurs personnes (division des MP)

Vous pouvez router **différents MP WhatsApp** vers différents agents tout en restant sur **un seul compte WhatsApp**. Faites correspondre l'expéditeur E.164 (comme `+15551234567`) avec `peer.kind: "direct"`. Les réponses proviennent toujours du même numéro WhatsApp (pas d'identité d'expéditeur par agent). Détail important : les chats directs se réduisent à la **clé de session principale** de l'agent, donc une véritable isolation nécessite **un agent par personne**. Exemple :

```json
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" },
    ],
  },
  bindings: [
    {
      agentId: "alex",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230001" } },
    },
    {
      agentId: "mia",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230002" } },
    },
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"],
    },
  },
}
```

Notes :

-   Le contrôle d'accès aux MP est **global par compte WhatsApp** (appariement/liste d'autorisation), pas par agent.
-   Pour les groupes partagés, liez le groupe à un agent ou utilisez [Groupes de diffusion](../channels/broadcast-groups.md).

## Règles de routage (comment les messages choisissent un agent)

Les liaisons (bindings) sont **déterministes** et **le plus spécifique l'emporte** :

1.  Correspondance `peer` (id exact de MP/groupe/canal)
2.  Correspondance `parentPeer` (héritage de fil de discussion)
3.  `guildId + roles` (routage par rôle Discord)
4.  `guildId` (Discord)
5.  `teamId` (Slack)
6.  Correspondance `accountId` pour un canal
7.  Correspondance au niveau du canal (`accountId: "*"`)
8.  Retour à l'agent par défaut (`agents.list[].default`, sinon première entrée de la liste, par défaut : `main`)

Si plusieurs liaisons correspondent dans le même niveau, la première dans l'ordre de configuration l'emporte. Si une liaison définit plusieurs champs de correspondance (par exemple `peer` + `guildId`), tous les champs spécifiés sont requis (sémantique `ET`). Détail important sur la portée du compte :

-   Une liaison qui omet `accountId` ne correspond qu'au compte par défaut.
-   Utilisez `accountId: "*"` pour un retour global sur le canal pour tous les comptes.
-   Si vous ajoutez plus tard la même liaison pour le même agent avec un id de compte explicite, OpenClaw met à niveau la liaison existante uniquement sur le canal pour qu'elle soit limitée au compte au lieu de la dupliquer.

## Comptes multiples / numéros de téléphone multiples

Les canaux qui prennent en charge **plusieurs comptes** (par ex. WhatsApp) utilisent `accountId` pour identifier chaque connexion. Chaque `accountId` peut être routé vers un agent différent, donc un serveur peut héberger plusieurs numéros de téléphone sans mélanger les sessions. Si vous voulez un compte par défaut global pour un canal lorsque `accountId` est omis, définissez `channels..defaultAccount` (optionnel). Si non défini, OpenClaw revient à `default` s'il est présent, sinon au premier id de compte configuré (trié). Les canaux courants prenant en charge ce modèle incluent :

-   `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`
-   `irc`, `line`, `googlechat`, `mattermost`, `matrix`, `nextcloud-talk`
-   `bluebubbles`, `zalo`, `zalouser`, `nostr`, `feishu`

## Concepts

-   `agentId` : un « cerveau » (espace de travail, authentification par agent, stockage de session par agent).
-   `accountId` : une instance de compte de canal (par ex. compte WhatsApp `"personal"` vs `"biz"`).
-   `binding` : route les messages entrants vers un `agentId` par `(channel, accountId, peer)` et optionnellement des ids de guilde/équipe.
-   Les chats directs se réduisent à `agent::` (« main » par agent ; `session.mainKey`).

## Exemples par plateforme

### Bots Discord par agent

Chaque compte de bot Discord correspond à un `accountId` unique. Liez chaque compte à un agent et gardez des listes d'autorisation par bot.

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "coding", workspace: "~/.openclaw/workspace-coding" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "discord", accountId: "default" } },
    { agentId: "coding", match: { channel: "discord", accountId: "coding" } },
  ],
  channels: {
    discord: {
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN_MAIN",
          guilds: {
            "123456789012345678": {
              channels: {
                "222222222222222222": { allow: true, requireMention: false },
              },
            },
          },
        },
        coding: {
          token: "DISCORD_BOT_TOKEN_CODING",
          guilds: {
            "123456789012345678": {
              channels: {
                "333333333333333333": { allow: true, requireMention: false },
              },
            },
          },
        },
      },
    },
  },
}
```

Notes :

-   Invitez chaque bot dans la guilde et activez l'intention Message Content.
-   Les tokens se trouvent dans `channels.discord.accounts..token` (le compte par défaut peut utiliser `DISCORD_BOT_TOKEN`).

### Bots Telegram par agent

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "alerts", workspace: "~/.openclaw/workspace-alerts" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "alerts", match: { channel: "telegram", accountId: "alerts" } },
  ],
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "123456:ABC...",
          dmPolicy: "pairing",
        },
        alerts: {
          botToken: "987654:XYZ...",
          dmPolicy: "allowlist",
          allowFrom: ["tg:123456789"],
        },
      },
    },
  },
}
```

Notes :

-   Créez un bot par agent avec BotFather et copiez chaque token.
-   Les tokens se trouvent dans `channels.telegram.accounts..botToken` (le compte par défaut peut utiliser `TELEGRAM_BOT_TOKEN`).

### Numéros WhatsApp par agent

Lie chaque compte avant de démarrer la gateway :

```bash
openclaw channels login --channel whatsapp --account personal
openclaw channels login --channel whatsapp --account biz
```

`~/.openclaw/openclaw.json` (JSON5) :

```json
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Routage déterministe : première correspondance gagne (la plus spécifique en premier).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Surcharge optionnelle par pair (exemple : envoyer un groupe spécifique à l'agent work).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Désactivé par défaut : la messagerie agent-à-agent doit être explicitement activée + autorisée.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Surcharge optionnelle. Par défaut : ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Surcharge optionnelle. Par défaut : ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

## Exemple : Chat quotidien WhatsApp + Travail approfondi Telegram

Diviser par canal : router WhatsApp vers un agent rapide quotidien et Telegram vers un agent Opus.

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

Notes :

-   Si vous avez plusieurs comptes pour un canal, ajoutez `accountId` à la liaison (par exemple `{ channel: "whatsapp", accountId: "personal" }`).
-   Pour router un seul MP/groupe vers Opus tout en gardant le reste sur chat, ajoutez une liaison `match.peer` pour ce pair ; les correspondances de pair gagnent toujours sur les règles globales du canal.

## Exemple : même canal, un pair vers Opus

Gardez WhatsApp sur l'agent rapide, mais routez un MP vers Opus :

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    {
      agentId: "opus",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551234567" } },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
  ],
}
```

Les liaisons de pair gagnent toujours, donc gardez-les au-dessus de la règle globale du canal.

## Agent familial lié à un groupe WhatsApp

Lie un agent familial dédié à un seul groupe WhatsApp, avec mention conditionnelle et une politique d'outils plus stricte :

```json
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"],
        },
        sandbox: {
          mode: "all",
          scope: "agent",
        },
        tools: {
          allow: [
            "exec",
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"],
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" },
      },
    },
  ],
}
```

Notes :

-   Les listes d'autorisation/interdiction d'outils concernent les **outils**, pas les compétences (skills). Si une compétence a besoin d'exécuter un binaire, assurez-vous que `exec` est autorisé et que le binaire existe dans le bac à sable.
-   Pour un conditionnement plus strict, définissez `agents.list[].groupChat.mentionPatterns` et gardez les listes d'autorisation de groupes activées pour le canal.

## Configuration du Bac à Sable et des Outils par Agent

À partir de la v2026.1.6, chaque agent peut avoir ses propres restrictions de bac à sable et d'outils :

```json
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // Pas de bac à sable pour l'agent personnel
        },
        // Pas de restrictions d'outils - tous les outils disponibles
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Toujours en bac à sable
          scope: "agent",  // Un conteneur par agent
          docker: {
            // Configuration unique optionnelle après la création du conteneur
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Seul l'outil read
          deny: ["exec", "write", "edit", "apply_patch"],    // Interdire les autres
        },
      },
    ],
  },
}
```

Note : `setupCommand` se trouve sous `sandbox.docker` et s'exécute une fois à la création du conteneur. Les surcharges `sandbox.docker.*` par agent sont ignorées lorsque la portée résolue est `"shared"`. **Avantages :**

-   **Isolation de sécurité** : Restreindre les outils pour les agents non fiables
-   **Contrôle des ressources** : Mettre en bac à sable des agents spécifiques tout en gardant les autres sur l'hôte
-   **Politiques flexibles** : Permissions différentes par agent

Note : `tools.elevated` est **global** et basé sur l'expéditeur ; il n'est pas configurable par agent. Si vous avez besoin de limites par agent, utilisez `agents.list[].tools` pour interdire `exec`. Pour le ciblage de groupe, utilisez `agents.list[].groupChat.mentionPatterns` pour que les @mentions correspondent clairement à l'agent souhaité. Voir [Bac à Sable & Outils Multi-Agents](../tools/multi-agent-sandbox-tools.md) pour des exemples détaillés.

[Compaction](./compaction.md)[Présence](./presence.md)