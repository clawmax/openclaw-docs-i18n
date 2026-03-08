title: "إعداد موفر OpenAI لمفتاح OpenClaw API وCodex"
description: "تعلم كيفية تكوين مفاتيح OpenAI API أو Codex OAuth في OpenClaw. قم بإعداد نماذج GPT، وإدارة النقل، وتمكين الضغط من جانب الخادم."
keywords: ["openai", "openclaw", "مفتاح api", "codex", "gpt-5.4", "oauth", "websocket", "responses api"]
---

  الموفرون

  
# OpenAI

يوفر OpenAI واجهات برمجة تطبيقات للمطورين لنماذج GPT. يدعم Codex **تسجيل دخول ChatGPT** للوصول عن طريق الاشتراك أو **تسجيل دخول بمفتاح API** للوصول القائم على الاستخدام. يتطلب Codex cloud تسجيل دخول ChatGPT. يدعم OpenAI صراحةً استخدام OAuth للاشتراك في الأدوات/سير العمل الخارجية مثل OpenClaw.

## الخيار أ: مفتاح OpenAI API (منصة OpenAI)

**الأفضل لـ:** الوصول المباشر لواجهة برمجة التطبيقات والفوترة القائمة على الاستخدام. احصل على مفتاح API الخاص بك من لوحة تحكم OpenAI.

### الإعداد عبر سطر الأوامر

```bash
openclaw onboard --auth-choice openai-api-key
# أو غير التفاعلي
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### مقتطف التكوين

```json
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.4" } } },
}
```

تدرج وثائق نموذج API الحالية من OpenAI `gpt-5.4` و `gpt-5.4-pro` للاستخدام المباشر لواجهة برمجة تطبيقات OpenAI. يقوم OpenClaw بتوجيه كليهما عبر مسار `openai/*` Responses.

## الخيار ب: اشتراك OpenAI Code (Codex)

**الأفضل لـ:** استخدام وصول اشتراك ChatGPT/Codex بدلاً من مفتاح API. يتطلب Codex cloud تسجيل دخول ChatGPT، بينما يدعم Codex CLI تسجيل دخول ChatGPT أو مفتاح API.

### الإعداد عبر سطر الأوامر (Codex OAuth)

```bash
# تشغيل Codex OAuth في المعالج
openclaw onboard --auth-choice openai-codex

# أو تشغيل OAuth مباشرة
openclaw models auth login --provider openai-codex
```

### مقتطف التكوين (اشتراك Codex)

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.4" } } },
}
```

تدرج وثائق Codex الحالية من OpenAI `gpt-5.4` كنموذج Codex الحالي. يقوم OpenClaw بتعيين ذلك إلى `openai-codex/gpt-5.4` لاستخدام OAuth الخاص بـ ChatGPT/Codex.

### النقل الافتراضي

يستخدم OpenClaw `pi-ai` للبث النموذجي. لكل من `openai/*` و `openai-codex/*`، يكون النقل الافتراضي هو `"auto"` (WebSocket أولاً، ثم التراجع إلى SSE). يمكنك تعيين `agents.defaults.models.<provider/model>.params.transport`:

-   `"sse"`: فرض SSE
-   `"websocket"`: فرض WebSocket
-   `"auto"`: تجربة WebSocket، ثم التراجع إلى SSE

لـ `openai/*` (Responses API)، يقوم OpenClaw أيضًا بتمكين الإحماء لـ WebSocket افتراضيًا (`openaiWsWarmup: true`) عند استخدام نقل WebSocket. الوثائق ذات الصلة من OpenAI:

-   [Realtime API مع WebSocket](https://platform.openai.com/docs/guides/realtime-websocket)
-   [استجابات API المتدفقة (SSE)](https://platform.openai.com/docs/guides/streaming-responses)

```json
{
  agents: {
    defaults: {
      model: { primary: "openai-codex/gpt-5.4" },
      models: {
        "openai-codex/gpt-5.4": {
          params: {
            transport: "auto",
          },
        },
      },
    },
  },
}
```

### إحماء OpenAI WebSocket

تصف وثائق OpenAI الإحماء على أنه اختياري. يقوم OpenClaw بتمكينه افتراضيًا لـ `openai/*` لتقليل زمن الوصول في الدور الأول عند استخدام نقل WebSocket.

### تعطيل الإحماء

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: false,
          },
        },
      },
    },
  },
}
```

### تمكين الإحماء صراحةً

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: true,
          },
        },
      },
    },
  },
}
```

### معالجة الأولوية في OpenAI

تكشف واجهة برمجة تطبيقات OpenAI عن معالجة الأولوية عبر `service_tier=priority`. في OpenClaw، قم بتعيين `agents.defaults.models["openai/"].params.serviceTier` لتمرير هذا الحقل عبر طلبات Responses المباشرة `openai/*`.

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            serviceTier: "priority",
          },
        },
      },
    },
  },
}
```

القيم المدعومة هي `auto`، `default`، `flex`، و `priority`.

### ضغط جانب الخادم لـ OpenAI Responses

لنماذج OpenAI Responses المباشرة (`openai/*` باستخدام `api: "openai-responses"` مع `baseUrl` على `api.openai.com`)، يقوم OpenClaw الآن بتمكين تلميحات حمولة الضغط من جانب الخادم لـ OpenAI تلقائيًا:

-   يجبر `store: true` (ما لم يحدد توافق النموذج `supportsStore: false`)
-   يحقن `context_management: [{ type: "compaction", compact_threshold: ... }]`

افتراضيًا، `compact_threshold` هو `70%` من `contextWindow` للنموذج (أو `80000` عندما لا يكون متاحًا).

### تمكين ضغط جانب الخادم صراحةً

استخدم هذا عندما تريد فرض حقن `context_management` على نماذج Responses المتوافقة (على سبيل المثال Azure OpenAI Responses):

```json
{
  agents: {
    defaults: {
      models: {
        "azure-openai-responses/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
          },
        },
      },
    },
  },
}
```

### التمكين مع حد مخصص

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
            responsesCompactThreshold: 120000,
          },
        },
      },
    },
  },
}
```

### تعطيل ضغط جانب الخادم

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: false,
          },
        },
      },
    },
  },
}
```

`responsesServerCompaction` يتحكم فقط في حقن `context_management`. لا تزال نماذج OpenAI Responses المباشرة تجبر `store: true` ما لم يحدد التوافق `supportsStore: false`.

## ملاحظات

-   تشير المراجع النموذجية دائمًا إلى `provider/model` (انظر [/concepts/models](../concepts/models.md)).
-   تفاصيل المصادقة + قواعد إعادة الاستخدام موجودة في [/concepts/oauth](../concepts/oauth.md).

[Ollama](./ollhma.md)[OpenCode Zen](./opencode.md)