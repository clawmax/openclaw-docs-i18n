

  الوسائط والأجهزة

  
# تحويل النص إلى كلام

يمكن لـ OpenClaw تحويل الردود الصادرة إلى صوت باستخدام ElevenLabs أو OpenAI أو Edge TTS. يعمل في أي مكان يمكن لـ OpenClaw إرسال الصوت إليه؛ يحصل Telegram على فقاعة صوتية دائرية.

## الخدمات المدعومة

-   **ElevenLabs** (المزود الأساسي أو الاحتياطي)
-   **OpenAI** (المزود الأساسي أو الاحتياطي؛ يُستخدم أيضًا للملخصات)
-   **Edge TTS** (المزود الأساسي أو الاحتياطي؛ يستخدم `node-edge-tts`، الافتراضي عند عدم وجود مفاتيح API)

### ملاحظات حول Edge TTS

يستخدم Edge TTS خدمة تحويل النص إلى كلام العصبية عبر الإنترنت من Microsoft Edge عبر مكتبة `node-edge-tts`. إنها خدمة مستضافة (ليست محلية)، تستخدم نقاط نهاية Microsoft، ولا تتطلب مفتاح API. تعرض `node-edge-tts` خيارات تكوين الكلام وتنسيقات الإخراج، ولكن ليس كل الخيارات مدعومة من خدمة Edge. citeturn2search0 نظرًا لأن Edge TTS هي خدمة ويب عامة بدون اتفاقية مستوى خدمة (SLA) أو حصة منشورة، عالجها على أنها خدمة بأفضل جهد. إذا كنت بحاجة إلى حدود و دعم مضمونين، استخدم OpenAI أو ElevenLabs. توثّق واجهة برمجة تطبيقات REST للكلام من Microsoft حدًا زمنيًا قدره 10 دقائق لكل طلب؛ لا ينشر Edge TTS حدودًا، لذا افترض حدودًا مماثلة أو أقل. citeturn0search3

## المفاتيح الاختيارية

إذا كنت تريد OpenAI أو ElevenLabs:

-   `ELEVENLABS_API_KEY` (أو `XI_API_KEY`)
-   `OPENAI_API_KEY`

لا يتطلب Edge TTS مفتاح API. إذا لم يتم العثور على مفاتيح API، يستخدم OpenClaw Edge TTS افتراضيًا (ما لم يتم تعطيله عبر `messages.tts.edge.enabled=false`). إذا تم تكوين عدة مزودين، يتم استخدام المزود المحدد أولاً والآخرون هم خيارات احتياطية. يستخدم الملخص التلقائي النموذج `summaryModel` المُهيأ (أو `agents.defaults.model.primary`)، لذا يجب أن يكون ذلك المزود مصادقًا عليه أيضًا إذا قمت بتمكين الملخصات.

## روابط الخدمات

