

  المزودون

  
# Venice AI

**Venice** هو إعدادنا البارز في Venice للاستدلال الذي يضع الخصوصية أولاً مع وصول مجهول اختياري إلى النماذج الاحتكارية. يوفر Venice AI استدلال ذكاء اصطناعي يركز على الخصوصية مع دعم النماذج غير الخاضعة للرقابة والوصول إلى النماذج الاحتكارية الرئيسية عبر وكيلهم المجهول. كل الاستدلالات خاصة افتراضيًا — لا تدريب على بياناتك، لا تسجيل.

## لماذا Venice في OpenClaw

-   **استدلال خاص** للنماذج مفتوحة المصدر (لا تسجيل).
-   **نماذج غير خاضعة للرقابة** عندما تحتاجها.
-   **وصول مجهول** إلى النماذج الاحتكارية (Opus/GPT/Gemini) عندما تكون الجودة مهمة.
-   نقاط نهاية متوافقة مع OpenAI `/v1`.

## أوضاع الخصوصية

يقدم Venice مستويين للخصوصية — فهم هذا مفتاح لاختيار نموذجك:

| الوضع | الوصف | النماذج |
| --- | --- | --- |
| **خاص** | خاص بالكامل. المطالبات/الردود **لا تُخزن أو تسجل أبدًا**. عابر. | Llama, Qwen, DeepSeek, Kimi, MiniMax, Venice Uncensored, إلخ. |
| **مجهول** | يتم تمريره عبر Venice مع تجريد البيانات الوصفية. مزود الخدمة الأساسي (OpenAI, Anthropic, Google, xAI) يرى طلبات مجهولة. | Claude, GPT, Gemini, Grok |

## الميزات

-   **يركز على الخصوصية**: اختر بين وضع "خاص" (خاص بالكامل) و "مجهول" (مُمرر)
-   **نماذج غير خاضعة للرقابة**: الوصول إلى نماذج بدون قيود محتوى
-   **وصول إلى نماذج رئيسية**: استخدم Claude وGPT وGemini وGrok عبر وكيل Venice المجهول
-   **واجهة برمجة تطبيقات متوافقة مع OpenAI**: نقاط نهاية قياسية `/v1` لسهولة التكامل
-   **البث**: ✅ مدعوم على جميع النماذج
-   **استدعاء الدوال**: ✅ مدعوم على نماذج مختارة (تحقق من إمكانيات النموذج)
-   **الرؤية**: ✅ مدعوم على النماذج ذات قدرة الرؤية
-   **لا حدود صارمة للمعدل**: قد يتم تطبيق تخفيف للاستخدام العادل للاستخدام المفرط

## الإعداد

### 1\. احصل على مفتاح واجهة برمجة التطبيقات

