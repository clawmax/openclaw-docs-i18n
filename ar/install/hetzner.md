title: "نشر بوابة OpenClaw على خادم VPS من Hetzner باستخدام Docker"
description: "دليل خطوة بخطوة لتشغيل بوابة OpenClaw دائمة على خادم VPS من Hetzner باستخدام Docker. تعلم كيفية تكوين الحالة، ودمج الملفات الثنائية، والوصول عبر نفق SSH."
keywords: ["خادم vps من hetzner", "بوابة openclaw", "نشر docker", "حالة دائمة", "نفق ssh", "docker compose", "استمرارية الملفات الثنائية", "openclaw ذاتي الاستضافة"]
---

  الاستضافة والنشر

  
# Hetzner

## الهدف

تشغيل بوابة OpenClaw دائمة على خادم VPS من Hetzner باستخدام Docker، مع حالة متينة، وملفات ثنائية مدمجة، وسلوك إعادة تشغيل آمن. إذا كنت تريد "OpenClaw تعمل 24/7 مقابل ~5 دولارات"، فهذا هو الإعداد الموثوق الأبسط. أسعار Hetzner تتغير؛ اختر أصغر خادم VPS يعمل بنظام Debian/Ubuntu وقم بالترقية إذا واجهت نفاد ذاكرة. تذكير بنموذج الأمان:

-   الوكلاء المشتركة بين الشركة مقبولة عندما يكون الجميع ضمن نفس حدود الثقة وبيئة التشغيل مخصصة للأعمال فقط.
-   حافظ على فصل صارم: خادم VPS/بيئة تشغيل مخصصة + حسابات مخصصة؛ لا تضع ملفات تعريف شخصية لـ Apple أو Google أو المتصفح أو مدير كلمات المرور على ذلك المضيف.
-   إذا كان المستخدمون في حالة تنافس مع بعضهم البعض، قم بالفصل حسب البوابة/المضيف/مستخدم نظام التشغيل.

راجع [الأمان](../gateway/security.md) و[استضافة VPS](../vps.md).

## ماذا سنفعل (بمصطلحات بسيطة)؟

-   استئجار خادم Linux صغير (خادم VPS من Hetzner)
-   تثبيت Docker (بيئة تشغيل معزولة للتطبيقات)
-   بدء بوابة OpenClaw داخل Docker
-   الحفاظ على `~/.openclaw` + `~/.openclaw/workspace` على المضيف (تبقى بعد عمليات إعادة التشغيل/إعادة البناء)
-   الوصول إلى واجهة التحكم من حاسوبك المحمول عبر نفق SSH

يمكن الوصول إلى البوابة عبر:

-   توجيه منفذ SSH من حاسوبك المحمول
-   تعريض المنفذ مباشرة إذا كنت تدير جدار الحماية والرموز المميزة بنفسك

يفترض هذا الدليل استخدام Ubuntu أو Debian على Hetzner.  
إذا كنت تستخدم خادم VPS آخر يعمل بنظام Linux، فقم بتعيين الحزم وفقًا لذلك. لسير العمل العام باستخدام Docker، راجع [Docker](./docker.md).

* * *

## المسار السريع (للمشغلين ذوي الخبرة)

1.  توفير خادم VPS من Hetzner
2.  تثبيت Docker
3.  استنساخ مستودع OpenClaw
4.  إنشاء أدلة مضيف دائمة
5.  تكوين `.env` و `docker-compose.yml`
6.  دمج الملفات الثنائية المطلوبة في الصورة
7.  `docker compose up -d`
8.  التحقق من استمرارية البيانات والوصول إلى البوابة

* * *

## ما تحتاجه

-   خادم VPS من Hetzner مع صلاحيات الجذر
-   وصول SSH من حاسوبك المحمول
-   معرفة أساسية بـ SSH + النسخ/اللصق
-   ~20 دقيقة
-   Docker و Docker Compose
-   بيانات اعتماد مصادقة النموذج
-   بيانات اعتماد مزود اختيارية
    -   رمز QR لـ WhatsApp
    -   رمز مميز لروبوت Telegram
    -   مصادقة OAuth لـ Gmail

* * *

## 1) توفير خادم VPS

أنشئ خادم VPS يعمل بنظام Ubuntu أو Debian في Hetzner. اتصل كجذر:

```bash
ssh root@YOUR_VPS_IP
```

يفترض هذا الدليل أن خادم VPS يحتفظ بالحالة. لا تعامله كبنية تحتية قابلة للتخلص منها.

* * *

## 2) تثبيت Docker (على خادم VPS)

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

تحقق:

```bash
docker --version
docker compose version
```

* * *

## 3) استنساخ مستودع OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

يفترض هذا الدليل أنك ستبني صورة مخصصة لضمان استمرارية الملفات الثنائية.

* * *

## 4) إنشاء أدلة مضيف دائمة

حاويات Docker عابرة. يجب أن توجد جميع الحالات طويلة الأمد على المضيف.

```bash
mkdir -p /root/.openclaw/workspace

# تعيين الملكية لمستخدم الحاوية (uid 1000):
chown -R 1000:1000 /root/.openclaw
```

* * *

## 5) تكوين متغيرات البيئة

أنشئ ملف `.env` في جذر المستودع.

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

أنشئ أسرارًا قوية:

```bash
openssl rand -hex 32
```

**لا تلتزم بهذا الملف.**

* * *

## 6) تكوين Docker Compose

