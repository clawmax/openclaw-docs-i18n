title: "التقاط الكاميرا لعقد iOS وAndroid وmacOS في OpenClaw"
description: "تعلم كيفية التقاط الصور ومقاطع الفيديو باستخدام عقد الكاميرا في OpenClaw. قم بتكوين الإعدادات واستخدام أوامر CLI ودمج التقاط الكاميرا في سير عمل الوكيل."
keywords: ["التقاط الكاميرا", "عقد OpenClaw", "كاميرا iOS", "كاميرا Android", "كاميرا macOS", "node.invoke", "مساعد CLI", "التقاط الوسائط"]
---

  الوسائط والأجهزة

  
# التقاط الكاميرا

يدعم OpenClaw **التقاط الكاميرا** لسير عمل الوكيل:

-   **عقدة iOS** (مقترنة عبر Gateway): التقاط **صورة** (`jpg`) أو **مقطع فيديو قصير** (`mp4`، مع صوت اختياري) عبر `node.invoke`.
-   **عقدة Android** (مقترنة عبر Gateway): التقاط **صورة** (`jpg`) أو **مقطع فيديو قصير** (`mp4`، مع صوت اختياري) عبر `node.invoke`.
-   **تطبيق macOS** (عقدة عبر Gateway): التقاط **صورة** (`jpg`) أو **مقطع فيديو قصير** (`mp4`، مع صوت اختياري) عبر `node.invoke`.

جميع صلاحيات الوصول إلى الكاميرا محمية خلف **إعدادات يتحكم فيها المستخدم**.

## عقدة iOS

### إعداد المستخدم (مفعل افتراضيًا)

-   علامة تبويب إعدادات iOS → **الكاميرا** → **السماح بالكاميرا** (`camera.enabled`)
    -   الافتراضي: **مفعل** (يتم التعامل مع المفتاح المفقود على أنه مفعل).
    -   عند الإيقاف: تعيد أوامر `camera.*` خطأ `CAMERA_DISABLED`.

### الأوامر (عبر node.invoke في Gateway)

-   `camera.list`
    -   حمولة الاستجابة:
        -   `devices`: مصفوفة من `{ id, name, position, deviceType }`
-   `camera.snap`
    -   المعاملات:
        -   `facing`: `front|back` (الافتراضي: `front`)
        -   `maxWidth`: رقم (اختياري؛ الافتراضي `1600` على عقدة iOS)
        -   `quality`: `0..1` (اختياري؛ الافتراضي `0.9`)
        -   `format`: حاليًا `jpg`
        -   `delayMs`: رقم (اختياري؛ الافتراضي `0`)
        -   `deviceId`: نص (اختياري؛ من `camera.list`)
    -   حمولة الاستجابة:
        -   `format: "jpg"`
        -   `base64: "<...>"`
        -   `width`, `height`
    -   حماية الحمولة: يتم إعادة ضغط الصور للحفاظ على حجم الحمولة base64 أقل من 5 ميجابايت.
-   `camera.clip`
    -   المعاملات:
        -   `facing`: `front|back` (الافتراضي: `front`)
        -   `durationMs`: رقم (الافتراضي `3000`، مقيد بحد أقصى `60000`)
        -   `includeAudio`: منطقي (الافتراضي `true`)
        -   `format`: حاليًا `mp4`
        -   `deviceId`: نص (اختياري؛ من `camera.list`)
    -   حمولة الاستجابة:
        -   `format: "mp4"`
        -   `base64: "<...>"`
        -   `durationMs`
        -   `hasAudio`

### متطلب العمل في المقدمة

مثل `canvas.*`، تسمح عقدة iOS بأوامر `camera.*` فقط في **المقدمة**. ستعيد الاستدعاءات التي تتم في الخلفية خطأ `NODE_BACKGROUND_UNAVAILABLE`.

### مساعد CLI (ملفات مؤقتة + MEDIA)

أسهل طريقة للحصول على المرفقات هي عبر مساعد CLI، الذي يكتب الوسائط المفكوكة إلى ملف مؤقت ويطبع `MEDIA:`. أمثلة:

