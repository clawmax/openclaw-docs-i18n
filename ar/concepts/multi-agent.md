title: "التوجيه والإعداد متعدد الوكلاء في بوابة OpenClaw"
description: "تعلم كيفية إعداد وكلاء ذكاء اصطناعي متعددة معزولة مع مساحات عمل منفصلة، وحسابات قنوات، وقواعد توجيه في OpenClaw لإدارة شخصيات وجلسات متميزة."
keywords: ["توجيه متعدد الوكلاء", "وكلاء openclaw", "وكلاء معزولون", "ارتباطات القناة", "مساحة عمل الوكيل", "إدارة الجلسات", "توجيه واتساب", "وكلاء بوت ديسكورد"]
---

  متعدد الوكلاء

  
# التوجيه متعدد الوكلاء

الهدف: وكلاء *معزولة* متعددة (مساحة عمل منفصلة + `agentDir` + جلسات)، بالإضافة إلى حسابات قنوات متعددة (مثل واتسابين) في بوابة واحدة قيد التشغيل. يتم توجيه الوارد إلى وكيل عبر الارتباطات.

## ما هو "الوكيل الواحد"؟

**الوكيل** هو دماغ ذو نطاق كامل خاص به:

-   **مساحة العمل** (الملفات، AGENTS.md/SOUL.md/USER.md، الملاحظات المحلية، قواعد الشخصية).
-   **دليل الحالة** (`agentDir`) لملفات تعريف المصادقة، سجل النماذج، والإعدادات الخاصة بكل وكيل.
-   **مخزن الجلسات** (سجل المحادثة + حالة التوجيه) تحت `~/.openclaw/agents//sessions`.

ملفات تعريف المصادقة تكون **خاصة بكل وكيل**. كل وكيل يقرأ من ملفاته الخاصة:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

