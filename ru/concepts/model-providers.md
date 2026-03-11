

  Конфигурация

  
# Провайдеры моделей

Эта страница посвящена **провайдерам LLM/моделей** (не каналам чата, таким как WhatsApp/Telegram). Для правил выбора моделей см. [/concepts/models](./models.md).

## Быстрые правила

-   Ссылки на модели используют формат `провайдер/модель` (пример: `opencode/claude-opus-4-6`).
-   Если вы установите `agents.defaults.models`, это станет разрешительным списком.
-   CLI-помощники: `openclaw onboard`, `openclaw models list`, `openclaw models set <провайдер/модель>`.

## Ротация API-ключей

-   Поддерживает ротацию ключей для выбранных провайдеров.
-   Настройте несколько ключей через:
    -   `OPENCLAW_LIVE_<ПРОВАЙДЕР>_KEY` (один живой ключ для переопределения, наивысший приоритет)
    -   `<ПРОВАЙДЕР>_API_KEYS` (список через запятую или точку с запятой)
    -   `<ПРОВАЙДЕР>_API_KEY` (основной ключ)
    -   `<ПРОВАЙДЕР>_API_KEY_*` (нумерованный список, например `<ПРОВАЙДЕР>_API_KEY_1`)
-   Для провайдеров Google `GOOGLE_API_KEY` также используется как запасной вариант.
-   Порядок выбора ключей сохраняет приоритет и удаляет дубликаты.
-   Запросы повторяются со следующим ключом только при ответах с ограничением скорости (например, `429`, `rate_limit`, `quota`, `resource exhausted`).
-   Сбои, не связанные с ограничением скорости, завершаются немедленно; ротация ключей не предпринимается.
-   Когда все кандидатные ключи терпят неудачу, возвращается последняя ошибка.

## Встроенные провайдеры (каталог pi-ai)

OpenClaw поставляется с каталогом pi‑ai. Эти провайдеры **не требуют** конфигурации `models.providers`; просто настройте аутентификацию и выберите модель.

### OpenAI

-   Провайдер: `openai`
-   Аутентификация: `OPENAI_API_KEY`
-   Опциональная ротация: `OPENAI_API_KEYS`, `OPENAI_API_KEY_1`, `OPENAI_API_KEY_2`, плюс `OPENCLAW_LIVE_OPENAI_KEY` (одиночное переопределение)
-   Пример модели: `openai/gpt-5.1-codex`
-   CLI: `openclaw onboard --auth-choice openai-api-key`
-   Транспорт по умолчанию `auto` (сначала WebSocket, затем SSE)
-   Переопределение для каждой модели через `agents.defaults.models["openai/<модель>"].params.transport` (`"sse"`, `"websocket"` или `"auto"`)
-   Прогрев WebSocket для ответов OpenAI по умолчанию включен через `params.openaiWsWarmup` (`true`/`false`)

```json
{
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

### Anthropic

-   Провайдер: `anthropic`
-   Аутентификация: `ANTHROPIC_API_KEY` или `claude setup-token`
-   Опциональная ротация: `ANTHROPIC_API_KEYS`, `ANTHROPIC_API_KEY_1`, `ANTHROPIC_API_KEY_2`, плюс `OPENCLAW_LIVE_ANTHROPIC_KEY` (одиночное переопределение)
-   Пример модели: `anthropic/claude-opus-4-6`
-   CLI: `openclaw onboard --auth-choice token` (вставьте setup-token) или `openclaw models auth paste-token --provider anthropic`
-   Примечание о политике: поддержка setup-token — это техническая совместимость; Anthropic в прошлом блокировал некоторое использование подписок вне Claude Code. Проверьте текущие условия Anthropic и принимайте решение, исходя из вашей терпимости к риску.
-   Рекомендация: аутентификация через API-ключ Anthropic — более безопасный и рекомендуемый путь, чем аутентификация через setup-token подписки.

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

### OpenAI Code (Codex)

-   Провайдер: `openai-codex`
-   Аутентификация: OAuth (ChatGPT)
-   Пример модели: `openai-codex/gpt-5.3-codex`
-   CLI: `openclaw onboard --auth-choice openai-codex` или `openclaw models auth login --provider openai-codex`
-   Транспорт по умолчанию `auto` (сначала WebSocket, затем SSE)
-   Переопределение для каждой модели через `agents.defaults.models["openai-codex/<модель>"].params.transport` (`"sse"`, `"websocket"` или `"auto"`)
-   Примечание о политике: OAuth для OpenAI Codex явно поддерживается для внешних инструментов/воркфлоу, таких как OpenClaw.

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

### OpenCode Zen

-   Провайдер: `opencode`
-   Аутентификация: `OPENCODE_API_KEY` (или `OPENCODE_ZEN_API_KEY`)
-   Пример модели: `opencode/claude-opus-4-6`
-   CLI: `openclaw onboard --auth-choice opencode-zen`

```json
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

