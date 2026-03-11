

  Configuration

  
# Fournisseurs de modèles

Cette page couvre les **fournisseurs de LLM/modèles** (et non les canaux de discussion comme WhatsApp/Telegram). Pour les règles de sélection des modèles, consultez [/concepts/models](./models.md).

## Règles rapides

-   Les références de modèles utilisent `fournisseur/modèle` (exemple : `opencode/claude-opus-4-6`).
-   Si vous définissez `agents.defaults.models`, cela devient la liste autorisée.
-   Aides en CLI : `openclaw onboard`, `openclaw models list`, `openclaw models set <fournisseur/modèle>`.

## Rotation des clés API

-   Prend en charge la rotation générique des fournisseurs pour les fournisseurs sélectionnés.
-   Configurez plusieurs clés via :
    -   `OPENCLAW_LIVE__KEY` (remplacement unique en direct, priorité la plus élevée)
    -   `_API_KEYS` (liste séparée par des virgules ou des points-virgules)
    -   `_API_KEY` (clé principale)
    -   `_API_KEY_*` (liste numérotée, par exemple `_API_KEY_1`)
-   Pour les fournisseurs Google, `GOOGLE_API_KEY` est également inclus comme solution de secours.
-   L'ordre de sélection des clés préserve la priorité et déduplique les valeurs.
-   Les requêtes sont réessayées avec la clé suivante uniquement en cas de réponses de limitation de débit (par exemple `429`, `rate_limit`, `quota`, `resource exhausted`).
-   Les échecs non liés à la limitation de débit échouent immédiatement ; aucune rotation de clé n'est tentée.
-   Lorsque toutes les clés candidates échouent, l'erreur finale est renvoyée depuis la dernière tentative.

## Fournisseurs intégrés (catalogue pi‑ai)

OpenClaw est livré avec le catalogue pi‑ai. Ces fournisseurs ne nécessitent **aucune** configuration `models.providers` ; il suffit de définir l'authentification et de choisir un modèle.

### OpenAI

-   Fournisseur : `openai`
-   Authentification : `OPENAI_API_KEY`
-   Rotation optionnelle : `OPENAI_API_KEYS`, `OPENAI_API_KEY_1`, `OPENAI_API_KEY_2`, plus `OPENCLAW_LIVE_OPENAI_KEY` (remplacement unique)
-   Exemple de modèle : `openai/gpt-5.1-codex`
-   CLI : `openclaw onboard --auth-choice openai-api-key`
-   Le transport par défaut est `auto` (WebSocket en premier, secours SSE)
-   Remplacement par modèle via `agents.defaults.models["openai/"].params.transport` (`"sse"`, `"websocket"`, ou `"auto"`)
-   Le préchauffage WebSocket des réponses OpenAI est activé par défaut via `params.openaiWsWarmup` (`true`/`false`)

```json
{
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

### Anthropic

-   Fournisseur : `anthropic`
-   Authentification : `ANTHROPIC_API_KEY` ou `claude setup-token`
-   Rotation optionnelle : `ANTHROPIC_API_KEYS`, `ANTHROPIC_API_KEY_1`, `ANTHROPIC_API_KEY_2`, plus `OPENCLAW_LIVE_ANTHROPIC_KEY` (remplacement unique)
-   Exemple de modèle : `anthropic/claude-opus-4-6`
-   CLI : `openclaw onboard --auth-choice token` (coller le setup-token) ou `openclaw models auth paste-token --provider anthropic`
-   Note sur la politique : la prise en charge du setup-token est une compatibilité technique ; Anthropic a bloqué certaines utilisations d'abonnement en dehors de Claude Code par le passé. Vérifiez les conditions actuelles d'Anthropic et décidez en fonction de votre tolérance au risque.
-   Recommandation : l'authentification par clé API Anthropic est la voie plus sûre et recommandée par rapport à l'authentification par setup-token d'abonnement.

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

### OpenAI Code (Codex)

-   Fournisseur : `openai-codex`
-   Authentification : OAuth (ChatGPT)
-   Exemple de modèle : `openai-codex/gpt-5.3-codex`
-   CLI : `openclaw onboard --auth-choice openai-codex` ou `openclaw models auth login --provider openai-codex`
-   Le transport par défaut est `auto` (WebSocket en premier, secours SSE)
-   Remplacement par modèle via `agents.defaults.models["openai-codex/"].params.transport` (`"sse"`, `"websocket"`, ou `"auto"`)
-   Note sur la politique : l'OAuth OpenAI Codex est explicitement pris en charge pour les outils/flux de travail externes comme OpenClaw.

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

### OpenCode Zen

-   Fournisseur : `opencode`
-   Authentification : `OPENCODE_API_KEY` (ou `OPENCODE_ZEN_API_KEY`)
-   Exemple de modèle : `opencode/claude-opus-4-6`
-   CLI : `openclaw onboard --auth-choice opencode-zen`

```json
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

