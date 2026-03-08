title: "دليل وظائف Cron في OpenClaw: الجدولة والأتمتة"
description: "تعلم كيفية جدولة المهام المؤتمتة باستخدام وظائف cron في OpenClaw. قم بإعداد تذكيرات لمرة واحدة، ومهام متكررة، واختر بين وضعي التنفيذ الرئيسي أو المعزول."
keywords: ["openclaw cron", "جدولة الأتمتة", "وظائف cron", "جدولة البوابة", "دورة الوكيل المعزولة", "أحداث النظام", "المهام المتكررة", "تسليم الوظيفة"]
---

  الأتمتة

  
# وظائف Cron

> **Cron مقابل Heartbeat؟** راجع [Cron مقابل Heartbeat](./cron-vs-heartbeat.md) للحصول على إرشادات حول متى تستخدم كل منهما.

Cron هو برنامج الجدولة المدمج في البوابة. يقوم بتخزين الوظائف، وإيقاظ الوكيل في الوقت المناسب، ويمكنه اختياريًا تسليم الناتج مرة أخرى إلى محادثة. إذا كنت تريد *"تشغيل هذا كل صباح"* أو *"تنبيه الوكيل بعد 20 دقيقة"*، فإن cron هو الآلية. استكشاف الأخطاء وإصلاحها: [/automation/troubleshooting](./troubleshooting.md)

## ملخص سريع

-   يعمل Cron **داخل البوابة** (وليس داخل النموذج).
-   يتم تخزين الوظائف في `~/.openclaw/cron/` حتى لا تضيع الجداول عند إعادة التشغيل.
-   أسلوبان للتنفيذ:
    -   **الجلسة الرئيسية**: قم بإدراج حدث نظام في قائمة الانتظار، ثم تشغيله في نبض القلب التالي.
    -   **المعزول**: تشغيل دورة وكيل مخصصة في `cron:`، مع التسليم (الإعلان افتراضيًا أو لا شيء).
-   عمليات الإيقاظ أساسية: يمكن للوظيفة طلب "الإيقاظ الآن" مقابل "نبض القلب التالي".
-   نشر Webhook يكون لكل وظيفة عبر `delivery.mode = "webhook"` + `delivery.to = ""`.
-   يظل التراجع القديم موجودًا للوظائف المخزنة مع `notify: true` عندما يتم تعيين `cron.webhook`، قم بترحيل تلك الوظائف إلى وضع تسليم webhook.

## البدء السريع (قابل للتنفيذ)

قم بإنشاء تذكير لمرة واحدة، وتحقق من وجوده، وقم بتشغيله فورًا:

```bash
openclaw cron add \
  --name "Reminder" \
  --at "2026-02-01T16:00:00Z" \
  --session main \
  --system-event "Reminder: check the cron docs draft" \
  --wake now \
  --delete-after-run

openclaw cron list
openclaw cron run <job-id>
openclaw cron runs --id <job-id>
```

جدولة وظيفة معزولة متكررة مع التسليم:

```bash
openclaw cron add \
  --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize overnight updates." \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

## المعادلات باستدعاء الأداة (أداة cron الخاصة بالبوابة)

للأشكال JSON والأمثلة الأساسية، راجع [مخطط JSON لاستدعاءات الأدوات](./cron-jobs.md#json-schema-for-tool-calls).

## مكان تخزين وظائف cron

يتم تخزين وظائف cron على مضيف البوابة في `~/.openclaw/cron/jobs.json` افتراضيًا. تقوم البوابة بتحميل الملف إلى الذاكرة وإعادة كتابته عند إجراء تغييرات، لذا فإن التعديلات اليدوية آمنة فقط عند إيقاف البوابة. يُفضل استخدام `openclaw cron add/edit` أو واجهة برمجة تطبيقات استدعاء أداة cron لإجراء التغييرات.

## نظرة عامة للمبتدئين

فكر في وظيفة cron على أنها: **متى** يتم التشغيل + **ماذا** تفعل.

1.  **اختر جدولًا زمنيًا**
    -   تذكير لمرة واحدة → `schedule.kind = "at"` (CLI: `--at`)
    -   وظيفة متكررة → `schedule.kind = "every"` أو `schedule.kind = "cron"`
    -   إذا كان الطابع الزمني ISO الخاص بك يفتقر إلى منطقة زمنية، فسيتم التعامل معه على أنه **UTC**.
2.  **اختر مكان تشغيلها**
    -   `sessionTarget: "main"` → التشغيل خلال نبض القلب التالي مع سياق الجلسة الرئيسية.
    -   `sessionTarget: "isolated"` → تشغيل دورة وكيل مخصصة في `cron:`.
3.  **اختر الحمولة**
    -   الجلسة الرئيسية → `payload.kind = "systemEvent"`
    -   الجلسة المعزولة → `payload.kind = "agentTurn"`

اختياري: الوظائف لمرة واحدة (`schedule.kind = "at"`) تحذف بعد النجاح افتراضيًا. قم بتعيين `deleteAfterRun: false` للاحتفاظ بها (سيتم تعطيلها بعد النجاح).

## المفاهيم

### الوظائف

وظيفة cron هي سجل مخزن يحتوي على:

-   **جدول زمني** (متى يجب أن تعمل)،
-   **حمولة** (ماذا يجب أن تفعل)،
-   **وضع تسليم** اختياري (`announce`, `webhook`, أو `none`).
-   **ربط وكيل** اختياري (`agentId`): تشغيل الوظيفة تحت وكيل محدد؛ إذا كان مفقودًا أو غير معروف، تعود البوابة إلى الوكيل الافتراضي.

يتم التعرف على الوظائف بواسطة `jobId` ثابت (يستخدمه CLI/واجهات برمجة تطبيقات البوابة). في استدعاءات أدوات الوكيل، يكون `jobId` هو الأساسي؛ يتم قبول `id` القديم لأغراض التوافق. الوظائف لمرة واحدة تحذف تلقائيًا بعد النجاح افتراضيًا؛ قم بتعيين `deleteAfterRun: false` للاحتفاظ بها.

### الجداول الزمنية

يدعم Cron ثلاثة أنواع من الجداول الزمنية:

-   `at`: طابع زمني لمرة واحدة عبر `schedule.at` (ISO 8601).
-   `every`: فاصل زمني ثابت (مللي ثانية).
-   `cron`: تعبير cron مكون من 5 حقول (أو 6 حقول مع الثواني) مع منطقة زمنية IANA اختيارية.

تستخدم تعبيرات cron `croner`. إذا تم حذف المنطقة الزمنية، يتم استخدام المنطقة الزمنية المحلية لمضيف البوابة. لتقليل ذروة الحمل في بداية الساعة عبر العديد من البوابات، يطبق OpenClaw نافذة تأخير حتمية لكل وظيفة تصل إلى 5 دقائق لتعبيرات cron المتكررة في بداية الساعة (على سبيل المثال `0 * * * *`, `0 */2 * * *`). تظل تعبيرات الساعة الثابتة مثل `0 7 * * *` دقيقة. لأي جدول cron، يمكنك تعيين نافذة تأخير صريحة باستخدام `schedule.staggerMs` (`0` يحافظ على التوقيت الدقيق). اختصارات CLI:

-   `--stagger 30s` (أو `1m`, `5m`) لتعيين نافذة تأخير صريحة.
-   `--exact` لإجبار `staggerMs = 0`.

### التنفيذ الرئيسي مقابل المعزول

#### وظائف الجلسة الرئيسية (أحداث النظام)

تقوم الوظائف الرئيسية بإدراج حدث نظام في قائمة الانتظار واختياريًا إيقاظ مُشغل نبض القلب. يجب أن تستخدم `payload.kind = "systemEvent"`.

-   `wakeMode: "now"` (الافتراضي): يؤدي الحدث إلى تشغيل نبض قلب فوري.
-   `wakeMode: "next-heartbeat"`: ينتظر الحدث نبض القلب المجدول التالي.

هذا هو الأنسب عندما تريد مطالبة نبض القلب العادية + سياق الجلسة الرئيسية. راجع [نبض القلب](../gateway/heartbeat.md).

#### الوظائف المعزولة (جلسات cron مخصصة)

تعمل الوظائف المعزولة على تشغيل دورة وكيل مخصصة في الجلسة `cron:`. السلوكيات الرئيسية:

-   يبدأ المطالبة بـ `[cron: ]` لإمكانية التتبع.
-   يبدأ كل تشغيل **معرف جلسة جديد** (بدون حمل محادثة سابقة).
-   السلوك الافتراضي: إذا تم حذف `delivery`، تعلن الوظائف المعزولة عن ملخص (`delivery.mode = "announce"`).
-   يختار `delivery.mode` ما يحدث:
    -   `announce`: تسليم ملخص إلى القناة المستهدفة ونشر ملخص موجز في الجلسة الرئيسية.
    -   `webhook`: إرسال POST لحمولة الحدث المنتهي إلى `delivery.to` عندما يتضمن الحدث المنتهي ملخصًا.
    -   `none`: داخلي فقط (لا تسليم، ولا ملخص للجلسة الرئيسية).
-   يتحكم `wakeMode` في وقت نشر ملخص الجلسة الرئيسية:
    -   `now`: نبض قلب فوري.
    -   `next-heartbeat`: ينتظر نبض القلب المجدول التالي.

استخدم الوظائف المعزولة للمهام الصاخبة أو المتكررة أو "المهام الخلفية" التي لا ينبغي أن تملأ سجل المحادثة الرئيسي.

### أشكال الحمولة (ما يتم تشغيله)

يتم دعم نوعين من الحمولة:

-   `systemEvent`: للجلسة الرئيسية فقط، يتم توجيهها عبر مطالبة نبض القلب.
-   `agentTurn`: للجلسة المعزولة فقط، تشغل دورة وكيل مخصصة.

الحقول الشائعة في `agentTurn`:

-   `message`: المطالبة النصية المطلوبة.
-   `model` / `thinking`: تجاوزات اختيارية (انظر أدناه).
-   `timeoutSeconds`: تجاوز مهلة اختياري.
-   `lightContext`: وضع تمهيد خفيف الوزن اختياري للوظائف التي لا تحتاج إلى حقن ملفات تمهيد مساحة العمل.

تكوين التسليم:

-   `delivery.mode`: `none` | `announce` | `webhook`.
-   `delivery.channel`: `last` أو قناة محددة.
-   `delivery.to`: هدف خاص بالقناة (للإعلان) أو عنوان URL لـ webhook (وضع webhook).
-   `delivery.bestEffort`: تجنب فشل الوظيفة إذا فشل تسليم الإعلان.

يقوم تسليم الإعلان بقمع إرسالات أداة المراسلة للتشغيل؛ استخدم `delivery.channel`/`delivery.to` لاستهداف المحادثة بدلاً من ذلك. عندما يكون `delivery.mode = "none"`، لا يتم نشر أي ملخص في الجلسة الرئيسية. إذا تم حذف `delivery` للوظائف المعزولة، فإن OpenClaw يفرض `announce` افتراضيًا.

#### تدفق تسليم الإعلان

عندما يكون `delivery.mode = "announce"`، يقوم cron بالتسليم مباشرة عبر محولات القناة الصادرة. لا يتم تشغيل الوكيل الرئيسي لصياغة أو إعادة توجيه الرسالة. تفاصيل السلوك:

-   المحتوى: يستخدم التسليم الحمولات الصادرة للتشغيل المعزول (نص/وسائط) مع التقسيم والتنسيق الطبيعي للقناة.
-   لا يتم تسليم الردود الخاصة بنبض القلب فقط (`HEARTBEAT_OK` بدون محتوى حقيقي).
-   إذا كان التشغيل المعزول قد أرسل بالفعل رسالة إلى نفس الهدف عبر أداة الرسالة، يتم تخطي التسليم لتجنب التكرار.
-   يؤدي فقدان الأهداف أو عدم صحتها إلى فشل الوظيفة ما لم يكن `delivery.bestEffort = true`.
-   يتم نشر ملخص موجز في الجلسة الرئيسية فقط عندما يكون `delivery.mode = "announce"`.
-   يحترم ملخص الجلسة الرئيسية `wakeMode`: يؤدي `now` إلى إطلاق نبض قلب فوري ويؤدي `next-heartbeat` إلى انتظار نبض القلب المجدول التالي.

#### تدفق تسليم Webhook

عندما يكون `delivery.mode = "webhook"`، يقوم cron بنشر حمولة الحدث المنتهي إلى `delivery.to` عندما يتضمن الحدث المنتهي ملخصًا. تفاصيل السلوك:

-   يجب أن يكون النقطة الطرفية عنوان URL HTTP(S) صالحًا.
-   لا تتم محاولة تسليم القناة في وضع webhook.
-   لا يتم نشر ملخص للجلسة الرئيسية في وضع webhook.
-   إذا تم تعيين `cron.webhookToken`، فإن رأس المصادقة هو `Authorization: Bearer <cron.webhookToken>`.
-   التراجع القديم: لا تزال الوظائف المخزنة القديمة مع `notify: true` تنشر إلى `cron.webhook` (إذا تم تكوينها)، مع تحذير حتى تتمكن من الترحيل إلى `delivery.mode = "webhook"`.

### تجاوزات النموذج ومستوى التفكير

يمكن للوظائف المعزولة (`agentTurn`) تجاوز النموذج ومستوى التفكير:

-   `model`: سلسلة المزود/النموذج (مثل، `anthropic/claude-sonnet-4-20250514`) أو اسم مستعار (مثل، `opus`)
-   `thinking`: مستوى التفكير (`off`, `minimal`, `low`, `medium`, `high`, `xhigh`؛ نماذج GPT-5.2 + Codex فقط)

ملاحظة: يمكنك تعيين `model` على وظائف الجلسة الرئيسية أيضًا، لكنه يغير نموذج الجلسة الرئيسية المشترك. نوصي بتجاوزات النموذج للوظائف المعزولة فقط لتجنب تحولات السياق غير المتوقعة. أولوية الحل:

1.  تجاوز حمولة الوظيفة (الأعلى)
2.  الإعدادات الافتراضية الخاصة بالخطاف (مثل، `hooks.gmail.model`)
3.  الإعداد الافتراضي لتكوين الوكيل

### سياق التمهيد خفيف الوزن

يمكن للوظائف المعزولة (`agentTurn`) تعيين `lightContext: true` للتشغيل بسياق تمهيد خفيف الوزن.

-   استخدم هذا للمهام المجدولة التي لا تحتاج إلى حقن ملفات تمهيد مساحة العمل.
-   عمليًا، يعمل وقت التشغيل المضمن مع `bootstrapContextMode: "lightweight"`، والذي يحافظ على سياق تمهيد cron فارغًا عن قصد.
-   معادلات CLI: `openclaw cron add --light-context ...` و `openclaw cron edit --light-context`.

### التسليم (القناة + الهدف)

يمكن للوظائف المعزولة تسليم الناتج إلى قناة عبر تكوين `delivery` على المستوى الأعلى:

-   `delivery.mode`: `announce` (تسليم القناة)، `webhook` (HTTP POST)، أو `none`.
-   `delivery.channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (المكون الإضافي) / `signal` / `imessage` / `last`.
-   `delivery.to`: هدف المستلم الخاص بالقناة.

