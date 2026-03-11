

  منصات المراسلة

  
# Discord

الحالة: جاهز للرسائل المباشرة وقنوات النقابة عبر بوابة Discord الرسمية.

## الإعداد السريع

ستحتاج إلى إنشاء تطبيق جديد مع بوت، إضافة البوت إلى خادمك، وإقرانه بـ OpenClaw. نوصي بإضافة البوت الخاص بك إلى خادمك الخاص. إذا لم يكن لديك واحد بعد، [قم بإنشاء واحد أولاً](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server) (اختر **Create My Own > For me and my friends**).

### الخطوة 1: إنشاء تطبيق وبوت Discord

اذهب إلى [بوابة مطوري Discord](https://discord.com/developers/applications) وانقر على **New Application**. سمّه شيئًا مثل "OpenClaw". انقر على **Bot** في الشريط الجانبي. عيّن **Username** إلى ما تسمي به وكيل OpenClaw الخاص بك.

### الخطوة 2: تمكين النوايا المميزة (Privileged Intents)

ما زلت في صفحة **Bot**، انتقل لأسفل إلى **Privileged Gateway Intents** وقم بتمكين:

-   **Message Content Intent** (مطلوب)
-   **Server Members Intent** (موصى به؛ مطلوب لقوائم السماح بالصلاحيات ومطابقة الاسم بالمعرف)
-   **Presence Intent** (اختياري؛ مطلوب فقط لتحديثات الحالة)

### الخطوة 3: نسخ رمز البوت (Bot Token) الخاص بك

ارجع لأعلى في صفحة **Bot** وانقر على **Reset Token**.

> **ℹ️** رغم الاسم، هذا يُنشئ أول رمز لك — لا يتم "إعادة تعيين" أي شيء.

انسخ الرمز واحفظه في مكان ما. هذا هو **رمز البوت (Bot Token)** الخاص بك وستحتاجه قريبًا.

### الخطوة 4: إنشاء رابط دعوة وإضافة البوت إلى خادمك

انقر على **OAuth2** في الشريط الجانبي. ستقوم بإنشاء رابط دعوة مع الأذونات الصحيحة لإضافة البوت إلى خادمك. انتقل لأسفل إلى **OAuth2 URL Generator** وقم بتمكين:

-   `bot`
-   `applications.commands`

سيظهر قسم **Bot Permissions** أدناه. قم بتمكين:

-   View Channels
-   Send Messages
-   Read Message History
-   Embed Links
-   Attach Files
-   Add Reactions (اختياري)

انسخ الرابط المُنشأ في الأسفل، الصقه في متصفحك، اختر خادمك، وانقر على **Continue** للتوصيل. يجب أن ترى الآن البوت الخاص بك في خادم Discord.

### الخطوة 5: تمكين وضع المطور وجمع معرفاتك

عد إلى تطبيق Discord، تحتاج إلى تمكين وضع المطور حتى تتمكن من نسخ المعرفات الداخلية.

1.  انقر على **User Settings** (أيقونة الترس بجوار صورتك الشخصية) → **Advanced** → شغّل **Developer Mode**
2.  انقر بزر الماوس الأيمن على **أيقونة الخادم** في الشريط الجانبي → **Copy Server ID**
3.  انقر بزر الماوس الأيمن على **صورتك الشخصية** → **Copy User ID**

احفظ **معرف الخادم (Server ID)** و **معرف المستخدم (User ID)** الخاصين بك جنبًا إلى جنب مع رمز البوت — ستُرسل الثلاثة إلى OpenClaw في الخطوة التالية.

### الخطوة 6: السماح بالرسائل المباشرة من أعضاء الخادم

لكي يعمل الإقران، يحتاج Discord إلى السماح للبوت الخاص بك بإرسال رسائل مباشرة إليك. انقر بزر الماوس الأيمن على **أيقونة الخادم** → **Privacy Settings** → شغّل **Direct Messages**. هذا يسمح لأعضاء الخادم (بما في ذلك البوتات) بإرسال رسائل مباشرة إليك. أبقِ هذا مفعلاً إذا كنت تريد استخدام الرسائل المباشرة في Discord مع OpenClaw. إذا كنت تخطط لاستخدام قنوات النقابة فقط، يمكنك تعطيل الرسائل المباشرة بعد الإقران.

### الخطوة 7: الخطوة 0: ضبط رمز البوت الخاص بك بشكل آمن (لا ترسله في الدردشة)

رمز بوت Discord الخاص بك هو سر (مثل كلمة المرور). عيّنه على الجهاز الذي يعمل عليه OpenClaw قبل مراسلة وكيلك.

```bash
openclaw config set channels.discord.token '"YOUR_BOT_TOKEN"' --json
openclaw config set channels.discord.enabled true --json
openclaw gateway
```

إذا كان OpenClaw يعمل بالفعل كخدمة خلفية، استخدم `openclaw gateway restart` بدلاً من ذلك.

### الخطوة 8: تكوين OpenClaw والإقران

تحدث مع وكيل OpenClaw الخاص بك على أي قناة موجودة (مثل Telegram) وأخبره. إذا كان Discord هو قناتك الأولى، استخدم واجهة سطر الأوامر / علامة التبويب config بدلاً من ذلك.

> “I already set my Discord bot token in config. Please finish Discord setup with User ID `<user_id>` and Server ID `<server_id>`.”

```json
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

### الخطوة 9: الموافقة على أول إقران عبر الرسائل المباشرة

انتظر حتى تعمل البوابة، ثم أرسل رسالة مباشرة إلى البوت الخاص بك في Discord. سيرد برمز إقران.

أرسل رمز الإقران إلى وكيلك على قناتك الحالية:

> “Approve this Discord pairing code: ``”

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

تنتهي صلاحية رموز الإقران بعد ساعة واحدة. يجب أن تكون الآن قادرًا على الدردشة مع وكيلك في Discord عبر الرسائل المباشرة.

 

> **ℹ️** حل الرمز المميز (Token) يراعي الحساب. قيم رمز التكوين تفوق القيمة الافتراضية للمتغير البيئي. `DISCORD_BOT_TOKEN` يُستخدم فقط للحساب الافتراضي.

## موصى به: إعداد مساحة عمل للنقابة (Guild)

بمجرد أن تعمل الرسائل المباشرة، يمكنك إعداد خادم Discord الخاص بك كمساحة عمل كاملة حيث تحصل كل قناة على جلسة وكيل خاصة بها مع سياقها الخاص. هذا موصى به للخوادم الخاصة حيث يكون فقط أنت والبوت الخاص بك.

### الخطوة 1: إضافة خادمك إلى قائمة السماح للنقابة

هذا يمكّن وكيلك من الرد في أي قناة على خادمك، وليس فقط الرسائل المباشرة.

> “Add my Discord Server ID `<server_id>` to the guild allowlist”

```json
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: true,
          users: ["YOUR_USER_ID"],
        },
      },
    },
  },
}
```

### الخطوة 2: السماح بالردود دون ذكر @

افتراضيًا، يرد وكيلك فقط في قنوات النقابة عندما يتم ذكره بـ @. بالنسبة لخادم خاص، ربما تريد منه الرد على كل رسالة.

> “Allow my agent to respond on this server without having to be @mentioned”

```json
{
  channels: {
    discord: {
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: false,
        },
      },
    },
  },
}
```

### الخطوة 3: التخطيط للذاكرة في قنوات النقابة

افتراضيًا، الذاكرة طويلة المدى (MEMORY.md) تُحمّل فقط في جلسات الرسائل المباشرة. قنوات النقابة لا تُحمّل MEMORY.md تلقائيًا.

> “When I ask questions in Discord channels, use memory\_search or memory\_get if you need long-term context from MEMORY.md.”

إذا كنت بحاجة إلى سياق مشترك في كل قناة، ضع التعليمات الثابتة في `AGENTS.md` أو `USER.md` (يتم حقنها في كل جلسة). احتفظ بالملاحظات طويلة المدى في `MEMORY.md` واستخدم أدوات الذاكرة للوصول إليها عند الطلب.

 الآن أنشئ بعض القنوات على خادم Discord الخاص بك وابدأ الدردشة. يمكن لوكيلك رؤية اسم القناة، وكل قناة تحصل على جلسة معزولة خاصة بها — لذا يمكنك إعداد `#coding`، `#home`، `#research`، أو أي ما يناسب سير عملك.

