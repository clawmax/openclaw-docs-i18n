

  نظرة عامة على المنصات

  
# Raspberry Pi

## الهدف

تشغيل بوابة OpenClaw دائمة التشغيل ومتاحة دائمًا على Raspberry Pi بتكلفة لمرة واحدة تبلغ **~35-80 دولارًا** (بدون رسوم شهرية). مثالي لـ:

-   مساعد ذكي شخصي يعمل 24/7
-   مركز أتمتة المنزل
-   بوت Telegram/WhatsApp منخفض الطاقة ومتاح دائمًا

## متطلبات الأجهزة

| موديل Pi | الذاكرة العشوائية | يعمل؟ | ملاحظات |
| --- | --- | --- | --- |
| **Pi 5** | 4GB/8GB | ✅ الأفضل | الأسرع، موصى به |
| **Pi 4** | 4GB | ✅ جيد | النقطة المثالية لمعظم المستخدمين |
| **Pi 4** | 2GB | ✅ مقبول | يعمل، أضف مساحة مبادلة |
| **Pi 4** | 1GB | ⚠️ ضيق | ممكن مع مساحة مبادلة، إعدادات دنيا |
| **Pi 3B+** | 1GB | ⚠️ بطيء | يعمل ولكنه خامل |
| **Pi Zero 2 W** | 512MB | ❌ | غير موصى به |

**الحد الأدنى للمواصفات:** 1GB ذاكرة عشوائية، نواة واحدة، 500MB مساحة تخزين  
**الموصى به:** 2GB+ ذاكرة عشوائية، نظام تشغيل 64 بت، بطاقة SD سعة 16GB+ (أو SSD عبر USB)

## ما ستحتاجه

-   Raspberry Pi 4 أو 5 (2GB+ موصى به)
-   بطاقة MicroSD (16GB+) أو SSD عبر USB (أداء أفضل)
-   مزود طاقة (مزود الطاقة الرسمي لـ Pi موصى به)
-   اتصال شبكة (Ethernet أو WiFi)
-   ~30 دقيقة

## 1) تثبيت نظام التشغيل

استخدم **Raspberry Pi OS Lite (64-bit)** — لا حاجة لسطح المكتب لخادم بدون واجهة.

