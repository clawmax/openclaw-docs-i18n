title: "دليل هوات OpenClaw للأتمتة للإجراءات المعتمدة على الأحداث"
description: "تعلم كيفية استخدام هوات أتمتة OpenClaw لتحريك إجراءات مخصصة بناءً على أوامر وأحداث الوكيل. تمكين وإدارة وإنشاء هوات لذاكرة الجلسة، والتسجيل، وسير العمل."
keywords: ["هوات openclaw", "هوات الأتمتة", "أتمتة معتمدة على الأحداث", "أحداث دورة حياة الوكيل", "معالج الهوك", "ذاكرة الجلسة", "مسجل الأوامر", "تكامل ويبهوك"]
---

  الأتمتة

  
# الهوات (Hooks)

توفر الهوات نظامًا قابلاً للتوسيع يعمل بالأحداث لأتمتة الإجراءات استجابة لأوامر وأحداث الوكيل. يتم اكتشاف الهوات تلقائيًا من الدلائل ويمكن إدارتها عبر أوامر CLI، بشكل مشابه لكيفية عمل المهارات في OpenClaw.

## التوجيه الأولي

الهوات هي نصوص برمجية صغيرة تعمل عند حدوث شيء ما. هناك نوعان:

-   **الهوات (هذه الصفحة)**: تعمل داخل Gateway عند تشغيل أحداث الوكيل، مثل `/new`، `/reset`، `/stop`، أو أحداث دورة الحياة.
-   **ويبهوكس**: ويبهوكات HTTP خارجية تسمح لأنظمة أخرى بتشغيل عمل في OpenClaw. راجع [هوات ويبهوك](./webhook.md) أو استخدم `openclaw webhooks` لأوامر مساعد Gmail.

