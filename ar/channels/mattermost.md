title: "دليل إعداد وتكوين تكامل OpenClaw مع Mattermost"
description: "تعلم كيفية تثبيت وتكوين إضافة Mattermost لـ OpenClaw. قم بإعداد رموز البوت، وأوامر الشرطة المائلة، ووضعيات الدردشة، والتحكم في الوصول، والميزات التفاعلية."
keywords: ["mاترموست", "أوبنكلو", "تكامل روبوت الدردشة", "إعداد الإضافة", "أوامر الشرطة المائلة", "مراسلة الفريق", "تكوين البوت", "دردشة ذاتية الاستضافة"]
---

  منصات المراسلة

  
# Mattermost

الحالة: مدعوم عبر إضافة (رمز بوت + أحداث WebSocket). القنوات والمجموعات والرسائل المباشرة مدعومة. Mattermost هي منصة مراسلة جماعية قابلة للاستضافة الذاتية؛ راجع الموقع الرسمي على [mattermost.com](https://mattermost.com) للحصول على تفاصيل المنتج والتنزيلات.

## الإضافة مطلوبة

يتم توزيع Mattermost كإضافة وليست مضمنة في التثبيت الأساسي. قم بالتثبيت عبر سطر الأوامر (سجل npm):

```bash
openclaw plugins install @openclaw/mattermost
```

التثبيت المحلي (عند التشغيل من مستودع git):

```bash
openclaw plugins install ./extensions/mattermost
```

إذا اخترت Mattermost أثناء التكوين/البدء وتم اكتشاف نسخة git، فسيعرض OpenClaw مسار التثبيت المحلي تلقائيًا. التفاصيل: [الإضافات](../tools/plugin.md)

## الإعداد السريع

1.  قم بتثبيت إضافة Mattermost.
2.  أنشئ حساب بوت Mattermost وانسخ **رمز البوت**.
3.  انسخ **عنوان URL الأساسي** لـ Mattermost (مثال: `https://chat.example.com`).
4.  قم بتكوين OpenClaw وابدأ البوابة.

التكوين الأدنى:

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## أوامر الشرطة المائلة الأصلية

أوامر الشرطة المائلة الأصلية اختيارية. عند تمكينها، يقوم OpenClaw بتسجيل أوامر `oc_*` عبر واجهة برمجة تطبيقات Mattermost ويتلقى استدعاءات POST على خادم HTTP الخاص بالبوابة.

```json
{
  channels: {
    mattermost: {
      commands: {
        native: true,
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // استخدم عندما لا تتمكن Mattermost من الوصول إلى البوابة مباشرة (خادم وكيل عكسي/عنوان URL عام).
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
    },
  },
}
```

ملاحظات:

-   `native: "auto"` تكون معطلة افتراضيًا لـ Mattermost. اضبط `native: true` للتمكين.
-   إذا تم حذف `callbackUrl`، فإن OpenClaw يستمد واحدًا من مضيف/منفذ البوابة + `callbackPath`.
-   لإعدادات الحسابات المتعددة، يمكن تعيين `commands` على المستوى الأعلى أو تحت `channels.mattermost.accounts..commands` (قيم الحساب تتجاوز الحقول على المستوى الأعلى).
-   يتم التحقق من صحة استدعاءات الأوامر باستخدام رموز لكل أمر وتفشل عند فشل فحوصات الرمز.
-   متطلبات إمكانية الوصول: يجب أن يكون نقطة نهاية الاستدعاء قابلة للوصول من خادم Mattermost.
    -   لا تقم بتعيين `callbackUrl` إلى `localhost` إلا إذا كان Mattermost يعمل على نفس المضيف/مساحة الشبكة مثل OpenClaw.
    -   لا تقم بتعيين `callbackUrl` إلى عنوان URL الأساسي لـ Mattermost الخاص بك إلا إذا كان هذا العنوان URL يقوم بالوكالة العكسية لـ `/api/channels/mattermost/command` إلى OpenClaw.
    -   فحص سريع هو `curl https://<gateway-host>/api/channels/mattermost/command`؛ يجب أن يعيد GET `405 Method Not Allowed` من OpenClaw، وليس `404`.
-   متطلبات قائمة السماح للخروج من Mattermost:
    -   إذا كان هدف استدعائك يستهدف عناوين خاصة/شبكة ذيل/داخلية، فقم بتعيين `ServiceSettings.AllowedUntrustedInternalConnections` في Mattermost لتشمل مضيف/نطاق الاستدعاء.
    -   استخدم إدخالات المضيف/النطاق، وليس عناوين URL كاملة.
        -   جيد: `gateway.tailnet-name.ts.net`
        -   سيء: `https://gateway.tailnet-name.ts.net`

## متغيرات البيئة (الحساب الافتراضي)

قم بتعيين هذه على مضيف البوابة إذا كنت تفضل متغيرات البيئة:

-   `MATTERMOST_BOT_TOKEN=...`
-   `MATTERMOST_URL=https://chat.example.com`

تنطبق متغيرات البيئة فقط على الحساب **الافتراضي** (`default`). يجب على الحسابات الأخرى استخدام قيم التكوين.

## وضعيات الدردشة

يستجيب Mattermost للرسائل المباشرة تلقائيًا. يتم التحكم في سلوك القناة بواسطة `chatmode`:

-   `oncall` (الافتراضي): الرد فقط عند ذكر @ في القنوات.
-   `onmessage`: الرد على كل رسالة في القناة.
-   `onchar`: الرد عندما تبدأ الرسالة ببادئة مشغلة.

مثال التكوين:

```json
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

ملاحظات:

-   `onchar` لا يزال يرد على ذكر @ الصريح.
-   يتم احترام `channels.mattermost.requireMention` للتكوينات القديمة ولكن `chatmode` هو المفضل.

## التحكم في الوصول (الرسائل المباشرة)

-   الافتراضي: `channels.mattermost.dmPolicy = "pairing"` (المرسلون غير المعروفين يحصلون على رمز إقران).
-   الموافقة عبر:
    -   `openclaw pairing list mattermost`
    -   `openclaw pairing approve mattermost `
-   الرسائل المباشرة العامة: `channels.mattermost.dmPolicy="open"` بالإضافة إلى `channels.mattermost.allowFrom=["*"]`.

## القنوات (المجموعات)

-   الافتراضي: `channels.mattermost.groupPolicy = "allowlist"` (مقيد بالذكر).
-   قائمة السماح للمرسلين مع `channels.mattermost.groupAllowFrom` (يوصى بمعرفات المستخدم).
-   مطابقة `@اسم المستخدم` قابلة للتغيير وتمكينها فقط عندما يكون `channels.mattermost.dangerouslyAllowNameMatching: true`.
-   القنوات المفتوحة: `channels.mattermost.groupPolicy="open"` (مقيد بالذكر).
-   ملاحظة وقت التشغيل: إذا كان `channels.mattermost` مفقودًا تمامًا، فإن وقت التشغيل يعود إلى `groupPolicy="allowlist"` لفحوصات المجموعة (حتى إذا تم تعيين `channels.defaults.groupPolicy`).

## الأهداف لتسليم الصادر

استخدم تنسيقات الهدف هذه مع `openclaw message send` أو cron/webhooks:

-   `channel:` للقناة
-   `user:` للرسالة المباشرة
-   `@username` للرسالة المباشرة (يتم حلها عبر واجهة برمجة تطبيقات Mattermost)

يتم التعامل مع المعرفات المجردة كقنوات.

## التفاعلات (أداة الرسالة)

-   استخدم `message action=react` مع `channel=mattermost`.
-   `messageId` هو معرف منشور Mattermost.
-   `emoji` يقبل أسماء مثل `thumbsup` أو `:+1:` (النقطتان اختياريتان).
-   اضبط `remove=true` (منطقية) لإزالة التفاعل.
-   يتم إعادة توجيه أحداث إضافة/إزالة التفاعل كأحداث نظام إلى جلسة الوكيل الموجهة.

أمثلة:

```bash
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup remove=true
```

التكوين:

-   `channels.mattermost.actions.reactions`: تمكين/تعطيل إجراءات التفاعل (صحيح افتراضيًا).
-   تجاوز لكل حساب: `channels.mattermost.accounts..actions.reactions`.

## الأزرار التفاعلية (أداة الرسالة)

أرسل رسائل بأزرار قابلة للنقر. عندما ينقر المستخدم على زر، يتلقى الوكيل الاختيار ويمكنه الرد. قم بتمكين الأزرار عن طريق إضافة `inlineButtons` إلى إمكانيات القناة:

```json
{
  channels: {
    mattermost: {
      capabilities: ["inlineButtons"],
    },
  },
}
```

استخدم `message action=send` مع معامل `buttons`. الأزرار هي مصفوفة ثنائية الأبعاد (صفوف من الأزرار):

```bash
message action=send channel=mattermost target=channel:<channelId> buttons=[[{"text":"نعم","callback_data":"نعم"},{"text":"لا","callback_data":"لا"}]]
```

حقول الزر:

-   `text` (مطلوب): تسمية العرض.
-   `callback_data` (مطلوب): القيمة المرسلة مرة أخرى عند النقر (تستخدم كمعرف الإجراء).
-   `style` (اختياري): `"default"`، أو `"primary"`، أو `"danger"`.

عندما ينقر المستخدم على زر:

1.  يتم استبدال جميع الأزرار بخط تأكيد (مثال: "✓ **نعم** تم اختياره بواسطة @user").
2.  يتلقى الوكيل الاختيار كرسالة واردة ويقوم بالرد.

ملاحظات:

-   تستخدم استدعاءات الأزرار التحقق HMAC-SHA256 (تلقائي، لا حاجة لتكوين).
-   تقوم Mattermost بإزالة بيانات الاستدعاء من استجابات واجهة برمجة التطبيقات الخاصة بها (ميزة أمان)، لذلك تتم إزالة جميع الأزرار عند النقر — الإزالة الجزئية غير ممكنة.
-   يتم تنظيف معرفات الإجراءات التي تحتوي على شرطات سفلية أو شرطات تلقائيًا (قيد توجيه Mattermost).

التكوين:

-   `channels.mattermost.capabilities`: مصفوفة من سلاسل الإمكانيات. أضف `"inlineButtons"` لتمكين وصف أداة الأزرار في مطالبة نظام الوكيل.
-   `channels.mattermost.interactions.callbackBaseUrl`: عنوان URL أساسي خارجي اختياري لاستدعاءات الأزرار (على سبيل المثال `https://gateway.example.com`). استخدم هذا عندما لا تتمكن Mattermost من الوصول إلى البوابة على مضيف الربط الخاص بها مباشرة.
-   في إعدادات الحسابات المتعددة، يمكنك أيضًا تعيين نفس الحقل تحت `channels.mattermost.accounts..interactions.callbackBaseUrl`.
-   إذا تم حذف `interactions.callbackBaseUrl`، فإن OpenClaw يستمد عنوان URL للاستدعاء من `gateway.customBindHost` + `gateway.port`، ثم يعود إلى `http://localhost:`.
-   قاعدة إمكانية الوصول: يجب أن يكون عنوان URL لاستدعاء الزر قابلاً للوصول من خادم Mattermost. يعمل `localhost` فقط عندما يعمل Mattermost و OpenClaw على نفس المضيف/مساحة الشبكة.
-   إذا كان هدف الاستدعاء الخاص بك خاص/شبكة ذيل/داخلي، فأضف مضيفه/نطاقه إلى `ServiceSettings.AllowedUntrustedInternalConnections` في Mattermost.

### التكامل المباشر مع واجهة برمجة التطبيقات (نصوص برمجية خارجية)

يمكن للنصوص البرمجية الخارجية و webhooks نشر أزرار مباشرة عبر واجهة برمجة تطبيقات REST الخاصة بـ Mattermost بدلاً من المرور عبر أداة `message` الخاصة بالوكيل. استخدم `buildButtonAttachments()` من الامتداد عندما يكون ذلك ممكنًا؛ إذا كنت تنشر JSON خام، فاتبع هذه القواعد: **هيكل الحمولة:**

```json
{
  channel_id: "<channelId>",
  message: "اختر خيارًا:",
  props: {
    attachments: [
      {
        actions: [
          {
            id: "mybutton01", // أبجدي رقمي فقط — انظر أدناه
            type: "button", // مطلوب، أو يتم تجاهل النقرات بصمت
            name: "موافقة", // تسمية العرض
            style: "primary", // اختياري: "default", "primary", "danger"
            integration: {
              url: "https://gateway.example.com/mattermost/interactions/default",
              context: {
                action_id: "mybutton01", // يجب أن يطابق معرف الزر (للبحث عن الاسم)
                action: "approve",
                // ... أي حقول مخصصة ...
                _token: "<hmac>", // انظر قسم HMAC أدناه
              },
            },
          },
        ],
      },
    ],
  },
}
```

**قواعد حرجة:**

1.  تذهب المرفقات في `props.attachments`، وليس `attachments` على المستوى الأعلى (يتم تجاهلها بصمت).
2.  كل إجراء يحتاج إلى `type: "button"` — بدونه، يتم ابتلاع النقرات بصمت.
3.  كل إجراء يحتاج إلى حقل `id` — يتجاهل Mattermost الإجراءات بدون معرفات.
4.  `id` للإجراء يجب أن يكون **أبجدي رقمي فقط** (`[a-zA-Z0-9]`). الشرطات والشرطات السفلية تكسر توجيه الإجراء من جانب الخادم في Mattermost (يعيد 404). قم بإزالتها قبل الاستخدام.
5.  يجب أن يطابق `context.action_id` `id` الخاص بالزر حتى تظهر رسالة التأكيد اسم الزر (مثال: "موافقة") بدلاً من معرف خام.
6.  `context.action_id` مطلوب — يعيد معالج التفاعل 400 بدونه.

**إنشاء رمز HMAC:** تتحقق البوابة من نقرات الأزرار باستخدام HMAC-SHA256. يجب على النصوص البرمجية الخارجية إنشاء رموز تطابق منطق التحقق الخاص بالبوابة:

1.  استخلص السر من رمز البوت: `HMAC-SHA256(key="openclaw-mattermost-interactions", data=botToken)`
2.  أنشئ كائن السياق بجميع الحقول **ما عدا** `_token`.
3.  قم بالتسلسل باستخدام **مفاتيح مرتبة** و **بدون مسافات** (تستخدم البوابة `JSON.stringify` مع مفاتيح مرتبة، مما ينتج عنه إخراج مضغوط).
4.  التوقيع: `HMAC-SHA256(key=secret, data=serializedContext)`
5.  أضف الناتج السداسي العشري كـ `_token` في السياق.

مثال بايثون:

```typescript
import hmac, hashlib, json

secret = hmac.new(
    b"openclaw-mattermost-interactions",
    bot_token.encode(), hashlib.sha256
).hexdigest()

ctx = {"action_id": "mybutton01", "action": "approve"}
payload = json.dumps(ctx, sort_keys=True, separators=(",", ":"))
token = hmac.new(secret.encode(), payload.encode(), hashlib.sha256).hexdigest()

context = {**ctx, "_token": token}
```

مزالق HMAC الشائعة:

-   `json.dumps` في بايثون يضيف مسافات افتراضيًا (`{"key": "val"}`). استخدم `separators=(",", ":")` لمطابقة الإخراج المضغوط لجافا سكريبت (`{"key":"val"}`).
-   وقّع دائمًا **جميع** حقول السياق (ناقص `_token`). تقوم البوابة بإزالة `_token` ثم توقيع كل ما تبقى. توقيع مجموعة فرعية يسبب فشل تحقق صامت.
-   استخدم `sort_keys=True` — تقوم البوابة بترتيب المفاتيح قبل التوقيع، وقد تعيد Mattermost ترتيب حقول السياق عند تخزين الحمولة.
-   استخلص السر من رمز البوت (حتمي)، وليس بايتات عشوائية. يجب أن يكون السر هو نفسه عبر العملية التي تنشئ الأزرار والبوابة التي تتحقق.

## محول الدليل

تتضمن إضافة Mattermost محول دليل يحل أسماء القنوات والمستخدمين عبر واجهة برمجة تطبيقات Mattermost. هذا يمكّن أهداف `#اسم-القناة` و `@اسم المستخدم` في `openclaw message send` وتسليمات cron/webhook. لا حاجة لتكوين — يستخدم المحول رمز البوت من تكوين الحساب.

## حسابات متعددة

يدعم Mattermost حسابات متعددة تحت `channels.mattermost.accounts`:

```json
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "أساسي", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "تنبيهات", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## استكشاف الأخطاء وإصلاحها

-   لا توجد ردود في القنوات: تأكد من أن البوت في القناة وقم بذكره (oncall)، أو استخدم بادئة مشغلة (onchar)، أو اضبط `chatmode: "onmessage"`.
-   أخطاء المصادقة: تحقق من رمز البوت، وعنوان URL الأساسي، وما إذا كان الحساب مفعلًا.
-   مشكلات الحسابات المتعددة: تنطبق متغيرات البيئة فقط على الحساب `default`.
-   تظهر الأزرار كصناديق بيضاء: قد يكون الوكيل يرسل بيانات زر غير صحيحة. تحقق من أن كل زر يحتوي على حقلي `text` و `callback_data`.
-   يتم عرض الأزرار ولكن النقرات لا تفعل شيئًا: تحقق من أن `AllowedUntrustedInternalConnections` في تكوين خادم Mattermost يتضمن `127.0.0.1 localhost`، وأن `EnablePostActionIntegration` هو `true` في ServiceSettings.
-   تعيد الأزرار 404 عند النقر: من المحتمل أن `id` الخاص بالزر يحتوي على شرطات أو شرطات سفلية. يتعطل جهاز توجيه الإجراء في Mattermost على المعرفات غير الأبجدية الرقمية. استخدم `[a-zA-Z0-9]` فقط.
-   سجلات البوابة `invalid _token`: عدم تطابق HMAC. تحقق من أنك وقعت على جميع حقول السياق (وليس مجموعة فرعية)، واستخدم مفاتيح مرتبة، واستخدم JSON مضغوط (بدون مسافات). راجع قسم HMAC أعلاه.
-   سجلات البوابة `missing _token in context`: حقل `_token` غير موجود في سياق الزر. تأكد من تضمينه عند بناء حمولة التكامل.
-   يظهر التأكيد معرفًا خامًا بدلاً من اسم الزر: `context.action_id` لا يطابق `id` الخاص بالزر. اضبط كلاهما على نفس القيمة المنظفة.
-   الوكيل لا يعرف عن الأزرار: أضف `capabilities: ["inlineButtons"]` إلى تكوين قناة Mattermost.

[Matrix](./matrix.md)[Microsoft Teams](./msteams.md)