-   [دليل تحويل النص إلى كلام من OpenAI](https://platform.openai.com/docs/guides/text-to-speech)
-   [مرجع واجهة برمجة تطبيقات الصوت من OpenAI](https://platform.openai.com/docs/api-reference/audio)
-   [تحويل النص إلى كلام من ElevenLabs](https://elevenlabs.io/docs/api-reference/text-to-speech)
-   [المصادقة من ElevenLabs](https://elevenlabs.io/docs/api-reference/authentication)
-   [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
-   [تنسيقات إخراج الكلام من Microsoft](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

## هل هو مفعّل افتراضيًا؟

لا. التحويل التلقائي للنص إلى كلام **معطّل** افتراضيًا. فعّله في الإعدادات باستخدام `messages.tts.auto` أو لكل جلسة باستخدام `/tts always` (الاسم المستعار: `/tts on`). Edge TTS **مفعّل** افتراضيًا بمجرد تشغيل TTS، ويُستخدم تلقائيًا عندما لا تتوفر مفاتيح API لـ OpenAI أو ElevenLabs.

## التهيئة

توجد تهيئة TTS تحت `messages.tts` في `openclaw.json`. المخطط الكامل موجود في [تهيئة البوابة](./gateway/configuration.md).

### التهيئة الدنيا (تمكين + مزود)

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs",
    },
  },
}
```

### OpenAI أساسي مع احتياطي ElevenLabs

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      openai: {
        apiKey: "openai_api_key",
        baseUrl: "https://api.openai.com/v1",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
    },
  },
}
```

### Edge TTS أساسي (بدون مفتاح API)

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%",
      },
    },
  },
}
```

### تعطيل Edge TTS

```json
{
  messages: {
    tts: {
      edge: {
        enabled: false,
      },
    },
  },
}
```

### حدود مخصصة + مسار التفضيلات

```json
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
    },
  },
}
```

### الرد بالصوت فقط بعد مذكرة صوتية واردة

```json
{
  messages: {
    tts: {
      auto: "inbound",
    },
  },
}
```

### تعطيل الملخص التلقائي للردود الطويلة

```json
{
  messages: {
    tts: {
      auto: "always",
    },
  },
}
```

ثم نفّذ:

```bash
/tts summary off
```

### ملاحظات على الحقول

-   `auto`: وضع التحويل التلقائي للنص إلى كلام (`off`, `always`, `inbound`, `tagged`).
    -   `inbound` يرسل الصوت فقط بعد مذكرة صوتية واردة.
    -   `tagged` يرسل الصوت فقط عندما يتضمن الرد علامات `[[tts]]`.
-   `enabled`: مفتاح تشغيل قديم (يقوم الطبيب بترحيل هذا إلى `auto`).
-   `mode`: `"final"` (الافتراضي) أو `"all"` (يتضمن ردود الأدوات/الكتل).
-   `provider`: `"elevenlabs"`, `"openai"`, أو `"edge"` (الاحتياطي تلقائي).
-   إذا كان `provider` **غير مضبوط**، يفضل OpenClaw `openai` (إذا كان هناك مفتاح)، ثم `elevenlabs` (إذا كان هناك مفتاح)، وإلا `edge`.
-   `summaryModel`: نموذج رخيص اختياري للملخص التلقائي؛ الافتراضي هو `agents.defaults.model.primary`.
    -   يقبل `provider/model` أو اسم مستعار للنموذج المُهيأ.
-   `modelOverrides`: السماح للنموذج بإصدار توجيهات TTS (مفعّل افتراضيًا).
    -   `allowProvider` الافتراضي هو `false` (تبديل المزود اختياري).
-   `maxTextLength`: حد صارم لمدخلات TTS (أحرف). يفشل `/tts audio` إذا تم تجاوزه.
-   `timeoutMs`: مهلة الطلب (ميلي ثانية).
-   `prefsPath`: تجاوز مسار ملف JSON للتفضيلات المحلية (المزود/الحد/الملخص).
-   قيم `apiKey` تعود إلى متغيرات البيئة (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
-   `elevenlabs.baseUrl`: تجاوز عنوان URL الأساسي لواجهة برمجة تطبيقات ElevenLabs.
-   `openai.baseUrl`: تجاوز نقطة نهاية TTS لـ OpenAI.
    -   ترتيب الدقة: `messages.tts.openai.baseUrl` -> `OPENAI_TTS_BASE_URL` -> `https://api.openai.com/v1`
    -   تعامل القيم غير الافتراضية كنقاط نهاية TTS متوافقة مع OpenAI، لذا يتم قبول أسماء النماذج والأصوات المخصصة.
-   `elevenlabs.voiceSettings`:
    -   `stability`, `similarityBoost`, `style`: `0..1`
    -   `useSpeakerBoost`: `true|false`
    -   `speed`: `0.5..2.0` (1.0 = عادي)