تسليم `announce` صالح فقط للوظائف المعزولة (`sessionTarget: "isolated"`). تسليم `webhook` صالح لكل من الوظائف الرئيسية والمعزولة. إذا تم حذف `delivery.channel` أو `delivery.to`، يمكن أن تعود cron إلى "المسار الأخير" للجلسة الرئيسية (آخر مكان رد فيه الوكيل). تذكير بتنسيق الهدف:

-   يجب أن تستخدم أهداف Slack/Discord/Mattermost (المكون الإضافي) بادئات صريحة (مثل `channel:`, `user:`) لتجنب الغموض.
-   يجب أن تستخدم مواضيع Telegram النموذج `:topic:` (انظر أدناه).

#### أهداف تسليم Telegram (المواضيع / سلاسل المنتدى)

يدعم Telegram مواضيع المنتدى عبر `message_thread_id`. لتسليم cron، يمكنك ترميز الموضوع/السلسلة في حقل `to`:

-   `-1001234567890` (معرف الدردشة فقط)
-   `-1001234567890:topic:123` (مفضل: علامة موضوع صريحة)
-   `-1001234567890:123` (اختصار: لاحقة رقمية)

يتم قبول الأهداف ذات البادئات مثل `telegram:...` / `telegram:group:...` أيضًا:

-   `telegram:group:-1001234567890:topic:123`

## مخطط JSON لاستدعاءات الأدوات

استخدم هذه الأشكال عند استدعاء أدوات `cron.*` الخاصة بالبوابة مباشرة (استدعاءات أداة الوكيل أو RPC). تقبل أعلام CLI فترات زمنية بشرية مثل `20m`، لكن يجب أن تستخدم استدعاءات الأداة سلسلة ISO 8601 لـ `schedule.at` ومللي ثانية لـ `schedule.everyMs`.

### معلمات cron.add

وظيفة لمرة واحدة، جلسة رئيسية (حدث نظام):

```json
{
  "name": "Reminder",
  "schedule": { "kind": "at", "at": "2026-02-01T16:00:00Z" },
  "sessionTarget": "main",
  "wakeMode": "now",
  "payload": { "kind": "systemEvent", "text": "Reminder text" },
  "deleteAfterRun": true
}
```

وظيفة متكررة، معزولة مع التسليم:

```json
{
  "name": "Morning brief",
  "schedule": { "kind": "cron", "expr": "0 7 * * *", "tz": "America/Los_Angeles" },
  "sessionTarget": "isolated",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "agentTurn",
    "message": "Summarize overnight updates.",
    "lightContext": true
  },
  "delivery": {
    "mode": "announce",
    "channel": "slack",
    "to": "channel:C1234567890",
    "bestEffort": true
  }
}
```

