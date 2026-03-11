

  أدوات مدمجة

  
# أدوات الويب

يُشحن OpenClaw بأداتين خفيفتين للويب:

-   `web_search` — ابحث على الويب باستخدام واجهة برمجة تطبيقات Perplexity Search أو Brave Search API أو Gemini مع تأريض Google Search أو Grok أو Kimi.
-   `web_fetch` — جلب HTTP + استخراج مقروء (HTML → تنسيق تخفيض السعر/نص).

هذه **ليست** أتمتة للمتصفح. للمواقع التي تعتمد بشكل كبير على JavaScript أو عمليات تسجيل الدخول، استخدم [أداة المتصفح](./browser.md).

## كيفية عملها

-   `web_search` تستدعي مزودك المُكوّن وتُرجع النتائج.
-   يتم تخزين النتائج مؤقتًا حسب الاستعلام لمدة 15 دقيقة (قابلة للتكوين).
-   `web_fetch` تقوم بجلب HTTP GET عادي وتستخرج محتوى مقروءًا (HTML → تنسيق تخفيض السعر/نص). إنها **لا** تنفذ JavaScript.
-   `web_fetch` مفعلة افتراضيًا (ما لم يتم تعطيلها صراحةً).

راجع [إعداد Perplexity Search](../perplexity.md) و [إعداد Brave Search](../brave-search.md) للحصول على تفاصيل خاصة بالمزود.

## اختيار مزود بحث

| المزود | الإيجابيات | السلبيات | مفتاح واجهة برمجة التطبيقات |
| --- | --- | --- | --- |
| **واجهة برمجة تطبيقات Perplexity Search** | نتائج سريعة ومنظمة؛ مرشحات النطاق واللغة والمنطقة والحداثة؛ استخراج المحتوى | — | `PERPLEXITY_API_KEY` |
| **واجهة برمجة تطبيقات Brave Search** | نتائج سريعة ومنظمة | خيارات تصفية أقل؛ تنطبق شروط استخدام الذكاء الاصطناعي | `BRAVE_API_KEY` |
| **Gemini** | تأريض Google Search، مُركّب بواسطة الذكاء الاصطناعي | يتطلب مفتاح واجهة برمجة تطبيقات Gemini | `GEMINI_API_KEY` |
| **Grok** | ردود xAI المؤرّضة على الويب | يتطلب مفتاح واجهة برمجة تطبيقات xAI | `XAI_API_KEY` |
| **Kimi** | قدرة بحث Moonshot على الويب | يتطلب مفتاح واجهة برمجة تطبيقات Moonshot | `KIMI_API_KEY` / `MOONSHOT_API_KEY` |

### الكشف التلقائي

إذا لم يتم تعيين `provider` صراحةً، يكتشف OpenClaw تلقائيًا المزود الذي يجب استخدامه بناءً على مفاتيح واجهة برمجة التطبيقات المتاحة، ويتحقق بهذا الترتيب:

1.  **Brave** — متغير البيئة `BRAVE_API_KEY` أو تكوين `tools.web.search.apiKey`
2.  **Gemini** — متغير البيئة `GEMINI_API_KEY` أو تكوين `tools.web.search.gemini.apiKey`
3.  **Kimi** — متغير البيئة `KIMI_API_KEY` / `MOONSHOT_API_KEY` أو تكوين `tools.web.search.kimi.apiKey`
4.  **Perplexity** — متغير البيئة `PERPLEXITY_API_KEY` أو تكوين `tools.web.search.perplexity.apiKey`
5.  **Grok** — متغير البيئة `XAI_API_KEY` أو تكوين `tools.web.search.grok.apiKey`

إذا لم يتم العثور على أي مفاتيح، فإنه يتراجع إلى Brave (ستحصل على خطأ مفقود للمفتاح يطالبك بتكوين واحد).

## إعداد البحث على الويب

استخدم `openclaw configure --section web` لإعداد مفتاح واجهة برمجة التطبيقات الخاصة بك واختيار مزود.

### Perplexity Search

1.  أنشئ حساب Perplexity في [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api)
2.  أنشئ مفتاح واجهة برمجة تطبيقات في لوحة التحكم
3.  شغّل `openclaw configure --section web` لتخزين المفتاح في التكوين، أو عيّن `PERPLEXITY_API_KEY` في بيئتك.