يمكن أيضًا تجميع الهوات داخل الإضافات؛ راجع [الإضافات](../tools/plugin.md#plugin-hooks). الاستخدامات الشائعة:

-   حفظ لقطة للذاكرة عند إعادة تعيين جلسة
-   الاحتفاظ بسجل تدقيق للأوامر للاستكشاف أو الامتثال
-   تشغيل أتمتة متابعة عند بدء جلسة أو انتهائها
-   كتابة ملفات في مساحة عمل الوكيل أو استدعاء واجهات برمجة تطبيقات خارجية عند تشغيل الأحداث

إذا كنت تستطيع كتابة دالة TypeScript صغيرة، يمكنك كتابة هوك. يتم اكتشاف الهوات تلقائيًا، ويمكنك تمكينها أو تعطيلها عبر CLI.

## نظرة عامة

يتيح نظام الهوات لك:

-   حفظ سياق الجلسة في الذاكرة عند إصدار `/new`
-   تسجيل جميع الأوامر للتدقيق
-   تشغيل أتمتات مخصصة عند أحداث دورة حياة الوكيل
-   توسيع سلوك OpenClaw دون تعديل الكود الأساسي

## البدء

### الهوات المضمنة

يأتي OpenClaw مع أربعة هوات مضمنة يتم اكتشافها تلقائيًا:

-   **💾 session-memory**: يحفظ سياق الجلسة في مساحة عمل وكيلك (الافتراضي `~/.openclaw/workspace/memory/`) عند إصدار `/new`
-   **📎 bootstrap-extra-files**: يحقن ملفات تمهيد إضافية لمساحة العمل من أنماط المسار/glob المُهيأة أثناء `agent:bootstrap`
-   **📝 command-logger**: يسجل جميع أحداث الأوامر في `~/.openclaw/logs/commands.log`
-   **🚀 boot-md**: يشغل `BOOT.md` عند بدء تشغيل البوابة (يتطلب تمكين الهوات الداخلية)

عرض الهوات المتاحة:

```bash
openclaw hooks list
```

تمكين هوك:

```bash
openclaw hooks enable session-memory
```

فحص حالة الهوك:

```bash
openclaw hooks check
```

الحصول على معلومات مفصلة:

```bash
openclaw hooks info session-memory
```

### الإعداد الأولي

خلال الإعداد الأولي (`openclaw onboard`)، سيتم مطالبتك بتمكين الهوات الموصى بها. يكتشف المعالج تلقائيًا الهوات المؤهلة ويعرضها للاختيار.

## اكتشاف الهوات

يتم اكتشاف الهوات تلقائيًا من ثلاثة أدلة (بترتيب الأولوية):

1.  **هوات مساحة العمل**: `/hooks/` (لكل وكيل، أعلى أولوية)
2.  **الهوات المدارة**: `~/.openclaw/hooks/` (مثبتة من قبل المستخدم، مشتركة عبر مساحات العمل)
3.  **الهوات المضمنة**: `/dist/hooks/bundled/` (مرفقة مع OpenClaw)

يمكن أن تكون أدلة الهوات المدارة إما **هوك واحد** أو **حزمة هوات** (دليل حزمة). كل هوك هو دليل يحتوي على:

```
my-hook/
├── HOOK.md          # بيانات وصفية + توثيق
└── handler.ts       # تنفيذ المعالج
```

## حزم الهوات (npm/أرشيفات)

حزم الهوات هي حزم npm قياسية تصدر واحدًا أو أكثر من الهوات عبر `openclaw.hooks` في `package.json`. قم بتثبيتها باستخدام:

```bash
openclaw hooks install <path-or-spec>
```

مواصفات npm هي للسجل فقط (اسم الحزمة + إصدار محدد اختياري أو علامة توزيع). يتم رفض مواصفات Git/URL/الملف ونطاقات semver. تبقى المواصفات العارية و `@latest` على المسار المستقر. إذا حل npm أيًا من هذين إلى إصدار أولي، يتوقف OpenClaw ويطلب منك الموافقة صراحةً بعلامة إصدار أولي مثل `@beta`/`@rc` أو إصدار أولي محدد. مثال `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

يشير كل إدخال إلى دليل هوك يحتوي على `HOOK.md` و `handler.ts` (أو `index.ts`). يمكن أن تشحن حزم الهوات تبعيات؛ سيتم تثبيتها تحت `~/.openclaw/hooks/`. يجب أن يبقى كل إدخال `openclaw.hooks` داخل دليل الحزمة بعد حل الارتباطات الرمزية؛ يتم رفض الإدخالات التي تتجاوز ذلك. ملاحظة أمنية: `openclaw hooks install` يثبت التبعيات باستخدام `npm install --ignore-scripts` (بدون نصوص برمجية لدورة الحياة). حافظ على أشجار تبعيات حزم الهوات "نقية JS/TS" وتجنب الحزم التي تعتمد على عمليات بناء `postinstall`.

## بنية الهوك

### تنسيق HOOK.md

يحتوي ملف `HOOK.md` على بيانات وصفية في YAML frontmatter بالإضافة إلى توثيق Markdown:

```
---
name: my-hook
description: "وصف قصير لما يفعله هذا الهوك"
homepage: https://docs.openclaw.ai/automation/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# هوكي

التوثيق التفصيلي يذهب هنا...

## ما يفعله

- يستمع لأوامر `/new`
- ينفذ بعض الإجراءات
- يسجل النتيجة

## المتطلبات

- يجب تثبيت Node.js

## التهيئة

لا حاجة للتهيئة.
```

### حقول البيانات الوصفية

يدعم كائن `metadata.openclaw`:

-   **`emoji`**: إيموجي للعرض في CLI (مثال: `"💾"`)
-   **`events`**: مصفوفة الأحداث للاستماع إليها (مثال: `["command:new", "command:reset"]`)
-   **`export`**: التصدير المسمى للاستخدام (الافتراضي `"default"`)
-   **`homepage`**: عنوان URL للتوثيق
-   **`requires`**: متطلبات اختيارية
    -   **`bins`**: الملفات الثنائية المطلوبة على PATH (مثال: `["git", "node"]`)
    -   **`anyBins`**: يجب أن يكون واحد على الأقل من هذه الملفات الثنائية موجودًا
    -   **`env`**: متغيرات البيئة المطلوبة
    -   **`config`**: مسارات التهيئة المطلوبة (مثال: `["workspace.dir"]`)
    -   **`os`**: المنصات المطلوبة (مثال: `["darwin", "linux"]`)
-   **`always``: تجاوز فحوصات الأهلية (قيمة منطقية)
-   **`install`**: طرق التثبيت (للهوات المضمنة: `[{"id":"bundled","kind":"bundled"}]`)

### تنفيذ المعالج

يصدر ملف `handler.ts` دالة `HookHandler`:

```typescript
const myHandler = async (event) => {
  // يتم التشغيل فقط عند أمر 'new'
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] تم تشغيل أمر جديد`);
  console.log(`  الجلسة: ${event.sessionKey}`);
  console.log(`  الطابع الزمني: ${event.timestamp.toISOString()}`);

  // منطقك المخصص هنا

  // إرسال رسالة للمستخدم اختياريًا
  event.messages.push("✨ تم تنفيذ هوكي!");
};

