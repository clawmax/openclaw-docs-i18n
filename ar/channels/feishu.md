

  منصات المراسلة

  
# Feishu

Feishu (Lark) هي منصة دردشة جماعية تستخدمها الشركات للمراسلة والتعاون. تربط هذه الإضافة OpenClaw ببوت Feishu/Lark باستخدام اشتراك أحداث WebSocket الخاص بالمنصة حتى يمكن استقبال الرسائل دون الكشف عن عنوان ويب هوك عام.

* * *

## الإضافة المطلوبة

قم بتثبيت إضافة Feishu:

```bash
openclaw plugins install @openclaw/feishu
```

التثبيت المحلي (عند التشغيل من مستودع git):

```bash
openclaw plugins install ./extensions/feishu
```

* * *

## البدء السريع

هناك طريقتان لإضافة قناة Feishu:

### الطريقة 1: معالج الإعداد (موصى به)

إذا قمت للتو بتثبيت OpenClaw، قم بتشغيل المعالج:

```bash
openclaw onboard
```

سيرشدك المعالج خلال:

1.  إنشاء تطبيق Feishu وجمع بيانات الاعتماد
2.  تكوين بيانات اعتماد التطبيق في OpenClaw
3.  بدء تشغيل البوابة

✅ **بعد التكوين**، تحقق من حالة البوابة:

-   `openclaw gateway status`
-   `openclaw logs --follow`

### الطريقة 2: الإعداد عبر سطر الأوامر

إذا كنت قد أكملت التثبيت الأولي بالفعل، أضف القناة عبر سطر الأوامر:

```bash
openclaw channels add
```

اختر **Feishu**، ثم أدخل معرف التطبيق (App ID) والرمز السري للتطبيق (App Secret). ✅ **بعد التكوين**، قم بإدارة البوابة:

-   `openclaw gateway status`
-   `openclaw gateway restart`
-   `openclaw logs --follow`

* * *

## الخطوة 1: إنشاء تطبيق Feishu

### 1\. افتح منصة Feishu المفتوحة