## نموذج وقت التشغيل

-   البوابة تمتلك اتصال Discord.
-   توجيه الرد حتمي: الردود الواردة من Discord تُعاد إلى Discord.
-   افتراضيًا (`session.dmScope=main`)، الدردشات المباشرة تشارك الجلسة الرئيسية للوكيل (`agent:main:main`).
-   قنوات النقابة هي مفاتيح جلسات معزولة (`agent::discord:channel:`).
-   الرسائل المباشرة الجماعية يتم تجاهلها افتراضيًا (`channels.discord.dm.groupEnabled=false`).
-   الأوامر المائلة (Slash) الأصلية تعمل في جلسات أوامر معزولة (`agent::discord:slash:`)، بينما لا تزال تحمل `CommandTargetSessionKey` إلى جلسة المحادثة الموجهة.

## قنوات المنتدى

قنوات منتدى ووسائط Discord تقبل فقط مشاركات المواضيع (Threads). يدعم OpenClaw طريقتين لإنشائها:

-   أرسل رسالة إلى المنتدى الأصلي (`channel:`) لإنشاء موضوع تلقائيًا. عنوان الموضوع يستخدم أول سطر غير فارغ من رسالتك.
-   استخدم `openclaw message thread create` لإنشاء موضوع مباشرة. لا تمرر `--message-id` لقنوات المنتدى.