```bash
openclaw nodes camera snap --node <id>               # default: both front + back (2 MEDIA lines)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

ملاحظات:

-   `nodes camera snap` يلتقط افتراضيًا من **كلا** الاتجاهين لإعطاء الوكيل كلا المنظورين.
-   ملفات الإخراج مؤقتة (في دليل temp الخاص بالنظام) ما لم تقم ببناء غلاف خاص بك.

## عقدة Android

### إعداد المستخدم على Android (مفعل افتراضيًا)

-   ورقة إعدادات Android → **الكاميرا** → **السماح بالكاميرا** (`camera.enabled`)
    -   الافتراضي: **مفعل** (يتم التعامل مع المفتاح المفقود على أنه مفعل).
    -   عند الإيقاف: تعيد أوامر `camera.*` خطأ `CAMERA_DISABLED`.

### الأذونات

-   يتطلب Android أذونات وقت التشغيل:
    -   `CAMERA` لكل من `camera.snap` و `camera.clip`.
    -   `RECORD_AUDIO` لـ `camera.clip` عندما يكون `includeAudio=true`.

إذا كانت الأذونات مفقودة، سيطلب التطبيق ذلك عند الإمكان؛ وإذا تم الرفض، ستفشل طلبات `camera.*` بخطأ `*_PERMISSION_REQUIRED`.

### متطلب العمل في المقدمة على Android

مثل `canvas.*`، تسمح عقدة Android بأوامر `camera.*` فقط في **المقدمة**. ستعيد الاستدعاءات التي تتم في الخلفية خطأ `NODE_BACKGROUND_UNAVAILABLE`.

### أوامر Android (عبر Gateway node.invoke)

-   `camera.list`
    -   حمولة الاستجابة:
        -   `devices`: مصفوفة من `{ id, name, position, deviceType }`

### حماية الحمولة

يتم إعادة ضغط الصور للحفاظ على حجم الحمولة base64 أقل من 5 ميجابايت.

## تطبيق macOS

### إعداد المستخدم (معطل افتراضيًا)

يعرض تطبيق macOS المرافق خانة اختيار:

-   **الإعدادات → عام → السماح بالكاميرا** (`openclaw.cameraEnabled`)
    -   الافتراضي: **معطل**
    -   عند التعطيل: تعيد طلبات الكاميرا رسالة "تم تعطيل الكاميرا من قبل المستخدم".

### مساعد CLI (استدعاء العقدة)

استخدم واجهة سطر الأوامر الرئيسية `openclaw` لاستدعاء أوامر الكاميرا على عقدة macOS. أمثلة:

```bash
openclaw nodes camera list --node <id>            # list camera ids
openclaw nodes camera snap --node <id>            # prints MEDIA:<path>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # prints MEDIA:<path>
openclaw nodes camera clip --node <id> --duration-ms 3000      # prints MEDIA:<path> (legacy flag)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

ملاحظات:

-   `openclaw nodes camera snap` يستخدم افتراضيًا `maxWidth=1600` ما لم يتم التجاوز.
-   على macOS، ينتظر `camera.snap` مدة `delayMs` (الافتراضي 2000 مللي ثانية) بعد الإحماء/ضبط التعريض قبل الالتقاط.
-   يتم إعادة ضغط حمولات الصور للحفاظ على حجم base64 أقل من 5 ميجابايت.

## السلامة + الحدود العملية

-   يثير الوصول إلى الكاميرا والميكروفون مطالبات الأذونات المعتادة لنظام التشغيل (وتتطلب سلاسل استخدام في Info.plist).
-   مقاطع الفيديو مقيدة (حاليًا `<= 60s`) لتجنب حمولات عقدة كبيرة الحجم (زيادة حجم base64 + حدود الرسائل).

## تسجيل شاشة macOS (على مستوى نظام التشغيل)

لتسجيل **شاشة** الفيديو (وليس الكاميرا)، استخدم تطبيق macOS المرافق:

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # prints MEDIA:<path>
```

ملاحظات:

-   يتطلب إذن **تسجيل الشاشة** (TCC) على macOS.

[الصوت والمذكرات الصوتية](./audio.md)[وضع المحادثة](./talk.md)