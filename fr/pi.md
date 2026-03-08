

  Fondamentaux

  
# Architecture d'intégration Pi

Ce document décrit comment OpenClaw s'intègre avec [pi-coding-agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) et ses packages frères (`pi-ai`, `pi-agent-core`, `pi-tui`) pour alimenter ses capacités d'agent d'IA.

## Vue d'ensemble

OpenClaw utilise le SDK pi pour intégrer un agent de codage d'IA dans son architecture de passerelle de messagerie. Au lieu de lancer pi en tant que sous-processus ou d'utiliser le mode RPC, OpenClaw importe et instancie directement `AgentSession` de pi via `createAgentSession()`. Cette approche embarquée offre :

-   Un contrôle total sur le cycle de vie de la session et la gestion des événements
-   L'injection d'outils personnalisés (messagerie, sandbox, actions spécifiques aux canaux)
-   La personnalisation du prompt système par canal/contexte
-   La persistance des sessions avec support de branchement/compaction
-   La rotation des profils d'authentification multi-comptes avec basculement
-   La commutation de modèles indépendante du fournisseur

## Dépendances des packages

```json
{
  "@mariozechner/pi-agent-core": "0.49.3",
  "@mariozechner/pi-ai": "0.49.3",
  "@mariozechner/pi-coding-agent": "0.49.3",
  "@mariozechner/pi-tui": "0.49.3"
}
```

| Package | Objectif |
| --- | --- |
| `pi-ai` | Abstractions principales des LLM : `Model`, `streamSimple`, types de messages, APIs des fournisseurs |
| `pi-agent-core` | Boucle de l'agent, exécution des outils, types `AgentMessage` |
| `pi-coding-agent` | SDK de haut niveau : `createAgentSession`, `SessionManager`, `AuthStorage`, `ModelRegistry`, outils intégrés |
| `pi-tui` | Composants d'interface terminal (utilisés dans le mode TUI local d'OpenClaw) |

## Structure des fichiers

