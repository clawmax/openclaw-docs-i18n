title: "دعم منصة OpenClaw لأنظمة macOS و iOS و Android و Windows و Linux"
description: "تعرف على المنصات التي يدعمها OpenClaw، بما في ذلك macOS و Windows و Linux و iOS و Android. ابحث عن أدلة التثبيت لاستضافة VPS وخدمة Gateway."
keywords: ["منصات openclaw", "تثبيت gateway", "استضافة vps", "تطبيق رفيق macos", "windows wsl2", "خدمة systemd لنظام linux", "العقد المحمولة", "نظرة عامة على المنصة"]
---

  نظرة عامة على المنصات

  
# المنصات

نواة OpenClaw مكتوبة بلغة TypeScript. **بيئة Node هي بيئة التشغيل الموصى بها**. لا يُنصح باستخدام Bun لخدمة Gateway (بسبب أخطاء في WhatsApp/Telegram). تتوفر تطبيقات رفيقة لنظام macOS (تطبيق شريط القوائم) والعقد المحمولة (iOS/Android). تطبيقات رفيقة لأنظمة Windows و Linux قيد التخطيط، ولكن خدمة Gateway مدعومة بالكامل اليوم. تطبيقات رفيقة أصلية لأنظمة Windows قيد التخطيط أيضًا؛ يُوصى باستخدام Gateway عبر WSL2.

## اختر نظام التشغيل الخاص بك

-   macOS: [macOS](./platforms/macos.md)
-   iOS: [iOS](./platforms/ios.md)
-   Android: [Android](./platforms/android.md)
-   Windows: [Windows](./platforms/windows.md)
-   Linux: [Linux](./platforms/linux.md)

## VPS والاستضافة

-   مركز VPS: [استضافة VPS](./vps.md)
-   Fly.io: [Fly.io](./install/fly.md)
-   Hetzner (Docker): [Hetzner](./install/hetzner.md)
-   GCP (Compute Engine): [GCP](./install/gcp.md)
-   exe.dev (VM + وكيل HTTPS): [exe.dev](./install/exe-dev.md)

## روابط شائعة

-   دليل التثبيت: [بدء الاستخدام](./start/getting-started.md)
-   دليل تشغيل Gateway: [Gateway](./gateway.md)
-   تهيئة Gateway: [التكوين](./gateway/configuration.md)
-   حالة الخدمة: `openclaw gateway status`

## تثبيت خدمة Gateway (واجهة سطر الأوامر)

استخدم إحدى الطرق التالية (جميعها مدعومة):

-   المعالج (موصى به): `openclaw onboard --install-daemon`
-   مباشر: `openclaw gateway install`
-   تدفق التهيئة: `openclaw configure` → اختر **خدمة Gateway**
-   الإصلاح/الترحيل: `openclaw doctor` (يعرض تثبيت الخدمة أو إصلاحها)

هدف الخدمة يعتمد على نظام التشغيل:

-   macOS: LaunchAgent (`ai.openclaw.gateway` أو `ai.openclaw.`؛ الإصدار القديم `com.openclaw.*`)
-   Linux/WSL2: خدمة مستخدم systemd (`openclaw-gateway[-].service`)

[تطبيق macOS](./platforms/macos.md)

---