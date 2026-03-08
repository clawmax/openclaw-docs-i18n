title: "دليل أوامر الشرطة المائلة لـ OpenClaw للمهارات والإعدادات"
description: "تعلم كيفية استخدام أوامر الشرطة المائلة والتوجيهات في OpenClaw للمهارات، والإعدادات، وإدارة الجلسات. يتضمن قائمة الأوامر وخيارات الإعداد."
keywords: ["أوامر الشرطة المائلة openclaw", "أوامر المهارات", "توجيهات البوابة", "إعداد روبوت المحادثة", "إدارة الجلسات", "أوامر bash", "أوامر discord telegram", "إعداد القائمة المسموح بها"]
---

  المهارات

  
# أوامر الشرطة المائلة

يتم التعامل مع الأوامر بواسطة البوابة. يجب إرسال معظم الأوامر كرسالة **منفردة** تبدأ بـ `/`. يستخدم أمر bash للمضيف فقط `! ` (مع `/bash ` كاسم مستعار). هناك نظامان مرتبطان:

-   **الأوامر**: رسائل منفردة `/...`.
-   **التوجيهات**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
    -   يتم إزالة التوجيهات من الرسالة قبل أن يراها النموذج.
    -   في رسائل الدردشة العادية (وليست توجيهات فقط)، يتم التعامل معها على أنها "تلميحات مضمنة" ولا تستمر في إعدادات الجلسة.
    -   في الرسائل التي تحتوي على توجيهات فقط (تحتوي الرسالة على توجيهات فقط)، تستمر في الجلسة وتستجيب بإقرار.
    -   يتم تطبيق التوجيهات فقط للمرسلين **المصرح لهم**. إذا تم تعيين `commands.allowFrom`، فهي القائمة المسموح بها الوحيدة المستخدمة؛ وإلا فإن التصريح يأتي من قوائم السماح/الاقتران الخاصة بالقناة بالإضافة إلى `commands.useAccessGroups`. يرى المرسلون غير المصرح لهم أن التوجيهات تُعامل كنص عادي.

هناك أيضًا بعض **الاختصارات المضمنة** (للمرسلين المسموح لهم/المصرح لهم فقط): `/help`, `/commands`, `/status`, `/whoami` (`/id`). تعمل هذه الأوامر فورًا، ويتم إزالتها قبل أن يرى النموذج الرسالة، ويستمر النص المتبقي في التدفق العادي.

## الإعدادات

