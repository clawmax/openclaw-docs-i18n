title: "دليل تكوين أدوات وكيل OpenClaw والمرجع"
description: "تعلم كيفية تكوين، السماح، منع، وإدارة أدوات وكيل OpenClaw من الدرجة الأولى للمتصفح، اللوحة القماشية، العقد، المهام المجدولة، والإضافات مع أمثلة مفصلة."
keywords: ["أدوات openclaw", "أدوات الوكيل", "تكوين الأداة", "ملفات تعريف الأدوات", "مجموعات الأدوات", "أداة المتصفح", "أداة التنفيذ", "إضافات الأدوات"]
---

  نظرة عامة

  
# الأدوات

يعرض OpenClaw **أدوات وكيل من الدرجة الأولى** للمتصفح، اللوحة القماشية، العقد، والمهام المجدولة. هذه تحل محل مهارات `openclaw-*` القديمة: الأدوات مكتوبة، لا تتطلب shell، ويجب أن يعتمد الوكيل عليها مباشرة.

## تعطيل الأدوات

يمكنك السماح/منع الأدوات عالميًا عبر `tools.allow` / `tools.deny` في `openclaw.json` (المنع يفوز). هذا يمنع إرسال الأدوات غير المسموح بها إلى مقدمي النماذج.

```json
{
  tools: { deny: ["browser"] },
}
```

ملاحظات:

-   المطابقة غير حساسة لحالة الأحرف.
-   الرموز البديلة `*` مدعومة (`"*"` تعني جميع الأدوات).
-   إذا كانت `tools.allow` تشير فقط إلى أسماء أدوات إضافات غير معروفة أو غير محملة، يسجل OpenClaw تحذيرًا ويتجاهل قائمة السماح حتى تبقى الأدوات الأساسية متاحة.

## ملفات تعريف الأدوات (قائمة السماح الأساسية)

يحدد `tools.profile` **قائمة السماح الأساسية للأدوات** قبل `tools.allow`/`tools.deny`. تجاوز لكل وكيل: `agents.list[].tools.profile`. الملفات الشخصية:

-   `minimal`: `session_status` فقط
-   `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
-   `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
-   `full`: لا قيود (نفس عدم التعيين)

مثال (المراسلة فقط افتراضيًا، السماح بأدوات Slack + Discord أيضًا):

```json
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

مثال (ملف تعريف البرمجة، ولكن منع exec/process في كل مكان):

```json
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

مثال (ملف تعريف برمجة عالمي، وكيل دعم للمراسلة فقط):

```json
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] },
      },
    ],
  },
}
```

## سياسة الأداة الخاصة بمقدم الخدمة

استخدم `tools.byProvider` **لتقييد** الأدوات لمقدمي خدمة محددين (أو `provider/model` واحد) دون تغيير الإعدادات الافتراضية العالمية. تجاوز لكل وكيل: `agents.list[].tools.byProvider`. يتم تطبيق هذا **بعد** ملف تعريف الأداة الأساسي و**قبل** قوائم السماح/المنع، لذا يمكنه فقط تضييق مجموعة الأدوات. تقبل مفاتيح المزود إما `provider` (مثل `google-antigravity`) أو `provider/model` (مثل `openai/gpt-5.2`). مثال (احتفظ بملف تعريف البرمجة العالمي، ولكن أدوات دنيا لـ Google Antigravity):

```json
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

مثال (قائمة السماح الخاصة بـ provider/model لنقطة نهاية غير مستقرة):

```json
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

مثال (تجاوز خاص بالوكيل لمزود واحد):

```json
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] },
          },
        },
      },
    ],
  },
}
```

## مجموعات الأدوات (اختصارات)

