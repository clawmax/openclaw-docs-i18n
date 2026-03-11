

  المزودون

  
# LiteLLM

[LiteLLM](https://litellm.ai) هو بوابة LLM مفتوحة المصدر توفر واجهة برمجة تطبيقات موحدة لأكثر من 100 مزود نموذج. قم بتوجيه OpenClaw عبر LiteLLM للحصول على تتبع مركزي للتكاليف، والتسجيل، ومرونة تبديل الخلفيات دون تغيير إعدادات OpenClaw.

## لماذا تستخدم LiteLLM مع OpenClaw؟

-   **تتبع التكاليف** — شاهد بالضبط ما ينفقه OpenClaw عبر جميع النماذج
-   **توجيه النماذج** — التبديل بين Claude و GPT-4 و Gemini و Bedrock دون تغييرات في الإعدادات
-   **مفاتيح افتراضية** — أنشئ مفاتيح بحدود إنفاق لـ OpenClaw
-   **التسجيل** — سجلات كاملة للطلبات والاستجابات لتصحيح الأخطاء
-   **البدائل الاحتياطية** — تبديل تلقائي في حالة تعطل مزودك الأساسي

## البدء السريع

### عبر عملية الإعداد

```bash
openclaw onboard --auth-choice litellm-api-key
```

### الإعداد اليدوي

1.  ابدأ تشغيل وكيل LiteLLM:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2.  وجه OpenClaw إلى LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

هذا كل شيء. OpenClaw الآن يتم توجيهه عبر LiteLLM.

## التكوين

### متغيرات البيئة

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### ملف التكوين

```json
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## المفاتيح الافتراضية

أنشئ مفتاحًا مخصصًا لـ OpenClaw بحدود إنفاق:

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

استخدم المفتاح المُنشأ كـ `LITELLM_API_KEY`.

## توجيه النماذج

يمكن لـ LiteLLM توجيه طلبات النموذج إلى خلفيات مختلفة. قم بالتكوين في ملف `config.yaml` الخاص بـ LiteLLM:

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

يستمر OpenClaw في طلب `claude-opus-4-6` — بينما يتولى LiteLLM عملية التوجيه.

## عرض الاستخدام

تحقق من لوحة تحكم LiteLLM أو واجهة برمجة التطبيقات الخاصة به:

```bash
# معلومات المفتاح
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# سجلات الإنفاق
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## ملاحظات

-   يعمل LiteLLM افتراضيًا على `http://localhost:4000`
-   يتصل OpenClaw عبر نقطة النهاية المتوافقة مع OpenAI `/v1/chat/completions`
-   جميع ميزات OpenClaw تعمل عبر LiteLLM — بدون قيود

## انظر أيضًا

-   [وثائق LiteLLM](https://docs.litellm.ai)
-   [مزودو النماذج](../concepts/model-providers.md)

[Kilocode](./kilocode.md)[نماذج GLM](./glm.md)