

  Automatisation

  
# Hooks

Les hooks fournissent un système extensible piloté par événements pour automatiser des actions en réponse aux commandes et événements de l'agent. Les hooks sont découverts automatiquement depuis des répertoires et peuvent être gérés via des commandes CLI, de manière similaire au fonctionnement des compétences dans OpenClaw.

## S'orienter

Les hooks sont de petits scripts qui s'exécutent lorsque quelque chose se produit. Il en existe deux types :

-   **Hooks** (cette page) : s'exécutent à l'intérieur de la Gateway lorsque des événements de l'agent se déclenchent, comme `/new`, `/reset`, `/stop`, ou des événements du cycle de vie.
-   **Webhooks** : webhooks HTTP externes qui permettent à d'autres systèmes de déclencher du travail dans OpenClaw. Voir [Webhook Hooks](./webhook.md) ou utilisez `openclaw webhooks` pour les commandes d'aide Gmail.

Les hooks peuvent également être regroupés dans des plugins ; voir [Plugins](../tools/plugin.md#plugin-hooks). Utilisations courantes :

-   Sauvegarder un instantané de mémoire lorsque vous réinitialisez une session
-   Conserver une piste d'audit des commandes pour le dépannage ou la conformité
-   Déclencher une automatisation de suivi lorsqu'une session commence ou se termine
-   Écrire des fichiers dans l'espace de travail de l'agent ou appeler des API externes lorsque des événements se déclenchent

Si vous pouvez écrire une petite fonction TypeScript, vous pouvez écrire un hook. Les hooks sont découverts automatiquement, et vous les activez ou désactivez via la CLI.

## Vue d'ensemble

Le système de hooks vous permet de :

-   Sauvegarder le contexte de session en mémoire lorsque `/new` est émis
-   Journaliser toutes les commandes pour l'audit
-   Déclencher des automatisations personnalisées sur les événements du cycle de vie de l'agent
-   Étendre le comportement d'OpenClaw sans modifier le code central

## Premiers pas

### Hooks intégrés

OpenClaw est livré avec quatre hooks intégrés qui sont découverts automatiquement :

-   **💾 session-memory** : Sauvegarde le contexte de session dans votre espace de travail de l'agent (par défaut `~/.openclaw/workspace/memory/`) lorsque vous émettez `/new`
-   **📎 bootstrap-extra-files** : Injecte des fichiers d'amorçage d'espace de travail supplémentaires depuis des modèles de chemin/glob configurés pendant `agent:bootstrap`
-   **📝 command-logger** : Journalise tous les événements de commande dans `~/.openclaw/logs/commands.log`
-   **🚀 boot-md** : Exécute `BOOT.md` au démarrage de la gateway (nécessite l'activation des hooks internes)

Lister les hooks disponibles :

```bash
openclaw hooks list
```

Activer un hook :

```bash
openclaw hooks enable session-memory
```

Vérifier l'état des hooks :

```bash
openclaw hooks check
```

Obtenir des informations détaillées :

```bash
openclaw hooks info session-memory
```

### Intégration

Pendant l'intégration (`openclaw onboard`), vous serez invité à activer les hooks recommandés. L'assistant découvre automatiquement les hooks éligibles et les présente pour sélection.

## Découverte des hooks

Les hooks sont automatiquement découverts depuis trois répertoires (par ordre de priorité) :

1.  **Hooks de l'espace de travail** : `/hooks/` (par agent, priorité la plus élevée)
2.  **Hooks gérés** : `~/.openclaw/hooks/` (installés par l'utilisateur, partagés entre les espaces de travail)
3.  **Hooks intégrés** : `/dist/hooks/bundled/` (livrés avec OpenClaw)

Les répertoires de hooks gérés peuvent être soit un **hook unique** soit un **pack de hooks** (répertoire de package). Chaque hook est un répertoire contenant :

```
my-hook/
├── HOOK.md          # Métadonnées + documentation
└── handler.ts       # Implémentation du gestionnaire
```

## Packs de hooks (npm/archives)

Les packs de hooks sont des packages npm standard qui exportent un ou plusieurs hooks via `openclaw.hooks` dans `package.json`. Installez-les avec :

```bash
openclaw hooks install <path-or-spec>
```

Les spécifications npm sont uniquement pour le registre (nom du package + version exacte ou dist-tag optionnelle). Les spécifications Git/URL/fichier et les plages semver sont rejetées. Les spécifications nues et `@latest` restent sur la voie stable. Si npm résout l'une d'elles en une préversion, OpenClaw s'arrête et vous demande d'accepter explicitement avec un tag de préversion tel que `@beta`/`@rc` ou une version de préversion exacte. Exemple de `package.json` :

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Chaque entrée pointe vers un répertoire de hook contenant `HOOK.md` et `handler.ts` (ou `index.ts`). Les packs de hooks peuvent embarquer des dépendances ; elles seront installées sous `~/.openclaw/hooks/`. Chaque entrée `openclaw.hooks` doit rester à l'intérieur du répertoire du package après résolution des liens symboliques ; les entrées qui en sortent sont rejetées. Note de sécurité : `openclaw hooks install` installe les dépendances avec `npm install --ignore-scripts` (pas de scripts de cycle de vie). Gardez les arbres de dépendances des packs de hooks "purs JS/TS" et évitez les packages qui dépendent de builds `postinstall`.

## Structure d'un hook

### Format de HOOK.md

Le fichier `HOOK.md` contient des métadonnées dans un frontmatter YAML ainsi qu'une documentation Markdown :

```
---
name: my-hook
description: "Courte description de ce que fait ce hook"
homepage: https://docs.openclaw.ai/automation/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# Mon Hook

La documentation détaillée va ici...

## Ce qu'il fait

- Écoute les commandes `/new`
- Effectue une action
- Journalise le résultat

## Prérequis

- Node.js doit être installé

## Configuration

Aucune configuration nécessaire.
```

### Champs de métadonnées

L'objet `metadata.openclaw` prend en charge :

-   **`emoji`** : Emoji d'affichage pour la CLI (ex. `"💾"`)
-   **`events`** : Tableau des événements à écouter (ex. `["command:new", "command:reset"]`)
-   **`export`** : Export nommé à utiliser (par défaut `"default"`)
-   **`homepage`** : URL de documentation
-   **`requires`** : Prérequis optionnels
    -   **`bins`** : Binaires requis dans le PATH (ex. `["git", "node"]`)
    -   **`anyBins`** : Au moins un de ces binaires doit être présent
    -   **`env`** : Variables d'environnement requises
    -   **`config`** : Chemins de configuration requis (ex. `["workspace.dir"]`)
    -   **`os`** : Plateformes requises (ex. `["darwin", "linux"]`)
-   **`always`** : Contourne les vérifications d'éligibilité (booléen)
-   **`install`** : Méthodes d'installation (pour les hooks intégrés : `[{"id":"bundled","kind":"bundled"}]`)

### Implémentation du gestionnaire

Le fichier `handler.ts` exporte une fonction `HookHandler` :

```typescript
const myHandler = async (event) => {
  // Ne se déclenche que sur la commande 'new'
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] Commande new déclenchée`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Horodatage: ${event.timestamp.toISOString()}`);

  // Votre logique personnalisée ici

  // Optionnellement, envoyer un message à l'utilisateur
  event.messages.push("✨ Mon hook exécuté !");
};

export default myHandler;
```

#### Contexte de l'événement

Chaque événement inclut :

```json
{
  type: 'command' | 'session' | 'agent' | 'gateway' | 'message',
  action: string,              // ex. 'new', 'reset', 'stop', 'received', 'sent'
  sessionKey: string,          // Identifiant de session
  timestamp: Date,             // Quand l'événement s'est produit
  messages: string[],          // Poussez des messages ici pour les envoyer à l'utilisateur
  context: {
    // Événements de commande :
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // ex. 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig,
    // Événements de message (voir la section Événements de message pour les détails complets) :
    from?: string,             // message:received
    to?: string,               // message:sent
    content?: string,
    channelId?: string,
    success?: boolean,         // message:sent
  }
}
```

## Types d'événements

### Événements de commande

Déclenchés lorsque des commandes de l'agent sont émises :

-   **`command`** : Tous les événements de commande (écouteur général)
-   **`command:new`** : Lorsque la commande `/new` est émise
-   **`command:reset`** : Lorsque la commande `/reset` est émise
-   **`command:stop`** : Lorsque la commande `/stop` est émise

### Événements de session

-   **`session:compact:before`** : Juste avant que la compaction ne résume l'historique
-   **`session:compact:after`** : Après que la compaction est terminée avec les métadonnées de résumé

Les charges utiles des hooks internes émettent ceux-ci comme `type: "session"` avec `action: "compact:before"` / `action: "compact:after"` ; les écouteurs s'abonnent avec les clés combinées ci-dessus. L'enregistrement spécifique du gestionnaire utilise le format de clé littérale `${type}:${action}`. Pour ces événements, enregistrez `session:compact:before` et `session:compact:after`.

### Événements de l'agent

-   **`agent:bootstrap`** : Avant que les fichiers d'amorçage de l'espace de travail ne soient injectés (les hooks peuvent muter `context.bootstrapFiles`)

### Événements de la gateway

Déclenchés au démarrage de la gateway :

-   **`gateway:startup`** : Après le démarrage des canaux et le chargement des hooks

### Événements de message

Déclenchés lorsque des messages sont reçus ou envoyés :

-   **`message`** : Tous les événements de message (écouteur général)
-   **`message:received`** : Lorsqu'un message entrant est reçu depuis n'importe quel canal. Se déclenche tôt dans le traitement avant la compréhension des médias. Le contenu peut contenir des espaces réservés bruts comme `<media:audio>` pour les pièces jointes multimédias qui n'ont pas encore été traitées.
-   **`message:transcribed`** : Lorsqu'un message a été entièrement traité, y compris la transcription audio et la compréhension des liens. À ce stade, `transcript` contient le texte complet de la transcription pour les messages audio. Utilisez ce hook lorsque vous avez besoin d'accéder au contenu audio transcrit.
-   **`message:preprocessed`** : Se déclenche pour chaque message après que toute la compréhension des médias + liens est terminée, donnant aux hooks accès au corps entièrement enrichi (transcriptions, descriptions d'images, résumés de liens) avant que l'agent ne le voie.
-   **`message:sent`** : Lorsqu'un message sortant est envoyé avec succès

#### Contexte de l'événement de message

Les événements de message incluent un contexte riche sur le message :

```
// contexte de message:received
{
  from: string,           // Identifiant de l'expéditeur (numéro de téléphone, ID utilisateur, etc.)
  content: string,        // Contenu du message
  timestamp?: number,     // Horodatage Unix de la réception
  channelId: string,      // Canal (ex. "whatsapp", "telegram", "discord")
  accountId?: string,     // ID de compte du fournisseur pour les configurations multi-comptes
  conversationId?: string, // ID de conversation/chat
  messageId?: string,     // ID du message provenant du fournisseur
  metadata?: {            // Données supplémentaires spécifiques au fournisseur
    to?: string,
    provider?: string,
    surface?: string,
    threadId?: string,
    senderId?: string,
    senderName?: string,
    senderUsername?: string,
    senderE164?: string,
  }
}

// contexte de message:sent
{
  to: string,             // Identifiant du destinataire
  content: string,        // Contenu du message qui a été envoyé
  success: boolean,       // Indique si l'envoi a réussi
  error?: string,         // Message d'erreur si l'envoi a échoué
  channelId: string,      // Canal (ex. "whatsapp", "telegram", "discord")
  accountId?: string,     // ID de compte du fournisseur
  conversationId?: string, // ID de conversation/chat
  messageId?: string,     // ID du message retourné par le fournisseur
  isGroup?: boolean,      // Indique si ce message sortant appartient à un contexte de groupe/canal
  groupId?: string,       // Identifiant de groupe/canal pour la corrélation avec message:received
}

// contexte de message:transcribed
{
  body?: string,          // Corps entrant brut avant enrichissement
  bodyForAgent?: string,  // Corps enrichi visible par l'agent
  transcript: string,     // Texte de la transcription audio
  channelId: string,      // Canal (ex. "telegram", "whatsapp")
  conversationId?: string,
  messageId?: string,
}

// contexte de message:preprocessed
{
  body?: string,          // Corps entrant brut
  bodyForAgent?: string,  // Corps final enrichi après compréhension des médias/liens
  transcript?: string,    // Transcription lorsqu'un audio était présent
  channelId: string,      // Canal (ex. "telegram", "whatsapp")
  conversationId?: string,
  messageId?: string,
  isGroup?: boolean,
  groupId?: string,
}
```

#### Exemple : Hook de journalisation des messages

```typescript
const isMessageReceivedEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "received";
const isMessageSentEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "sent";

const handler = async (event) => {
  if (isMessageReceivedEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Reçu de ${event.context.from}: ${event.context.content}`);
  } else if (isMessageSentEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Envoyé à ${event.context.to}: ${event.context.content}`);
  }
};

export default handler;
```

### Hooks de résultat d'outil (API Plugin)

Ces hooks ne sont pas des écouteurs de flux d'événements ; ils permettent aux plugins d'ajuster de manière synchrone les résultats d'outils avant qu'OpenClaw ne les persiste.

-   **`tool_result_persist`** : transforme les résultats d'outils avant qu'ils ne soient écrits dans la transcription de la session. Doit être synchrone ; retourne la charge utile du résultat d'outil mise à jour ou `undefined` pour la garder telle quelle. Voir [Boucle de l'agent](../concepts/agent-loop.md).

### Événements de hook de plugin

Hooks du cycle de vie de la compaction exposés via le gestionnaire de hooks du plugin :

-   **`before_compaction`** : S'exécute avant la compaction avec les métadonnées de comptage/jeton
-   **`after_compaction`** : S'exécute après la compaction avec les métadonnées de résumé de la compaction

### Événements futurs

Types d'événements prévus :

-   **`session:start`** : Lorsqu'une nouvelle session commence
-   **`session:end`** : Lorsqu'une session se termine
-   **`agent:error`** : Lorsqu'un agent rencontre une erreur

## Création de hooks personnalisés

### 1\. Choisir l'emplacement

-   **Hooks de l'espace de travail** (`/hooks/`) : Par agent, priorité la plus élevée
-   **Hooks gérés** (`~/.openclaw/hooks/`) : Partagés entre les espaces de travail

### 2\. Créer la structure de répertoire

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3\. Créer HOOK.md

```
---
name: my-hook
description: "Fait quelque chose d'utile"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# Mon Hook Personnalisé

Ce hook fait quelque chose d'utile lorsque vous émettez `/new`.
```

### 4\. Créer handler.ts

```typescript
const handler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Exécution !");
  // Votre logique ici
};

export default handler;
```

### 5\. Activer et tester

```bash
# Vérifier que le hook est découvert
openclaw hooks list

# L'activer
openclaw hooks enable my-hook

# Redémarrer votre processus de gateway (redémarrage de l'application de la barre des tâches sur macOS, ou redémarrage de votre processus de développement)

# Déclencher l'événement
# Envoyer /new via votre canal de messagerie
```

## Configuration

### Nouveau format de configuration (Recommandé)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### Configuration par hook

Les hooks peuvent avoir une configuration personnalisée :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### Répertoires supplémentaires

Charger des hooks depuis des répertoires supplémentaires :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### Format de configuration hérité (Toujours pris en charge)

L'ancien format de configuration fonctionne toujours pour la rétrocompatibilité :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

Note : `module` doit être un chemin relatif à l'espace de travail. Les chemins absolus et les traversées en dehors de l'espace de travail sont rejetés. **Migration** : Utilisez le nouveau système basé sur la découverte pour les nouveaux hooks. Les gestionnaires hérités sont chargés après les hooks basés sur les répertoires.

## Commandes CLI

### Lister les hooks

```bash
# Lister tous les hooks
openclaw hooks list

# Afficher uniquement les hooks éligibles
openclaw hooks list --eligible

# Sortie verbeuse (afficher les prérequis manquants)
openclaw hooks list --verbose

# Sortie JSON
openclaw hooks list --json
```

### Informations sur un hook

```bash
# Afficher des informations détaillées sur un hook
openclaw hooks info session-memory

# Sortie JSON
openclaw hooks info session-memory --json
```

### Vérifier l'éligibilité

```bash
# Afficher un résumé d'éligibilité
openclaw hooks check

# Sortie JSON
openclaw hooks check --json
```

### Activer/Désactiver

```bash
# Activer un hook
openclaw hooks enable session-memory

# Désactiver un hook
openclaw hooks disable command-logger
```

## Référence des hooks intégrés

### session-memory

Sauvegarde le contexte de session en mémoire lorsque vous émettez `/new`. **Événements** : `command:new` **Prérequis** : `workspace.dir` doit être configuré **Sortie** : `/memory/YYYY-MM-DD-slug.md` (par défaut `~/.openclaw/workspace`) **Ce qu'il fait** :

1.  Utilise l'entrée de session pré-réinitialisation pour localiser la bonne transcription
2.  Extrait les 15 dernières lignes de conversation
3.  Utilise un LLM pour générer un slug de nom de fichier descriptif
4.  Sauvegarde les métadonnées de session dans un fichier de mémoire daté

**Exemple de sortie** :

```bash
# Session: 2026-01-16 14:30:00 UTC

- **Clé de session**: agent:main:main
- **ID de session**: abc123def456
- **Source**: telegram
```

**Exemples de noms de fichier** :

-   `2026-01-16-vendor-pitch.md`
-   `2026-01-16-api-design.md`
-   `2026-01-16-1430.md` (horodatage de secours si la génération du slug échoue)

**Activer** :

```bash
openclaw hooks enable session-memory
```

### bootstrap-extra-files

Injecte des fichiers d'amorçage supplémentaires (par exemple `AGENTS.md` / `TOOLS.md` locaux au monorepo) pendant `agent:bootstrap`. **Événements** : `agent:bootstrap` **Prérequis** : `workspace.dir` doit être configuré **Sortie** : Aucun fichier écrit ; le contexte d'amorçage est modifié uniquement en mémoire. **Configuration** :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "bootstrap-extra-files": {
          "enabled": true,
          "paths": ["packages/*/AGENTS.md", "packages/*/TOOLS.md"]
        }
      }
    }
  }
}
```

**Notes** :

-   Les chemins sont résolus relativement à l'espace de travail.
-   Les fichiers doivent rester à l'intérieur de l'espace de travail (vérification realpath).
-   Seuls les noms de base d'amorçage reconnus sont chargés.
-   La liste d'autorisation des sous-agents est préservée (`AGENTS.md` et `TOOLS.md` uniquement).

**Activer** :

```bash
openclaw hooks enable bootstrap-extra-files
```

### command-logger

Journalise tous les événements de commande dans un fichier d'audit centralisé. **Événements** : `command` **Prérequis** : Aucun **Sortie** : `~/.openclaw/logs/commands.log` **Ce qu'il fait** :

1.  Capture les détails de l'événement (action de commande, horodatage, clé de session, ID de l'expéditeur, source)
2.  Ajoute au fichier journal au format JSONL
3.  S'exécute silencieusement en arrière-plan

**Exemples d'entrées de journal** :

```json
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Voir les journaux** :