export default myHandler;
```

#### سياق الحدث

يتضمن كل حدث:

```json
{
  type: 'command' | 'session' | 'agent' | 'gateway' | 'message',
  action: string,              // مثال: 'new', 'reset', 'stop', 'received', 'sent'
  sessionKey: string,          // معرف الجلسة
  timestamp: Date,             // وقت حدوث الحدث
  messages: string[],          // ادفع الرسائل هنا لإرسالها للمستخدم
  context: {
    // أحداث الأوامر:
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // مثال: 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig,
    // أحداث الرسائل (راجع قسم أحداث الرسائل للحصول على تفاصيل كاملة):
    from?: string,             // message:received
    to?: string,               // message:sent
    content?: string,
    channelId?: string,
    success?: boolean,         // message:sent
  }
}
```

## أنواع الأحداث

### أحداث الأوامر

يتم تشغيلها عند إصدار أوامر الوكيل:

-   **`command`**: جميع أحداث الأوامر (مستمع عام)
-   **`command:new`**: عند إصدار أمر `/new`
-   **`command:reset`**: عند إصدار أمر `/reset`
-   **`command:stop`**: عند إصدار أمر `/stop`

### أحداث الجلسة

-   **`session:compact:before`**: مباشرة قبل أن يلخص الضغط التاريخ
-   **`session:compact:after`**: بعد اكتمال الضغط مع بيانات وصفية ملخصة

تصدر حمولات الهوك الداخلية هذه كـ `type: "session"` مع `action: "compact:before"` / `action: "compact:after"`؛ يستمع المشتركون باستخدام المفاتيح المدمجة أعلاه. يستخدم تسجيل المعالج المحدد تنسيق المفتاح الحرفي `${type}:${action}`. لهذه الأحداث، سجل `session:compact:before` و `session:compact:after`.

### أحداث الوكيل

-   **`agent:bootstrap`**: قبل حقن ملفات تمهيد مساحة العمل (قد تقوم الهوات بتعديل `context.bootstrapFiles`)

### أحداث البوابة

يتم تشغيلها عند بدء تشغيل البوابة:

-   **`gateway:startup`**: بعد بدء القنوات وتحميل الهوات

### أحداث الرسائل

يتم تشغيلها عند استلام أو إرسال الرسائل:

-   **`message`**: جميع أحداث الرسائل (مستمع عام)
-   **`message:received`**: عند استلام رسالة واردة من أي قناة. يتم تشغيلها مبكرًا في المعالجة قبل فهم الوسائط. قد يحتوي المحتوى على عناصر نائبة خام مثل `<media:audio>` للمرفقات التي لم تتم معالجتها بعد.
-   **`message:transcribed`**: عند معالجة الرسالة بالكامل، بما في ذلك نسخ الصوت وفهم الروابط. في هذه المرحلة، يحتوي `transcript` على نص النسخ الكامل للرسائل الصوتية. استخدم هذا الهوك عندما تحتاج إلى الوصول إلى محتوى الصوت المنقول.
-   **`message:preprocessed`**: يتم تشغيله لكل رسالة بعد اكتمال فهم جميع الوسائط والروابط، مما يمنح الهوات الوصول إلى النص المخصب بالكامل (النصوص المنقولة، أوصاف الصور، ملخصات الروابط) قبل أن يراها الوكيل.
-   **`message:sent`**: عند إرسال رسالة صادرة بنجاح

#### سياق حدث الرسالة

تتضمن أحداث الرسائل سياقًا غنيًا عنها:

```
// سياق message:received
{
  from: string,           // معرف المرسل (رقم هاتف، معرف مستخدم، إلخ)
  content: string,        // محتوى الرسالة
  timestamp?: number,     // الطابع الزمني Unix عند الاستلام
  channelId: string,      // القناة (مثال: "whatsapp", "telegram", "discord")
  accountId?: string,     // معرف حساب المزود لإعدادات الحسابات المتعددة
  conversationId?: string, // معرف الدردشة/المحادثة
  messageId?: string,     // معرف الرسالة من المزود
  metadata?: {            // بيانات إضافية خاصة بالمزود
    to?: string,
    provider?: string,
    surface?: string,
    threadId?: string,
    senderId?: string,
    senderName?: string,
    senderUsername?: string,
    senderE164?: string,
  }
}

