

  التكوين

  
# المجموعات

يعامل OpenClaw الدردشات الجماعية بشكل متسق عبر المنصات: WhatsApp وTelegram وDiscord وSlack وSignal وiMessage وMicrosoft Teams وZalo.

## مقدمة للمبتدئين (دقيقتان)

يعيش OpenClaw على حسابات المراسلة الخاصة بك. لا يوجد مستخدم بوت WhatsApp منفصل. إذا كنت **أنت** في مجموعة، يمكن لـ OpenClaw رؤية تلك المجموعة والرد فيها. السلوك الافتراضي:

-   المجموعات مقيدة (`groupPolicy: "allowlist"`).
-   تتطلب الردود إشارة ما لم تقم بتعطيل بوابة الإشارات صراحةً.

الترجمة: يمكن للمرسلين المسموح لهم تشغيل OpenClaw عن طريق الإشارة إليه.

> ملخص سريع
>
> -   يتم التحكم في **وصول الرسائل المباشرة** بواسطة `*.allowFrom`.
> -   يتم التحكم في **وصول المجموعة** بواسطة `*.groupPolicy` + القوائم المسموح بها (`*.groups`, `*.groupAllowFrom`).
> -   يتم التحكم في **تشغيل الرد** بواسطة بوابة الإشارات (`requireMention`, `/activation`).

التدفق السريع (ما يحدث لرسالة المجموعة):

```
groupPolicy? disabled -> تجاهل
groupPolicy? allowlist -> المجموعة مسموحة؟ لا -> تجاهل
requireMention? نعم -> تمت الإشارة؟ لا -> تخزين للسياق فقط
بخلاف ذلك -> رد
```

![تدفق رسالة المجموعة](../images/channels-groups-flow.svg.md) إذا كنت تريد…

| الهدف | ما يجب تعيينه |
| --- | --- |
| السماح بجميع المجموعات ولكن الرد فقط عند @mentions | `groups: { "*": { requireMention: true } }` |
| تعطيل جميع ردود المجموعة | `groupPolicy: "disabled"` |
| مجموعات محددة فقط | `groups: { "<group-id>": { ... } }` (بدون مفتاح `"*"`) |
| يمكنك فقط التشغيل في المجموعات | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

## مفاتيح الجلسة

-   تستخدم جلسات المجموعة مفاتيح الجلسة `agent:::group:` (تستخدم الغرف/القنوات `agent:::channel:`).
-   تضيف مواضيع منتدى Telegram `:topic:` إلى معرف المجموعة بحيث يكون لكل موضوع جلسته الخاصة.
-   تستخدم الدردشات المباشرة الجلسة الرئيسية (أو لكل مرسل إذا تم تكوينه).
-   يتم تخطي نبضات القلب لجلسات المجموعة.

## النمط: الرسائل المباشرة الشخصية + المجموعات العامة (وكيل واحد)

نعم — يعمل هذا بشكل جيد إذا كان حركة المرور "الشخصية" الخاصة بك هي **الرسائل المباشرة** وحركة المرور "العامة" الخاصة بك هي **المجموعات**. السبب: في وضع الوكيل الواحد، تهبط الرسائل المباشرة عادةً في مفتاح الجلسة **الرئيسي** (`agent:main:main`)، بينما تستخدم المجموعات دائمًا مفاتيح الجلسة **غير الرئيسية** (`agent:main::group:`). إذا قمت بتمكين الحماية باستخدام `mode: "non-main"`، فإن تلك الجلسات الجماعية تعمل في Docker بينما تبقى جلسة الرسائل المباشرة الرئيسية على المضيف. هذا يمنحك "دماغ" وكيل واحد (مساحة عمل مشتركة + ذاكرة)، ولكن وضعي تنفيذ:

-   **الرسائل المباشرة**: أدوات كاملة (المضيف)
-   **المجموعات**: حماية + أدوات مقيدة (Docker)

> إذا كنت بحاجة إلى مساحات عمل/شخصيات منفصلة حقًا ("الشخصية" و"العامة" يجب ألا تختلط أبدًا)، استخدم وكيلًا ثانيًا + ربط. انظر [التوجيه متعدد الوكلاء](../concepts/multi-agent.md).

مثال (الرسائل المباشرة على المضيف، المجموعات محمية + أدوات مراسلة فقط):

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // المجموعات/القنوات غير رئيسية -> محمية
        scope: "session", // أقوى عزل (حاوية واحدة لكل مجموعة/قناة)
        workspaceAccess: "none",
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        // إذا كان السماح غير فارغ، يتم حظر كل شيء آخر (الرفض لا يزال يفوز).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"],
      },
    },
  },
}
```

تريد "يمكن للمجموعات رؤية المجلد X فقط" بدلاً من "لا وصول للمضيف"؟ احتفظ بـ `workspaceAccess: "none"` وقم بتثبيت المسارات المسموح بها فقط في الحماية:

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // hostPath:containerPath:mode
            "/home/user/FriendsShared:/data:ro",
          ],
        },
      },
    },
  },
}
```

