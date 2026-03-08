title: "تشغيل بوابة OpenClaw على الطبقة المجانية الدائمة ARM من Oracle Cloud"
description: "تعلم كيفية نشر وتشغيل بوابة OpenClaw دائمة على الطبقة المجانية الدائمة ARM من Oracle Cloud، بما في ذلك خطوات الإعداد والأمان واستكشاف الأخطاء وإصلاحها."
keywords: ["openclaw oracle", "oracle cloud الطبقة المجانية", "بوابة openclaw", "نشر خادم arm", "tailscale oracle", "تثبيت openclaw", "oci مجاني دائم", "أمان openclaw"]
---

  نظرة عامة على المنصات

  
# Oracle Cloud

## الهدف

تشغيل بوابة OpenClaw دائمة على الطبقة **المجانية الدائمة** ARM من Oracle Cloud. يمكن أن تكون الطبقة المجانية من Oracle مناسبة جدًا لـ OpenClaw (خاصة إذا كان لديك حساب OCI بالفعل)، لكنها تأتي مع مقايضات:

-   بنية ARM (معظم الأشياء تعمل، لكن بعض الملفات الثنائية قد تكون مخصصة لـ x86 فقط)
-   قد تكون السعة والتسجيل صعبة بعض الشيء

## مقارنة التكلفة (2026)

| المزود | الخطة | المواصفات | السعر/شهر | ملاحظات |
| --- | --- | --- | --- | --- |
| Oracle Cloud | Always Free ARM | حتى 4 OCPU، 24 جيجابايت ذاكرة وصول عشوائي | $0 | ARM، سعة محدودة |
| Hetzner | CX22 | 2 وحدة معالجة مركزية افتراضية، 4 جيجابايت ذاكرة وصول عشوائي | ~ $4 | أرخص خطة مدفوعة |
| DigitalOcean | Basic | 1 وحدة معالجة مركزية افتراضية، 1 جيجابايت ذاكرة وصول عشوائي | $6 | واجهة مستخدم سهلة، وثائق جيدة |
| Vultr | Cloud Compute | 1 وحدة معالجة مركزية افتراضية، 1 جيجابايت ذاكرة وصول عشوائي | $6 | مواقع عديدة |
| Linode | Nanode | 1 وحدة معالجة مركزية افتراضية، 1 جيجابايت ذاكرة وصول عشوائي | $5 | الآن جزء من Akamai |

* * *

## المتطلبات الأساسية

