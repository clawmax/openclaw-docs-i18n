

  منصات المراسلة

  
# Slack

الحالة: جاهز للإنتاج للرسائل المباشرة + القنوات عبر تكاملات تطبيق Slack. الوضع الافتراضي هو Socket Mode؛ كما يتم دعم وضع HTTP Events API.

## الإعداد السريع

## نموذج الرموز المميزة

-   `botToken` + `appToken` مطلوبان لوضع Socket Mode.
-   وضع HTTP يتطلب `botToken` + `signingSecret`.
-   رموز التكوين تتجاوز الرجوع إلى متغيرات البيئة.
-   الرجوع إلى متغير البيئة `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` ينطبق فقط على الحساب الافتراضي.
-   `userToken` (`xoxp-...`) خاص بالتكوين فقط (لا رجوع إلى متغير البيئة) ويفترض سلوك القراءة فقط افتراضيًا (`userTokenReadOnly: true`).
-   اختياري: أضف `chat:write.customize` إذا كنت تريد أن تستخدم الرسائل الصادرة هوية الوكيل النشط (`username` مخصص وأيقونة). يستخدم `icon_emoji` بناء الجملة `:emoji_name:`.

> **💡** للإجراءات/قراءات الدليل، يمكن تفضيل رمز المستخدم عند تكوينه. للكتابة، يبقى رمز البوت مفضلًا؛ كتابات رمز المستخدم مسموح بها فقط عندما يكون `userTokenReadOnly: false` ورمز البوت غير متوفر.

## التحكم في الوصول والتوجيه

يتحكم `channels.slack.dmPolicy` في الوصول إلى الرسائل المباشرة (قديم: `channels.slack.dm.policy`):

-   `pairing` (افتراضي)
-   `allowlist`
-   `open` (يتطلب تضمين `"*"` في `channels.slack.allowFrom`؛ قديم: `channels.slack.dm.allowFrom`)
-   `disabled`

علامات الرسائل المباشرة:

-   `dm.enabled` (افتراضي true)
-   `channels.slack.allowFrom` (مفضل)
-   `dm.allowFrom` (قديم)
-   `dm.groupEnabled` (الرسائل المباشرة الجماعية افتراضيًا false)
-   `dm.groupChannels` (قائمة السماح MPIM اختيارية)

أولوية الحسابات المتعددة:

-   `channels.slack.accounts.default.allowFrom` ينطبق فقط على الحساب `default`.
-   الحسابات المسماة ترث `channels.slack.allowFrom` عندما يكون `allowFrom` الخاص بها غير معين.
-   الحسابات المسماة لا ترث `channels.slack.accounts.default.allowFrom`.

يستخدم الاقتران في الرسائل المباشرة الأمر `openclaw pairing approve slack `.

يتحكم `channels.slack.groupPolicy` في معالجة القناة:

-   `open`
-   `allowlist`
-   `disabled`

توجد قائمة السماح للقناة تحت `channels.slack.channels`.
ملاحظة وقت التشغيل: إذا كان `channels.slack` مفقودًا تمامًا (إعداد يعتمد فقط على متغيرات البيئة)، فإن وقت التشغيل يتراجع إلى `groupPolicy="allowlist"` ويسجل تحذيرًا (حتى إذا تم تعيين `channels.defaults.groupPolicy`).
تحديد الاسم/المعرف:

-   يتم حل إدخالات قائمة السماح للقناة وإدخالات قائمة السماح للرسائل المباشرة عند بدء التشغيل عندما يسمح الوصول بالرمز المميز
-   يتم الاحتفاظ بالإدخالات غير المحلولة كما تم تكوينها
-   مطابقة التفويض الوارد تكون بالمعرف أولاً افتراضيًا؛ مطابقة اسم المستخدم/الرابط المباشر تتطلب `channels.slack.dangerouslyAllowNameMatching: true`

رسائل القناة مقيدة بالإشارة افتراضيًا.
مصادر الإشارة:

-   إشارة صريحة للتطبيق (`<@botId>`)
-   أنماط التعبير العادي للإشارة (`agents.list[].groupChat.mentionPatterns`، الرجوع `messages.groupChat.mentionPatterns`)
-   سلوك الرد على بوت ضمن الخيط الضمني

ضوابط لكل قناة (`channels.slack.channels.<id|name>`):

-   `requireMention`
-   `users` (قائمة السماح)
-   `allowBots`
-   `skills`
-   `systemPrompt`
-   `tools`, `toolsBySender`
-   تنسيق مفتاح `toolsBySender`: `id:`, `e164:`, `username:`, `name:`, أو الحرف العام `"*"` (المفاتيح القديمة غير المسبوقة لا تزال تُرسم إلى `id:` فقط)

