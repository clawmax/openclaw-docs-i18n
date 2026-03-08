

  Configuración

  
# Proveedores de Modelos

Esta página cubre **proveedores de LLM/modelos** (no canales de chat como WhatsApp/Telegram). Para reglas de selección de modelos, consulta [/concepts/models](./models.md).

## Reglas rápidas

-   Las referencias a modelos usan `proveedor/modelo` (ejemplo: `opencode/claude-opus-4-6`).
-   Si configuras `agents.defaults.models`, se convierte en la lista de permitidos.
-   Ayudantes de CLI: `openclaw onboard`, `openclaw models list`, `openclaw models set <proveedor/modelo>`.

## Rotación de claves API

-   Soporta rotación genérica de proveedores para proveedores seleccionados.
-   Configura múltiples claves mediante:
    -   `OPENCLAW_LIVE__KEY` (anulación única en vivo, prioridad más alta)
    -   `_API_KEYS` (lista separada por comas o punto y coma)
    -   `_API_KEY` (clave principal)
    -   `_API_KEY_*` (lista numerada, ej. `_API_KEY_1`)
-   Para proveedores de Google, `GOOGLE_API_KEY` también se incluye como respaldo.
-   El orden de selección de claves preserva la prioridad y elimina valores duplicados.
-   Las solicitudes se reintentan con la siguiente clave solo en respuestas de límite de tasa (por ejemplo `429`, `rate_limit`, `quota`, `resource exhausted`).
-   Los fallos que no son de límite de tasa fallan inmediatamente; no se intenta la rotación de claves.
-   Cuando todas las claves candidatas fallan, se devuelve el error final del último intento.

## Proveedores integrados (catálogo pi-ai)

OpenClaw incluye el catálogo pi‑ai. Estos proveedores **no** requieren configuración `models.providers`; solo configura la autenticación y elige un modelo.

### OpenAI

-   Proveedor: `openai`
-   Autenticación: `OPENAI_API_KEY`
-   Rotación opcional: `OPENAI_API_KEYS`, `OPENAI_API_KEY_1`, `OPENAI_API_KEY_2`, más `OPENCLAW_LIVE_OPENAI_KEY` (anulación única)
-   Modelo de ejemplo: `openai/gpt-5.1-codex`
-   CLI: `openclaw onboard --auth-choice openai-api-key`
-   El transporte predeterminado es `auto` (WebSocket primero, SSE como respaldo)
-   Anular por modelo mediante `agents.defaults.models["openai/"].params.transport` (`"sse"`, `"websocket"`, o `"auto"`)
-   El calentamiento de WebSocket para respuestas de OpenAI está habilitado por defecto mediante `params.openaiWsWarmup` (`true`/`false`)

```json
{
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

### Anthropic

-   Proveedor: `anthropic`
-   Autenticación: `ANTHROPIC_API_KEY` o `claude setup-token`
-   Rotación opcional: `ANTHROPIC_API_KEYS`, `ANTHROPIC_API_KEY_1`, `ANTHROPIC_API_KEY_2`, más `OPENCLAW_LIVE_ANTHROPIC_KEY` (anulación única)
-   Modelo de ejemplo: `anthropic/claude-opus-4-6`
-   CLI: `openclaw onboard --auth-choice token` (pegar setup-token) o `openclaw models auth paste-token --provider anthropic`
-   Nota de política: el soporte para setup-token es una compatibilidad técnica; Anthropic ha bloqueado algunos usos de suscripción fuera de Claude Code en el pasado. Verifica los términos actuales de Anthropic y decide según tu tolerancia al riesgo.
-   Recomendación: la autenticación con clave API de Anthropic es el camino más seguro y recomendado sobre la autenticación con setup-token de suscripción.

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

### OpenAI Code (Codex)

-   Proveedor: `openai-codex`
-   Autenticación: OAuth (ChatGPT)
-   Modelo de ejemplo: `openai-codex/gpt-5.3-codex`
-   CLI: `openclaw onboard --auth-choice openai-codex` o `openclaw models auth login --provider openai-codex`
-   El transporte predeterminado es `auto` (WebSocket primero, SSE como respaldo)
-   Anular por modelo mediante `agents.defaults.models["openai-codex/"].params.transport` (`"sse"`, `"websocket"`, o `"auto"`)
-   Nota de política: OAuth de OpenAI Codex está explícitamente soportado para herramientas/flujos de trabajo externos como OpenClaw.

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

### OpenCode Zen

-   Proveedor: `opencode`
-   Autenticación: `OPENCODE_API_KEY` (o `OPENCODE_ZEN_API_KEY`)
-   Modelo de ejemplo: `opencode/claude-opus-4-6`
-   CLI: `openclaw onboard --auth-choice opencode-zen`

```json
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