بيانات اعتماد الوكيل الرئيسي **لا** تتم مشاركتها تلقائياً. لا تعيد استخدام `agentDir` عبر وكلاء متعددين (يسبب ذلك تعارضات في المصادقة/الجلسات). إذا أردت مشاركة بيانات الاعتماد، انسخ `auth-profiles.json` إلى `agentDir` الخاص بالوكيل الآخر. المهارات تكون خاصة بكل وكيل عبر مجلد `skills/` الخاص بكل مساحة عمل، مع مهارات مشتركة متاحة من `~/.openclaw/skills`. راجع [المهارات: خاصة بالوكيل مقابل المشتركة](../tools/skills.md#per-agent-vs-shared-skills). يمكن للبوابة استضافة **وكيل واحد** (الافتراضي) أو **عدة وكلاء** جنباً إلى جنب. **ملاحظة حول مساحة العمل:** مساحة العمل لكل وكيل هي **مسار العمل الافتراضي (cwd)**، وليست صندوقاً حصيناً. يتم حل المسارات النسبية داخل مساحة العمل، لكن المسارات المطلقة يمكنها الوصول إلى مواقع أخرى على المضيف ما لم يتم تمكين العزل. راجع [العزل](../gateway/sandboxing.md).

## المسارات (خريطة سريعة)

-   الإعدادات: `~/.openclaw/openclaw.json` (أو `OPENCLAW_CONFIG_PATH`)
-   دليل الحالة: `~/.openclaw` (أو `OPENCLAW_STATE_DIR`)
-   مساحة العمل: `~/.openclaw/workspace` (أو `~/.openclaw/workspace-`)
-   دليل الوكيل: `~/.openclaw/agents//agent` (أو `agents.list[].agentDir`)
-   الجلسات: `~/.openclaw/agents//sessions`

### وضع الوكيل الواحد (الافتراضي)

إذا لم تفعل شيئاً، يعمل OpenClaw بوكيل واحد:

-   `agentId` يكون افتراضياً **`main`**.
-   يتم تخزين الجلسات كمفتاح `agent:main:`.
-   مساحة العمل الافتراضية هي `~/.openclaw/workspace` (أو `~/.openclaw/workspace-` عند تعيين `OPENCLAW_PROFILE`).
-   الحالة الافتراضية تكون في `~/.openclaw/agents/main/agent`.

## مساعد الوكيل

استخدم معالج الوكيل لإضافة وكيل معزول جديد:

```bash
openclaw agents add work
```

ثم أضف `bindings` (أو دع المعالج يقوم بذلك) لتوجيه الرسائل الواردة. تحقق باستخدام:

```bash
openclaw agents list --bindings
```

## البدء السريع

### الخطوة 1: إنشاء مساحة عمل لكل وكيل

استخدم المعالج أو أنشئ مساحات العمل يدوياً:

```bash
openclaw agents add coding
openclaw agents add social
```

يحصل كل وكيل على مساحة عمل خاصة به تحتوي على `SOUL.md`، `AGENTS.md`، و `USER.md` اختياري، بالإضافة إلى `agentDir` مخصص ومخزن جلسات تحت `~/.openclaw/agents/`.

### الخطوة 2: إنشاء حسابات القنوات

أنشئ حساباً واحداً لكل وكيل على القنوات المفضلة لديك:

-   ديسكورد: بوت واحد لكل وكيل، فعّل نية محتوى الرسالة، انسخ كل رمز مميز.
-   تيليجرام: بوت واحد لكل وكيل عبر BotFather، انسخ كل رمز مميز.
-   واتساب: اربط كل رقم هاتف لكل حساب.

```bash
openclaw channels login --channel whatsapp --account work
```

راجع أدلة القنوات: [ديسكورد](../channels/discord.md)، [تيليجرام](../channels/telegram.md)، [واتساب](../channels/whatsapp.md).

### الخطوة 3: إضافة الوكلاء والحسابات والارتباطات

أضف الوكلاء تحت `agents.list`، وحسابات القنوات تحت `channels..accounts`، واربطها باستخدام `bindings` (أمثلة أدناه).

### الخطوة 4: إعادة التشغيل والتحقق

```bash
openclaw gateway restart
openclaw agents list --bindings
openclaw channels status --probe
```

## وكلاء متعددون = أشخاص متعددون، شخصيات متعددة

مع **وكلاء متعددين**، يصبح كل `agentId` **شخصية معزولة بالكامل**:

-   **أرقام هواتف/حسابات مختلفة** (لكل `accountId` للقناة).
-   **شخصيات مختلفة** (ملفات مساحة العمل الخاصة بكل وكيل مثل `AGENTS.md` و `SOUL.md`).
-   **مصادقة وجلسات منفصلة** (لا تتداخل ما لم يتم تمكين ذلك صراحةً).

هذا يسمح **لأشخاص متعددين** بمشاركة خادم بوابة واحد مع الحفاظ على "أدمغة" الذكاء الاصطناعي وبياناتهم معزولة.

## رقم واتساب واحد، أشخاص متعددون (تقسيم المراسلة المباشرة)

يمكنك توجيه **مراسلات واتساب مباشرة مختلفة** إلى وكلاء مختلفة مع البقاء على **حساب واتساب واحد**. قم بالمطابقة على معرف المرسل E.164 (مثل `+15551234567`) مع `peer.kind: "direct"`. الردود ستأتي من نفس رقم واتساب (لا هوية مرسل خاصة بكل وكيل). تفصيل مهم: تنهار الدردشات المباشرة إلى **مفتاح الجلسة الرئيسي** للوكيل، لذا فإن العزل الحقيقي يتطلب **وكيل واحد لكل شخص**. مثال:

```json
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" },
    ],
  },
  bindings: [
    {
      agentId: "alex",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230001" } },
    },
    {
      agentId: "mia",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230002" } },
    },
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"],
    },
  },
}
```

ملاحظات:

-   التحكم في الوصول للمراسلة المباشرة يكون **عالمياً لكل حساب واتساب** (الاقتران/القائمة المسموحة)، وليس لكل وكيل.
-   للمجموعات المشتركة، اربط المجموعة بوكيل واحد أو استخدم [مجموعات البث](../channels/broadcast-groups.md).

## قواعد التوجيه (كيف تختار الرسائل وكيلاً)

الارتباطات تكون **حتمية** و **الأكثر تحديداً يفوز**:

1.  مطابقة `peer` (معرف مباشر/مجموعة/قناة مطابق تماماً)
2.  مطابقة `parentPeer` (التوريث في الموضوع)
3.  `guildId + roles` (توجيه دور ديسكورد)
4.  `guildId` (ديسكورد)
5.  `teamId` (سلاك)
6.  مطابقة `accountId` لقناة
7.  مطابقة على مستوى القناة (`accountId: "*"`)
8.  الرجوع إلى الوكيل الافتراضي (`agents.list[].default`، وإلا أول مدخل في القائمة، الافتراضي: `main`)

إذا تطابق ارتباطان متعددان في نفس المستوى، فإن الأول في ترتيب الإعدادات يفوز. إذا حدد ارتباط حقول مطابقة متعددة (على سبيل المثال `peer` + `guildId`)، تكون جميع الحقول المحددة مطلوبة (دلالات `AND`). تفصيل مهم حول نطاق الحساب:

-   الارتباط الذي يحذف `accountId` يطابق الحساب الافتراضي فقط.
-   استخدم `accountId: "*"` للرجوع على مستوى القناة عبر جميع الحسابات.
-   إذا أضفت لاحقاً نفس الارتباط لنفس الوكيل مع معرف حساب صريح، يقوم OpenClaw بترقية الارتباط الحالي الخاص بالقناة فقط ليصبح ذا نطاق حسابي بدلاً من تكراره.

## حسابات متعددة / أرقام هواتف متعددة

القنوات التي تدعم **حسابات متعددة** (مثل واتساب) تستخدم `accountId` لتحديد كل تسجيل دخول. يمكن توجيه كل `accountId` إلى وكيل مختلف، لذا يمكن لخادم واحد استضافة أرقام هواتف متعددة دون خلط الجلسات. إذا أردت حساباً افتراضياً على مستوى القناة عند حذف `accountId`، عيّن `channels..defaultAccount` (اختياري). عند عدم التعيين، يعود OpenClaw إلى `default` إذا كان موجوداً، وإلا أول معرف حساب تم تكوينه (مرتب). القنوات الشائعة التي تدعم هذا النمط تشمل:

-   `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`
-   `irc`, `line`, `googlechat`, `mattermost`, `matrix`, `nextcloud-talk`
-   `bluebubbles`, `zalo`, `zalouser`, `nostr`, `feishu`

## المفاهيم

-   `agentId`: "دماغ" واحد (مساحة عمل، مصادقة خاصة بالوكيل، مخزن جلسات خاص بالوكيل).
-   `accountId`: مثيل حساب قناة واحد (مثل حساب واتساب `"personal"` مقابل `"biz"`).
-   `binding`: يوجه الرسائل الواردة إلى `agentId` عبر `(channel, accountId, peer)` واختيارياً معرفات النقابة/الفريق.
-   تنهار الدردشات المباشرة إلى `agent::` ("الرئيسي" الخاص بالوكيل؛ `session.mainKey`).

## أمثلة على المنصات

### بوتات ديسكورد لكل وكيل

كل حساب بوت ديسكورد يخريط إلى `accountId` فريد. اربط كل حساب بوكيل واحتفظ بقوائم مسموحة لكل بوت.

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "coding", workspace: "~/.openclaw/workspace-coding" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "discord", accountId: "default" } },
    { agentId: "coding", match: { channel: "discord", accountId: "coding" } },
  ],
  channels: {
    discord: {
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN_MAIN",
          guilds: {
            "123456789012345678": {
              channels: {
                "222222222222222222": { allow: true, requireMention: false },
              },
            },
          },
        },
        coding: {
          token: "DISCORD_BOT_TOKEN_CODING",
          guilds: {
            "123456789012345678": {
              channels: {
                "333333333333333333": { allow: true, requireMention: false },
              },
            },
          },
        },
      },
    },
  },
}
```

ملاحظات:

-   ادعُ كل بوت إلى النقابة وفعّل نية محتوى الرسالة.
-   توجد الرموز المميزة في `channels.discord.accounts..token` (يمكن للحساب الافتراضي استخدام `DISCORD_BOT_TOKEN`).

### بوتات تيليجرام لكل وكيل

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "alerts", workspace: "~/.openclaw/workspace-alerts" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "alerts", match: { channel: "telegram", accountId: "alerts" } },
  ],
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "123456:ABC...",
          dmPolicy: "pairing",
        },
        alerts: {
          botToken: "987654:XYZ...",
          dmPolicy: "allowlist",
          allowFrom: ["tg:123456789"],
        },
      },
    },
  },
}
```

