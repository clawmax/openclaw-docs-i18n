title: "إعداد النماذج المحلية لبوابة OpenClaw باستخدام LM Studio"
description: "تعلم كيفية إعداد وتكوين نماذج الذكاء الاصطناعي المحلية مثل MiniMax M2.5 مع LM Studio لبوابة OpenClaw، بما في ذلك التكوينات الهجينة وترتيبات الاحتياطي."
keywords: ["نماذج openclaw المحلية", "إعداد lm studio", "minimax m2.5", "وكيل محلي متوافق مع openai", "تكوين البوابة", "نشر الذكاء الاصطناعي المحلي", "الاحتياطي للنموذج الهجين", "vllm litellm"]
---

  البروتوكولات وواجهات برمجة التطبيقات

  
# النماذج المحلية

التشغيل المحلي ممكن، لكن OpenClaw تتوقع سياقًا كبيرًا + دفاعات قوية ضد حقن الأوامر. البطاقات الصغيرة تقطع السياق وتتسرب منها إجراءات الأمان. اهتم بالجودة العالية: **≥2 جهاز Mac Studio بكامل إمكانياته أو جهاز GPU مكافئ (~30 ألف دولار+)**. وحدة معالجة رسومية واحدة بسعة **24 جيجابايت** تعمل فقط مع الأوامر الأخف وبتأخير أعلى. استخدم **أكبر متغير للنموذج / الحجم الكامل الذي يمكنك تشغيله**؛ النماذج المكممة بشدة أو "الصغيرة" تزيد من خطر حقن الأوامر (انظر [الأمان](./security.md)).

## الموصى به: LM Studio + MiniMax M2.5 (واجهة Responses API، الحجم الكامل)

أفضل تكوين محلي حاليًا. قم بتحميل MiniMax M2.5 في LM Studio، وشغل الخادم المحلي (الافتراضي `http://127.0.0.1:1234`)، واستخدم واجهة Responses API للحفاظ على التفكير المنطقي منفصلًا عن النص النهائي.

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**قائمة التحقق للإعداد**

-   تثبيت LM Studio: [https://lmstudio.ai](https://lmstudio.ai)
-   في LM Studio، حمل **أكبر إصدار متاح من MiniMax M2.5** (تجنب المتغيرات "الصغيرة"/المكممة بشدة)، ابدأ الخادم، وتأكد من أن `http://127.0.0.1:1234/v1/models` يعرضه.
-   حافظ على تحميل النموذج؛ التحميل البارد يضيف تأخيرًا في البدء.
-   اضبط `contextWindow`/`maxTokens` إذا كان إصدار LM Studio الخاص بك مختلفًا.
-   بالنسبة لـ WhatsApp، التزم باستخدام واجهة Responses API بحيث يُرسل النص النهائي فقط.

حافظ على تكوين النماذج المستضافة حتى عند التشغيل المحلي؛ استخدم `models.mode: "merge"` لتبقى خيارات الاحتياطي متاحة.

### التكوين الهجين: أساسي مستضاف، احتياطي محلي

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.5-gs32", "anthropic/claude-opus-4-6"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.5-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### محلي أولاً مع شبكة أمان مستضافة

قم بتبديل ترتيب الأساسي والاحتياطي؛ حافظ على نفس كتلة `providers` و `models.mode: "merge"` حتى تتمكن من الرجوع إلى Sonnet أو Opus عندما يكون الجهاز المحلي معطلاً.

### الاستضافة الإقليمية / توجيه البيانات

-   توجد متغيرات مستضافة من MiniMax/Kimi/GLM أيضًا على OpenRouter مع نقاط نهاية محددة إقليميًا (مثل المستضافة في الولايات المتحدة). اختر المتغير الإقليمي هناك للحفاظ على حركة المرور ضمن الولاية القضائية التي اخترتها مع الاستمرار في استخدام `models.mode: "merge"` للاحتياطي من Anthropic/OpenAI.
-   يبقى التشغيل المحلي فقط هو أقوى مسار للخصوصية؛ التوجيه الإقليمي المستضاف هو الحل الوسط عندما تحتاج إلى ميزات المزود ولكنك تريد التحكم في تدفق البيانات.

## وكلاء محليون آخرون متوافقون مع OpenAI

تعمل vLLM، LiteLLM، OAI-proxy، أو البوابات المخصصة إذا كانت تعرض نقطة نهاية على نمط `/v1` من OpenAI. استبدل كتلة المزود أعلاه بنقطة النهاية ومعرف النموذج الخاص بك:

```json
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

حافظ على `models.mode: "merge"` لتبقى النماذج المستضافة متاحة كخيارات احتياطية.

## استكشاف الأخطاء وإصلاحها

-   هل يمكن للبوابة الوصول إلى الوكيل؟ `curl http://127.0.0.1:1234/v1/models`.
-   هل تم إلغاء تحميل النموذج في LM Studio؟ أعد التحميل؛ البدء البارد سبب شائع "للتعلق".
-   أخطاء في السياق؟ قلل `contextWindow` أو ارفع الحد الخاص بخادمك.
-   الأمان: النماذج المحلية تتخطى المرشحات من جانب المزود؛ حافظ على وكلاء ضيقي النطاق وضغط السياق قيد التشغيل للحد من نطاق تأثير حقن الأوامر.

[واجهات سطر الأوامر الخلفية](./cli-backends.md)[نموذج الشبكة](./network-model.md)