```json
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

-   `commands.text` (الافتراضي `true`) يمكّن تحليل `/...` في رسائل الدردشة.
    -   على الأسطح التي لا تحتوي على أوامر أصلية (WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams)، تعمل الأوامر النصية حتى إذا قمت بتعيين هذا إلى `false`.
-   `commands.native` (الافتراضي `"auto"`) يسجل الأوامر الأصلية.
    -   تلقائي: مفعل لـ Discord/Telegram؛ معطل لـ Slack (حتى تقوم بإضافة أوامر الشرطة المائلة)؛ يتم تجاهله لمزودي الخدمة الذين لا يدعمون الأوامر الأصلية.
    -   عيّن `channels.discord.commands.native`، أو `channels.telegram.commands.native`، أو `channels.slack.commands.native` لتجاوز الإعداد لكل مزود (منطقي أو `"auto"`).
    -   `false` يمسح الأوامر المسجلة مسبقًا على Discord/Telegram عند بدء التشغيل. يتم إدارة أوامر Slack في تطبيق Slack ولا تتم إزالتها تلقائيًا.
-   `commands.nativeSkills` (الافتراضي `"auto"`) يسجل أوامر **المهارات** بشكل أصلي عند دعمها.
    -   تلقائي: مفعل لـ Discord/Telegram؛ معطل لـ Slack (يتطلب Slack إنشاء أمر شرطة مائلة لكل مهارة).
    -   عيّن `channels.discord.commands.nativeSkills`، أو `channels.telegram.commands.nativeSkills`، أو `channels.slack.commands.nativeSkills` لتجاوز الإعداد لكل مزود (منطقي أو `"auto"`).
-   `commands.bash` (الافتراضي `false`) يمكّن `! ` لتشغيل أوامر مضيف shell (`/bash ` هو اسم مستعار؛ يتطلب قوائم السماح `tools.elevated`).
-   `commands.bashForegroundMs` (الافتراضي `2000`) يتحكم في المدة التي ينتظرها bash قبل التبديل إلى وضع الخلفية (`0` ينتقل إلى الخلفية فورًا).
-   `commands.config` (الافتراضي `false`) يمكّن `/config` (يقرأ ويكتب `openclaw.json`).
-   `commands.debug` (الافتراضي `false`) يمكّن `/debug` (تجاوزات وقت التشغيل فقط).
-   `commands.allowFrom` (اختياري) يحدد قائمة السماح لكل مزود لتصريح الأوامر. عند التكوين، فهي مصدر التصريح الوحيد للأوامر والتوجيهات (يتم تجاهل قوائم السماح/الاقتران الخاصة بالقناة و `commands.useAccessGroups`). استخدم `"*"` كافتراضي عام؛ تتجاوز المفاتيح الخاصة بالمزود هذا الإعداد.
-   `commands.useAccessGroups` (الافتراضي `true`) يفرض قوائم السماح/السياسات للأوامر عندما لا يتم تعيين `commands.allowFrom`.

## قائمة الأوامر

نصي + أصلي (عند التمكين):

-   `/help`
-   `/commands`
-   `/skill  [input]` (تشغيل مهارة بالاسم)
-   `/status` (عرض الحالة الحالية؛ يتضمن استخدام/حصة المزود للنموذج الحالي عند التوفر)
-   `/allowlist` (عرض/إضافة/إزالة مدخلات القائمة المسموح بها)
-   `/approve  allow-once|allow-always|deny` (حل مطالبات الموافقة على التنفيذ)
-   `/context [list|detail|json]` (شرح "السياق"؛ `detail` يعرض حجم لكل ملف + لكل أداة + لكل مهارة + موجه النظام)
-   `/export-session [path]` (اسم مستعار: `/export`) (تصدير الجلسة الحالية إلى HTML مع موجه النظام الكامل)
-   `/whoami` (عرض معرف المرسل الخاص بك؛ اسم مستعار: `/id`)
-   `/session idle <duration|off>` (إدارة إلغاء التركيز التلقائي بسبب الخمول للارتباطات المواضيعية المركزة)
-   `/session max-age <duration|off>` (إدارة إلغاء التركيز التلقائي بسبب الحد الأقصى للعمر للارتباطات المواضيعية المركزة)
-   `/subagents list|kill|log|info|send|steer|spawn` (فحص، أو التحكم، أو إنشاء عمليات الوكلاء الفرعية للجلسة الحالية)
-   `/acp spawn|cancel|steer|close|status|set-mode|set|cwd|permissions|timeout|model|reset-options|doctor|install|sessions` (فحص والتحكم في جلسات وقت تشغيل ACP)
-   `/agents` (سرد الوكلاء المرتبطين بالموضوع لهذه الجلسة)
-   `/focus ` (Discord: ربط هذا الموضوع، أو موضوع جديد، بهدف جلسة/وكيل فرعي)
-   `/unfocus` (Discord: إزالة الارتباط الحالي للموضوع)
-   `/kill <id|#|all>` (إجهاض واحد أو جميع الوكلاء الفرعية قيد التشغيل لهذه الجلسة فورًا؛ بدون رسالة تأكيد)
-   `/steer <id|#> ` (توجيه وكيل فرعي قيد التشغيل فورًا: أثناء التشغيل عند الإمكان، وإلا إجهاض العمل الحالي وإعادة البدء على رسالة التوجيه)
-   `/tell <id|#> ` (اسم مستعار لـ `/steer`)
-   `/config show|get|set|unset` (استمرار الإعدادات على القرص، للمالك فقط؛ يتطلب `commands.config: true`)
-   `/debug show|set|unset|reset` (تجاوزات وقت التشغيل، للمالك فقط؛ يتطلب `commands.debug: true`)
-   `/usage off|tokens|full|cost` (تذييل الاستخدام لكل استجابة أو ملخص التكلفة المحلية)
-   `/tts off|always|inbound|tagged|status|provider|limit|summary|audio` (التحكم في TTS؛ انظر [/tts](../tts.md))
    -   Discord: الأمر الأصلي هو `/voice` (Discord يحتفظ بـ `/tts`)؛ النص `/tts` لا يزال يعمل.
-   `/stop`
-   `/restart`
-   `/dock-telegram` (اسم مستعار: `/dock_telegram`) (تبديل الردود إلى Telegram)
-   `/dock-discord` (اسم مستعار: `/dock_discord`) (تبديل الردود إلى Discord)
-   `/dock-slack` (اسم مستعار: `/dock_slack`) (تبديل الردود إلى Slack)
-   `/activation mention|always` (المجموعات فقط)
-   `/send on|off|inherit` (للمالك فقط)
-   `/reset` أو `/new [model]` (تلميح نموذج اختياري؛ الباقي يتم تمريره)
-   `/think <off|minimal|low|medium|high|xhigh>` (خيارات ديناميكية حسب النموذج/المزود؛ أسماء مستعارة: `/thinking`, `/t`)
-   `/verbose on|full|off` (اسم مستعار: `/v`)
-   `/reasoning on|off|stream` (اسم مستعار: `/reason`؛ عند التمكين، يرسل رسالة منفصلة مسبوقة بـ `Reasoning:`؛ `stream` = مسودة Telegram فقط)
-   `/elevated on|off|ask|full` (اسم مستعار: `/elev`؛ `full` يتخطى موافقات التنفيذ)
-   `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=` (إرسال `/exec` لعرض الحالي)
-   `/model ` (اسم مستعار: `/models`؛ أو `/` من `agents.defaults.models.*.alias`)
-   `/queue ` (بالإضافة إلى خيارات مثل `debounce:2s cap:25 drop:summarize`؛ أرسل `/queue` لرؤية الإعدادات الحالية)
-   `/bash ` (للمضيف فقط؛ اسم مستعار لـ `! `؛ يتطلب `commands.bash: true` + قوائم السماح `tools.elevated`)