ملاحظات:

-   أنشئ بوتاً واحداً لكل وكيل باستخدام BotFather وانسخ كل رمز مميز.
-   توجد الرموز المميزة في `channels.telegram.accounts..botToken` (يمكن للحساب الافتراضي استخدام `TELEGRAM_BOT_TOKEN`).

### أرقام واتساب لكل وكيل

اربط كل حساب قبل بدء تشغيل البوابة:

```bash
openclaw channels login --channel whatsapp --account personal
openclaw channels login --channel whatsapp --account biz
```

`~/.openclaw/openclaw.json` (JSON5):

```json
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // التوجيه الحتمي: أول تطابق يفوز (الأكثر تحديداً أولاً).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // تجاوز اختياري لكل نظير (مثال: إرسال مجموعة محددة إلى وكيل العمل).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // معطّل افتراضياً: يجب تمكين المراسلة بين الوكيل والوكيل صراحةً وإضافتها للقائمة المسموحة.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // تجاوز اختياري. الافتراضي: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // تجاوز اختياري. الافتراضي: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

## مثال: دردشة واتساب يومية + عمل عميق على تيليجرام

قسّم حسب القناة: وجّه واتساب إلى وكيل سريع يومي ووجّه تيليجرام إلى وكيل Opus.

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

ملاحظات:

-   إذا كان لديك حسابات متعددة لقناة ما، أضف `accountId` إلى الارتباط (على سبيل المثال `{ channel: "whatsapp", accountId: "personal" }`).
-   لتوجيه مراسلة مباشرة/مجموعة واحدة إلى Opus مع بقاء الباقي على Chat، أضف ارتباط `match.peer` لذلك النظير؛ مطابقات النظير تفوز دائماً على القواعد العامة للقناة.

## مثال: نفس القناة، نظير واحد إلى Opus

احتفظ بواتساب على الوكيل السريع، لكن وجّه مراسلة مباشرة واحدة إلى Opus:

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    {
      agentId: "opus",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551234567" } },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
  ],
}
```

