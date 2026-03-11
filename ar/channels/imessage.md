

  منصات المراسلة

  
# iMessage

> **⚠️** بالنسبة لنشرات iMessage الجديدة، استخدم [BlueBubbles](./bluebubbles.md). تكامل `imsg` قديم وقد يتم إزالته في إصدار مستقبلي.

 الحالة: تكامل CLI خارجي قديم. تقوم البوابة بتشغيل `imsg rpc` والتواصل عبر JSON-RPC على stdio (بدون برنامج خفي/منفذ منفصل). 

## الإعداد السريع

## المتطلبات والأذونات (macOS)

-   يجب أن تكون تطبيقات Messages مسجلة الدخول على جهاز Mac الذي يشغل `imsg`.
-   مطلوب وصول القرص الكامل لسياق العملية الذي يشغل OpenClaw/`imsg` (الوصول إلى قاعدة بيانات Messages).
-   مطلوب إذن الأتمتة لإرسال الرسائل عبر Messages.app.

> **💡** يتم منح الأذونات لكل سياق عملية. إذا كانت البوابة تعمل بدون واجهة مستخدم (LaunchAgent/SSH)، قم بتشغيل أمر تفاعلي لمرة واحدة في نفس سياق العملية لتفعيل المطالبات:
> 
> نسخ
> 
> ```
> imsg chats --limit 1
> # أو
> imsg send <handle> "test"
> ```

## التحكم في الوصول والتوجيه

`channels.imessage.dmPolicy` تتحكم في الرسائل المباشرة:

-   `pairing` (افتراضي)
-   `allowlist`
-   `open` (يتطلب تضمين `"*"` في `allowFrom`)
-   `disabled`

حقل القائمة المسموح بها: `channels.imessage.allowFrom`. يمكن أن تكون إدخالات القائمة المسموح بها مقابض أو أهداف دردشة (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).

`channels.imessage.groupPolicy` تتحكم في معالجة المجموعات:

-   `allowlist` (افتراضي عند التكوين)
-   `open`
-   `disabled`

قائمة المرسلين المسموح لهم في المجموعات: `channels.imessage.groupAllowFrom`. التراجع في وقت التشغيل: إذا لم يتم تعيين `groupAllowFrom`، فإن فحوصات مرسل مجموعة iMessage تتراجع إلى `allowFrom` عند توفرها. ملاحظة وقت التشغيل: إذا كان `channels.imessage` مفقودًا تمامًا، فإن وقت التشغيل يتراجع إلى `groupPolicy="allowlist"` ويسجل تحذيرًا (حتى إذا تم تعيين `channels.defaults.groupPolicy`). التحكم في الإشارات للمجموعات:

-   لا تحتوي iMessage على بيانات وصفية أصلية للإشارات
-   يستخدم كشف الإشارات أنماط regex (`agents.list[].groupChat.mentionPatterns`، تراجع `messages.groupChat.mentionPatterns`)
-   بدون أنماط مكونة، لا يمكن فرض التحكم في الإشارات

يمكن للأوامر التحكمية من المرسلين المصرح لهم تجاوز التحكم في الإشارات في المجموعات.

-   تستخدم الرسائل المباشرة التوجيه المباشر؛ المجموعات تستخدم توجيه المجموعة.
-   مع الإعداد الافتراضي `session.dmScope=main`، تندمج رسائل iMessage المباشرة في الجلسة الرئيسية للوكيل.
-   جلسات المجموعات معزولة (`agent::imessage:group:<chat_id>`).
-   يتم توجيه الردود مرة أخرى إلى iMessage باستخدام بيانات قناة/هدف الرسالة الأصلية.

سلوك المحادثة الشبيهة بالمجموعة:يمكن أن تصل بعض محادثات iMessage متعددة المشاركين مع `is_group=false`. إذا تم تكوين `chat_id` هذا صراحةً تحت `channels.imessage.groups`، تعامل OpenClaw معه كحركة مرور جماعية (تحكم جماعي + عزل جلسة المجموعة).

## أنماط النشر

استخدم Apple ID ومستخدم macOS مخصصين حتى تكون حركة مرور البوت معزولة عن ملف Messages الشخصي الخاص بك.التدفق النموذجي:

1.  إنشاء/تسجيل دخول مستخدم macOS مخصص.
2.  تسجيل الدخول إلى Messages باستخدام Apple ID الخاص بالبوت في ذلك المستخدم.
3.  تثبيت `imsg` في ذلك المستخدم.
4.  إنشاء غلاف SSH حتى يتمكن OpenClaw من تشغيل `imsg` في سياق ذلك المستخدم.
5.  توجيه `channels.imessage.accounts..cliPath` و `.dbPath` إلى ملف تعريف ذلك المستخدم.

