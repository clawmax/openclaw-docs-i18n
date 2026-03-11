

  منصات المراسلة

  
# تيليجرام

الحالة: جاهز للإنتاج للرسائل الخاصة للبوت + المجموعات عبر grammY. الاقتراع الطويل هو الوضع الافتراضي؛ وضع webhook اختياري.

## الإعداد السريع

### الخطوة 1: إنشاء الرمز المميز للبوت في BotFather

افتح تيليجرام وتحدث مع **@BotFather** (تأكد من أن المعرف هو بالضبط `@BotFather`).شغل `/newbot`، اتبع التعليمات، واحفظ الرمز المميز.

### الخطوة 2: تكوين الرمز المميز وسياسة الرسائل الخاصة

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

البديل البيئي: `TELEGRAM_BOT_TOKEN=...` (الحساب الافتراضي فقط). تيليجرام **لا** يستخدم `openclaw channels login telegram`؛ قم بتكوين الرمز المميز في config/env، ثم ابدأ البوابة.

### الخطوة 3: ابدأ البوابة ووافق على أول رسالة خاصة

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

رموز الاقتران تنتهي صلاحيتها بعد ساعة واحدة.

### الخطوة 4: أضف البوت إلى مجموعة

أضف البوت إلى مجموعتك، ثم عيّن `channels.telegram.groups` و `groupPolicy` لتتوافق مع نموذج الوصول الخاص بك.

 

> **ℹ️** ترتيب حل الرمز المميز يراعي الحساب. عمليًا، قيم التكوين تفوق البديل البيئي، و `TELEGRAM_BOT_TOKEN` ينطبق فقط على الحساب الافتراضي.

## إعدادات جانب تيليجرام

بوتات تيليجرام تفترض بشكل افتراضي **وضع الخصوصية**، مما يحد من رسائل المجموعة التي تتلقاها.إذا كان يجب على البوت رؤية جميع رسائل المجموعة، إما:

-   قم بتعطيل وضع الخصوصية عبر `/setprivacy`، أو
-   اجعل البوت مشرفًا على المجموعة.

عند تبديل وضع الخصوصية، أزل وأعد إضافة البوت في كل مجموعة حتى يطبق تيليجرام التغيير.

حالة المشرف يتم التحكم بها في إعدادات مجموعة تيليجرام.البوتات المشرفة تتلقى جميع رسائل المجموعة، وهو أمر مفيد لسلوك المجموعة الدائم.

-   `/setjoingroups` للسماح/رفض الإضافة إلى المجموعات
-   `/setprivacy` لسلوك مدى الرؤية في المجموعات

## التحكم في الوصول والتفعيل

/getUpdates"', lang: 'bash' }, { label: 'سياسة المجموعة وقوائم السماح', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "-1001234567890": {\n          groupPolicy: "open",\n          requireMention: false,\n        },\n      },\n    },\n  },\n}', lang: 'json' }, { label: 'سلوك الإشارة', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "*": { requireMention: false },\n      },\n    },\n  },\n}', lang: 'json' }]} />

## سلوك التشغيل

-   تيليجرام مملوك من قبل عملية البوابة.
-   التوجيه حتمي: الردود الواردة من تيليجرام تعود إلى تيليجرام (النموذج لا يختار القنوات).
-   الرسائل الواردة تُطبع إلى ظرف القناة المشترك مع بيانات الرد والعناصر النائبة للوسائط.
-   جلسات المجموعات معزولة حسب معرف المجموعة. مواضيع المنتدى تضيف `:topic:` للحفاظ على عزل المواضيع.
-   رسائل الرسائل الخاصة يمكن أن تحمل `message_thread_id`؛ OpenClaw يوجهها بمفاتيح جلسة تراعي الخيط ويحفظ معرف الخيط للردود.
-   الاقتراع الطويل يستخدم مشغل grammY مع تسلسل لكل محادثة/خيط. إجمالي تزامن مصرف المشغل يستخدم `agents.defaults.maxConcurrent`.
-   واجهة برمجة تطبيقات بوت تيليجرام لا تدعم إيصالات القراءة (`sendReadReceipts` لا ينطبق).

## مرجع الميزات

يمكن لـ OpenClaw بث الردود الجزئية في الوقت الفعلي:

-   المحادثات المباشرة: بث المسودات الأصلية لتيليجرام عبر `sendMessageDraft`
-   المجموعات/المواضيع: رسالة معاينة + `editMessageText`

المتطلب:

-   `channels.telegram.streaming` هي `off | partial | block | progress` (الافتراضي: `partial`)
-   `progress` تُعيّن إلى `partial` على تيليجرام (توافق مع تسمية عبر القنوات)
-   القيم القديمة `channels.telegram.streamMode` والقيم المنطقية `streaming` تُعيّن تلقائيًا

تيليجرام مكّن `sendMessageDraft` لجميع البوتات في Bot API 9.5 (1 مارس 2026).للردود النصية فقط:

-   الرسائل الخاصة: OpenClaw يحدّث المسودة في مكانها (لا توجد رسالة معاينة إضافية)
-   المجموعة/الموضوع: OpenClaw يحتفظ بنفس رسالة المعاينة ويقوم بتعديل نهائي في مكانها (لا توجد رسالة ثانية)

للردود المعقدة (على سبيل المثال حمولات الوسائط)، OpenClaw يتراجع إلى التسليم النهائي العادي ثم ينظف رسالة المعاينة.بث المعاينة منفصل عن بث الكتل. عندما يتم تمكين بث الكتل صراحةً لتيليجرام، يتخطى OpenClaw بث المعاينة لتجنب البث المزدوج.إذا كان نقل المسودة الأصلية غير متاح/مرفوض، يتراجع OpenClaw تلقائيًا إلى `sendMessage` + `editMessageText`.تيليجرام فقط تيار التفكير:

-   `/reasoning stream` يرسل التفكير إلى معاينة البث المباشر أثناء التوليد
-   الإجابة النهائية تُرسل بدون نص التفكير

النص الصادر يستخدم تيليجرام `parse_mode: "HTML"`.

-   النص الشبيه بـ Markdown يُصاغ إلى HTML آمن لتيليجرام.
-   HTML النموذج الخام يُهرب لتقليل فشل تحليل تيليجرام.
-   إذا رفض تيليجرام HTML المحلل، يعيد OpenClaw المحاولة كنص عادي.

معاينات الروابط مفعلة افتراضيًا ويمكن تعطيلها بـ `channels.telegram.linkPreview: false`.

تسجيل قائمة أوامر تيليجرام يتم التعامل معه عند بدء التشغيل بـ `setMyCommands`.الأوامر الأصلية الافتراضية:

-   `commands.native: "auto"` يُفعّل الأوامر الأصلية لتيليجرام

أضف إدخالات قائمة أوامر مخصصة:

```json
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "نسخ احتياطي Git" },
        { command: "generate", description: "إنشاء صورة" },
      ],
    },
  },
}
```

القواعد:

-   الأسماء تُطبّع (إزالة `/` البادئة، أحرف صغيرة)
-   النمط الصالح: `a-z`, `0-9`, `_`, الطول `1..32`
-   الأوامر المخصصة لا يمكنها تجاوز الأوامر الأصلية
-   التعارضات/التكرارات يتم تخطيها وتسجيلها

ملاحظات:

-   الأوامر المخصصة هي إدخالات قائمة فقط؛ لا تنفذ السلوك تلقائيًا
-   أوامر الإضافات/المهارات يمكن أن تعمل عند كتابتها حتى لو لم تظهر في قائمة تيليجرام

إذا تم تعطيل الأوامر الأصلية، تُزال الأوامر المدمجة. الأوامر المخصصة/الإضافية قد تسجل إذا تم تكوينها.فشل الإعداد الشائع:

-   `setMyCommands failed` يعني عادةً أن DNS/HTTPS الصادر إلى `api.telegram.org` محظور.

### أوامر اقتران الجهاز (إضافة device-pair)

عند تثبيت إضافة `device-pair`:

1.  `/pair` يولد رمز إعداد
2.  الصق الرمز في تطبيق iOS
3.  `/pair approve` يوافق على أحدث طلب معلق