قم بزيارة [منصة Feishu المفتوحة](https://open.feishu.cn/app) وقم بتسجيل الدخول. يجب على مستأجري Lark (العالمي) استخدام [https://open.larksuite.com/app](https://open.larksuite.com/app) وتعيين `domain: "lark"` في تكوين Feishu.

### 2\. أنشئ تطبيقًا

1.  انقر على **إنشاء تطبيق مؤسسة**
2.  املأ اسم التطبيق + الوصف
3.  اختر أيقونة للتطبيق

![إنشاء تطبيق مؤسسة](../images/channels-feishu-step2-create-app.png.md)

### 3\. انسخ بيانات الاعتماد

من **البيانات الأساسية والمعلومات**، انسخ:

-   **معرف التطبيق (App ID)** (التنسيق: `cli_xxx`)
-   **الرمز السري للتطبيق (App Secret)**

❗ **هام:** احتفظ بالرمز السري للتطبيق (App Secret) خاصًا. ![الحصول على بيانات الاعتماد](../images/channels-feishu-step3-credentials.png.md)

### 4\. قم بتكوين الأذونات

في **الأذونات**، انقر على **استيراد دفعة** والصق:

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:read",
      "cardkit:card:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

![تكوين الأذونات](../images/channels-feishu-step4-permissions.png.md)

### 5\. تمكين قدرة البوت

في **قدرات التطبيق** > **البوت**:

1.  قم بتمكين قدرة البوت
2.  عيّن اسم البوت

![تمكين قدرة البوت](../images/channels-feishu-step5-bot-capability.png.md)

### 6\. تكوين اشتراك الأحداث

⚠️ **هام:** قبل تعيين اشتراك الأحداث، تأكد من:

1.  أنك قمت بالفعل بتشغيل `openclaw channels add` لـ Feishu
2.  أن البوابة قيد التشغيل (`openclaw gateway status`)

في **اشتراك الأحداث**:

1.  اختر **استخدام اتصال طويل لاستقبال الأحداث** (WebSocket)
2.  أضف الحدث: `im.message.receive_v1`

⚠️ إذا لم تكن البوابة قيد التشغيل، قد يفشل حفظ إعداد الاتصال الطويل. ![تكوين اشتراك الأحداث](../images/channels-feishu-step6-event-subscription.png.md)

### 7\. نشر التطبيق

1.  أنشئ إصدارًا في **إدارة الإصدارات والنشر**
2.  قدّم للمراجعة وانشر
3.  انتظر موافقة المسؤول (عادةً ما تتم الموافقة تلقائيًا على تطبيقات المؤسسة)

* * *

## الخطوة 2: تكوين OpenClaw

### التكوين باستخدام المعالج (موصى به)

```bash
openclaw channels add
```

اختر **Feishu** والصق معرف التطبيق والرمز السري للتطبيق الخاصين بك.

### التكوين عبر ملف الإعدادات

قم بتحرير `~/.openclaw/openclaw.json`:

```json
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "مساعد الذكاء الاصطناعي الخاص بي",
        },
      },
    },
  },
}
```

إذا كنت تستخدم `connectionMode: "webhook"`، عيّن `verificationToken`. خادم ويب هوك Feishu يرتبط بـ `127.0.0.1` افتراضيًا؛ عيّن `webhookHost` فقط إذا كنت تحتاج عمدًا إلى عنوان ربط مختلف.

#### رمز التحقق (وضع ويب هوك)

عند استخدام وضع ويب هوك، عيّن `channels.feishu.verificationToken` في إعداداتك. للحصول على القيمة:

1.  في منصة Feishu المفتوحة، افتح تطبيقك
2.  انتقل إلى **التطوير** → **الأحداث والاستدعاءات** (开发配置 → 事件与回调)
3.  افتح علامة التبويب **التشفير** (加密策略)
4.  انسخ **رمز التحقق (Verification Token)**

![موقع رمز التحقق](../images/channels-feishu-verification-token.png.md)

### التكوين عبر متغيرات البيئة

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### نطاق Lark (العالمي)

إذا كان مستأجرك على Lark (الدولي)، عيّن النطاق إلى `lark` (أو سلسلة نطاق كاملة). يمكنك تعيينه في `channels.feishu.domain` أو لكل حساب (`channels.feishu.accounts..domain`).

```json
{
  channels: {
    feishu: {
      domain: "lark",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```

### أعلام تحسين الحصة

يمكنك تقليل استخدام واجهة برمجة تطبيقات Feishu باستخدام علمين اختياريين:

-   `typingIndicator` (الافتراضي `true`): عند `false`، تخطّي استدعاءات مؤشر الكتابة.
-   `resolveSenderNames` (الافتراضي `true`): عند `false`، تخطّي استدعاءات البحث عن ملفات تعريف المرسلين.

عيّنهما على المستوى الأعلى أو لكل حساب:

```json
{
  channels: {
    feishu: {
      typingIndicator: false,
      resolveSenderNames: false,
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          typingIndicator: true,
          resolveSenderNames: false,
        },
      },
    },
  },
}
```

* * *

## الخطوة 3: البدء + الاختبار

### 1\. ابدأ تشغيل البوابة

```bash
openclaw gateway
```

### 2\. أرسل رسالة اختبار

في Feishu، ابحث عن البوت الخاص بك وأرسل رسالة.

### 3\. وافق على الاقتران

افتراضيًا، يرد البوت برمز اقتران. وافق عليه:

```bash
openclaw pairing approve feishu <CODE>
```

بعد الموافقة، يمكنك الدردشة بشكل طبيعي.

* * *

## نظرة عامة

-   **قناة بوت Feishu**: بوت Feishu تتم إدارته بواسطة البوابة
-   **التوجيه الحتمي**: الردود تعود دائمًا إلى Feishu
-   **عزل الجلسات**: تشارك الرسائل المباشرة جلسة رئيسية واحدة؛ المجموعات معزولة
-   **اتصال WebSocket**: اتصال طويل عبر SDK الخاص بـ Feishu، لا حاجة لعنوان URL عام

* * *

## التحكم في الوصول

### الرسائل المباشرة

-   **الافتراضي**: `dmPolicy: "pairing"` (يحصل المستخدمون غير المعروفين على رمز اقتران)
-   **الموافقة على الاقتران**:
    
    نسخ
    
    ```bash
    openclaw pairing list feishu
    openclaw pairing approve feishu <CODE>
    ```
    
-   **وضع القائمة المسموح بها**: عيّن `channels.feishu.allowFrom` مع معرفات Open المسموح بها

### دردشات المجموعة

**1\. سياسة المجموعة** (`channels.feishu.groupPolicy`):

-   `"open"` = السماح للجميع في المجموعات (الافتراضي)
-   `"allowlist"` = السماح فقط لـ `groupAllowFrom`
-   `"disabled"` = تعطيل رسائل المجموعة

**2\. شرط الإشارة** (`channels.feishu.groups.<chat_id>.requireMention`):

-   `true` = يتطلب الإشارة @ (الافتراضي)
-   `false` = الرد دون إشارات

* * *

## أمثلة تكوين المجموعة

### السماح لجميع المجموعات، يتطلب الإشارة @ (الافتراضي)

```json
{
  channels: {
    feishu: {
      groupPolicy: "open",
      // الافتراضي requireMention: true
    },
  },
}
```

### السماح لجميع المجموعات، لا يتطلب الإشارة @

```json
{
  channels: {
    feishu: {
      groups: {
        oc_xxx: { requireMention: false },
      },
    },
  },
}
```

### السماح لمجموعات محددة فقط

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      // معرفات مجموعة Feishu (chat_id) تبدو مثل: oc_xxx
      groupAllowFrom: ["oc_xxx", "oc_yyy"],
    },
  },
}
```

### تقييد المرسلين الذين يمكنهم إرسال رسائل في مجموعة (قائمة مسموح بها للمرسلين)

بالإضافة إلى السماح للمجموعة نفسها، **جميع الرسائل** في تلك المجموعة مقيدة بواسطة معرف open_id للمرسل: فقط المستخدمون المدرجون في `groups.<chat_id>.allowFrom` تتم معالجة رسائلهم؛ يتم تجاهل رسائل الأعضاء الآخرين (هذا هو التقييم على مستوى المرسل بالكامل، وليس فقط لأوامر التحكم مثل /reset أو /new).

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["oc_xxx"],
      groups: {
        oc_xxx: {
          // معرفات مستخدم Feishu (open_id) تبدو مثل: ou_xxx
          allowFrom: ["ou_user1", "ou_user2"],
        },
      },
    },
  },
}
```

