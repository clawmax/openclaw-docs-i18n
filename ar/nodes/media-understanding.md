title: "دليل تكوين عقد فهم الوسائط لـ OpenClaw AI"
description: "تعلم كيفية تكوين OpenClaw AI لتلخيص الصور والصوت والفيديو. قم بإعداد واجهات برمجة التطبيقات (APIs) للمزودين، والبدائل عبر سطر الأوامر (CLI)، وسياسات المرفقات لمعالجة الوسائط المثلى."
keywords: ["فهم الوسائط", "openclaw ai", "نسخ الصوت", "تلخيص الصور", "تحليل الفيديو", "تكوين الذكاء الاصطناعي", "ذكاء اصطناعي متعدد الوسائط", "معالجة الوسائط"]
---

  الوسائط والأجهزة

  
# فهم الوسائط

يمكن لـ OpenClaw **تلخيص الوسائط الواردة** (صورة/صوت/فيديو) قبل تشغيل خط الرد. يكتشف تلقائيًا متى تكون الأدوات المحلية أو مفاتيح المزود متاحة، ويمكن تعطيله أو تخصيصه. إذا كان الفهم معطلاً، فإن النماذج لا تزال تستقبل الملفات/الروابط الأصلية كالمعتاد.

## الأهداف

-   اختياري: هضم الوسائط الواردة مسبقًا وتحويلها إلى نص قصير لتوجيه أسرع + تحليل أفضل للأوامر.
-   الحفاظ على تسليم الوسائط الأصلية للنموذج (دائمًا).
-   دعم **واجهات برمجة التطبيقات (APIs) للمزودين** و**البدائل عبر سطر الأوامر (CLI)**.
-   السماح بتعدد النماذج مع ترتيب بديل عند الخطأ/الحجم/انتهاء المهلة.

## السلوك عالي المستوى

