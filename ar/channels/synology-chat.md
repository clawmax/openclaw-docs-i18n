

  منصات المراسلة

  
# Synology Chat

الحالة: مدعوم عبر إضافة كقناة رسائل مباشرة باستخدام ويب هوك Synology Chat. تقبل الإضافة الرسائل الواردة من ويب هوك الصادر لـ Synology Chat وترسل الردود عبر ويب هوك وارد لـ Synology Chat.

## الإضافة مطلوبة

يعتمد Synology Chat على الإضافات وليس جزءًا من تثبيت القناة الأساسي الافتراضي. قم بالتثبيت من نسخة محلية:

```bash
openclaw plugins install ./extensions/synology-chat
```

التفاصيل: [الإضافات](../tools/plugin.md)

## الإعداد السريع

1.  قم بتثبيت وتمكين إضافة Synology Chat.
2.  في تكاملات Synology Chat:
    -   أنشئ ويب هوك واردًا وانسخ عنوان URL الخاص به.
    -   أنشئ ويب هوك صادرًا باستخدام رمزك السري (token).
3.  وجه عنوان URL للويب هوك الصادر إلى بوابة OpenClaw الخاصة بك:
    -   `https://gateway-host/webhook/synology` بشكل افتراضي.
    -   أو المسار المخصص الخاص بك `channels.synology-chat.webhookPath`.
4.  قم بتكوين `channels.synology-chat` في OpenClaw.
5.  أعد تشغيل البوابة وأرسل رسالة مباشرة إلى بوت Synology Chat.

التهيئة الدنيا:

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      token: "synology-outgoing-token",
      incomingUrl: "https://nas.example.com/webapi/entry.cgi?api=SYNO.Chat.External&method=incoming&version=2&token=...",
      webhookPath: "/webhook/synology",
      dmPolicy: "allowlist",
      allowedUserIds: ["123456"],
      rateLimitPerMinute: 30,
      allowInsecureSsl: false,
    },
  },
}
```

## متغيرات البيئة

للحساب الافتراضي، يمكنك استخدام متغيرات البيئة:

-   `SYNOLOGY_CHAT_TOKEN`
-   `SYNOLOGY_CHAT_INCOMING_URL`
-   `SYNOLOGY_NAS_HOST`
-   `SYNOLOGY_ALLOWED_USER_IDS` (مفصولة بفواصل)
-   `SYNOLOGY_RATE_LIMIT`
-   `OPENCLAW_BOT_NAME`

قيم التكوين تتجاوز متغيرات البيئة.

## سياسة الرسائل المباشرة والتحكم في الوصول

-   `dmPolicy: "allowlist"` هو الافتراضي الموصى به.
-   `allowedUserIds` يقبل قائمة (أو سلسلة مفصولة بفواصل) من معرفات مستخدمي Synology.
-   في وضع `allowlist`، يتم التعامل مع قائمة `allowedUserIds` الفارغة على أنها خطأ في التكوين ولن يبدأ مسار الويب هوك (استخدم `dmPolicy: "open"` للسماح للجميع).
-   `dmPolicy: "open"` يسمح لأي مرسل.
-   `dmPolicy: "disabled"` يمنع الرسائل المباشرة.
-   تعمل موافقات الاقتران مع:
    -   `openclaw pairing list synology-chat`
    -   `openclaw pairing approve synology-chat `

## إرسال الرسائل الصادرة

استخدم معرفات مستخدمي Synology Chat الرقمية كأهداف. أمثلة:

```bash
openclaw message send --channel synology-chat --target 123456 --text "Hello from OpenClaw"
openclaw message send --channel synology-chat --target synology-chat:123456 --text "Hello again"
```

إرسال الوسائط مدعوم عبر تسليم الملفات المعتمد على URL.

## حسابات متعددة

يتم دعم حسابات Synology Chat متعددة تحت `channels.synology-chat.accounts`. يمكن لكل حساب تجاوز الرمز السري (token)، وعنوان URL الوارد، ومسار الويب هوك، وسياسة الرسائل المباشرة، والحدود.

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      accounts: {
        default: {
          token: "token-a",
          incomingUrl: "https://nas-a.example.com/...token=...",
        },
        alerts: {
          token: "token-b",
          incomingUrl: "https://nas-b.example.com/...token=...",
          webhookPath: "/webhook/synology-alerts",
          dmPolicy: "allowlist",
          allowedUserIds: ["987654"],
        },
      },
    },
  },
}
```

## ملاحظات أمنية

-   احتفظ بـ `token` سريًا وقم بتغييره إذا تم تسريبه.
-   احتفظ بـ `allowInsecureSsl: false` ما لم تثق صراحة بشهادة NAS محلية موقعة ذاتيًا.
-   طلبات الويب هوك الواردة يتم التحقق من صحتها بواسطة الرمز السري ويتم تحديد معدل الطلبات لكل مرسل.
-   يُفضل استخدام `dmPolicy: "allowlist"` للبيئة الإنتاجية.

[Signal](./signal.md)[Slack](./slack.md)

---