```bash
# Voir les commandes récentes
tail -n 20 ~/.openclaw/logs/commands.log

# Afficher joliment avec jq
cat ~/.openclaw/logs/commands.log | jq .

# Filtrer par action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Activer** :

```bash
openclaw hooks enable command-logger
```

### boot-md

Exécute `BOOT.md` au démarrage de la gateway (après le démarrage des canaux). Les hooks internes doivent être activés pour que cela s'exécute. **Événements** : `gateway:startup` **Prérequis** : `workspace.dir` doit être configuré **Ce qu'il fait** :

1.  Lit `BOOT.md` depuis votre espace de travail
2.  Exécute les instructions via l'exécuteur de l'agent
3.  Envoie les messages sortants demandés via l'outil de message

**Activer** :

```bash
openclaw hooks enable boot-md
```

## Bonnes pratiques

### Garder les gestionnaires rapides

Les hooks s'exécutent pendant le traitement des commandes. Gardez-les légers :

```
// ✓ Bon - travail asynchrone, retourne immédiatement
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Déclenche et oublie
};

// ✗ Mauvais - bloque le traitement des commandes
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### Gérer les erreurs avec élégance

Toujours encapsuler les opérations risquées :

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Échec :", err instanceof Error ? err.message : String(err));
    // Ne pas lancer - laisser les autres gestionnaires s'exécuter
  }
};
```

### Filtrer les événements tôt

Retourner tôt si l'événement n'est pas pertinent :

```typescript
const handler: HookHandler = async (event) => {
  // Ne gérer que les commandes 'new'
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Votre logique ici
};
```

### Utiliser des clés d'événement spécifiques

Spécifiez des événements exacts dans les métadonnées lorsque possible :

```
metadata: { "openclaw": { "events": ["command:new"] } } # Spécifique
```

Plutôt que :

```
metadata: { "openclaw": { "events": ["command"] } } # Général - plus de surcharge
```

## Débogage

### Activer la journalisation des hooks

La gateway journalise le chargement des hooks au démarrage :

```bash
Registered hook: session-memory -> command:new
Registered hook: bootstrap-extra-files -> agent:bootstrap
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Vérifier la découverte