المزيد من التفاصيل: [الاقتران](./pairing.md#pair-via-telegram-recommended-for-ios).

تكوين نطاق لوحة المفاتيح المضمنة:

```json
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

تجاوز لكل حساب:

```json
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

النطاقات:

-   `off`
-   `dm`
-   `group`
-   `all`
-   `allowlist` (الافتراضي)

التراث `capabilities: ["inlineButtons"]` يُعيّن إلى `inlineButtons: "all"`.مثال على إجراء الرسالة:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "اختر خيارًا:",
  buttons: [
    [
      { text: "نعم", callback_data: "yes" },
      { text: "لا", callback_data: "no" },
    ],
    [{ text: "إلغاء", callback_data: "cancel" }],
  ],
}
```

نقرات الرد تُمرّر إلى الوكيل كنص: `callback_data: `

إجراءات أداة تيليجرام تتضمن:

-   `sendMessage` (`to`, `content`, اختياري `mediaUrl`, `replyToMessageId`, `messageThreadId`)
-   `react` (`chatId`, `messageId`, `emoji`)
-   `deleteMessage` (`chatId`, `messageId`)
-   `editMessage` (`chatId`, `messageId`, `content`)
-   `createForumTopic` (`chatId`, `name`, اختياري `iconColor`, `iconCustomEmojiId`)

إجراءات رسالة القناة تعرض أسماء مستعارة مريحة (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`, `topic-create`).ضوابط البوابة:

-   `channels.telegram.actions.sendMessage`
-   `channels.telegram.actions.deleteMessage`
-   `channels.telegram.actions.reactions`
-   `channels.telegram.actions.sticker` (الافتراضي: معطل)

ملاحظة: `edit` و `topic-create` مفعلان افتراضيًا حاليًا وليس لهما مفاتيح تبديل منفصلة `channels.telegram.actions.*`.دلالات إزالة التفاعل: [/tools/reactions](../tools/reactions.md)

تيليجرام يدعم وسوم خيط الرد الصريحة في المخرجات المُنشأة:

-   `[[reply_to_current]]` يرد على الرسالة المشغلة
-   `[[reply_to:]]` يرد على معرف رسالة تيليجرام محدد

`channels.telegram.replyToMode` يتحكم في المعالجة:

-   `off` (الافتراضي)
-   `first`
-   `all`

ملاحظة: `off` يعطل خيط الرد الضمني. وسوم `[[reply_to_*]]` الصريحة لا تزال تُحترم.

مجموعات المنتدى الفائقة:

-   مفاتيح جلسة الموضوع تضيف `:topic:`
-   الردود والكتابة تستهدف خيط الموضوع
-   مسار تكوين الموضوع: `channels.telegram.groups..topics.`

حالة خاصة للموضوع العام (`threadId=1`):

-   إرسال الرسائل يحذف `message_thread_id` (تيليجرام يرفض `sendMessage(...thread_id=1)`)
-   إجراءات الكتابة لا تزال تتضمن `message_thread_id`