* * *

## الحصول على معرفات المجموعة/المستخدم

### معرفات المجموعة (chat\_id)

تبدو معرفات المجموعة مثل `oc_xxx`. **الطريقة 1 (موصى بها)**

1.  ابدأ تشغيل البوابة وأشر إلى البوت في المجموعة باستخدام @
2.  قم بتشغيل `openclaw logs --follow` وابحث عن `chat_id`

**الطريقة 2** استخدم مصحح أخطاء واجهة برمجة تطبيقات Feishu لسرد دردشات المجموعة.

### معرفات المستخدم (open\_id)

تبدو معرفات المستخدم مثل `ou_xxx`. **الطريقة 1 (موصى بها)**

1.  ابدأ تشغيل البوابة وأرسل رسالة مباشرة إلى البوت
2.  قم بتشغيل `openclaw logs --follow` وابحث عن `open_id`

**الطريقة 2** تحقق من طلبات الاقتران للحصول على معرفات Open للمستخدمين:

```bash
openclaw pairing list feishu
```

* * *

## الأوامر الشائعة

| الأمر | الوصف |
| --- | --- |
| `/status` | عرض حالة البوت |
| `/reset` | إعادة تعيين الجلسة |
| `/model` | عرض/تبديل النموذج |

> ملاحظة: لا يدعم Feishu قوائم الأوامر الأصلية بعد، لذا يجب إرسال الأوامر كنص.