مثال: أرسل إلى المنتدى الأصلي لإنشاء موضوع

```bash
openclaw message send --channel discord --target channel:<forumId> \
  --message "Topic title\nBody of the post"
```

مثال: إنشاء موضوع منتدى صراحةً

```bash
openclaw message thread create --channel discord --target channel:<forumId> \
  --thread-name "Topic title" --message "Body of the post"
```

المنتديات الأصلية لا تقبل مكونات Discord. إذا كنت بحاجة إلى مكونات، أرسل إلى الموضوع نفسه (`channel:`).

## المكونات التفاعلية

يدعم OpenClaw حاويات مكونات Discord الإصدار 2 لرسائل الوكيل. استخدم أداة الرسالة مع حمولة `components`. يتم توجيه نتائج التفاعل مرة أخرى إلى الوكيل كرسائل واردة عادية وتتبع إعدادات Discord الحالية `replyToMode`. الكتل المدعومة:

-   `text`, `section`, `separator`, `actions`, `media-gallery`, `file`
-   صفوف الإجراءات تسمح بما يصل إلى 5 أزرار أو قائمة اختيار واحدة
-   أنواع الاختيار: `string`, `user`, `role`, `mentionable`, `channel`

افتراضيًا، المكونات للاستخدام لمرة واحدة. عيّن `components.reusable=true` للسماح للأزرار والقوائم والنماذج بالاستخدام عدة مرات حتى تنتهي صلاحيتها. لتقييد من يمكنه النقر على زر، عيّن `allowedUsers` على ذلك الزر (معرفات مستخدم Discord، أو علامات، أو `*`). عند التكوين، يتلقى المستخدمون غير المطابقين رفضًا مؤقتًا. تفتح الأوامر المائلة `/model` و `/models` منتقي نماذج تفاعلي مع قوائم منسدلة للمزود والنموذج بالإضافة إلى خطوة Submit. رد المنتقي مؤقت ولا يمكن استخدامه إلا من قبل المستخدم الذي استدعاه. مرفقات الملفات:

-   كتل `file` يجب أن تشير إلى مرجع مرفق (`attachment://`)
-   قدّم المرفق عبر `media`/`path`/`filePath` (ملف واحد)؛ استخدم `media-gallery` لملفات متعددة
-   استخدم `filename` لتجاوز اسم التحميل عندما يجب أن يطابق مرجع المرفق

النماذج المنبثقة (Modal forms):

-   أضف `components.modal` مع ما يصل إلى 5 حقول
-   أنواع الحقول: `text`, `checkbox`, `radio`, `select`, `role-select`, `user-select`
-   يضيف OpenClaw زر التشغيل تلقائيًا