توارث الموضوع: إدخالات الموضوع ترث إعدادات المجموعة ما لم يتم تجاوزها (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`). `agentId` خاص بالموضوع فقط ولا يرث من إعدادات المجموعة الافتراضية.**توجيه الوكيل لكل موضوع**: يمكن لكل موضوع توجيه إلى وكيل مختلف عن طريق تعيين `agentId` في تكوين الموضوع. هذا يعطي لكل موضوع مساحة عمل وذاكرة وجلسة معزولة خاصة به. مثال:

```json
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "1": { agentId: "main" },      // الموضوع العام → الوكيل الرئيسي
            "3": { agentId: "zu" },        // موضوع التطوير → وكيل zu
            "5": { agentId: "coder" }      // مراجعة الكود → وكيل coder
          }
        }
      }
    }
  }
}
```

كل موضوع له بعد ذلك مفتاح جلسة خاص به: `agent:zu:telegram:group:-1001234567890:topic:3`**ربط موضوع ACP الدائم**: يمكن لمواضيع المنتدى تثبيت جلسات ACP عبر روابط ACP مكتوبة على المستوى الأعلى:

-   `bindings[]` مع `type: "acp"` و `match.channel: "telegram"`

مثال:

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
    },
  ],
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "42": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

هذا محدد حاليًا لمواضيع المنتدى في المجموعات والمجموعات الفائقة.**إنشاء ACP مرتبط بالخيط من المحادثة**:

-   `/acp spawn  --thread here|auto` يمكنه ربط موضوع تيليجرام الحالي بجلسة ACP جديدة.
-   رسائل الموضوع اللاحقة تُوجّه مباشرة إلى جلسة ACP المرتبطة (لا حاجة لـ `/acp steer`).
-   OpenClaw يثبت رسالة تأكيد الإنشاء داخل الموضوع بعد ربط ناجح.
-   يتطلب `channels.telegram.threadBindings.spawnAcpSessions=true`.

سياق القالب يتضمن:

-   `MessageThreadId`
-   `IsForum`

سلوك خيط الرسائل الخاصة:

-   المحادثات الخاصة مع `message_thread_id` تحتفظ بتوجيه الرسائل الخاصة ولكن تستخدم مفاتيح جلسة/أهداف رد تراعي الخيط.

### رسائل الصوت

تيليجرام يميز بين ملاحظات الصوت وملفات الصوت.

-   الافتراضي: سلوك ملف الصوت
-   الوسم `[[audio_as_voice]]` في رد الوكيل لإجبار إرسال ملاحظة صوتية

مثال على إجراء الرسالة:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

### رسائل الفيديو

تيليجرام يميز بين ملفات الفيديو وملاحظات الفيديو.مثال على إجراء الرسالة:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

ملاحظات الفيديو لا تدعم التسميات التوضيحية؛ نص الرسالة المقدم يُرسل بشكل منفصل.

### الملصقات

معالجة الملصقات الواردة:

-   WEBP ثابت: يتم تنزيله ومعالجته (عنصر نائب `<media:sticker>`)
-   TGS متحرك: يتم تخطيه
-   WEBM فيديو: يتم تخطيه

حقول سياق الملصق:

-   `Sticker.emoji`
-   `Sticker.setName`
-   `Sticker.fileId`
-   `Sticker.fileUniqueId`
-   `Sticker.cachedDescription`

ملف ذاكرة التخزين المؤقت للملصقات:

-   `~/.openclaw/telegram/sticker-cache.json`

يتم وصف الملصقات مرة واحدة (عند الإمكان) وتخزينها مؤقتًا لتقليل استدعاءات الرؤية المتكررة.تفعيل إجراءات الملصقات:

```json
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

إرسال إجراء الملصق:

```json
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

البحث في الملصقات المخزنة مؤقتًا:

```json
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

تفاعلات تيليجرام تصل كتحديثات `message_reaction` (منفصلة عن حمولة الرسالة).عند التمكين، OpenClaw يضيف أحداث نظام مثل:

-   `تمت إضافة تفاعل تيليجرام: 👍 بواسطة Alice (@alice) على الرسالة 42`

التكوين:

-   `channels.telegram.reactionNotifications`: `off | own | all` (الافتراضي: `own`)
-   `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (الافتراضي: `minimal`)

ملاحظات:

-   `own` تعني تفاعلات المستخدم على الرسائل المرسلة من البوت فقط (أفضل جهد عبر ذاكرة التخزين المؤقت للرسائل المرسلة).
-   أحداث التفاعل لا تزال تحترم ضوابط وصول تيليجرام (`dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`); المرسلون غير المصرح لهم يتم تجاهلهم.
-   تيليجرام لا يوفر معرفات الخيط في تحديثات التفاعل.
    -   المجموعات غير المنتدى تُوجّه إلى جلسة محادثة المجموعة
    -   مجموعات المنتدى تُوجّه إلى جلسة موضوع المجموعة العام (`:topic:1`)، وليس الموضوع الأصلي الدقيق

`allowed_updates` للاقتراع/webhook تتضمن `message_reaction` تلقائيًا.

`ackReaction` يرسل رمز تعبيري تأكيد بينما OpenClaw يعالج رسالة واردة.ترتيب الحل:

-   `channels.telegram.accounts..ackReaction`
-   `channels.telegram.ackReaction`
-   `messages.ackReaction`
-   رمز تعبيري هوية الوكيل الاحتياطي (`agents.list[].identity.emoji`, وإلا ”👀”)

ملاحظات:

-   تيليجرام يتوقع رمز تعبيري يونيكود (على سبيل المثال ”👀”).
-   استخدم `""` لتعطيل التفاعل لقناة أو حساب.

كتابات تكوين القناة مفعلة افتراضيًا (`configWrites !== false`).الكتابات التي يتم تشغيلها بواسطة تيليجرام تتضمن:

-   أحداث ترحيل المجموعة (`migrate_to_chat_id`) لتحديث `channels.telegram.groups`
-   `/config set` و `/config unset` (يتطلب تمكين الأمر)

التعطيل:

```json
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

