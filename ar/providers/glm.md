title: "كيفية استخدام نماذج GLM في OpenClaw عبر موفر Z.AI"
description: "تعلم كيفية إعداد وتكوين نماذج GLM مثل glm-5 في OpenClaw باستخدام موفر Z.AI. يتضمن أمثلة للإعداد عبر سطر الأوامر والتكوين."
keywords: ["نماذج glm", "موفر openclaw zai", "إعداد glm-5", "مفتاح api لـ zai", "تكوين openclaw", "عائلة نموذج glm", "منصة zai", "موفرو openclaw"]
---

  الموفرون

  
# نماذج GLM

GLM هي **عائلة نماذج** (وليست شركة) متاحة عبر منصة Z.AI. في OpenClaw، يتم الوصول إلى نماذج GLM عبر موفر `zai` ومعرفات النماذج مثل `zai/glm-5`.

## الإعداد عبر سطر الأوامر

```bash
openclaw onboard --auth-choice zai-api-key
```

## مقتطف التكوين

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## ملاحظات

-   إصدارات GLM وتوافرها قد تتغير؛ تحقق من وثائق Z.AI للحصول على أحدث المعلومات.
-   تتضمن أمثلة معرفات النماذج `glm-5`، و `glm-4.7`، و `glm-4.6`.
-   للحصول على تفاصيل الموفر، انظر [/providers/zai](./zai.md).

[Litellm](./litellm.md)[MiniMax](./minimax.md)

---