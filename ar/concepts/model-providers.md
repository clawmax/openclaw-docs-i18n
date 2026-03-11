

  التكوين

  
# موفرو النماذج

تغطي هذه الصفحة **موفري LLM/النماذج** (وليس قنوات الدردشة مثل WhatsApp/Telegram). لقواعد اختيار النموذج، راجع [/concepts/models](./models.md).

## قواعد سريعة

-   تستخدم مراجع النماذج `provider/model` (مثال: `opencode/claude-opus-4-6`).
-   إذا قمت بتعيين `agents.defaults.models`، فإنه يصبح قائمة السماح.
-   أدوات CLI المساعدة: `openclaw onboard`, `openclaw models list`, `openclaw models set <provider/model>`.

## تدوير مفتاح API

-   يدعم التدوير العام للمزودين للمزودين المحددين.
-   قم بتكوين مفاتيح متعددة عبر:
    -   `OPENCLAW_LIVE__KEY` (تجاوز مباشر واحد، أعلى أولوية)
    -   `_API_KEYS` (قائمة مفصولة بفاصلة أو فاصلة منقوطة)
    -   `_API_KEY` (المفتاح الأساسي)
    -   `_API_KEY_*` (قائمة مرقمة، مثال: `_API_KEY_1`)
-   لمزودي Google، يتم تضمين `GOOGLE_API_KEY` أيضًا كخيار احتياطي.
-   يحافظ ترتيب اختيار المفتاح على الأولوية ويزيل القيم المكررة.
-   تتم إعادة محاولة الطلبات باستخدام المفتاح التالي فقط عند استجابات تحديد المعدل (على سبيل المثال `429`, `rate_limit`, `quota`, `resource exhausted`).
-   حالات الفشل غير المرتبطة بتحديد المعدل تفشل فورًا؛ لا تتم محاولة تدوير المفتاح.
-   عندما تفشل جميع المفاتيح المرشحة، يتم إرجاع الخطأ النهائي من المحاولة الأخيرة.

## المزودون المدمجون (كتالوج pi‑ai)

يأتي OpenClaw مع كتالوج pi‑ai. لا تتطلب هذه المزودين **أي** تكوين `models.providers`؛ ما عليك سوى تعيين المصادقة واختيار نموذج.

### OpenAI

-   المزود: `openai`
-   المصادقة: `OPENAI_API_KEY`
-   التدوير الاختياري: `OPENAI_API_KEYS`, `OPENAI_API_KEY_1`, `OPENAI_API_KEY_2`، بالإضافة إلى `OPENCLAW_LIVE_OPENAI_KEY` (تجاوز واحد)
-   نموذج مثال: `openai/gpt-5.1-codex`
-   CLI: `openclaw onboard --auth-choice openai-api-key`
-   النقل الافتراضي هو `auto` (WebSocket أولاً، ثم SSE كاحتياطي)
-   تجاوز لكل نموذج عبر `agents.defaults.models["openai/"].params.transport` (`"sse"`, `"websocket"`, أو `"auto"`)
-   تسخين WebSocket لاستجابات OpenAI مفعل افتراضيًا عبر `params.openaiWsWarmup` (`true`/`false`)

