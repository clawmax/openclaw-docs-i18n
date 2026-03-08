title: "إعداد موفر Kilocode في OpenClaw: مفتاح API والنماذج"
description: "تعلّم كيفية إعداد موفر Kilocode في OpenClaw، والحصول على مفتاح API الخاص بك، واستخدام بوابة Kilo الموحدة للوصول إلى نماذج الذكاء الاصطناعي المتعددة."
keywords: ["kilocode", "kilo gateway", "مزودو openclaw", "واجهة برمجة تطبيقات موحدة", "توجيه النماذج", "إعداد مفتاح API", "claude sonnet", "gpt-5.2"]
---

  المزودون

  
# Kilocode

توفر بوابة Kilo **واجهة برمجة تطبيقات موحدة** تقوم بتوجيه الطلبات إلى العديد من النماذج خلف نقطة نهاية واحدة ومفتاح API واحد. وهي متوافقة مع OpenAI، لذا تعمل معظم حزم تطوير البرمجيات (SDK) الخاصة بـ OpenAI عن طريق تبديل عنوان URL الأساسي.

## الحصول على مفتاح API

1.  انتقل إلى [app.kilo.ai](https://app.kilo.ai)
2.  سجّل الدخول أو أنشئ حسابًا
3.  انتقل إلى "مفاتيح API" وقم بإنشاء مفتاح جديد

## الإعداد عبر سطر الأوامر (CLI)

```bash
openclaw onboard --kilocode-api-key <key>
```

أو قم بتعيين متغير البيئة:

```bash
export KILOCODE_API_KEY="<your-kilocode-api-key>" # pragma: allowlist secret
```

## مقتطف التكوين

```json
{
  env: { KILOCODE_API_KEY: "<your-kilocode-api-key>" }, // pragma: allowlist secret
  agents: {
    defaults: {
      model: { primary: "kilocode/kilo/auto" },
    },
  },
}
```

## النموذج الافتراضي

النموذج الافتراضي هو `kilocode/kilo/auto`، وهو نموذج توجيه ذكي يختار تلقائيًا أفضل نموذج أساسي بناءً على المهمة:

-   يتم توجيه مهام التخطيط، وتصحيح الأخطاء، والتنسيق إلى Claude Opus
-   يتم توجيه مهام كتابة واستكشاف التعليمات البرمجية إلى Claude Sonnet

## النماذج المتاحة

يكتشف OpenClaw النماذج المتاحة من بوابة Kilo ديناميكيًا عند بدء التشغيل. استخدم الأمر `/models kilocode` لرؤية القائمة الكاملة للنماذج المتاحة مع حسابك. يمكن استخدام أي نموذج متاح على البوابة مع البادئة `kilocode/`:

```
kilocode/kilo/auto              (الافتراضي - توجيه ذكي)
kilocode/anthropic/claude-sonnet-4
kilocode/openai/gpt-5.2
kilocode/google/gemini-3-pro-preview
...وغيرها الكثير
```

## ملاحظات

-   مراجع النماذج هي `kilocode/<model-id>` (على سبيل المثال، `kilocode/anthropic/claude-sonnet-4`).
-   النموذج الافتراضي: `kilocode/kilo/auto`
-   عنوان URL الأساسي: `https://api.kilo.ai/api/gateway/`
-   لمزيد من خيارات النماذج/المزودين، راجع [/concepts/model-providers](../concepts/model-providers.md).
-   تستخدم بوابة Kilo رمز Bearer مع مفتاح API الخاص بك في الخلفية.

[Hugging Face (الاستدلال)](./huggingface.md)[Litellm](./litellm.md)