1.  سجل في [venice.ai](https://venice.ai)
2.  انتقل إلى **الإعدادات → مفاتيح واجهة برمجة التطبيقات → إنشاء مفتاح جديد**
3.  انسخ مفتاح واجهة برمجة التطبيقات الخاص بك (التنسيق: `vapi_xxxxxxxxxxxx`)

### 2\. تكوين OpenClaw

**الخيار أ: متغير البيئة**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**الخيار ب: الإعداد التفاعلي (موصى به)**

```bash
openclaw onboard --auth-choice venice-api-key
```

سيؤدي هذا إلى:

1.  المطالبة بمفتاح واجهة برمجة التطبيقات الخاص بك (أو استخدام `VENICE_API_KEY` الموجود)
2.  عرض جميع نماذج Venice المتاحة
3.  السماح لك باختيار النموذج الافتراضي الخاص بك
4.  تكوين المزود تلقائيًا

**الخيار ج: غير تفاعلي**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```

### 3\. التحقق من الإعداد

```bash
openclaw agent --model venice/kimi-k2-5 --message "Hello, are you working?"
```

## اختيار النموذج

بعد الإعداد، يعرض OpenClaw جميع نماذج Venice المتاحة. اختر بناءً على احتياجاتك:

-   **النموذج الافتراضي**: `venice/kimi-k2-5` للاستدلال الخاص القوي بالإضافة إلى الرؤية.
-   **خيار عالي الإمكانيات**: `venice/claude-opus-4-6` لأقوى مسار Venice مجهول.
-   **الخصوصية**: اختر النماذج "الخاصة" للاستدلال الخاص بالكامل.
-   **الإمكانية**: اختر النماذج "المجهولة" للوصول إلى Claude وGPT وGemini عبر وكيل Venice.

غير النموذج الافتراضي الخاص بك في أي وقت:

```bash
openclaw models set venice/kimi-k2-5
openclaw models set venice/claude-opus-4-6
```

اعرض جميع النماذج المتاحة:

```bash
openclaw models list | grep venice
```

## التكوين عبر openclaw configure

1.  شغّل `openclaw configure`
2.  اختر **النموذج/المصادقة**
3.  اختر **Venice AI**

## أي نموذج يجب أن أستخدم؟

| حالة الاستخدام | النموذج الموصى به | السبب |
| --- | --- | --- |
| **دردشة عامة (افتراضي)** | `kimi-k2-5` | استدلال خاص قوي بالإضافة إلى الرؤية |
| **أفضل جودة بشكل عام** | `claude-opus-4-6` | أقوى خيار Venice مجهول |
| **خصوصية + برمجة** | `qwen3-coder-480b-a35b-instruct` | نموذج برمجة خاص مع سياق كبير |
| **رؤية خاصة** | `kimi-k2-5` | دعم الرؤية دون مغادرة الوضع الخاص |
| **سريع + رخيص** | `qwen3-4b` | نموذج استدلال خفيف الوزن |
| **مهام خاصة معقدة** | `deepseek-v3.2` | استدلال قوي، لكن بدون دعم أدوات Venice |
| **غير خاضع للرقابة** | `venice-uncensored` | لا قيود على المحتوى |

## النماذج المتاحة (41 إجمالاً)

### نماذج خاصة (26) — خاصة بالكامل، لا تسجيل

| معرف النموذج | الاسم | السياق | الميزات |
| --- | --- | --- | --- |
| `kimi-k2-5` | Kimi K2.5 | 256k | افتراضي، استدلال، رؤية |
| `kimi-k2-thinking` | Kimi K2 Thinking | 256k | استدلال |
| `llama-3.3-70b` | Llama 3.3 70B | 128k | عام |
| `llama-3.2-3b` | Llama 3.2 3B | 128k | عام |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 128k | عام، أدوات معطلة |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 128k | استدلال |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 128k | عام |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 256k | برمجة |
| `qwen3-coder-480b-a35b-instruct-turbo` | Qwen3 Coder 480B Turbo | 256k | برمجة |
| `qwen3-5-35b-a3b` | Qwen3.5 35B A3B | 256k | استدلال، رؤية |
| `qwen3-next-80b` | Qwen3 Next 80B | 256k | عام |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B (رؤية) | 256k | رؤية |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | سريع، استدلال |
| `deepseek-v3.2` | DeepSeek V3.2 | 160k | استدلال، أدوات معطلة |
| `venice-uncensored` | Venice Uncensored (Dolphin-Mistral) | 32k | غير خاضع للرقابة، أدوات معطلة |
| `mistral-31-24b` | Venice Medium (Mistral) | 128k | رؤية |
| `google-gemma-3-27b-it` | Google Gemma 3 27B Instruct | 198k | رؤية |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 128k | عام |
| `nvidia-nemotron-3-nano-30b-a3b` | NVIDIA Nemotron 3 Nano 30B | 128k | عام |
| `olafangensan-glm-4.7-flash-heretic` | GLM 4.7 Flash Heretic | 128k | استدلال |
| `zai-org-glm-4.6` | GLM 4.6 | 198k | عام |
| `zai-org-glm-4.7` | GLM 4.7 | 198k | استدلال |
| `zai-org-glm-4.7-flash` | GLM 4.7 Flash | 128k | استدلال |
| `zai-org-glm-5` | GLM 5 | 198k | استدلال |
| `minimax-m21` | MiniMax M2.1 | 198k | استدلال |
| `minimax-m25` | MiniMax M2.5 | 198k | استدلال |

### نماذج مجهولة (15) — عبر وكيل Venice

| معرف النموذج | الاسم | السياق | الميزات |
| --- | --- | --- | --- |
| `claude-opus-4-6` | Claude Opus 4.6 (عبر Venice) | 1M | استدلال، رؤية |
| `claude-opus-4-5` | Claude Opus 4.5 (عبر Venice) | 198k | استدلال، رؤية |
| `claude-sonnet-4-6` | Claude Sonnet 4.6 (عبر Venice) | 1M | استدلال، رؤية |
| `claude-sonnet-4-5` | Claude Sonnet 4.5 (عبر Venice) | 198k | استدلال، رؤية |
| `openai-gpt-54` | GPT-5.4 (عبر Venice) | 1M | استدلال، رؤية |
| `openai-gpt-53-codex` | GPT-5.3 Codex (عبر Venice) | 400k | استدلال، رؤية، برمجة |
| `openai-gpt-52` | GPT-5.2 (عبر Venice) | 256k | استدلال |
| `openai-gpt-52-codex` | GPT-5.2 Codex (عبر Venice) | 256k | استدلال، رؤية، برمجة |
| `openai-gpt-4o-2024-11-20` | GPT-4o (عبر Venice) | 128k | رؤية |
| `openai-gpt-4o-mini-2024-07-18` | GPT-4o Mini (عبر Venice) | 128k | رؤية |
| `gemini-3-1-pro-preview` | Gemini 3.1 Pro (عبر Venice) | 1M | استدلال، رؤية |
| `gemini-3-pro-preview` | Gemini 3 Pro (عبر Venice) | 198k | استدلال، رؤية |
| `gemini-3-flash-preview` | Gemini 3 Flash (عبر Venice) | 256k | استدلال، رؤية |
| `grok-41-fast` | Grok 4.1 Fast (عبر Venice) | 1M | استدلال، رؤية |
| `grok-code-fast-1` | Grok Code Fast 1 (عبر Venice) | 256k | استدلال، برمجة |

## اكتشاف النماذج

يكتشف OpenClaw النماذج تلقائيًا من واجهة برمجة تطبيقات Venice عند تعيين `VENICE_API_KEY`. إذا كانت واجهة برمجة التطبيقات غير قابلة للوصول، فإنه يتراجع إلى كتالوج ثابت. نقطة النهاية `/models` عامة (لا حاجة للمصادقة للعرض)، لكن الاستدلال يتطلب مفتاح واجهة برمجة تطبيقات صالحًا.

## البث ودعم الأدوات

| الميزة | الدعم |
| --- | --- |
| **البث** | ✅ جميع النماذج |
| **استدعاء الدوال** | ✅ معظم النماذج (تحقق من `supportsFunctionCalling` في واجهة برمجة التطبيقات) |
| **الرؤية/الصور** | ✅ النماذج المميزة بـ "رؤية" |
| **وضع JSON** | ✅ مدعوم عبر `response_format` |

## التسعير

يستخدم Venice نظامًا قائمًا على الاعتمادات. تحقق من [venice.ai/pricing](https://venice.ai/pricing) للأسعار الحالية:

-   **نماذج خاصة**: عمومًا تكلفة أقل
-   **نماذج مجهولة**: مشابه لتسعير واجهة برمجة التطبيقات المباشرة + رسوم Venice صغيرة

## المقارنة: Venice مقابل واجهة برمجة التطبيقات المباشرة

| الجانب | Venice (مجهول) | واجهة برمجة التطبيقات المباشرة |
| --- | --- | --- |
| **الخصوصية** | تجريد البيانات الوصفية، مجهول | حسابك مرتبط |
| **زمن الاستجابة** | +10-50 مللي ثانية (وكيل) | مباشر |
| **الميزات** | معظم الميزات مدعومة | جميع الميزات |
| **الفواتير** | اعتمادات Venice | فواتير المزود |

## أمثلة الاستخدام

```bash
# استخدم النموذج الخاص الافتراضي
openclaw agent --model venice/kimi-k2-5 --message "Quick health check"

# استخدم Claude Opus عبر Venice (مجهول)
openclaw agent --model venice/claude-opus-4-6 --message "Summarize this task"

# استخدم نموذج غير خاضع للرقابة
openclaw agent --model venice/venice-uncensored --message "Draft options"

# استخدم نموذج رؤية مع صورة
openclaw agent --model venice/qwen3-vl-235b-a22b --message "Review attached image"

# استخدم نموذج برمجة
openclaw agent --model venice/qwen3-coder-480b-a35b-instruct --message "Refactor this function"
```

## استكشاف الأخطاء وإصلاحها

### مفتاح واجهة برمجة التطبيقات غير معترف به

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

تأكد من أن المفتاح يبدأ بـ `vapi_`.

### النموذج غير متاح

يتحديث كتالوج نماذج Venice ديناميكيًا. شغّل `openclaw models list` لرؤية النماذج المتاحة حاليًا. قد تكون بعض النماذج غير متاحة مؤقتًا.

### مشاكل الاتصال

توجد واجهة برمجة تطبيقات Venice على `https://api.venice.ai/api/v1`. تأكد من أن شبكتك تسمح باتصالات HTTPS.

## مثال ملف التكوين

```json
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/kimi-k2-5" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2-5",
            name: "Kimi K2.5",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

## روابط

-   [Venice AI](https://venice.ai)
-   [توثيق واجهة برمجة التطبيقات](https://docs.venice.ai)
-   [التسعير](https://venice.ai/pricing)
-   [الحالة](https://status.venice.ai)

[Vercel AI Gateway](./vercel-ai-gateway.md)[vLLM](./vllm.md)