```json
{
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

### Anthropic

-   المزود: `anthropic`
-   المصادقة: `ANTHROPIC_API_KEY` أو `claude setup-token`
-   التدوير الاختياري: `ANTHROPIC_API_KEYS`, `ANTHROPIC_API_KEY_1`, `ANTHROPIC_API_KEY_2`، بالإضافة إلى `OPENCLAW_LIVE_ANTHROPIC_KEY` (تجاوز واحد)
-   نموذج مثال: `anthropic/claude-opus-4-6`
-   CLI: `openclaw onboard --auth-choice token` (الصق setup-token) أو `openclaw models auth paste-token --provider anthropic`
-   ملاحظة سياسة: دعم setup-token هو توافق تقني؛ قامت Anthropic بحظر بعض استخدامات الاشتراك خارج Claude Code في الماضي. تحقق من شروط Anthropic الحالية وقرر بناءً على تحملك للمخاطر.
-   التوصية: مصادقة مفتاح Anthropic API هي المسار الأكثر أمانًا والمُوصى به مقارنة بمصادقة setup-token للاشتراك.

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

### OpenAI Code (Codex)

-   المزود: `openai-codex`
-   المصادقة: OAuth (ChatGPT)
-   نموذج مثال: `openai-codex/gpt-5.3-codex`
-   CLI: `openclaw onboard --auth-choice openai-codex` أو `openclaw models auth login --provider openai-codex`
-   النقل الافتراضي هو `auto` (WebSocket أولاً، ثم SSE كاحتياطي)
-   تجاوز لكل نموذج عبر `agents.defaults.models["openai-codex/"].params.transport` (`"sse"`, `"websocket"`, أو `"auto"`)
-   ملاحظة سياسة: OAuth لـ OpenAI Codex مدعوم صراحةً للأدوات/سير العمل الخارجية مثل OpenClaw.

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

### OpenCode Zen

-   المزود: `opencode`
-   المصادقة: `OPENCODE_API_KEY` (أو `OPENCODE_ZEN_API_KEY`)
-   نموذج مثال: `opencode/claude-opus-4-6`
-   CLI: `openclaw onboard --auth-choice opencode-zen`

```json
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

### Google Gemini (مفتاح API)

-   المزود: `google`
-   المصادقة: `GEMINI_API_KEY`
-   التدوير الاختياري: `GEMINI_API_KEYS`, `GEMINI_API_KEY_1`, `GEMINI_API_KEY_2`, `GOOGLE_API_KEY` كخيار احتياطي، و `OPENCLAW_LIVE_GEMINI_KEY` (تجاوز واحد)
-   نموذج مثال: `google/gemini-3-pro-preview`
-   CLI: `openclaw onboard --auth-choice gemini-api-key`

### Google Vertex و Antigravity و Gemini CLI

-   المزودون: `google-vertex`, `google-antigravity`, `google-gemini-cli`
-   المصادقة: يستخدم Vertex gcloud ADC؛ يستخدم Antigravity/Gemini CLI تدفقات المصادقة الخاصة بهما على التوالي
-   تحذير: OAuth لـ Antigravity و Gemini CLI في OpenClaw هي تكاملات غير رسمية. أبلغ بعض المستخدمين عن قيود على حسابات Google بعد استخدام عملاء تابعين لجهات خارجية. راجع شروط Google واستخدم حسابًا غير حاسم إذا اخترت المتابعة.
-   OAuth لـ Antigravity يتم شحنه كإضافة مدمجة (`google-antigravity-auth`، معطلة افتراضيًا).
    -   التمكين: `openclaw plugins enable google-antigravity-auth`
    -   تسجيل الدخول: `openclaw models auth login --provider google-antigravity --set-default`
-   OAuth لـ Gemini CLI يتم شحنه كإضافة مدمجة (`google-gemini-cli-auth`، معطلة افتراضيًا).
    -   التمكين: `openclaw plugins enable google-gemini-cli-auth`
    -   تسجيل الدخول: `openclaw models auth login --provider google-gemini-cli --set-default`
    -   ملاحظة: لا تقوم **بصق** معرف عميل أو سر في `openclaw.json`. يقوم تدفق تسجيل الدخول عبر CLI بتخزين الرموز في ملفات تعريف المصادقة على مضيف البوابة.

### Z.AI (GLM)

-   المزود: `zai`
-   المصادقة: `ZAI_API_KEY`
-   نموذج مثال: `zai/glm-5`
-   CLI: `openclaw onboard --auth-choice zai-api-key`
    -   الأسماء المستعارة: `z.ai/*` و `z-ai/*` يتم تسويتها إلى `zai/*`

### Vercel AI Gateway

-   المزود: `vercel-ai-gateway`
-   المصادقة: `AI_GATEWAY_API_KEY`
-   نموذج مثال: `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

### Kilo Gateway

-   المزود: `kilocode`
-   المصادقة: `KILOCODE_API_KEY`
-   نموذج مثال: `kilocode/anthropic/claude-opus-4.6`
-   CLI: `openclaw onboard --kilocode-api-key `
-   عنوان URL الأساسي: `https://api.kilo.ai/api/gateway/`
-   يتضمن الكتالوج المدمج الموسع GLM-5 Free و MiniMax M2.5 Free و GPT-5.2 و Gemini 3 Pro Preview و Gemini 3 Flash Preview و Grok Code Fast 1 و Kimi K2.5.

