title: "تكوين Perplexity Sonar للبحث على الويب في OpenClaw"
description: "تعلّم كيفية إعداد واستخدام Perplexity Sonar للبحث على الويب في OpenClaw عبر API المباشر أو OpenRouter، بما في ذلك أمثلة التكوين وخيارات النماذج."
keywords: ["perplexity sonar", "البحث على الويب في openclaw", "واجهة برمجة تطبيقات perplexity", "openrouter", "أداة البحث على الويب", "sonar-pro", "تكوين API", "مزود البحث"]
---

  أدوات مدمجة

  
# Perplexity Sonar

يمكن لـ OpenClaw استخدام Perplexity Sonar لأداة `web_search`. يمكنك الاتصال عبر واجهة برمجة تطبيقات Perplexity المباشرة أو عبر OpenRouter.

## خيارات API

### Perplexity (مباشر)

-   عنوان URL الأساسي: [https://api.perplexity.ai](https://api.perplexity.ai)
-   متغير البيئة: `PERPLEXITY_API_KEY`

### OpenRouter (بديل)

-   عنوان URL الأساسي: [https://openrouter.ai/api/v1](https://openrouter.ai/api/v1)
-   متغير البيئة: `OPENROUTER_API_KEY`
-   يدعم الرصيد المسبق الدفع / العملات المشفرة.

## مثال التكوين

```json
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

## التبديل من Brave

```json
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
        },
      },
    },
  },
}
```

إذا تم تعيين كل من `PERPLEXITY_API_KEY` و `OPENROUTER_API_KEY`، قم بتعيين `tools.web.search.perplexity.baseUrl` (أو `tools.web.search.perplexity.apiKey`) لإزالة الغموض. إذا لم يتم تعيين عنوان URL أساسي، يختار OpenClaw افتراضيًا بناءً على مصدر مفتاح API:

-   `PERPLEXITY_API_KEY` أو `pplx-...` → Perplexity المباشر (`https://api.perplexity.ai`)
-   `OPENROUTER_API_KEY` أو `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
-   تنسيقات مفاتيح غير معروفة → OpenRouter (خيار آمن احتياطي)

## النماذج

-   `perplexity/sonar` — أسئلة وأجوبة سريعة مع بحث على الويب
-   `perplexity/sonar-pro` (الافتراضي) — استدلال متعدد الخطوات + بحث على الويب
-   `perplexity/sonar-reasoning-pro` — بحث عميق

راجع [أدوات الويب](./tools/web.md) للحصول على تكوين web\_search الكامل.

[Brave Search](./brave-search.md)[Diffs](./tools/diffs.md)