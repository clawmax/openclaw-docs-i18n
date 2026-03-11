

  المزودون

  
# Deepgram

Deepgram هو واجهة برمجة تطبيقات لتحويل الكلام إلى نص. في OpenClaw، يُستخدم لـ **تحويل الصوت الوارد/ملاحظات الصوت إلى نص** عبر `tools.media.audio`. عند التمكين، يقوم OpenClaw بتحميل ملف الصوت إلى Deepgram وحقن النص المُحول في مسار الرد (`{{Transcript}}` + كتلة `[Audio]`). هذه العملية **ليست بثًا مباشرًا**؛ فهي تستخدم نقطة نهاية التحويل المسجل مسبقًا. الموقع: [https://deepgram.com](https://deepgram.com)  
الوثائق: [https://developers.deepgram.com](https://developers.deepgram.com)

## البدء السريع

1.  عيّن مفتاح API الخاص بك:

```
DEEPGRAM_API_KEY=dg_...
```

2.  فعّل المزود:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## الخيارات

-   `model`: معرف نموذج Deepgram (الافتراضي: `nova-3`)
-   `language`: تلميح اللغة (اختياري)
-   `tools.media.audio.providerOptions.deepgram.detect_language`: تمكين اكتشاف اللغة (اختياري)
-   `tools.media.audio.providerOptions.deepgram.punctuate`: تمكين علامات الترقيم (اختياري)
-   `tools.media.audio.providerOptions.deepgram.smart_format`: تمكين التنسيق الذكي (اختياري)

مثال مع لغة:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3", language: "en" }],
      },
    },
  },
}
```

مثال مع خيارات Deepgram:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true,
          },
        },
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## ملاحظات

-   يتبع المصادقة ترتيب مصادقة المزود القياسي؛ `DEEPGRAM_API_KEY` هو المسار الأبسط.
-   يمكن تجاوز نقاط النهاية أو الرؤوس باستخدام `tools.media.audio.baseUrl` و `tools.media.audio.headers` عند استخدام وكيل.
-   يتبع الناتج نفس قواعد الصوت الخاصة بالمزودين الآخرين (حدود الحجم، المهلات، حقن النص المُحول).

[وكيل Claude Max API](./claude-max-api-proxy.md)[GitHub Copilot](./github-copilot.md)