أنشئ أو حدّث ملف `docker-compose.yml`.

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
      # موصى به: احتفظ بالبوابة محلية على خادم VPS؛ قم بالوصول عبر نفق SSH.
      # لتعريضها للعامة، أزل البادئة `127.0.0.1:` وقم بتكوين جدار الحماية وفقًا لذلك.
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
        "--allow-unconfigured",
      ]
```

`--allow-unconfigured` هو فقط لتسهيل عملية الإقلاع، وهو ليس بديلاً عن تكوين بوابة صحيح. لا يزال يتعين عليك تعيين المصادقة (`gateway.auth.token` أو كلمة مرور) واستخدام إعدادات ربط آمنة لنشرك.

* * *

## 7) دمج الملفات الثنائية المطلوبة في الصورة (حرج)

تثبيت الملفات الثنائية داخل حاوية قيد التشغيل هو فخ. أي شيء يتم تثبيته أثناء وقت التشغيل سيفقد عند إعادة التشغيل. يجب تثبيت جميع الملفات الثنائية الخارجية التي تتطلبها المهارات أثناء بناء الصورة. الأمثلة أدناه تظهر ثلاثة ملفات ثنائية شائعة فقط:

-   `gog` للوصول إلى Gmail
-   `goplaces` لـ Google Places
-   `wacli` لـ WhatsApp

هذه أمثلة، وليست قائمة كاملة. يمكنك تثبيت العديد من الملفات الثنائية كما تحتاج باستخدام نفس النمط. إذا أضفت مهارات جديدة لاحقًا تعتمد على ملفات ثنائية إضافية، يجب عليك:

1.  تحديث Dockerfile
2.  إعادة بناء الصورة
3.  إعادة تشغيل الحاويات

**مثال Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# مثال للملف الثنائي 1: واجهة سطر أوامر Gmail
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# مثال للملف الثنائي 2: واجهة سطر أوامر Google Places
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# مثال للملف الثنائي 3: واجهة سطر أوامر WhatsApp
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

## 8) البناء والإطلاق

```bash
docker compose build
docker compose up -d openclaw-gateway
```

تحقق من الملفات الثنائية:

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

## 9) التحقق من البوابة

```bash
docker compose logs -f openclaw-gateway
```

النجاح:

```ini
[gateway] listening on ws://0.0.0.0:18789
```

من حاسوبك المحمول:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

افتح: `http://127.0.0.1:18789/` الصق رمز البوابة المميز الخاص بك.

* * *

## ما الذي يبقى وأين (مصدر الحقيقة)

يعمل OpenClaw داخل Docker، لكن Docker ليس مصدر الحقيقة. يجب أن تنجو جميع الحالات طويلة الأمد من عمليات إعادة التشغيل وإعادة البناء وإعادة تشغيل النظام.

| المكون | الموقع | آلية الاستمرارية | ملاحظات |
| --- | --- | --- | --- |
| تكوين البوابة | `/home/node/.openclaw/` | نقطة تركيب حجم المضيف | يتضمن `openclaw.json`، الرموز المميزة |
| ملفات تعريف مصادقة النموذج | `/home/node/.openclaw/` | نقطة تركيب حجم المضيف | رموز OAuth، مفاتيح API |
| تكوينات المهارات | `/home/node/.openclaw/skills/` | نقطة تركيب حجم المضيف | حالة على مستوى المهارة |
| مساحة عمل الوكيل | `/home/node/.openclaw/workspace/` | نقطة تركيب حجم المضيف | الكود وقطع أثرية الوكيل |
| جلسة WhatsApp | `/home/node/.openclaw/` | نقطة تركيب حجم المضيف | تحافظ على تسجيل الدخول عبر QR |
| حلقة مفاتيح Gmail | `/home/node/.openclaw/` | حجم المضيف + كلمة مرور | يتطلب `GOG_KEYRING_PASSWORD` |
| الملفات الثنائية الخارجية | `/usr/local/bin/` | صورة Docker | يجب دمجها أثناء وقت البناء |
| وقت تشغيل Node | نظام ملفات الحاوية | صورة Docker | يعاد بناؤه في كل بناء للصورة |
| حزم نظام التشغيل | نظام ملفات الحاوية | صورة Docker | لا تقم بالتثبيت أثناء وقت التشغيل |
| حاوية Docker | عابرة | قابلة لإعادة التشغيل | آمنة للتدمير |

* * *

## البنية التحتية ككود (Terraform)

للفرق التي تفضل سير عمل البنية التحتية ككود، يوفر إعداد Terraform الذي تديره المجتمع:

-   تكوين Terraform معياري مع إدارة حالة عن بُعد
-   توفير آلي عبر cloud-init
-   نصوص النشر (الإقلاع، النشر، النسخ الاحتياطي/الاستعادة)
-   تعزيز الأمان (جدار الحماية، UFW، وصول SSH فقط)
-   تكوين نفق SSH للوصول إلى البوابة

**المستودعات:**

-   البنية التحتية: [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)
-   تكوين Docker: [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)

يكمل هذا النهج إعداد Docker أعلاه بنشرات قابلة للتكرار، وبنية تحتية خاضعة للتحكم بالإصدار، واستعادة من الكوارث الآلية.

> **ملاحظة:** تديرها المجتمع. للمشكلات أو المساهمات، راجع روابط المستودعات أعلاه.

[Fly.io](./fly.md)[GCP](./gcp.md)