-   حساب Oracle Cloud ([التسجيل](https://www.oracle.com/cloud/free/)) — راجع [دليل التسجيل المجتمعي](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) إذا واجهت مشاكل
-   حساب Tailscale (مجاني على [tailscale.com](https://tailscale.com))
-   ~30 دقيقة

## 1) إنشاء مثيل OCI

1.  سجل الدخول إلى [Oracle Cloud Console](https://cloud.oracle.com/)
2.  انتقل إلى **Compute → Instances → Create Instance**
3.  قم بالتكوين:
    -   **الاسم:** `openclaw`
    -   **الصورة:** Ubuntu 24.04 (aarch64)
    -   **الشكل:** `VM.Standard.A1.Flex` (Ampere ARM)
    -   **OCPUs:** 2 (أو حتى 4)
    -   **الذاكرة:** 12 جيجابايت (أو حتى 24 جيجابايت)
    -   **حجم التمهيد:** 50 جيجابايت (حتى 200 جيجابايت مجانًا)
    -   **مفتاح SSH:** أضف مفتاحك العام
4.  انقر على **Create**
5.  لاحظ عنوان IP العام

**نصيحة:** إذا فشل إنشاء المثيل مع رسالة "Out of capacity"، جرب نطاق توفر مختلف أو أعد المحاولة لاحقًا. سعة الطبقة المجانية محدودة.

## 2) الاتصال والتحديث

```bash
# الاتصال عبر عنوان IP العام
ssh ubuntu@YOUR_PUBLIC_IP

# تحديث النظام
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**ملاحظة:** `build-essential` مطلوب لتجميع بعض التبعيات لبنية ARM.

## 3) تكوين المستخدم واسم المضيف

```bash
# تعيين اسم المضيف
sudo hostnamectl set-hostname openclaw

# تعيين كلمة مرور لمستخدم ubuntu
sudo passwd ubuntu

# تمكين lingering (يحافظ على تشغيل خدمات المستخدم بعد تسجيل الخروج)
sudo loginctl enable-linger ubuntu
```

## 4) تثبيت Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

هذا يمكّن Tailscale SSH، بحيث يمكنك الاتصال عبر `ssh openclaw` من أي جهاز على شبكتك الذيلية (tailnet) — لا حاجة لعنوان IP عام. تحقق:

```bash
tailscale status
```

**من الآن فصاعدًا، اتصل عبر Tailscale:** `ssh ubuntu@openclaw` (أو استخدم عنوان IP الخاص بـ Tailscale).

## 5) تثبيت OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
source ~/.bashrc
```

عند المطالبة بـ "How do you want to hatch your bot?"، اختر **"Do this later"**.

> ملاحظة: إذا واجهت مشاكل في بناء الملفات الثنائية الأصلية لـ ARM، ابدأ بحزم النظام (مثل `sudo apt install -y build-essential`) قبل اللجوء إلى Homebrew.

## 6) تكوين البوابة (loopback + مصادقة الرمز) وتمكين Tailscale Serve

استخدم مصادقة الرمز كإعداد افتراضي. إنها متوقعة وتجنب الحاجة إلى أي علامات "مصادقة غير آمنة" في واجهة التحكم.

```bash
# احتفظ بالبوابة خاصة على الجهاز الظاهري
openclaw config set gateway.bind loopback

# اشترط المصادقة للبوابة + واجهة التحكم
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# عرضها عبر Tailscale Serve (HTTPS + وصول عبر tailnet)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

## 7) التحقق

```bash
# التحقق من الإصدار
openclaw --version

# التحقق من حالة الخدمة
systemctl --user status openclaw-gateway

# التحقق من Tailscale Serve
tailscale serve status

# اختبار الاستجابة المحلية
curl http://localhost:18789
```

## 8) تأمين شبكة VCN

الآن بعد أن أصبح كل شيء يعمل، قم بتأمين شبكة VCN لحظر كل حركة المرور باستثناء Tailscale. تعمل شبكة Oracle السحابية الافتراضية (VCN) كجدار حماية عند حافة الشبكة — يتم حظر حركة المرور قبل وصولها إلى مثيلك.

1.  انتقل إلى **Networking → Virtual Cloud Networks** في وحدة تحكم OCI
2.  انقر على شبكة VCN الخاصة بك → **Security Lists** → Default Security List
3.  **أزل** جميع قواعد الوارد (ingress) باستثناء:
    -   `0.0.0.0/0 UDP 41641` (Tailscale)
4.  احتفظ بقواعد الصادر (egress) الافتراضية (السماح بكل الصادر)

هذا يحظر SSH على المنفذ 22، وHTTP، وHTTPS، وكل شيء آخر عند حافة الشبكة. من الآن فصاعدًا، يمكنك الاتصال فقط عبر Tailscale.

* * *

## الوصول إلى واجهة التحكم

من أي جهاز على شبكة Tailscale الخاصة بك:

```
https://openclaw.<tailnet-name>.ts.net/
```

استبدل `<tailnet-name>` باسم شبكتك الذيلية (مرئي في `tailscale status`). لا حاجة لنفق SSH. يوفر Tailscale:

-   تشفير HTTPS (شهادات تلقائية)
-   المصادقة عبر هوية Tailscale
-   الوصول من أي جهاز على شبكتك الذيلية (لابتوب، هاتف، إلخ.)

* * *

## الأمان: VCN + Tailscale (الخط الأساسي الموصى به)

مع تأمين شبكة VCN (فقط UDP 41641 مفتوح) وربط البوابة بـ loopback، تحصل على دفاع قوي متعدد الطبقات: يتم حظر حركة المرور العامة عند حافة الشبكة، ويحدث الوصول الإداري عبر شبكتك الذيلية. غالبًا ما يزيل هذا الإعداد *الحاجة* إلى قواعد جدار حماية إضافية على مستوى المضيف فقط لوقف هجمات القوة الغاشمة على SSH عبر الإنترنت — لكن يجب أن تحافظ على تحديث نظام التشغيل، وتشغل `openclaw security audit`، وتتحقق من أنك لا تستمع عن طريق الخطأ على واجهات عامة.

### ما تم حمايته بالفعل

| الخطوة التقليدية | مطلوبة؟ | السبب |
| --- | --- | --- |
| جدار حماية UFW | لا | شبكة VCN تحظر قبل وصول حركة المرور إلى المثيل |
| fail2ban | لا | لا هجمات قوة غاشمة إذا تم حظر المنفذ 22 على مستوى VCN |
| تعزيز أمان sshd | لا | Tailscale SSH لا يستخدم sshd |
| تعطيل تسجيل دخول الجذر | لا | Tailscale يستخدم هوية Tailscale، وليس مستخدمي النظام |
| مصادقة مفتاح SSH فقط | لا | Tailscale يصادق عبر شبكتك الذيلية |
| تعزيز أمان IPv6 | عادة لا | يعتمد على إعدادات VCN/الشبكة الفرعية الخاصة بك؛ تحقق مما تم تعيينه/عرضه فعليًا |

### لا يزال موصى به

-   **أذونات بيانات الاعتماد:** `chmod 700 ~/.openclaw`
-   **مراجعة الأمان:** `openclaw security audit`
-   **تحديثات النظام:** `sudo apt update && sudo apt upgrade` بانتظام
-   **مراقبة Tailscale:** راجع الأجهزة في [وحدة تحكم إدارة Tailscale](https://login.tailscale.com/admin)

### التحقق من وضع الأمان

```bash
# تأكيد عدم وجود منافذ عامة تستمع
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# التحقق من أن Tailscale SSH نشط
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# اختياري: تعطيل sshd تمامًا
sudo systemctl disable --now ssh
```

* * *

## الاحتياطي: نفق SSH

إذا لم يعمل Tailscale Serve، استخدم نفق SSH:

```bash
# من جهازك المحلي (عبر Tailscale)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

ثم افتح `http://localhost:18789`.

* * *

## استكشاف الأخطاء وإصلاحها

### فشل إنشاء المثيل ("Out of capacity")

مثيلات ARM في الطبقة المجانية شائعة. جرب:

-   نطاق توفر مختلف
-   إعادة المحاولة خلال ساعات غير الذروة (الصباح الباكر)
-   استخدم عامل التصفية "Always Free" عند اختيار الشكل

### Tailscale لا يتصل

```bash
# التحقق من الحالة
sudo tailscale status

# إعادة المصادقة
sudo tailscale up --ssh --hostname=openclaw --reset
```

### البوابة لا تبدأ

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

### لا يمكن الوصول إلى واجهة التحكم

```bash
# التحقق من أن Tailscale Serve يعمل
tailscale serve status

# التحقق من أن البوابة تستمع
curl http://localhost:18789

# إعادة التشغيل إذا لزم الأمر
systemctl --user restart openclaw-gateway
```

### مشاكل في الملفات الثنائية لـ ARM

قد لا تحتوي بعض الأدوات على إصدارات لـ ARM. تحقق:

```bash
uname -m  # يجب أن يظهر aarch64
```

معظم حزم npm تعمل بشكل جيد. بالنسبة للملفات الثنائية، ابحث عن إصدارات `linux-arm64` أو `aarch64`.

* * *

## الاستمرارية

كل الحالة موجودة في:

-   `~/.openclaw/` — التكوين، بيانات الاعتماد، بيانات الجلسة
-   `~/.openclaw/workspace/` — مساحة العمل (SOUL.md، الذاكرة، القطع الأثرية)

احتفظ بنسخة احتياطية بشكل دوري:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## انظر أيضًا

-   [الوصول عن بعد للبوابة](../gateway/remote.md) — أنماط أخرى للوصول عن بعد
-   [تكامل Tailscale](../gateway/tailscale.md) — وثائق Tailscale الكاملة
-   [تكوين البوابة](../gateway/configuration.md) — جميع خيارات التكوين
-   [دليل DigitalOcean](./digitalocean.md) — إذا كنت تريد خطة مدفوعة + تسجيل أسهل
-   [دليل Hetzner](../install/hetzner.md) — بديل قائم على Docker

[DigitalOcean](./digitalocean.md)[Raspberry Pi](./raspberry-pi.md)

---