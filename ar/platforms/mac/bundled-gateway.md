

  تطبيق macOS المرافق

  
# البوابة على macOS

لم يعد تطبيق OpenClaw.app يتضمن Node/Bun أو وقت تشغيل البوابة. يتوقع تطبيق macOS تثبيت **خارجي** لواجهة سطر أوامر `openclaw`، ولا يقوم بتشغيل البوابة كعملية فرعية، بل يدير خدمة launchd لكل مستخدم للحفاظ على تشغيل البوابة (أو يتصل ببوابة محلية موجودة إذا كانت تعمل بالفعل).

## تثبيت واجهة سطر الأوامر (مطلوب للوضع المحلي)

تحتاج إلى Node 22+ على جهاز Mac، ثم قم بتثبيت `openclaw` بشكل عام:

```bash
npm install -g openclaw@<version>
```

يقوم زر **تثبيت واجهة سطر الأوامر** في تطبيق macOS بنفس العملية عبر npm/pnpm (لا يُنصح باستخدام bun لوقت تشغيل البوابة).

## Launchd (البوابة كـ LaunchAgent)

الوسم:

-   `ai.openclaw.gateway` (أو `ai.openclaw.`؛ قد تبقى التسميات القديمة `com.openclaw.*`)

موقع ملف Plist (لكل مستخدم):

-   `~/Library/LaunchAgents/ai.openclaw.gateway.plist` (أو `~/Library/LaunchAgents/ai.openclaw..plist`)

المدير:

-   يمتلك تطبيق macOS عملية تثبيت/تحديث LaunchAgent في الوضع المحلي.
-   يمكن لواجهة سطر الأوامر أيضًا تثبيتها: `openclaw gateway install`.

السلوك:

-   يقوم خيار "OpenClaw Active" بتمكين/تعطيل LaunchAgent.
-   إغلاق التطبيق **لا** يوقف البوابة (يبقى launchd محافظًا على تشغيلها).
-   إذا كانت البوابة تعمل بالفعل على المنفذ المُهيأ، يتصل التطبيق بها بدلاً من بدء بوابة جديدة.

التسجيل:

-   stdout/err لـ launchd: `/tmp/openclaw/openclaw-gateway.log`

## توافق الإصدارات

يتحقق تطبيق macOS من إصدار البوابة مقابل إصداره الخاص. إذا كانا غير متوافقين، قم بتحديث واجهة سطر الأوامر العامة لتطابق إصدار التطبيق.

## فحص سريع

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

ثم:

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```

[إصدار macOS](./release.md)[IPC لـ macOS](./xpc.md)