تدعم سياسات الأدوات (العالمية، الوكيل، sandbox) إدخالات `group:*` التي تتوسع إلى أدوات متعددة. استخدم هذه في `tools.allow` / `tools.deny`. المجموعات المتاحة:

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:web`: `web_search`, `web_fetch`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: جميع أدوات OpenClaw المدمجة (تستثني إضافات المزود)

مثال (السماح بأدوات الملفات + المتصفح فقط):

```json
{
  tools: {
    allow: ["group:fs", "browser"],
  },
}
```

## الإضافات + الأدوات

يمكن للإضافات تسجيل **أدوات إضافية** (وأوامر CLI) تتجاوز المجموعة الأساسية. راجع [الإضافات](./tools/plugin.md) للتثبيت + التكوين، و[المهارات](./tools/skills.md) لكيفية حقن إرشادات استخدام الأداة في المحفزات. بعض الإضافات تشحن مهاراتها الخاصة جنبًا إلى جنب مع الأدوات (على سبيل المثال، إضافة voice-call). أدوات الإضافات الاختيارية:

-   [Lobster](./tools/lobster.md): وقت تشغيل سير عمل مكتوب مع موافقات قابلة للاستئناف (يتطلب Lobster CLI على مضيف البوابة).
-   [LLM Task](./tools/llm-task.md): خطوة LLM بـ JSON فقط لإخراج سير عمل منظم (تحقق اختياري من المخطط).
-   [Diffs](./tools/diffs.md): عارض فروق للقراءة فقط وعارض ملفات PNG أو PDF للنصوص قبل/بعد أو تصحيحات موحدة.

## جرد الأدوات

### apply\_patch

تطبيق تصحيحات منظمة عبر ملف واحد أو أكثر. استخدم للتعديلات متعددة الأجزاء. تجريبي: تمكين عبر `tools.exec.applyPatch.enabled` (نماذج OpenAI فقط). `tools.exec.applyPatch.workspaceOnly` الافتراضي هو `true` (محتوى مساحة العمل). اضبطه على `false` فقط إذا كنت تريد عمدًا أن يكتب `apply_patch`/يحذف خارج دليل مساحة العمل.

### exec

تشغيل أوامر shell في مساحة العمل. المعلمات الأساسية:

-   `command` (مطلوب)
-   `yieldMs` (خلفية تلقائية بعد المهلة، الافتراضي 10000)
-   `background` (خلفية فورية)
-   `timeout` (ثواني؛ يقتل العملية إذا تم تجاوزها، الافتراضي 1800)
-   `elevated` (منطقي؛ تشغيل على المضيف إذا تم تمكين/السماح بالوضع المرتفع؛ يغير السلوك فقط عندما يكون الوكيل في sandbox)
-   `host` (`sandbox | gateway | node`)
-   `security` (`deny | allowlist | full`)
-   `ask` (`off | on-miss | always`)
-   `node` (معرف/اسم العقدة لـ `host=node`)
-   تحتاج إلى TTY حقيقي؟ اضبط `pty: true`.

ملاحظات:

-   يُرجع `status: "running"` مع `sessionId` عند الانتقال للخلفية.
-   استخدم `process` للاستطلاع/التسجيل/الكتابة/القتل/مسح جلسات الخلفية.
-   إذا تم منع `process`، يعمل `exec` بشكل متزامن ويتجاهل `yieldMs`/`background`.
-   `elevated` محمي بـ `tools.elevated` بالإضافة إلى أي تجاوز `agents.list[].tools.elevated` (يجب أن يسمح كلاهما) وهو اسم مستعار لـ `host=gateway` + `security=full`.
-   `elevated` يغير السلوك فقط عندما يكون الوكيل في sandbox (وإلا فهو لا يعمل).
-   يمكن لـ `host=node` استهداف تطبيق مرافق macOS أو مضيف عقدة بدون واجهة (`openclaw node run`).
-   موافقات البوابة/العقدة وقوائم السماح: [موافقات التنفيذ](./tools/exec-approvals.md).

### process

إدارة جلسات exec في الخلفية. الإجراءات الأساسية:

-   `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

ملاحظات:

-   يُرجع `poll` مخرجات جديدة وحالة خروج عند الاكتمال.
-   يدعم `log` `offset`/`limit` قائم على الأسطر (احذف `offset` لأخذ آخر N سطر).
-   `process` محدد لكل وكيل؛ الجلسات من وكلاء آخرين غير مرئية.