### Google Gemini (clave API)

-   Proveedor: `google`
-   Autenticación: `GEMINI_API_KEY`
-   Rotación opcional: `GEMINI_API_KEYS`, `GEMINI_API_KEY_1`, `GEMINI_API_KEY_2`, `GOOGLE_API_KEY` como respaldo, y `OPENCLAW_LIVE_GEMINI_KEY` (anulación única)
-   Modelo de ejemplo: `google/gemini-3-pro-preview`
-   CLI: `openclaw onboard --auth-choice gemini-api-key`

### Google Vertex, Antigravity y Gemini CLI

-   Proveedores: `google-vertex`, `google-antigravity`, `google-gemini-cli`
-   Autenticación: Vertex usa gcloud ADC; Antigravity/Gemini CLI usan sus respectivos flujos de autenticación
-   Precaución: OAuth de Antigravity y Gemini CLI en OpenClaw son integraciones no oficiales. Algunos usuarios han reportado restricciones en sus cuentas de Google después de usar clientes de terceros. Revisa los términos de Google y usa una cuenta no crítica si decides proceder.
-   OAuth de Antigravity se incluye como un plugin empaquetado (`google-antigravity-auth`, deshabilitado por defecto).
    -   Habilitar: `openclaw plugins enable google-antigravity-auth`
    -   Iniciar sesión: `openclaw models auth login --provider google-antigravity --set-default`
-   OAuth de Gemini CLI se incluye como un plugin empaquetado (`google-gemini-cli-auth`, deshabilitado por defecto).
    -   Habilitar: `openclaw plugins enable google-gemini-cli-auth`
    -   Iniciar sesión: `openclaw models auth login --provider google-gemini-cli --set-default`
    -   Nota: **no** pegas un ID de cliente o secreto en `openclaw.json`. El flujo de inicio de sesión CLI almacena tokens en perfiles de autenticación en el host de la puerta de enlace.

### Z.AI (GLM)

-   Proveedor: `zai`
-   Autenticación: `ZAI_API_KEY`
-   Modelo de ejemplo: `zai/glm-5`
-   CLI: `openclaw onboard --auth-choice zai-api-key`
    -   Alias: `z.ai/*` y `z-ai/*` se normalizan a `zai/*`

### Vercel AI Gateway

-   Proveedor: `vercel-ai-gateway`
-   Autenticación: `AI_GATEWAY_API_KEY`
-   Modelo de ejemplo: `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

### Kilo Gateway

-   Proveedor: `kilocode`
-   Autenticación: `KILOCODE_API_KEY`
-   Modelo de ejemplo: `kilocode/anthropic/claude-opus-4.6`
-   CLI: `openclaw onboard --kilocode-api-key `
-   URL base: `https://api.kilo.ai/api/gateway/`
-   El catálogo integrado ampliado incluye GLM-5 Free, MiniMax M2.5 Free, GPT-5.2, Gemini 3 Pro Preview, Gemini 3 Flash Preview, Grok Code Fast 1 y Kimi K2.5.

Consulta [/providers/kilocode](../providers/kilocode.md) para detalles de configuración.

