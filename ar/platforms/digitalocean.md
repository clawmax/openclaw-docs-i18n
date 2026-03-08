

  نظرة عامة على المنصات

  
# DigitalOcean

## الهدف

تشغيل OpenClaw Gateway دائم على DigitalOcean مقابل **6 دولارات/شهر** (أو 4 دولارات/شهر مع التسعير المحجوز). إذا كنت تريد خيارًا بـ 0 دولار/شهر ولا تمانع في ARM والإعداد الخاص بالمزود، راجع [دليل Oracle Cloud](./oracle.md).

## مقارنة التكلفة (2026)

| المزود | الخطة | المواصفات | السعر/شهر | ملاحظات |
| --- | --- | --- | --- | --- |
| Oracle Cloud | Always Free ARM | حتى 4 OCPU، 24GB RAM | $0 | ARM، سعة محدودة / خصوصيات في التسجيل |
| Hetzner | CX22 | 2 vCPU، 4GB RAM | €3.79 (~$4) | أرخص خيار مدفوع |
| DigitalOcean | Basic | 1 vCPU، 1GB RAM | $6 | واجهة مستخدم بسيطة، وثائق جيدة |
| Vultr | Cloud Compute | 1 vCPU، 1GB RAM | $6 | مواقع عديدة |
| Linode | Nanode | 1 vCPU، 1GB RAM | $5 | الآن جزء من Akamai |

**اختيار مزود:**

-   DigitalOcean: أبسط تجربة مستخدم + إعداد متوقع (هذا الدليل)
-   Hetzner: سعر/أداء جيد (انظر [دليل Hetzner](../install/hetzner.md))
-   Oracle Cloud: يمكن أن يكون 0 دولار/شهر، ولكنه أكثر تعقيدًا ومقتصر على ARM (انظر [دليل Oracle](./oracle.md))

* * *

## المتطلبات الأساسية