### Google Gemini (clé API)

-   Fournisseur : `google`
-   Authentification : `GEMINI_API_KEY`
-   Rotation optionnelle : `GEMINI_API_KEYS`, `GEMINI_API_KEY_1`, `GEMINI_API_KEY_2`, secours `GOOGLE_API_KEY`, et `OPENCLAW_LIVE_GEMINI_KEY` (remplacement unique)
-   Exemple de modèle : `google/gemini-3-pro-preview`
-   CLI : `openclaw onboard --auth-choice gemini-api-key`

### Google Vertex, Antigravity et Gemini CLI

-   Fournisseurs : `google-vertex`, `google-antigravity`, `google-gemini-cli`
-   Authentification : Vertex utilise gcloud ADC ; Antigravity/Gemini CLI utilisent leurs flux d'authentification respectifs
-   Prudence : les OAuth Antigravity et Gemini CLI dans OpenClaw sont des intégrations non officielles. Certains utilisateurs ont signalé des restrictions de compte Google après avoir utilisé des clients tiers. Consultez les conditions de Google et utilisez un compte non critique si vous choisissez de continuer.
-   L'OAuth Antigravity est livré sous forme de plugin groupé (`google-antigravity-auth`, désactivé par défaut).
    -   Activer : `openclaw plugins enable google-antigravity-auth`
    -   Connexion : `openclaw models auth login --provider google-antigravity --set-default`
-   L'OAuth Gemini CLI est livré sous forme de plugin groupé (`google-gemini-cli-auth`, désactivé par défaut).
    -   Activer : `openclaw plugins enable google-gemini-cli-auth`
    -   Connexion : `openclaw models auth login --provider google-gemini-cli --set-default`
    -   Note : vous ne collez **pas** d'ID client ou de secret dans `openclaw.json`. Le flux de connexion CLI stocke les jetons dans les profils d'authentification sur l'hôte de la passerelle.

### Z.AI (GLM)

-   Fournisseur : `zai`
-   Authentification : `ZAI_API_KEY`
-   Exemple de modèle : `zai/glm-5`
-   CLI : `openclaw onboard --auth-choice zai-api-key`
    -   Alias : `z.ai/*` et `z-ai/*` se normalisent en `zai/*`

### Vercel AI Gateway

-   Fournisseur : `vercel-ai-gateway`
-   Authentification : `AI_GATEWAY_API_KEY`
-   Exemple de modèle : `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   CLI : `openclaw onboard --auth-choice ai-gateway-api-key`

### Kilo Gateway

-   Fournisseur : `kilocode`
-   Authentification : `KILOCODE_API_KEY`
-   Exemple de modèle : `kilocode/anthropic/claude-opus-4.6`
-   CLI : `openclaw onboard --kilocode-api-key `
-   URL de base : `https://api.kilo.ai/api/gateway/`
-   Le catalogue intégré étendu inclut GLM-5 Free, MiniMax M2.5 Free, GPT-5.2, Gemini 3 Pro Preview, Gemini 3 Flash Preview, Grok Code Fast 1 et Kimi K2.5.

Consultez [/providers/kilocode](../providers/kilocode.md) pour les détails de configuration.

### Autres fournisseurs intégrés

