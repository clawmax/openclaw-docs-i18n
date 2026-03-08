title: "دليل إعداد موفر Together AI والنماذج في OpenClaw"
description: "تعلّم كيفية تكوين موفر Together AI في OpenClaw، وتعيين مفتاح API الخاص بك، والوصول إلى نماذج مثل Llama وDeepSeek وKimi."
keywords: ["together ai", "موفر openclaw", "نموذج llama", "deepseek", "kimi ai", "واجهة برمجة تطبيقات متوافقة مع openai", "تكوين النموذج", "إعداد مفتاح api"]
---

  الموفرون

  
# Together

توفر [Together AI](https://together.ai) الوصول إلى نماذج مفتوحة المصدر الرائدة بما في ذلك Llama وDeepSeek وKimi والمزيد من خلال واجهة برمجة تطبيقات موحدة.

-   الموفر: `together`
-   المصادقة: `TOGETHER_API_KEY`
-   واجهة برمجة التطبيقات: متوافقة مع OpenAI

## البدء السريع

1.  عيّن مفتاح API (موصى به: احفظه للبوابة):

```bash
openclaw onboard --auth-choice together-api-key
```

2.  عيّن نموذجًا افتراضيًا:

```json
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## مثال غير تفاعلي

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

سيؤدي هذا إلى تعيين `together/moonshotai/Kimi-K2.5` كنموذج افتراضي.

## ملاحظة حول البيئة

إذا كانت البوابة تعمل كخدمة خلفية (launchd/systemd)، فتأكد من أن `TOGETHER_API_KEY` متاح لتلك العملية (على سبيل المثال، في `~/.openclaw/.env` أو عبر `env.shellEnv`).

## النماذج المتاحة

توفر Together AI الوصول إلى العديد من النماذج مفتوحة المصدر الشهيرة:

-   **GLM 4.7 Fp8** - النموذج الافتراضي مع نافذة سياق 200K
-   **Llama 3.3 70B Instruct Turbo** - اتباع تعليمات سريع وفعال
-   **Llama 4 Scout** - نموذج رؤية مع فهم الصور
-   **Llama 4 Maverick** - رؤية واستدلال متقدم
-   **DeepSeek V3.1** - نموذج برمجة واستدلال قوي
-   **DeepSeek R1** - نموذج استدلال متقدم
-   **Kimi K2 Instruct** - نموذج عالي الأداء مع نافذة سياق 262K

جميع النماذج تدعم إكمال المحادثة القياسي وهي متوافقة مع واجهة برمجة تطبيقات OpenAI.

[اصطناعي](./synthetic.md)[بوابة Vercel AI](./vercel-ai-gateway.md)