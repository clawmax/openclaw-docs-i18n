

  نظرة عامة على المنصات

  
# تطبيق لينكس

البوابة مدعومة بالكامل على لينكس. **Node هو بيئة التشغيل الموصى بها**. لا يُنصح باستخدام Bun للبوابة (بسبب أخطاء في واتساب/تيليجرام). تطبيقات لينكس الأصلية المرافقة قيد التخطيط. المساهمات مرحب بها إذا كنت ترغب في المساعدة في بناء واحد.

## المسار السريع للمبتدئين (VPS)

1.  ثبّت Node 22+
2.  `npm i -g openclaw@latest`
3.  `openclaw onboard --install-daemon`
4.  من حاسوبك المحمول: `ssh -N -L 18789:127.0.0.1:18789 @`
5.  افتح `http://127.0.0.1:18789/` والصق رمزك (token)

دليل VPS خطوة بخطوة: [exe.dev](../install/exe-dev.md)

## التثبيت

-   [بدء الاستخدام](../start/getting-started.md)
-   [التثبيت والتحديثات](../install/updating.md)
-   مسارات اختيارية: [Bun (تجريبي)](../install/bun.md), [Nix](../install/nix.md), [Docker](../install/docker.md)

## البوابة

-   [كتيب تشغيل البوابة](../gateway.md)
-   [التكوين](../gateway/configuration.md)

## تثبيت خدمة البوابة (CLI)

استخدم أحد هذه الأوامر:

```bash
openclaw onboard --install-daemon
```

أو:

```bash
openclaw gateway install
```

أو:

```bash
openclaw configure
```

اختر **خدمة البوابة** عند المطالبة. للإصلاح/الترحيل:

```bash
openclaw doctor
```

## التحكم بالنظام (وحدة systemd للمستخدم)

يقوم OpenClaw بتثبيت خدمة **مستخدم** systemd افتراضيًا. استخدم خدمة **نظام** للخوادم المشتركة أو العاملة دائمًا. مثال الوحدة الكامل والإرشادات موجودة في [كتيب تشغيل البوابة](../gateway.md). الإعداد الأدنى: أنشئ `~/.config/systemd/user/openclaw-gateway[-].service`:

```ini
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

فعّلها:

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

[تطبيق macOS](./macos.md)[Windows (WSL2)](./windows.md)