-   OpenRouter : `openrouter` (`OPENROUTER_API_KEY`)
-   Exemple de modèle : `openrouter/anthropic/claude-sonnet-4-5`
-   Kilo Gateway : `kilocode` (`KILOCODE_API_KEY`)
-   Exemple de modèle : `kilocode/anthropic/claude-opus-4.6`
-   xAI : `xai` (`XAI_API_KEY`)
-   Mistral : `mistral` (`MISTRAL_API_KEY`)
-   Exemple de modèle : `mistral/mistral-large-latest`
-   CLI : `openclaw onboard --auth-choice mistral-api-key`
-   Groq : `groq` (`GROQ_API_KEY`)
-   Cerebras : `cerebras` (`CEREBRAS_API_KEY`)
    -   Les modèles GLM sur Cerebras utilisent les identifiants `zai-glm-4.7` et `zai-glm-4.6`.
    -   URL de base compatible OpenAI : `https://api.cerebras.ai/v1`.
-   GitHub Copilot : `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)
-   Hugging Face Inference : `huggingface` (`HUGGINGFACE_HUB_TOKEN` ou `HF_TOKEN`) — routeur compatible OpenAI ; exemple de modèle : `huggingface/deepseek-ai/DeepSeek-R1` ; CLI : `openclaw onboard --auth-choice huggingface-api-key`. Voir [Hugging Face (Inference)](../providers/huggingface.md).

## Fournisseurs via models.providers (personnalisé/URL de base)

Utilisez `models.providers` (ou `models.json`) pour ajouter des fournisseurs **personnalisés** ou des proxys compatibles OpenAI/Anthropic.

### Moonshot AI (Kimi)

Moonshot utilise des points de terminaison compatibles OpenAI, configurez-le donc comme un fournisseur personnalisé :

-   Fournisseur : `moonshot`
-   Authentification : `MOONSHOT_API_KEY`
-   Exemple de modèle : `moonshot/kimi-k2.5`

Identifiants de modèle Kimi K2 :

-   `moonshot/kimi-k2.5`
-   `moonshot/kimi-k2-0905-preview`
-   `moonshot/kimi-k2-turbo-preview`
-   `moonshot/kimi-k2-thinking`
-   `moonshot/kimi-k2-thinking-turbo`

```json
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }],
      },
    },
  },
}
```

### Kimi Coding

Kimi Coding utilise le point de terminaison compatible Anthropic de Moonshot AI :

-   Fournisseur : `kimi-coding`
-   Authentification : `KIMI_API_KEY`
-   Exemple de modèle : `kimi-coding/k2p5`

```json
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-coding/k2p5" } },
  },
}
```

### Qwen OAuth (niveau gratuit)

Qwen fournit un accès OAuth à Qwen Coder + Vision via un flux de code d'appareil. Activez le plugin groupé, puis connectez-vous :

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

Références de modèles :

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

Consultez [/providers/qwen](../providers/qwen.md) pour les détails de configuration et les notes.

### Volcano Engine (Doubao)

Volcano Engine (火山引擎) fournit l'accès à Doubao et à d'autres modèles en Chine.

-   Fournisseur : `volcengine` (codage : `volcengine-plan`)
-   Authentification : `VOLCANO_ENGINE_API_KEY`
-   Exemple de modèle : `volcengine/doubao-seed-1-8-251228`
-   CLI : `openclaw onboard --auth-choice volcengine-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "volcengine/doubao-seed-1-8-251228" } },
  },
}
```

Modèles disponibles :

-   `volcengine/doubao-seed-1-8-251228` (Doubao Seed 1.8)
-   `volcengine/doubao-seed-code-preview-251028`
-   `volcengine/kimi-k2-5-260127` (Kimi K2.5)
-   `volcengine/glm-4-7-251222` (GLM 4.7)
-   `volcengine/deepseek-v3-2-251201` (DeepSeek V3.2 128K)

Modèles de codage (`volcengine-plan`) :

-   `volcengine-plan/ark-code-latest`
-   `volcengine-plan/doubao-seed-code`
-   `volcengine-plan/kimi-k2.5`
-   `volcengine-plan/kimi-k2-thinking`
-   `volcengine-plan/glm-4.7`

### BytePlus (International)

BytePlus ARK fournit l'accès aux mêmes modèles que Volcano Engine pour les utilisateurs internationaux.

-   Fournisseur : `byteplus` (codage : `byteplus-plan`)
-   Authentification : `BYTEPLUS_API_KEY`
-   Exemple de modèle : `byteplus/seed-1-8-251228`
-   CLI : `openclaw onboard --auth-choice byteplus-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "byteplus/seed-1-8-251228" } },
  },
}
```

Modèles disponibles :

-   `byteplus/seed-1-8-251228` (Seed 1.8)
-   `byteplus/kimi-k2-5-260127` (Kimi K2.5)
-   `byteplus/glm-4-7-251222` (GLM 4.7)

Modèles de codage (`byteplus-plan`) :

-   `byteplus-plan/ark-code-latest`
-   `byteplus-plan/doubao-seed-code`
-   `byteplus-plan/kimi-k2.5`
-   `byteplus-plan/kimi-k2-thinking`
-   `byteplus-plan/glm-4.7`

### Synthetic

Synthetic fournit des modèles compatibles Anthropic derrière le fournisseur `synthetic` :

-   Fournisseur : `synthetic`
-   Authentification : `SYNTHETIC_API_KEY`
-   Exemple de modèle : `synthetic/hf:MiniMaxAI/MiniMax-M2.5`
-   CLI : `openclaw onboard --auth-choice synthetic-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.5" } },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [{ id: "hf:MiniMaxAI/MiniMax-M2.5", name: "MiniMax M2.5" }],
      },
    },
  },
}
```

### MiniMax

MiniMax est configuré via `models.providers` car il utilise des points de terminaison personnalisés :

-   MiniMax (compatible Anthropic) : `--auth-choice minimax-api`
-   Authentification : `MINIMAX_API_KEY`

Consultez [/providers/minimax](../providers/minimax.md) pour les détails de configuration, les options de modèles et les extraits de configuration.

### Ollama

Ollama est un runtime LLM local qui fournit une API compatible OpenAI :

-   Fournisseur : `ollama`
-   Authentification : Aucune requise (serveur local)
-   Exemple de modèle : `ollama/llama3.3`
-   Installation : [https://ollama.ai](https://ollama.ai)

```bash
# Installez Ollama, puis téléchargez un modèle :
ollama pull llama3.3
```

```json
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } },
  },
}
```

Ollama est automatiquement détecté lorsqu'il fonctionne localement à `http://127.0.0.1:11434/v1`. Consultez [/providers/ollama](../providers/ollama.md) pour les recommandations de modèles et la configuration personnalisée.