راجع [وثائق واجهة برمجة تطبيقات Perplexity Search](https://docs.perplexity.ai/guides/search-quickstart) لمزيد من التفاصيل.

### Brave Search

1.  أنشئ حساب واجهة برمجة تطبيقات Brave Search في [brave.com/search/api](https://brave.com/search/api/)
2.  في لوحة التحكم، اختر خطة **Data for Search** (وليس "Data for AI") وأنشئ مفتاح واجهة برمجة تطبيقات.
3.  شغّل `openclaw configure --section web` لتخزين المفتاح في التكوين (موصى به)، أو عيّن `BRAVE_API_KEY` في بيئتك.

تقدم Brave خططًا مدفوعة؛ تحقق من بوابة واجهة برمجة تطبيقات Brave للاطلاع على الحدود الحالية والتسعير.

### أين يتم تخزين المفتاح

**عبر التكوين (موصى به):** شغّل `openclaw configure --section web`. يقوم بتخزين المفتاح تحت `tools.web.search.perplexity.apiKey` أو `tools.web.search.apiKey`. **عبر البيئة:** عيّن `PERPLEXITY_API_KEY` أو `BRAVE_API_KEY` في بيئة عملية Gateway. لتثبيت بوابة، ضعه في `~/.openclaw/.env` (أو بيئة الخدمة الخاصة بك). راجع [متغيرات البيئة](../help/faq.md#how-does-openclaw-load-environment-variables).

### أمثلة التكوين

**Perplexity Search:**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...", // اختياري إذا تم تعيين PERPLEXITY_API_KEY
        },
      },
    },
  },
}
```

**Brave Search:**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "brave",
        apiKey: "YOUR_BRAVE_API_KEY", // اختياري إذا تم تعيين BRAVE_API_KEY // pragma: allowlist secret
      },
    },
  },
}
```

## استخدام Gemini (تأريض Google Search)