## الأوامر وسلوك الشرطة المائلة

-   الوضع التلقائي للأمر الأصلي **معطل** لـ Slack (`commands.native: "auto"` لا يُفعّل أوامر Slack الأصلية).
-   فعّل معالجات أوامر Slack الأصلية باستخدام `channels.slack.commands.native: true` (أو `commands.native: true` عام).
-   عند تمكين الأوامر الأصلية، سجل أوامر الشرطة المائلة المطابقة في Slack (`/` أسماء)، باستثناء واحد:
    -   سجل `/agentstatus` لأمر الحالة (Slack يحتفظ بـ `/status`)
-   إذا لم تكن الأوامر الأصلية مفعلة، يمكنك تشغيل أمر شرطة مائلة واحد مُكون عبر `channels.slack.slashCommand`.
-   قوائم وسيطات الأمر الأصلي الآن تتكيف مع استراتيجية العرض:
    -   حتى 5 خيارات: كتل أزرار
    -   6-100 خيار: قائمة اختيار ثابتة
    -   أكثر من 100 خيار: اختيار خارجي مع تصفية خيارات غير متزامنة عندما تكون معالجات خيارات التفاعل متاحة
    -   إذا تجاوزت قيم الخيارات المشفرة حدود Slack، يتراجع التدفق إلى الأزرار
-   للحِمل الطويل للخيارات، تستخدم قوائم وسيطات أمر الشرطة المائلة مربع حوار تأكيد قبل إرسال القيمة المحددة.

إعدادات أمر الشرطة المائلة الافتراضية:

-   `enabled: false`
-   `name: "openclaw"`
-   `sessionPrefix: "slack:slash"`
-   `ephemeral: true`

جلسات الشرطة المائلة تستخدم مفاتيح معزولة:

-   `agent::slack:slash:`

ولا تزال توجه تنفيذ الأمر ضد جلسة المحادثة المستهدفة (`CommandTargetSessionKey`).

## الخيوط، الجلسات، وعلامات الرد

-   الرسائل المباشرة توجه كـ `direct`؛ القنوات كـ `channel`؛ MPIMs كـ `group`.
-   مع `session.dmScope=main` الافتراضي، تنهار الرسائل المباشرة في Slack إلى الجلسة الرئيسية للوكيل.
-   جلسات القناة: `agent::slack:channel:`.
-   يمكن لردود الخيط إنشاء لاحقات جلسة خيط (`:thread:`) عند الاقتضاء.
-   `channels.slack.thread.historyScope` الافتراضي هو `thread`؛ `thread.inheritParent` الافتراضي هو `false`.
-   يتحكم `channels.slack.thread.initialHistoryLimit` في عدد رسائل الخيط الموجودة التي يتم جلبها عند بدء جلسة خيط جديدة (افتراضي `20`؛ عيّن `0` لتعطيل).

ضوابط خيط الرد:

-   `channels.slack.replyToMode`: `off|first|all` (افتراضي `off`)
-   `channels.slack.replyToModeByChatType`: لكل `direct|group|channel`
-   الرجوع القديم للمحادثات المباشرة: `channels.slack.dm.replyToMode`

علامات الرد اليدوية مدعومة:

-   `[[reply_to_current]]`
-   `[[reply_to:]]`

ملاحظة: `replyToMode="off"` يعطل **جميع** خيوط الرد في Slack، بما في ذلك علامات `[[reply_to_*]]` الصريحة. هذا يختلف عن Telegram، حيث لا تزال علامات الرد الصريحة مُحترمة في الوضع `"off"`. يعكس الاختلاف نماذج الخيوط في المنصة: خيوط Slack تخفي الرسائل من القناة، بينما تبقى ردود Telegram مرئية في تدفق المحادثة الرئيسي.

## الوسائط، التقسيم، والتسليم

يتم تنزيل مرفقات ملفات Slack من عناوين URL خاصة مستضافة على Slack (تدفق طلب مصادق بالرمز المميز) وكتابتها إلى مخزن الوسائط عند نجاح الجلب والسماح بحدود الحجم.
الحد الأقصى لحجم الوارد وقت التشغيل افتراضيًا `20MB` ما لم يتم تجاوزه بواسطة `channels.slack.mediaMaxMb`.

