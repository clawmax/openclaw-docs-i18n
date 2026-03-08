title: "نشر OpenClaw على Railway باستخدام قالب النقر الواحد"
description: "تعلّم كيفية نشر OpenClaw على Railway باستخدام قالب النقر الواحد. اتبع معالج الإعداد، قم بتكوين المتغيرات المطلوبة، واستضف بوابتك دون أوامر طرفية."
keywords: ["openclaw railway", "نشر openclaw", "نشر railway", "نشر بنقرة واحدة", "معالج إعداد openclaw", "استضافة openclaw", "حجم railway", "بوابة openclaw"]
---

  الاستضافة والنشر

  
# النشر على Railway

انشر OpenClaw على Railway باستخدام قالب النقر الواحد وأكمل الإعداد في متصفحك. هذه أسهل طريقة "بدون طرفية على الخادم": يقوم Railway بتشغيل البوابة نيابة عنك، وتقوم بتكوين كل شيء عبر معالج الويب `/setup`.

## قائمة مراجعة سريعة (للمستخدمين الجدد)

1.  انقر على **Deploy on Railway** (أدناه).
2.  أضف **حجمًا** (Volume) مُثبتًا على المسار `/data`.
3.  عيّن **المتغيرات** المطلوبة (على الأقل `SETUP_PASSWORD`).
4.  فعّل **الوكيل HTTP** (HTTP Proxy) على المنفذ `8080`.
5.  افتح `https://<your-railway-domain>/setup` وأكمل المعالج.

## النشر بنقرة واحدة

[Deploy on Railway](https://railway.com/deploy/clawdbot-railway-template) بعد النشر، ابحث عن عنوان URL العام في **Railway → خدمتك → الإعدادات → النطاقات**. سيقوم Railway إما بـ:

-   منحك نطاقًا مُنشأً (غالبًا `https://.up.railway.app`)، أو
-   استخدام نطاقك المخصص إذا قمت بإرفاق واحد.

ثم افتح:

-   `https://<your-railway-domain>/setup` — معالج الإعداد (محمي بكلمة مرور)
-   `https://<your-railway-domain>/openclaw` — واجهة التحكم

## ما تحصل عليه

-   بوابة OpenClaw مستضافة + واجهة تحكم
-   معالج إعداد ويب على المسار `/setup` (بدون أوامر طرفية)
-   تخزين دائم عبر حجم Railway (`/data`) بحيث تبقى التكوينات/الاعتمادات/مساحة العمل بعد إعادة النشر
-   تصدير نسخة احتياطية على المسار `/setup/export` للهجرة من Railway لاحقًا

## إعدادات Railway المطلوبة

### الشبكة العامة

فعّل **الوكيل HTTP** (HTTP Proxy) للخدمة.

-   المنفذ: `8080`

### الحجم (مطلوب)

ارفق حجمًا مُثبتًا على المسار:

-   `/data`

### المتغيرات

عيّن هذه المتغيرات على الخدمة:

-   `SETUP_PASSWORD` (مطلوب)
-   `PORT=8080` (مطلوب — يجب أن يطابق المنفذ في الشبكة العامة)
-   `OPENCLAW_STATE_DIR=/data/.openclaw` (موصى به)
-   `OPENCLAW_WORKSPACE_DIR=/data/workspace` (موصى به)
-   `OPENCLAW_GATEWAY_TOKEN` (موصى به؛ عالجه كسر إداري)

## تدفق الإعداد

1.  زر `https://<your-railway-domain>/setup` وأدخل `SETUP_PASSWORD` الخاص بك.
2.  اختر مزود نموذج/مصادقة والصق مفتاحك.
3.  (اختياري) أضف رموز Telegram/Discord/Slack.
4.  انقر على **Run setup**.

إذا كانت الرسائل المباشرة (DMs) في Telegram مضبوطة على الاقتران، يمكن لمعالج الإعداد الموافقة على رمز الاقتران.

## الحصول على رموز الدردشة

### رمز بوت Telegram

1.  راسل `@BotFather` في Telegram
2.  نفّذ `/newbot`
3.  انسخ الرمز (يبدو مثل `123456789:AA...`)
4.  الصقه في `/setup`

### رمز بوت Discord

1.  اذهب إلى [https://discord.com/developers/applications](https://discord.com/developers/applications)
2.  **New Application** → اختر اسمًا
3.  **Bot** → **Add Bot**
4.  **Enable MESSAGE CONTENT INTENT** تحت Bot → Privileged Gateway Intents (مطلوب وإلا سيتعطل البوت عند بدء التشغيل)
5.  انسخ **Bot Token** والصقه في `/setup`
6.  ادعُ البوت إلى خادمك (OAuth2 URL Generator؛ النطاقات: `bot`, `applications.commands`)

## النسخ الاحتياطية والهجرة

حمّل نسخة احتياطية من:

-   `https://<your-railway-domain>/setup/export`

هذا يُصدّر حالة OpenClaw + مساحة العمل الخاصة بك حتى تتمكن من الهجرة إلى مضيف آخر دون فقدان التكوين أو الذاكرة.

[exe.dev](./exe-dev.md)[Deploy on Render](./render.md)

---