title: "دليل تكامل BlueBubbles لـ iMessage مع OpenClaw AI"
description: "تعلم كيفية إعداد وتكوين قناة BlueBubbles لتكامل iMessage مع OpenClaw AI، بما في ذلك الميزات المتقدمة مثل التفاعلات والتحرير وإدارة المجموعات."
keywords: ["bluebubbles", "تكامل imessage", "openclaw", "مراسلة macos", "إعداد bluebubbles", "أتمتة imessage", "webhook bluebubbles", "بوت imessage"]
---

  منصات المراسلة

  
# BlueBubbles

الحالة: مكون إضافي مجمع يتحدث إلى خادم BlueBubbles لنظام macOS عبر HTTP. **موصى به لتكامل iMessage** نظرًا لواجهة برمجة التطبيقات (API) الأغنى وإعداد أسهل مقارنة بقناة imsg القديمة.

## نظرة عامة

-   يعمل على macOS عبر التطبيق المساعد BlueBubbles ([bluebubbles.app](https://bluebubbles.app)).
-   موصى به/مختبر: macOS Sequoia (15). يعمل macOS Tahoe (26)؛ التحرير معطل حاليًا على Tahoe، وقد تبلغ تحديثات أيقونة المجموعة عن النجاح ولكن لا تتم المزامنة.
-   يتحدث OpenClaw إليه من خلال واجهة برمجة التطبيقات REST الخاصة به (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
-   تصل الرسائل الواردة عبر webhooks؛ الردود الصادرة، ومؤشرات الكتابة، وإيصالات القراءة، وردود الفعل (tapbacks) هي استدعاءات REST.
-   يتم استيعاب المرفقات والملصقات كوسائط واردة (وعرضها للوكيل عند الإمكان).
-   يعمل الاقتران/القائمة المسموح بها بنفس طريقة القنوات الأخرى (`/channels/pairing` إلخ) مع `channels.bluebubbles.allowFrom` + رموز الاقتران.
-   يتم عرض التفاعلات كأحداث نظام تمامًا مثل Slack/Telegram حتى يتمكن الوكلاء من "الإشارة" إليها قبل الرد.
-   ميزات متقدمة: التحرير، الإلغاء، خيط الردود، تأثيرات الرسائل، إدارة المجموعات.

## البدء السريع

1.  قم بتثبيت خادم BlueBubbles على جهاز Mac الخاص بك (اتبع التعليمات الموجودة على [bluebubbles.app/install](https://bluebubbles.app/install)).
2.  في تكوين BlueBubbles، قم بتمكين واجهة برمجة التطبيقات على الويب (web API) وتعيين كلمة مرور.
3.  قم بتشغيل `openclaw onboard` واختر BlueBubbles، أو قم بالتكوين يدويًا:
    
    نسخ
    
    ```json
    {
      channels: {
        bluebubbles: {
          enabled: true,
          serverUrl: "http://192.168.1.100:1234",
          password: "example-password",
          webhookPath: "/bluebubbles-webhook",
        },
      },
    }
    ```
    
4.  وجه webhooks الخاصة بـ BlueBubbles إلى بوابة الوصول الخاصة بك (مثال: `https://your-gateway-host:3000/bluebubbles-webhook?password=`).
5.  ابدأ تشغيل البوابة؛ سيقوم بتسجيل معالج webhook وبدء الاقتران.

ملاحظة أمنية:

-   قم دائمًا بتعيين كلمة مرور لـ webhook.
-   المصادقة على webhook مطلوبة دائمًا. يرفض OpenClaw طلبات webhook الخاصة بـ BlueBubbles ما لم تتضمن كلمة مرور/معرف (guid) يتطابق مع `channels.bluebubbles.password` (على سبيل المثال `?password=` أو `x-password`)، بغض النظر عن طوبولوجيا loopback/الوكيل.
-   يتم التحقق من مصادقة كلمة المرور قبل قراءة/تحليل أجسام webhook الكاملة.

## الحفاظ على تطبيق Messages نشطًا (إعدادات VM / بدون واجهة مستخدم)

يمكن أن تنتهي بعض إعدادات VM لنظام macOS / التشغيل الدائم بتوقف تطبيق Messages عن العمل "خاملًا" (تتوقف الأحداث الواردة حتى يتم فتح التطبيق/وضعه في المقدمة). الحل البسيط هو **تحفيز Messages كل 5 دقائق** باستخدام AppleScript + LaunchAgent.

### 1) حفظ AppleScript

احفظ هذا كـ:

-   `~/Scripts/poke-messages.scpt`

مثال على البرنامج النصي (غير تفاعلي؛ لا يسرق التركيز):

```
try
  tell application "Messages"
    if not running then
      launch
    end if

    -- Touch the scripting interface to keep the process responsive.
    set _chatCount to (count of chats)
  end tell
on error
  -- Ignore transient failures (first-run prompts, locked session, etc).
end try
```

### 2) تثبيت LaunchAgent

احفظ هذا كـ:

-   `~/Library/LaunchAgents/com.user.poke-messages.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>

    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript &quot;$HOME/Scripts/poke-messages.scpt&quot;</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>StartInterval</key>
    <integer>300</integer>

    <key>StandardOutPath</key>
    <string>/tmp/poke-messages.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/poke-messages.err</string>
  </dict>
</plist>
```

ملاحظات:

-   يعمل هذا **كل 300 ثانية** و**عند تسجيل الدخول**.
-   قد يؤدي التشغيل الأول إلى تشغيل مطالبات **الأتمتة** في macOS (`osascript` → Messages). وافق عليها في جلسة المستخدم نفسها التي تشغل LaunchAgent.

قم بتحميله:

```bash
launchctl unload ~/Library/LaunchAgents/com.user.poke-messages.plist 2>/dev/null || true
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

## الإعداد والتشغيل

BlueBubbles متاح في معالج الإعداد التفاعلي:

```bash
openclaw onboard
```

يطلب المعالج:

-   **عنوان URL للخادم** (مطلوب): عنوان خادم BlueBubbles (مثل `http://192.168.1.100:1234`)
-   **كلمة المرور** (مطلوب): كلمة مرور واجهة برمجة التطبيقات (API) من إعدادات خادم BlueBubbles
-   **مسار webhook** (اختياري): الافتراضي هو `/bluebubbles-webhook`
-   **سياسة المراسلة المباشرة (DM)**: الاقتران، القائمة المسموح بها، مفتوح، أو معطل
-   **القائمة المسموح بها**: أرقام الهواتف، عناوين البريد الإلكتروني، أو أهداف الدردشة

يمكنك أيضًا إضافة BlueBubbles عبر سطر الأوامر (CLI):

```bash
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## التحكم في الوصول (المراسلة المباشرة + المجموعات)

المراسلة المباشرة (DMs):

-   الافتراضي: `channels.bluebubbles.dmPolicy = "pairing"`.
-   يتلقى المرسلون غير المعروفين رمز اقتران؛ يتم تجاهل الرسائل حتى تتم الموافقة عليها (تنتهي صلاحية الرموز بعد ساعة واحدة).
-   الموافقة عبر:
    -   `openclaw pairing list bluebubbles`
    -   `openclaw pairing approve bluebubbles `
-   الاقتران هو تبادل الرمز الافتراضي. التفاصيل: [الاقتران](./pairing.md)

المجموعات:

-   `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (الافتراضي: `allowlist`).
-   `channels.bluebubbles.groupAllowFrom` يتحكم في من يمكنه التشغيل في المجموعات عند تعيين `allowlist`.

### التحكم بالإشارات (في المجموعات)

يدعم BlueBubbles التحكم بالإشارات (mention gating) لدردشات المجموعات، بما يتطابق مع سلوك iMessage/WhatsApp:

-   يستخدم `agents.list[].groupChat.mentionPatterns` (أو `messages.groupChat.mentionPatterns`) للكشف عن الإشارات.
-   عند تمكين `requireMention` لمجموعة ما، يستجيب الوكيل فقط عند الإشارة إليه.
-   تتجاوز أوامر التحكم من المرسلين المصرح لهم التحكم بالإشارات.

التكوين لكل مجموعة:

```json
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }, // الافتراضي لجميع المجموعات
        "iMessage;-;chat123": { requireMention: false }, // تجاوز لمجموعة محددة
      },
    },
  },
}
```

### التحكم بالأوامر

-   تتطلب أوامر التحكم (مثل `/config`, `/model`) التفويض.
-   يستخدم `allowFrom` و `groupAllowFrom` لتحديد تفويض الأمر.
-   يمكن للمرسلين المصرح لهم تشغيل أوامر التحكم حتى بدون الإشارة في المجموعات.

## مؤشرات الكتابة + إيصالات القراءة

-   **مؤشرات الكتابة**: يتم إرسالها تلقائيًا قبل وأثناء توليد الرد.
-   **إيصالات القراءة**: يتم التحكم فيها بواسطة `channels.bluebubbles.sendReadReceipts` (الافتراضي: `true`).
-   **مؤشرات الكتابة**: يرسل OpenClaw أحداث بدء الكتابة؛ يقوم BlueBubbles بمسح مؤشر الكتابة تلقائيًا عند الإرسال أو انتهاء المهلة (الإيقاف اليدوي عبر DELETE غير موثوق).

```json
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false, // تعطيل إيصالات القراءة
    },
  },
}
```

## الإجراءات المتقدمة

يدعم BlueBubbles إجراءات الرسائل المتقدمة عند تمكينها في التكوين:

```json
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true, // ردود الفعل (tapbacks) (الافتراضي: true)
        edit: true, // تحرير الرسائل المرسلة (macOS 13+، معطل على macOS 26 Tahoe)
        unsend: true, // إلغاء إرسال الرسائل (macOS 13+)
        reply: true, // خيط الردود بواسطة معرف الرسالة (GUID)
        sendWithEffect: true, // تأثيرات الرسائل (slam, loud, إلخ.)
        renameGroup: true, // إعادة تسمية دردشات المجموعات
        setGroupIcon: true, // تعيين أيقونة/صورة دردشة المجموعة (غير مستقر على macOS 26 Tahoe)
        addParticipant: true, // إضافة مشاركين إلى المجموعات
        removeParticipant: true, // إزالة مشاركين من المجموعات
        leaveGroup: true, // مغادرة دردشات المجموعات
        sendAttachment: true, // إرسال المرفقات/الوسائط
      },
    },
  },
}
```

الإجراءات المتاحة:

-   **react**: إضافة/إزالة تفاعلات رد الفعل (tapback) (`messageId`, `emoji`, `remove`)
-   **edit**: تحرير رسالة مرسلة (`messageId`, `text`)
-   **unsend**: إلغاء إرسال رسالة (`messageId`)
-   **reply**: الرد على رسالة محددة (`messageId`, `text`, `to`)
-   **sendWithEffect**: الإرسال بتأثير iMessage (`text`, `to`, `effectId`)
-   **renameGroup**: إعادة تسمية دردشة مجموعة (`chatGuid`, `displayName`)
-   **setGroupIcon**: تعيين أيقونة/صورة دردشة مجموعة (`chatGuid`, `media`) — غير مستقر على macOS 26 (Tahoe) (قد تعيد واجهة برمجة التطبيقات النجاح ولكن الأيقونة الجديدة لا تتم مزامنتها).
-   **addParticipant**: إضافة شخص إلى مجموعة (`chatGuid`, `address`)
-   **removeParticipant**: إزالة شخص من مجموعة (`chatGuid`, `address`)
-   **leaveGroup**: مغادرة دردشة مجموعة (`chatGuid`)
-   **sendAttachment**: إرسال الوسائط/الملفات (`to`, `buffer`, `filename`, `asVoice`)
    -   المذكرات الصوتية: عيّن `asVoice: true` مع صوت **MP3** أو **CAF** لإرساله كرسالة صوتية على iMessage. يقوم BlueBubbles بتحويل MP3 → CAF عند إرسال المذكرات الصوتية.

### معرفات الرسائل (قصيرة مقابل كاملة)

قد يعرض OpenClaw معرفات رسائل *قصيرة* (مثل `1`, `2`) لتوفير الرموز (tokens).

-   يمكن أن تكون `MessageSid` / `ReplyToId` معرفات قصيرة.
-   تحتوي `MessageSidFull` / `ReplyToIdFull` على معرفات كاملة من المزود.
-   المعرفات القصيرة موجودة في الذاكرة؛ يمكن أن تنتهي صلاحيتها عند إعادة التشغيل أو إخلاء ذاكرة التخزين المؤقت.
-   تقبل الإجراءات معرف `messageId` قصير أو كامل، ولكن المعرفات القصيرة ستؤدي إلى خطأ إذا لم تعد متاحة.

استخدم المعرفات الكاملة للأتمتة والتخزين الدائم:

-   القوالب: `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
-   السياق: `MessageSidFull` / `ReplyToIdFull` في حمولات البيانات الواردة

راجع [التكوين](../gateway/configuration.md) لمتغيرات القالب.

## البث المجزأ (Block Streaming)

التحكم فيما إذا كانت الردود تُرسل كرسالة واحدة أو يتم بثها مجزأة:

```json
{
  channels: {
    bluebubbles: {
      blockStreaming: true, // تمكين البث المجزأ (معطل افتراضيًا)
    },
  },
}
```

## الوسائط + الحدود

-   يتم تنزيل المرفقات الواردة وتخزينها في ذاكرة التخزين المؤقت للوسائط.
-   حد الوسائط عبر `channels.bluebubbles.mediaMaxMb` للوسائط الواردة والصادرة (الافتراضي: 8 ميجابايت).
-   يتم تقسيم النص الصادر إلى `channels.bluebubbles.textChunkLimit` (الافتراضي: 4000 حرف).

## مرجع التكوين

التكوين الكامل: [التكوين](../gateway/configuration.md) خيارات المزود:

-   `channels.bluebubbles.enabled`: تمكين/تعطيل القناة.
-   `channels.bluebubbles.serverUrl`: عنوان URL الأساسي لواجهة برمجة تطبيقات REST الخاصة بـ BlueBubbles.
-   `channels.bluebubbles.password`: كلمة مرور واجهة برمجة التطبيقات (API).
-   `channels.bluebubbles.webhookPath`: مسار نقطة نهاية webhook (الافتراضي: `/bluebubbles-webhook`).
-   `channels.bluebubbles.dmPolicy`: `pairing | allowlist | open | disabled` (الافتراضي: `pairing`).
-   `channels.bluebubbles.allowFrom`: القائمة المسموح بها للمراسلة المباشرة (المعرفات، عناوين البريد الإلكتروني، أرقام E.164، `chat_id:*`, `chat_guid:*`).
-   `channels.bluebubbles.groupPolicy`: `open | allowlist | disabled` (الافتراضي: `allowlist`).
-   `channels.bluebubbles.groupAllowFrom`: قائمة المرسلين المسموح لهم في المجموعات.
-   `channels.bluebubbles.groups`: التكوين لكل مجموعة (`requireMention`، إلخ.).
-   `channels.bluebubbles.sendReadReceipts`: إرسال إيصالات القراءة (الافتراضي: `true`).
-   `channels.bluebubbles.blockStreaming`: تمكين البث المجزأ (الافتراضي: `false`؛ مطلوب للردود المتدفقة).
-   `channels.bluebubbles.textChunkLimit`: حجم القطعة الصادرة بالأحرف (الافتراضي: 4000).
-   `channels.bluebubbles.chunkMode`: `length` (الافتراضي) يقسم فقط عند تجاوز `textChunkLimit`؛ `newline` يقسم على الأسطر الفارغة (حدود الفقرات) قبل التقسيم حسب الطول.
-   `channels.bluebubbles.mediaMaxMb`: حد الوسائط الواردة/الصادرة بالميجابايت (الافتراضي: 8).
-   `channels.bluebubbles.mediaLocalRoots`: قائمة مسموح بها صريحة للمسارات المحلية المطلقة المسموح بها لمسارات الوسائط المحلية الصادرة. يتم رفض إرسال المسارات المحلية افتراضيًا ما لم يتم تكوين هذا. تجاوز لكل حساب: `channels.bluebubbles.accounts..mediaLocalRoots`.
-   `channels.bluebubbles.historyLimit`: الحد الأقصى لرسائل المجموعة للسياق (0 يعطل).
-   `channels.bluebubbles.dmHistoryLimit`: حد سجل المراسلة المباشرة.
-   `channels.bluebubbles.actions`: تمكين/تعطيل إجراءات محددة.
-   `channels.bluebubbles.accounts`: تكوين متعدد الحسابات.

الخيارات العامة ذات الصلة:

-   `agents.list[].groupChat.mentionPatterns` (أو `messages.groupChat.mentionPatterns`).
-   `messages.responsePrefix`.

## العنونة / أهداف التسليم

تفضيل `chat_guid` للتوجيه المستقر:

-   `chat_guid:iMessage;-;+15555550123` (مفضل للمجموعات)
-   `chat_id:123`
-   `chat_identifier:...`
-   المعرفات المباشرة: `+15555550123`, `user@example.com`
    -   إذا لم يكن للمعرف المباشر دردشة مراسلة مباشرة (DM) موجودة، فسيقوم OpenClaw بإنشاء واحدة عبر `POST /api/v1/chat/new`. يتطلب هذا تمكين واجهة برمجة التطبيقات الخاصة (Private API) لـ BlueBubbles.

## الأمان

-   يتم مصادقة طلبات webhook بمقارنة معلمات الاستعلام أو رؤوس `guid`/`password` مقابل `channels.bluebubbles.password`. يتم قبول الطلبات من `localhost` أيضًا.
-   احتفظ بكلمة مرور واجهة برمجة التطبيقات ونقطة نهاية webhook سرية (عاملها مثل بيانات الاعتماد).
-   يعني ثقة localhost أن الوكيل العكسي على نفس المضيف يمكنه تجاوز كلمة المرور عن غير قصد. إذا كنت تستخدم وكيلًا للبوابة، فاطلب المصادقة عند الوكيل وقم بتكوين `gateway.trustedProxies`. راجع [أمان البوابة](../gateway/security.md#reverse-proxy-configuration).
-   قم بتمكين HTTPS + قواعد جدار الحماية على خادم BlueBubbles إذا كنت تعرضه خارج شبكتك المحلية (LAN).

## استكشاف الأخطاء وإصلاحها

-   إذا توقفت أحداث الكتابة/القراءة عن العمل، تحقق من سجلات webhook الخاصة بـ BlueBubbles وتأكد من تطابق مسار البوابة مع `channels.bluebubbles.webhookPath`.
-   تنتهي صلاحية رموز الاقتران بعد ساعة واحدة؛ استخدم `openclaw pairing list bluebubbles` و `openclaw pairing approve bluebubbles `.
-   تتطلب التفاعلات واجهة برمجة التطبيقات الخاصة بـ BlueBubbles (`POST /api/v1/message/react`)؛ تأكد من أن إصدار الخادم يعرضها.
-   يتطلب التحرير/الإلغاء إرسال macOS 13+ وإصدار خادم BlueBubbles متوافق. على macOS 26 (Tahoe)، التحرير معطل حاليًا بسبب تغييرات في واجهة برمجة التطبيقات الخاصة.
-   يمكن أن تكون تحديثات أيقونة المجموعة غير مستقرة على macOS 26 (Tahoe): قد تعيد واجهة برمجة التطبيقات النجاح ولكن الأيقونة الجديدة لا تتم مزامنتها.
-   يقوم OpenClaw بإخفاء الإجراءات المعطلة المعروفة تلقائيًا بناءً على إصدار macOS لخادم BlueBubbles. إذا ظهر التحرير على macOS 26 (Tahoe)، فقم بتعطيله يدويًا باستخدام `channels.bluebubbles.actions.edit=false`.
-   للحصول على معلومات الحالة/الصحة: `openclaw status --all` أو `openclaw status --deep`.

للرجوع إلى سير عمل القناة العام، راجع [القنوات](../channels.md) ودليل [المكونات الإضافية](../tools/plugin.md).

[قنوات الدردشة](../channels.md)[Discord](./discord.md)