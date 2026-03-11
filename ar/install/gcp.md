

  الاستضافة والنشر

  
# GCP

## الهدف

تشغيل بوابة OpenClaw دائمة على جهاز ظاهري في GCP Compute Engine باستخدام Docker، مع حالة دائمة، وملفات ثنائية مدمجة، وسلوك إعادة تشغيل آمن. إذا كنت تريد "OpenClaw يعمل 24/7 مقابل ~5-12 دولارًا شهريًا"، فهذا إعداد موثوق على Google Cloud. يختلف التسعير حسب نوع الجهاز والمنطقة؛ اختر أصغر جهاز ظاهري يناسب عبء العمل الخاص بك وقم بالترقية إذا واجهت مشاكل في نفاد الذاكرة.

## ماذا سنفعل (بمصطلحات بسيطة)؟

-   إنشاء مشروع GCP وتمكين الفوترة
-   إنشاء جهاز ظاهري على Compute Engine
-   تثبيت Docker (بيئة تشغيل معزولة للتطبيقات)
-   تشغيل بوابة OpenClaw في Docker
-   الحفاظ على `~/.openclaw` + `~/.openclaw/workspace` على المضيف (تبقى بعد عمليات إعادة التشغيل/إعادة البناء)
-   الوصول إلى واجهة التحكم من حاسوبك المحمول عبر نفق SSH

يمكن الوصول إلى البوابة عبر:

-   توجيه منفذ SSH من حاسوبك المحمول
-   تعريض المنفذ مباشرة إذا كنت تدير جدار الحماية والرموز بنفسك

يستخدم هذا الدليل Debian على GCP Compute Engine. يعمل Ubuntu أيضًا؛ قم بتعيين الحزم وفقًا لذلك. لتدفق Docker العام، راجع [Docker](./docker.md).

* * *

## المسار السريع (للمشغلين ذوي الخبرة)

1.  إنشاء مشروع GCP + تمكين Compute Engine API
2.  إنشاء جهاز ظاهري على Compute Engine (e2-small، Debian 12، 20 جيجابايت)
3.  الدخول عبر SSH إلى الجهاز الظاهري
4.  تثبيت Docker
5.  استنساخ مستودع OpenClaw
6.  إنشاء أدلة مضيف دائمة
7.  تهيئة `.env` و `docker-compose.yml`
8.  دمج الملفات الثنائية المطلوبة، والبناء، والإطلاق

* * *

## ما تحتاجه

-   حساب GCP (مؤهل للطبقة المجانية مع e2-micro)
-   تثبيت gcloud CLI (أو استخدام Cloud Console)
-   وصول SSH من حاسوبك المحمول
-   معرفة أساسية بالتعامل مع SSH + النسخ/اللصق
-   ~20-30 دقيقة
-   Docker و Docker Compose
-   بيانات اعتماد مصادقة النماذج
-   بيانات اعتماد مزود اختيارية
    -   رمز QR لـ WhatsApp
    -   رمز بوت Telegram
    -   مصادقة OAuth لـ Gmail

* * *

## 1) تثبيت gcloud CLI (أو استخدام Console)

**الخيار أ: gcloud CLI** (موصى به للأتمتة) قم بالتثبيت من [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install) التهيئة والمصادقة:

```bash
gcloud init
gcloud auth login
```