الافتراضي: الاقتراع الطويل.وضع webhook:

-   عيّن `channels.telegram.webhookUrl`
-   عيّن `channels.telegram.webhookSecret` (مطلوب عند تعيين عنوان URL لـ webhook)
-   اختياري `channels.telegram.webhookPath` (الافتراضي `/telegram-webhook`)
-   اختياري `channels.telegram.webhookHost` (الافتراضي `127.0.0.1`)
-   اختياري `channels.telegram.webhookPort` (الافتراضي `8787`)

المستمع المحلي الافتراضي لوضع webhook يرتبط بـ `127.0.0.1:8787`.إذا كان نقطة النهاية العامة لديك مختلفة، ضع وكيل عكسي في المقدمة وأشر `webhookUrl` إلى عنوان URL العام. عيّن `webhookHost` (على سبيل المثال `0.0.0.0`) عندما تحتاج عمدًا إلى دخول خارجي.

-   `channels.telegram.textChunkLimit` الافتراضي هو 4000.
-   `channels.telegram.chunkMode="newline"` يفضل حدود الفقرات (أسطر فارغة) قبل التقسيم حسب الطول.
-   `channels.telegram.mediaMaxMb` (الافتراضي 100) يحدد حجم وسائط تيليجرام الواردة والصادرة.
-   `channels.telegram.timeoutSeconds` يتجاوز مهلة عميل Telegram API (إذا لم يتم تعيينه، ينطبق الافتراضي لـ grammY).
-   تاريخ سياق المجموعة يستخدم `channels.telegram.historyLimit` أو `messages.groupChat.historyLimit` (الافتراضي 50); `0` يعطل.
-   ضوابط تاريخ الرسائل الخاصة:
    -   `channels.telegram.dmHistoryLimit`
    -   `channels.telegram.dms["<user_id>"].historyLimit`
-   `channels.telegram.retry` config ينطبق على مساعدات إرسال تيليجرام (CLI/الأدوات/الإجراءات) لأخطاء API الصادرة القابلة للاسترداد.

هدف إرسال CLI يمكن أن يكون معرف محادثة رقمي أو اسم مستخدم:

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

استطلاعات تيليجرام تستخدم `openclaw message poll` وتدعم مواضيع المنتدى:

```bash
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "Ship it?" --poll-option "Yes" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Pick a time" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300 --poll-public
```

أعلام الاستطلاع الخاصة بتيليجرام فقط:

-   `--poll-duration-seconds` (5-600)
-   `--poll-anonymous`
-   `--poll-public`
-   `--thread-id` لمواضيع المنتدى (أو استخدم هدف `:topic:`)

بوابة الإجراءات:

-   `channels.telegram.actions.sendMessage=false` يعطل رسائل تيليجرام الصادرة، بما في ذلك الاستطلاعات
-   `channels.telegram.actions.poll=false` يعطل إنشاء استطلاعات تيليجرام مع ترك الإرسال العادي مفعلًا

## استكشاف الأخطاء وإصلاحها

-   إذا كان `requireMention=false`، يجب أن يسمح وضع خصوصية تيليجرام بالرؤية الكاملة.
    -   BotFather: `/setprivacy` -> تعطيل
    -   ثم أزل + أعد إضافة البوت إلى المجموعة
-   `openclaw channels status` يحذر عندما يتوقع التكوين رسائل مجموعة غير مذكورة.
-   `openclaw channels status --probe` يمكنه التحقق من معرفات المجموعات الرقمية الصريحة؛ النمط العام `"*"` لا يمكن التحقق من عضويته.
-   اختبار جلسة سريع: `/activation always`.

-   عندما يوجد `channels.telegram.groups`، يجب أن تكون المجموعة مدرجة (أو تتضمن `"*"`)
-   تحقق من عضوية البوت في المجموعة
-   راجع السجلات: `openclaw logs --follow` لأسباب التخطي