// سياق message:sent
{
  to: string,             // معرف المستلم
  content: string,        // محتوى الرسالة الذي تم إرساله
  success: boolean,       // ما إذا نجح الإرسال
  error?: string,         // رسالة خطأ في حالة فشل الإرسال
  channelId: string,      // القناة (مثال: "whatsapp", "telegram", "discord")
  accountId?: string,     // معرف حساب المزود
  conversationId?: string, // معرف الدردشة/المحادثة
  messageId?: string,     // معرف الرسالة الذي أعاده المزود
  isGroup?: boolean,      // ما إذا كانت هذه الرسالة الصادرة تنتمي إلى سياق مجموعة/قناة
  groupId?: string,       // معرف المجموعة/القناة للارتباط بـ message:received
}

// سياق message:transcribed
{
  body?: string,          // النص الوارد الخام قبل الإثراء
  bodyForAgent?: string,  // النص المخصب المرئي للوكيل
  transcript: string,     // نص نسخ الصوت
  channelId: string,      // القناة (مثال: "telegram", "whatsapp")
  conversationId?: string,
  messageId?: string,
}

// سياق message:preprocessed
{
  body?: string,          // النص الوارد الخام
  bodyForAgent?: string,  // النص المخصب النهائي بعد فهم الوسائط/الروابط
  transcript?: string,    // النص المنقول عند وجود صوت
  channelId: string,      // القناة (مثال: "telegram", "whatsapp")
  conversationId?: string,
  messageId?: string,
  isGroup?: boolean,
  groupId?: string,
}
```

#### مثال: هوك مسجل الرسائل

```typescript
const isMessageReceivedEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "received";
const isMessageSentEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "sent";

const handler = async (event) => {
  if (isMessageReceivedEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Received from ${event.context.from}: ${event.context.content}`);
  } else if (isMessageSentEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Sent to ${event.context.to}: ${event.context.content}`);
  }
};

export default handler;
```

### هوات نتائج الأدوات (واجهة برمجة تطبيقات الإضافة)

هذه الهوات ليست مستمعين لتدفق الأحداث؛ فهي تسمح للإضافات بتعديل نتائج الأدوات بشكل متزامن قبل أن يحفظها OpenClaw.

-   **`tool_result_persist`**: تحويل نتائج الأدوات قبل كتابتها في نص الجلسة. يجب أن يكون متزامنًا؛ قم بإرجاع حمولة نتيجة الأداة المحدثة أو `undefined` للحفاظ عليها كما هي. راجع [حلقة الوكيل](../concepts/agent-loop.md).

### أحداث هوات الإضافة

هوات دورة حياة الضغط المعروضة عبر مشغل هوات الإضافة:

-   **`before_compaction`**: يعمل قبل الضغط مع بيانات وصفية للعدد/الرموز
-   **`after_compaction`**: يعمل بعد الضغط مع بيانات وصفية ملخصة للضغط

### الأحداث المستقبلية

أنواع الأحداث المخطط لها:

-   **`session:start`**: عند بدء جلسة جديدة
-   **`session:end`**: عند انتهاء جلسة
-   **`agent:error`**: عند مواجهة الوكيل لخطأ

## إنشاء هوات مخصصة

### 1\. اختر الموقع

-   **هوات مساحة العمل** (`/hooks/`): لكل وكيل، أعلى أولوية
-   **الهوات المدارة** (`~/.openclaw/hooks/`): مشتركة عبر مساحات العمل

### 2\. إنشاء بنية الدليل

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3\. إنشاء HOOK.md

```
---
name: my-hook
description: "يفعل شيئًا مفيدًا"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# هوكي المخصص