```
src/agents/
├── pi-embedded-runner.ts          # Ré-exportations depuis pi-embedded-runner/
├── pi-embedded-runner/
│   ├── run.ts                     # Point d'entrée principal : runEmbeddedPiAgent()
│   ├── run/
│   │   ├── attempt.ts             # Logique d'une tentative unique avec configuration de session
│   │   ├── params.ts              # Type RunEmbeddedPiAgentParams
│   │   ├── payloads.ts            # Construction des réponses à partir des résultats d'exécution
│   │   ├── images.ts              # Injection d'images pour les modèles de vision
│   │   └── types.ts               # EmbeddedRunAttemptResult
│   ├── abort.ts                   # Détection des erreurs d'interruption
│   ├── cache-ttl.ts               # Suivi de la durée de vie du cache pour l'élagage du contexte
│   ├── compact.ts                 # Logique de compaction manuelle/automatique
│   ├── extensions.ts              # Chargement des extensions pi pour les exécutions embarquées
│   ├── extra-params.ts            # Paramètres de flux spécifiques aux fournisseurs
│   ├── google.ts                  # Corrections de l'ordre des tours pour Google/Gemini
│   ├── history.ts                 # Limitation de l'historique (DM vs groupe)
│   ├── lanes.ts                   # Voies de commandes session/globales
│   ├── logger.ts                  # Journal du sous-système
│   ├── model.ts                   # Résolution du modèle via ModelRegistry
│   ├── runs.ts                    # Suivi des exécutions actives, interruption, file d'attente
│   ├── sandbox-info.ts            # Informations sur le sandbox pour le prompt système
│   ├── session-manager-cache.ts   # Mise en cache des instances SessionManager
│   ├── session-manager-init.ts    # Initialisation des fichiers de session
│   ├── system-prompt.ts           # Constructeur de prompt système
│   ├── tool-split.ts              # Séparation des outils en builtIn vs custom
│   ├── types.ts                   # EmbeddedPiAgentMeta, EmbeddedPiRunResult
│   └── utils.ts                   # Mapping ThinkLevel, description d'erreur
├── pi-embedded-subscribe.ts       # Abonnement aux événements de session / dispatch
├── pi-embedded-subscribe.types.ts # SubscribeEmbeddedPiSessionParams
├── pi-embedded-subscribe.handlers.ts # Fabrique de gestionnaires d'événements
├── pi-embedded-subscribe.handlers.lifecycle.ts
├── pi-embedded-subscribe.handlers.types.ts
├── pi-embedded-block-chunker.ts   # Découpage en blocs des réponses en flux
├── pi-embedded-messaging.ts       # Suivi des messages d'outils envoyés
├── pi-embedded-helpers.ts         # Classification des erreurs, validation des tours
├── pi-embedded-helpers/           # Modules d'aide
├── pi-embedded-utils.ts           # Utilitaires de formatage
├── pi-tools.ts                    # createOpenClawCodingTools()
├── pi-tools.abort.ts              # Encapsulation AbortSignal pour les outils
├── pi-tools.policy.ts             # Politique de liste autorisée/interdite des outils
├── pi-tools.read.ts               # Personnalisations de l'outil read
├── pi-tools.schema.ts             # Normalisation des schémas d'outils
├── pi-tools.types.ts              # Alias de type AnyAgentTool
├── pi-tool-definition-adapter.ts  # Adaptateur AgentTool -> ToolDefinition
├── pi-settings.ts                 # Remplacements des paramètres
├── pi-extensions/                 # Extensions pi personnalisées
│   ├── compaction-safeguard.ts    # Extension de sauvegarde de la compaction
│   ├── compaction-safeguard-runtime.ts
│   ├── context-pruning.ts         # Extension d'élagage du contexte basée sur Cache-TTL
│   └── context-pruning/
├── model-auth.ts                  # Résolution du profil d'authentification
├── auth-profiles.ts               # Stockage des profils, refroidissement, basculement
├── model-selection.ts             # Résolution du modèle par défaut
├── models-config.ts               # Génération de models.json
├── model-catalog.ts               # Cache du catalogue de modèles
├── context-window-guard.ts        # Validation de la fenêtre de contexte
├── failover-error.ts              # Classe FailoverError
├── defaults.ts                    # DEFAULT_PROVIDER, DEFAULT_MODEL
├── system-prompt.ts               # buildAgentSystemPrompt()
├── system-prompt-params.ts        # Résolution des paramètres du prompt système
├── system-prompt-report.ts        # Génération de rapport de débogage
├── tool-summaries.ts              # Résumés des descriptions d'outils
├── tool-policy.ts                 # Résolution de la politique des outils
├── transcript-policy.ts           # Politique de validation des transcriptions
├── skills.ts                      # Instantané des compétences / construction de prompt
├── skills/                        # Sous-système des compétences
├── sandbox.ts                     # Résolution du contexte sandbox
├── sandbox/                       # Sous-système sandbox
├── channel-tools.ts               # Injection d'outils spécifiques au canal
├── openclaw-tools.ts              # Outils spécifiques à OpenClaw
├── bash-tools.ts                  # Outils exec/process
├── apply-patch.ts                 # Outil apply_patch (OpenAI)
├── tools/                         # Implémentations individuelles d'outils
│   ├── browser-tool.ts
│   ├── canvas-tool.ts
│   ├── cron-tool.ts
│   ├── discord-actions*.ts
│   ├── gateway-tool.ts
│   ├── image-tool.ts
│   ├── message-tool.ts
│   ├── nodes-tool.ts
│   ├── session*.ts
│   ├── slack-actions.ts
│   ├── telegram-actions.ts
│   ├── web-*.ts
│   └── whatsapp-actions.ts
└── ...
```

## Flux d'intégration principal

### 1\. Exécution d'un agent embarqué

Le point d'entrée principal est `runEmbeddedPiAgent()` dans `pi-embedded-runner/run.ts` :

```typescript
import { runEmbeddedPiAgent } from "./agents/pi-embedded-runner.js";

const result = await runEmbeddedPiAgent({
  sessionId: "user-123",
  sessionKey: "main:whatsapp:+1234567890",
  sessionFile: "/path/to/session.jsonl",
  workspaceDir: "/path/to/workspace",
  config: openclawConfig,
  prompt: "Hello, how are you?",
  provider: "anthropic",
  model: "claude-sonnet-4-20250514",
  timeoutMs: 120_000,
  runId: "run-abc",
  onBlockReply: async (payload) => {
    await sendToChannel(payload.text, payload.mediaUrls);
  },
});
```

### 2\. Création de session

À l'intérieur de `runEmbeddedAttempt()` (appelé par `runEmbeddedPiAgent()`), le SDK pi est utilisé :

```typescript
import {
  createAgentSession,
  DefaultResourceLoader,
  SessionManager,
  SettingsManager,
} from "@mariozechner/pi-coding-agent";

const resourceLoader = new DefaultResourceLoader({
  cwd: resolvedWorkspace,
  agentDir,
  settingsManager,
  additionalExtensionPaths,
});
await resourceLoader.reload();

const { session } = await createAgentSession({
  cwd: resolvedWorkspace,
  agentDir,
  authStorage: params.authStorage,
  modelRegistry: params.modelRegistry,
  model: params.model,
  thinkingLevel: mapThinkingLevel(params.thinkLevel),
  tools: builtInTools,
  customTools: allCustomTools,
  sessionManager,
  settingsManager,
  resourceLoader,
});

applySystemPromptOverrideToSession(session, systemPromptOverride);
```