-   أجزاء النص تستخدم `channels.slack.textChunkLimit` (افتراضي 4000)
-   `channels.slack.chunkMode="newline"` يُفعّل التقسيم حسب الفقرة أولاً
-   إرسال الملفات يستخدم واجهات برمجة تطبيقات تحميل Slack ويمكن أن يتضمن ردود الخيط (`thread_ts`)
-   يتبع حد الوسائط الصادر `channels.slack.mediaMaxMb` عند التكوين؛ وإلا تستخدم إرسالات القناة الافتراضيات حسب نوع MIME من خط أنابيب الوسائط

الأهداف الصريحة المفضلة:

-   `user:` للرسائل المباشرة
-   `channel:` للقنوات

يتم فتح الرسائل المباشرة في Slack عبر واجهات برمجة تطبيقات محادثة Slack عند الإرسال إلى أهداف المستخدم.

## الإجراءات والبوابات

يتم التحكم في إجراءات Slack بواسطة `channels.slack.actions.*`. مجموعات الإجراءات المتاحة في أدوات Slack الحالية:

| المجموعة | الافتراضي |
| --- | --- |
| messages | مفعل |
| reactions | مفعل |
| pins | مفعل |
| memberInfo | مفعل |
| emojiList | مفعل |

## الأحداث والسلوك التشغيلي

-   تحرير/حذف الرسائل وبث الخيوط يتم تعيينها في أحداث النظام.
-   أحداث إضافة/إزالة التفاعل يتم تعيينها في أحداث النظام.
-   أحداث انضمام/مغادرة الأعضاء، وإنشاء/إعادة تسمية القناة، وإضافة/إزالة التثبيت يتم تعيينها في أحداث النظام.
-   تحديثات حالة خيط المساعد (لمؤشرات "يكتب..." في الخيوط) تستخدم `assistant.threads.setStatus` وتتطلب نطاق البوت `assistant:write`.
-   يمكن لـ `channel_id_changed` نقل مفاتيح تكوين القناة عندما يكون `configWrites` مفعلًا.
-   يتم التعامل مع بيانات وصفية لموضوع/غرض القناة كسياق غير موثوق ويمكن حقنها في سياق التوجيه.
-   إجراءات الكتل وتفاعلات النماذج تنبعث أحداث نظام منظمة `Slack interaction: ...` بحقول حمولة غنية:
    -   إجراءات الكتل: القيم المحددة، التسميات، قيم منتقي القيم، وبيانات وصفية `workflow_*`
    -   أحداث `view_submission` و `view_closed` للنماذج مع بيانات وصفية قناة موجهة ومدخلات النموذج

## تفاعلات التأكيد

`ackReaction` يرسل رمز تعبيري تأكيد بينما يعالج OpenClaw رسالة واردة. ترتيب الحل:

-   `channels.slack.accounts..ackReaction`
-   `channels.slack.ackReaction`
-   `messages.ackReaction`
-   الرجوع إلى رمز تعبيري هوية الوكيل (`agents.list[].identity.emoji`، وإلا ”👀“)

ملاحظات:

-   يتوقع Slack رموز قصيرة (على سبيل المثال `"eyes"`).
-   استخدم `""` لتعطيل التفاعل لحساب Slack أو عالميًا.

## الرجوع إلى تفاعل الكتابة

`typingReaction` يضيف تفاعلًا مؤقتًا إلى رسالة Slack الواردة بينما يعالج OpenClaw ردًا، ثم يزيله عند انتهاء التشغيل. هذا رجوع مفيد عندما يكون الكتابة الأصلية للمساعد في Slack غير متاحة، خاصة في الرسائل المباشرة. ترتيب الحل:

-   `channels.slack.accounts..typingReaction`
-   `channels.slack.typingReaction`

ملاحظات:

-   يتوقع Slack رموز قصيرة (على سبيل المثال `"hourglass_flowing_sand"`).
-   التفاعل هو أفضل جهد ويتم محاولة التنظيف تلقائيًا بعد اكتمال مسار الرد أو الفشل.

## البيان وقائمة التحقق للنطاقات

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "assistant:write",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

إذا قمت بتكوين `channels.slack.userToken`، فإن النطاقات النموذجية للقراءة هي:

-   `channels:history`, `groups:history`, `im:history`, `mpim:history`
-   `channels:read`, `groups:read`, `im:read`, `mpim:read`
-   `users:read`
-   `reactions:read`
-   `pins:read`
-   `emoji:read`
-   `search:read` (إذا كنت تعتمد على قراءات بحث Slack)

## استكشاف الأخطاء وإصلاحها