قد تتطلب المرة الأولى موافقات واجهة المستخدم الرسومية (الأتمتة + وصول القرص الكامل) في جلسة مستخدم البوت تلك.

الطوبولوجيا الشائعة:

-   تعمل البوابة على Linux/VM
-   يعمل iMessage + `imsg` على جهاز Mac في شبكة Tailnet الخاصة بك
-   يستخدم غلاف `cliPath` SSH لتشغيل `imsg`
-   يمكّن `remoteHost` جلب المرفقات عبر SCP

مثال:

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

استخدم مفاتيح SSH حتى يكون كل من SSH و SCP غير تفاعليين. تأكد من أن مفتاح المضيف موثوق به أولاً (على سبيل المثال `ssh bot@mac-mini.tailnet-1234.ts.net`) حتى يتم ملء `known_hosts`.

يدعم iMessage تكوينًا لكل حساب تحت `channels.imessage.accounts`.يمكن لكل حساب تجاوز حقول مثل `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, إعدادات السجل، وقوائم الجذور المسموح بها للمرفقات.

## الوسائط، التقسيم، وأهداف التسليم

-   استيراد المرفقات الواردة اختياري: `channels.imessage.includeAttachments`
-   يمكن جلب مسارات المرفقات البعيدة عبر SCP عند تعيين `remoteHost`
-   يجب أن تتطابق مسارات المرفقات مع الجذور المسموح بها:
    -   `channels.imessage.attachmentRoots` (محلي)
    -   `channels.imessage.remoteAttachmentRoots` (وضع SCP البعيد)
    -   نمط الجذر الافتراضي: `/Users/*/Library/Messages/Attachments`
-   يستخدم SCP فحصًا صارمًا لمفتاح المضيف (`StrictHostKeyChecking=yes`)
-   يستخدم حجم الوسائط الصادرة `channels.imessage.mediaMaxMb` (افتراضي 16 ميجابايت)

-   حد تقسيم النص: `channels.imessage.textChunkLimit` (افتراضي 4000)
-   وضع التقسيم: `channels.imessage.chunkMode`
    -   `length` (افتراضي)
    -   `newline` (تقسيم حسب الفقرات أولاً)

الأهداف الصريحة المفضلة:

-   `chat_id:123` (مُوصى به للتوجيه المستقر)
-   `chat_guid:...`
-   `chat_identifier:...`

أيضًا مدعومة أهداف المقابض:

-   `imessage:+1555...`
-   `sms:+1555...`
-   `user@example.com`

```bash
imsg chats --limit 20
```

## كتابات التكوين

يسمح iMessage بكتابات التكوين التي يبدأها القناة افتراضيًا (لـ `/config set|unset` عندما يكون `commands.config: true`). تعطيل:

```json
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## استكشاف الأخطاء وإصلاحها

التحقق من صحة الملف الثنائي ودعم RPC:

```bash
imsg rpc --help
openclaw channels status --probe
```

إذا أبلغ الفحص أن RPC غير مدعوم، قم بتحديث `imsg`.

تحقق من:

-   `channels.imessage.dmPolicy`
-   `channels.imessage.allowFrom`
-   موافقات الاقتران (`openclaw pairing list imessage`)

تحقق من:

-   `channels.imessage.groupPolicy`
-   `channels.imessage.groupAllowFrom`
-   سلوك القائمة المسموح بها `channels.imessage.groups`
-   تكوين نمط الإشارة (`agents.list[].groupChat.mentionPatterns`)

تحقق من:

-   `channels.imessage.remoteHost`
-   `channels.imessage.remoteAttachmentRoots`
-   مصادقة مفتاح SSH/SCP من مضيف البوابة
-   وجود مفتاح المضيف في `~/.ssh/known_hosts` على مضيف البوابة
-   قابلية قراءة المسار البعيد على جهاز Mac الذي يشغل Messages

أعد التشغيل في طرفية واجهة مستخدم رسومية تفاعلية في نفس سياق المستخدم/الجلسة ووافق على المطالبات:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

تأكد من منح وصول القرص الكامل + الأتمتة لسياق العملية التي تشغل OpenClaw/`imsg`.

## مؤشرات مرجع التكوين

-   [مرجع التكوين - iMessage](../gateway/configuration-reference.md#imessage)
-   [تكوين البوابة](../gateway/configuration.md)
-   [الاقتران](./pairing.md)
-   [BlueBubbles](./bluebubbles.md)

[Google Chat](./googlechat.md)[IRC](./irc.md)

---