### 3\. Abonnement aux événements

`subscribeEmbeddedPiSession()` s'abonne aux événements de `AgentSession` de pi :

```typescript
const subscription = subscribeEmbeddedPiSession({
  session: activeSession,
  runId: params.runId,
  verboseLevel: params.verboseLevel,
  reasoningMode: params.reasoningLevel,
  toolResultFormat: params.toolResultFormat,
  onToolResult: params.onToolResult,
  onReasoningStream: params.onReasoningStream,
  onBlockReply: params.onBlockReply,
  onPartialReply: params.onPartialReply,
  onAgentEvent: params.onAgentEvent,
});
```

Les événements gérés incluent :

-   `message_start` / `message_end` / `message_update` (texte/réflexion en flux)
-   `tool_execution_start` / `tool_execution_update` / `tool_execution_end`
-   `turn_start` / `turn_end`
-   `agent_start` / `agent_end`
-   `auto_compaction_start` / `auto_compaction_end`

### 4\. Prompting

Après la configuration, la session est sollicitée :

```typescript
await session.prompt(effectivePrompt, { images: imageResult.images });
```

Le SDK gère la boucle complète de l'agent : envoi au LLM, exécution des appels d'outils, diffusion des réponses. L'injection d'images est locale au prompt : OpenClaw charge les références d'images du prompt actuel et les transmet via `images` uniquement pour ce tour. Il ne re-scanne pas les tours d'historique plus anciens pour réinjecter les images.

## Architecture des outils

### Pipeline des outils

1.  **Outils de base** : `codingTools` de pi (read, bash, edit, write)
2.  **Remplacements personnalisés** : OpenClaw remplace bash par `exec`/`process`, personnalise read/edit/write pour le sandbox
3.  **Outils OpenClaw** : messagerie, navigateur, canvas, sessions, cron, passerelle, etc.
4.  **Outils de canal** : Outils d'action spécifiques à Discord/Telegram/Slack/WhatsApp
5.  **Filtrage par politique** : Outils filtrés par profil, fournisseur, agent, groupe, politiques sandbox
6.  **Normalisation des schémas** : Schémas nettoyés pour les particularités de Gemini/OpenAI
7.  **Encapsulation AbortSignal** : Outils encapsulés pour respecter les signaux d'interruption

### Adaptateur de définition d'outil

`AgentTool` de pi-agent-core a une signature `execute` différente de `ToolDefinition` de pi-coding-agent. L'adaptateur dans `pi-tool-definition-adapter.ts` fait le pont :

```bash
export function toToolDefinitions(tools: AnyAgentTool[]): ToolDefinition[] {
  return tools.map((tool) => ({
    name: tool.name,
    label: tool.label ?? name,
    description: tool.description ?? "",
    parameters: tool.parameters,
    execute: async (toolCallId, params, onUpdate, _ctx, signal) => {
      // La signature de pi-coding-agent diffère de celle de pi-agent-core
      return await tool.execute(toolCallId, params, signal, onUpdate);
    },
  }));
}
```

### Stratégie de séparation des outils

`splitSdkTools()` passe tous les outils via `customTools` :

```bash
export function splitSdkTools(options: { tools: AnyAgentTool[]; sandboxEnabled: boolean }) {
  return {
    builtInTools: [], // Vide. Nous remplaçons tout
    customTools: toToolDefinitions(options.tools),
  };
}
```

Cela garantit que le filtrage par politique d'OpenClaw, l'intégration sandbox et l'ensemble d'outils étendu restent cohérents entre les fournisseurs.

## Construction du prompt système

Le prompt système est construit dans `buildAgentSystemPrompt()` (`system-prompt.ts`). Il assemble un prompt complet avec des sections incluant Outillage, Style d'appel d'outil, Garde-fous de sécurité, Référence CLI OpenClaw, Compétences, Docs, Espace de travail, Sandbox, Messagerie, Balises de réponse, Voix, Réponses silencieuses, Heartbeats, Métadonnées d'exécution, plus Mémoire et Réactions lorsqu'activées, et des fichiers de contexte et du contenu de prompt système supplémentaires optionnels. Les sections sont réduites pour le mode prompt minimal utilisé par les sous-agents. Le prompt est appliqué après la création de la session via `applySystemPromptOverrideToSession()` :

