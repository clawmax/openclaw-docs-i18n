title: "دليل إعداد وتكوين موفر Z.AI في OpenClaw"
description: "تعلّم كيفية إعداد وتكوين موفر Z.AI في OpenClaw لاستخدام نماذج GLM عبر واجهة برمجة التطبيقات (API). يتضمن إعداد سطر الأوامر وأمثلة للتكوين."
keywords: ["z.ai", "openclaw", "نماذج glm", "موفر api", "مفتاح api", "بث الأدوات", "إعداد سطر الأوامر", "تكوين"]
---

  الموفرون

  
# Z.AI

Z.AI هو منصة واجهة برمجة التطبيقات (API) لنماذج **GLM**. يوفر واجهات REST API لنماذج GLM ويستخدم مفاتيح API للمصادقة. أنشئ مفتاح API الخاص بك من خلال وحدة تحكم Z.AI. يستخدم OpenClaw الموفر `zai` مع مفتاح API من Z.AI.

## الإعداد عبر سطر الأوامر

```bash
openclaw onboard --auth-choice zai-api-key
# أو بشكل غير تفاعلي
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## مقتطف التكوين

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## ملاحظات

-   نماذج GLM متاحة كـ `zai/` (مثال: `zai/glm-5`).
-   `tool_stream` مفعّل افتراضيًا لبث استدعاءات الأدوات (tool-call) في Z.AI. اضبط `agents.defaults.models["zai/"].params.tool_stream` على `false` لتعطيله.
-   راجع [/providers/glm](./glm.md) للحصول على نظرة عامة على عائلة النماذج.
-   يستخدم Z.AI مصادقة Bearer مع مفتاح API الخاص بك.

[Xiaomi MiMo](./xiaomi.md)

---