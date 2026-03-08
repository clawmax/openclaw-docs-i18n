title: "إعداد موفر Synthetic وتكوين النموذج في OpenClaw"
description: "تعلّم كيفية إعداد وتكوين موفر Synthetic في OpenClaw. يغطي هذا الدليل إعداد مفتاح API، تكوين النموذج، والكتالوج الكامل للنماذج المتاحة."
keywords: ["موفر اصطناعي", "تكوين OpenClaw", "واجهة برمجة تطبيقات متوافقة مع Anthropic", "كتالوج النماذج", "مفتاح API الاصطناعي", "موفرو OpenClaw", "إعداد النموذج", "نماذج اصطناعية"]
---

  الموفرون

  
# Synthetic

يعرض Synthetic نقاط نهاية متوافقة مع Anthropic. يسجل OpenClaw هذا الموفر باسم `synthetic` ويستخدم واجهة برمجة تطبيقات الرسائل الخاصة بـ Anthropic.

## الإعداد السريع

1.  عيّن `SYNTHETIC_API_KEY` (أو شغّل المعالج أدناه).
2.  شغّل عملية التعريف:

```bash
openclaw onboard --auth-choice synthetic-api-key
```

النموذج الافتراضي مضبوط على:

```
synthetic/hf:MiniMaxAI/MiniMax-M2.5
```

## مثال التكوين

```json
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.5" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.5": { alias: "MiniMax M2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.5",
            name: "MiniMax M2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

ملاحظة: يضيف عميل Anthropic الخاص بـ OpenClaw `/v1` إلى عنوان URL الأساسي، لذا استخدم `https://api.synthetic.new/anthropic` (وليس `/anthropic/v1`). إذا غيّر Synthetic عنوان URL الأساسي الخاص به، فقم بتجاوز `models.providers.synthetic.baseUrl`.

## كتالوج النماذج

جميع النماذج أدناه تستخدم تكلفة `0` (إدخال/إخراج/ذاكرة تخزين مؤقت).

| معرف النموذج | نافذة السياق | الحد الأقصى للرموز | التفكير | الإدخال |
| --- | --- | --- | --- | --- |
| `hf:MiniMaxAI/MiniMax-M2.5` | 192000 | 65536 | false | نص |
| `hf:moonshotai/Kimi-K2-Thinking` | 256000 | 8192 | true | نص |
| `hf:zai-org/GLM-4.7` | 198000 | 128000 | false | نص |
| `hf:deepseek-ai/DeepSeek-R1-0528` | 128000 | 8192 | false | نص |
| `hf:deepseek-ai/DeepSeek-V3-0324` | 128000 | 8192 | false | نص |
| `hf:deepseek-ai/DeepSeek-V3.1` | 128000 | 8192 | false | نص |
| `hf:deepseek-ai/DeepSeek-V3.1-Terminus` | 128000 | 8192 | false | نص |
| `hf:deepseek-ai/DeepSeek-V3.2` | 159000 | 8192 | false | نص |
| `hf:meta-llama/Llama-3.3-70B-Instruct` | 128000 | 8192 | false | نص |
| `hf:meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | 524000 | 8192 | false | نص |
| `hf:moonshotai/Kimi-K2-Instruct-0905` | 256000 | 8192 | false | نص |
| `hf:openai/gpt-oss-120b` | 128000 | 8192 | false | نص |
| `hf:Qwen/Qwen3-235B-A22B-Instruct-2507` | 256000 | 8192 | false | نص |
| `hf:Qwen/Qwen3-Coder-480B-A35B-Instruct` | 256000 | 8192 | false | نص |
| `hf:Qwen/Qwen3-VL-235B-A22B-Instruct` | 250000 | 8192 | false | نص + صورة |
| `hf:zai-org/GLM-4.5` | 128000 | 128000 | false | نص |
| `hf:zai-org/GLM-4.6` | 198000 | 128000 | false | نص |
| `hf:deepseek-ai/DeepSeek-V3` | 128000 | 8192 | false | نص |
| `hf:Qwen/Qwen3-235B-A22B-Thinking-2507` | 256000 | 8192 | true | نص |

## ملاحظات

-   مراجع النماذج تستخدم `synthetic/`.
-   إذا قمت بتمكين قائمة السماح للنماذج (`agents.defaults.models`)، أضف كل نموذج تخطط لاستخدامه.
-   راجع [موفرو النماذج](../concepts/model-providers.md) للاطلاع على قواعد المزود.

[Qwen](./qwen.md)[Together](./together.md)

---