ملاحظات:

-   `schedule.kind`: `at` (`at`), `every` (`everyMs`), أو `cron` (`expr`, `tz` اختياري).
-   `schedule.at` يقبل ISO 8601 (المنطقة الزمنية اختيارية؛ يتم التعامل معها على أنها UTC عند حذفها).
-   `everyMs` هي مللي ثانية.
-   يجب أن يكون `sessionTarget` إما `"main"` أو `"isolated"` ويجب أن يتطابق مع `payload.kind`.
-   الحقول الاختيارية: `agentId`, `description`, `enabled`, `deleteAfterRun` (الافتراضي true لـ `at`), `delivery`.
-   `wakeMode` الافتراضي هو `"now"` عند حذفه.

### معلمات cron.update

```json
{
  "jobId": "job-123",
  "patch": {
    "enabled": false,
    "schedule": { "kind": "every", "everyMs": 3600000 }
  }
}
```

ملاحظات:

-   `jobId` هو الأساسي؛ يتم قبول `id` لأغراض التوافق.
-   استخدم `agentId: null` في التصحيح لمسح ربط الوكيل.

### معلمات cron.run و cron.remove

```json
{ "jobId": "job-123", "mode": "force" }
```

```json
{ "jobId": "job-123" }
```

## التخزين والتاريخ

-   مخزن الوظائف: `~/.openclaw/cron/jobs.json` (JSON تديره البوابة).
-   سجل التشغيل: `~/.openclaw/cron/runs/.jsonl` (JSONL، يتم التقليم التلقائي حسب الحجم وعدد الأسطر).
-   يتم تقليم جلسات تشغيل cron المعزولة في `sessions.json` بواسطة `cron.sessionRetention` (الافتراضي `24h`؛ قم بتعيين `false` لتعطيل).
-   تجاوز مسار التخزين: `cron.store` في التكوين.

## سياسة إعادة المحاولة

عند فشل وظيفة، يصنف OpenClaw الأخطاء على أنها **عابرة** (قابلة لإعادة المحاولة) أو **دائمة** (تعطيل فوري).

### أخطاء عابرة (يتم إعادة محاولتها)

-   حد المعدل (429، طلبات كثيرة جدًا، استنفاد الموارد)
-   حمل زائد للمزود (على سبيل المثال Anthropic `529 overloaded_error`، ملخصات التراجع عند الحمل الزائد)
-   أخطاء الشبكة (مهلة، ECONNRESET، فشل الجلب، مقبس)
-   أخطاء الخادم (5xx)
-   أخطاء متعلقة بـ Cloudflare

### أخطاء دائمة (لا إعادة محاولة)

-   فشل المصادقة (مفتاح API غير صالح، غير مصرح)
-   أخطاء التكوين أو التحقق
-   أخطاء أخرى غير عابرة

### السلوك الافتراضي (بدون تكوين)

**الوظائف لمرة واحدة (`schedule.kind: "at"`):**