-   صلّح هوية المرسل الخاصة بك (الاقتران و/أو `allowFrom` رقمي)
-   تفويض الأمر لا يزال ينطبق حتى عندما تكون سياسة المجموعة `open`
-   `setMyCommands failed` يشير عادةً إلى مشاكل وصول DNS/HTTPS إلى `api.telegram.org`

-   Node 22+ + fetch/وكيل مخصص يمكن أن يثير سلوك الإحباط الفوري إذا كانت أنواع AbortSignal غير متطابقة.
-   بعض المضيفين تحل `api.telegram.org` إلى IPv6 أولاً؛ خروج IPv6 معطل يمكن أن يسبب فشل متقطع لـ Telegram API.
-   إذا كانت السجلات تتضمن `TypeError: fetch failed` أو `Network request for 'getUpdates' failed!`، OpenClaw الآن يعيد محاولة هذه الأخطاء كأخطاء شبكة قابلة للاسترداد.
-   على مضيفات VPS مع خروج/TLS مباشر غير مستقر، وجّه استدعاءات Telegram API عبر `channels.telegram.proxy`:

```yaml
channels:
  telegram:
    proxy: socks5://<user>:<password>@proxy-host:1080
```

-   Node 22+ يفترض افتراضيًا `autoSelectFamily=true` (باستثناء WSL2) و `dnsResultOrder=ipv4first`.
-   إذا كان مضيفك هو WSL2 أو يعمل بشكل أفضل صراحةً مع سلوك IPv4 فقط، فرض اختيار العائلة:

```yaml
channels:
  telegram:
    network:
      autoSelectFamily: false
```

-   تجاوزات البيئة (مؤقتة):
    -   `OPENCLAW_TELEGRAM_DISABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_ENABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_DNS_RESULT_ORDER=ipv4first`
-   تحقق من إجابات DNS:

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

 المزيد من المساعدة: [استكشاف أخطاء القنوات وإصلاحها](./troubleshooting.md).

## مؤشرات مرجع تكوين تيليجرام

المرجع الأساسي:

-   `channels.telegram.enabled`: تفعيل/تعطيل بدء تشغيل القناة.
-   `channels.telegram.botToken`: الرمز المميز للبوت (BotFather).
-   `channels.telegram.tokenFile`: قراءة الرمز المميز من مسار الملف.
-   `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (الافتراضي: pairing).
-   `channels.telegram.allowFrom`: قائمة السماح للرسائل الخاصة (معرفات مستخدم تيليجرام الرقمية). `allowlist` تتطلب معرف مرسل واحد على الأقل. `open` تتطلب `"*"`. `openclaw doctor --fix` يمكنه حل إدخالات `@username` القديمة إلى معرفات ويمكنه استرداد إدخالات قائمة السماح من ملفات مخزن الاقتران في تدفقات هجرة قائمة السماح.
-   `channels.telegram.actions.poll`: تفعيل أو تعطيل إنشاء استطلاعات تيليجرام (الافتراضي: مفعل؛ لا يزال يتطلب `sendMessage`).
-   `channels.telegram.defaultTo`: هدف تيليجرام الافتراضي المستخدم بواسطة CLI `--deliver` عندما لا يتم توفير `--reply-to` صريح.
-   `channels.telegram.groupPolicy`: `open | allowlist | disabled` (الافتراضي: allowlist).
-   `channels.telegram.groupAllowFrom`: قائمة السماح لمرسلي المجموعة (معرفات مستخدم تيليجرام الرقمية). `openclaw doctor --fix` يمكنه حل إدخالات `@username` القديمة إلى معرفات. الإدخالات غير الرقمية يتم تجاهلها في وقت المصادقة. مصادقة المجموعة لا تستخدم الاحتياطي لمخزن الاقتران للرسائل الخاصة (`2026.2.25+`).
-   أولوية الحسابات المتعددة:
    -   عند تكوين معرفين حسابين أو أكثر، عيّن `channels.telegram.defaultAccount` (أو أدرج `channels.telegram.accounts.default`) لجعل التوجيه الافتراضي صريحًا.
    -   إذا لم يتم تعيين أي منهما، يتراجع OpenClaw إلى أول معرف حساب مطب