ذات صلة:

-   مفاتيح التكوين والإعدادات الافتراضية: [تكوين البوابة](../gateway/configuration.md#agentsdefaultssandbox)
-   تصحيح سبب حظر أداة: [الحماية مقابل سياسة الأداة مقابل المرتفعة](../gateway/sandbox-vs-tool-policy-vs-elevated.md)
-   تفاصيل نقاط الربط: [الحماية](../gateway/sandboxing.md#custom-bind-mounts)

## تسميات العرض

-   تستخدم تسميات واجهة المستخدم `displayName` عند توفرها، بتنسيق `:`.
-   `#room` محجوز للغرف/القنوات؛ تستخدم الدردشات الجماعية `g-` (أحرف صغيرة، المسافات -> `-`، احتفظ بـ `#@+._-`).

## سياسة المجموعة

تحكم في كيفية معالجة رسائل المجموعة/الغرفة لكل قناة:

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789"], // معرف مستخدم Telegram رقمي (يمكن للساحر حل @username)
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: { channels: { help: { allow: true } } },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
    },
  },
}
```

| السياسة | السلوك |
| --- | --- |
| `"open"` | تتجاوز المجموعات القوائم المسموح بها؛ لا تزال بوابة الإشارات تنطبق. |
| `"disabled"` | حظر جميع رسائل المجموعة بالكامل. |
| `"allowlist"` | السماح فقط بالمجموعات/الغرف التي تطابق القائمة المسموح بها المكونة. |

ملاحظات:

-   `groupPolicy` منفصل عن بوابة الإشارات (والتي تتطلب @mentions).
-   WhatsApp/Telegram/Signal/iMessage/Microsoft Teams/Zalo: استخدم `groupAllowFrom` (الاحتياطي: `allowFrom` صريح).
-   موافقات اقتران الرسائل المباشرة (إدخالات مخزن `*-allowFrom`) تنطبق على وصول الرسائل المباشرة فقط؛ يبقى تفويض مرسل المجموعة صريحًا للقوائم المسموح بها للمجموعة.
-   Discord: تستخدم القائمة المسموح بها `channels.discord.guilds..channels`.
-   Slack: تستخدم القائمة المسموح بها `channels.slack.channels`.
-   Matrix: تستخدم القائمة المسموح بها `channels.matrix.groups` (معرفات الغرف، الأسماء المستعارة، أو الأسماء). استخدم `channels.matrix.groupAllowFrom` لتقييد المرسلين؛ مدعومة أيضًا القوائم المسموح بها `users` لكل غرفة.
-   يتم التحكم في الرسائل المباشرة الجماعية بشكل منفصل (`channels.discord.dm.*`, `channels.slack.dm.*`).
-   يمكن أن تطابق القائمة المسموح بها في Telegram معرفات المستخدمين (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) أو أسماء المستخدمين (`"@alice"` أو `"alice"`)؛ البادئات غير حساسة لحالة الأحرف.
-   الافتراضي هو `groupPolicy: "allowlist"`؛ إذا كانت قائمة المجموعات المسموح بها فارغة، يتم حظر رسائل المجموعة.
-   سلامة وقت التشغيل: عندما يكون كتلة المزود مفقودًا تمامًا (`channels.` غائب)، تعود سياسة المجموعة إلى وضع الإغلاق عند الفشل (عادةً `allowlist`) بدلاً من وراثة `channels.defaults.groupPolicy`.

النموذج الذهني السريع (ترتيب التقييم لرسائل المجموعة):

1.  `groupPolicy` (open/disabled/allowlist)
2.  القوائم المسموح بها للمجموعة (`*.groups`, `*.groupAllowFrom`, قائمة مسموح بها خاصة بالقناة)
3.  بوابة الإشارات (`requireMention`, `/activation`)

## بوابة الإشارات (الافتراضي)

تتطلب رسائل المجموعة إشارة ما لم يتم تجاوزها لكل مجموعة. الإعدادات الافتراضية موجودة لكل نظام فرعي تحت `*.groups."*"`. يعتبر الرد على رسالة بوت بمثابة إشارة ضمنية (عندما تدعم القناة بيانات وصفية للرد). هذا ينطبق على Telegram وWhatsApp وSlack وDiscord وMicrosoft Teams.

```json
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false },
      },
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false },
      },
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50,
        },
      },
    ],
  },
}
```

ملاحظات:

-   `mentionPatterns` هي تعبيرات عادية غير حساسة لحالة الأحرف.
-   لا تزال المنصات التي توفر إشارات صريحة تمرر؛ الأنماط هي احتياطي.
-   تجاوز لكل وكيل: `agents.list[].groupChat.mentionPatterns` (مفيد عندما يشارك عدة وكلاء مجموعة واحدة).
-   يتم فرض بوابة الإشارات فقط عندما يكون اكتشاف الإشارة ممكنًا (الإشارات الأصلية أو `mentionPatterns` مكونة).
-   الإعدادات الافتراضية لـ Discord موجودة في `channels.discord.guilds."*"` (قابلة للتجاوز لكل نقابة/قناة).
-   سياق سجل المجموعة مغلف بشكل موحد عبر القنوات وهو **معلق فقط** (الرسائل التي تم تخطيها بسبب بوابة الإشارات)؛ استخدم `messages.groupChat.historyLimit` للافتراضي العام و `channels..historyLimit` (أو `channels..accounts.*.historyLimit`) للتجاوزات. عيّن `0` لتعطيل.

## قيود أداة المجموعة/القناة (اختياري)

تدعم بعض تكوينات القناة تقييد الأدوات المتاحة **داخل مجموعة/غرفة/قناة محددة**.

-   `tools`: السماح/الرفض للأدوات للمجموعة بأكملها.
-   `toolsBySender`: تجاوزات لكل مرسل داخل المجموعة. استخدم بادئات المفاتيح الصريحة: `id:`, `e164:`, `username:`, `name:`, و `"*"` كحرف بدل. لا تزال المفاتيح غير المسبوقة القديمة مقبولة ومطابقة كـ `id:` فقط.

ترتيب الدقة (الأكثر تحديدًا يفوز):

1.  مطابقة `toolsBySender` للمجموعة/القناة
2.  `tools` للمجموعة/القناة
3.  مطابقة `toolsBySender` الافتراضية (`"*"`)
4.  `tools` الافتراضية (`"*"`)

مثال (Telegram):

```json
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "id:123456789": { alsoAllow: ["exec"] },
          },
        },
      },
    },
  },
}
```

ملاحظات:

-   يتم تطبيق قيود أداة المجموعة/القناة بالإضافة إلى سياسة الأداة العالمية/للوكيل (الرفض لا يزال يفوز).
-   تستخدم بعض القنوات تداخلًا مختلفًا للغرف/القنوات (مثل Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`).