### loop-detection (حواجز حماية حلقة استدعاء الأداة)

يتتبع OpenClaw تاريخ استدعاءات الأداة الحديثة ويمنع أو يحذر عند اكتشاف حلقات متكررة بدون تقدم. تمكين بـ `tools.loopDetection.enabled: true` (الافتراضي هو `false`).

```json
{
  tools: {
    loopDetection: {
      enabled: true,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      historySize: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

-   `genericRepeat`: نمط استدعاء نفس الأداة + نفس المعلمات المتكرر.
-   `knownPollNoProgress`: تكرار أدوات شبيهة بـ poll مع مخرجات متطابقة.
-   `pingPong`: أنماط `A/B/A/B` متناوبة بدون تقدم.
-   تجاوز لكل وكيل: `agents.list[].tools.loopDetection`.

### web\_search

البحث على الويب باستخدام Perplexity أو Brave أو Gemini أو Grok أو Kimi. المعلمات الأساسية:

-   `query` (مطلوب)
-   `count` (1–10؛ الافتراضي من `tools.web.search.maxResults`)

ملاحظات:

-   يتطلب مفتاح API لمزود الخدمة المختار (موصى به: `openclaw configure --section web`).
-   تمكين عبر `tools.web.search.enabled`.
-   يتم تخزين الردود مؤقتًا (افتراضي 15 دقيقة).
-   راجع [أدوات الويب](./tools/web.md) للإعداد.

### web\_fetch

جلب واستخراج محتوى قابل للقراءة من عنوان URL (HTML → markdown/نص). المعلمات الأساسية:

-   `url` (مطلوب)
-   `extractMode` (`markdown` | `text`)
-   `maxChars` (اقتطاع الصفحات الطويلة)

ملاحظات:

-   تمكين عبر `tools.web.fetch.enabled`.
-   `maxChars` مقيد بـ `tools.web.fetch.maxCharsCap` (الافتراضي 50000).
-   يتم تخزين الردود مؤقتًا (افتراضي 15 دقيقة).
-   للمواقع كثيرة الـ JS، يُفضل استخدام أداة المتصفح.
-   راجع [أدوات الويب](./tools/web.md) للإعداد.
-   راجع [Firecrawl](./tools/firecrawl.md) للبديل الاختياري المضاد للروبوتات.

### browser

التحكم في المتصفح المخصص الذي يديره OpenClaw. الإجراءات الأساسية:

-   `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
-   `snapshot` (aria/ai)
-   `screenshot` (تُرجع كتلة صورة + `MEDIA:`)
-   `act` (إجراءات واجهة المستخدم: click/type/press/hover/drag/select/fill/resize/wait/evaluate)
-   `navigate`, `console`, `pdf`, `upload`, `dialog`

إدارة الملفات الشخصية:

-   `profiles` — سرد جميع ملفات تعريف المتصفح مع الحالة
-   `create-profile` — إنشاء ملف تعريف جديد مع منفذ مخصص تلقائيًا (أو `cdpUrl`)
-   `delete-profile` — إيقاف المتصفح، حذف بيانات المستخدم، الإزالة من التكوين (محلي فقط)
-   `reset-profile` — قتل عملية يتيمة على منفذ الملف الشخصي (محلي فقط)

المعلمات الشائعة:

-   `profile` (اختياري؛ الافتراضي `browser.defaultProfile`)
-   `target` (`sandbox` | `host` | `node`)
-   `node` (اختياري؛ يختار معرف/اسم عقدة محدد) ملاحظات:
-   يتطلب `browser.enabled=true` (الافتراضي هو `true`؛ اضبط `false` لتعطيل).
-   جميع الإجراءات تقبل معلمة `profile` اختيارية لدعم مثيلات متعددة.
-   عند حذف `profile`، يستخدم `browser.defaultProfile` (الافتراضي "chrome").
-   أسماء الملفات الشخصية: أحرف أبجدية رقمية صغيرة + شرطات فقط (بحد أقصى 64 حرفًا).
-   نطاق المنفذ: 18800-18899 (~100 ملف تعريف كحد أقصى).
-   ملفات التعريف البعيدة هي إرفاق فقط (بدون start/stop/reset).
-   إذا كانت عقدة قادرة على المتصفح متصلة، قد تقوم الأداة بتوجيه تلقائي إليها (ما لم تثبت `target`).
-   `snapshot` الافتراضي هو `ai` عند تثبيت Playwright؛ استخدم `aria` لشجرة إمكانية الوصول.
-   يدعم `snapshot` أيضًا خيارات لقطة الدور (`interactive`, `compact`, `depth`, `selector`) والتي تُرجع مراجع مثل `e12`.
-   يتطلب `act` `ref` من `snapshot` (`12` رقمي من لقطات AI، أو `e12` من لقطات الدور)؛ استخدم `evaluate` للاحتياجات النادرة لمحدد CSS.
-   تجنب `act` → `wait` افتراضيًا؛ استخدمه فقط في حالات استثنائية (لا توجد حالة واجهة مستخدم موثوقة للانتظار عليها).
-   يمكن لـ `upload` تمرير `ref` اختياري للنقر التلقائي بعد التجهيز.
-   يدعم `upload` أيضًا `inputRef` (مرجع aria) أو `element` (محدد CSS) لتعيين `` مباشرة.

### canvas

قيادة لوحة العقدة Canvas (present, eval, snapshot, A2UI). الإجراءات الأساسية:

-   `present`, `hide`, `navigate`, `eval`
-   `snapshot` (تُرجع كتلة صورة + `MEDIA:`)
-   `a2ui_push`, `a2ui_reset`

ملاحظات:

-   يستخدم `node.invoke` للبوابة تحت الغطاء.
-   إذا لم يتم توفير `node`، تختار الأداة افتراضيًا (عقدة واحدة متصلة أو عقدة mac محلية).
-   A2UI هو v0.8 فقط (لا `createSurface`)؛ يرفض CLI v0.9 JSONL مع أخطاء الأسطر.
-   اختبار سريع: `openclaw nodes canvas a2ui push --node  --text "Hello from A2UI"`.

### nodes

اكتشاف واستهداف العقد المقترنة؛ إرسال إشعارات؛ التقاط الكاميرا/الشاشة. الإجراءات الأساسية:

-   `status`, `describe`
-   `pending`, `approve`, `reject` (الاقتران)
-   `notify` (macOS `system.notify`)
-   `run` (macOS `system.run`)
-   `camera_list`, `camera_snap`, `camera_clip`, `screen_record`
-   `location_get`, `notifications_list`, `notifications_action`
-   `device_status`, `device_info`, `device_permissions`, `device_health`

ملاحظات:

-   تتطلب أوامر الكاميرا/الشاشة أن يكون تطبيق العقدة في المقدمة.
-   تُرجع الصور كتل صور + `MEDIA:`.
-   تُرجع مقاطع الفيديو `FILE:` (mp4).
-   يُرجع الموقع حمولة JSON (lat/lon/accuracy/timestamp).
-   معلمات `run`: `command` مصفوفة argv؛ اختياري `cwd`, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording`.

مثال (`run`):

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

### image

تحليل صورة باستخدام نموذج الصورة المُكون. المعلمات الأساسية:

-   `image` (مسار أو عنوان URL مطلوب)
-   `prompt` (اختياري؛ الافتراضي "Describe the image.")
-   `model` (تجاوز اختياري)
-   `maxBytesMb` (حد حجم اختياري)

ملاحظات:

-   متاح فقط عند تكوين `agents.defaults.imageModel` (أولي أو احتياطي)، أو عندما يمكن استنتاج نموذج صورة ضمني من نموذجك الافتراضي + المصادقة المُكونة (اقتران بأفضل جهد).
-   يستخدم نموذج الصورة مباشرة (مستقل عن نموذج الدردشة الرئيسي).

### pdf

تحليل مستند PDF واحد أو أكثر. للسلوك الكامل، الحدود، التكوين، والأمثلة، راجع [أداة PDF](./tools/pdf.md).

### message

إرسال رسائل وإجراءات قنوات عبر Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams. الإجراءات الأساسية:

-   `send` (نص + وسائط اختيارية؛ يدعم MS Teams أيضًا `card` لبطاقات Adaptive)
-   `poll` (استطلاعات WhatsApp/Discord/MS Teams)
-   `react` / `reactions` / `read` / `edit` / `delete`
-   `pin` / `unpin` / `list-pins`
-   `permissions`
-   `thread-create` / `thread-list` / `thread-reply`
-   `search`
-   `sticker`
-   `member-info` / `role-info`
-   `emoji-list` / `emoji-upload` / `sticker-upload`
-   `role-add` / `role-remove`
-   `channel-info` / `channel-list`
-   `voice-status`
-   `event-list` / `event-create`
-   `timeout` / `kick` / `ban`

ملاحظات:

-   `send` يوجه WhatsApp عبر البوابة؛ القنوات الأخرى تذهب مباشرة.
-   يستخدم `poll` البوابة لـ WhatsApp و MS Teams؛ استطلاعات Discord تذهب مباشرة.
-   عندما يكون استدعاء أداة الرسالة مرتبطًا بجلسة دردشة نشطة، تكون عمليات الإرسال مقيدة بهدف تلك الجلسة لتجنب تسريبات عبر السياقات.

### cron

إدارة وظائف cron المجدولة للبوابة والاستيقاظ. الإجراءات الأساسية:

-   `status`, `list`
-   `add`, `update`, `remove`, `run`, `runs`
-   `wake` (إدراج حدث نظام + نبض قلب فوري اختياري)

ملاحظات:

-   يتوقع `add` كائن وظيفة cron كامل (نفس مخطط `cron.add` RPC).
-   يستخدم `update` `{ jobId, patch }` (يُقبل `id` للتطابق).

### gateway

إعادة تشغيل أو تطبيق تحديثات على عملية البوابة قيد التشغيل (في المكان). الإجراءات الأساسية:

-   `restart` (يصرح + يرسل `SIGUSR1` لإعادة التشغيل داخل العملية؛ `openclaw gateway` restart في المكان)
-   `config.schema.lookup` (فحص مسار تكوين واحد في كل مرة دون تحميل المخطط الكامل في سياق المحفز)
-   `config.get`
-   `config.apply` (التحقق + كتابة التكوين + إعادة التشغيل + استيقاظ)
-   `config.patch` (دمج تحديث جزئي + إعادة التشغيل + استيقاظ)
-   `update.run` (تشغيل تحديث + إعادة التشغيل + استيقاظ)

ملاحظات:

-   يتوقع `config.schema.lookup` مسار تكوين مستهدف مثل `gateway.auth` أو `agents.list.*.heartbeat`.
-   قد تتضمن المسارات معرفات إضافات مفصولة بشرطة مائلة عند معالجة `plugins.entries.`، على سبيل المثال `plugins.entries.pack/one.config`.
-   استخدم `delayMs` (الافتراضي 2000) لتجنب مقاطعة رد قيد التنفيذ.
-   يبقى `config.schema` متاحًا لتدفقات واجهة التحكم الداخلية ولا يتم كشفه من خلال أداة `gateway` للوكيل.
-   `restart` مفعل افتراضيًا؛ اضبط `commands.restart: false` لتعطيله.

### sessions\_list / sessions\_history / sessions\_send / sessions\_spawn / session\_status

سرد الجلسات، فحص سجل المحادثة، أو الإرسال إلى جلسة أخرى. المعلمات الأساسية:

-   `sessions_list`: `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = لا شيء)
-   `sessions_history`: `sessionKey` (أو `sessionId`), `limit?`, `includeTools?`
-   `sessions_send`: `sessionKey` (أو `sessionId`), `message`, `timeoutSeconds?` (0 = إطلاق ونسيان)
-   `sessions_spawn`: `task`, `label?`, `runtime?`, `agentId?`, `model?`, `thinking?`, `cwd?`, `runTimeoutSeconds?`, `thread?`, `mode?`, `cleanup?`, `sandbox?`, `streamTo?`, `attachments?`, `attachAs?`
-   `session_status`: `sessionKey?` (الافتراضي الحالي؛ يقبل `sessionId`), `model?` (`default` يمسح التجاوز)