ارتباطات النظير تفوز دائماً، لذا احتفظ بها فوق القاعدة العامة للقناة.

## وكيل عائلي مرتبط بمجموعة واتساب

اربط وكيلاً عائلياً مخصصاً بمجموعة واتساب واحدة، مع بوابة ذكر وسياسة أدوات أكثر صرامة:

```json
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"],
        },
        sandbox: {
          mode: "all",
          scope: "agent",
        },
        tools: {
          allow: [
            "exec",
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"],
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" },
      },
    },
  ],
}
```

ملاحظات:

-   قوائم السماح/الرفض للأدوات هي **أدوات**، وليست مهارات. إذا احتاجت مهارة لتشغيل ثنائي، تأكد من أن `exec` مسموح به وأن الثنائي موجود في الصندوق الحصين.
-   للبوابة الأكثر صرامة، عيّن `agents.list[].groupChat.mentionPatterns` واحتفظ بقوائم السماح للمجموعات مفعلة للقناة.

## إعدادات الصندوق الحصين والأدوات الخاصة بكل وكيل

بدءاً من الإصدار v2026.1.6، يمكن أن يكون لكل وكيل صندوق حصين وقيود أدوات خاصة به:

```json
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // لا يوجد صندوق حصين للوكيل الشخصي
        },
        // لا توجد قيود على الأدوات - جميع الأدوات متاحة
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // دائمًا في صندوق حصين
          scope: "agent",  // حاوية واحدة لكل وكيل
          docker: {
            // إعداد لمرة واحدة اختياري بعد إنشاء الحاوية
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // أداة القراءة فقط
          deny: ["exec", "write", "edit", "apply_patch"],    // رفض الآخرين
        },
      },
    ],
  },
}
```

ملاحظة: `setupCommand` موجود تحت `sandbox.docker` ويعمل مرة واحدة عند إنشاء الحاوية. يتم تجاهل تجاوزات `sandbox.docker.*` الخاصة بكل وكيل عندما يكون النطاق المحلول هو `"shared"`. **المزايا:**

-   **عزل أمني**: قيّد الأدوات للوكلاء غير الموثوق بهم
-   **تحكم في الموارد**: ضع وكلاء معينة في صندوق حصين مع إبقاء الآخرين على المضيف
-   **سياسات مرنة**: أذونات مختلفة لكل وكيل

ملاحظة: `tools.elevated` يكون **عالمياً** ويعتمد على المرسل؛ ولا يمكن تكوينه لكل وكيل. إذا كنت بحاجة إلى حدود خاصة بكل وكيل، استخدم `agents.list[].tools` لرفض `exec`. لاستهداف المجموعات، استخدم `agents.list[].groupChat.mentionPatterns` بحيث تنتقل @mentions بوضوح إلى الوكيل المقصود. راجع [الصندوق الحصين والأدوات متعددة الوكلاء](../tools/multi-agent-sandbox-tools.md) للحصول على أمثلة مفصلة.

[الضغط](./compaction.md)[الحضور](./presence.md)