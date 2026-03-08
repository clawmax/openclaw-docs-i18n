title: "نشر بوابة OpenClaw على Fly.io مع تخزين مستمر"
description: "دليل خطوة بخطوة لاستضافة ونشر بوابة OpenClaw AI على Fly.io. تعلم كيفية تكوين التخزين المستمر، تعيين الأسرار، تمكين HTTPS، والاتصال بـ Discord."
keywords: ["نشر fly.io", "بوابة openclaw", "تخزين مستمر", "استضافة بوت ديسكورد", "إعداد بوابة الذكاء الاصطناعي", "تكوين fly.toml", "نشر خاص", "ذكاء اصطناعي بدون خادم"]
---

  الاستضافة والنشر

  
# Fly.io

**الهدف:** تشغيل بوابة OpenClaw على جهاز [Fly.io](https://fly.io) مع تخزين مستمر، HTTPS تلقائي، ووصول إلى Discord/القنوات.

## ما تحتاجه

-   تثبيت [أداة سطر أوامر flyctl](https://fly.io/docs/hands-on/install-flyctl/)
-   حساب على Fly.io (النسخة المجانية تعمل)
-   مصادقة النموذج: مفتاح API لمزود النموذج الذي اخترته
-   بيانات اعتماد القناة: رمز بوت Discord، رمز Telegram، إلخ.

## المسار السريع للمبتدئين

1.  استنساخ المستودع → تخصيص `fly.toml`
2.  إنشاء التطبيق + وحدة التخزين → تعيين الأسرار
3.  النشر باستخدام `fly deploy`
4.  الدخول عبر SSH لإنشاء التكوين أو استخدام واجهة التحكم

## 1) إنشاء تطبيق Fly

```bash
# استنساخ المستودع
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# إنشاء تطبيق Fly جديد (اختر اسمك الخاص)
fly apps create my-openclaw

# إنشاء وحدة تخزين مستمرة (1 جيجابايت تكفي عادة)
fly volumes create openclaw_data --size 1 --region iad
```

**نصيحة:** اختر منطقة قريبة منك. خيارات شائعة: `lhr` (لندن)، `iad` (فرجينيا)، `sjc` (سان خوسيه).

## 2) تكوين fly.toml

قم بتحرير `fly.toml` ليتطابق مع اسم تطبيقك ومتطلباتك. **ملاحظة أمنية:** التكوين الافتراضي يعرض عنوان URL عام. لنشر محصن بدون عنوان IP عام، راجع [النشر الخاص](#private-deployment-hardened) أو استخدم `fly.private.toml`.

```
app = "my-openclaw"  # اسم تطبيقك
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**الإعدادات الرئيسية:**

| الإعداد | السبب |
| --- | --- |
| `--bind lan` | يربط بـ `0.0.0.0` حتى يتمكن وكيل Fly من الوصول إلى البوابة |
| `--allow-unconfigured` | يبدأ التشغيل بدون ملف تكوين (ستقوم بإنشاء واحد لاحقًا) |
| `internal_port = 3000` | يجب أن يتطابق مع `--port 3000` (أو `OPENCLAW_GATEWAY_PORT`) لفحوصات صحة Fly |
| `memory = "2048mb"` | 512 ميجابايت صغير جدًا؛ يوصى بـ 2 جيجابايت |
| `OPENCLAW_STATE_DIR = "/data"` | يحفظ الحالة على وحدة التخزين |

## 3) تعيين الأسرار

```bash
# مطلوب: رمز البوابة (للالتحام غير المحلي)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# مفاتيح API لمزودي النماذج
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# اختياري: مزودون آخرون
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# رموز القنوات
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**ملاحظات:**

-   عمليات الالتحام غير المحلية (`--bind lan`) تتطلب `OPENCLAW_GATEWAY_TOKEN` للأمان.
-   عالج هذه الرموز ككلمات مرور.
-   **فضل متغيرات البيئة على ملف التكوين** لجميع مفاتيح API والرموز. هذا يحفظ الأسرار بعيدًا عن `openclaw.json` حيث يمكن أن تتعرض أو تسجل عن طريق الخطأ.

## 4) النشر

```bash
fly deploy
```

أول عملية نشر تبني صورة Docker (~2-3 دقائق). عمليات النشر اللاحقة أسرع. بعد النشر، تحقق:

```bash
fly status
fly logs
```

يجب أن ترى:

```ini
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```

## 5) إنشاء ملف التكوين

ادخل عبر SSH إلى الجهاز لإنشاء تكوين مناسب:

```bash
fly ssh console
```

أنشئ مجلد التكوين والملف:

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**ملاحظة:** مع `OPENCLAW_STATE_DIR=/data`، مسار التكوين هو `/data/openclaw.json`. **ملاحظة:** يمكن أن يأتي رمز Discord من:

-   متغير البيئة: `DISCORD_BOT_TOKEN` (موصى به للأسرار)
-   ملف التكوين: `channels.discord.token`

إذا كنت تستخدم متغير البيئة، لا حاجة لإضافة الرمز إلى التكوين. تقرأ البوابة `DISCORD_BOT_TOKEN` تلقائيًا. أعد التشغيل للتطبيق:

```
exit
fly machine restart <machine-id>
```

## 6) الوصول إلى البوابة

### واجهة التحكم

افتح في المتصفح:

```bash
fly open
```

أو زر `https://my-openclaw.fly.dev/` الصق رمز البوابة الخاص بك (الذي من `OPENCLAW_GATEWAY_TOKEN`) للمصادقة.

### السجلات

```bash
fly logs              # سجلات مباشرة
fly logs --no-tail    # سجلات حديثة
```

### وحدة التحكم عبر SSH

```bash
fly ssh console
```

## استكشاف الأخطاء وإصلاحها

### "التطبيق لا يستمع على العنوان المتوقع"

البوابة تربط بـ `127.0.0.1` بدلاً من `0.0.0.0`. **الإصلاح:** أضف `--bind lan` إلى أمر العملية في `fly.toml`.

### فحوصات الصحة تفشل / رفض الاتصال

لا يستطيع Fly الوصول إلى البوابة على المنفذ المكون. **الإصلاح:** تأكد من أن `internal_port` يطابق منفذ البوابة (اضبط `--port 3000` أو `OPENCLAW_GATEWAY_PORT=3000`).

### نفاد الذاكرة / مشاكل الذاكرة

الحاوية تستمر في إعادة التشغيل أو يتم إيقافها. علامات: `SIGABRT`، `v8::internal::Runtime_AllocateInYoungGeneration`، أو إعادة تشغيل صامتة. **الإصلاح:** زد الذاكرة في `fly.toml`:

```
[[vm]]
  memory = "2048mb"
```

أو قم بتحديث جهاز موجود:

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**ملاحظة:** 512 ميجابايت صغير جدًا. 1 جيجابايت قد يعمل ولكن قد ينفد الذاكرة تحت الحمل أو مع تسجيل مفصل. **يوصى بـ 2 جيجابايت.**

### مشاكل قفل البوابة

البوابة ترفض البدء مع أخطاء "قيد التشغيل بالفعل". يحدث هذا عندما تعيد الحاوية التشغيل ولكن ملف قفل PID يبقى على وحدة التخزين. **الإصلاح:** احذف ملف القفل:

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

يوجد ملف القفل في `/data/gateway.*.lock` (ليس في مجلد فرعي).

### عدم قراءة التكوين

إذا كنت تستخدم `--allow-unconfigured`، تنشئ البوابة تكوينًا بسيطًا. يجب قراءة تكوينك المخصص في `/data/openclaw.json` عند إعادة التشغيل. تحقق من وجود التكوين:

```bash
fly ssh console --command "cat /data/openclaw.json"
```

### كتابة التكوين عبر SSH

أمر `fly ssh console -C` لا يدعم إعادة توجيه shell. لكتابة ملف تكوين:

```bash
# استخدم echo + tee (أنبوب من المحلي إلى البعيد)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# أو استخدم sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**ملاحظة:** قد يفشل `fly sftp` إذا كان الملف موجودًا بالفعل. احذفه أولاً:

```bash
fly ssh console --command "rm /data/openclaw.json"
```

### الحالة لا تستمر

إذا فقدت بيانات الاعتماد أو الجلسات بعد إعادة التشغيل، فإن مجلد الحالة يكتب على نظام ملفات الحاوية. **الإصلاح:** تأكد من تعيين `OPENCLAW_STATE_DIR=/data` في `fly.toml` وأعد النشر.

## التحديثات

```bash
# جلب أحدث التغييرات
git pull

# إعادة النشر
fly deploy

# التحقق من الصحة
fly status
fly logs
```

### تحديث أمر الجهاز

إذا كنت بحاجة إلى تغيير أمر بدء التشغيل بدون إعادة نشر كاملة:

```bash
# الحصول على معرف الجهاز
fly machines list

# تحديث الأمر
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# أو مع زيادة الذاكرة
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**ملاحظة:** بعد `fly deploy`، قد يعود أمر الجهاز إلى ما هو موجود في `fly.toml`. إذا أجريت تغييرات يدوية، أعد تطبيقها بعد النشر.

## النشر الخاص (محصن)

بشكل افتراضي، يخصص Fly عناوين IP عامة، مما يجعل بوابتك قابلة للوصول على `https://your-app.fly.dev`. هذا مريح ولكن يعني أن نشرك يمكن اكتشافه بواسطة ماسحات الإنترنت (Shodan، Censys، إلخ). لنشر محصن **بدون تعرض عام**، استخدم القالب الخاص.

### متى تستخدم النشر الخاص

-   تقوم فقط بإجراء مكالمات/رسائل **صادرة** (لا توجد webhooks واردة)
-   تستخدم أنفاق **ngrok أو Tailscale** لأي ردود اتصال webhook
-   تصل إلى البوابة عبر **SSH، وكيل، أو WireGuard** بدلاً من المتصفح
-   تريد إخفاء النشر **عن ماسحات الإنترنت**

### الإعداد

استخدم `fly.private.toml` بدلاً من التكوين القياسي:

```bash
# النشر بتكوين خاص
fly deploy -c fly.private.toml
```

أو قم بتحويل نشر موجود:

```bash
# عرض عناوين IP الحالية
fly ips list -a my-openclaw

# إطلاق عناوين IP العامة
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# التبديل إلى التكوين الخاص حتى لا تعيد عمليات النشر المستقبلية تخصيص عناوين IP عامة
# (احذف [http_service] أو انشر باستخدام القالب الخاص)
fly deploy -c fly.private.toml

# تخصيص IPv6 خاص فقط
fly ips allocate-v6 --private -a my-openclaw
```

بعد هذا، يجب أن يظهر `fly ips list` عنوان IP من النوع `private` فقط:

```bash
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```

### الوصول إلى نشر خاص

بما أنه لا يوجد عنوان URL عام، استخدم إحدى هذه الطرق: **الخيار 1: وكيل محلي (الأبسط)**

```bash
# توجيه المنفذ المحلي 3000 إلى التطبيق
fly proxy 3000:3000 -a my-openclaw

# ثم افتح http://localhost:3000 في المتصفح
```

**الخيار 2: شبكة VPN WireGuard**

```bash
# إنشاء تكوين WireGuard (مرة واحدة)
fly wireguard create

# استيراده إلى عميل WireGuard، ثم الوصول عبر IPv6 الداخلي
# مثال: http://[fdaa:x:x:x:x::x]:3000
```

**الخيار 3: SSH فقط**

```bash
fly ssh console -a my-openclaw
```

### Webhooks مع نشر خاص

إذا كنت بحاجة إلى ردود اتصال webhook (Twilio، Telnyx، إلخ.) بدون تعرض عام:

1.  **نفق ngrok** - شغل ngrok داخل الحاوية أو كحاوية جانبية
2.  **قناة Tailscale** - اعرض مسارات محددة عبر Tailscale
3.  **صادر فقط** - بعض المزودين (Twilio) يعملون بشكل جيد للمكالمات الصادرة بدون webhooks

مثال تكوين مكالمة صوتية مع ngrok:

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" },
          "webhookSecurity": {
            "allowedHosts": ["example.ngrok.app"]
          }
        }
      }
    }
  }
}
```

يعمل نفق ngrok داخل الحاوية ويوفر عنوان URL عام لـ webhook بدون تعريض تطبيق Fly نفسه. عيّن `webhookSecurity.allowedHosts` إلى اسم المضيف العام للنفق حتى يتم قبول رؤوس المضيف التي تم توجيهها.

### فوائد الأمان

| الجانب | عام | خاص |
| --- | --- | --- |
| ماسحات الإنترنت | قابل للاكتشاف | مخفي |
| هجمات مباشرة | ممكنة | محظورة |
| الوصول لواجهة التحكم | متصفح | وكيل/VPN |
| تسليم Webhook | مباشر | عبر نفق |

## ملاحظات

-   يستخدم Fly.io **بنية x86** (ليست ARM)
-   ملف Dockerfile متوافق مع كلا البنيتين
-   للتسجيل في WhatsApp/Telegram، استخدم `fly ssh console`
-   البيانات المستمرة تعيش على وحدة التخزين في `/data`
-   Signal يتطلب Java + signal-cli؛ استخدم صورة مخصصة واحتفظ بالذاكرة عند 2 جيجابايت+.

## التكلفة

مع التكوين الموصى به (`shared-cpu-2x`، 2 جيجابايت ذاكرة وصول عشوائي):

-   ~10-15 دولارًا شهريًا حسب الاستخدام
-   النسخة المجانية تتضمن بعض الحصة

راجع [أسعار Fly.io](https://fly.io/docs/about/pricing/) للتفاصيل.

[استضافة VPS](../vps.md)[Hetzner](./hetzner.md)