### vLLM

vLLM est un serveur compatible OpenAI local (ou auto-hébergé) :

-   Fournisseur : `vllm`
-   Authentification : Optionnelle (dépend de votre serveur)
-   URL de base par défaut : `http://127.0.0.1:8000/v1`

Pour activer la détection automatique localement (toute valeur fonctionne si votre serveur n'applique pas d'authentification) :

```bash
export VLLM_API_KEY="vllm-local"
```

Puis définissez un modèle (remplacez par l'un des identifiants renvoyés par `/v1/models`) :

```json
{
  agents: {
    defaults: { model: { primary: "vllm/your-model-id" } },
  },
}
```

Consultez [/providers/vllm](../providers/vllm.md) pour les détails.

### Proxys locaux (LM Studio, vLLM, LiteLLM, etc.)

Exemple (compatible OpenAI) :

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Notes :

-   Pour les fournisseurs personnalisés, `reasoning`, `input`, `cost`, `contextWindow` et `maxTokens` sont optionnels. Lorsqu'ils sont omis, OpenClaw utilise par défaut :
    -   `reasoning: false`
    -   `input: ["text"]`
    -   `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
    -   `contextWindow: 200000`
    -   `maxTokens: 8192`
-   Recommandé : définissez des valeurs explicites qui correspondent aux limites de votre proxy/modèle.

## Exemples CLI

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-6
openclaw models list
```

Voir aussi : [/gateway/configuration](../gateway/configuration.md) pour des exemples de configuration complets.

[CLI des modèles](./models.md)[Basculement de modèle](./model-failover.md)