## أوامر إدارة البوابة

| الأمر | الوصف |
| --- | --- |
| `openclaw gateway status` | عرض حالة البوابة |
| `openclaw gateway install` | تثبيت/بدء خدمة البوابة |
| `openclaw gateway stop` | إيقاف خدمة البوابة |
| `openclaw gateway restart` | إعادة تشغيل خدمة البوابة |
| `openclaw logs --follow` | متابعة سجلات البوابة |

* * *

## استكشاف الأخطاء وإصلاحها

### البوت لا يستجيب في دردشات المجموعة

1.  تأكد من إضافة البوت إلى المجموعة
2.  تأكد من الإشارة إلى البوت باستخدام @ (السلوك الافتراضي)
3.  تحقق من أن `groupPolicy` لم يتم تعيينه على `"disabled"`
4.  تحقق من السجلات: `openclaw logs --follow`

### البوت لا يستقبل الرسائل

1.  تأكد من نشر التطبيق وموافقته
2.  تأكد من أن اشتراك الأحداث يتضمن `im.message.receive_v1`
3.  تأكد من تمكين **الاتصال الطويل**
4.  تأكد من اكتمال أذونات التطبيق
5.  تأكد من أن البوابة قيد التشغيل: `openclaw gateway status`
6.  تحقق من السجلات: `openclaw logs --follow`

### تسريب الرمز السري للتطبيق

1.  أعد تعيين الرمز السري للتطبيق في منصة Feishu المفتوحة
2.  قم بتحديث الرمز السري للتطبيق في إعداداتك
3.  أعد تشغيل البوابة

### فشل إرسال الرسائل

1.  تأكد من أن التطبيق لديه إذن `im:message:send_as_bot`
2.  تأكد من نشر التطبيق
3.  تحقق من السجلات للحصول على أخطاء مفصلة

* * *

## التكوين المتقدم

### حسابات متعددة

```json
{
  channels: {
    feishu: {
      defaultAccount: "main",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "البوت الأساسي",
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          botName: "البوت الاحتياطي",
          enabled: false,
        },
      },
    },
  },
}
```

`defaultAccount` يتحكم في حساب Feishu المستخدم عندما لا تحدد واجهات برمجة التطبيقات الصادرة `accountId` بشكل صريح.

### حدود الرسائل

-   `textChunkLimit`: حجم جزء النص الصادر (الافتراضي: 2000 حرف)
-   `mediaMaxMb`: حد تحميل/تنزيل الوسائط (الافتراضي: 30 ميجابايت)

### البث

يدعم Feishu الردود عبر البث باستخدام البطاقات التفاعلية. عند التمكين، يقوم البوت بتحديث بطاقة أثناء إنشاء النص.

```json
{
  channels: {
    feishu: {
      streaming: true, // تمكين إخراج البطاقة عبر البث (الافتراضي true)
      blockStreaming: true, // تمكين البث على مستوى الكتلة (الافتراضي true)
    },
  },
}
```

عيّن `streaming: false` للانتظار حتى اكتمال الرد قبل الإرسال.

### التوجيه متعدد الوكلاء

استخدم `bindings` لتوجيه الرسائل المباشرة أو مجموعات Feishu إلى وكلاء مختلفين.