Lister tous les hooks découverts :

```bash
openclaw hooks list --verbose
```

### Vérifier l'enregistrement

Dans votre gestionnaire, journaliser quand il est appelé :

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Déclenché :", event.type, event.action);
  // Votre logique
};
```

### Vérifier l'éligibilité

Vérifier pourquoi un hook n'est pas éligible :

```bash
openclaw hooks info my-hook
```

Cherchez les prérequis manquants dans la sortie.

## Tests

### Journaux de la gateway

Surveiller les journaux de la gateway pour voir l'exécution des hooks :

```bash
# macOS
./scripts/clawlog.sh -f

# Autres plateformes
tail -f ~/.openclaw/gateway.log
```

### Tester les hooks directement

Tester vos gestionnaires de manière isolée :

```typescript
import { test } from "vitest";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = {
    type: "command",
    action: "new",
    sessionKey: "test-session",
    timestamp: new Date(),
    messages: [],
    context: { foo: "bar" },
  };

  await myHandler(event);

  // Vérifier les effets secondaires
});
```

## Architecture

### Composants principaux

-   **`src/hooks/types.ts`** : Définitions de types
-   **`src/hooks/workspace.ts`** : Balayage et chargement des répertoires
-   **`src/hooks/frontmatter.ts`** : Analyse des métadonnées de HOOK.md
-   **`src/hooks/config.ts`** : Vérification d'éligibilité
-   **`src/hooks/hooks-status.ts`** : Rapport d'état
-   **`src/hooks/loader.ts`** : Chargeur de module dynamique
-   **`src/cli/hooks-cli.ts`** : Commandes CLI
-   **`src/gateway/server-startup.ts`** : Charge les hooks au démarrage de la gateway
-   **`src/auto-reply/reply/commands-core.ts`** : Déclenche les événements de commande

### Flux de découverte

```
Démarrage de la gateway
    ↓