```typescript
const systemPromptOverride = createSystemPromptOverride(appendPrompt);
applySystemPromptOverrideToSession(session, systemPromptOverride);
```

## Gestion des sessions

### Fichiers de session

Les sessions sont des fichiers JSONL avec une structure arborescente (liaison id/parentId). Le `SessionManager` de pi gère la persistance :

```typescript
const sessionManager = SessionManager.open(params.sessionFile);
```

OpenClaw encapsule cela avec `guardSessionManager()` pour la sécurité des résultats d'outils.

### Mise en cache des sessions

`session-manager-cache.ts` met en cache les instances SessionManager pour éviter l'analyse répétée des fichiers :

```typescript
await prewarmSessionFile(params.sessionFile);
sessionManager = SessionManager.open(params.sessionFile);
trackSessionManagerAccess(params.sessionFile);
```

### Limitation de l'historique

`limitHistoryTurns()` réduit l'historique de conversation en fonction du type de canal (DM vs groupe).

### Compaction

La compaction automatique se déclenche en cas de dépassement de contexte. `compactEmbeddedPiSessionDirect()` gère la compaction manuelle :

```typescript
const compactResult = await compactEmbeddedPiSessionDirect({
  sessionId, sessionFile, provider, model, ...
});
```

## Authentification et résolution des modèles

### Profils d'authentification

OpenClaw maintient un magasin de profils d'authentification avec plusieurs clés API par fournisseur :

```typescript
const authStore = ensureAuthProfileStore(agentDir, { allowKeychainPrompt: false });
const profileOrder = resolveAuthProfileOrder({ cfg, store: authStore, provider, preferredProfile });
```

Les profils tournent en cas d'échec avec suivi du refroidissement :

```typescript
await markAuthProfileFailure({ store, profileId, reason, cfg, agentDir });
const rotated = await advanceAuthProfile();
```

### Résolution du modèle

```typescript
import { resolveModel } from "./pi-embedded-runner/model.js";

const { model, error, authStorage, modelRegistry } = resolveModel(
  provider,
  modelId,
  agentDir,
  config,
);

// Utilise ModelRegistry et AuthStorage de pi
authStorage.setRuntimeApiKey(model.provider, apiKeyInfo.apiKey);
```

### Basculement

`FailoverError` déclenche la bascule de modèle lorsqu'elle est configurée :

```
if (fallbackConfigured && isFailoverErrorMessage(errorText)) {
  throw new FailoverError(errorText, {
    reason: promptFailoverReason ?? "unknown",
    provider,
    model: modelId,
    profileId,
    status: resolveFailoverStatus(promptFailoverReason),
  });
}
```

## Extensions Pi

OpenClaw charge des extensions pi personnalisées pour des comportements spécialisés :

### Sauvegarde de la compaction

`src/agents/pi-extensions/compaction-safeguard.ts` ajoute des garde-fous à la compaction, incluant un budget de jetons adaptatif ainsi que des résumés des échecs d'outils et des opérations sur fichiers :

```
if (resolveCompactionMode(params.cfg) === "safeguard") {
  setCompactionSafeguardRuntime(params.sessionManager, { maxHistoryShare });
  paths.push(resolvePiExtensionPath("compaction-safeguard"));
}
```

### Élagage du contexte

`src/agents/pi-extensions/context-pruning.ts` implémente l'élagage du contexte basé sur la durée de vie du cache (cache-TTL) :

```
if (cfg?.agents?.defaults?.contextPruning?.mode === "cache-ttl") {
  setContextPruningRuntime(params.sessionManager, {
    settings,
    contextWindowTokens,
    isToolPrunable,
    lastCacheTouchAt,
  });
  paths.push(resolvePiExtensionPath("context-pruning"));
}
```

## Diffusion en flux et réponses par blocs

### Découpage en blocs

`EmbeddedBlockChunker` gère la diffusion de texte en blocs de réponse discrets :

```typescript
const blockChunker = blockChunking ? new EmbeddedBlockChunker(blockChunking) : null;
```

### Suppression des balises de réflexion/final

La sortie en flux est traitée pour supprimer les blocs ``/`` et extraire le contenu `` :

```typescript
const stripBlockTags = (text: string, state: { thinking: boolean; final: boolean }) => {
  // Supprime le contenu <think>...</think>
  // Si enforceFinalTag, ne retourne que le contenu <final>...</final>
};
```

### Directives de réponse

Les directives de réponse comme `[[media:url]]`, `[[voice]]`, `[[reply:id]]` sont analysées et extraites :

