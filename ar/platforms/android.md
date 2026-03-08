title: "دليل إعداد تطبيق OpenClaw لنظام Android ونظرة عامة على المنصة"
description: "تعلم كيفية إعداد عقدة الرفيق OpenClaw لنظام Android، وتوصيلها بالبوابة، واستخدام ميزات مثل الدردشة واللوحة والكاميرا والصوت."
keywords: ["openclaw android", "إعداد عقدة android", "إقران البوابة", "تطبيق رفيق android", "بوابة openclaw", "لوحة android", "التحكم الصوتي في android", "tailscale android"]
---

  نظرة عامة على المنصات

  
# تطبيق Android

## لقطة دعم

-   الدور: تطبيق عقدة رفيق (Android لا يستضيف البوابة).
-   البوابة مطلوبة: نعم (شغلها على macOS أو Linux أو Windows عبر WSL2).
-   التثبيت: [بدء الاستخدام](../start/getting-started.md) + [الإقران](../channels/pairing.md).
-   البوابة: [كتيب التشغيل](../gateway.md) + [التكوين](../gateway/configuration.md).
    -   البروتوكولات: [بروتوكول البوابة](../gateway/protocol.md) (العقد + مستوى التحكم).

## التحكم في النظام

التحكم في النظام (launchd/systemd) يعمل على مضيف البوابة. انظر [البوابة](../gateway.md).

## كتيب التشغيل للاتصال

تطبيق عقدة Android ⇄ (mDNS/NSD + WebSocket) ⇄ **البوابة** يتصل Android مباشرة بويب سوكيت البوابة (الافتراضي `ws://:18789`) ويستخدم إقران الجهاز (`role: node`).

### المتطلبات الأساسية

-   يمكنك تشغيل البوابة على الجهاز "الرئيسي".
-   يمكن لجهاز/محاكي Android الوصول إلى ويب سوكيت البوابة:
    -   نفس شبكة LAN مع mDNS/NSD، **أو**
    -   نفس شبكة Tailscale باستخدام Wide-Area Bonjour / unicast DNS-SD (انظر أدناه)، **أو**
    -   مضيف/منفذ البوابة يدوياً (الخيار الاحتياطي)
-   يمكنك تشغيل واجهة سطر الأوامر (`openclaw`) على جهاز البوابة (أو عبر SSH).

### 1) ابدأ البوابة

```bash
openclaw gateway --port 18789 --verbose
```

تأكد في السجلات من رؤية شيء مثل:

-   `listening on ws://0.0.0.0:18789`

لإعدادات شبكة Tailscale فقط (موصى بها لـ Vienna ⇄ London)، اربط البوابة بعنوان IP شبكة Tailscale:

-   عيّن `gateway.bind: "tailnet"` في `~/.openclaw/openclaw.json` على مضيف البوابة.
-   أعد تشغيل البوابة / تطبيق شريط القائمة على macOS.

### 2) تحقق من الاكتشاف (اختياري)

من جهاز البوابة:

```bash
dns-sd -B _openclaw-gw._tcp local.
```

ملاحظات تصحيح إضافية: [Bonjour](../gateway/bonjour.md).

#### اكتشاف شبكة Tailscale (Vienna ⇄ London) عبر unicast DNS-SD

اكتشاف NSD/mDNS لنظام Android لن يعبر الشبكات. إذا كانت عقدة Android والبوابة على شبكات مختلفة ولكن متصلتين عبر Tailscale، استخدم Wide-Area Bonjour / unicast DNS-SD بدلاً من ذلك:

1.  قم بإعداد منطقة DNS-SD (مثال `openclaw.internal.`) على مضيف البوابة وانشر سجلات `_openclaw-gw._tcp`.
2.  قم بتكوين تقسيم DNS في Tailscale للنطاق المختار بحيث يشير إلى خادم DNS ذلك.

التفاصيل ومثال تكوين CoreDNS: [Bonjour](../gateway/bonjour.md).

### 3) الاتصال من Android

في تطبيق Android:

-   يحافظ التطبيق على اتصال البوابة نشطاً عبر **خدمة في المقدمة** (إشعار مستمر).
-   افتح علامة التبويب **Connect**.
-   استخدم **كود الإعداد** أو وضع **يدوي**.
-   إذا كان الاكتشاف محظوراً، استخدم المضيف/المنفذ يدوياً (و TLS/الرمز المميز/كلمة المرور عند الحاجة) في **عناصر التحكم المتقدمة**.

