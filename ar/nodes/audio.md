

  الوسائط والأجهزة

  
# الصوت والمذكرات الصوتية

## ما الذي يعمل

-   **فهم الوسائط (الصوت)**: إذا كان فهم الصوت مفعلاً (أو تم اكتشافه تلقائياً)، فإن OpenClaw:
    1.  يحدد موقع أول مرفق صوتي (مسار محلي أو URL) ويقوم بتنزيله إذا لزم الأمر.
    2.  يفرض `maxBytes` قبل الإرسال إلى كل إدخال نموذج.
    3.  يشغل أول إدخال نموذج مؤهل بالترتيب (مزود أو CLI).
    4.  إذا فشل أو تم تخطيه (حجم/مهلة)، يحاول الإدخال التالي.
    5.  عند النجاح، يستبدل `Body` بكتلة `[Audio]` ويضبط `{{Transcript}}`.
-   **تحليل الأوامر**: عندما ينجح التحويل إلى نص، يتم تعيين `CommandBody`/`RawBody` إلى النص المحول بحيث تعمل أوامر الشرطة المائلة.
-   **التسجيل التفصيلي**: في الوضع `--verbose`، نسجل وقت تشغيل التحويل إلى نص ووقت استبدال النص الأساسي.

## الكشف التلقائي (الافتراضي)

إذا **لم تقم بتكوين النماذج** وكان `tools.media.audio.enabled` **لم** يتم ضبطه على `false`، فإن OpenClaw يكتشف تلقائياً بهذا الترتيب ويتوقف عند أول خيار يعمل:

1.  **أوامر CLI المحلية** (إذا كانت مثبتة)
    -   `sherpa-onnx-offline` (يتطلب `SHERPA_ONNX_MODEL_DIR` مع encoder/decoder/joiner/tokens)
    -   `whisper-cli` (من `whisper-cpp`؛ يستخدم `WHISPER_CPP_MODEL` أو النموذج الصغير المرفق)
    -   `whisper` (Python CLI؛ يقوم بتنزيل النماذج تلقائياً)
2.  **Gemini CLI** (`gemini`) باستخدام `read_many_files`
3.  **مفاتيح المزود** (OpenAI → Groq → Deepgram → Google)

لتعطيل الكشف التلقائي، اضبط `tools.media.audio.enabled: false`. للتخصيص، اضبط `tools.media.audio.models`. ملاحظة: اكتشاف الملفات الثنائية هو أفضل جهد عبر أنظمة macOS/Linux/Windows؛ تأكد من أن CLI موجود في `PATH` (نقوم بتوسيع `~`)، أو اضبط نموذج CLI صريحاً مع مسار أمر كامل.

## أمثلة التكوين

### مزود + بديل CLI (OpenAI + Whisper CLI)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### مزود فقط مع بوابة النطاق

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### مزود فقط (Deepgram)

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

### مزود فقط (Mistral Voxtral)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

### صدى النص المحول إلى الدردشة (اختياري)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        echoTranscript: true, // الافتراضي هو false
        echoFormat: '📝 "{transcript}"', // اختياري، يدعم {transcript}
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

## ملاحظات وقيود

-   مصادقة المزود تتبع ترتيب مصادقة النموذج القياسي (ملفات تعريف المصادقة، متغيرات البيئة، `models.providers.*.apiKey`).
-   Deepgram يلتقط `DEEPGRAM_API_KEY` عند استخدام `provider: "deepgram"`.
-   تفاصيل إعداد Deepgram: [Deepgram (تحويل الصوت إلى نص)](../providers/deepgram.md).
-   تفاصيل إعداد Mistral: [Mistral](../providers/mistral.md).
-   يمكن لمزودي الصوت تجاوز `baseUrl` و `headers` و `providerOptions` عبر `tools.media.audio`.
-   الحد الأقصى الافتراضي للحجم هو 20 ميجابايت (`tools.media.audio.maxBytes`). يتم تخطي الملفات الصوتية التي تتجاوز الحجم لذلك النموذج ويتم تجربة الإدخال التالي.
-   يتم تخطي الملفات الصوتية الصغيرة/الفارغة التي تقل عن 1024 بايت قبل التحويل إلى نص بواسطة المزود/CLI.
-   القيمة الافتراضية لـ `maxChars` للصوت هي **غير مضبوطة** (نص كامل). اضبط `tools.media.audio.maxChars` أو `maxChars` لكل إدخال لتقليم المخرجات.
-   الافتراضي التلقائي لـ OpenAI هو `gpt-4o-mini-transcribe`؛ اضبط `model: "gpt-4o-transcribe"` للحصول على دقة أعلى.
-   استخدم `tools.media.audio.attachments` لمعالجة مذكرات صوتية متعددة (`mode: "all"` + `maxAttachments`).
-   النص المحول متاح للقوالب كـ `{{Transcript}}`.
-   `tools.media.audio.echoTranscript` معطل افتراضياً؛ فعّله لإرسال تأكيد النص المحول إلى الدردشة الأصلية قبل معالجة الوكيل.
-   `tools.media.audio.echoFormat` يخصص نص الصدى (العنصر النائب: `{transcript}`).
-   مخرجات stdout لـ CLI محدودة (5 ميجابايت)؛ حافظ على إخراج CLI موجزاً.