## القوائم المسموح بها للمجموعة

عند تكوين `channels.whatsapp.groups` أو `channels.telegram.groups` أو `channels.imessage.groups`، تعمل المفاتيح كقائمة مسموح بها للمجموعة. استخدم `"*"` للسماح بجميع المجموعات مع الاستمرار في تعيين سلوك الإشارة الافتراضي. النوايا الشائعة (نسخ/لصق):

1.  تعطيل جميع ردود المجموعة

```json
{
  channels: { whatsapp: { groupPolicy: "disabled" } },
}
```

2.  السماح بمجموعات محددة فقط (WhatsApp)

```json
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false },
      },
    },
  },
}
```

3.  السماح بجميع المجموعات ولكن تتطلب إشارة (صريحة)

```json
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

4.  يمكن للمالك فقط التشغيل في المجموعات (WhatsApp)

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## التنشيط (للمالك فقط)

يمكن لملاك المجموعة تبديل التنشيط لكل مجموعة:

-   `/activation mention`
-   `/activation always`

يتم تحديد المالك بواسطة `channels.whatsapp.allowFrom` (أو E.164 الذاتي للبوت عند عدم التعيين). أرسل الأمر كرسالة منفردة. تتجاهل المنصات الأخرى حاليًا `/activation`.

## حقول السياق

تحميلات المجموعة الواردة تعيّن:

-   `ChatType=group`
-   `GroupSubject` (إذا كان معروفًا)
-   `GroupMembers` (إذا كان معروفًا)
-   `WasMentioned` (نتيجة بوابة الإشارات)
-   تتضمن مواضيع منتدى Telegram أيضًا `MessageThreadId` و `IsForum`.

يتضمن موجه نظام الوكيل مقدمة جماعية في المنعطف الأول من جلسة مجموعة جديدة. يذكر النموذج بالرد مثل الإنسان، وتجنب جداول Markdown، وتجنب كتابة تسلسلات `\n` الحرفية.

## تفاصيل iMessage

-   فضّل `chat_id:` عند التوجيه أو السماح.
-   قائمة الدردشات: `imsg chats --limit 20`.
-   تذهب ردود المجموعة دائمًا إلى نفس `chat_id`.

## تفاصيل WhatsApp

انظر [رسائل المجموعة](./group-messages.md) لسلوك WhatsApp فقط (حقن السجل، تفاصيل معالجة الإشارات).

[رسائل المجموعة](./group-messages.md)[مجموعات البث](./broadcast-groups.md)

---