```json
{
  agents: {
    list: [
      { id: "main" },
      {
        id: "clawd-fan",
        workspace: "/home/user/clawd-fan",
        agentDir: "/home/user/.openclaw/agents/clawd-fan/agent",
      },
      {
        id: "clawd-xi",
        workspace: "/home/user/clawd-xi",
        agentDir: "/home/user/.openclaw/agents/clawd-xi/agent",
      },
    ],
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "clawd-fan",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_yyy" },
      },
    },
    {
      agentId: "clawd-xi",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

حقول التوجيه:

-   `match.channel`: `"feishu"`
-   `match.peer.kind`: `"direct"` أو `"group"`
-   `match.peer.id`: معرف Open للمستخدم (`ou_xxx`) أو معرف المجموعة (`oc_xxx`)

راجع [الحصول على معرفات المجموعة/المستخدم](#get-groupuser-ids) للحصول على نصائح البحث.

* * *

## مرجع التكوين

التكوين الكامل: [تكوين البوابة](../gateway/configuration.md) الخيارات الرئيسية:

| الإعداد | الوصف | الافتراضي |
| --- | --- | --- |
| `channels.feishu.enabled` | تمكين/تعطيل القناة | `true` |
| `channels.feishu.domain` | نطاق واجهة برمجة التطبيقات (`feishu` أو `lark`) | `feishu` |
| `channels.feishu.connectionMode` | وضع نقل الأحداث | `websocket` |
| `channels.feishu.defaultAccount` | معرف الحساب الافتراضي للتوجيه الصادر | `default` |
| `channels.feishu.verificationToken` | مطلوب لوضع ويب هوك | \- |
| `channels.feishu.webhookPath` | مسار مسار ويب هوك | `/feishu/events` |
| `channels.feishu.webhookHost` | مضيف ربط ويب هوك | `127.0.0.1` |
| `channels.feishu.webhookPort` | منفذ ربط ويب هوك | `3000` |
| `channels.feishu.accounts..appId` | معرف التطبيق | \- |
| `channels.feishu.accounts..appSecret` | الرمز السري للتطبيق | \- |
| `channels.feishu.accounts..domain` | تجاوز نطاق واجهة برمجة التطبيقات لكل حساب | `feishu` |
| `channels.feishu.dmPolicy` | سياسة الرسائل المباشرة | `pairing` |
| `channels.feishu.allowFrom` | قائمة المسموح بها للرسائل المباشرة (قائمة open\_id) | \- |
| `channels.feishu.groupPolicy` | سياسة المجموعة | `open` |
| `channels.feishu.groupAllowFrom` | قائمة المسموح بها للمجموعة | \- |
| `channels.feishu.groups.<chat_id>.requireMention` | يتطلب الإشارة @ | `true` |
| `channels.feishu.groups.<chat_id>.enabled` | تمكين المجموعة | `true` |
| `channels.feishu.textChunkLimit` | حجم جزء الرسالة | `2000` |
| `channels.feishu.mediaMaxMb` | حد حجم الوسائط | `30` |
| `channels.feishu.streaming` | تمكين إخراج البطاقة عبر البث | `true` |
| `channels.feishu.blockStreaming` | تمكين البث على مستوى الكتلة | `true` |

* * *

## مرجع dmPolicy

| القيمة | السلوك |
| --- | --- |
| `"pairing"` | **الافتراضي.** يحصل المستخدمون غير المعروفين على رمز اقتران؛ يجب الموافقة عليه |
| `"allowlist"` | فقط المستخدمون الموجودون في `allowFrom` يمكنهم الدردشة |
| `"open"` | السماح لجميع المستخدمين (يتطلب `"*"` في allowFrom) |
| `"disabled"` | تعطيل الرسائل المباشرة |

* * *

## أنواع الرسائل المدعومة

### الاستقبال

-   ✅ نص
-   ✅ نص منسق (منشور)
-   ✅ صور
-   ✅ ملفات
-   ✅ صوت
-   ✅ فيديو
-   ✅ ملصقات

### الإرسال

-   ✅ نص
-   ✅ صور
-   ✅ ملفات
-   ✅ صوت
-   ⚠️ نص منسق (دعم جزئي)

[Discord](./discord.md)[Google Chat](./googlechat.md)