### Otros proveedores integrados

-   OpenRouter: `openrouter` (`OPENROUTER_API_KEY`)
-   Modelo de ejemplo: `openrouter/anthropic/claude-sonnet-4-5`
-   Kilo Gateway: `kilocode` (`KILOCODE_API_KEY`)
-   Modelo de ejemplo: `kilocode/anthropic/claude-opus-4.6`
-   xAI: `xai` (`XAI_API_KEY`)
-   Mistral: `mistral` (`MISTRAL_API_KEY`)
-   Modelo de ejemplo: `mistral/mistral-large-latest`
-   CLI: `openclaw onboard --auth-choice mistral-api-key`
-   Groq: `groq` (`GROQ_API_KEY`)
-   Cerebras: `cerebras` (`CEREBRAS_API_KEY`)
    -   Los modelos GLM en Cerebras usan los ids `zai-glm-4.7` y `zai-glm-4.6`.
    -   URL base compatible con OpenAI: `https://api.cerebras.ai/v1`.
-   GitHub Copilot: `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)
-   Hugging Face Inference: `huggingface` (`HUGGINGFACE_HUB_TOKEN` o `HF_TOKEN`) — enrutador compatible con OpenAI; modelo de ejemplo: `huggingface/deepseek-ai/DeepSeek-R1`; CLI: `openclaw onboard --auth-choice huggingface-api-key`. Consulta [Hugging Face (Inference)](../providers/huggingface.md).

## Proveedores vía models.providers (personalizados/URL base)

Usa `models.providers` (o `models.json`) para agregar proveedores **personalizados** o proxies compatibles con OpenAI/Anthropic.

### Moonshot AI (Kimi)

Moonshot usa endpoints compatibles con OpenAI, así que configúralo como un proveedor personalizado:

-   Proveedor: `moonshot`
-   Autenticación: `MOONSHOT_API_KEY`
-   Modelo de ejemplo: `moonshot/kimi-k2.5`

IDs de modelo Kimi K2:

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

Kimi Coding usa el endpoint compatible con Anthropic de Moonshot AI:

-   Proveedor: `kimi-coding`
-   Autenticación: `KIMI_API_KEY`
-   Modelo de ejemplo: `kimi-coding/k2p5`

```json
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-coding/k2p5" } },
  },
}
```

### Qwen OAuth (nivel gratuito)

Qwen proporciona acceso OAuth a Qwen Coder + Vision mediante un flujo de código de dispositivo. Habilita el plugin empaquetado, luego inicia sesión:

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

Referencias de modelo:

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

Consulta [/providers/qwen](../providers/qwen.md) para detalles de configuración y notas.

### Volcano Engine (Doubao)

Volcano Engine (火山引擎) proporciona acceso a Doubao y otros modelos en China.

-   Proveedor: `volcengine` (codificación: `volcengine-plan`)
-   Autenticación: `VOLCANO_ENGINE_API_KEY`
-   Modelo de ejemplo: `volcengine/doubao-seed-1-8-251228`
-   CLI: `openclaw onboard --auth-choice volcengine-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "volcengine/doubao-seed-1-8-251228" } },
  },
}
```

Modelos disponibles:

-   `volcengine/doubao-seed-1-8-251228` (Doubao Seed 1.8)
-   `volcengine/doubao-seed-code-preview-251028`
-   `volcengine/kimi-k2-5-260127` (Kimi K2.5)
-   `volcengine/glm-4-7-251222` (GLM 4.7)
-   `volcengine/deepseek-v3-2-251201` (DeepSeek V3.2 128K)

Modelos de codificación (`volcengine-plan`):

-   `volcengine-plan/ark-code-latest`
-   `volcengine-plan/doubao-seed-code`
-   `volcengine-plan/kimi-k2.5`
-   `volcengine-plan/kimi-k2-thinking`
-   `volcengine-plan/glm-4.7`

### BytePlus (Internacional)

BytePlus ARK proporciona acceso a los mismos modelos que Volcano Engine para usuarios internacionales.

-   Proveedor: `byteplus` (codificación: `byteplus-plan`)
-   Autenticación: `BYTEPLUS_API_KEY`
-   Modelo de ejemplo: `byteplus/seed-1-8-251228`
-   CLI: `openclaw onboard --auth-choice byteplus-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "byteplus/seed-1-8-251228" } },
  },
}
```

Modelos disponibles:

-   `byteplus/seed-1-8-251228` (Seed 1.8)
-   `byteplus/kimi-k2-5-260127` (Kimi K2.5)
-   `byteplus/glm-4-7-251222` (GLM 4.7)

Modelos de codificación (`byteplus-plan`):

-   `byteplus-plan/ark-code-latest`
-   `byteplus-plan/doubao-seed-code`
-   `byteplus-plan/kimi-k2.5`
-   `byteplus-plan/kimi-k2-thinking`
-   `byteplus-plan/glm-4.7`

### Synthetic

Synthetic proporciona modelos compatibles con Anthropic detrás del proveedor `synthetic`:

-   Proveedor: `synthetic`
-   Autenticación: `SYNTHETIC_API_KEY`
-   Modelo de ejemplo: `synthetic/hf:MiniMaxAI/MiniMax-M2.5`
-   CLI: `openclaw onboard --auth-choice synthetic-api-key`

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

MiniMax se configura mediante `models.providers` porque usa endpoints personalizados:

-   MiniMax (compatible con Anthropic): `--auth-choice minimax-api`
-   Autenticación: `MINIMAX_API_KEY`

Consulta [/providers/minimax](../providers/minimax.md) para detalles de configuración, opciones de modelos y fragmentos de configuración.

### Ollama

Ollama es un entorno de ejecución de LLM local que proporciona una API compatible con OpenAI:

-   Proveedor: `ollama`
-   Autenticación: No se requiere (servidor local)
-   Modelo de ejemplo: `ollama/llama3.3`
-   Instalación: [https://ollama.ai](https://ollama.ai)

```bash
# Instala Ollama, luego descarga un modelo:
ollama pull llama3.3
```

```json
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } },
  },
}
```

Ollama se detecta automáticamente cuando se ejecuta localmente en `http://127.0.0.1:11434/v1`. Consulta [/providers/ollama](../providers/ollama.md) para recomendaciones de modelos y configuración personalizada.

