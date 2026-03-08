title: "مرجع إعدادات بوابة OpenClaw ودليل الإعدادات"
description: "مرجع شامل لجميع حقول إعدادات بوابة OpenClaw. تعلم كيفية إعداد القنوات، سياسات الرسائل المباشرة، تجاوزات النماذج، وتكوين واتساب، تيليجرام، وديسكورد."
keywords: ["إعدادات openclaw", "إعداد البوابة", "إعداد القناة", "سياسة الرسائل المباشرة", "تجاوزات النموذج", "بوت واتساب", "بوت تيليجرام", "بوت ديسكورد"]
---

  الإعدادات والعمليات

  
# مرجع الإعدادات

كل حقل متاح في `~/.openclaw/openclaw.json`. للحصول على نظرة موجهة للمهام، راجع [الإعدادات](./configuration.md). تنسيق الإعداد هو **JSON5** (يسمح بالتعليقات والفاصلة الزائدة). جميع الحقول اختيارية — يستخدم OpenClaw قيمًا افتراضية آمنة عند حذفها.

* * *

## القنوات

تبدأ كل قناة تلقائيًا عندما يكون قسم إعداداتها موجودًا (ما لم يكن `enabled: false`).

### الوصول للرسائل المباشرة والمجموعات

تدعم جميع القنوات سياسات الرسائل المباشرة وسياسات المجموعات:

| سياسة الرسائل المباشرة | السلوك |
| --- | --- |
| `pairing` (الافتراضي) | المرسلون غير المعروفين يحصلون على رمز اقتران لمرة واحدة؛ يجب أن يوافق المالك |
| `allowlist` | فقط المرسلون الموجودون في `allowFrom` (أو مخزن السماح المقترن) |
| `open` | السماح بجميع الرسائل المباشرة الواردة (يتطلب `allowFrom: ["*"]`) |
| `disabled` | تجاهل جميع الرسائل المباشرة الواردة |

| سياسة المجموعة | السلوك |
| --- | --- |
| `allowlist` (الافتراضي) | فقط المجموعات المطابقة لقائمة السماح المُعدة |
| `open` | تجاوز قوائم السماح للمجموعات (لا يزال شرط الذكر ينطبق) |
| `disabled` | حظر جميع رسائل المجموعة/الغرفة |

> **ℹ️** `channels.defaults.groupPolicy` يحدد القيمة الافتراضية عندما تكون `groupPolicy` لمزود الخدمة غير محددة. تنتهي صلاحية رموز الاقتران بعد ساعة واحدة. طلبات اقتران الرسائل المباشرة المعلقة محدودة بـ **3 لكل قناة**. إذا كان كتلة المزود مفقودة تمامًا (`channels.` غير موجودة)، تعود سياسة المجموعة في وقت التشغيل إلى `allowlist` (فشل مغلق) مع تحذير عند بدء التشغيل.

### تجاوزات نموذج القناة

استخدم `channels.modelByChannel` لتثبيت معرفات قنوات محددة على نموذج. تقبل القيم `provider/model` أو أسماء النماذج المُعدة. يتم تطبيق تعيين القناة عندما لا يكون للجلسة تجاوز نموذج مسبقًا (على سبيل المثال، تم تعيينه عبر `/model`).

```json
{
  channels: {
    modelByChannel: {
      discord: {
        "123456789012345678": "anthropic/claude-opus-4-6",
      },
      slack: {
        C1234567890: "openai/gpt-4.1",
      },
      telegram: {
        "-1001234567890": "openai/gpt-4.1-mini",
        "-1001234567890:topic:99": "anthropic/claude-sonnet-4-6",
      },
    },
  },
}
```

### الإعدادات الافتراضية للقنوات ونبضات الحياة

استخدم `channels.defaults` لسياسة المجموعة المشتركة وسلوك نبضات الحياة عبر المزودين:

```json
{
  channels: {
    defaults: {
      groupPolicy: "allowlist", // open | allowlist | disabled
      heartbeat: {
        showOk: false,
        showAlerts: true,
        useIndicator: true,
      },
    },
  },
}
```

