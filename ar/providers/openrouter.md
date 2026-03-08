title: "إعداد وتكوين موفر OpenRouter لـ OpenClaw AI"
description: "تعلم كيفية إعداد وتكوين موفر OpenRouter في OpenClaw AI. استخدم واجهة برمجة تطبيقات موحدة للوصول إلى نماذج ذكاء اصطناعي متعددة بمفتاح واجهة برمجة تطبيقات واحد ونقطة نهاية متوافقة مع OpenAI."
keywords: ["openrouter", "openclaw ai", "موفر api", "واجهة برمجة تطبيقات موحدة", "توجيه النموذج", "متوافق مع openai", "claude sonnet", "تكوين الذكاء الاصطناعي"]
---

  الموفرون

  
# OpenRouter

يوفر OpenRouter **واجهة برمجة تطبيقات موحدة** توجه الطلبات إلى العديد من النماذج خلف نقطة نهاية واحدة ومفتاح واجهة برمجة تطبيقات واحد. وهي متوافقة مع OpenAI، لذا تعمل معظم حزم تطوير برامج OpenAI عن طريق تبديل عنوان URL الأساسي.

## الإعداد عبر سطر الأوامر

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## مقتطف التكوين

```json
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## ملاحظات

-   مراجع النماذج تكون على الشكل `openrouter//`.
-   لمزيد من خيارات النماذج/الموفرين، راجع [/concepts/model-providers](../concepts/model-providers.md).
-   يستخدم OpenRouter رمز Bearer مع مفتاح واجهة برمجة التطبيقات الخاص بك في الخلفية.

[OpenCode Zen](./opencode.md)[Qianfan](./qianfan.md)