يفعل هذا الهوك شيئًا مفيدًا عند إصدار `/new`.
```

### 4\. إنشاء handler.ts

```typescript
const handler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] يعمل!");
  // منطقك هنا
};

export default handler;
```

### 5\. التمكين والاختبار

```bash
# التحقق من اكتشاف الهوك
openclaw hooks list

# تمكينه
openclaw hooks enable my-hook

# أعد تشغيل عملية البوابة الخاصة بك (إعادة تشغيل تطبيق شريط القوائم على macOS، أو إعادة تشغيل عملية التطوير الخاصة بك)

# تشغيل الحدث
# أرسل /new عبر قناة المراسلة الخاصة بك
```

## التهيئة

### تنسيق التهيئة الجديد (موصى به)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### تهيئة لكل هوك

يمكن أن يكون للهوات تهيئة مخصصة:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### أدلة إضافية

تحميل هوات من أدلة إضافية:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### تنسيق التهيئة القديم (لا يزال مدعومًا)

لا يزال تنسيق التهيئة القديم يعمل للتوافق مع الإصدارات السابقة:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

ملاحظة: يجب أن يكون `module` مسارًا نسبيًا لمساحة العمل. يتم رفض المسارات المطلقة والتجاوز خارج مساحة العمل. **الترحيل**: استخدم نظام الاكتشاف الجديد للهوات الجديدة. يتم تحميل المعالجات القديمة بعد الهوات القائمة على الدليل.

## أوامر CLI

### عرض الهوات

```bash
# عرض جميع الهوات
openclaw hooks list

# عرض الهوات المؤهلة فقط
openclaw hooks list --eligible

# إخراج مفصل (عرض المتطلبات المفقودة)
openclaw hooks list --verbose

# إخراج JSON
openclaw hooks list --json
```

### معلومات الهوك

```bash
# عرض معلومات مفصلة عن هوك
openclaw hooks info session-memory

# إخراج JSON
openclaw hooks info session-memory --json
```

### فحص الأهلية

```bash
# عرض ملخص الأهلية
openclaw hooks check

# إخراج JSON
openclaw hooks check --json
```

### تمكين/تعطيل

```bash
# تمكين هوك
openclaw hooks enable session-memory

# تعطيل هوك
openclaw hooks disable command-logger
```

## مرجع الهوات المضمنة

### session-memory

يحفظ سياق الجلسة في الذاكرة عند إصدار `/new`. **الأحداث**: `command:new` **المتطلبات**: يجب تهيئة `workspace.dir` **الإخراج**: `/memory/YYYY-MM-DD-slug.md` (الافتراضي `~/.openclaw/workspace`) **ما يفعله**:

1.  يستخدم إدخال الجلسة قبل إعادة التعيين لتحديد النص الصحيح
2.  يستخرج آخر 15 سطرًا من المحادثة
3.  يستخدم LLM لإنشاء اسم ملف وصفي
4.  يحفظ البيانات الوصفية للجلسة في ملف ذاكرة مؤرخ

**مثال للإخراج**:

```bash
# الجلسة: 2026-01-16 14:30:00 UTC

- **مفتاح الجلسة**: agent:main:main
- **معرف الجلسة**: abc123def456
- **المصدر**: telegram
```

**أمثلة على أسماء الملفات**:

-   `2026-01-16-vendor-pitch.md`
-   `2026-01-16-api-design.md`
-   `2026-01-16-1430.md` (طابع زمني احتياطي إذا فشل إنشاء الاسم)

**التمكين**:

```bash
openclaw hooks enable session-memory
```

### bootstrap-extra-files

يحقن ملفات تمهيد إضافية (على سبيل المثال `AGENTS.md` / `TOOLS.md` المحلية لمستودع أحادي) أثناء `agent:bootstrap`. **الأحداث**: `agent:bootstrap` **المتطلبات**: يجب تهيئة `workspace.dir` **الإخراج**: لا يتم كتابة ملفات؛ يتم تعديل سياق التمهيد في الذاكرة فقط. **التهيئة**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "bootstrap-extra-files": {
          "enabled": true,
          "paths": ["packages/*/AGENTS.md", "packages/*/TOOLS.md"]
        }
      }
    }
  }
}
```