**الخيار ب: Cloud Console** يمكن تنفيذ جميع الخطوات عبر واجهة المستخدم على الويب على [https://console.cloud.google.com](https://console.cloud.google.com)

* * *

## 2) إنشاء مشروع GCP

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

قم بتمكين الفوترة على [https://console.cloud.google.com/billing](https://console.cloud.google.com/billing) (مطلوب لـ Compute Engine). قم بتمكين Compute Engine API:

```bash
gcloud services enable compute.googleapis.com
```

**Console:**

1.  انتقل إلى IAM & Admin > إنشاء مشروع
2.  قم بتسميته وإنشائه
3.  تمكين الفوترة للمشروع
4.  انتقل إلى APIs & Services > تمكين APIs > ابحث عن "Compute Engine API" > تمكين

* * *

## 3) إنشاء الجهاز الظاهري

**أنواع الأجهزة:**

| النوع | المواصفات | التكلفة | ملاحظات |
| --- | --- | --- | --- |
| e2-medium | 2 وحدة معالجة مركزية افتراضية، 4 جيجابايت ذاكرة وصول عشوائي | ~25 دولارًا/شهر | الأكثر موثوقية لعمليات بناء Docker المحلية |
| e2-small | 2 وحدة معالجة مركزية افتراضية، 2 جيجابايت ذاكرة وصول عشوائي | ~12 دولارًا/شهر | الحد الأدنى الموصى به لبناء Docker |
| e2-micro | 2 وحدة معالجة مركزية افتراضية (مشتركة)، 1 جيجابايت ذاكرة وصول عشوائي | مؤهل للطبقة المجانية | غالبًا ما يفشل مع نفاد ذاكرة بناء Docker (خروج 137) |

**CLI:**

```
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Console:**

1.  انتقل إلى Compute Engine > VM instances > إنشاء مثيل
2.  الاسم: `openclaw-gateway`
3.  المنطقة: `us-central1`، المنطقة الفرعية: `us-central1-a`
4.  نوع الجهاز: `e2-small`
5.  قرص التمهيد: Debian 12، 20 جيجابايت
6.  إنشاء

* * *

## 4) الدخول عبر SSH إلى الجهاز الظاهري

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Console:** انقر على زر "SSH" بجوار جهازك الظاهري في لوحة تحكم Compute Engine. ملاحظة: قد يستغرق نشر مفتاح SSH 1-2 دقيقة بعد إنشاء الجهاز الظاهري. إذا تم رفض الاتصال، انتظر وأعد المحاولة.

* * *

## 5) تثبيت Docker (على الجهاز الظاهري)

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

اخرج وأعد الدخول لتفعيل تغيير المجموعة:

```
exit
```

ثم أدخل عبر SSH مرة أخرى:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

تحقق:

```bash
docker --version
docker compose version
```

* * *

## 6) استنساخ مستودع OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

يفترض هذا الدليل أنك ستبني صورة مخصصة لضمان استمرارية الملفات الثنائية.

* * *

## 7) إنشاء أدلة مضيف دائمة

حاويات Docker مؤقتة. يجب أن توجد جميع الحالات طويلة الأمد على المضيف.

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

* * *

## 8) تهيئة متغيرات البيئة

قم بإنشاء `.env` في جذر المستودع.

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

قم بإنشاء أسرار قوية:

```bash
openssl rand -hex 32
```

**لا تقم بإيداع هذا الملف.**

* * *

## 9) تهيئة Docker Compose

قم بإنشاء أو تحديث `docker-compose.yml`.

```
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # موصى به: احتفظ بالبوابة محلية على الجهاز الظاهري فقط؛ قم بالوصول عبر نفق SSH.
      # لتعريضها للعامة، قم بإزالة البادئة `127.0.0.1:` وقم بتكوين جدار الحماية وفقًا لذلك.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
      ]
```

* * *

## 10) دمج الملفات الثنائية المطلوبة في الصورة (حرج)

تثبيت الملفات الثنائية داخل حاوية قيد التشغيل هو فخ. أي شيء يتم تثبيته أثناء وقت التشغيل سيفقد عند إعادة التشغيل. يجب تثبيت جميع الملفات الثنائية الخارجية المطلوبة بواسطة المهارات في وقت بناء الصورة. تظهر الأمثلة أدناه ثلاثة ملفات ثنائية شائعة فقط:

-   `gog` للوصول إلى Gmail
-   `goplaces` لـ Google Places
-   `wacli` لـ WhatsApp

هذه أمثلة، وليست قائمة كاملة. يمكنك تثبيت أكبر عدد من الملفات الثنائية كما تحتاج باستخدام نفس النمط. إذا أضفت مهارات جديدة لاحقًا تعتمد على ملفات ثنائية إضافية، يجب عليك:

1.  تحديث Dockerfile
2.  إعادة بناء الصورة
3.  إعادة تشغيل الحاويات

**مثال Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# مثال للملف الثنائي 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# مثال للملف الثنائي 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# مثال للملف الثنائي 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# أضف المزيد من الملفات الثنائية أدناه باستخدام نفس النمط

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

* * *

## 11) البناء والإطلاق

```bash
docker compose build
docker compose up -d openclaw-gateway
```

إذا فشل البناء مع `Killed` / `exit code 137` أثناء `pnpm install --frozen-lockfile`، فإن الجهاز الظاهري يعاني من نفاد الذاكرة. استخدم `e2-small` كحد أدنى، أو `e2-medium` لعمليات البناء الأولى الأكثر موثوقية. عند الربط مع الشبكة المحلية (`OPENCLAW_GATEWAY_BIND=lan`)، قم بتهيئة أصل متصفح موثوق به قبل المتابعة:

```bash
docker compose run --rm openclaw-cli config set gateway.controlUi.allowedOrigins '["http://127.0.0.1:18789"]' --strict-json
```

إذا قمت بتغيير منفذ البوابة، استبدل `18789` بالمنفذ الذي قمت بتهيئته. تحقق من الملفات الثنائية:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

الناتج المتوقع:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

* * *

## 12) التحقق من البوابة

```bash
docker compose logs -f openclaw-gateway
```

النجاح:

```ini
[gateway] listening on ws://0.0.0.0:18789
```

* * *

## 13) الوصول من حاسوبك المحمول

قم بإنشاء نفق SSH لتوجيه منفذ البوابة:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

افتح في متصفحك: `http://127.0.0.1:18789/` احصل على رابط لوحة تحكم جديد مع رمز:

```bash
docker compose run --rm openclaw-cli dashboard --no-open
```

الصق الرمز من عنوان URL ذلك. إذا أظهرت واجهة التحكم `unauthorized` أو `disconnected (1008): pairing required`، قم بالموافقة على جهاز المتصفح:

```bash
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

* * *

## ما الذي يبقى وأين (مصدر الحقيقة)

يعمل OpenClaw في Docker، لكن Docker ليس مصدر الحقيقة. يجب أن تبقى جميع الحالات طويلة الأمد بعد عمليات إعادة التشغيل وإعادة البناء وإعادة التشغيل.

| المكون | الموقع | آلية الاستمرارية | ملاحظات |
| --- | --- | --- | --- |
| تهيئة البوابة | `/home/node/.openclaw/` | نقطة تركيب حجم المضيف | يتضمن `openclaw.json`، الرموز |
| ملفات تعريف مصادقة النماذج | `/home/node/.openclaw/` | نقطة تركيب حجم المضيف | رموز OAuth، مفاتيح API |
| تهيئات المهارات | `/home/node/.openclaw/skills/` | نقطة تركيب حجم المضيف | حالة مستوى المهارة |
| مساحة عمل الوكيل | `/home/node/.openclaw/workspace/` | نقطة تركيب حجم المضيف | الكود وقطع الوكيل |
| جلسة WhatsApp | `/home/node/.openclaw/` | نقطة تركيب حجم المضيف | يحفظ تسجيل الدخول عبر QR |
| حلقة مفاتيح Gmail | `/home/node/.openclaw/` | حجم المضيف + كلمة مرور | يتطلب `GOG_KEYRING_PASSWORD` |
| الملفات الثنائية الخارجية | `/usr/local/bin/` | صورة Docker | يجب دمجها في وقت البناء |
| وقت تشغيل Node | نظام ملفات الحاوية | صورة Docker | يعاد بناؤه في كل بناء للصورة |
| حزم نظام التشغيل | نظام ملفات الحاوية | صورة Docker | لا تقم بالتثبيت أثناء وقت التشغيل |
| حاوية Docker | مؤقتة | قابلة لإعادة التشغيل | آمنة للتدمير |

* * *

## التحديثات

لتحديث OpenClaw على الجهاز الظاهري:

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

* * *

## استكشاف الأخطاء وإصلاحها

**رفض اتصال SSH** قد يستغرق نشر مفتاح SSH 1-2 دقيقة بعد إنشاء الجهاز الظاهري. انتظر وأعد المحاولة. **مشاكل OS Login** تحقق من ملف تعريف OS Login الخاص بك:

```bash
gcloud compute os-login describe-profile
```

تأكد من أن حسابك لديه أذونات IAM المطلوبة (Compute OS Login أو Compute OS Admin Login). **نفاد الذاكرة (OOM)** إذا فشل بناء Docker مع `Killed` و `exit code 137`، فقد تم إنهاء الجهاز الظاهري بسبب نفاد الذاكرة. قم بالترقية إلى e2-small (الحد الأدنى) أو e2-medium (موصى به لعمليات البناء المحلية الموثوقة):

```bash
# أوقف الجهاز الظاهري أولاً
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# تغيير نوع الجهاز
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# ابدأ الجهاز الظاهري
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

* * *

## حسابات الخدمة (أفضل ممارسة أمنية)

للاستخدام الشخصي، يعمل حساب المستخدم الافتراضي الخاص بك بشكل جيد. للأتمتة أو خطوط أنابيب CI/CD، قم بإنشاء حساب خدمة مخصص بأقل الصلاحيات:

1.  إنشاء حساب خدمة:
    
    نسخ
    
    ```
    gcloud iam service-accounts create openclaw-deploy \
      --display-name="OpenClaw Deployment"
    ```
    
2.  منح دور مسؤول مثيل Compute (أو دور مخصص أضيق):
    
    نسخ
    
    ```
    gcloud projects add-iam-policy-binding my-openclaw-project \
      --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
      --role="roles/compute.instanceAdmin.v1"
    ```
    

تجنب استخدام دور المالك للأتمتة. استخدم مبدأ أقل امتياز. راجع [https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles) للحصول على تفاصيل أدوار IAM.

* * *

## الخطوات التالية

-   إعداد قنوات المراسلة: [القنوات](../channels.md)
-   إقران الأجهزة المحلية كعقد: [العقد](../nodes.md)
-   تهيئة البوابة: [تهيئة البوابة](../gateway/configuration.md)

[Hetzner](./hetzner.md)[أجهزة ظاهرية لنظام macOS](./macos-vm.md)