بعد أول إقران ناجح، يعيد Android الاتصال تلقائياً عند التشغيل:

-   نقطة النهاية اليدوية (إذا تم تمكينها)، وإلا
-   آخر بوابة تم اكتشافها (بأفضل جهد).

### 4) الموافقة على الإقران (واجهة سطر الأوامر)

على جهاز البوابة:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

تفاصيل الإقران: [الإقران](../channels/pairing.md).

### 5) تحقق من اتصال العقدة

-   عبر حالة العقد:
    
    نسخ
    
    ```bash
    openclaw nodes status
    ```
    
-   عبر البوابة:
    
    نسخ
    
    ```bash
    openclaw gateway call node.list --params "{}"
    ```
    

### 6) الدردشة + السجل

تدعم علامة تبويب الدردشة في Android اختيار الجلسة (الافتراضي `main`، بالإضافة إلى الجلسات الأخرى الموجودة):

-   السجل: `chat.history`
-   الإرسال: `chat.send`
-   تحديثات الدفع (بأفضل جهد): `chat.subscribe` → `event:"chat"`

### 7) اللوحة + الشاشة + الكاميرا

#### مضيف اللوحة على البوابة (موصى به لمحتوى الويب)

إذا كنت تريد أن تعرض العقدة HTML/CSS/JS حقيقية يمكن للوكيل تعديلها على القرص، فوجّه العقدة إلى مضيف اللوحة على البوابة. ملاحظة: تقوم العقد بتحميل اللوحة من خادم HTTP للبوابة (نفس المنفذ مثل `gateway.port`، الافتراضي `18789`).

1.  أنشئ `~/.openclaw/workspace/canvas/index.html` على مضيف البوابة.
2.  انتقل بالعقدة إليها (شبكة LAN):

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18789/__openclaw__/canvas/"}'
```

شبكة Tailscale (اختياري): إذا كان كلا الجهازين على Tailscale، استخدم اسم MagicDNS أو عنوان IP لشبكة Tailscale بدلاً من `.local`، على سبيل المثال `http://<gateway-magicdns>:18789/__openclaw__/canvas/`. يقوم هذا الخادم بحقن عميل إعادة تحميل حي في HTML ويعيد التحميل عند تغيير الملفات. مضيف A2UI موجود في `http://<gateway-host>:18789/__openclaw__/a2ui/`. أوامر اللوحة (في المقدمة فقط):

-   `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (استخدم `{"url":""}` أو `{"url":"/"}` للعودة إلى السقالة الافتراضية). `canvas.snapshot` تُرجع `{ format, base64 }` (الافتراضي `format="jpeg"`).
-   A2UI: `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL` اسم مستعار قديم)

أوامر الكاميرا (في المقدمة فقط؛ محمية بالإذن):

-   `camera.snap` (jpg)
-   `camera.clip` (mp4)

انظر [عقدة الكاميرا](../nodes/camera.md) للمعلمات وأدوات مساعدة لواجهة سطر الأوامر. أوامر الشاشة:

-   `screen.record` (mp4؛ في المقدمة فقط)

### 8) الصوت + سطح أمر موسع لنظام Android

-   الصوت: يستخدم Android تدفق تشغيل/إيقاف واحد للميكروفون في علامة تبويب الصوت مع التقاط النص والاستماع إلى TTS (ElevenLabs عند التكوين، TTS النظام كخيار احتياطي).
-   تمت إزالة مفاتيح تفعيل/تبديل وضع التحدث الصوتي حاليًا من واجهة مستخدم Android/وقت التشغيل.
-   عائلات أوامر Android إضافية (التوفر يعتمد على الجهاز + الأذونات):
    -   `device.status`, `device.info`, `device.permissions`, `device.health`
    -   `notifications.list`, `notifications.actions`
    -   `photos.latest`
    -   `contacts.search`, `contacts.add`
    -   `calendar.events`, `calendar.add`
    -   `motion.activity`, `motion.pedometer`
    -   `app.update`

[Windows (WSL2)](./windows.md)[تطبيق iOS](./ios.md)