1.  جمع المرفقات الواردة (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2.  لكل قدرة مفعلة (صورة/صوت/فيديو)، حدد المرفقات حسب السياسة (الافتراضي: **الأول**).
3.  اختر أول إدخال نموذج مؤهل (الحجم + القدرة + المصادقة).
4.  إذا فشل النموذج أو كان حجم الوسائط كبيرًا جدًا، **الانتقال إلى الإدخال التالي**.
5.  عند النجاح:
    -   يصبح `Body` كتلة `[Image]`، أو `[Audio]`، أو `[Video]`.
    -   يضبط الصوت `{{Transcript}}`؛ يستخدم تحليل الأوامر نص التسمية التوضيحية عند وجوده، وإلا يستخدم النص المنقول.
    -   يتم الاحتفاظ بالتسميات التوضيحية كـ `User text:` داخل الكتلة.

إذا فشل الفهم أو تم تعطيله، **يستمر تدفق الرد** مع النص الأصلي + المرفقات.

## نظرة عامة على التكوين

يدعم `tools.media` **نماذج مشتركة** بالإضافة إلى تجاوزات لكل قدرة:

-   `tools.media.models`: قائمة النماذج المشتركة (استخدم `capabilities` للتحكم).
-   `tools.media.image` / `tools.media.audio` / `tools.media.video`:
    -   القيم الافتراضية (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
    -   تجاوزات المزود (`baseUrl`, `headers`, `providerOptions`)
    -   خيارات Deepgram للصوت عبر `tools.media.audio.providerOptions.deepgram`
    -   عناصر تحكم صدى النص المنقول للصوت (`echoTranscript`, الافتراضي `false`; `echoFormat`)
    -   **قائمة `models` لكل قدرة** اختيارية (مفضلة قبل النماذج المشتركة)
    -   سياسة `attachments` (`mode`, `maxAttachments`, `prefer`)
    -   `scope` (تحكم اختياري حسب القناة/نوع الدردشة/مفتاح الجلسة)
-   `tools.media.concurrency`: الحد الأقصى للتشغيل المتزامن للقدرات (الافتراضي **2**).

```json
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
        echoTranscript: true,
        echoFormat: '📝 "{transcript}"',
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### إدخالات النموذج

يمكن أن يكون كل إدخال في `models[]` **مزود** أو **CLI**:

```json
{
  type: "provider", // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, used for multi‑modal entries
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

يمكن للنماذج في CLI أيضًا استخدام:

-   `{{MediaDir}}` (الدليل الذي يحتوي على ملف الوسائط)
-   `{{OutputDir}}` (دليل مؤقت تم إنشاؤه لهذا التشغيل)
-   `{{OutputBase}}` (مسار أساسي للملف المؤقت، بدون امتداد)

## القيم الافتراضية والحدود

القيم الافتراضية الموصى بها:

-   `maxChars`: **500** للصورة/الفيديو (قصير، مناسب للأوامر)
-   `maxChars`: **غير محدد** للصوت (نص منقول كامل ما لم تحدد حدًا)
-   `maxBytes`:
    -   صورة: **10MB**
    -   صوت: **20MB**
    -   فيديو: **50MB**

القواعد:

-   إذا تجاوزت الوسائط `maxBytes`، يتم تخطي هذا النموذج ويتم **تجربة النموذج التالي**.
-   الملفات الصوتية الأصغر من **1024 بايت** تعتبر فارغة/تالفة ويتم تخطيها قبل النسخ عبر المزود/CLI.
-   إذا أعاد النموذج أكثر من `maxChars`، يتم تقليم الناتج.
-   `prompt` الافتراضي هو "Describe the ." بسيط بالإضافة إلى توجيه `maxChars` (للصورة/الفيديو فقط).
-   إذا كان `.enabled: true` ولكن لم يتم تكوين أي نماذج، يحاول OpenClaw استخدام **نموذج الرد النشط** عندما يدعم مزوده هذه القدرة.

### الكشف التلقائي عن فهم الوسائط (الافتراضي)

إذا لم يتم تعيين `tools.media..enabled` إلى `false` ولم تقم بتكوين نماذج، يكتشف OpenClaw تلقائيًا بهذا الترتيب **ويتوقف عند أول خيار يعمل**:

1.  **CLI المحلي** (للصوت فقط؛ إذا كان مثبتًا)
    -   `sherpa-onnx-offline` (يتطلب `SHERPA_ONNX_MODEL_DIR` مع encoder/decoder/joiner/tokens)
    -   `whisper-cli` (`whisper-cpp`; يستخدم `WHISPER_CPP_MODEL` أو النموذج الصغير المرفق)
    -   `whisper` (Python CLI; يقوم بتنزيل النماذج تلقائيًا)
2.  **Gemini CLI** (`gemini`) باستخدام `read_many_files`
3.  **مفاتيح المزود**
    -   الصوت: OpenAI → Groq → Deepgram → Google
    -   الصورة: OpenAI → Anthropic → Google → MiniMax
    -   الفيديو: Google

لتعطيل الكشف التلقائي، عيّن:

```json
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

ملاحظة: الكشف عن الملفات الثنائية هو بأفضل جهد عبر أنظمة macOS/Linux/Windows؛ تأكد من أن CLI موجود في `PATH` (نقوم بتوسيع `~`)، أو قم بتعيين نموذج CLI صريح مع مسار أمر كامل.

### دعم بيئة الوكيل (نماذج المزود)

عند تمكين فهم الوسائط القائم على المزود للـ**صوت** و**الفيديو**، يحترم OpenClaw متغيرات بيئة الوكيل الصادرة القياسية لمكالمات HTTP الخاصة بالمزود:

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

إذا لم يتم تعيين أي متغيرات بيئة للوكيل، يستخدم فهم الوسائط اتصالاً مباشرًا. إذا كانت قيمة الوكيل غير صحيحة، يسجل OpenClaw تحذيرًا ويرجع إلى الجلب المباشر.

## القدرات (اختياري)

إذا قمت بتعيين `capabilities`، يعمل الإدخال فقط لأنواع الوسائط تلك. للقوائم المشتركة، يمكن لـ OpenClaw استنتاج القيم الافتراضية:

-   `openai`, `anthropic`, `minimax`: **صورة**
-   `google` (Gemini API): **صورة + صوت + فيديو**
-   `groq`: **صوت**
-   `deepgram`: **صوت**

لإدخالات CLI، **قم بتعيين `capabilities` بشكل صريح** لتجنب المطابقات المفاجئة. إذا حذفت `capabilities`، يكون الإدخال مؤهلاً للقائمة التي يظهر فيها.

## مصفوفة دعم المزود (تكاملات OpenClaw)

| القدرة | تكامل المزود | ملاحظات |
| --- | --- | --- |
| الصورة | OpenAI / Anthropic / Google / آخرون عبر `pi-ai` | أي نموذج يدعم الصور في السجل يعمل. |
| الصوت | OpenAI, Groq, Deepgram, Google, Mistral | النسخ عبر المزود (Whisper/Deepgram/Gemini/Voxtral). |
| الفيديو | Google (Gemini API) | فهم الفيديو عبر المزود. |

## إرشادات اختيار النموذج

-   فضّل أقوى نموذج من الجيل الأخير المتاح لكل قدرة وسائط عندما تكون الجودة والسلامة مهمة.
-   للوكلاء المدعومين بالأدوات الذين يتعاملون مع مدخلات غير موثوقة، تجنب نماذج الوسائط القديمة/الضعيفة.
-   احتفظ ببديل واحد على الأقل لكل قدرة للتوفّر (نموذج عالي الجودة + نموذج أسرع/أرخص).
-   البدائل عبر CLI (`whisper-cli`, `whisper`, `gemini`) مفيدة عندما تكون واجهات برمجة التطبيقات (APIs) للمزود غير متاحة.
-   ملاحظة `parakeet-mlx`: مع `--output-dir`، يقرأ OpenClaw `<output-dir>/<media-basename>.txt` عندما يكون تنسيق الإخراج هو `txt` (أو غير محدد)؛ التنسيقات غير `txt` ترجع إلى stdout.

## سياسة المرفقات

تتحكم `attachments` لكل قدرة في المرفقات التي تتم معالجتها:

-   `mode`: `first` (الافتراضي) أو `all`
-   `maxAttachments`: تحديد الحد الأقصى للمرفقات المعالجة (الافتراضي **1**)
-   `prefer`: `first`, `last`, `path`, `url`

عند `mode: "all"`، يتم تسمية المخرجات `[Image 1/2]`، `[Audio 2/2]`، إلخ.

## أمثلة التكوين

### 1) قائمة النماذج المشتركة + تجاوزات

```json
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2) الصوت + الفيديو فقط (الصورة معطلة)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3) فهم الصورة الاختياري

```json
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4) إدخال واحد متعدد الوسائط (قدرات صريحة)

```json
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## إخراج الحالة

عند تشغيل فهم الوسائط، يتضمن `/status` سطر ملخص قصير:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

يظهر هذا النتائج لكل قدرة والمزود/النموذج المختار عند الاقتضاء.

## ملاحظات

-   الفهم هو **بأفضل جهد**. الأخطاء لا تمنع الردود.
-   لا تزال المرفقات تُمرر إلى النماذج حتى عندما يكون الفهم معطلاً.
-   استخدم `scope` للحد من مكان تشغيل الفهم (مثل: الرسائل المباشرة فقط).

## وثائق ذات صلة

-   [التكوين](../gateway/configuration.md)
-   [دعم الصور والوسائط](./images.md)

[استكشاف أخطاء العقد وإصلاحها](./troubleshooting.md)[دعم الصور والوسائط](./images.md)