مثال:

```json
{
  channel: "discord",
  action: "send",
  to: "channel:123456789012345678",
  message: "Optional fallback text",
  components: {
    reusable: true,
    text: "Choose a path",
    blocks: [
      {
        type: "actions",
        buttons: [
          {
            label: "Approve",
            style: "success",
            allowedUsers: ["123456789012345678"],
          },
          { label: "Decline", style: "danger" },
        ],
      },
      {
        type: "actions",
        select: {
          type: "string",
          placeholder: "Pick an option",
          options: [
            { label: "Option A", value: "a" },
            { label: "Option B", value: "b" },
          ],
        },
      },
    ],
    modal: {
      title: "Details",
      triggerLabel: "Open form",
      fields: [
        { type: "text", label: "Requester" },
        {
          type: "select",
          label: "Priority",
          options: [
            { label: "Low", value: "low" },
            { label: "High", value: "high" },
          ],
        },
      ],
    },
  },
}
```

## التحكم في الوصول والتوجيه

`channels.discord.dmPolicy` تتحكم في الوصول للرسائل المباشرة (قديم: `channels.discord.dm.policy`):

-   `pairing` (افتراضي)
-   `allowlist`
-   `open` (يتطلب تضمين `"*"` في `channels.discord.allowFrom`؛ قديم: `channels.discord.dm.allowFrom`)
-   `disabled`

إذا لم تكن سياسة الرسائل المباشرة مفتوحة، يتم حظر المستخدمين غير المعروفين (أو مطالبتهم بالإقران في وضع `pairing`). أولوية الحسابات المتعددة:

-   `channels.discord.accounts.default.allowFrom` تُطبق فقط على الحساب `default`.
-   الحسابات المسماة ترث `channels.discord.allowFrom` عندما يكون `allowFrom` الخاص بها غير مضبوط.
-   الحسابات المسماة لا ترث `channels.discord.accounts.default.allowFrom`.

تنسيق هدف الرسائل المباشرة للتسليم:

-   `user:`
-   ذكر `<@id>`

معرفات رقمية مجردة غامضة ويتم رفضها ما لم يتم تقديم نوع هدف صريح للمستخدم/القناة.

```json
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          ignoreOtherMentions: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

رسائل النقابة مقيدة بالذكر افتراضيًا. اكتشاف الذكر يشمل:

-   ذكر صريح للبوت
-   أنماط ذكر مُكونة (`agents.list[].groupChat.mentionPatterns`، الاحتياطي `messages.groupChat.mentionPatterns`)
-   سلوك الرد-على-البوت الضمني في الحالات المدعومة

`requireMention` يتم تكوينها لكل نقابة/قناة (`channels.discord.guilds...`). `ignoreOtherMentions` تختياريًا تحذف الرسائل التي تذكر مستخدم/صلاحية آخر ولكن ليس البوت (باستثناء @everyone/@here). الرسائل المباشرة الجماعية:

-   افتراضي: يتم تجاهلها (`dm.groupEnabled=false`)
-   قائمة سماح اختيارية عبر `dm.groupChannels` (معرفات القنوات أو السلاجات)

### توجيه الوكيل بناءً على الصلاحية

استخدم `bindings[].match.roles` لتوجيه أعضاء نقابة Discord إلى وكلاء مختلفين حسب معرف الصلاحية. الربط بناءً على الصلاحية يقبل معرفات الصلاحية فقط ويتم تقييمه بعد ربط الأقران أو الأقران-الأبوين وقبل الربط المقتصر على النقابة. إذا حدد الربط أيضًا حقول مطابقة أخرى (على سبيل المثال `peer` + `guildId` + `roles`)، يجب أن تتطابق جميع الحقول المُكونة.

```json
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## إعداد بوابة المطور

1.  بوابة مطوري Discord -> **Applications** -> **New Application**
2.  **Bot** -> **Add Bot**
3.  نسخ رمز البوت

في **Bot -> Privileged Gateway Intents**، قم بتمكين:

-   Message Content Intent
-   Server Members Intent (موصى به)