### Google Gemini (API-ключ)

-   Провайдер: `google`
-   Аутентификация: `GEMINI_API_KEY`
-   Опциональная ротация: `GEMINI_API_KEYS`, `GEMINI_API_KEY_1`, `GEMINI_API_KEY_2`, запасной `GOOGLE_API_KEY` и `OPENCLAW_LIVE_GEMINI_KEY` (одиночное переопределение)
-   Пример модели: `google/gemini-3-pro-preview`
-   CLI: `openclaw onboard --auth-choice gemini-api-key`

### Google Vertex, Antigravity и Gemini CLI

-   Провайдеры: `google-vertex`, `google-antigravity`, `google-gemini-cli`
-   Аутентификация: Vertex использует gcloud ADC; Antigravity/Gemini CLI используют соответствующие потоки аутентификации
-   Внимание: OAuth для Antigravity и Gemini CLI в OpenClaw — неофициальные интеграции. Некоторые пользователи сообщали об ограничениях аккаунтов Google после использования сторонних клиентов. Ознакомьтесь с условиями Google и используйте некритичный аккаунт, если решите продолжить.
-   OAuth для Antigravity поставляется в виде встроенного плагина (`google-antigravity-auth`, отключен по умолчанию).
    -   Включить: `openclaw plugins enable google-antigravity-auth`
    -   Войти: `openclaw models auth login --provider google-antigravity --set-default`
-   OAuth для Gemini CLI поставляется в виде встроенного плагина (`google-gemini-cli-auth`, отключен по умолчанию).
    -   Включить: `openclaw plugins enable google-gemini-cli-auth`
    -   Войти: `openclaw models auth login --provider google-gemini-cli --set-default`
    -   Примечание: вам **не нужно** вставлять client id или secret в `openclaw.json`. Процесс входа через CLI сохраняет токены в профилях аутентификации на хосте шлюза.

### Z.AI (GLM)

-   Провайдер: `zai`
-   Аутентификация: `ZAI_API_KEY`
-   Пример модели: `zai/glm-5`
-   CLI: `openclaw onboard --auth-choice zai-api-key`
    -   Алиасы: `z.ai/*` и `z-ai/*` нормализуются до `zai/*`

### Vercel AI Gateway

-   Провайдер: `vercel-ai-gateway`
-   Аутентификация: `AI_GATEWAY_API_KEY`
-   Пример модели: `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

### Kilo Gateway

-   Провайдер: `kilocode`
-   Аутентификация: `KILOCODE_API_KEY`
-   Пример модели: `kilocode/anthropic/claude-opus-4.6`
-   CLI: `openclaw onboard --kilocode-api-key `
-   Базовый URL: `https://api.kilo.ai/api/gateway/`
-   Расширенный встроенный каталог включает GLM-5 Free, MiniMax M2.5 Free, GPT-5.2, Gemini 3 Pro Preview, Gemini 3 Flash Preview, Grok Code Fast 1 и Kimi K2.5.

