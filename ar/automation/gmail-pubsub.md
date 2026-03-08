title: "أتمتة Gmail إلى OpenClaw عبر إشعارات الدفع PubSub"
description: "تعلم كيفية إعداد أتمتة Gmail باستخدام Google PubSub. إعادة توجيه رسائل البريد الإلكتروني إلى OpenClaw عبر webhooks للمعالجة بواسطة الذكاء الاصطناعي والتسليم إلى الدردشة."
keywords: ["أتمتة gmail", "pubsub", "webhook openclaw", "مراقبة gmail", "أتمتة البريد الإلكتروني", "gogcli", "قناة tailscale", "إشعارات الدفع"]
---

  الأتمتة

  
# Gmail PubSub

الهدف: مراقبة Gmail -> دفع Pub/Sub -> `gog gmail watch serve` -> webhook OpenClaw.

## المتطلبات الأساسية

-   `gcloud` مثبت وقيد تسجيل الدخول ([دليل التثبيت](https://docs.cloud.google.com/sdk/docs/install-sdk)).
-   `gog` (gogcli) مثبت ومصرح به لحساب Gmail ([gogcli.sh](https://gogcli.sh/)).
-   تمكين خطاطيف OpenClaw (انظر [Webhooks](./webhook.md)).
-   `tailscale` قيد تسجيل الدخول ([tailscale.com](https://tailscale.com/)). يستخدم الإعداد المدعوم Tailscale Funnel لنقطة النهاية العامة HTTPS. يمكن أن تعمل خدمات النفق الأخرى، لكنها تتطلب إعدادًا يدويًا وغير مدعومة. حاليًا، Tailscale هو ما ندعمه.

مثال على تكوين الخطاف (تمكين تعيين الإعداد المسبق لـ Gmail):

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"],
  },
}
```

لتسليم ملخص Gmail إلى سطح دردشة، تجاوز الإعداد المسبق بتعيين يحدد `deliver` + اختياريًا `channel`/`to`:

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last",
        // to: "+15551234567"
      },
    ],
  },
}
```

إذا كنت تريد قناة ثابتة، عيّن `channel` + `to`. وإلا فإن `channel: "last"` تستخدم مسار التسليم الأخير (يتراجع إلى WhatsApp). لإجبار استخدام نموذج أرخص لتشغيلات Gmail، عيّن `model` في التعيين (`provider/model` أو اسم مستعار). إذا فرضت `agents.defaults.models`، فقم بتضمينه هناك. لتعيين نموذج افتراضي ومستوى تفكير مخصص لخطاطيف Gmail، أضف `hooks.gmail.model` / `hooks.gmail.thinking` في التكوين الخاص بك:

```json
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

ملاحظات:

-   `model`/`thinking` لكل خطاف في التعيين لا يزال يتجاوز هذه الإعدادات الافتراضية.
-   ترتيب التراجع: `hooks.gmail.model` → `agents.defaults.model.fallbacks` → الأساسي (المصادقة/حد المعدل/المهلات الزمنية).
-   إذا تم تعيين `agents.defaults.models`، يجب أن يكون نموذج Gmail في القائمة المسموح بها.
-   محتوى خطاف Gmail مغلف بحدود أمان المحتوى الخارجي افتراضيًا. لتعطيله (خطير)، عيّن `hooks.gmail.allowUnsafeExternalContent: true`.

لتخصيص معالجة الحمولة بشكل أكبر، أضف `hooks.mappings` أو وحدة تحويل JS/TS تحت `~/.openclaw/hooks/transforms` (انظر [Webhooks](./webhook.md)).

## المعالج (موصى به)

استخدم مساعد OpenClaw لربط كل شيء معًا (يثبت التبعيات على macOS عبر brew):

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

الإعدادات الافتراضية:

-   يستخدم Tailscale Funnel لنقطة دفع النهاية العامة.
-   يكتب تكوين `hooks.gmail` لـ `openclaw webhooks gmail run`.
-   يمكّن الإعداد المسبق لخطاف Gmail (`hooks.presets: ["gmail"]`).

ملاحظة على المسار: عند تمكين `tailscale.mode`، يضبط OpenClaw تلقائيًا `hooks.gmail.serve.path` على `/` ويحتفظ بالمسار العام عند `hooks.gmail.tailscale.path` (الافتراضي `/gmail-pubsub`) لأن Tailscale يزيل بادئة المسار المحدد قبل التوكيل. إذا كنت بحاجة إلى أن يستقبل الخلفي المسار ذو البادئة، عيّن `hooks.gmail.tailscale.target` (أو `--tailscale-target`) إلى عنوان URL كامل مثل `http://127.0.0.1:8788/gmail-pubsub` وطابق `hooks.gmail.serve.path`. تريد نقطة نهاية مخصصة؟ استخدم `--push-endpoint ` أو `--tailscale off`. ملاحظة على المنصة: على macOS يقوم المعالج بتثبيت `gcloud` و `gogcli` و `tailscale` عبر Homebrew؛ على Linux قم بتثبيتها يدويًا أولاً. بدء البوابة التلقائي (موصى به):

-   عندما يكون `hooks.enabled=true` و `hooks.gmail.account` مضبوطًا، تبدأ البوابة `gog gmail watch serve` عند التمهيد وتجديد المراقبة تلقائيًا.
-   عيّن `OPENCLAW_SKIP_GMAIL_WATCHER=1` للانسحاب (مفيد إذا كنت تشغل البرنامج الخفي يدويًا).
-   لا تشغل البرنامج الخفي اليدوي في نفس الوقت، وإلا ستواجه `listen tcp 127.0.0.1:8788: bind: address already in use`.

البرنامج الخفي اليدوي (يبدأ `gog gmail watch serve` + تجديد تلقائي):

```bash
openclaw webhooks gmail run
```

## الإعداد لمرة واحدة

1.  حدد مشروع GCP **الذي يمتلك عميل OAuth** المستخدم من قبل `gog`.

```bash
gcloud auth login
gcloud config set project <project-id>
```

ملاحظة: تتطلب مراقبة Gmail أن يكون موضوع Pub/Sub موجودًا في نفس مشروع عميل OAuth.

2.  تمكين واجهات برمجة التطبيقات:

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3.  إنشاء موضوع:

```bash
gcloud pubsub topics create gog-gmail-watch
```

4.  السماح لدفع Gmail بالنشر:

```
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

## بدء المراقبة

```
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

احفظ `history_id` من الناتج (لأغراض التصحيح).

## تشغيل معالج الدفع

مثال محلي (مصادقة الرمز المشترك):

```
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

ملاحظات:

-   `--token` يحمي نقطة نهاية الدفع (`x-gog-token` أو `?token=`).
-   `--hook-url` يشير إلى OpenClaw `/hooks/gmail` (معيّن؛ تشغيل معزول + ملخص إلى الرئيسي).
-   `--include-body` و `--max-bytes` يتحكمان في مقتطف النص المرسل إلى OpenClaw.

موصى به: `openclaw webhooks gmail run` يغلف نفس التدفق ويجدد المراقبة تلقائيًا.

## تعريض المعالج (متقدم، غير مدعوم)

إذا كنت بحاجة إلى نفق غير Tailscale، قم بتوصيله يدويًا واستخدم عنوان URL العام في اشتراك الدفع (غير مدعوم، بدون ضوابط أمان):

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

استخدم عنوان URL المُنشأ كنقطة نهاية الدفع:

```
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

للإنتاج: استخدم نقطة نهاية HTTPS مستقرة وقم بتكوين Pub/Sub OIDC JWT، ثم شغل:

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

## الاختبار

أرسل رسالة إلى صندوق الوارد المراقب:

```
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

تحقق من حالة المراقبة والسجل:

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

## استكشاف الأخطاء وإصلاحها

-   `Invalid topicName`: عدم تطابق المشروع (الموضوع ليس في مشروع عميل OAuth).
-   `User not authorized`: مفقود `roles/pubsub.publisher` على الموضوع.
-   رسائل فارغة: يوفر دفع Gmail فقط `historyId`؛ جلبها عبر `gog gmail history`.

## التنظيف

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```

[Webhooks](./webhook.md)[Polls](./poll.md)