راجع [/providers/kilocode](../providers/kilocode.md) للحصول على تفاصيل الإعداد.

### مزودون مدمجون آخرون

-   OpenRouter: `openrouter` (`OPENROUTER_API_KEY`)
-   نموذج مثال: `openrouter/anthropic/claude-sonnet-4-5`
-   Kilo Gateway: `kilocode` (`KILOCODE_API_KEY`)
-   نموذج مثال: `kilocode/anthropic/claude-opus-4.6`
-   xAI: `xai` (`XAI_API_KEY`)
-   Mistral: `mistral` (`MISTRAL_API_KEY`)
-   نموذج مثال: `mistral/mistral-large-latest`
-   CLI: `openclaw onboard --auth-choice mistral-api-key`
-   Groq: `groq` (`GROQ_API_KEY`)
-   Cerebras: `cerebras` (`CEREBRAS_API_KEY`)
    -   نماذج GLM على Cerebras تستخدم المعرفات `zai-glm-4.7` و `zai-glm-4.6`.
    -   عنوان URL الأساسي المتوافق مع OpenAI: `https://api.cerebras.ai/v1`.
-   GitHub Copilot: `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)
-   Hugging Face Inference: `huggingface` (`HUGGINGFACE_HUB_TOKEN` أو `HF_TOKEN`) — موجه متوافق مع OpenAI؛ نموذج مثال: `huggingface/deepseek-ai/DeepSeek-R1`؛ CLI: `openclaw onboard --auth-choice huggingface-api-key`. راجع [Hugging Face (Inference)](../providers/huggingface.md).

## المزودون عبر models.providers (مخصص/عنوان URL أساسي)

استخدم `models.providers` (أو `models.json`) لإضافة مزودين **مخصصين** أو وكلاء متوافقين مع OpenAI/Anthropic.

### Moonshot AI (Kimi)

يستخدم Moonshot نقاط نهاية متوافقة مع OpenAI، لذا قم بتكوينه كمزود مخصص:

-   المزود: `moonshot`
-   المصادقة: `MOONSHOT_API_KEY`
-   نموذج مثال: `moonshot/kimi-k2.5`

معرفات نموذج Kimi K2:

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

يستخدم Kimi Coding نقطة نهاية Moonshot AI المتوافقة مع Anthropic:

-   المزود: `kimi-coding`
-   المصادقة: `KIMI_API_KEY`
-   نموذج مثال: `kimi-coding/k2p5`

```json
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-coding/k2p5" } },
  },
}
```

### Qwen OAuth (الطبقة المجانية)

يوفر Qwen وصول OAuth إلى Qwen Coder + Vision عبر تدفق رمز الجهاز. قم بتمكين الإضافة المدمجة، ثم سجل الدخول:

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

مراجع النماذج:

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

راجع [/providers/qwen](../providers/qwen.md) للحصول على تفاصيل الإعداد والملاحظات.

### Volcano Engine (Doubao)

يوفر Volcano Engine (火山引擎) الوصول إلى Doubao ونماذج أخرى في الصين.

-   المزود: `volcengine` (البرمجة: `volcengine-plan`)
-   المصادقة: `VOLCANO_ENGINE_API_KEY`
-   نموذج مثال: `volcengine/doubao-seed-1-8-251228`
-   CLI: `openclaw onboard --auth-choice volcengine-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "volcengine/doubao-seed-1-8-251228" } },
  },
}
```

النماذج المتاحة:

-   `volcengine/doubao-seed-1-8-251228` (Doubao Seed 1.8)
-   `volcengine/doubao-seed-code-preview-251028`
-   `volcengine/kimi-k2-5-260127` (Kimi K2.5)
-   `volcengine/glm-4-7-251222` (GLM 4.7)
-   `volcengine/deepseek-v3-2-251201` (DeepSeek V3.2 128K)

نماذج البرمجة (`volcengine-plan`):

-   `volcengine-plan/ark-code-latest`
-   `volcengine-plan/doubao-seed-code`
-   `volcengine-plan/kimi-k2.5`
-   `volcengine-plan/kimi-k2-thinking`
-   `volcengine-plan/glm-4.7`

### BytePlus (دولي)

يوفر BytePlus ARK الوصول إلى نفس النماذج مثل Volcano Engine للمستخدمين الدوليين.

-   المزود: `byteplus` (البرمجة: `byteplus-plan`)
-   المصادقة: `BYTEPLUS_API_KEY`
-   نموذج مثال: `byteplus/seed-1-8-251228`
-   CLI: `openclaw onboard --auth-choice byteplus-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "byteplus/seed-1-8-251228" } },
  },
}
```

النماذج المتاحة:

-   `byteplus/seed-1-8-251228` (Seed 1.8)
-   `byteplus/kimi-k2-5-260127` (Kimi K2.5)
-   `byteplus/glm-4-7-251222` (GLM 4.7)

نماذج البرمجة (`byteplus-plan`):

-   `byteplus-plan/ark-code-latest`
-   `byteplus-plan/doubao-seed-code`
-   `byteplus-plan/kimi-k2.5`
-   `byteplus-plan/kimi-k2-thinking`
-   `byteplus-plan/glm-4.7`

### Synthetic

يوفر Synthetic نماذج متوافقة مع Anthropic خلف مزود `synthetic`:

-   المزود: `synthetic`
-   المصادقة: `SYNTHETIC_API_KEY`
-   نموذج مثال: `synthetic/hf:MiniMaxAI/MiniMax-M2.5`
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

يتم تكوين MiniMax عبر `models.providers` لأنه يستخدم نقاط نهاية مخصصة:

-   MiniMax (متوافق مع Anthropic): `--auth-choice minimax-api`
-   المصادقة: `MINIMAX_API_KEY`

راجع [/providers/minimax](../providers/minimax.md) للحصول على تفاصيل الإعداد وخيارات النماذج ومقتطفات التكوين.

### Ollama

Ollama هو وقت تشغيل LLM محلي يوفر واجهة برمجة تطبيقات متوافقة مع OpenAI:

-   المزود: `ollama`
-   المصادقة: غير مطلوبة (خادم محلي)
-   نموذج مثال: `ollama/llama3.3`
-   التثبيت: [https://ollama.ai](https://ollama.ai)

```bash
# قم بتثبيت Ollama، ثم اسحب نموذجًا:
ollama pull llama3.3
```

```json
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } },
  },
}
```

يتم اكتشاف Ollama تلقائيًا عند التشغيل محليًا على `http://127.0.0.1:11434/v1`. راجع [/providers/ollama](../providers/ollama.md) للحصول على توصيات النماذج والتكوين المخصص.