ملاحظات:

-   `main` هو مفتاح الدردشة المباشرة الأساسي؛ global/unknown مخفية.
-   `messageLimit > 0` يجلب آخر N رسالة لكل جلسة (يتم تصفية رسائل الأداة).
-   يتم التحكم في استهداف الجلسة بواسطة `tools.sessions.visibility` (الافتراضي `tree`: الجلسة الحالية + جلسات الوكيل الفرعي المنبثقة). إذا قمت بتشغيل وكيل مشترك لمستخدمين متعددين، ففكر في تعيين `tools.sessions.visibility: "self"` لمنع التصفح عبر الجلسات.
-   ينتظر `sessions_send` الاكتمال النهائي عندما يكون `timeoutSeconds > 0`.
-   يحدث التسليم/الإعلان بعد الاكتمال وهو بأفضل جهد؛ `status: "ok"` تؤكد انتهاء تشغيل الوكيل، وليس تسليم الإعلان.
-   يدعم `sessions_spawn` `runtime: "subagent" | "acp"` (`subagent` افتراضي). لسلوك وقت تشغيل ACP، راجع [وكلاء ACP](./tools/acp-agents.md).
-   لوقت تشغيل ACP، `streamTo: "parent"` يوجه ملخصات تقدم التشغيل الأولي مرة أخرى إلى جلسة الطالب كأحداث نظام بدلاً من التسليم المباشر للطفل.
-   يبدأ `sessions_spawn` تشغيل وكيل فرعي وينشر رد إعلان مرة أخرى إلى دردشة الطالب.
    -   يدعم وضع اللقطة الواحدة (`mode: "run"`) والوضع المستمر المرتبط بموضوع (`mode: "session"` مع `thread: true`).
    -   إذا كان `thread: true` وتم حذف `mode`، يكون الوضع الافتراضي `session`.
    -   يتطلب `mode: "session"` `thread: true`.
    -   إذا تم حذف `runTimeoutSeconds`، يستخدم OpenClaw `agents.defaults.subagents.runTimeoutSeconds` عند التعيين؛ وإلا المهلة الافتراضية `0` (لا مهلة).
    -   تعتمد تدفقات Discord المرتبطة بالموضوع على `session.threadBindings.*` و `channels.discord.threadBindings.*`.
    -   يتضمن تنسيق الرد `Status` و `Result` وإحصائيات مضغوطة.
    -   `Result` هو نص إكمال المساعد؛ إذا كان مفقودًا، يتم استخدام أحدث `toolResult` كاحتياطي.
-   ترسل عمليات الإنشاء ذات وضع الإكمال اليدوي مباشرة أولاً، مع قائمة انتظار احتياطية وإعادة محاولة على حالات الفشل العابرة (`status: "ok"` يعني انتهاء التشغيل، وليس تسليم الإعلان).
-   يدعم `sessions_spawn` مرفقات الملفات المضمنة لوقت تشغيل الوكيل الفرعي فقط (يرفضها ACP). لكل مرفق `name` و `content` و `encoding` اختياري (`utf8` أو `base64`) و `mimeType`. يتم تجسيد الملفات في مساحة عمل الطفل في `.openclaw/attachments//` مع ملف بيانات وصفية `.manifest.json`. تُرجع الأداة إيصالاً يحتوي على `count` و `totalBytes` و `sha256` لكل ملف و `relDir`. يتم حذف محتوى المرفق تلقائيًا من استمرارية المحادثة.
    -   تكوين الحدود عبر `tools.sessions_spawn.attachments` (`enabled`, `maxTotalBytes`, `maxFiles`, `maxFileBytes`, `retainOnSessionKeep`).
    -   `attachAs.mountPath` هو تلميح محجوز لتطبيقات التركيب المستقبلية.