### دعم بيئة الوكيل

يحترم التحويل إلى نص المستند إلى مزود متغيرات بيئة الوكيل الصادرة القياسية:

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

إذا لم يتم تعيين أي متغيرات بيئة للوكيل، يتم استخدام الخروج المباشر. إذا كان تكوين الوكيل معطوباً، يسجل OpenClaw تحذيراً ويرجع إلى الجلب المباشر.

## كشف الإشارات في المجموعات

عند ضبط `requireMention: true` لمحادثة جماعية، يقوم OpenClaw الآن بتحويل الصوت إلى نص **قبل** التحقق من الإشارات. هذا يسمح بمعالجة المذكرات الصوتية حتى عندما تحتوي على إشارات. **كيف يعمل:**

1.  إذا كانت الرسالة الصوتية لا تحتوي على نص أساسي وتتطلب المجموعة الإشارات، يقوم OpenClaw بإجراء تحويل إلى نص "استباقي".
2.  يتم التحقق من النص المحول للبحث عن أنماط الإشارة (مثل `@BotName`، محفزات الإيموجي).
3.  إذا تم العثور على إشارة، تستمر الرسالة عبر خط أنابيب الرد الكامل.
4.  يتم استخدام النص المحول لكشف الإشارات حتى تتمكن المذكرات الصوتية من تجاوز بوابة الإشارة.

**سلوك الرجوع الاحتياطي:**

-   إذا فشل التحويل إلى نص أثناء العملية الاستباقية (مهلة، خطأ في API، إلخ)، تتم معالجة الرسالة بناءً على كشف الإشارات النصي فقط.
-   يضمن هذا عدم إسقاط الرسائل المختلطة (نص + صوت) بشكل غير صحيح.

**الانسحاب الاختياري لكل مجموعة/موضوع في Telegram:**

-   اضبط `channels.telegram.groups..disableAudioPreflight: true` لتخطي فحوصات الإشارات للنص المحول الاستباقي لتلك المجموعة.
-   اضبط `channels.telegram.groups..topics..disableAudioPreflight` للتجاوز لكل موضوع (`true` للتخطي، `false` للإجبار على التمكين).
-   الافتراضي هو `false` (الاستباقية مفعلة عندما تتطابق شروط بوابة الإشارة).

**مثال:** يرسل مستخدم مذكرة صوتية تقول "مرحباً @Claude، ما هي حالة الطقس؟" في مجموعة Telegram مع `requireMention: true`. يتم تحويل المذكرة الصوتية إلى نص، يتم اكتشاف الإشارة، ويقوم الوكيل بالرد.

## المحاذير

-   قواعد النطاق تستخدم قاعدة الفوز بأول تطابق. يتم توحيد `chatType` إلى `direct` أو `group` أو `room`.
-   تأكد من أن CLI يخرج بـ 0 ويطبع نصاً عادياً؛ يحتاج JSON إلى معالجة عبر `jq -r .text`.
-   بالنسبة لـ `parakeet-mlx`، إذا قمت بتمرير `--output-dir`، يقرأ OpenClaw `<output-dir>/<media-basename>.txt` عندما يكون `--output-format` هو `txt` (أو محذوف)؛ تنسيقات الإخراج غير `txt` تعود إلى تحليل stdout.
-   حافظ على المهلات معقولة (`timeoutSeconds`، الافتراضي 60 ثانية) لتجنب حظر قائمة انتظار الرد.
-   التحويل إلى نص الاستباقي يعالج فقط **أول** مرفق صوتي لكشف الإشارات. يتم معالجة الصوت الإضافي خلال مرحلة فهم الوسائط الرئيسية.

[دعم الصور والوسائط](./images.md)[التقاط الكاميرا](./camera.md)