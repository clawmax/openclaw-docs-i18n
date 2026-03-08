

  طرق تثبيت أخرى

  
# Nix

الطريقة الموصى بها لتشغيل OpenClaw مع Nix هي عبر **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — وحدة Home Manager شاملة وجاهزة للاستخدام.

## البدء السريع

الصق هذا في وكيل الذكاء الاصطناعي الخاص بك (Claude، Cursor، إلخ):

```
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, model provider API key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 الدليل الكامل: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)** مستودع nix-openclaw هو المصدر الموثوق لتثبيت Nix. هذه الصفحة هي مجرد نظرة عامة سريعة.

## ما الذي ستحصل عليه

-   البوابة + تطبيق macOS + الأدوات (whisper، spotify، الكاميرات) — جميعها مثبتة ومثبتة الإصدار
-   خدمة Launchd تبقى قيد التشغيل بعد إعادة التشغيل
-   نظام إضافات مع تكوين تصريحي
-   التراجع الفوري: `home-manager switch --rollback`

* * *

## سلوك وقت التشغيل لوضع Nix

عند تعيين `OPENCLAW_NIX_MODE=1` (يتم ذلك تلقائيًا مع nix-openclaw): يدعم OpenClaw **وضع Nix** الذي يجعل التكوين حتميًا ويعطل عمليات التثبيت التلقائي. قم بتمكينه عن طريق التصدير:

```
OPENCLAW_NIX_MODE=1
```

على نظام macOS، لا يرث تطبيق واجهة المستخدم الرسومية متغيرات بيئة الطرفية تلقائيًا. يمكنك أيضًا تمكين وضع Nix عبر defaults:

```bash
defaults write ai.openclaw.mac openclaw.nixMode -bool true
```

### مسارات التكوين + الحالة

يقرأ OpenClaw تكوين JSON5 من `OPENCLAW_CONFIG_PATH ويخزن البيانات القابلة للتعديل في `OPENCLAW_STATE_DIR`. عند الحاجة، يمكنك أيضًا تعيين `OPENCLAW_HOME` للتحكم في دليل المنزل الأساسي المستخدم لحل المسارات الداخلية.

-   `OPENCLAW_HOME` (الأولوية الافتراضية: `HOME` / `USERPROFILE` / `os.homedir()`)
-   `OPENCLAW_STATE_DIR` (الافتراضي: `~/.openclaw`)
-   `OPENCLAW_CONFIG_PATH` (الافتراضي: `$OPENCLAW_STATE_DIR/openclaw.json`)

عند التشغيل تحت Nix، قم بتعيين هذه المواضع بشكل صريح إلى مواقع تديرها Nix بحيث تبقى حالة التشغيل والتكوين خارج المخزن الثابت.

### سلوك وقت التشغيل في وضع Nix

-   يتم تعطيل عمليات التثبيت التلقائي والتحول الذاتي
-   تظهر رسائل علاجية خاصة بـ Nix للتبعيات المفقودة
-   تظهر واجهة المستخدم لافتة وضع Nix للقراءة فقط عند وجوده

## ملاحظة التعبئة والتغليف (macOS)

يتوقع تدفق تعبئة تطبيقات macOS قالب Info.plist ثابت في:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

يقوم [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) بنسخ هذا القالب إلى حزمة التطبيق وترقيع الحقول الديناميكية (معرف الحزمة، الإصدار/البناء، Git SHA، مفاتيح Sparkle). هذا يحافظ على ملف plist حتميًا لتعبئة SwiftPM وبناءات Nix (التي لا تعتمد على مجموعة أدوات Xcode كاملة).

## ذات صلة

-   [nix-openclaw](https://github.com/openclaw/nix-openclaw) — دليل الإعداد الكامل
-   [المساعد](../start/wizard.md) — إعداد CLI بدون Nix
-   [Docker](./docker.md) — إعداد معتمد على الحاويات

[Podman](./podman.md)[Ansible](./ansible.md)

---