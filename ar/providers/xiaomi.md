title: "إعداد وتكوين مزود Xiaomi MiMo لـ OpenClaw AI"
description: "تعلم كيفية إعداد وتكوين مزود Xiaomi MiMo في OpenClaw AI. استخدم نموذج mimo-v2-flash مع واجهات برمجة تطبيقات متوافقة مع Anthropic عبر مصادقة مفتاح API."
keywords: ["xiaomi mimo", "مزود openclaw", "mimo-v2-flash", "واجهة برمجة تطبيقات anthropic", "مصادقة مفتاح api", "تكوين النموذج", "إعداد openclaw", "واجهة برمجة تطبيقات xiaomi"]
---

  المزودون

  
# Xiaomi MiMo

Xiaomi MiMo هو منصة واجهات برمجة التطبيقات لنماذج **MiMo**. يوفر واجهات برمجة تطبيقات REST متوافقة مع تنسيقات OpenAI وAnthropic ويستخدم مفاتيح API للمصادقة. أنشئ مفتاح API الخاص بك في [وحدة تحكم Xiaomi MiMo](https://platform.xiaomimimo.com/#/console/api-keys). يستخدم OpenClaw المزود `xiaomi` مع مفتاح API لـ Xiaomi MiMo.

## نظرة عامة على النموذج

-   **mimo-v2-flash**: نافذة سياق 262144 رمزًا، متوافق مع واجهة برمجة تطبيقات رسائل Anthropic.
-   عنوان URL الأساسي: `https://api.xiaomimimo.com/anthropic`
-   التفويض: `Bearer $XIAOMI_API_KEY`

## الإعداد عبر سطر الأوامر

```bash
openclaw onboard --auth-choice xiaomi-api-key
# أو بشكل غير تفاعلي
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

## مقتطف التكوين

```json
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## ملاحظات

-   مرجع النموذج: `xiaomi/mimo-v2-flash`.
-   يتم حقن المزود تلقائيًا عند تعيين `XIAOMI_API_KEY` (أو عند وجود ملف تعريف مصادقة).
-   راجع [/concepts/model-providers](../concepts/model-providers.md) لقواعد المزودين.

[vLLM](./vllm.md)[Z.AI](./zai.md)