-   `sessions_spawn` غير معطل ويُرجع `status: "accepted"` على الفور.
-   قد تتضمن استجابات ACP `streamTo: "parent"` `streamLogPath` (نطاق الجلسة `*.acp-stream.jsonl`) لتتبع سجل التقدم.
-   يقوم `sessions_send` بتشغيل ping‑pong رد‑رجوع (رد `REPLY_SKIP` للتوقف؛ الحد الأقصى للدورات عبر `session.agentToAgent.maxPingPongTurns`, 0–5).
-   بعد ping‑pong، يقوم الوكيل الهدف بتشغيل **خطوة إعلان**؛ رد `ANNOUNCE_SKIP` لقمع الإعلان.
-   تقييد sandbox: عندما تكون الجلسة الحالية في sandbox و `agents.defaults.sandbox.sessionToolsVisibility: "spawned"`، يحد OpenClaw `tools.sessions.visibility` إلى `tree`.

### agents\_list

سرد معرفات الوكلاء التي قد تستهدفها الجلسة الحالية بـ `sessions_spawn`. ملاحظات:

-   النتيجة مقيدة بقوائم السماح لكل وكيل (`agents.list[].subagents.allowAgents`).
-   عند تكوين `["*"]`، تتضمن الأداة جميع الوكلاء المُكونة وتضع علامة `allowAny: true`.

## المعلمات (المشتركة)

أدوات مدعومة بالبوابة (`canvas`, `nodes`, `cron`):

-   `gatewayUrl` (افتراضي `ws://127.0.0.1:18789`)
-   `gatewayToken` (إذا تم تمكين المصادقة)
-   `timeoutMs`

ملاحظة: عند تعيين `gatewayUrl`، قم بتضمين `gatewayToken` صراحة. لا ترث الأدوات بيانات اعتماد التكوين أو البيئة للتجاوزات، وغياب بيانات الاعتماد الصريحة هو خطأ. أداة المتصفح:

-   `profile` (اختياري؛ الافتراضي `browser.defaultProfile`)
-   `target` (`sandbox` | `host` | `node`)
-   `node` (اختياري؛ تثبيت معرف/اسم عقدة محدد)

## تدفقات الوكيل الموصى بها

أتمتة المتصفح:

1.  `browser` → `status` / `start`
2.  `snapshot` (ai أو aria)
3.  `act` (click/type/press)
4.  `screenshot` إذا كنت بحاجة إلى تأكيد مرئي

عرض اللوحة القماشية:

1.  `canvas` → `present`
2.  `a2ui_push` (اختياري)
3.  `snapshot`

استهداف العقدة:

1.  `nodes` → `status`
2.  `describe` على العقدة المختارة
3.  `notify` / `run` / `camera_snap` / `screen_record`

## السلامة

-   تجنب `system.run` المباشر؛ استخدم `nodes` → `run` فقط بموافقة مستخدم صريحة.
-   احترم موافقة المستخدم لالتقاط الكاميرا/الشاشة.
-   استخدم `status/describe` لضمان الأذونات قبل استدعاء أوامر الوسائط.

## كيف يتم تقديم الأدوات للوكيل

يتم عرض الأدوات في قناتين متوازيتين:

1.  **نص المحفز النظامي**: قائمة مقروءة للإنسان + إرشادات.
2.  **مخطط الأداة**: تعريفات الدوال المنظمة المرسلة إلى واجهة برمجة تطبيقات النموذج.

هذا يعني أن الوكيل يرى كلًا من "ما هي الأدوات الموجودة" و"كيفية استدعائها." إذا لم تظهر أداة في محفز النظام أو المخطط، لا يمكن للنموذج استدعاؤها.

[أداة apply\_patch](./tools/apply-patch.md)