См. [/providers/kilocode](../providers/kilocode.md) для подробностей настройки.

### Другие встроенные провайдеры

-   OpenRouter: `openrouter` (`OPENROUTER_API_KEY`)
-   Пример модели: `openrouter/anthropic/claude-sonnet-4-5`
-   Kilo Gateway: `kilocode` (`KILOCODE_API_KEY`)
-   Пример модели: `kilocode/anthropic/claude-opus-4.6`
-   xAI: `xai` (`XAI_API_KEY`)
-   Mistral: `mistral` (`MISTRAL_API_KEY`)
-   Пример модели: `mistral/mistral-large-latest`
-   CLI: `openclaw onboard --auth-choice mistral-api-key`
-   Groq: `groq` (`GROQ_API_KEY`)
-   Cerebras: `cerebras` (`CEREBRAS_API_KEY`)
    -   Модели GLM на Cerebras используют идентификаторы `zai-glm-4.7` и `zai-glm-4.6`.
    -   Совместимый с OpenAI базовый URL: `https://api.cerebras.ai/v1`.
-   GitHub Copilot: `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)
-   Hugging Face Inference: `huggingface` (`HUGGINGFACE_HUB_TOKEN` или `HF_TOKEN`) — совместимый с OpenAI роутер; пример модели: `huggingface/deepseek-ai/DeepSeek-R1`; CLI: `openclaw onboard --auth-choice huggingface-api-key`. См. [Hugging Face (Inference)](../providers/huggingface.md).

## Провайдеры через models.providers (пользовательские/базовый URL)

Используйте `models.providers` (или `models.json`) для добавления **пользовательских** провайдеров или прокси, совместимых с OpenAI/Anthropic.

### Moonshot AI (Kimi)

Moonshot использует конечные точки, совместимые с OpenAI, поэтому настройте его как пользовательский провайдер:

-   Провайдер: `moonshot`
-   Аутентификация: `MOONSHOT_API_KEY`
-   Пример модели: `moonshot/kimi-k2.5`

Идентификаторы моделей Kimi K2:

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

Kimi Coding использует конечную точку Moonshot AI, совместимую с Anthropic:

-   Провайдер: `kimi-coding`
-   Аутентификация: `KIMI_API_KEY`
-   Пример модели: `kimi-coding/k2p5`

```json
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-coding/k2p5" } },
  },
}
```

### Qwen OAuth (бесплатный тариф)

Qwen предоставляет доступ через OAuth к Qwen Coder + Vision с помощью потока device-code. Включите встроенный плагин, затем войдите:

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

Ссылки на модели:

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

См. [/providers/qwen](../providers/qwen.md) для подробностей настройки и примечаний.

### Volcano Engine (Doubao)

Volcano Engine (火山引擎) предоставляет доступ к Doubao и другим моделям в Китае.

-   Провайдер: `volcengine` (кодирование: `volcengine-plan`)
-   Аутентификация: `VOLCANO_ENGINE_API_KEY`
-   Пример модели: `volcengine/doubao-seed-1-8-251228`
-   CLI: `openclaw onboard --auth-choice volcengine-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "volcengine/doubao-seed-1-8-251228" } },
  },
}
```

Доступные модели:

-   `volcengine/doubao-seed-1-8-251228` (Doubao Seed 1.8)
-   `volcengine/doubao-seed-code-preview-251028`
-   `volcengine/kimi-k2-5-260127` (Kimi K2.5)
-   `volcengine/glm-4-7-251222` (GLM 4.7)
-   `volcengine/deepseek-v3-2-251201` (DeepSeek V3.2 128K)

Модели для кодирования (`volcengine-plan`):

-   `volcengine-plan/ark-code-latest`
-   `volcengine-plan/doubao-seed-code`
-   `volcengine-plan/kimi-k2.5`
-   `volcengine-plan/kimi-k2-thinking`
-   `volcengine-plan/glm-4.7`

### BytePlus (Международный)

BytePlus ARK предоставляет доступ к тем же моделям, что и Volcano Engine, для международных пользователей.

-   Провайдер: `byteplus` (кодирование: `byteplus-plan`)
-   Аутентификация: `BYTEPLUS_API_KEY`
-   Пример модели: `byteplus/seed-1-8-251228`
-   CLI: `openclaw onboard --auth-choice byteplus-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "byteplus/seed-1-8-251228" } },
  },
}
```

Доступные модели:

-   `byteplus/seed-1-8-251228` (Seed 1.8)
-   `byteplus/kimi-k2-5-260127` (Kimi K2.5)
-   `byteplus/glm-4-7-251222` (GLM 4.7)

Модели для кодирования (`byteplus-plan`):

-   `byteplus-plan/ark-code-latest`
-   `byteplus-plan/doubao-seed-code`
-   `byteplus-plan/kimi-k2.5`
-   `byteplus-plan/kimi-k2-thinking`
-   `byteplus-plan/glm-4.7`

### Synthetic

Synthetic предоставляет модели, совместимые с Anthropic, через провайдера `synthetic`:

-   Провайдер: `synthetic`
-   Аутентификация: `SYNTHETIC_API_KEY`
-   Пример модели: `synthetic/hf:MiniMaxAI/MiniMax-M2.5`
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

MiniMax настраивается через `models.providers`, потому что использует пользовательские конечные точки:

-   MiniMax (совместимый с Anthropic): `--auth-choice minimax-api`
-   Аутентификация: `MINIMAX_API_KEY`

См. [/providers/minimax](../providers/minimax.md) для подробностей настройки, вариантов моделей и примеров конфигурации.

### Ollama

Ollama — это локальная среда выполнения LLM, предоставляющая API, совместимый с OpenAI:

-   Провайдер: `ollama`
-   Аутентификация: Не требуется (локальный сервер)
-   Пример модели: `ollama/llama3.3`
-   Установка: [https://ollama.ai](https://ollama.ai)

```bash
# Установите Ollama, затем загрузите модель:
ollama pull llama3.3
```

```json
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } },
  },
}
```

Ollama автоматически обнаруживается при локальном запуске на `http://127.0.0.1:11434/v1`. См. [/providers/ollama](../providers/ollama.md) для рекомендаций по моделям и пользовательской конфигурации.

