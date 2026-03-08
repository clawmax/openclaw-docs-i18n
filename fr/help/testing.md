title: "Guide de test OpenClaw : suites de tests unitaires, E2E et live"
description: "Apprenez à tester OpenClaw avec les suites de tests unitaires, d'intégration, E2E et live. Exécutez les commandes pour le débogage, la couverture et la validation réelle des fournisseurs et modèles."
keywords: ["tests openclaw", "suites vitest", "tests e2e", "tests live", "test de fumée gateway", "tests node android", "intégration de modèles", "couverture de test"]
---

  Environnement et débogage

  
# Tests

OpenClaw a trois suites Vitest (unitaire/intégration, e2e, live) et un petit ensemble de lanceurs Docker. Ce document est un guide "comment nous testons" :

-   Ce que couvre chaque suite (et ce qu'elle ne couvre *délibérément pas*)
-   Quelles commandes exécuter pour les workflows courants (local, pré-push, débogage)
-   Comment les tests live découvrent les identifiants et sélectionnent les modèles/fournisseurs
-   Comment ajouter des régressions pour les problèmes réels de modèles/fournisseurs

## Démarrage rapide

La plupart du temps :

-   Porte complète (attendue avant un push) : `pnpm build && pnpm check && pnpm test`

Lorsque vous touchez aux tests ou voulez une confiance supplémentaire :

-   Porte de couverture : `pnpm test:coverage`
-   Suite E2E : `pnpm test:e2e`

Lors du débogage de fournisseurs/modèles réels (nécessite de vrais identifiants) :

-   Suite live (modèles + sondes d'outil/image gateway) : `pnpm test:live`

Astuce : lorsque vous n'avez besoin que d'un seul cas en échec, privilégiez le rétrécissement des tests live via les variables d'environnement de liste autorisée décrites ci-dessous.

## Suites de tests (ce qui s'exécute où)

Considérez les suites comme une "réalité croissante" (et une instabilité/coût croissants) :

### Unitaire / intégration (par défaut)

-   Commande : `pnpm test`
-   Configuration : `scripts/test-parallel.mjs` (exécute `vitest.unit.config.ts`, `vitest.extensions.config.ts`, `vitest.gateway.config.ts`)
-   Fichiers : `src/**/*.test.ts`, `extensions/**/*.test.ts`
-   Portée :
    -   Tests unitaires purs
    -   Tests d'intégration en processus (authentification gateway, routage, outils, analyse, configuration)
    -   Régressions déterministes pour les bogues connus
-   Attentes :
    -   S'exécute en CI
    -   Aucune clé réelle requise
    -   Doit être rapide et stable
-   Note sur le pool :
    -   OpenClaw utilise les `vmForks` de Vitest sur Node 22/23 pour des shards unitaires plus rapides.
    -   Sur Node 24+, OpenClaw revient automatiquement aux `forks` réguliers pour éviter les erreurs de liaison VM Node (`ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`).
    -   Remplacez manuellement avec `OPENCLAW_TEST_VM_FORKS=0` (force `forks`) ou `OPENCLAW_TEST_VM_FORKS=1` (force `vmForks`).

### E2E (test de fumée gateway)

-   Commande : `pnpm test:e2e`
-   Configuration : `vitest.e2e.config.ts`
-   Fichiers : `src/**/*.e2e.test.ts`
-   Valeurs par défaut d'exécution :
    -   Utilise les `vmForks` de Vitest pour un démarrage de fichier plus rapide.
    -   Utilise des travailleurs adaptatifs (CI : 2-4, local : 4-8).
    -   S'exécute en mode silencieux par défaut pour réduire la surcharge d'E/S console.
-   Remplacements utiles :
    -   `OPENCLAW_E2E_WORKERS=` pour forcer le nombre de travailleurs (limité à 16).
    -   `OPENCLAW_E2E_VERBOSE=1` pour réactiver la sortie console verbeuse.
-   Portée :
    -   Comportement de bout en bout de la gateway multi-instances
    -   Surfaces WebSocket/HTTP, appairage de nœuds et réseau plus lourd
-   Attentes :
    -   S'exécute en CI (lorsqu'activé dans le pipeline)
    -   Aucune clé réelle requise
    -   Plus de pièces mobiles que les tests unitaires (peut être plus lent)

### Live (fournisseurs réels + modèles réels)

-   Commande : `pnpm test:live`
-   Configuration : `vitest.live.config.ts`
-   Fichiers : `src/**/*.live.test.ts`
-   Par défaut : **activé** par `pnpm test:live` (définit `OPENCLAW_LIVE_TEST=1`)
-   Portée :
    -   "Est-ce que ce fournisseur/modèle fonctionne *réellement* *aujourd'hui* avec de vrais identifiants ?"
    -   Détecter les changements de format de fournisseur, les particularités d'appel d'outil, les problèmes d'authentification et le comportement des limites de taux
-   Attentes :
    -   Non stable en CI par conception (réseaux réels, politiques réelles des fournisseurs, quotas, pannes)
    -   Coûte de l'argent / utilise les limites de taux
    -   Préférez exécuter des sous-ensembles restreints au lieu de "tout"
    -   Les exécutions live sourceront `~/.profile` pour récupérer les clés API manquantes
-   Rotation des clés API (spécifique au fournisseur) : définissez `*_API_KEYS` avec un format virgule/point-virgule ou `*_API_KEY_1`, `*_API_KEY_2` (par exemple `OPENAI_API_KEYS`, `ANTHROPIC_API_KEYS`, `GEMINI_API_KEYS`) ou remplacement par live via `OPENCLAW_LIVE_*_KEY` ; les tests réessaient sur les réponses de limite de taux.

## Quelle suite dois-je exécuter ?

Utilisez cette table de décision :

-   Modification de la logique/des tests : exécutez `pnpm test` (et `pnpm test:coverage` si vous avez beaucoup changé)
-   Toucher au réseau gateway / protocole WS / appairage : ajoutez `pnpm test:e2e`
-   Débogage "mon bot est en panne" / échecs spécifiques au fournisseur / appel d'outil : exécutez un `pnpm test:live` restreint

## Live : balayage des capacités du nœud Android

-   Test : `src/gateway/android-node.capabilities.live.test.ts`
-   Script : `pnpm android:test:integration`
-   Objectif : invoquer **chaque commande actuellement annoncée** par un nœud Android connecté et vérifier le comportement du contrat de commande.
-   Portée :
    -   Configuration préconditionnée/manuelle (la suite n'installe/exécute/apparie pas l'application).
    -   Validation `node.invoke` gateway commande par commande pour le nœud Android sélectionné.
-   Pré-configuration requise :
    -   Application Android déjà connectée + appairée à la gateway.
    -   Application maintenue au premier plan.
    -   Permissions/consentement de capture accordé pour les capacités que vous attendez de réussir.
-   Remplacements de cible optionnels :
    -   `OPENCLAW_ANDROID_NODE_ID` ou `OPENCLAW_ANDROID_NODE_NAME`.
    -   `OPENCLAW_ANDROID_GATEWAY_URL` / `OPENCLAW_ANDROID_GATEWAY_TOKEN` / `OPENCLAW_ANDROID_GATEWAY_PASSWORD`.
-   Détails complets de la configuration Android : [Application Android](../platforms/android.md)

## Live : test de fumée des modèles (clés de profil)

Les tests live sont divisés en deux couches pour isoler les échecs :

-   "Modèle direct" nous indique si le fournisseur/modèle peut répondre du tout avec la clé donnée.
-   "Test de fumée gateway" nous indique si le pipeline complet gateway+agent fonctionne pour ce modèle (sessions, historique, outils, politique de sandbox, etc.).

### Couche 1 : Complétion directe de modèle (sans gateway)

-   Test : `src/agents/models.profiles.live.test.ts`
-   Objectif :
    -   Énumérer les modèles découverts
    -   Utiliser `getApiKeyForModel` pour sélectionner les modèles pour lesquels vous avez des identifiants
    -   Exécuter une petite complétion par modèle (et des régressions ciblées si nécessaire)
-   Comment activer :
    -   `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
-   Définissez `OPENCLAW_LIVE_MODELS=modern` (ou `all`, alias pour modern) pour réellement exécuter cette suite ; sinon elle est ignorée pour garder `pnpm test:live` concentré sur le test de fumée gateway
-   Comment sélectionner les modèles :
    -   `OPENCLAW_LIVE_MODELS=modern` pour exécuter la liste autorisée moderne (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.5, Grok 4)
    -   `OPENCLAW_LIVE_MODELS=all` est un alias pour la liste autorisée moderne
    -   ou `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,..."` (liste autorisée par virgules)
-   Comment sélectionner les fournisseurs :
    -   `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (liste autorisée par virgules)
-   D'où viennent les clés :
    -   Par défaut : magasin de profil et replis d'environnement
    -   Définissez `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` pour imposer **uniquement le magasin de profil**
-   Pourquoi cela existe :
    -   Sépare "l'API du fournisseur est cassée / la clé est invalide" de "le pipeline d'agent gateway est cassé"
    -   Contient de petites régressions isolées (exemple : rejeu de raisonnement OpenAI Responses/Codex Responses + flux d'appel d'outil)

### Couche 2 : Gateway + test de fumée de l'agent de développement (ce que fait réellement "@openclaw")

-   Test : `src/gateway/gateway-models.profiles.live.test.ts`
-   Objectif :
    -   Démarrer une gateway en processus
    -   Créer/modifier une session `agent:dev:*` (remplacement de modèle par exécution)
    -   Itérer sur les modèles avec clés et vérifier :
        -   Une réponse "significative" (sans outils)
        -   Une invocation d'outil réel fonctionne (sonde de lecture)
        -   Sondes d'outil supplémentaires optionnelles (sonde exec+read)
        -   Les chemins de régression OpenAI (appel d'outil uniquement → suivi) continuent de fonctionner
-   Détails des sondes (pour expliquer rapidement les échecs) :
    -   Sonde `read` : le test écrit un fichier nonce dans l'espace de travail et demande à l'agent de le `read` et de renvoyer le nonce.
    -   Sonde `exec+read` : le test demande à l'agent d'`exec`-écrire un nonce dans un fichier temporaire, puis de le `read`.
    -   Sonde image : le test attache un PNG généré (chat + code randomisé) et attend que le modèle renvoie `cat `.
    -   Référence d'implémentation : `src/gateway/gateway-models.profiles.live.test.ts` et `src/gateway/live-image-probe.ts`.
-   Comment activer :
    -   `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
-   Comment sélectionner les modèles :
    -   Par défaut : liste autorisée moderne (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.5, Grok 4)
    -   `OPENCLAW_LIVE_GATEWAY_MODELS=all` est un alias pour la liste autorisée moderne
    -   Ou définissez `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (ou liste par virgules) pour restreindre
-   Comment sélectionner les fournisseurs (éviter "OpenRouter tout") :
    -   `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (liste autorisée par virgules)
-   Les sondes d'outil et d'image sont toujours activées dans ce test live :
    -   Sonde `read` + sonde `exec+read` (stress d'outil)
    -   La sonde image s'exécute lorsque le modèle annonce la prise en charge de l'entrée d'image
    -   Flux (haut niveau) :
        -   Le test génère un petit PNG avec "CAT" + un code aléatoire (`src/gateway/live-image-probe.ts`)
        -   L'envoie via `agent` `attachments: [{ mimeType: "image/png", content: "" }]`
        -   La gateway analyse les pièces jointes en `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
        -   L'agent intégré transmet un message utilisateur multimodal au modèle
        -   Assertion : la réponse contient `cat` + le code (tolérance OCR : erreurs mineures autorisées)

Astuce : pour voir ce que vous pouvez tester sur votre machine (et les identifiants exacts `provider/model`), exécutez :

```bash
openclaw models list
openclaw models list --json
```

## Live : test de fumée du jeton de configuration Anthropic

-   Test : `src/agents/anthropic.setup-token.live.test.ts`
-   Objectif : vérifier que le jeton de configuration CLI Claude Code (ou un profil de jeton de configuration collé) peut compléter une invite Anthropic.
-   Activer :
    -   `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
    -   `OPENCLAW_LIVE_SETUP_TOKEN=1`
-   Sources du jeton (choisissez une) :
    -   Profil : `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
    -   Jeton brut : `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
-   Remplacement de modèle (optionnel) :
    -   `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-6`

Exemple de configuration :

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

## Live : test de fumée du backend CLI (Claude Code CLI ou autres CLI locaux)

-   Test : `src/gateway/gateway-cli-backend.live.test.ts`
-   Objectif : valider le pipeline Gateway + agent en utilisant un backend CLI local, sans toucher à votre configuration par défaut.
-   Activer :
    -   `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
    -   `OPENCLAW_LIVE_CLI_BACKEND=1`
-   Valeurs par défaut :
    -   Modèle : `claude-cli/claude-sonnet-4-6`
    -   Commande : `claude`
    -   Arguments : `["-p","--output-format","json","--permission-mode","bypassPermissions"]`
-   Remplacements (optionnels) :
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-6"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.4"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` pour envoyer une pièce jointe image réelle (les chemins sont injectés dans l'invite).
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` pour passer les chemins de fichiers image comme arguments CLI au lieu d'une injection dans l'invite.
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (ou `"list"`) pour contrôler comment les arguments image sont passés lorsque `IMAGE_ARG` est défini.
    -   `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` pour envoyer un deuxième tour et valider le flux de reprise.
    -   `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` pour garder la configuration MCP de Claude Code CLI activée (par défaut désactive la configuration MCP avec un fichier vide temporaire).

Exemple :

```
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-6" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

### Recettes live recommandées

Les listes autorisées explicites et restreintes sont les plus rapides et les moins instables :

-   Modèle unique, direct (sans gateway) :
    -   `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`
-   Modèle unique, test de fumée gateway :
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   Appel d'outil sur plusieurs fournisseurs :
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   Focus Google (clé API Gemini + Antigravity) :
    -   Gemini (clé API) : `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
    -   Antigravity (OAuth) : `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

Notes :

-   `google/...` utilise l'API Gemini (clé API).
-   `google-antigravity/...` utilise le pont OAuth Antigravity (point de terminaison d'agent de type Cloud Code Assist).
-   `google-gemini-cli/...` utilise le CLI Gemini local sur votre machine (authentification distincte + particularités d'outillage).
-   API Gemini vs CLI Gemini :
    -   API : OpenClaw appelle l'API Gemini hébergée de Google via HTTP (authentification par clé API / profil) ; c'est ce que la plupart des utilisateurs entendent par "Gemini".
    -   CLI : OpenClaw exécute un binaire `gemini` local ; il a sa propre authentification et peut se comporter différemment (support du streaming/des outils/décalage de version).

## Live : matrice des modèles (ce que nous couvrons)

Il n'y a pas de "liste de modèles CI" fixe (live est optionnel), mais voici les modèles **recommandés** à couvrir régulièrement sur une machine de développement avec des clés.

### Ensemble de test de fumée moderne (appel d'outil + image)

C'est l'exécution "modèles courants" que nous nous attendons à voir fonctionner :

-   OpenAI (non-Codex) : `openai/gpt-5.2` (optionnel : `openai/gpt-5.1`)
-   OpenAI Codex : `openai-codex/gpt-5.4`
-   Anthropic : `anthropic/claude-opus-4-6` (ou `anthropic/claude-sonnet-4-5`)
-   Google (API Gemini) : `google/gemini-3-pro-preview` et `google/gemini-3-flash-preview` (évitez les anciens modèles Gemini 2.x)
-   Google (Antigravity) : `google-antigravity/claude-opus-4-6-thinking` et `google-antigravity/gemini-3-flash`
-   Z.AI (GLM) : `zai/glm-4.7`
-   MiniMax : `minimax/minimax-m2.5`

Exécutez le test de fumée gateway avec outils + image : `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.4,anthropic/claude-opus-4-6,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

### Base de référence : appel d'outil (Read + optionnel Exec)

Choisissez au moins un par famille de fournisseurs :

-   OpenAI : `openai/gpt-5.2` (ou `openai/gpt-5-mini`)
-   Anthropic : `anthropic/claude-opus-4-6` (ou `anthropic/claude-sonnet-4-5`)
-   Google : `google/gemini-3-flash-preview` (ou `google/gemini-3-pro-preview`)
-   Z.AI (GLM) : `zai/glm-4.7`
-   MiniMax : `minimax/minimax-m2.5`

Couverture supplémentaire optionnelle (bon à avoir) :

-   xAI : `xai/grok-4` (ou le dernier disponible)
-   Mistral : `mistral/`… (choisissez un modèle capable d'"outils" que vous avez activé)
-   Cerebras : `cerebras/`… (si vous y avez accès)
-   LM Studio : `lmstudio/`… (local ; l'appel d'outil dépend du mode API)

### Vision : envoi d'image (pièce jointe → message multimodal)

Incluez au moins un modèle capable de traiter les images dans `OPENCLAW_LIVE_GATEWAY_MODELS` (variantes compatibles vision de Claude/Gemini/OpenAI, etc.) pour exercer la sonde image.

### Agrégateurs / gateways alternatives

Si vous avez des clés activées, nous prenons également en charge les tests via :

-   OpenRouter : `openrouter/...` (des centaines de modèles ; utilisez `openclaw models scan` pour trouver des candidats capables d'outils+images)
-   OpenCode Zen : `opencode/...` (authentification via `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

Plus de fournisseurs que vous pouvez inclure dans la matrice live (si vous avez des identifiants/config) :

-   Intégrés : `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
-   Via `models.providers` (points de terminaison personnalisés) : `minimax` (cloud/API), plus tout proxy compatible OpenAI/Anthropique (LM Studio, vLLM, LiteLLM, etc.)

Astuce : n'essayez pas de coder en dur "tous les modèles" dans la documentation. La liste faisant autorité est ce que `discoverModels(...)` renvoie sur votre machine + toutes les clés disponibles.

## Identifiants (ne jamais les commettre)

Les tests live découvrent les identifiants de la même manière que le CLI. Implications pratiques :

-   Si le CLI fonctionne, les tests live devraient trouver les mêmes clés.
-   Si un test live indique "pas d'identifiants", déboguez de la même manière que vous débogueriez `openclaw models list` / la sélection de modèle.
-   Magasin de profil : `~/.openclaw/credentials/` (préféré ; ce que signifie "clés de profil" dans les tests)
-   Configuration : `~/.openclaw/openclaw.json` (ou `OPENCLAW_CONFIG_PATH`)

Si vous voulez vous fier aux clés d'environnement (par exemple exportées dans votre `~/.profile`), exécutez les tests locaux après `source ~/.profile`, ou utilisez les lanceurs Docker ci-dessous (ils peuvent monter `~/.profile` dans le conteneur).

## Deepgram live (transcription audio)

-   Test : `src/media-understanding/providers/deepgram/audio.live.test.ts`
-   Activer : `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

## BytePlus live du plan de codage

-   Test : `src/agents/byteplus.live.test.ts`
-   Activer : `BYTEPLUS_API_KEY=... BYTEPLUS_LIVE_TEST=1 pnpm test:live src/agents/byteplus.live.test.ts`
-   Remplacement de modèle optionnel : `BYTEPLUS_CODING_MODEL=ark-code-latest`

## Lanceurs Docker (vérifications optionnelles "fonctionne sous Linux")

Ceux-ci exécutent `pnpm test:live` dans l'image Docker du dépôt, montant votre répertoire de configuration local et l'espace de travail (et sourçant `~/.profile` si monté) :

-   Modèles directs : `pnpm test:docker:live-models` (script : `scripts/test-live-models-docker.sh`)
-   Gateway + agent de développement : `pnpm test:docker:live-gateway` (script : `scripts/test-live-gateway-models-docker.sh`)
-   Assistant d'intégration (TTY, échafaudage complet) : `pnpm test:docker:onboard` (script : `scripts/e2e/onboard-docker.sh`)
-   Réseau gateway (deux conteneurs, authentification WS + santé) : `pnpm test:docker:gateway-network` (script : `scripts/e2e/gateway-network-docker.sh`)
-   Plugins (chargement d'extension personnalisé + test de fumée du registre) : `pnpm test:docker:plugins` (script : `scripts/e2e/plugins-docker.sh`)

Test de fumée manuel de fil en langage simple ACP (pas CI) :

-   `bun scripts/dev/discord-acp-plain-language-smoke.ts --channel <discord-channel-id> ...`
-   Gardez ce script pour les workflows de régression/débogage. Il pourrait être nécessaire à nouveau pour la validation du routage des fils ACP, donc ne le supprimez pas.

Variables d'environnement utiles :

-   `OPENCLAW_CONFIG_DIR=...` (par défaut : `~/.openclaw`) monté sur `/home/node/.openclaw`
-   `OPENCLAW_WORKSPACE_DIR=...` (par défaut : `~/.openclaw/workspace`) monté sur `/home/node/.openclaw/workspace`
-   `OPENCLAW_PROFILE_FILE=...` (par défaut : `~/.profile`) monté sur `/home/node/.profile` et sourcé avant d'exécuter les tests
-   `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` pour restreindre l'exécution
-   `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` pour s'assurer que les identifiants proviennent du magasin de profil (pas de l'environnement)

## Vérification de la documentation

Exécutez les vérifications de documentation après les modifications de doc : `pnpm docs:list`.

## Régression hors ligne (sûre pour CI)

Ce sont des régressions de "pipeline réel" sans fournisseurs réels :

-   Appel d'outil gateway (OpenAI simulé, vraie gateway + boucle d'agent) : `src/gateway/gateway.test.ts` (cas : "exécute un appel d'outil OpenAI simulé de bout en bout via la boucle d'agent gateway")
-   Assistant gateway (WS `wizard.start`/`wizard.next`, écrit la configuration + authentification appliquée) : `src/gateway/gateway.test.ts` (cas : "exécute l'assistant via ws et écrit la configuration du jeton d'authentification")

## Évaluations de fiabilité de l'agent (compétences)

Nous avons déjà quelques tests sûrs pour CI qui se comportent comme des "évaluations de fiabilité de l'agent" :

-   Appel d'outil simulé à travers la vraie gateway + boucle d'agent (`src/gateway/gateway.test.ts`).
-   Flux d'assistant de bout en bout qui valident le câblage des sessions et les effets de configuration (`src/gateway/gateway.test.ts`).

Ce qui manque encore pour les compétences (voir [Compétences](../tools/skills.md)) :

-   **Décision :** lorsque les compétences sont listées dans l'invite, l'agent choisit-il la bonne compétence (ou évite-t-il celles qui ne sont pas pertinentes) ?
-   **Conformité :** l'agent lit-il `SKILL.md` avant utilisation et suit-il les étapes/arguments requis ?
-   **Contrats de workflow :** scénarios multi-tours qui vérifient l'ordre des outils, la persistance de l'historique des sessions et les limites du sandbox.

Les futures évaluations doivent d'abord rester déterministes :

-   Un exécuteur de scénario utilisant des fournisseurs simulés pour vérifier les appels d'outil + l'ordre, les lectures de fichiers de compétences et le câblage des sessions.
-   Une petite suite de scénarios axés sur les compétences (utilisation vs évitement, contrôle d'accès, injection d'invite).
-   Évaluations live optionnelles (opt-in, conditionnées par l'environnement) uniquement après que la suite sûre pour CI est en place.

## Ajout de régressions (guide)

Lorsque vous corrigez un problème de fournisseur/modèle découvert en live :

-   Ajoutez une régression sûre pour CI si possible (fournisseur simulé/bouchon, ou capturez la transformation exacte de la forme de requête)
-   Si c'est intrinsèquement live uniquement (limites de taux, politiques d'authentification), gardez le test live restreint et optionnel via des variables d'environnement
-   Privilégiez le ciblage de la plus petite couche qui détecte le bogue :
    -   bogue de conversion/rejeu de requête de fournisseur → test de modèles directs
    -   bogue de pipeline de session/historique/outil gateway → test de fumée gateway live ou test de simulation gateway sûr pour CI

[Débogage](./debugging.md)[Scripts](./scripts.md)