1.  حمل [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2.  اختر نظام التشغيل: **Raspberry Pi OS Lite (64-bit)**
3.  انقر على أيقونة الترس (⚙️) للإعداد المسبق:
    -   عيّن اسم المضيف: `gateway-host`
    -   فعّل SSH
    -   عيّن اسم المستخدم/كلمة المرور
    -   اضبط WiFi (إذا كنت لا تستخدم Ethernet)
4.  ثبّت على بطاقة SD / محرك USB الخاص بك
5.  أدخل Pi وشغله

## 2) الاتصال عبر SSH

```bash
ssh user@gateway-host
# أو استخدم عنوان IP
ssh user@192.168.x.x
```

## 3) إعداد النظام

```bash
# تحديث النظام
sudo apt update && sudo apt upgrade -y

# تثبيت الحزم الأساسية
sudo apt install -y git curl build-essential

# تعيين النطاق الزمني (مهم للمذكرات/المنبهات)
sudo timedatectl set-timezone America/Chicago  # غيّر إلى نطاقك الزمني
```

## 4) تثبيت Node.js 22 (ARM64)

```bash
# تثبيت Node.js عبر NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# التحقق
node --version  # يجب أن يظهر v22.x.x
npm --version
```

## 5) إضافة مساحة مبادلة (مهم لـ 2GB أو أقل)

مساحة المبادلة تمنع تعطل النظام بسبب نفاد الذاكرة:

```bash
# إنشاء ملف مبادلة سعة 2GB
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# جعله دائمًا
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# تحسين للذاكرة المنخفضة (تقليل swappiness)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## 6) تثبيت OpenClaw

### الخيار أ: التثبيت القياسي (موصى به)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### الخيار ب: التثبيت القابل للتعديل (للهواة)

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

التثبيت القابل للتعديل يمنحك وصولاً مباشرًا للسجلات والشفرة — مفيد لتصحيح الأخطاء الخاصة بـ ARM.

## 7) تشغيل الإعداد الأولي

```bash
openclaw onboard --install-daemon
```

اتبع المعالج:

1.  **وضع البوابة:** محلي
2.  **المصادقة:** مفاتيح API موصى بها (OAuth قد يكون معقدًا على Pi بدون واجهة)
3.  **القنوات:** Telegram هو الأسهل للبدء
4.  **الخدمة الخلفية:** نعم (systemd)

## 8) التحقق من التثبيت

```bash
# التحقق من الحالة
openclaw status

# التحقق من الخدمة
sudo systemctl status openclaw

# عرض السجلات
journalctl -u openclaw -f
```

## 9) الوصول إلى لوحة التحكم

بما أن Pi يعمل بدون واجهة، استخدم نفق SSH:

```bash
# من حاسوبك المحمول/السطحي
ssh -L 18789:localhost:18789 user@gateway-host

# ثم افتح في المتصفح
open http://localhost:18789
```

أو استخدم Tailscale للوصول الدائم:

```bash
# على الـ Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# تحديث الإعدادات
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

* * *

## تحسينات الأداء

### استخدام SSD عبر USB (تحسين كبير)

بطاقات SD بطيئة وتتلف. SSD عبر USB يحسن الأداء بشكل كبير:

```bash
# تحقق إذا كان التشغيل من USB
lsblk
```

راجع [دليل تشغيل Pi من USB](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) للإعداد.

### تسريع بدء تشغيل CLI (ذاكرة تخزين مؤقت لتجميع الوحدات)

على مضيفات Pi ذات الطاقة المنخفضة، فعّل ذاكرة التخزين المؤقت لتجميع وحدات Node لتكون عمليات تشغيل CLI المتكررة أسرع:

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF' # pragma: allowlist secret
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

ملاحظات:

-   `NODE_COMPILE_CACHE` يسرع التشغيل اللاحق (`status`, `health`, `--help`).
-   `/var/tmp` يتحمل إعادة التشغيل أفضل من `/tmp`.
-   `OPENCLAW_NO_RESPAWN=1` يتجنب تكلفة البدء الإضافية من إعادة تشغيل CLI الذاتي.
-   التشغيل الأول يسخن الذاكرة المؤقتة؛ التشغيل اللاحق يستفيد أكثر.

### ضبط بدء تشغيل systemd (اختياري)

إذا كان هذا الـ Pi يعمل بشكل أساسي على تشغيل OpenClaw، أضف خدمة drop-in لتقليل تقلبات إعادة التشغيل والحفاظ على بيئة بدء تشغيل مستقرة:

```bash
sudo systemctl edit openclaw
```

```ini
[Service]
Environment=OPENCLAW_NO_RESPAWN=1
Environment=NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
Restart=always
RestartSec=2
TimeoutStartSec=90
```

ثم طبق:

```bash
sudo systemctl daemon-reload
sudo systemctl restart openclaw
```

إذا أمكن، احتفظ بحالة/ذاكرة تخزين مؤقت لـ OpenClaw على تخزين مدعوم بـ SSD لتجنب اختناقات الإدخال/الإخراج العشوائي لبطاقة SD أثناء بدء التشغيل البارد. كيف تساعد سياسات `Restart=` في الاسترداد التلقائي: [systemd يمكنه أتمتة استرداد الخدمة](https://www.redhat.com/en/blog/systemd-automate-recovery).

### تقليل استخدام الذاكرة

```bash
# تعطيل تخصيص ذاكرة GPU (بدون واجهة)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# تعطيل Bluetooth إذا لم يكن مطلوبًا
sudo systemctl disable bluetooth
```

### مراقبة الموارد

```bash
# التحقق من الذاكرة
free -h

# التحقق من درجة حرارة المعالج
vcgencmd measure_temp

# المراقبة الحية
htop
```

* * *

## ملاحظات خاصة بـ ARM

### التوافق الثنائي

معظم ميزات OpenClaw تعمل على ARM64، ولكن بعض البرامج الثنائية الخارجية قد تحتاج إلى إصدارات ARM:

| الأداة | حالة ARM64 | ملاحظات |
| --- | --- | --- |
| Node.js | ✅ | يعمل بشكل ممتاز |
| WhatsApp (Baileys) | ✅ | JS خالص، لا مشاكل |
| Telegram | ✅ | JS خالص، لا مشاكل |
| gog (Gmail CLI) | ⚠️ | تحقق من إصدار ARM |
| Chromium (المتصفح) | ✅ | `sudo apt install chromium-browser` |

إذا فشلت مهارة، تحقق مما إذا كان برنامجها الثنائي يحتوي على إصدار ARM. العديد من أدوات Go/Rust تفعل؛ والبعض لا.

### 32 بت مقابل 64 بت

**استخدم دائمًا نظام تشغيل 64 بت.** Node.js والعديد من الأدوات الحديثة تتطلبه. تحقق باستخدام:

```bash
uname -m
# يجب أن يظهر: aarch64 (64-bit) وليس armv7l (32-bit)
```

* * *

## إعداد النموذج الموصى به

بما أن الـ Pi هو مجرد البوابة (النماذج تعمل في السحابة)، استخدم نماذج قائمة على API:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**لا تحاول تشغيل نماذج LLM محلية على Pi** — حتى النماذج الصغيرة بطيئة جدًا. دع Claude/GPT يقوم بالعمل الشاق.

* * *

## التشغيل التلقائي عند الإقلاع

يقوم معالج الإعداد الأولي بإعداد هذا، ولكن للتحقق:

```bash
# تحقق من تفعيل الخدمة
sudo systemctl is-enabled openclaw

# فعّل إذا لم تكن مفعلة
sudo systemctl enable openclaw

# ابدأ عند الإقلاع
sudo systemctl start openclaw
```

* * *

## استكشاف الأخطاء وإصلاحها

### نفاد الذاكرة (OOM)

```bash
# تحقق من الذاكرة
free -h

# أضف مساحة مبادلة أكثر (انظر الخطوة 5)
# أو قلل الخدمات قيد التشغيل على الـ Pi
```

### أداء بطيء

-   استخدم SSD عبر USB بدلاً من بطاقة SD
-   عطّل الخدمات غير المستخدمة: `sudo systemctl disable cups bluetooth avahi-daemon`
-   تحقق من خنق المعالج: `vcgencmd get_throttled` (يجب أن يعيد `0x0`)

### الخدمة لا تبدأ

```bash
# تحقق من السجلات
journalctl -u openclaw --no-pager -n 100

# الإصلاح الشائع: إعادة البناء
cd ~/openclaw  # إذا كنت تستخدم التثبيت القابل للتعديل
npm run build
sudo systemctl restart openclaw
```

### مشاكل البرامج الثنائية لـ ARM

إذا فشلت مهارة مع خطأ "exec format error":

1.  تحقق مما إذا كان البرنامج الثنائي يحتوي على إصدار ARM64
2.  حاول البناء من المصدر
3.  أو استخدم حاوية Docker مع دعم ARM

### انقطاع WiFi

لأجهزة Pi بدون واجهة على WiFi:

```bash
# تعطيل إدارة طاقة WiFi
sudo iwconfig wlan0 power off

# جعله دائمًا
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

* * *

## مقارنة التكلفة

| الإعداد | التكلفة لمرة واحدة | التكلفة الشهرية | ملاحظات |
| --- | --- | --- | --- |
| **Pi 4 (2GB)** | ~$45 | $0 | \+ الطاقة (~$5/سنة) |
| **Pi 4 (4GB)** | ~$55 | $0 | موصى به |
| **Pi 5 (4GB)** | ~$60 | $0 | أفضل أداء |
| **Pi 5 (8GB)** | ~$80 | $0 | مبالغ فيه ولكنه مستقبلي |
| DigitalOcean | $0 | $6/شهر | $72/سنة |
| Hetzner | $0 | €3.79/شهر | ~$50/سنة |

**نقطة التعادل:** الـ Pi يغطي تكلفته خلال ~6-12 شهرًا مقارنة بـ VPS السحابي.

* * *

## انظر أيضًا

-   [دليل Linux](./linux.md) — إعداد Linux العام
-   [دليل DigitalOcean](./digitalocean.md) — بديل سحابي
-   [دليل Hetzner](../install/hetzner.md) — إعداد Docker
-   [Tailscale](../gateway/tailscale.md) — الوصول عن بُعد
-   [العُقد](../nodes.md) — زوّد حاسوبك المحمول/هاتفك ببوابة الـ Pi

[Oracle Cloud](./oracle.md)[إعداد التطوير على macOS](./mac/dev-setup.md)