نية الحالة اختيارية ومطلوبة فقط إذا كنت تريد تلقي تحديثات الحالة. تعيين حالة البوت (`setPresence`) لا يتطلب تمكين تحديثات الحالة للأعضاء.

منشئ رابط OAuth:

-   النطاقات: `bot`, `applications.commands`

الأذونات الأساسية النموذجية:

-   View Channels
-   Send Messages
-   Read Message History
-   Embed Links
-   Attach Files
-   Add Reactions (اختياري)

تجنب `Administrator` ما لم يكن مطلوبًا صراحةً.

قم بتمكين وضع المطور في Discord، ثم انسخ:

-   معرف الخادم
-   معرف القناة
-   معرف المستخدم

يفضل استخدام المعرفات الرقمية في تكوين OpenClaw لعمليات التدقيق والفحص الموثوقة.

## الأوامر الأصلية ومصادقة الأوامر

-   `commands.native` افتراضيًا `"auto"` ومُمكّن لـ Discord.
-   تجاوز لكل قناة: `channels.discord.commands.native`.
-   `commands.native=false` تمسح صراحةً أوامر Discord الأصلية المسجلة مسبقًا.
-   مصادقة الأمر الأصلي تستخدم نفس قوائم السماح/السياسات في Discord مثل معالجة الرسائل العادية.
-   قد تظل الأوامر مرئية في واجهة مستخدم Discord للمستخدمين غير المصرح لهم؛ التنفيذ لا يزال يفرض مصادقة OpenClaw ويعيد "غير مصرح".

راجع [الأوامر المائلة (Slash commands)](../tools/slash-commands.md) لفهرس الأوامر وسلوكها. إعدادات الأمر المائل الافتراضية:

-   `ephemeral: true`

## تفاصيل الميزات

يدعم Discord علامات الرد في مخرجات الوكيل:

-   `[[reply_to_current]]`
-   `[[reply_to:]]`

تتحكم بها `channels.discord.replyToMode`:

-   `off` (افتراضي)
-   `first`
-   `all`

ملاحظة: `off` تعطيل خيط الرد الضمني. لا تزال علامات `[[reply_to_*]]` الصريحة مُحترمة. معرفات الرسائل تظهر في السياق/السجل حتى يتمكن الوكلاء من استهداف رسائل محددة.

يمكن لـ OpenClaw بث مسودات الردود عن طريق إرسال رسالة مؤقتة وتحريرها مع وصول النص.

-   `channels.discord.streaming` تتحكم في بث المعاينة (`off` | `partial` | `block` | `progress`, افتراضي: `off`).
-   `progress` مقبول لاتساق عبر القنوات ويُحوّل إلى `partial` على Discord.
-   `channels.discord.streamMode` اسم قديم ويتم ترحيله تلقائيًا.
-   `partial` تحرر رسالة معاينة واحدة مع وصول الرموز (Tokens).
-   `block` تُصدر أجزاء بحجم المسودة (استخدم `draftChunk` لضبط الحجم ونقاط التوقف).

مثال:

```json
{
  channels: {
    discord: {
      streaming: "partial",
    },
  },
}
```

تجزئة وضع `block` الافتراضية (مقيدة بـ `channels.discord.textChunkLimit`):

```json
{
  channels: {
    discord: {
      streaming: "block",
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph",
      },
    },
  },
}
```

بث المعاينة للنص فقط؛ ردود الوسائط تعود إلى التسليم العادي. ملاحظة: بث المعاينة منفصل عن البث المجزأ. عندما يتم تمكين البث المجزأ صراحةً لـ Discord، يتخطى OpenClaw بث المعاينة لتجنب البث المزدوج.

سياق سجل النقابة:

-   `channels.discord.historyLimit` افتراضي `20`
-   الاحتياطي: `messages.groupChat.historyLimit`
-   `0` تعطيل

ضوابط سجل الرسائل المباشرة:

-   `channels.discord.dmHistoryLimit`
-   `channels.discord.dms["<user_id>"].historyLimit`

سلوك المواضيع (Threads):

-   مواضيع Discord يتم توجيهها كجلسات قنوات
-   يمكن استخدام بيانات وصفية للموضوع الأصلي لربط الجلسة الأصلية
-   تكوين الموضوع يرث تكوين القناة الأصلية ما لم يكن هناك إدخال محدد للموضوع