-   عند حدوث خطأ عابر: إعادة المحاولة حتى 3 مرات مع تراجع أسي (30s → 1m → 5m).
-   عند حدوث خطأ دائم: تعطيل فوري.
-   عند النجاح أو التخطي: تعطيل (أو حذف إذا كان `deleteAfterRun: true`).

**الوظائف المتكررة (`cron` / `every`):**

-   عند أي خطأ: تطبيق تراجع أسي (30s → 1m → 5m → 15m → 60m) قبل التشغيل المجدول التالي.
-   تظل الوظيفة مفعلة؛ يتم إعادة تعيين التراجع بعد التشغيل الناجح التالي.

قم بتكوين `cron.retry` لتجاوز هذه الإعدادات الافتراضية (راجع [التكوين](./cron-jobs.md#configuration)).

## التكوين

```json
{
  cron: {
    enabled: true, // default true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1, // default 1
    // Optional: override retry policy for one-shot jobs
    retry: {
      maxAttempts: 3,
      backoffMs: [60000, 120000, 300000],
      retryOn: ["rate_limit", "overloaded", "network", "server_error"],
    },
    webhook: "https://example.invalid/legacy", // deprecated fallback for stored notify:true jobs
    webhookToken: "replace-with-dedicated-webhook-token", // optional bearer token for webhook mode
    sessionRetention: "24h", // duration string or false
    runLog: {
      maxBytes: "2mb", // default 2_000_000 bytes
      keepLines: 2000, // default 2000
    },
  },
}
```

سلوك تقليم سجل التشغيل:

-   `cron.runLog.maxBytes`: الحد الأقصى لحجم ملف سجل التشغيل قبل التقليم.
-   `cron.runLog.keepLines`: عند التقليم، احتفظ فقط بأحدث N سطر.
-   ينطبق كلاهما على ملفات `cron/runs/.jsonl`.

سلوك Webhook:

-   مفضل: قم بتعيين `delivery.mode: "webhook"` مع `delivery.to: "https://..."` لكل وظيفة.
-   يجب أن تكون عناوين URL لـ webhook صالحة `http://` أو `https://`.
-   عند النشر، تكون الحمولة هي JSON لحدث cron المنتهي.
-   إذا تم تعيين `cron.webhookToken`، فإن رأس المصادقة هو `Authorization: Bearer <cron.webhookToken>`.
-   إذا لم يتم تعيين `cron.webhookToken`، لا يتم إرسال رأس `Authorization`.
-   التراجع القديم: لا تزال الوظائف المخزنة القديمة مع `notify: true` تستخدم `cron.webhook` عند وجودها.

تعطيل cron بالكامل:

-   `cron.enabled: false` (تكوين)
-   `OPENCLAW_SKIP_CRON=1` (متغير بيئة)

## الصيانة

يحتوي Cron على مسارين مدمجين للصيانة: الاحتفاظ بجلسات التشغيل المعزولة وتقليم سجل التشغيل.

### الإعدادات الافتراضية

-   `cron.sessionRetention`: `24h` (قم بتعيين `false` لتعطيل تقليم جلسة التشغيل)
-   `cron.runLog.maxBytes`: `2_000_000` بايت
-   `cron.runLog.keepLines`: `2000`

### كيفية العمل

-   تقوم عمليات التشغيل المعزولة بإنشاء إدخالات جلسة (`...:cron::run:`) وملفات النصوص.
-   يقوم الحاصد بإزالة إدخالات جلسات التشغيل المنتهية الصلاحية الأقدم من `cron.sessionRetention`.
-   لجلسات التشغيل التي تمت إزالتها ولم يعد يتم الرجوع إليها بواسطة مخزن الجلسات، يقوم OpenClaw بأرشفة ملفات النصوص وتنظيف الأرشيفات المحذوفة القديمة في نفس نافذة الاحتفاظ.
-   بعد كل إلحاق للتشغيل، يتم فحص حجم `cron/runs/.jsonl`:
    -   إذا تجاوز حجم الملف `runLog.maxBytes`، يتم تقليمه إلى أحدث `runLog.keepLines` سطر.

### تحذير الأداء لمجدولي الحجم العالي

يمكن لإعدادات cron عالية التردد إنشاء بصمات كبيرة لجلسات التشغيل وسجلات التشغيل. الصيانة مدمجة، لكن الحدود الفضفاضة يمكن أن تخلق عبء I/O وعمل تنظيف يمكن تجنبه. ما يجب مراقبته:

-   نافذة `cron.sessionRetention` طويلة مع العديد من عمليات التشغيل المعزولة
-   `cron.runLog.keepLines` عالي مجتمع مع `runLog.maxBytes` كبير
-   العديد من الوظائف المتكررة الصاخبة التي تكتب في نفس `cron/runs/.jsonl`

ما يجب فعله:

-   احتفظ بـ `cron.sessionRetention` قصيرًا قدر ما تسمح به احتياجات التصحيح/التدقيق الخاصة بك
-   احتفظ بسجلات التشغيل محدودة بـ `runLog.maxBytes` و `runLog.keepLines` معتدلين
-   انقل الوظائف الخلفية الصاخبة إلى الوضع المعزول مع قواعد تسليم تتجنب الثرثرة غير الضرورية
-   راجع النمو بشكل دوري باستخدام `openclaw cron runs` وضبط الاحتفاظ قبل أن تصبح السجلات كبيرة

### أمثلة التخصيص

احتفظ بجلسات التشغيل لمدة أسبوع واسمح بسجلات تشغيل أكبر:

```json
{
  cron: {
    sessionRetention: "7d",
    runLog: {
      maxBytes: "10mb",
      keepLines: 5000,
    },
  },
}
```

عطل تقليم جلسات التشغيل المعزولة ولكن احتفظ بتقليم سجل التشغيل:

```json
{
  cron: {
    sessionRetention: false,
    runLog: {
      maxBytes: "5mb",
      keepLines: 3000,
    },
  },
}
```

ضبط لاستخدام cron عالي الحجم (مثال):

```json
{
  cron: {
    sessionRetention: "12h",
    runLog: {
      maxBytes: "3mb",
      keepLines: 1500,
    },
  },
}
```

## بدء سريع لـ CLI

تذكير لمرة واحدة (UTC ISO، حذف تلقائي بعد النجاح):

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

تذكير لمرة واحدة (جلسة رئيسية، إيقاظ فوري):

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

وظيفة معزولة متكررة (الإعلان إلى WhatsApp):

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

وظيفة cron متكررة مع تأخير صريح 30 ثانية:

```bash
openclaw cron add \
  --name "Minute watcher" \
  --cron "0 * * * * *" \
  --tz "UTC" \
  --stagger 30s \
  --session isolated \
  --message "Run minute watcher checks." \
  --announce
```

وظيفة معزولة متكررة (التسليم إلى موضوع Telegram):

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --announce \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

وظيفة معزولة مع تجاوز النموذج ومستوى التفكير:

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

اختيار الوكيل (إعدادات متعددة الوكلاء):

```bash
# تثبيت وظيفة على الوكيل "ops" (يعود إلى الافتراضي إذا كان هذا الوكيل مفقودًا)
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# تبديل أو مسح الوكيل في وظيفة موجودة
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
```

تشغيل يدوي (force هو الافتراضي، استخدم `--due` للتشغيل فقط عند الاستحقاق):

```bash
openclaw cron run <jobId>
openclaw cron run <jobId> --due
```

تحرير وظيفة موجودة (تصحيح الحقول):

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

إجبار وظيفة cron موجودة على التشغيل بالضبط حسب الجدول (بدون تأخير):

```bash
openclaw cron edit <jobId> --exact
```

سجل التشغيل:

```bash
openclaw cron runs --id <jobId> --limit 50
```

حدث نظام فوري دون إنشاء وظيفة:

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

## سطح واجهة برمجة تطبيقات البوابة

-   `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
-   `cron.run` (force أو due), `cron.runs` لأحداث النظام الفورية بدون وظيفة، استخدم [`openclaw system event`](../cli/system.md).

## استكشاف الأخطاء وإصلاحها

### "لا شيء يعمل"

-   تحقق من تفعيل cron: `cron.enabled` و `OPENCLAW_SKIP_CRON`.
-   تحقق من أن البوابة تعمل باستمرار (يعمل cron داخل عملية البوابة).
-   لجداول `cron`: تأكد من المنطقة