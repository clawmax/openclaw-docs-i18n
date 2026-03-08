title: "دليل إعداد وتكوين تكامل دردشة Twitch مع OpenClaw"
description: "تعلم كيفية إعداد وتكوين OpenClaw لدعم دردشة Twitch. قم بتوصيل حساب البوت الخاص بك، وإنشاء بيانات الاعتماد، وإدارة التحكم في الوصول، واستكشاف الأخطاء الشائعة وإصلاحها."
keywords: ["بوت تويش", "openclaw تويش", "تكامل دردشة تويش", "اتصال IRC", "رمز تويش", "تكوين البوت", "التحكم في الوصول", "حسابات متعددة"]
---

  منصات المراسلة

  
# Twitch

دعم دردشة Twitch عبر اتصال IRC. يتصل OpenClaw كمستخدم Twitch (حساب بوت) لاستقبال وإرسال الرسائل في القنوات.

## المكون الإضافي مطلوب

يتم توزيع Twitch كمكون إضافي ولا يتم تضمينه مع التثبيت الأساسي. قم بالتثبيت عبر سطر الأوامر (npm registry):

```bash
openclaw plugins install @openclaw/twitch
```

نسخة محلية (عند التشغيل من مستودع git):

```bash
openclaw plugins install ./extensions/twitch
```

التفاصيل: [المكونات الإضافية](../tools/plugin.md)

## الإعداد السريع (للمبتدئين)

1.  أنشئ حساب Twitch مخصصًا للبوت (أو استخدم حسابًا موجودًا).
2.  إنشاء بيانات الاعتماد: [منشئ رمز Twitch](https://twitchtokengenerator.com/)
    -   اختر **رمز البوت (Bot Token)**
    -   تحقق من تحديد النطاقات `chat:read` و `chat:write`
    -   انسخ **معرف العميل (Client ID)** و **رمز الوصول (Access Token)**
3.  ابحث عن معرف مستخدم Twitch الخاص بك: [https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)
4.  قم بتكوين الرمز:
    -   متغير البيئة: `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (الحساب الافتراضي فقط)
    -   أو التكوين: `channels.twitch.accessToken`
    -   إذا تم تعيين كليهما، فإن التكوين له الأسبقية (الاحتياطي لمتغير البيئة للحساب الافتراضي فقط).
5.  ابدأ البوابة.

**⚠️ مهم:** أضف تحكمًا في الوصول (`allowFrom` أو `allowedRoles`) لمنع المستخدمين غير المصرح لهم من تشغيل البوت. `requireMention` تكون `true` افتراضيًا. التكوين الأدنى:

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw", // حساب البوت على Twitch
      accessToken: "oauth:abc123...", // رمز وصول OAuth (أو استخدم متغير البيئة OPENCLAW_TWITCH_ACCESS_TOKEN)
      clientId: "xyz789...", // معرف العميل من منشئ الرمز
      channel: "vevisk", // قناة Twitch التي يجب الانضمام إلى دردشتها (مطلوب)
      allowFrom: ["123456789"], // (موصى به) معرف مستخدم Twitch الخاص بك فقط - احصل عليه من https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    },
  },
}
```

## ما هو

-   قناة Twitch مملوكة للبوابة.
-   التوجيه الحتمي: الردود تعود دائمًا إلى Twitch.
-   كل حساب يعين إلى مفتاح جلسة معزول `agent::twitch:`.
-   `username` هو حساب البوت (الذي يقوم بالمصادقة)، `channel` هي غرفة الدردشة التي يجب الانضمام إليها.

## الإعداد (مفصل)

### إنشاء بيانات الاعتماد

استخدم [منشئ رمز Twitch](https://twitchtokengenerator.com/):

-   اختر **رمز البوت (Bot Token)**
-   تحقق من تحديد النطاقات `chat:read` و `chat:write`
-   انسخ **معرف العميل (Client ID)** و **رمز الوصول (Access Token)**

لا حاجة لتسجيل التطبيق يدويًا. تنتهي صلاحية الرموز بعد عدة ساعات.

### تكوين البوت

**متغير البيئة (الحساب الافتراضي فقط):**

```
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**أو التكوين:**

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
    },
  },
}
```

إذا تم تعيين كل من متغير البيئة والتكوين، فإن التكوين له الأسبقية.

### التحكم في الوصول (موصى به)

```json
{
  channels: {
    twitch: {
      allowFrom: ["123456789"], // (موصى به) معرف مستخدم Twitch الخاص بك فقط
    },
  },
}
```

يفضل استخدام `allowFrom` لقائمة السماح الصارمة. استخدم `allowedRoles` بدلاً من ذلك إذا كنت تريد وصولًا قائمًا على الأدوار. **الأدوار المتاحة:** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`. **لماذا معرفات المستخدمين؟** يمكن أن تتغير أسماء المستخدمين، مما يسمح بانتحال الشخصية. معرفات المستخدمين دائمة. ابحث عن معرف مستخدم Twitch الخاص بك: [https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/) (تحويل اسم مستخدم Twitch الخاص بك إلى معرف)

## تجديد الرمز (اختياري)

لا يمكن تجديد الرموز من [منشئ رمز Twitch](https://twitchtokengenerator.com/) تلقائيًا - قم بإعادة توليدها عند انتهاء صلاحيتها. للتجديد التلقائي للرموز، أنشئ تطبيق Twitch الخاص بك في [وحدة تحكم مطوري Twitch](https://dev.twitch.tv/console) وأضفه إلى التكوين:

```json
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token",
    },
  },
}
```

يقوم البوت تلقائيًا بتجديد الرموز قبل انتهاء صلاحيتها ويسجل أحداث التجديد.

## دعم الحسابات المتعددة

استخدم `channels.twitch.accounts` مع رموز لكل حساب. راجع [`gateway/configuration`](../gateway/configuration.md) للنمط المشترك. مثال (حساب بوت واحد في قناتين):

```json
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk",
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel",
        },
      },
    },
  },
}
```

**ملاحظة:** كل حساب يحتاج إلى رمز خاص به (رمز واحد لكل قناة).

## التحكم في الوصول

### قيود قائمة على الأدوار

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"],
        },
      },
    },
  },
}
```

### قائمة السماح حسب معرف المستخدم (الأكثر أمانًا)

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"],
        },
      },
    },
  },
}
```

### الوصول القائم على الأدوار (بديل)

`allowFrom` هي قائمة سماح صارمة. عند تعيينها، يُسمح فقط لهؤلاء المستخدمين. إذا كنت تريد وصولًا قائمًا على الأدوار، اترك `allowFrom` غير معيّن وقم بتكوين `allowedRoles` بدلاً من ذلك:

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

### تعطيل شرط الإشارة @

افتراضيًا، `requireMention` تكون `true`. لتعطيلها والرد على جميع الرسائل:

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false,
        },
      },
    },
  },
}
```

## استكشاف الأخطاء وإصلاحها

أولاً، قم بتشغيل أوامر التشخيص:

```bash
openclaw doctor
openclaw channels status --probe
```

### البوت لا يستجيب للرسائل

**تحقق من التحكم في الوصول:** تأكد من أن معرف المستخدم الخاص بك موجود في `allowFrom`، أو قم مؤقتًا بإزالة `allowFrom` وتعيين `allowedRoles: ["all"]` للاختبار. **تحقق من أن البوت في القناة:** يجب أن ينضم البوت إلى القناة المحددة في `channel`.

### مشاكل الرمز

**"فشل الاتصال" أو أخطاء المصادقة:**

-   تحقق من أن `accessToken` هو قيمة رمز وصول OAuth (تبدأ عادةً ببادئة `oauth:`)
-   تحقق من أن الرمز يحتوي على نطاقات `chat:read` و `chat:write`
-   إذا كنت تستخدم تجديد الرمز، تحقق من تعيين `clientSecret` و `refreshToken`

### تجديد الرمز لا يعمل

**تحقق من السجلات لأحداث التجديد:**

```bash
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

إذا رأيت "تم تعطيل تجديد الرمز (لا يوجد رمز تجديد)":

-   تأكد من توفير `clientSecret`
-   تأكد من توفير `refreshToken`

## التكوين

**تكوين الحساب:**

-   `username` - اسم مستخدم البوت
-   `accessToken` - رمز وصول OAuth مع `chat:read` و `chat:write`
-   `clientId` - معرف عميل Twitch (من منشئ الرمز أو تطبيقك)
-   `channel` - القناة للانضمام إليها (مطلوب)
-   `enabled` - تمكين هذا الحساب (افتراضي: `true`)
-   `clientSecret` - اختياري: للتجديد التلقائي للرمز
-   `refreshToken` - اختياري: للتجديد التلقائي للرمز
-   `expiresIn` - انتهاء صلاحية الرمز بالثواني
-   `obtainmentTimestamp` - الطابع الزمني للحصول على الرمز
-   `allowFrom` - قائمة السماح بمعرفات المستخدمين
-   `allowedRoles` - التحكم في الوصول القائم على الأدوار (`"moderator" | "owner" | "vip" | "subscriber" | "all"`)
-   `requireMention` - يتطلب الإشارة @ (افتراضي: `true`)

**خيارات المزود:**

-   `channels.twitch.enabled` - تمكين/تعطيل بدء تشغيل القناة
-   `channels.twitch.username` - اسم مستخدم البوت (تكوين حساب واحد مبسط)
-   `channels.twitch.accessToken` - رمز وصول OAuth (تكوين حساب واحد مبسط)
-   `channels.twitch.clientId` - معرف عميل Twitch (تكوين حساب واحد مبسط)
-   `channels.twitch.channel` - القناة للانضمام إليها (تكوين حساب واحد مبسط)
-   `channels.twitch.accounts.` - تكوين الحسابات المتعددة (جميع حقول الحساب أعلاه)

مثال كامل:

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

## إجراءات الأداة

يمكن للوكيل استدعاء `twitch` بالإجراء:

-   `send` - إرسال رسالة إلى قناة

مثال:

```json
{
  action: "twitch",
  params: {
    message: "Hello Twitch!",
    to: "#mychannel",
  },
}
```

## السلامة والعمليات

-   **عامل الرموز مثل كلمات المرور** - لا تضع الرموز في git أبدًا
-   **استخدم التجديد التلقائي للرموز** للبوتات طويلة الأمد
-   **استخدم قوائم السماح بمعرفات المستخدمين** بدلاً من أسماء المستخدمين للتحكم في الوصول
-   **راقب السجلات** لأحداث تجديد الرمز وحالة الاتصال
-   **قلل نطاق الرموز** - اطلب فقط `chat:read` و `chat:write`
-   **إذا علقت**: أعد تشغيل البوابة بعد التأكد من عدم وجود عملية أخرى تمتلك الجلسة

## الحدود

-   **500 حرف** لكل رسالة (يتم تقسيمها تلقائيًا عند حدود الكلمات)
-   يتم إزالة Markdown قبل التقسيم
-   لا يوجد تحديد لمعدل الإرسال (يستخدم الحدود المضمنة في Twitch)

[Tlon](./tlon.md)[WhatsApp](./whatsapp.md)