نصي فقط:

-   `/compact [instructions]` (انظر [/concepts/compaction](../concepts/compaction.md))
-   `! ` (للمضيف فقط؛ واحد في كل مرة؛ استخدم `!poll` + `!stop` للوظائف طويلة الأمد)
-   `!poll` (التحقق من الإخراج / الحالة؛ يقبل `sessionId` اختياريًا؛ `/bash poll` يعمل أيضًا)
-   `!stop` (إيقاف مهمة bash قيد التشغيل؛ يقبل `sessionId` اختياريًا؛ `/bash stop` يعمل أيضًا)

ملاحظات:

-   تقبل الأوامر `:` اختياريًا بين الأمر والوسائط (مثال: `/think: high`, `/send: on`, `/help:`).
-   `/new ` يقبل اسمًا مستعارًا للنموذج، أو `provider/model`، أو اسم مزود (مطابقة تقريبية)؛ إذا لم توجد مطابقة، يتم التعامل مع النص كجسم الرسالة.
-   للحصول على تفصيل كامل لاستخدام المزود، استخدم `openclaw status --usage`.
-   `/allowlist add|remove` يتطلب `commands.config=true` ويلتزم بـ `configWrites` الخاصة بالقناة.
-   `/usage` يتحكم في تذييل الاستخدام لكل استجابة؛ `/usage cost` يطبع ملخص التكلفة المحلية من سجلات جلسة OpenClaw.
-   `/restart` مفعل افتراضيًا؛ عيّن `commands.restart: false` لتعطيله.
-   أمر Discord الأصلي فقط: `/vc join|leave|status` يتحكم في قنوات الصوت (يتطلب `channels.discord.voice` والأوامر الأصلية؛ غير متاح كنص).
-   أوامر ربط مواضيع Discord (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`) تتطلب تمكين ارتباطات المواضيع الفعالة (`session.threadBindings.enabled` و/أو `channels.discord.threadBindings.enabled`).
-   مرجع أمر ACP وسلوك وقت التشغيل: [وكلاء ACP](./acp-agents.md).
-   `/verbose` مخصص لتصحيح الأخطاء وزيادة الوضوح؛ ابقيه **معطلًا** في الاستخدام العادي.
-   لا تزال تُعرض ملخصات فشل الأداة عند الاقتضاء، ولكن يتم تضمين نص الفشل التفصيلي فقط عندما يكون `/verbose` `on` أو `full`.
-   `/reasoning` (و `/verbose`) محفوفان بالمخاطر في إعدادات المجموعة: قد يكشفان عن التفكير الداخلي أو إخراج الأداة الذي لم تنوي الكشف عنه. يُفضل تركهما معطلين، خاصة في محادثات المجموعة.
-   **مسار سريع:** يتم التعامل مع الرسائل التي تحتوي على أوامر فقط من المرسلين المسموح لهم فورًا (يتجاوز قائمة الانتظار + النموذج).
-   **بوابة الإشارة للمجموعة:** تتجاوز الرسائل التي تحتوي على أوامر فقط من المرسلين المسموح لهم متطلبات الإشارة.
-   **الاختصارات المضمنة (للمرسلين المسموح لهم فقط):** تعمل بعض الأوامر أيضًا عند تضمينها في رسالة عادية ويتم إزالتها قبل أن يرى النموذج النص المتبقي.
    -   مثال: `hey /status` يُطلق رد حالة، ويستمر النص المتبقي في التدفق العادي.
-   حاليًا: `/help`, `/commands`, `/status`, `/whoami` (`/id`).
-   يتم تجاهل الرسائل التي تحتوي على أوامر فقط من مرسلين غير مصرح لهم بصمت، ويتم التعامل مع الرموز `/...` المضمنة كنص عادي.
-   **أوامر المهارات:** يتم عرض المهارات `user-invocable` كأوامر شرطة مائلة. يتم تطهير الأسماء إلى `a-z0-9_` (بحد أقصى 32 حرفًا)؛ تحصل التصادمات على لاحقات رقمية (مثال: `_2`).
    -   `/skill  [input]` يشغل مهارة بالاسم (مفيد عندما تمنع حدود الأمر الأصلي وجود أوامر لكل مهارة).
    -   افتراضيًا، يتم إعادة توجيه أوامر المهارات إلى النموذج كطلب عادي.
    -   قد تعلن المهارات اختياريًا عن `command-dispatch: tool` لتوجيه الأمر مباشرة إلى أداة (حتمي، بدون نموذج).
    -   مثال: `/prose` (ملحق OpenProse) — انظر [OpenProse](../prose.md).
-   **وسائط الأمر الأصلي:** يستخدم Discord الإكمال التلقائي للخيارات الديناميكية (وقوائم الأزرار عند حذف الوسائط المطلوبة). يعرض Telegram و Slack قائمة أزرار عندما يدعم الأمر خيارات وتحذف الوسيط.

## أسطح الاستخدام (ما يظهر أين)

-   **استخدام/حصة المزود** (مثال: "Claude 80% left") يظهر في `/status` لمزود النموذج الحالي عند تمكين تتبع الاستخدام.
-   **الرموز/التكلفة لكل استجابة** يتم التحكم فيها بواسطة `/usage off|tokens|full` (مُلحق بالردود العادية).
-   `/model status` يتعلق **بالنماذج/المصادقة/نقاط النهاية**، وليس الاستخدام.

## اختيار النموذج (/model)

يتم تنفيذ `/model` كتوجيه. أمثلة:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

ملاحظات:

-   `/model` و `/model list` يعرضان منتقيًا مضغوطًا ومرقمًا (عائلة النموذج + المزودون المتاحون).
-   على Discord، يفتح `/model` و `/models` منتقيًا تفاعليًا مع قوائم منسدلة للمزود والنموذج بالإضافة إلى خطوة إرسال.
-   `/model <#>` يختار من ذلك المنتقي (ويفضل المزود الحالي عند الإمكان).
-   `/model status` يعرض العرض التفصيلي، بما في ذلك نقطة نهاية المزود المُعدة (`baseUrl`) ووضع API (`api`) عند التوفر.

## تجاوزات التصحيح

يسمح لك `/debug` بتعيين تجاوزات إعداد **وقت التشغيل فقط** (ذاكرة، وليس قرص). للمالك فقط. معطل افتراضيًا؛ قم بتمكينه بـ `commands.debug: true`. أمثلة:

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

ملاحظات:

-   يتم تطبيق التجاوزات فورًا على قراءات الإعداد الجديدة، ولكنها **لا** تكتب في `openclaw.json`.
-   استخدم `/debug reset` لمسح جميع التجاوزات والعودة إلى الإعداد الموجود على القرص.

## تحديثات الإعدادات

يكتب `/config` في إعداداتك الموجودة على القرص (`openclaw.json`). للمالك فقط. معطل افتراضيًا؛ قم بتمكينه بـ `commands.config: true`. أمثلة:

```bash
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

ملاحظات:

-   يتم التحقق من صحة الإعداد قبل الكتابة؛ يتم رفض التغييرات غير الصالحة.
-   تستمر تحديثات `/config` عبر عمليات إعادة التشغيل.

## ملاحظات السطح

-   تعمل **الأوامر النصية** في جلسة الدردشة العادية (تشارك الرسائل المباشرة `main`، وللمجموعات جلسة خاصة بها).
-   تستخدم **الأوامر الأصلية** جلسات معزولة:
    -   Discord: `agent::discord:slash:`
    -   Slack: `agent::slack:slash:` (بادئة قابلة للتكوين عبر `channels.slack.slashCommand.sessionPrefix`)
    -   Telegram: `telegram:slash:` (يستهدف جلسة الدردشة عبر `CommandTargetSessionKey`)
-   يستهدف **`/stop`** جلسة الدردشة النشطة حتى يتمكن من إجهاض التشغيل الحالي.
-   **Slack:** لا يزال `channels.slack.slashCommand` مدعومًا لأمر واحد من نوع `/openclaw`. إذا قمت بتمكين `commands.native`، يجب عليك إنشاء أمر Slack شرطة مائلة واحد لكل أمر مدمج (بنفس أسماء `/help`). يتم تسليم قوائم وسائط الأمر لـ Slack كأزرار Block Kit عابرة.
    -   استثناء Slack الأصلي: سجل `/agentstatus` (وليس `/status`) لأن Slack يحتفظ بـ `/status`. لا يزال النص `/status` يعمل في رسائل Slack.

[إنشاء المهارات](./creating-skills.md)[المهارات](./skills.md)