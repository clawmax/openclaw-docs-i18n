title: "تكوين نموذج MiniMax M2.5 الذكاء الاصطناعي في وثائق OpenClaw"
description: "تعلم كيفية إعداد وتكوين نماذج MiniMax M2.5 و M2.5 Highspeed الذكاء الاصطناعي في OpenClaw عبر OAuth، أو مفتاح API، أو الاستدلال المحلي. يتضمن أمثلة إعداد واستكشاف الأخطاء وإصلاحها."
keywords: ["minimax m2.5", "تكوين openclaw", "مزود نموذج الذكاء الاصطناعي", "minimax highspeed", "متوافق مع anthropic api", "خطة البرمجة", "الاستدلال المحلي lm studio", "إعداد النموذج الاحتياطي"]
---

  المزودون

  
# MiniMax

MiniMax هي شركة ذكاء اصطناعي تبني عائلة النماذج **M2/M2.5**. الإصدار الحالي المركّز على البرمجة هو **MiniMax M2.5** (23 ديسمبر 2025)، المبني للمهام المعقدة في العالم الحقيقي. المصدر: [ملاحظة إصدار MiniMax M2.5](https://www.minimax.io/news/minimax-m25)

## نظرة عامة على النموذج (M2.5)

تسلط MiniMax الضوء على هذه التحسينات في M2.5:

-   **برمجة متعددة اللغات** أقوى (Rust، Java، Go، C++، Kotlin، Objective-C، TS/JS).
-   جودة إخراج أفضل **لتطوير الويب/التطبيقات** والجمالية (بما في ذلك تطبيقات الجوال الأصلية).
-   تحسين التعامل مع **التعليمات المركبة** لسير عمل نمط المكاتب، بناءً على التفكير المتداخل والتنفيذ المتكامل للقيود.
-   **ردود أكثر إيجازًا** مع استخدام أقل للرموز المميزة وحلقات تكرار أسرع.
-   توافق أقوى مع **أطر الأدوات/الوكلاء** وإدارة السياق (Claude Code، Droid/Factory AI، Cline، Kilo Code، Roo Code، BlackBox).
-   مخرجات **حوار وكتابة تقنية** عالية الجودة.

## MiniMax M2.5 مقابل MiniMax M2.5 Highspeed

-   **السرعة:** `MiniMax-M2.5-highspeed` هي الطبقة السريعة الرسمية في وثائق MiniMax.
-   **التكلفة:** تسعير MiniMax يسرد نفس تكلفة الإدخال وتكلفة إخراج أعلى للـ highspeed.
-   **التوافق:** لا يزال OpenClaw يقبل تكوينات `MiniMax-M2.5-Lightning` القديمة، لكن يُفضل استخدام `MiniMax-M2.5-highspeed` للإعداد الجديد.

## اختر طريقة إعداد

### MiniMax OAuth (خطة البرمجة) — موصى به

**الأفضل لـ:** الإعداد السريع مع خطة البرمجة MiniMax عبر OAuth، لا حاجة لمفتاح API. قم بتمكين البرنامج المساعد OAuth المدمج والمصادقة:

```bash
openclaw plugins enable minimax-portal-auth  # تخطى إذا كان محملاً بالفعل.
openclaw gateway restart  # أعد التشغيل إذا كان البوابة قيد التشغيل بالفعل
openclaw onboard --auth-choice minimax-portal
```

سيُطلب منك تحديد نقطة نهاية:

-   **Global** - المستخدمون الدوليون (`api.minimax.io`)
-   **CN** - المستخدمون في الصين (`api.minimaxi.com`)

راجع [ملف README لبرنامج MiniMax OAuth المساعد](https://github.com/openclaw/openclaw/tree/main/extensions/minimax-portal-auth) للتفاصيل.

### MiniMax M2.5 (مفتاح API)

**الأفضل لـ:** MiniMax المستضاف مع واجهة برمجة تطبيقات متوافقة مع Anthropic. التكوين عبر سطر الأوامر:

-   نفّذ `openclaw configure`
-   اختر **Model/auth**
-   اختر **MiniMax M2.5**

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.5" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.5",
            name: "MiniMax M2.5",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
          {
            id: "MiniMax-M2.5-highspeed",
            name: "MiniMax M2.5 Highspeed",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### MiniMax M2.5 كنموذج احتياطي (مثال)

**الأفضل لـ:** الاحتفاظ بأقوى نموذج من الجيل الأخير كأساسي، والتراجع إلى MiniMax M2.5. المثال أدناه يستخدم Opus كنموذج أساسي ملموس؛ استبدله بنموذجك الأساسي المفضل من الجيل الأخير.

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "primary" },
        "minimax/MiniMax-M2.5": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.5"],
      },
    },
  },
}
```

### اختياري: محليًا عبر LM Studio (يدوي)

**الأفضل لـ:** الاستدلال المحلي مع LM Studio. لقد رأينا نتائج قوية مع MiniMax M2.5 على أجهزة قوية (مثل سطح مكتب/خادم) باستخدام الخادم المحلي لـ LM Studio. قم بالتكوين يدويًا عبر `openclaw.json`:

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
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

## التكوين عبر openclaw configure

استخدم معالج التكوين التفاعلي لتعيين MiniMax دون تحرير JSON:

1.  نفّذ `openclaw configure`.
2.  اختر **Model/auth**.
3.  اختر **MiniMax M2.5**.
4.  اختر نموذجك الافتراضي عند المطالبة.

## خيارات التكوين

-   `models.providers.minimax.baseUrl`: يُفضل `https://api.minimax.io/anthropic` (متوافق مع Anthropic)؛ `https://api.minimax.io/v1` اختياري للحزم المتوافقة مع OpenAI.
-   `models.providers.minimax.api`: يُفضل `anthropic-messages`؛ `openai-completions` اختياري للحزم المتوافقة مع OpenAI.
-   `models.providers.minimax.apiKey`: مفتاح واجهة برمجة تطبيقات MiniMax (`MINIMAX_API_KEY`).
-   `models.providers.minimax.models`: عرّف `id`، `name`، `reasoning`، `contextWindow`، `maxTokens`، `cost`.
-   `agents.defaults.models`: أطلق أسماء مستعارة للنماذج التي تريدها في القائمة المسموح بها.
-   `models.mode`: احتفظ بـ `merge` إذا كنت تريد إضافة MiniMax بجانب النماذج المدمجة.