تدعم نماذج Gemini [تأريض Google Search](https://ai.google.dev/gemini-api/docs/grounding) المدمج، والذي يُرجع إجابات مُركّبة بواسطة الذكاء الاصطناعي مدعومة بنتائج بحث Google المباشرة مع استشهادات.

### الحصول على مفتاح واجهة برمجة تطبيقات Gemini

1.  انتقل إلى [Google AI Studio](https://aistudio.google.com/apikey)
2.  أنشئ مفتاح واجهة برمجة تطبيقات
3.  عيّن `GEMINI_API_KEY` في بيئة Gateway، أو كوّن `tools.web.search.gemini.apiKey`

### إعداد بحث Gemini

```json
{
  tools: {
    web: {
      search: {
        provider: "gemini",
        gemini: {
          // مفتاح واجهة برمجة التطبيقات (اختياري إذا تم تعيين GEMINI_API_KEY)
          apiKey: "AIza...",
          // النموذج (الافتراضي هو "gemini-2.5-flash")
          model: "gemini-2.5-flash",
        },
      },
    },
  },
}
```

**بديل البيئة:** عيّن `GEMINI_API_KEY` في بيئة Gateway. لتثبيت بوابة، ضعه في `~/.openclaw/.env`.

### ملاحظات

-   يتم حل عناوين URL الاستشهاد من تأريض Gemini تلقائيًا من عناوين URL إعادة التوجيه الخاصة بـ Google إلى عناوين URL مباشرة.
-   يستخدم حل إعادة التوجيه مسار حارس SSRF (HEAD + فحوصات إعادة التوجيه + التحقق من http/https) قبل إرجاع عنوان URL الاستشهاد النهائي.
-   يستخدم حل إعادة التوجيه إعدادات SSRF الصارمة الافتراضية، لذلك يتم حظر عمليات إعادة التوجيه إلى الأهداف الخاصة/الداخلية.
-   النموذج الافتراضي (`gemini-2.5-flash`) سريع وفعال من حيث التكلفة. يمكن استخدام أي نموذج Gemini يدعم التأريض.

## web\_search

ابحث على الويب باستخدام المزود الذي قمت بتكوينه.

### المتطلبات

-   يجب ألا تكون `tools.web.search.enabled` معينة إلى `false` (الافتراضي: مفعل)
-   مفتاح واجهة برمجة تطبيقات للمزود الذي اخترته:
    -   **Brave**: `BRAVE_API_KEY` أو `tools.web.search.apiKey`
    -   **Perplexity**: `PERPLEXITY_API_KEY` أو `tools.web.search.perplexity.apiKey`
    -   **Gemini**: `GEMINI_API_KEY` أو `tools.web.search.gemini.apiKey`
    -   **Grok**: `XAI_API_KEY` أو `tools.web.search.grok.apiKey`
    -   **Kimi**: `KIMI_API_KEY` أو `MOONSHOT_API_KEY` أو `tools.web.search.kimi.apiKey`

### التكوين

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // اختياري إذا تم تعيين BRAVE_API_KEY
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### معلمات الأداة

جميع المعلمات تعمل مع كل من Brave و Perplexity ما لم يُذكر خلاف ذلك.

| المعلمة | الوصف |
| --- | --- |
| `query` | استعلام البحث (مطلوب) |
| `count` | عدد النتائج المراد إرجاعها (1-10، الافتراضي: 5) |
| `country` | رمز البلد ISO مكون من حرفين (مثال: "US"، "DE") |
| `language` | رمز اللغة ISO 639-1 (مثال: "en"، "de") |
| `freshness` | مرشح الوقت: `day`، `week`، `month`، أو `year` |
| `date_after` | النتائج بعد هذا التاريخ (YYYY-MM-DD) |
| `date_before` | النتائج قبل هذا التاريخ (YYYY-MM-DD) |
| `ui_lang` | رمز لغة واجهة المستخدم (Brave فقط) |
| `domain_filter` | مصفوفة القائمة البيضاء/القائمة السوداء للنطاقات (Perplexity فقط) |
| `max_tokens` | ميزانية المحتوى الإجمالية، الافتراضي 25000 (Perplexity فقط) |
| `max_tokens_per_page` | حد الرموز لكل صفحة، الافتراضي 2048 (Perplexity فقط) |

**أمثلة:**

```
// بحث محدد لألمانيا
await web_search({
  query: "TV online schauen",
  country: "DE",
  language: "de",
});

// نتائج حديثة (الأسبوع الماضي)
await web_search({
  query: "TMBG interview",
  freshness: "week",
});

// بحث بنطاق تاريخ
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});

// تصفية النطاقات (Perplexity فقط)
await web_search({
  query: "climate research",
  domain_filter: ["nature.com", "science.org", ".edu"],
});

// استبعاد النطاقات (Perplexity فقط)
await web_search({
  query: "product reviews",
  domain_filter: ["-reddit.com", "-pinterest.com"],
});

// استخراج محتوى أكثر (Perplexity فقط)
await web_search({
  query: "detailed AI research",
  max_tokens: 50000,
  max_tokens_per_page: 4096,
});
```

## web\_fetch

اجلب عنوان URL واستخرج محتوى مقروءًا.

### متطلبات web\_fetch

-   يجب ألا تكون `tools.web.fetch.enabled` معينة إلى `false` (الافتراضي: مفعل)
-   تراجع Firecrawl اختياري: عيّن `tools.web.fetch.firecrawl.apiKey` أو `FIRECRAWL_API_KEY`.

### تكوين web\_fetch

```json
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // اختياري إذا تم تعيين FIRECRAWL_API_KEY
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // مللي ثانية (يوم واحد)
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### معلمات أداة web\_fetch

-   `url` (مطلوب، http/https فقط)
-   `extractMode` (`markdown` | `text`)
-   `maxChars` (اقطع الصفحات الطويلة)

ملاحظات:

-   تستخدم `web_fetch` Readability (استخراج المحتوى الرئيسي) أولاً، ثم Firecrawl (إذا تم تكوينه). إذا فشل كلاهما، تُرجع الأداة خطأ.
-   طلبات Firecrawl تستخدم وضع تجاوز البوت وتخزن النتائج مؤقتًا افتراضيًا.
-   تُرسل `web_fetch` وكيل مستخدم يشبه Chrome و `Accept-Language` افتراضيًا؛ غيّر `userAgent` إذا لزم الأمر.
-   تمنع `web_fetch` أسماء المضيفين الخاصة/الداخلية وتعيد فحص عمليات إعادة التوجيه (اقصرها بـ `maxRedirects`).
-   يتم تحديد `maxChars` بـ `tools.web.fetch.maxCharsCap`.
-   تحد `web_fetch` حجم نص الاستجابة الذي تم تنزيله إلى `tools.web.fetch.maxResponseBytes` قبل التحليل؛ يتم اقتطاع الردود كبيرة الحجم وتتضمن تحذيرًا.
-   `web_fetch` هي استخراج بأفضل جهد؛ بعض المواقع ستحتاج إلى أداة المتصفح.
-   راجع [Firecrawl](./firecrawl.md) للحصول على تفاصيل إعداد المفتاح والخدمة.
-   يتم تخزين الردود مؤقتًا (15 دقيقة افتراضيًا) لتقليل عمليات الجلب المتكررة.
-   إذا كنت تستخدم ملفات تعريف/قوائم بيضاء للأدوات، أضف `web_search`/`web_fetch` أو `group:web`.
-   إذا كان مفتاح واجهة برمجة التطبيقات مفقودًا، تُرجع `web_search` تلميح إعداد قصير مع رابط للوثائق.

[مستويات التفكير](./thinking.md)[المتصفح (يديره OpenClaw)](./browser.md)