**ملاحظات**:

-   يتم حل المسارات بالنسبة لمساحة العمل.
-   يجب أن تبقى الملفات داخل مساحة العمل (يتم التحقق من realpath).
-   يتم تحميل أسماء الملفات الأساسية المعروفة فقط للتمهيد.
-   يتم الحفاظ على القائمة المسموح بها للوكلاء الفرعيين (`AGENTS.md` و `TOOLS.md` فقط).

**التمكين**:

```bash
openclaw hooks enable bootstrap-extra-files
```

### command-logger

يسجل جميع أحداث الأوامر في ملف تدقيق مركزي. **الأحداث**: `command` **المتطلبات**: لا شيء **الإخراج**: `~/.openclaw/logs/commands.log` **ما يفعله**:

1.  يلتقط تفاصيل الحدث (إجراء الأمر، الطابع الزمني، مفتاح الجلسة، معرف المرسل، المصدر)
2.  يلحق بملف السجل بتنسيق JSONL
3.  يعمل في الخلفية بصمت

**أمثلة على إدخالات السجل**:

```json
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**عرض السجلات**:

```bash
# عرض الأوامر الحديثة
tail -n 20 ~/.openclaw/logs/commands.log

# طباعة جميلة باستخدام jq
cat ~/.openclaw/logs/commands.log | jq .

# التصفية حسب الإجراء
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**التمكين**:

```bash
openclaw hooks enable command-logger
```

### boot-md

يشغل `BOOT.md` عند بدء تشغيل البوابة (بعد بدء القنوات). يجب تمكين الهوات الداخلية لكي يعمل هذا. **الأحداث**: `gateway:startup` **المتطلبات**: يجب تهيئة `workspace.dir` **ما يفعله**:

1.  يقرأ `BOOT.md` من مساحة عملك
2.  يشغل التعليمات عبر مشغل الوكيل
3.  يرسل أي رسائل صادرة مطلوبة عبر أداة الرسائل

**التمكين**:

```bash
openclaw hooks enable boot-md
```

## أفضل الممارسات

### حافظ على سرعة المعالجات

تعمل الهوات أثناء معالجة الأوامر. حافظ عليها خفيفة الوزن:

```
// ✓ جيد - عمل غير متزامن، يعود فورًا
const handler: HookHandler = async (event) => {
  void processInBackground(event); // إطلاق ونسيان
};

// ✗ سيء - يعيق معالجة الأوامر
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### تعامل مع الأخطاء بأسلوب لطيف

احزم دائمًا العمليات المحفوفة بالمخاطر:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] فشل:", err instanceof Error ? err.message : String(err));
    // لا ترمي خطأ - دع المعالجات الأخرى تعمل
  }
};
```

### قم بتصفية الأحداث مبكرًا

ارجع مبكرًا إذا كان الحدث غير ذي صلة:

```typescript
const handler: HookHandler = async (event) => {
  // التعامل مع أوامر 'new' فقط
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // منطقك هنا
};
```

### استخدم مفاتيح أحداث محددة

حدد أحداثًا دقيقة في البيانات الوصفية عندما يكون ذلك ممكنًا:

```
metadata: { "openclaw": { "events": ["command:new"] } } # محدد
```

بدلاً من:

```
metadata: { "openclaw": { "events": ["command"] } } # عام - حمل أكبر
```

## التصحيح

### تمكين تسجيل الهوات

يسجل Gateway تحميل الهوات عند بدء التشغيل:

```bash
Registered hook: session-memory -> command:new
Registered hook: bootstrap-extra-files -> agent:bootstrap
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### فحص الاكتشاف

عرض جميع الهوات المكتشفة:

```bash
openclaw hooks list --verbose
```

### فحص التسجيل

في معالجك، سجل عندما يتم استدعاؤه:

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] تم تشغيله:", event.type, event.action);
  // منطقك
};
```