تحقق، بالترتيب:

-   `groupPolicy`
-   قائمة السماح للقناة (`channels.slack.channels`)
-   `requireMention`
-   قائمة السماح `users` لكل قناة

أوامر مفيدة:

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

تحقق:

-   `channels.slack.dm.enabled`
-   `channels.slack.dmPolicy` (أو القديم `channels.slack.dm.policy`)
-   موافقات الاقتران / إدخالات قائمة السماح

```bash
openclaw pairing list slack
```

تحقق من صلاحية رموز البوت + التطبيق وتمكين Socket Mode في إعدادات تطبيق Slack.

تحقق من:

-   سر التوقيع
-   مسار webhook
-   عناوين URL لطلب Slack (الأحداث + التفاعل + أوامر الشرطة المائلة)
-   `webhookPath` فريد لكل حساب HTTP

تحقق مما إذا كنت تقصد:

-   وضع الأمر الأصلي (`channels.slack.commands.native: true`) مع أوامر شرطة مائلة مطابقة مسجلة في Slack
-   أو وضع أمر شرطة مائلة واحد (`channels.slack.slashCommand.enabled: true`)

تحقق أيضًا من `commands.useAccessGroups` وقوائم السماح للقناة/المستخدم.

## بث النص

يدعم OpenClaw بث النص الأصلي لـ Slack عبر واجهة برمجة تطبيقات الوكيلين والتطبيقات الذكية. يتحكم `channels.slack.streaming` في سلوك المعاينة المباشرة:

-   `off`: تعطيل بث المعاينة المباشرة.
-   `partial` (افتراضي): استبدال نص المعاينة بأحدث الإخراج الجزئي.
-   `block`: إلحاق تحديثات المعاينة المجزأة.
-   `progress`: عرض نص حالة التقدم أثناء التوليد، ثم إرسال النص النهائي.

يتحكم `channels.slack.nativeStreaming` في واجهة برمجة تطبيقات البث الأصلية لـ Slack (`chat.startStream` / `chat.appendStream` / `chat.stopStream`) عندما يكون `streaming` هو `partial` (افتراضي: `true`). عطّل بث Slack الأصلي (احتفظ بسلوب مسودة المعاينة):

```yaml
channels:
  slack:
    streaming: partial
    nativeStreaming: false
```

مفاتيح قديمة:

-   `channels.slack.streamMode` (`replace | status_final | append`) يتم ترحيله تلقائيًا إلى `channels.slack.streaming`.
-   القيمة المنطقية `channels.slack.streaming` يتم ترحيلها تلقائيًا إلى `channels.slack.nativeStreaming`.

### المتطلبات

1.  فعّل **الوكلاء والتطبيقات الذكية** في إعدادات تطبيق Slack الخاص بك.
2.  تأكد من أن التطبيق لديه نطاق `assistant:write`.
3.  يجب أن يكون خيط رد متاحًا لتلك الرسالة. اختيار الخيط لا يزال يتبع `replyToMode`.

### السلوك

-   تبدأ أول قطعة نصية بثًا (`chat.startStream`).
-   تقوم القطع النصية اللاحقة بإلحاق نفس البث (`chat.appendStream`).
-   نهاية الرد ينهي البث (`chat.stopStream`).
-   الوسائط وحِمل غير النص تتراجع إلى التسليم العادي.
-   إذا فشل البث أثناء الرد، يتراجع OpenClaw إلى التسليم العادي للحمولات المتبقية.

## مؤشرات مرجع التكوين

المرجع الأساسي:

-   [مرجع التكوين - Slack](../gateway/configuration-reference.md#slack) حقول Slack عالية الإشارة:
    -   الوضع/المصادقة: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
    -   الوصول للرسائل المباشرة: `dm.enabled`, `dmPolicy`, `allowFrom` (قديم: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
    -   تبديل التوافق: `dangerouslyAllowNameMatching` (كسر الزجاج؛ أبقِه معطلاً إلا إذا لزم الأمر)
    -   الوصول للقناة: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
    -   الخيوط/السجل: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
    -   التسليم: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `streaming`, `nativeStreaming`
    -   العمليات/الميزات: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## ذات صلة

-   [الاقتران](./pairing.md)
-   [توجيه القناة](./channel-routing.md)
-   [استكشاف الأخطاء وإصلاحها](./troubleshooting.md)
-   [التكوين](../gateway/configuration.md)
-   [أوامر الشرطة المائلة](../tools/slash-commands.md)

[Synology Chat](./synology-chat.md)[Telegram](./telegram.md)