-   `channels.defaults.groupPolicy`: سياسة المجموعة الاحتياطية عندما تكون `groupPolicy` على مستوى المزود غير محددة.
-   `channels.defaults.heartbeat.showOk`: تضمين حالات القناة السليمة في إخراج نبضات الحياة.
-   `channels.defaults.heartbeat.showAlerts`: تضمين الحالات المتدهورة/الخطأ في إخراج نبضات الحياة.
-   `channels.defaults.heartbeat.useIndicator`: عرض إخراج نبضات الحياة المضغوط على شكل مؤشر.

### واتساب

يعمل واتساب عبر قناة الويب للبوابة (Baileys Web). يبدأ تلقائيًا عندما تكون هناك جلسة مرتبطة موجودة.

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // علامات القراءة الزرقاء (false في وضع الدردشة الذاتية)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

```json
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

-   الأوامر الصادرة الافتراضية تستخدم الحساب `default` إذا كان موجودًا؛ وإلا أول حساب مُعد معرفه (مرتب).
-   `channels.whatsapp.defaultAccount` الاختياري يتجاوز اختيار الحساب الافتراضي الاحتياطي عندما يتطابق مع معرف حساب مُعد.
-   يتم ترحيل دليل مصادقة Baileys لحساب واحد قديم بواسطة `openclaw doctor` إلى `whatsapp/default`.
-   تجاوزات لكل حساب: `channels.whatsapp.accounts..sendReadReceipts`, `channels.whatsapp.accounts..dmPolicy`, `channels.whatsapp.accounts..allowFrom`.

### تيليجرام

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "اجعل الإجابات مختصرة.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "التزم بالموضوع.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "نسخ احتياطي Git" },
        { command: "generate", description: "إنشاء صورة" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streaming: "partial", // off | partial | block | progress (الافتراضي: off)
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 100,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        autoSelectFamily: true,
        dnsResultOrder: "ipv4first",
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

-   رمز البوت: `channels.telegram.botToken` أو `channels.telegram.tokenFile`، مع `TELEGRAM_BOT_TOKEN` كاحتياطي للحساب الافتراضي.
-   `channels.telegram.defaultAccount` الاختياري يتجاوز اختيار الحساب الافتراضي عندما يتطابق مع معرف حساب مُعد.
-   في إعدادات الحسابات المتعددة (2+ معرف حساب)، عيّن افتراضيًا صريحًا (`channels.telegram.defaultAccount` أو `channels.telegram.accounts.default`) لتجنب التوجيه الاحتياطي؛ يحذر `openclaw doctor` عندما يكون هذا مفقودًا أو غير صالح.
-   `configWrites: false` يمنع كتابات الإعدادات التي يبدأها تيليجرام (هجرات معرف المجموعة الفائقة، `/config set|unset`).
-   إدخالات `bindings[]` ذات المستوى الأعلى مع `type: "acp"` تُعد ربطات ACP دائمة لمواضيع المنتدى (استخدم `chatId:topic:topicId` الأساسي في `match.peer.id`). دلالات الحقل مشتركة في [وكلاء ACP](../tools/acp-agents.md#channel-specific-settings).
-   تستخدم معاينات البث في تيليجرام `sendMessage` + `editMessageText` (تعمل في الدردشات المباشرة والمجموعات).
-   سياسة إعادة المحاولة: راجع [سياسة إعادة المحاولة](../concepts/retry.md).

### ديسكورد

```json
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "123456789012345678"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          ignoreOtherMentions: true,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "إجابات قصيرة فقط.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      streaming: "off", // off | partial | block | progress (progress تُرسم إلى partial على ديسكورد)
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // اختيار يدوي لـ sessions_spawn({ thread: true })
      },
      voice: {
        enabled: true,
        autoJoin: [
          {
            guildId: "123456789012345678",
            channelId: "234567890123456789",
          },
        ],
        daveEncryption: true,
        decryptionFailureTolerance: 24,
        tts: {
          provider: "openai",
          openai: { voice: "alloy" },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

-   الرمز: `channels.discord.token`، مع `DISCORD_BOT_TOKEN` كاحتياطي للحساب الافتراضي.
-   `channels.discord.defaultAccount` الاختياري يتجاوز اختيار الحساب الافتراضي عندما يتطابق مع معرف حساب مُعد.
-   استخدم `user:` (رسالة مباشرة) أو `channel:` (قناة النقابة) لأهداف التسليم؛ يتم رفض المعرفات الرقمية العارية.
-   سلاجات النقابة تكون بأحرف صغيرة مع استبدال المسافات بـ `-`؛ مفاتيح القناة تستخدم الاسم المُسلّغ (بدون `#`). يُفضل استخدام معرفات النقابة.
-   يتم تجاهل الرسائل التي كتبها البوت افتراضيًا. `allowBots: true` يُمكنها؛ استخدم `allowBots: "mentions"` لقبول رسائل البوت التي تذكر البوت فقط (لا تزال رسائله الخاصة مُرشحة).
-   `channels.discord.guilds..ignoreOtherMentions` (وتجاوزات القناة) يحذف الرسائل التي تذكر مستخدمًا أو دورًا آخر ولكن ليس البوت (باستثناء @everyone/@here).
-   `maxLinesPerMessage` (الافتراضي 17) يقسم الرسائل الطويلة حتى عندما تكون أقل من 2000 حرف.
-   `channels.discord.threadBindings` يتحكم في توجيه الجلسات المقيدة بالخيوط في ديسكورد:
    -   `enabled`: تجاوز ديسكورد لميزات الجلسة المقيدة بالخيط (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`، والتسليم/التوجيه المقيد)
    -   `idleHours`: تجاوز ديسكورد للإلغاء التلقائي للتركيز بسبب الخمول بالساعات (`0` يعطله)
    -   `maxAgeHours`: تجاوز ديسكورد للعمر الأقصى الصارم بالساعات (`0` يعطله)
    -   `spawnSubagentSessions`: مفتاح اختيار يدوي لـ `sessions_spawn({ thread: true })` إنشاء/ربط خيط تلقائي
-   إدخالات `bindings[]` ذات المستوى الأعلى مع `type: "acp"` تُعد ربطات ACP دائمة للقنوات والخيوط (استخدم معرف القناة/الخيط في `match.peer.id`). دلالات الحقل مشتركة في [وكلاء ACP](../tools/acp-agents.md#channel-specific-settings).
-   `channels.discord.ui.components.accentColor` يحدد لون التمييز لحاويات مكونات ديسكورد الإصدار 2.
-   `channels.discord.voice` يُمكن محادثات قناة الصوت في ديسكورد وتجاوزات الانضمام التلقائي + TTS الاختيارية.
-   `channels.discord.voice.daveEncryption` و `channels.discord.voice.decryptionFailureTolerance` يتم تمريرهما إلى خيارات DAVE لـ `@discordjs/voice` (`true` و `24` افتراضيًا).
-   يحاول OpenClaw أيضًا استعادة استقبال الصوت عن طريق مغادرة/إعادة الانضمام إلى جلسة صوت بعد فشل فك التشفير المتكرر.
-   `channels.discord.streaming` هو مفتاح وضع البث الأساسي. يتم ترحيل قيم `streamMode` القديمة والقيم المنطقية `streaming` تلقائيًا.
-   `channels.discord.autoPresence` يُرسم توفر وقت التشغيل إلى حالة البوت (سليم => متصل، متدهور => خامل، منهك => لا تزعج) ويسمح بتجاوزات نص الحالة الاختيارية.
-   `channels.discord.dangerouslyAllowNameMatching` يُعيد تمكين مطابقة الاسم/العلامة القابلة للتغيير (وضع التوافق لكسر الزجاج).

**أوضاع إشعارات التفاعل:** `off` (لا شيء)، `own` (رسائل البوت، الافتراضي)، `all` (جميع الرسائل)، `allowlist` (من `guilds..users` على جميع الرسائل).

### دردشة جوجل

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

-   JSON حساب الخدمة: مضمن (`serviceAccount`) أو قائم على الملف (`serviceAccountFile`).
-   أيضًا مدعوم SecretRef لحساب الخدمة (`serviceAccountRef`).
-   احتياطيات البيئة: `GOOGLE_CHAT_SERVICE_ACCOUNT` أو `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
-   استخدم `spaces/` أو `users/` لأهداف التسليم.
-   `channels.googlechat.dangerouslyAllowNameMatching` يُعيد تمكين مطابقة المبدأ الأساسي للبريد الإلكتروني القابلة للتغيير (وضع التوافق لكسر الزجاج).

### سلاك

```json
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "إجابات قصيرة فقط.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      typingReaction: "hourglass_flowing_sand",
      textChunkLimit: 4000,
      chunkMode: "length",
      streaming: "partial", // off | partial | block | progress (وضع المعاينة)
      nativeStreaming: true, // استخدم واجهة برمجة تطبيقات البث الأصلية لسلاك عندما يكون streaming=partial
      mediaMaxMb: 20,
    },
  },
}
```

-   **وضع المقبس** يتطلب كلًا من `botToken` و `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` لاحتياطي بيئة الحساب الافتراضي).
-   **وضع HTTP** يتطلب `botToken` بالإضافة إلى `signingSecret` (في الجذر أو لكل حساب).
-   `configWrites: false` يمنع كتابات الإعدادات التي يبدأها سلاك.
-   `channels.slack.defaultAccount` الاختياري يتجاوز اختيار الحساب الافتراضي عندما يتطابق مع معرف حساب مُعد.
-   `channels.slack.streaming` هو مفتاح وضع البث الأساسي. يتم ترحيل قيم `streamMode` القديمة والقيم المنطقية `streaming` تلقائيًا.
-   استخدم `user:` (رسالة مباشرة) أو `channel:` لأهداف التسليم.

**أوضاع إشعارات التفاعل:** `off`, `own` (الافتراضي), `all`, `allowlist` (من `reactionAllowlist`). **عزل جلسة الخيط:** `thread.historyScope` لكل خيط (الافتراضي) أو مشترك عبر القناة. `thread.inheritParent` ينسخ نصوص قناة الأصل إلى الخيوط الجديدة.

-   `typingReaction` يضيف تفاعلًا مؤقتًا إلى رسالة سلاك الواردة أثناء تشغيل الرد، ثم يزيله عند الانتهاء. استخدم رمز إيموجي قصير لسلاك مثل `"hourglass_flowing_sand"`.

| مجموعة الإجراء | الافتراضي | ملاحظات |
| --- | --- | --- |
| reactions | مفعل | تفاعل + قائمة التفاعلات |
| messages | مفعل | قراءة/إرسال/تحرير/حذف |
| pins | مفعل | تثبيت/إلغاء تثبيت/قائمة |
| memberInfo | مفعل | معلومات العضو |
| emojiList | مفعل | قائمة الإيموجي المخصص |

### ماترموست

يُشحن ماترموست كإضافة: `openclaw plugins install @openclaw/mattermost`.

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      commands: {
        native: true, // اختيار يدوي
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // عنوان URL صريح اختياري للنشرات العكسية/العامة
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

أوضاع الدردشة: `oncall` (الرد عند ذكر @، الافتراضي)، `onmessage` (كل رسالة)، `onchar` (الرسائل التي تبدأ ببادئة المشغل). عند تمكين الأوامر الأصلية لماترموست:

-   يجب أن يكون `commands.callbackPath` مسارًا (على سبيل المثال `/api/channels/mattermost/command`)، وليس عنوان URL كاملًا.
-   يجب أن يحل `commands.callbackUrl` إلى نقطة نهاية بوابة OpenClaw وأن يكون قابلًا للوصول من خادم ماترموست.
-   لاستدعاء المضيفين الخاصة/الشبكة الذيلية/الداخلية، قد يتطلب ماترموست تضمين `ServiceSettings.AllowedUntrustedInternalConnections` للمضيف/النطاق المستدعي. استخدم قيم المضيف/النطاق، وليس عناوين URL كاملة.
-   `channels.mattermost.configWrites`: السماح أو رفض كتابات الإعدادات التي يبدأها ماترموست.
-   `channels.mattermost.requireMention`: يتطلب `@mention` قبل الرد في القنوات.
-   `channels.mattermost.defaultAccount` الاختياري يتجاوز اختيار الحساب الافتراضي عندما يتطابق مع معرف حساب مُعد.

### سيجنال

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15555550123", // ربط حساب اختياري
      dmPolicy: "pairing",
      allowFrom: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      configWrites: true,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**أوضاع إشعارات التفاعل:** `off`, `own` (الافتراضي), `all`, `allowlist` (من `reactionAllowlist`).

-   `channels.signal.account`: يثبت بدء تشغيل القناة على هوية حساب سيجنال محددة.
-   `channels.signal.configWrites`: السماح أو رفض كتابات الإعدادات التي يبدأها سيجنال.
-   `channels.signal.defaultAccount` الاختياري يتجاوز اختيار الحساب الافتراضي عندما يتطابق مع معرف حساب مُعد.

### بلو بابلز

بلو بابلز هو المسار الموصى به لـ iMessage (مدعوم بإضافة، مُعد تحت `channels.bluebubbles`).

```json
{
  channels: {
    bluebubbles: {
      enabled: true,
      dmPolicy: "pairing",
      // serverUrl, password, webhookPath, ضوابط المجموعة، والإجراءات المتقدمة:
      // راجع /channels/bluebubbles
    },
  },
}
```

-   مسارات المفاتيح الأساسية المغطاة هنا: `channels.bluebubbles`, `channels.bluebubbles.dmPolicy`.
-   `channels.bluebubbles.defaultAccount` الاختياري يتجاوز اختيار الحساب الافتراضي عندما يتطابق مع معرف حساب مُعد.
-   إعداد قناة بلو بابلز الكامل موثق في [بلو بابلز](../channels/bluebubbles.md).

### iMessage

ينشئ OpenClaw `imsg rpc` (JSON-RPC عبر stdio). لا يتطلب برنامجًا خفيًا أو منفذًا.

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      attachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      remoteAttachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

-   `channels.imessage.defaultAccount` الاختياري يتجاوز اختيار الحساب الافتراضي عندما يتطابق مع معرف حساب مُعد.
-   يتطلب وصول القرص الكامل إلى قاعدة بيانات الرسائل.
-   يُفضل استخدام `chat_id:` كأهداف. استخدم `imsg chats --limit 20` لسرد الدردشات.
-   يمكن أن يشير `cliPath` إلى غلاف SSH؛ عيّن `remoteHost` (`host` أو `user@host`) لجلب المرفقات عبر SCP.
-   `attachmentRoots` و `remoteAttachmentRoots` تقيد مسارات المرفقات الواردة (الافتراضي: `/Users/*/Library/Messages/Attachments`).
-   يستخدم SCP فحص مفتاح المضيف الصارم، لذا تأكد من وجود مفتاح مضيف النقل بالفعل في `~/.ssh/known_hosts`.
-   `channels.imessage.configWrites`: السماح أو رفض كتابات الإعدادات التي يبدأها iMessage.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### مايكروسوفت تيمز

مايكروسوفت تيمز مدعوم بالإضافة ومُعد تحت `channels.msteams`.

```json
{
  channels: {
    msteams: {
      enabled: true,
      configWrites: true,
      // appId, appPassword, tenantId, webhook, سياسات الفريق/القناة:
      // راجع /channels/msteams
    },
  },
}
```

-   مسارات المفاتيح الأساسية المغطاة هنا: `channels.msteams`, `channels.msteams.configWrites`.
-   إعداد تيمز الكامل (بيانات الاعتماد، webhook، سياسة الرسائل المباشرة/المجموعة، تجاوزات لكل فريق/قناة) موثق في [مايكروسوفت تيمز](../channels/msteams.md).

### IRC

IRC مدعوم بالإضافة ومُعد تحت `channels.irc`.

```json
{
  channels: {
    irc: {
      enabled: true,
      dmPolicy: "pairing",
      configWrites: true,
      nickserv: {
        enabled: true,
        service: "NickServ",
        password: "${IRC_NICKSERV_PASSWORD}",
        register: false,
        registerEmail: "bot@example.com",
      },
    },
  },
}
```

-   مسارات المفاتيح الأساسية المغطاة هنا: `channels.irc`, `channels.irc.dmPolicy`, `channels.irc.configWrites`, `channels.irc.nickserv.*`.
-   `channels.irc.defaultAccount` الاختياري يتجاوز اختيار الحساب الافتراضي عندما يتطابق مع معرف حساب مُعد.
-   إعداد قناة IRC الكامل (المضيف/المنفذ/TLS/القنوات/قوائم السماح/بوابة الذكر) موثق في [IRC](../channels/irc.md).

### حسابات متعددة (جميع القنوات)

تشغيل حسابات متعددة لكل قناة (كل منها له `accountId` الخاص به):

```json
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "البوت الأساسي",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "بوت التنبيهات",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

-   يُستخدم `default` عندما يتم حذف `accountId` (CLI + التوجيه).
-   تنطبق رموز البيئة فقط على الحساب **الافتراضي**.
-   تنطبق إعدادات القناة الأساسية على جميع الحسابات ما لم يتم تجاوزها لكل حساب.
-   استخدم `bindings[].match.accountId` لتوجيه كل حساب إلى وكيل مختلف.
-   إذا أضفت حسابًا غير افتراضي عبر `openclaw channels add` (أو إعداد القناة) بينما لا تزال على إعداد قناة ذات حساب واحد على المستوى الأعلى، ينقل OpenClaw قيم الحساب الواحد على المستوى الأعلى ذات النطاق الحسابي إلى `channels..accounts.default` أولاً حتى يستمر الحساب الأصلي في العمل.
-   تظل الربطات الحالية للقناة فقط (بدون `accountId`) مطابقة للحساب الافتراضي؛ تظل الربطات ذات النطاق الحسابي اختيارية.
-   يقوم `openclaw doctor --fix` أيضًا بإصلاح الأشكال المختلطة عن طريق نقل قيم الحساب الواحد على المستوى الأعلى ذات النطاق الحسابي إلى `accounts.default` عندما تكون الحسابات المسماة موجودة ولكن `default` مفقود.

### قنوات الإضافة الأخرى

يتم تكوين العديد من قنوات الإضافة كـ `channels.` وتوثيقها في صفحات القناة المخصصة (على سبيل المثال Feishu، Matrix، LINE، Nostr، Zalo، Nextcloud Talk، Synology Chat، و Twitch). راجع فهرس القنوات الكامل: [القنوات](../channels.md).

### بوابة ذكر محادثة المجموعة

الرسائل الجماعية الافتراضية **تتطلب ذكر** (ذكر في البيانات الوصفية أو أنماط regex). تنطبق على دردشات المجموعة في واتساب، تيليجرام، ديسكورد، دردشة جوجل، و iMessage. **أنواع الذكر:**

-   **الذكر في البيانات الوصفية**: ذكر @ الأصلي للمنصة. يتم تجاهله في وضع الدردشة الذاتية لواتساب.
-   **أنماط النص**: أنماط regex في `agents.list[].groupChat.mentionPatterns`. يتم التحقق منها دائمًا.
-   يتم فرض بوابة الذكر فقط عندما يكون الكشف ممكنًا (الذكر الأصلي أو نمط واحد على الأقل).

```json
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

يحدد `messages.groupChat.historyLimit` الافتراضي العام. يمكن للقنوات التجاوز باستخدام `channels..historyLimit` (أو لكل حساب). عيّن `0` لتعطيل.

#### حدود سجل الرسائل المباشرة

```json
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789