### vLLM

vLLM هو خادم متوافق مع OpenAI محلي (أو مستضاف ذاتيًا):

-   المزود: `vllm`
-   المصادقة: اختيارية (تعتمد على خادمك)
-   عنوان URL الأساسي الافتراضي: `http://127.0.0.1:8000/v1`

للاكتشاف التلقائي محليًا (أي قيمة تعمل إذا كان خادمك لا يفرض المصادقة):

```bash
export VLLM_API_KEY="vllm-local"
```

ثم قم بتعيين نموذج (استبدل بأحد المعرفات التي يتم إرجاعها بواسطة `/v1/models`):

```json
{
  agents: {
    defaults: { model: { primary: "vllm/your-model-id" } },
  },
}
```

راجع [/providers/vllm](../providers/vllm.md) للحصول على التفاصيل.

### الوكلاء المحليون (LM Studio، vLLM، LiteLLM، إلخ.)

مثال (متوافق مع OpenAI):

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

ملاحظات:

-   للمزودين المخصصين، `reasoning`, `input`, `cost`, `contextWindow`, و `maxTokens` اختيارية. عند حذفها، يستخدم OpenClaw القيم الافتراضية التالية:
    -   `reasoning: false`
    -   `input: ["text"]`
    -   `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
    -   `contextWindow: 200000`
    -   `maxTokens: 8192`
-   موصى به: قم بتعيين قيم صريحة تتطابق مع حدود الوكيل/النموذج الخاص بك.

## أمثلة CLI

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-6
openclaw models list
```

راجع أيضًا: [/gateway/configuration](../gateway/configuration.md) للحصول على أمثلة تكوين كاملة.

[CLI النماذج](./models.md)[التبديل الاحتياطي للنموذج](./model-failover.md)