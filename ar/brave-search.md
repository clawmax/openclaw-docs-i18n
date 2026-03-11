

  أدوات مدمجة

  
# Brave Search

يدعم OpenClaw خدمة Brave Search كمزود بحث ويب لأداة `web_search`.

## الحصول على مفتاح API

1.  أنشئ حسابًا لواجهة برمجة تطبيقات Brave Search على [https://brave.com/search/api/](https://brave.com/search/api/)
2.  في لوحة التحكم، اختر خطة **Data for Search** وقم بتوليد مفتاح API.
3.  قم بتخزين المفتاح في ملف الإعدادات (مُوصى به) أو اضبط متغير البيئة `BRAVE_API_KEY` في Gateway.

## مثال على التكوين

```json
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

## معلمات الأداة

| المعلمة | الوصف |
| --- | --- |
| `query` | استعلام البحث (مطلوب) |
| `count` | عدد النتائج المراد إرجاعها (1-10، الافتراضي: 5) |
| `country` | رمز الدولة المكون من حرفين وفقًا لمعيار ISO (مثال: "US"، "DE") |
| `language` | رمز اللغة ISO 639-1 لنتائج البحث (مثال: "en"، "de"، "fr") |
| `ui_lang` | رمز اللغة ISO لعناصر واجهة المستخدم |
| `freshness` | عامل تصفية زمني: `day` (24 ساعة)، `week`، `month`، أو `year` |
| `date_after` | النتائج المنشورة بعد هذا التاريخ فقط (YYYY-MM-DD) |
| `date_before` | النتائج المنشورة قبل هذا التاريخ فقط (YYYY-MM-DD) |

**أمثلة:**

```javascript
// بحث محدد بالدولة واللغة
await web_search({
  query: "renewable energy",
  country: "DE",
  language: "de",
});

// نتائج حديثة (خلال الأسبوع الماضي)
await web_search({
  query: "AI news",
  freshness: "week",
});

// بحث ضمن نطاق تاريخي
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});
```

## ملاحظات

-   خطة **Data for AI** **غير** متوافقة مع أداة `web_search`.
-   تقدم Brave خططًا مدفوعة؛ تحقق من بوابة واجهة برمجة تطبيقات Brave للاطلاع على الحدود الحالية.
-   تشمل شروط Brave قيودًا على بعض الاستخدامات المتعلقة بالذكاء الاصطناعي لنتائج البحث. راجع شروط خدمة Brave وتأكد من أن استخدامك المقصود متوافق معها. للأسئلة القانونية، استشر مستشارك القانوني.
-   يتم تخزين النتائج مؤقتًا لمدة 15 دقيقة افتراضيًا (قابلة للتعديل عبر `cacheTtlMinutes`).

راجع [أدوات الويب](./tools/web.md) للحصول على التكوين الكامل لأداة web_search.

[أداة apply\_patch](./tools/apply-patch.md)[Perplexity Sonar](./perplexity.md)

---