### التحقق من الأهلية

تحقق من سبب عدم أهلية هوك:

```bash
openclaw hooks info my-hook
```

ابحث عن المتطلبات المفقودة في الإخراج.

## الاختبار

### سجلات البوابة

راقب سجلات البوابة لرؤية تنفيذ الهوك:

```bash
# macOS
./scripts/clawlog.sh -f

# منصات أخرى
tail -f ~/.openclaw/gateway.log
```

### اختبار الهوات مباشرة

اختبر معالجاتك في عزلة:

```typescript
import { test } from "vitest";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = {
    type: "command",
    action: "new",
    sessionKey: "test-session",
    timestamp: new Date(),
    messages: [],
    context: { foo: "bar" },
  };

  await myHandler(event);

  // تأكد من التأثيرات الجانبية
});
```

## البنية

### المكونات الأساسية

-   **`src/hooks/types.ts`**: تعريفات الأنواع
-   **`src/hooks/workspace.ts`**: المسح الضوئي للأدلة والتحميل
-   **`src/hooks/frontmatter.ts`**: تحليل البيانات الوصفية لـ HOOK.md
-   **`src/hooks/config.ts`**: فحص الأهلية
-   **`src/hooks/hooks-status.ts`**: إعداد التقارير عن الحالة
-   **`src/hooks/loader.ts`**: محمل الوحدات الديناميكي
-   **`src/cli/hooks-cli.ts`**: أوامر CLI
-   **`src/gateway/server-startup.ts`**: يحمل الهوات عند بدء تشغيل البوابة
-   **`src/auto-reply/reply/commands-core.ts`**: يشغل أحداث الأوامر

### تدفق الاكتشاف

```
بدء تشغيل البوابة
    ↓
مسح الأدلة (مساحة العمل → المدارة → المضمنة)
    ↓
تحليل ملفات HOOK.md
    ↓
فحص الأهلية (bins, env, config, os)
    ↓
تحميل المعالجات من الهوات المؤهلة
    ↓
تسجيل المعالجات للأحداث
```

### تدفق الأحداث

```
المستخدم يرسل /new
    ↓
التحقق من صحة الأمر
    ↓
إنشاء حدث هوك
    ↓
تشغيل الهوك (جميع المعالجات المسجلة)
    ↓
تستمر معالجة الأمر
    ↓
إعادة تعيين الجلسة
```

## استكشاف الأخطاء وإصلاحها

### لم يتم اكتشاف الهوك

1.  تحقق من بنية الدليل:
    
    ```bash
    ls -la ~/.openclaw/hooks/my-hook/
    # يجب أن يظهر: HOOK.md, handler.ts
    ```
    
2.  تحقق من تنسيق HOOK.md:
    
    ```bash
    cat ~/.openclaw/hooks/my-hook/HOOK.md
    # يجب أن يحتوي على YAML frontmatter مع name و metadata
    ```
    
3.  عرض جميع الهوات المكتشفة:
    
    ```bash
    openclaw hooks list
    ```
    

### الهوك غير مؤهل

تحقق من المتطلبات:

```bash
openclaw hooks info my-hook
```

ابحث عن المفقود:

-   الملفات الثنائية (تحقق من PATH)
-   متغيرات البيئة
-   قيم التهيئة
-   توافق نظام التشغيل

### الهوك لا ينفذ

1.  تحقق من تمكين الهوك:
    
    ```bash
    openclaw hooks list
    # يجب أن يظهر ✓ بجوار الهوات الممكّنة
    ```
    
2.  أعد تشغيل عملية البوابة الخاصة بك لإعادة تحميل الهوات.
3.  تحقق من سجلات البوابة بحثًا عن أخطاء:
    
    ```bash
    ./scripts/clawlog.sh | grep hook
    ```
    

### أخطاء المعالج

تحقق من أخطاء TypeScript/الاستيراد:

```bash
# اختبر الاستيراد مباشرة
node -e "import('./path/to/handler.ts').then(console.log)"
```

## دليل الترحيل

### من التهيئة القديمة إلى الاكتشاف

**قبل**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }