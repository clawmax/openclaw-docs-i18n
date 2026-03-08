title: "دليل إعداد وتكوين وقنوات IRC لأوبنكلو"
description: "تعلم كيفية تكوين أوبنكلو لـ IRC، وإعداد سياسات الأمان، وإدارة التحكم في الوصول، واستكشاف الأخطاء الشائعة وإصلاحها لـ Libera.Chat والشبكات الأخرى."
keywords: ["أوبنكلو irc", "تكوين بوت IRC", "أمان قناة IRC", "التحكم في الوصول لـ IRC", "بوابة الإشارات لـ IRC", "إعداد nickserv", "استكشاف أخطاء IRC وإصلاحها", "سياسة مجموعة IRC"]
---

  منصات المراسلة

  
# IRC

استخدم IRC عندما تريد أوبنكلو في القنوات الكلاسيكية (`#room`) والرسائل المباشرة. يُشحن IRC كإضافة ملحقة، ولكن يتم تكوينه في الإعدادات الرئيسية تحت `channels.irc`.

## البدء السريع

1.  قم بتمكين إعدادات IRC في `~/.openclaw/openclaw.json`.
2.  عيّن على الأقل:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3.  ابدأ/أعد تشغيل البوابة:

```bash
openclaw gateway run
```

## الإعدادات الافتراضية للأمان

-   `channels.irc.dmPolicy` الافتراضي هو `"pairing"`.
-   `channels.irc.groupPolicy` الافتراضي هو `"allowlist"`.
-   مع `groupPolicy="allowlist"`، عيّن `channels.irc.groups` لتحديد القنوات المسموح بها.
-   استخدم TLS (`channels.irc.tls=true`) ما لم تقبل عمدًا النقل بنص عادي.

## التحكم في الوصول

هناك "بوابتان" منفصلتان لقنوات IRC:

1.  **وصول القناة** (`groupPolicy` + `groups`): ما إذا كان البوت يقبل الرسائل من قناة على الإطلاق.
2.  **وصول المرسل** (`groupAllowFrom` / `groups["#channel"].allowFrom` لكل قناة): من المسموح له بتشغيل البوت داخل تلك القناة.

مفاتيح التكوين:

-   قائمة السماح للرسائل المباشرة (وصول مرسل الرسائل المباشرة): `channels.irc.allowFrom`
-   قائمة السماح لمرسلي المجموعة (وصول مرسل القناة): `channels.irc.groupAllowFrom`
-   عناصر التحكم لكل قناة (قناة + مرسل + قواعد الإشارة): `channels.irc.groups["#channel"]`
-   `channels.irc.groupPolicy="open"` تسمح بالقنوات غير المُهيأة (**ما تزال مقيدة بالإشارة افتراضيًا**)

يجب أن تستخدم مدخلات قائمة السماح هويات مرسل ثابتة (`nick!user@host`). مطابقة الاسم المجرد قابلة للتغيير ولا يتم تمكينها إلا عند `channels.irc.dangerouslyAllowNameMatching: true`.

### خطأ شائع: allowFrom مخصص للرسائل المباشرة، وليس للقنوات

إذا رأيت سجلات مثل:

-   `irc: drop group sender alice!ident@host (policy=allowlist)`

…فهذا يعني أن المرسل لم يكن مسموحًا له **بـرسائل المجموعة/القناة**. أصلحه إما عن طريق:

-   تعيين `channels.irc.groupAllowFrom` (عام لجميع القنوات)، أو
-   تعيين قوائم السماح للمرسل لكل قناة: `channels.irc.groups["#channel"].allowFrom`

مثال (السماح لأي شخص في `#tuirc-dev` بالتحدث إلى البوت):

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## تشغيل الرد (الإشارات)

حتى لو كانت القناة مسموحًا بها (عبر `groupPolicy` + `groups`) وكان المرسل مسموحًا له، فإن أوبنكلو يفرض افتراضيًا **بوابة الإشارة** في سياقات المجموعة. هذا يعني أنك قد ترى سجلات مثل `drop channel … (missing-mention)` ما لم تتضمن الرسالة نمط إشارة يطابق البوت. لجعل البوت يرد في قناة IRC **بدون الحاجة إلى إشارة**، قم بتعطيل بوابة الإشارة لتلك القناة:

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

أو للسماح **بجميع** قنوات IRC (بدون قائمة سماح لكل قناة) والرد مع ذلك بدون إشارات:

```json
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## ملاحظة أمنية (موصى بها للقنوات العامة)

إذا سمحت بـ `allowFrom: ["*"]` في قناة عامة، يمكن لأي شخص إرسال أوامر للبوت. لتقليل المخاطر، قم بتقييد الأدوات لتلك القناة.

### نفس الأدوات للجميع في القناة

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### أدوات مختلفة لكل مرسل (المالك يحصل على صلاحيات أكبر)

استخدم `toolsBySender` لتطبيق سياسة أشد صرامة على `"*"` وأخرى أكثر مرونة على اسمك المستعار:

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            "id:eigen": {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

ملاحظات:

-   يجب أن تستخدم مفاتيح `toolsBySender` البادئة `id:` لقيم هوية مرسل IRC: `id:eigen` أو `id:eigen!~eigen@174.127.248.171` لمطابقة أقوى.
-   لا تزال المفاتيح غير المسبوقة بالبادئة مقبولة ويتم مطابقتها كـ `id:` فقط.
-   أول سياسة مرسل مطابقة تفوز؛ `"*"` هو الحلقة الاحتياطية العامة.

لمزيد من المعلومات حول وصول المجموعة مقابل بوابة الإشارة (وكيفية تفاعلهما)، راجع: [/channels/groups](./groups.md).

## NickServ

للتعريف مع NickServ بعد الاتصال:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

تسجيل لمرة واحدة اختياري عند الاتصال:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

عطّل `register` بعد تسجيل الاسم المستعار لتجنب محاولات REGISTER المتكررة.

## متغيرات البيئة

الحساب الافتراضي يدعم:

-   `IRC_HOST`
-   `IRC_PORT`
-   `IRC_TLS`
-   `IRC_NICK`
-   `IRC_USERNAME`
-   `IRC_REALNAME`
-   `IRC_PASSWORD`
-   `IRC_CHANNELS` (مفصولة بفواصل)
-   `IRC_NICKSERV_PASSWORD`
-   `IRC_NICKSERV_REGISTER_EMAIL`

## استكشاف الأخطاء وإصلاحها

-   إذا اتصل البوت ولكن لم يرد أبدًا في القنوات، تحقق من `channels.irc.groups` **و** ما إذا كانت بوابة الإشارة ترفض الرسائل (`missing-mention`). إذا كنت تريد أن يرد بدون إشارات، عيّن `requireMention:false` للقناة.
-   إذا فشل تسجيل الدخول، تحقق من توفر الاسم المستعار وكلمة مرور الخادم.
-   إذا فشل TLS على شبكة مخصصة، تحقق من المضيف/المنفذ وإعداد الشهادة.

[iMessage](./imessage.md)[LINE](./line.md)