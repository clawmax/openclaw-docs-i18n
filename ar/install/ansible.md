

  طرق تثبيت أخرى

  
# Ansible

الطريقة الموصى بها لنشر OpenClaw على خوادم الإنتاج هي عبر **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — وهو مثبت آلي بهندسة معمارية تعطي الأولوية للأمان.

## البدء السريع

تثبيت بأمر واحد:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 الدليل الكامل: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** مستودع openclaw-ansible هو المصدر الموثوق لنشر Ansible. هذه الصفحة هي نظرة عامة سريعة.

## ما الذي ستحصل عليه

-   🔒 **أمان يعتمد أولاً على جدار الحماية**: UFW + عزل Docker (فقط SSH + Tailscale يمكن الوصول إليهما)
-   🔐 **شبكة VPN خاصة Tailscale**: وصول آمن عن بُعد دون تعرض الخدمات للعامة
-   🐳 **Docker**: حاويات معزولة (sandbox)، وربط محلي فقط (localhost-only bindings)
-   🛡️ **دفاع متعدد الطبقات**: هندسة أمان من 4 طبقات
-   🚀 **إعداد بأمر واحد**: نشر كامل في دقائق
-   🔧 **تكامل مع Systemd**: بدء تلقائي عند الإقلاع مع تعزيزات أمنية

## المتطلبات

-   **نظام التشغيل**: Debian 11+ أو Ubuntu 20.04+
-   **الوصول**: صلاحيات root أو sudo
-   **الشبكة**: اتصال بالإنترنت لتثبيت الحزم
-   **Ansible**: 2.14+ (يتم تثبيته تلقائيًا بواسطة سكريبت البدء السريع)

## ما الذي يتم تثبيته

يقوم playbook الخاص بـ Ansible بتثبيت وتكوين:

1.  **Tailscale** (شبكة VPN mesh للوصول الآمن عن بُعد)
2.  **جدار الحماية UFW** (فقط منافذ SSH + Tailscale)
3.  **Docker CE + Compose V2** (لصناديق العزل الخاصة بالوكلاء)
4.  **Node.js 22.x + pnpm** (تبعيات وقت التشغيل)
5.  **OpenClaw** (يعمل على المضيف مباشرة، وليس داخل حاوية)
6.  **خدمة Systemd** (بدء تلقائي مع تعزيزات أمنية)

ملاحظة: تعمل **البوابة مباشرة على المضيف** (وليس داخل Docker)، لكن صناديق عزل الوكلاء تستخدم Docker للعزل. راجع [العزل](../gateway/sandboxing.md) للتفاصيل.

## الإعداد بعد التثبيت

بعد اكتمال التثبيت، انتقل إلى مستخدم openclaw:

```bash
sudo -i -u openclaw
```

سيرشدك سكريبت ما بعد التثبيت خلال:

1.  **معالج الإعداد الأولي**: تكوين إعدادات OpenClaw
2.  **تسجيل الدخول إلى مزود الخدمة**: ربط WhatsApp/Telegram/Discord/Signal
3.  **اختبار البوابة**: التحقق من التثبيت
4.  **إعداد Tailscale**: الاتصال بشبكة VPN mesh الخاصة بك

### أوامر سريعة

```bash
# التحقق من حالة الخدمة
sudo systemctl status openclaw

# عرض السجلات الحية
sudo journalctl -u openclaw -f

# إعادة تشغيل البوابة
sudo systemctl restart openclaw

# تسجيل الدخول إلى مزود الخدمة (تشغيل كمستخدم openclaw)
sudo -i -u openclaw
openclaw channels login
```

## الهندسة المعمارية للأمان

### دفاع من 4 طبقات

1.  **جدار الحماية (UFW)**: فقط SSH (22) + Tailscale (41641/udp) معرضان للعامة
2.  **شبكة VPN (Tailscale)**: يمكن الوصول إلى البوابة فقط عبر شبكة VPN mesh
3.  **عزل Docker**: سلسلة iptables الخاصة بـ DOCKER-USER تمنع تعرض المنافذ الخارجية
4.  **تعزيز Systemd**: NoNewPrivileges, PrivateTmp, مستخدم بدون صلاحيات مميزة

