title: "دليل إعداد وتكوين مزود vLLM لـ OpenClaw"
description: "تعلم كيفية ربط OpenClaw بـ vLLM لتقديم نماذج مفتوحة المصدر عبر واجهة برمجة تطبيقات متوافقة مع OpenAI. يتضمن بدء سريع، الاكتشاف التلقائي، والتكوين الصريح."
keywords: ["vllm", "واجهة برمجة تطبيقات متوافقة مع openai", "تقديم النماذج", "مزودي openclaw", "نموذج لغة محلي", "اكتشاف النماذج", "مكملات openai", "تكوين vllm"]
---

  المزودون

  
# vLLM

يمكن لـ vLLM تقديم نماذج مفتوحة المصدر (وبعض النماذج المخصصة) عبر **واجهة برمجة تطبيقات HTTP متوافقة مع OpenAI**. يمكن لـ OpenClaw الاتصال بـ vLLM باستخدام واجهة برمجة تطبيقات `openai-completions`. يمكن لـ OpenClaw أيضًا **اكتشاف** النماذج المتاحة من vLLM تلقائيًا عند الموافقة باستخدام `VLLM_API_KEY` (أي قيمة تعمل إذا لم يفرض خادمك المصادقة) ولم تقم بتعريف إدخال صريح `models.providers.vllm`.

## البدء السريع

1.  ابدأ vLLM مع خادم متوافق مع OpenAI.

يجب أن يعرض عنوان URL الأساسي نقاط النهاية `/v1` (مثل `/v1/models`، `/v1/chat/completions`). يعمل vLLM عادةً على:

-   `http://127.0.0.1:8000/v1`

2.  الموافقة (أي قيمة تعمل إذا لم يتم تكوين مصادقة):

```bash
export VLLM_API_KEY="vllm-local"
```

3.  اختر نموذجًا (استبدل بأحد معرفات النموذج الخاصة بـ vLLM لديك):

```json
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## اكتشاف النماذج (مزود ضمني)

عند تعيين `VLLM_API_KEY` (أو وجود ملف تعريف مصادقة) و**لم** تقم بتعريف `models.providers.vllm`، سيقوم OpenClaw بالاستعلام عن:

-   `GET http://127.0.0.1:8000/v1/models`

…وتحويل المعرفات المعادة إلى إدخالات نموذج. إذا قمت بتعيين `models.providers.vllm` صراحةً، يتم تخطي الاكتشاف التلقائي ويجب عليك تعريف النماذج يدويًا.

## التكوين الصريح (نماذج يدوية)

استخدم التكوين الصريح عندما:

-   يعمل vLLM على مضيف/منفذ مختلف.
-   تريد تثبيت قيم `contextWindow`/`maxTokens`.
-   يتطلب خادمك مفتاح واجهة برمجة تطبيقات حقيقي (أو تريد التحكم في الرؤوس).

```json
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "نموذج vLLM المحلي",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## استكشاف الأخطاء وإصلاحها

-   تحقق من إمكانية الوصول إلى الخادم:

```bash
curl http://127.0.0.1:8000/v1/models
```

-   إذا فشلت الطلبات بأخطاء مصادقة، قم بتعيين `VLLM_API_KEY` حقيقي يتطابق مع تكوين خادمك، أو قم بتكوين المزود صراحةً تحت `models.providers.vllm`.

[Venice AI](./venice.md)[Xiaomi MiMo](./xiaomi.md)