## ملاحظات

-   مراجع النماذج هي `minimax/`.
-   معرفات النماذج الموصى بها: `MiniMax-M2.5` و `MiniMax-M2.5-highspeed`.
-   واجهة برمجة تطبيقات استخدام خطة البرمجة: `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (يتطلب مفتاح خطة برمجة).
-   قم بتحديث قيم التسعير في `models.json` إذا كنت بحاجة إلى تتبع التكلفة الدقيقة.
-   رابط الإحالة لخطة البرمجة MiniMax (خصم 10٪): [https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link](https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link)
-   راجع [/concepts/model-providers](../concepts/model-providers.md) لقواعد المزودين.
-   استخدم `openclaw models list` و `openclaw models set minimax/MiniMax-M2.5` للتبديل.

## استكشاف الأخطاء وإصلاحها

### "نموذج غير معروف: minimax/MiniMax-M2.5"

هذا يعني عادةً أن **مزود MiniMax غير مهيأ** (لا يوجد إدخال مزود ولم يتم العثور على ملف تعريف مصادقة MiniMax / مفتاح بيئة). الإصلاح لهذا الاكتشاف موجود في **2026.1.12** (غير منشور وقت الكتابة). أصلح عن طريق:

-   الترقية إلى **2026.1.12** (أو التشغيل من المصدر `main`)، ثم إعادة تشغيل البوابة.
-   تشغيل `openclaw configure` واختيار **MiniMax M2.5**، أو
-   إضافة كتلة `models.providers.minimax` يدويًا، أو
-   تعيين `MINIMAX_API_KEY` (أو ملف تعريف مصادقة MiniMax) حتى يمكن حقن المزود.

تأكد من أن معرف النموذج **حساس لحالة الأحرف**:

-   `minimax/MiniMax-M2.5`
-   `minimax/MiniMax-M2.5-highspeed`
-   `minimax/MiniMax-M2.5-Lightning` (قديم)

ثم تحقق مرة أخرى باستخدام:

```bash
openclaw models list
```

[نماذج GLM](./glm.md)[Moonshot AI](./moonshot.md)