Balayer les répertoires (espace de travail → gérés → intégrés)
    ↓
Analyser les fichiers HOOK.md
    ↓
Vérifier l'éligibilité (bins, env, config, os)
    ↓
Charger les gestionnaires des hooks éligibles
    ↓
Enregistrer les gestionnaires pour les événements
```

### Flux d'événements

```
L'utilisateur envoie /new
    ↓
Validation de la commande
    ↓
Créer un événement de hook
    ↓
Déclencher le hook (tous les gestionnaires enregistrés)
    ↓
Le traitement de la commande continue
    ↓
Réinitialisation de la session
```

## Dépannage

### Hook non découvert

1.  Vérifier la structure du répertoire :
    
    ```bash
    ls -la ~/.openclaw/hooks/my-hook/
    # Devrait afficher : HOOK.md, handler.ts
    ```
    
2.  Vérifier le format de HOOK.md :
    
    ```bash
    cat ~/.openclaw/hooks/my-hook/HOOK.md
    # Devrait avoir un frontmatter YAML avec name et metadata
    ```
    
3.  Lister tous les hooks découverts :
    
    ```bash
    openclaw hooks list
    ```
    

### Hook non éligible

Vérifier les prérequis :

```bash
openclaw hooks info my-hook
```

Chercher les éléments manquants :

-   Binaires (vérifier PATH)
-   Variables d'environnement
-   Valeurs de configuration
-   Compatibilité OS

### Hook non exécuté

1.  Vérifier que le hook est activé :
    
    ```bash
    openclaw hooks list
    # Devrait afficher ✓ à côté des hooks activés
    ```
    
2.  Redémarrer votre processus de gateway pour recharger les hooks.
3.  Vérifier les journaux de la gateway pour les erreurs :
    
    ```bash
    ./scripts/clawlog.sh | grep hook
    ```
    

### Erreurs du gestionnaire

Vérifier les erreurs TypeScript/import :

```bash
# Tester l'import directement
node -e "import('./path/to/handler.ts').then(console.log)"
```

## Guide de migration