مواضيع القنوات تُحقن كسياق **غير موثوق** (ليس كتوجيه نظام).

يمكن لـ Discord ربط موضوع بجلسة مستهدفة بحيث تحافظ الرسائل اللاحقة في ذلك الموضوع على التوجيه إلى نفس الجلسة (بما في ذلك جلسات الوكلاء الفرعيين). الأوامر:

-   `/focus ` ربط الموضوع الحالي/الجديد بوكيل فرعي/جلسة مستهدفة
-   `/unfocus` إزالة ربط الموضوع الحالي
-   `/agents` عرض التشغيلات النشطة وحالة الربط
-   `/session idle <duration|off>` فحص/تحديث عدم النشاط التلقائي لفك الربط للربطات المركزة
-   `/session max-age <duration|off>` فحص/تحديث الحد الأقصى للعمر الصارم للربطات المركزة

التكوين:

```json
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // اختياري
      },
    },
  },
}
```

ملاحظات:

-   `session.threadBindings.*` يضع الإعدادات الافتراضية العالمية.
-   `channels.discord.threadBindings.*` يتجاوز سلوك Discord.
-   `spawnSubagentSessions` يجب أن تكون true لإنشاء/ربط مواضيع تلقائيًا لـ `sessions_spawn({ thread: true })`.
-   `spawnAcpSessions` يجب أن تكون true لإنشاء/ربط مواضيع تلقائيًا لـ ACP (`/acp spawn ... --thread ...` أو `sessions_spawn({ runtime: "acp", thread: true })`).
-   إذا تم تعطيل ربط المواضيع لحساب، فإن `/focus` وعمليات ربط المواضيع ذات الصلة غير متاحة.

راجع [الوكلاء الفرعيون](../tools/subagents.md)، [وكلاء ACP](../tools/acp-agents.md)، و [مرجع التكوين](../gateway/configuration-reference.md).

لمساحات عمل ACP "دائمة التشغيل" مستقرة، قم بتكوين ربطات ACP مُكتوبة على المستوى الأعلى تستهدف محادثات Discord. مسار التكوين:

-   `bindings[]` مع `type: "acp"` و `match.channel: "discord"`

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
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

ملاحظات:

-   يمكن لرسائل الموضوع أن ترث ربط ACP للقناة الأصلية.
-   في قناة أو موضوع مُربط، `/new` و `/reset` يعيدان تعيين نفس جلسة ACP في مكانها.
-   ربطات المواضيع المؤقتة لا تزال تعمل ويمكنها تجاوز حل الهدف أثناء نشاطها.

راجع [وكلاء ACP](../tools/acp-agents.md) للحصول على تفاصيل سلوك الربط.

وضع إشعار التفاعل لكل نقابة:

-   `off`
-   `own` (افتراضي)
-   `all`
-   `allowlist` (يستخدم `guilds..users`)

يتم تحويل أحداث التفاعل إلى أحداث نظام وإرفاقها بجلسة Discord الموجهة.

`ackReaction` ترسل رمز تعبيري تأكيد بينما يعالج OpenClaw رسالة واردة. ترتيب الحل:

-   `channels.discord.accounts..ackReaction`
-   `channels.discord.ackReaction`
-   `messages.ackReaction`
-   رمز تعبيري احتياطي لهوية الوكيل (`agents.list[].identity.emoji`، وإلا ”👀”)

ملاحظات:

-   Discord يقبل رموز التعبيري يونيكود أو أسماء رموز التعبيري المخصصة.
-   استخدم `""` لتعطيل التفاعل لقناة أو حساب.

كتابات التكوين التي يبدأها القناة مفعلة افتراضيًا. هذا يؤثر على تدفقات `/config set|unset` (عند تمكين ميزات الأوامر). التعطيل:

```json
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

توجيه حركة مرور WebSocket لبوابة Discord وطلبات REST عند بدء التشغيل (معرف التطبيق + حل قائمة السماح) عبر وكيل وسيط HTTP(S) مع `channels.discord.proxy`.

```json
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

تجاوز لكل حساب:

```json
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

تمكين حل PluralKit لتعيين الرسائل التي تمر عبر الوكيل إلى هوية عضو النظام:

```json
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // اختياري؛ مطلوب للأنظمة الخاصة
      },
    },
  },
}
```

ملاحظات:

-   قوائم السماح يمكنها استخدام `pk:`
-   أسماء عرض الأعضاء تتم مطابقتها بالاسم/السلاج فقط عندما `channels.discord.dangerouslyAllowNameMatching: true`
-   عمليات البحث تستخدم معرف الرسالة الأصلي ومقيدة بنافذة زمنية
-   إذا فشل البحث، يتم التعامل مع الرسائل التي تمر عبر الوكيل كرسائل بوت ويتم إسقاطها ما لم يكن `allowBots=true`

يتم تطبيق تحديثات الحالة عند تعيينك لحقل حالة أو نشاط، أو عند تمكين الحالة التلقائية. مثال للحالة فقط:

```json
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

مثال للنشاط (الحالة المخصصة هي نوع النشاط الافتراضي):

```json
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

مثال للبث:

```json
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

خريطة أنواع النشاط:

-   0: يلعب
-   1: يبث (يتطلب `activityUrl`)
-   2: يستمع
-   3: يشاهد
-   4: مخصص (يستخدم نص النشاط كحالة الحالة؛ الإيموجي اختياري)
-   5: يتنافس

مثال الحالة التلقائية (إشارة صحة وقت التشغيل):

```json
{
  channels: {
    discord: {
      autoPresence: {
        enabled: true,
        intervalMs: 30000,
        minUpdateIntervalMs: 15000,
        exhaustedText: "tokens exhausted",
      },
    },
  },
}
```

تربط الحالة التلقائية توفر وقت التشغيل بحالة Discord: صحي => متصل، متدهور أو غير معروف => خاملاً، مستنفد أو غير متاح => لا تزعج. استبدالات النص اختيارية:

-   `autoPresence.healthyText`
-   `autoPresence.degradedText`
-   `autoPresence.exhaustedText` (يدعم العنصر النائب `{reason}`)

يدعم Discord موافقات التنفيذ المستندة إلى الأزرار في الرسائل المباشرة ويمكنه نشر مطالبات الموافقة اختيارياً في القناة الأصلية. مسار التكوين:

-   `channels.discord.execApprovals.enabled`
-   `channels.discord.execApprovals.approvers`
-   `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, افتراضي: `dm`)
-   `agentFilter`, `sessionFilter`, `cleanupAfterResolve`

عندما يكون `target` هو `channel` أو `both`، تكون مطالبة الموافقة مرئية في القناة. يمكن للموافقين المكونين فقط استخدام الأزرار؛ المستخدمون الآخرون يتلقون رفضاً مؤقتاً. تتضمن مطالبات الموافقة نص الأمر، لذا قم بتمكين التسليم في القناة فقط في القنوات الموثوقة. إذا تعذر اشتقاق معرف القناة من مفتاح الجلسة، يعود OpenClaw إلى التسليم عبر الرسالة المباشرة.

إذا فشلت الموافقات مع معرفات موافقة غير معروفة، تحقق من قائمة الموافقين وتفعيل الميزة.

المستندات ذات الصلة: [موافقات التنفيذ](../tools/exec-approvals.md)

## أدوات وبوابات الإجراء

## الأمان والعمليات

-   عامل التكامل كمفاتيح سرية (`DISCORD_BOT_TOKEN` المفضل في البيئات الخاضعة للإشراف).
-   امنح أقل امتيازات Discord.
-   إذا كان نشر الأمر/الحالة قديماً، أعد تشغيل البوابة وتحقق مرة أخرى باستخدام `openclaw channels status --probe`.

## ذي صلة

-   [الإقران](./pairing.md)
-   [توجيه القنوات](./channel-routing.md)
-   [توجيه الوكلاء المتعددين](../concepts/multi-agent.md)
-   [استكشاف الأخطاء وإصلاحها](./troubleshooting.md)
-   [أوامر الشرطة المائلة](../tools/slash-commands.md)