-   حساب DigitalOcean ([سجل بـ 200 دولار رصيد مجاني](https://m.do.co/c/signup))
-   زوج مفاتيح SSH (أو الاستعداد لاستخدام المصادقة بكلمة المرور)
-   ~20 دقيقة

## 1) إنشاء Droplet

> **⚠️** استخدم صورة أساسية نظيفة (Ubuntu 24.04 LTS). تجنب صور السوق الجاهزة من أطراف ثالثة ما لم تكن قد راجعت نصوص التشغيل الخاصة بها وإعدادات الجدار الناري الافتراضية.

1.  سجّل الدخول إلى [DigitalOcean](https://cloud.digitalocean.com/)
2.  انقر على **Create → Droplets**
3.  اختر:
    -   **المنطقة:** الأقرب إليك (أو إلى مستخدميك)
    -   **الصورة:** Ubuntu 24.04 LTS
    -   **الحجم:** Basic → Regular → **6 دولارات/شهر** (1 vCPU، 1GB RAM، 25GB SSD)
    -   **المصادقة:** مفتاح SSH (موصى به) أو كلمة مرور
4.  انقر على **Create Droplet**
5.  لاحظ عنوان IP

## 2) الاتصال عبر SSH

```bash
ssh root@YOUR_DROPLET_IP
```

## 3) تثبيت OpenClaw

```bash
# Update system
apt update && apt upgrade -y

# Install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Install OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# Verify
openclaw --version
```

## 4) تشغيل الإعداد الأولي

```bash
openclaw onboard --install-daemon
```

سيرشدك المعالج خلال:

-   مصادقة النموذج (مفاتيح API أو OAuth)
-   إعداد القنوات (Telegram، WhatsApp، Discord، إلخ.)
-   رمز البوابة (يتم إنشاؤه تلقائيًا)
-   تثبيت Daemon (systemd)

## 5) التحقق من البوابة

```bash
# Check status
openclaw status

# Check service
systemctl --user status openclaw-gateway.service

# View logs
journalctl --user -u openclaw-gateway.service -f
```

## 6) الوصول إلى لوحة التحكم

ترتبط البوابة بحلقة العودة افتراضيًا. للوصول إلى واجهة تحكم Control UI: **الخيار أ: نفق SSH (موصى به)**

```bash
# From your local machine
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# Then open: http://localhost:18789
```

**الخيار ب: Tailscale Serve (HTTPS، حلقة العودة فقط)**

```bash
# On the droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configure Gateway to use Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

افتح: `https:///` ملاحظات:

-   يحافظ Serve على البوابة مقصورة على حلقة العودة ويوثق حركة Control UI/WebSocket عبر رؤوس هوية Tailscale (تفترض المصادقة بدون رمز أن مضيف البوابة موثوق؛ واجهات برمجة تطبيقات HTTP لا تزال تتطلب رمز/كلمة مرور).
-   لطلب رمز/كلمة مرور بدلاً من ذلك، عيّن `gateway.auth.allowTailscale: false` أو استخدم `gateway.auth.mode: "password"`.

**الخيار ج: ربط Tailnet (بدون Serve)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

افتح: `http://<tailscale-ip>:18789` (مطلوب رمز).

## 7) توصيل قنواتك

### Telegram

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

### WhatsApp

```bash
openclaw channels login whatsapp
# Scan QR code
```

انظر [القنوات](../channels.md) لمزودين آخرين.

* * *

## تحسينات لـ 1GB RAM

لدى droplet بقيمة 6 دولارات فقط 1GB RAM. للحفاظ على سير العمل بسلاسة:

### إضافة مبادلة (موصى به)

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### استخدام نموذج أخف

إذا كنت تواجه مشاكل نفاد الذاكرة (OOMs)، فكّر في:

-   استخدام نماذج قائمة على API (Claude، GPT) بدلاً من النماذج المحلية
-   تعيين `agents.defaults.model.primary` إلى نموذج أصغر

### مراقبة الذاكرة

```
free -h
htop
```

* * *

## الاستمرارية

توجد جميع الحالات في:

-   `~/.openclaw/` — التكوين، بيانات الاعتماد، بيانات الجلسة
-   `~/.openclaw/workspace/` — مساحة العمل (SOUL.md، الذاكرة، إلخ.)

هذه تبقى بعد إعادة التشغيل. احفظ نسخة احتياطية منها بشكل دوري:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## بديل Oracle Cloud المجاني

تقدم Oracle Cloud حالات **Always Free** ARM أقوى بكثير من أي خيار مدفوع هنا — مقابل 0 دولار/شهر.

| ما تحصل عليه | المواصفات |
| --- | --- |
| **4 OCPUs** | ARM Ampere A1 |
| **24GB RAM** | أكثر من كافٍ |
| **200GB تخزين** | وحدة تخزين كتلية |
| **مجاني للأبد** | لا توجد رسوم على البطاقة الائتمانية |

**محاذير:**

-   قد يكون التسجيل معقدًا (أعد المحاولة إذا فشل)
-   بنية ARM — معظم الأشياء تعمل، لكن بعض الملفات الثنائية تحتاج إلى تجميع لـ ARM

للحصول على دليل الإعداد الكامل، انظر [Oracle Cloud](./oracle.md). للحصول على نصائح التسجيل واستكشاف أخطاء عملية التسجيل وإصلاحها، انظر هذا [الدليل المجتمعي](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd).

* * *

## استكشاف الأخطاء وإصلاحها

### البوابة لا تبدأ

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

### المنفذ قيد الاستخدام بالفعل

```bash
lsof -i :18789
kill <PID>
```

### نفاد الذاكرة

```bash
# Check memory
free -h

# Add more swap
# Or upgrade to $12/mo droplet (2GB RAM)
```

* * *

## انظر أيضًا

-   [دليل Hetzner](../install/hetzner.md) — أرخص، أقوى
-   [تثبيت Docker](../install/docker.md) — إعداد معزول
-   [Tailscale](../gateway/tailscale.md) — وصول آمن عن بُعد
-   [التكوين](../gateway/configuration.md) — مرجع التكوين الكامل

[iOS App](./ios.md)[Oracle Cloud](./oracle.md)

---