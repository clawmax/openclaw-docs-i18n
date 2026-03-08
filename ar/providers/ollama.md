title: "دليل إعداد وتكوين تكامل OpenClaw مع Ollama"
description: "تعلم كيفية دمج Ollama مع OpenClaw للاستدلال المحلي لنماذج اللغة الكبيرة. قم بتكوين الاكتشاف التلقائي واستدعاء الأدوات والبث المباشر باستخدام واجهة برمجة التطبيقات الأصلية."
keywords: ["ollama", "openclaw", "نموذج لغة كبير محلي", "مزودو النماذج", "استدعاء الأدوات", "تكامل واجهة برمجة التطبيقات", "توافق openai", "اكتشاف النماذج"]
---

  المزودون

  
# Ollama

Ollama هو بيئة تشغيل محلية لنماذج اللغة الكبيرة يسهل تشغيل النماذج مفتوحة المصدر على جهازك. يتكامل OpenClaw مع واجهة برمجة التطبيقات الأصلية لـ Ollama (`/api/chat`)، ويدعم البث المباشر واستدعاء الأدوات، ويمكنه **اكتشاف النماذج القادرة على استخدام الأدوات تلقائيًا** عندما توافق على ذلك عن طريق تعيين `OLLAMA_API_KEY` (أو ملف تعريف مصادقة) وعدم تعريف إدخال صريح لـ `models.providers.ollama`.

> **⚠️** **مستخدمو Ollama البعيد**: لا تستخدم عنوان URL المتوافق مع OpenAI (`/v1`) (`http://host:11434/v1`) مع OpenClaw. هذا يكسر استدعاء الأدوات وقد تخرج النماذج JSON الأدوات الخام كنص عادي. استخدم عنوان URL واجهة برمجة التطبيقات الأصلية لـ Ollama بدلاً من ذلك: `baseUrl: "http://host:11434"` (بدون `/v1`).

## البدء السريع