### vLLM

vLLM — это локальный (или самостоятельно размещенный) сервер, совместимый с OpenAI:

-   Провайдер: `vllm`
-   Аутентификация: Опционально (зависит от вашего сервера)
-   Базовый URL по умолчанию: `http://127.0.0.1:8000/v1`

Чтобы включить автообнаружение локально (любое значение подходит, если ваш сервер не требует аутентификации):

```bash
export VLLM_API_KEY="vllm-local"
```

Затем установите модель (замените на один из идентификаторов, возвращаемых `/v1/models`):

```json
{
  agents: {
    defaults: { model: { primary: "vllm/your-model-id" } },
  },
}
```

См. [/providers/vllm](../providers/vllm.md) для подробностей.

### Локальные прокси (LM Studio, vLLM, LiteLLM и т.д.)

Пример (совместимый с OpenAI):

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

Примечания:

-   Для пользовательских провайдеров `reasoning`, `input`, `cost`, `contextWindow` и `maxTokens` являются опциональными. Если они опущены, OpenClaw использует значения по умолчанию:
    -   `reasoning: false`
    -   `input: ["text"]`
    -   `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
    -   `contextWindow: 200000`
    -   `maxTokens: 8192`
-   Рекомендуется: установите явные значения, соответствующие ограничениям вашего прокси/модели.

## Примеры CLI

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-6
openclaw models list
```

См. также: [/gateway/configuration](../gateway/configuration.md) для полных примеров конфигурации.

[CLI для моделей](./models.md)[Резервирование моделей](./model-failover.md)