-   `elevenlabs.applyTextNormalization`: `auto|on|off`
-   `elevenlabs.languageCode`: كود لغة ISO 639-1 مكون من حرفين (مثل `en`, `de`)
-   `elevenlabs.seed`: عدد صحيح `0..4294967295` (حتمية بأفضل جهد)
-   `edge.enabled`: السماح باستخدام Edge TTS (الافتراضي `true`؛ بدون مفتاح API).
-   `edge.voice`: اسم الصوت العصبي لـ Edge (مثل `en-US-MichelleNeural`).
-   `edge.lang`: كود اللغة (مثل `en-US`).
-   `edge.outputFormat`: تنسيق إخراج Edge (مثل `audio-24khz-48kbitrate-mono-mp3`).
    -   راجع تنسيقات إخراج الكلام من Microsoft للقيم الصالحة؛ ليس كل التنسيقات مدعومة من Edge.
-   `edge.rate` / `edge.pitch` / `edge.volume`: سلاسل نسب مئوية (مثل `+10%`, `-5%`).
-   `edge.saveSubtitles`: كتابة ترجمات JSON بجانب ملف الصوت.
-   `edge.proxy`: عنوان URL للوكيل لطلبات Edge TTS.
-   `edge.timeoutMs`: تجاوز مهلة الطلب (ميلي ثانية).

## تجاوزات يقودها النموذج (مفعّل افتراضيًا)

افتراضيًا، يمكن للنموذج **أن** يصدر توجيهات TTS لرد واحد. عندما يكون `messages.tts.auto` مضبوطًا على `tagged`، تكون هذه التوجيهات مطلوبة لتحفيز الصوت. عند التمكين، يمكن للنموذج إصدار توجيهات `[[tts:...]]` لتجاوز الصوت لرد واحد، بالإضافة إلى كتلة اختيارية `[[tts:text]]...[[/tts:text]]` لتوفير علامات تعبيرية (ضحك، إشارات غناء، إلخ) يجب أن تظهر فقط في الصوت. يتم تجاهل توجيهات `provider=...` ما لم يكن `modelOverrides.allowProvider: true`. مثال على حمولة الرد:

```
ها أنت ذا.

[[tts:voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](يضحك) اقرأ الأغنية مرة أخرى.[[/tts:text]]
```

مفاتيح التوجيه المتاحة (عند التمكين):

