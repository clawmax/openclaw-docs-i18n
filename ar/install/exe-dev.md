title: "نشر بوابة OpenClaw على الجهاز الظاهري exe.dev"
description: "دليل خطوة بخطوة لتثبيت وتشغيل بوابة الذكاء الاصطناعي OpenClaw على جهاز ظاهري من exe.dev. تعرف على الإعداد الآلي واليدوي للوصول عن بعد عبر exe.xyz."
keywords: ["openclaw", "exe.dev", "نشر البوابة", "جهاز ظاهري", "وكيل nginx", "وصول عن بعد", "تثبيت آلي", "تثبيت يدوي"]
---

  الاستضافة والنشر

  
# exe.dev

الهدف: تشغيل بوابة OpenClaw على جهاز ظاهري من exe.dev، يمكن الوصول إليها من حاسوبك المحمول عبر: `https://<vm-name>.exe.xyz` تفترض هذه الصفحة صورة **exeuntu** الافتراضية لـ exe.dev. إذا اخترت توزيعة مختلفة، فقم بتعيين الحزم وفقًا لذلك.

## المسار السريع للمبتدئين

1.  [https://exe.new/openclaw](https://exe.new/openclaw)
2.  املأ مفتاح/رمز المصادقة الخاص بك حسب الحاجة
3.  انقر على "Agent" بجوار الجهاز الظاهري الخاص بك، وانتظر…
4.  ؟؟؟
5.  اربح

## ما تحتاجه

-   حساب على exe.dev
-   وصول `ssh exe.dev` إلى الأجهزة الظاهرية على [exe.dev](https://exe.dev) (اختياري)

## التثبيت الآلي باستخدام Shelley

يمكن لـ Shelley، [وكيل exe.dev](https://exe.dev)، تثبيت OpenClaw فورًا باستخدام مطالبتنا. المطالبة المستخدمة هي كما يلي:

```bash
Set up OpenClaw (https://docs.openclaw.ai/install) on this VM. Use the non-interactive and accept-risk flags for openclaw onboarding. Add the supplied auth or token as needed. Configure nginx to forward from the default port 18789 to the root location on the default enabled site config, making sure to enable Websocket support. Pairing is done by "openclaw devices list" and "openclaw devices approve <request id>". Make sure the dashboard shows that OpenClaw's health is OK. exe.dev handles forwarding from port 8000 to port 80/443 and HTTPS for us, so the final "reachable" should be <vm-name>.exe.xyz, without port specification.
```

## التثبيت اليدوي

## 1) إنشاء الجهاز الظاهري

من جهازك:

```bash
ssh exe.dev new
```

ثم اتصل:

```bash
ssh <vm-name>.exe.xyz
```

نصيحة: احتفظ بهذا الجهاز الظاهري **بحالة ثابتة**. يقوم OpenClaw بتخزين الحالة تحت `~/.openclaw/` و `~/.openclaw/workspace/`.

## 2) تثبيت المتطلبات الأساسية (على الجهاز الظاهري)

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```

## 3) تثبيت OpenClaw

شغّل نص تثبيت OpenClaw:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

## 4) إعداد nginx لتمرير OpenClaw إلى المنفذ 8000

قم بتحرير `/etc/nginx/sites-enabled/default` باستخدام

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeout settings for long-lived connections
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

## 5) الوصول إلى OpenClaw ومنح الصلاحيات

اذهب إلى `https://<vm-name>.exe.xyz/` (انظر ناتج واجهة التحكم من عملية الإعداد). إذا طلب المصادقة، الصق الرمز من `gateway.auth.token` على الجهاز الظاهري (استرجعه باستخدام `openclaw config get gateway.auth.token`، أو أنشئ واحدًا باستخدام `openclaw doctor --generate-gateway-token`). قم بالموافقة على الأجهزة باستخدام `openclaw devices list` و `openclaw devices approve `. عند الشك، استخدم Shelley من متصفحك!

## الوصول عن بعد

يتم التعامل مع الوصول عن بعد بواسطة مصادقة [exe.dev](https://exe.dev). بشكل افتراضي، يتم توجيه حركة مرور HTTP من المنفذ 8000 إلى `https://<vm-name>.exe.xyz` مع مصادقة البريد الإلكتروني.

## التحديث

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

دليل: [التحديث](./updating.md)

[أجهزة macOS الظاهرية](./macos-vm.md)[النشر على Railway](./railway.md)