### vLLM

vLLM es un servidor compatible con OpenAI local (o autoalojado):

-   Proveedor: `vllm`
-   Autenticación: Opcional (depende de tu servidor)
-   URL base predeterminada: `http://127.0.0.1:8000/v1`

Para optar por la auto-detección local (cualquier valor funciona si tu servidor no aplica autenticación):

```bash
export VLLM_API_KEY="vllm-local"
```

Luego establece un modelo (reemplaza con uno de los IDs devueltos por `/v1/models`):

```json
{
  agents: {
    defaults: { model: { primary: "vllm/your-model-id" } },
  },
}
```

Consulta [/providers/vllm](../providers/vllm.md) para detalles.

### Proxies locales (LM Studio, vLLM, LiteLLM, etc.)

Ejemplo (compatible con OpenAI):

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

Notas:

-   Para proveedores personalizados, `reasoning`, `input`, `cost`, `contextWindow` y `maxTokens` son opcionales. Cuando se omiten, OpenClaw usa por defecto:
    -   `reasoning: false`
    -   `input: ["text"]`
    -   `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
    -   `contextWindow: 200000`
    -   `maxTokens: 8192`
-   Recomendado: establece valores explícitos que coincidan con los límites de tu proxy/modelo.

## Ejemplos de CLI

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-6
openclaw models list
```

Consulta también: [/gateway/configuration](../gateway/configuration.md) para ejemplos de configuración completos.

[CLI de Modelos](./models.md)[Conmutación por Error de Modelo](./model-failover.md)