-   `provider` (`openai` | `elevenlabs` | `edge`، يتطلب `allowProvider: true`)
-   `voice` (صوت OpenAI) أو `voiceId` (ElevenLabs)
-   `model` (نموذج TTS من OpenAI أو معرف نموذج ElevenLabs)
-   `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
-   `applyTextNormalization` (`auto|on|off`)
-   `languageCode` (ISO 639-1)
-   `seed`

تعطيل جميع تجاوزات النموذج:

```json
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false,
      },
    },
  },
}
```

قائمة السماح الاختيارية (تمكين تبديل المزود مع الحفاظ على إمكانية ضبط المقابض الأخرى):

```json
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: true,
        allowSeed: false,
      },
    },
  },
}
```

## التفضيلات لكل مستخدم

تكتب أوامر الشرطة المائلة تجاوزات محلية إلى `prefsPath` (الافتراضي: `~/.openclaw/settings/tts.json`، تجاوز باستخدام `OPENCLAW_TTS_PREFS` أو `messages.tts.prefsPath`). الحقول المخزنة:

-   `enabled`
-   `provider`
-   `maxLength` (حد الملخص؛ الافتراضي 1500 حرف)
-   `summarize` (الافتراضي `true`)

هذه تتجاوز `messages.tts.*` لذلك المضيف.

## تنسيقات الإخراج (ثابتة)

-   **Telegram**: مذكرة صوتية Opus (`opus_48000_64` من ElevenLabs، `opus` من OpenAI).
    -   48 كيلوهرتز / 64 كيلوبت في الثانية هو توازن جيد للمذكرة الصوتية ومطلوب للفقاعة الدائرية.
-   **القنوات الأخرى**: MP3 (`mp3_44100_128` من ElevenLabs، `mp3` من OpenAI).
    -   44.1 كيلوهرتز / 128 كيلوبت في الثانية هو التوازن الافتراضي لوضوح الكلام.
-   **Edge TTS**: يستخدم `edge.outputFormat` (الافتراضي `audio-24khz-48kbitrate-mono-mp3`).
    -   يقبل `node-edge-tts` `outputFormat`، ولكن ليس كل التنسيقات متاحة من خدمة Edge. citeturn2search0
    -   تتبع قيم تنسيق الإخراج تنسيقات إخراج الكلام من Microsoft (بما في ذلك Ogg/WebM Opus). citeturn1search0
    -   يقبل `sendVoice` في Telegram OGG/MP3/M4A؛ استخدم OpenAI/ElevenLabs إذا كنت بحاجة إلى مذكرات صوتية Opus مضمونة. citeturn1search1
    -   إذا فشل تنسيق إخراج Edge المُهيأ، يعيد OpenClaw المحاولة باستخدام MP3.

تنسيقات OpenAI/ElevenLabs ثابتة؛ يتوقع Telegram Opus لتجربة استخدام المذكرة الصوتية.

## سلوك التحويل التلقائي للنص إلى كلام

عند التمكين، يقوم OpenClaw بما يلي:

-   يتخطى TTS إذا كان الرد يحتوي بالفعل على وسائط أو توجيه `MEDIA:`.
-   يتخطى الردود القصيرة جدًا (< 10 أحرف).
-   يلخص الردود الطويلة عند التمكين باستخدام `agents.defaults.model.primary` (أو `summaryModel`).
-   يرفق الصوت المُنشأ بالرد.

إذا تجاوز الرد `maxLength` وكان الملخص معطلاً (أو لا يوجد مفتاح API لنموذج الملخص)، يتم تخطي الصوت وإرسال الرد النصي العادي.

## مخطط التدفق

```
الرد -> هل TTS مفعّل؟
  لا  -> إرسال نص
  نعم -> هل يحتوي على وسائط / MEDIA: / قصير؟
          نعم -> إرسال نص
          لا  -> هل الطول > الحد؟
                   لا  -> TTS -> إرفاق صوت
                   نعم -> هل الملخص مفعّل؟
                            لا  -> إرسال نص
                            نعم -> تلخيص (summaryModel أو agents.defaults.model.primary)
                                      -> TTS -> إرفاق صوت
```

## استخدام أوامر الشرطة المائلة

هناك أمر واحد: `/tts`. راجع [أوامر الشرطة المائلة](./tools/slash-commands.md) للحصول على تفاصيل التمكين. ملاحظة Discord: `/tts` هو أمر مضمن في Discord، لذلك يسجل OpenClaw `/voice` كالأمر الأصلي هناك. لا يزال النص `/tts ...` يعمل.

```bash
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

ملاحظات:

-   تتطلب الأوامر مرسلًا مصرحًا به (لا تزال قواعد القائمة المسموح بها/المالك سارية).
-   يجب تمكين `commands.text` أو تسجيل الأمر الأصلي.
-   `off|always|inbound|tagged` هي مفاتيح تبديل لكل جلسة (`/tts on` هو اسم مستعار لـ `/tts always`).
-   يتم تخزين `limit` و `summary` في التفضيلات المحلية، وليس في الإعدادات الرئيسية.
-   `/tts audio` يولد رد صوتي لمرة واحدة (لا يقوم بتشغيل TTS).

## أداة الوكيل

تقوم أداة `tts` بتحويل النص إلى كلام وإرجاع مسار `MEDIA:`. عندما تكون النتيجة متوافقة مع Telegram، تتضمن الأداة `[[audio_as_voice]]` حتى يرسل Telegram فقاعة صوت.

## بوابة RPC

طرق البوابة:

-   `tts.status`
-   `tts.enable`
-   `tts.disable`
-   `tts.convert`
-   `tts.setProvider`
-   `tts.providers`

[أمر الموقع](./nodes/location-command.md)