### التحقق

اختبر السطح المعرض للهجوم من الخارج:

```bash
nmap -p- YOUR_SERVER_IP
```

يجب أن يظهر **فقط المنفذ 22** (SSH) مفتوحًا. جميع الخدمات الأخرى (البوابة، Docker) مقفلة.

### توفر Docker

يتم تثبيت Docker من أجل **صناديق عزل الوكلاء** (تنفيذ أدوات معزول)، وليس لتشغيل البوابة نفسها. تربط البوابة نفسها بـ localhost فقط ويمكن الوصول إليها عبر شبكة VPN الخاصة بـ Tailscale. راجع [صندوق عزل متعدد الوكلاء والأدوات](../tools/multi-agent-sandbox-tools.md) لتكوين صندوق العزل.

## التثبيت اليدوي

إذا كنت تفضل التحكم اليدوي بدلاً عن الأتمتة:

```bash
# 1. تثبيت المتطلبات الأساسية
sudo apt update && sudo apt install -y ansible git

# 2. استنساخ المستودع
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. تثبيت مجموعات Ansible
ansible-galaxy collection install -r requirements.yml

# 4. تشغيل playbook
./run-playbook.sh

# أو التشغيل مباشرة (ثم تنفيذ /tmp/openclaw-setup.sh يدويًا بعد ذلك)
# ansible-playbook playbook.yml --ask-become-pass
```

## تحديث OpenClaw

يقوم مثبت Ansible بإعداد OpenClaw للتحديثات اليدوية. راجع [التحديث](./updating.md) لسير العمل القياسي للتحديث. لإعادة تشغيل playbook الخاص بـ Ansible (على سبيل المثال، لتغييرات التكوين):

```bash
cd openclaw-ansible
./run-playbook.sh
```

ملاحظة: هذا أمر آمن للتشغيل المتكرر (idempotent).

## استكشاف الأخطاء وإصلاحها

### جدار الحماية يحجب اتصالي

إذا تم حظر وصولك:

-   تأكد أولاً من إمكانية الوصول عبر شبكة VPN الخاصة بـ Tailscale
-   الوصول عبر SSH (المنفذ 22) مسموح به دائمًا
-   البوابة **فقط** يمكن الوصول إليها عبر Tailscale حسب التصميم

### الخدمة لا تبدأ

```bash
# التحقق من السجلات
sudo journalctl -u openclaw -n 100

# التحقق من الصلاحيات
sudo ls -la /opt/openclaw

# اختبار البدء اليدوي
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### مشاكل في صندوق عزل Docker

```bash
# التحقق من أن Docker يعمل
sudo systemctl status docker

# التحقق من صورة صندوق العزل
sudo docker images | grep openclaw-sandbox

# بناء صورة صندوق العزل إذا كانت مفقودة
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### فشل تسجيل الدخول إلى مزود الخدمة

تأكد من أنك تعمل كمستخدم `openclaw`:

```bash
sudo -i -u openclaw
openclaw channels login
```

## التكوين المتقدم

للحصول على تفاصيل الهندسة المعمارية للأمان واستكشاف الأخطاء:

-   [الهندسة المعمارية للأمان](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
-   [التفاصيل التقنية](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
-   [دليل استكشاف الأخطاء وإصلاحها](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## ذات صلة

-   [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — دليل النشر الكامل
-   [Docker](./docker.md) — إعداد البوابة داخل حاوية
-   [العزل](../gateway/sandboxing.md) — تكوين صندوق عزل الوكلاء
-   [صندوق عزل متعدد الوكلاء والأدوات](../tools/multi-agent-sandbox-tools.md) — عزل لكل وكيل

[Nix](./nix.md)[Bun (تجريبي)](./bun.md)