1.  قم بتثبيت Ollama: [https://ollama.ai](https://ollama.ai)
2.  اسحب نموذجًا:

```bash
ollama pull gpt-oss:20b
# أو
ollama pull llama3.3
# أو
ollama pull qwen2.5-coder:32b
# أو
ollama pull deepseek-r1:32b
```

3.  قم بتمكين Ollama لـ OpenClaw (أي قيمة تعمل؛ Ollama لا يتطلب مفتاحًا حقيقيًا):

```bash
# تعيين متغير البيئة
export OLLAMA_API_KEY="ollama-local"

# أو قم بالتكوين في ملف الإعدادات الخاص بك
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4.  استخدم نماذج Ollama:

```json
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## اكتشاف النماذج (مزود ضمني)

عندما تقوم بتعيين `OLLAMA_API_KEY` (أو ملف تعريف مصادقة) **ولا** تعرف `models.providers.ollama`، يكتشف OpenClaw النماذج من مثيل Ollama المحلي على `http://127.0.0.1:11434`:

-   يستعلم عن `/api/tags` و `/api/show`
-   يحتفظ فقط بالنماذج التي تبلغ عن قدرة `tools`
-   يضع علامة `reasoning` عندما يبلغ النموذج عن `thinking`
-   يقرأ `contextWindow` من `model_info[".context_length"]` عندما يكون متاحًا
-   يضبط `maxTokens` على 10× نافذة السياق
-   يضبط جميع التكاليف على `0`

هذا يتجنب الإدخالات اليدوية للنماذج مع الحفاظ على الفهرس متوافقًا مع إمكانيات Ollama. لمعرفة النماذج المتاحة:

```bash
ollama list
openclaw models list
```

لإضافة نموذج جديد، ما عليك سوى سحبه باستخدام Ollama:

```bash
ollama pull mistral
```

سيتم اكتشاف النموذج الجديد تلقائيًا ويكون متاحًا للاستخدام. إذا قمت بتعيين `models.providers.ollama` بشكل صريح، يتم تخطي الاكتشاف التلقائي ويجب عليك تعريف النماذج يدويًا (انظر أدناه).

## التكوين

### الإعداد الأساسي (اكتشاف ضمني)

أسهل طريقة لتمكين Ollama هي عبر متغير البيئة:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### الإعداد الصريح (نماذج يدوية)

استخدم التكوين الصريح عندما:

-   يعمل Ollama على مضيف/منفذ آخر.
-   تريد فرض نوافذ سياق أو قوائم نماذج محددة.
-   تريد تضمين نماذج لا تبلغ عن دعم الأدوات.

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434",
        apiKey: "ollama-local",
        api: "ollama",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

إذا تم تعيين `OLLAMA_API_KEY`، يمكنك حذف `apiKey` في إدخال المزود وسيقوم OpenClaw بتعبئته لفحوصات التوفر.

### عنوان URL أساسي مخصص (تكوين صريح)

إذا كان Ollama يعمل على مضيف أو منفذ مختلف (يتعطّل الاكتشاف التلقائي في التكوين الصريح، لذا قم بتعريف النماذج يدويًا):

```json
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434", // لا /v1 - استخدم عنوان URL واجهة برمجة التطبيقات الأصلية لـ Ollama
        api: "ollama", // اضبط بشكل صريح لضمان سلوك استدعاء الأدوات الأصلي
      },
    },
  },
}
```

> **⚠️** لا تضيف `/v1` إلى عنوان URL. مسار `/v1` يستخدم وضع التوافق مع OpenAI، حيث لا يكون استدعاء الأدوات موثوقًا. استخدم عنوان URL الأساسي لـ Ollama بدون لاحقة مسار.

### اختيار النموذج

بعد التكوين، تصبح جميع نماذج Ollama الخاصة بك متاحة:

```json
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## متقدم

### نماذج التفكير المنطقي

يضع OpenClaw علامة على النماذج بأنها قادرة على التفكير المنطقي عندما يبلغ Ollama عن `thinking` في `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

### تكاليف النموذج

Ollama مجاني ويعمل محليًا، لذا يتم ضبط جميع تكاليف النموذج على $0.

### تكوين البث المباشر

يستخدم تكامل OpenClaw مع Ollام **واجهة برمجة التطبيقات الأصلية لـ Ollama** (`/api/chat`) افتراضيًا، والتي تدعم البث المباشر واستدعاء الأدوات في وقت واحد بشكل كامل. لا حاجة لتكوين خاص.

#### وضع التوافق مع OpenAI القديم

> **⚠️** **استدعاء الأدوات غير موثوق في وضع التوافق مع OpenAI.** استخدم هذا الوضع فقط إذا كنت بحاجة إلى تنسيق OpenAI لبروكسي ولا تعتمد على سلوك استدعاء الأدوات الأصلي.

 إذا كنت بحاجة إلى استخدام نقطة نهاية التوافق مع OpenAI بدلاً من ذلك (على سبيل المثال، خلف بروكسي يدعم تنسيق OpenAI فقط)، قم بتعيين `api: "openai-completions"` بشكل صريح:

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: true, // الافتراضي: true
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

قد لا يدعم هذا الوضع البث المباشر + استدعاء الأدوات في وقت واحد. قد تحتاج إلى تعطيل البث باستخدام `params: { streaming: false }` في تكوين النموذج. عند استخدام `api: "openai-completions"` مع Ollama، يحقن OpenClaw `options.num_ctx` افتراضيًا حتى لا يتراجع Ollama بصمت إلى نافذة سياق 4096. إذا رفض البروكسي/الخادم العلوي حقول `options` غير المعروفة، قم بتعطيل هذا السلوك:

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: false,
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

### نوافذ السياق

بالنسبة للنماذج المكتشفة تلقائيًا، يستخدم OpenClaw نافذة السياق التي يبلغ عنها Ollama عندما تكون متاحة، وإلا فإنه يستخدم القيمة الافتراضية `8192`. يمكنك تجاوز `contextWindow` و `maxTokens` في تكوين المزود الصريح.

## استكشاف الأخطاء وإصلاحها

### لم يتم اكتشاف Ollama

تأكد من أن Ollama قيد التشغيل وأنك قمت بتعيين `OLLAMA_API_KEY` (أو ملف تعريف مصادقة)، وأنك **لم** تقم بتعريف إدخال صريح لـ `models.providers.ollama`:

```bash
ollama serve
```

وأن واجهة برمجة التطبيقات يمكن الوصول إليها:

```bash
curl http://localhost:11434/api/tags
```

### لا توجد نماذج متاحة

يكتشف OpenClaw تلقائيًا فقط النماذج التي تبلغ عن دعم الأدوات. إذا لم يكن نموذجك مدرجًا، فإما:

-   اسحب نموذجًا قادرًا على استخدام الأدوات، أو
-   عرّف النموذج بشكل صريح في `models.providers.ollama`.

لإضافة نماذج:

```bash
ollama list  # انظر ما هو مثبت
ollama pull gpt-oss:20b  # اسحب نموذجًا قادرًا على استخدام الأدوات
ollama pull llama3.3     # أو نموذج آخر
```

### تم رفض الاتصال

تحقق من أن Ollama يعمل على المنفذ الصحيح:

```bash
# تحقق مما إذا كان Ollama قيد التشغيل
ps aux | grep ollama

# أو أعد تشغيل Ollama
ollama serve
```

## انظر أيضًا

-   [مزودو النماذج](../concepts/model-providers.md) - نظرة عامة على جميع المزودين
-   [اختيار النموذج](../concepts/models.md) - كيفية اختيار النماذج
-   [التكوين](../gateway/configuration.md) - مرجع التكوين الكامل

[NVIDIA](./nvidia.md)[OpenAI](./openai.md)