```typescript
const { text: cleanedText, mediaUrls, audioAsVoice, replyToId } = consumeReplyDirectives(chunk);
```

## Gestion des erreurs

### Classification des erreurs

`pi-embedded-helpers.ts` classe les erreurs pour une gestion appropriée :

```
isContextOverflowError(errorText)     // Contexte trop volumineux
isCompactionFailureError(errorText)   // Échec de la compaction
isAuthAssistantError(lastAssistant)   // Échec d'authentification
isRateLimitAssistantError(...)        // Limite de débit atteinte
isFailoverAssistantError(...)         // Devrait basculer
classifyFailoverReason(errorText)     // "auth" | "rate_limit" | "quota" | "timeout" | ...
```

### Repli du niveau de réflexion

Si un niveau de réflexion n'est pas supporté, il revient à un niveau inférieur :

```typescript
const fallbackThinking = pickFallbackThinkingLevel({
  message: errorText,
  attempted: attemptedThinking,
});
if (fallbackThinking) {
  thinkLevel = fallbackThinking;
  continue;
}
```

## Intégration Sandbox

Lorsque le mode sandbox est activé, les outils et chemins sont contraints :

```typescript
const sandbox = await resolveSandboxContext({
  config: params.config,
  sessionKey: sandboxSessionKey,
  workspaceDir: resolvedWorkspace,
});

if (sandboxRoot) {
  // Utilise les outils read/edit/write sandboxés
  // Exec s'exécute dans un conteneur
  // Le navigateur utilise l'URL de pont
}
```

## Gestion spécifique aux fournisseurs

### Anthropic

-   Nettoyage des chaînes magiques de refus
-   Validation des tours pour les rôles consécutifs
-   Compatibilité avec le paramètre Claude Code

### Google/Gemini

-   Corrections de l'ordre des tours (`applyGoogleTurnOrderingFix`)
-   Assainissement des schémas d'outils (`sanitizeToolsForGoogle`)
-   Assainissement de l'historique des sessions (`sanitizeSessionHistory`)

### OpenAI

-   Outil `apply_patch` pour les modèles Codex
-   Gestion de la rétrogradation du niveau de réflexion

## Intégration TUI

OpenClaw possède également un mode TUI local qui utilise directement les composants pi-tui :

```bash
// src/tui/tui.ts
import { ... } from "@mariozechner/pi-tui";
```

Cela fournit l'expérience terminal interactive similaire au mode natif de pi.

## Différences clés avec Pi CLI

| Aspect | Pi CLI | OpenClaw Embarqué |
| --- | --- | --- |
| Invocation | Commande `pi` / RPC | SDK via `createAgentSession()` |
| Outils | Outils de codage par défaut | Suite d'outils OpenClaw personnalisée |
| Prompt système | AGENTS.md + prompts | Dynamique par canal/contexte |
| Stockage des sessions | `~/.pi/agent/sessions/` | `~/.openclaw/agents//sessions/` (ou `$OPENCLAW_STATE_DIR/agents//sessions/`) |
| Authentification | Identifiant unique | Multi-profils avec rotation |
| Extensions | Chargées depuis le disque | Programmatiques + chemins disque |
| Gestion des événements | Rendu TUI | Basée sur des rappels (onBlockReply, etc.) |

## Considérations futures

Domaines potentiels de retravail :

1.  **Alignement des signatures d'outils** : Adaptation actuelle entre les signatures de pi-agent-core et pi-coding-agent
2.  **Encapsulation du gestionnaire de session** : `guardSessionManager` ajoute de la sécurité mais augmente la complexité
3.  **Chargement des extensions** : Pourrait utiliser `ResourceLoader` de pi plus directement
4.  **Complexité des gestionnaires de flux** : `subscribeEmbeddedPiSession` est devenu volumineux
5.  **Particularités des fournisseurs** : De nombreux chemins de code spécifiques aux fournisseurs que pi pourrait potentiellement gérer

## Tests

La couverture d'intégration Pi s'étend sur ces suites :

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-auth-json.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-embedded-helpers*.test.ts`
-   `src/agents/pi-embedded-runner*.test.ts`
-   `src/agents/pi-embedded-runner/**/*.test.ts`
-   `src/agents/pi-embedded-subscribe*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-tool-definition-adapter*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-extensions/**/*.test.ts`

Live/opt-in :

-   `src/agents/pi-embedded-runner-extraparams.live.test.ts` (activer `OPENCLAW_LIVE_TEST=1`)

Pour les commandes d'exécution actuelles, voir [Workflow de développement Pi](./pi-dev.md).

[Architecture de la passerelle](./concepts/architecture.md)