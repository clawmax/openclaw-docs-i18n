title: "مساعدات تصحيح OpenClaw لإخراج البث والبيئة"
description: "تعلم كيفية تصحيح OpenClaw باستخدام التجاوزات أثناء التشغيل، ملفات التعريف للتطوير، تسجيل البث الخام، ووضع المراقبة للبوابة للتكرار السريع."
keywords: ["تصحيح openclaw", "تصحيح إخراج البث", "تجاوزات وقت التشغيل", "ملف تعريف التطوير", "وضع مراقبة البوابة", "تسجيل البث الخام", "تسجيل pi-mono", "تكوين البيئة"]
---

  البيئة والتصحيح

  
# التصحيح

تغطي هذه الصفحة مساعدات التصحيح لإخراج البث، خاصةً عندما يخلط مزود الخدمة بين التفكير والنص العادي.

## تجاوزات تصحيح وقت التشغيل

استخدم `/debug` في الدردشة لتعيين تجاوزات التكوين **لوقت التشغيل فقط** (في الذاكرة، وليس على القرص). `/debug` معطلة افتراضيًا؛ قم بتمكينها باستخدام `commands.debug: true`. هذا مفيد عندما تحتاج إلى تبديل إعدادات غير واضحة دون تعديل `openclaw.json`. أمثلة:

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` تمسح جميع التجاوزات وتعود إلى التكوين المخزن على القرص.

## وضع مراقبة البوابة

للتكرار السريع، قم بتشغيل البوابة تحت مراقب الملفات:

```bash
pnpm gateway:watch
```

هذا يطابق:

```bash
node --watch-path src --watch-path tsconfig.json --watch-path package.json --watch-preserve-output scripts/run-node.mjs gateway --force
```

أضف أي وسائط سطر أوامر للبوابة بعد `gateway:watch` وسيتم تمريرها في كل إعادة تشغيل.

## ملف تعريف التطوير + بوابة التطوير (—dev)

استخدم ملف تعريف التطوير لعزل الحالة وإنشاء إعداد آمن وقابل للتخلص منه لتصحيح الأخطاء. هناك **وسيطتان** `--dev`:

-   **`--dev` العام (ملف التعريف):** يعزل الحالة تحت `~/.openclaw-dev` ويجعل منفذ البوابة الافتراضي `19001` (تتغير المنافذ المشتقة معه).
-   **`gateway --dev`: يخبر البوابة بإنشاء تكوين افتراضي + مساحة عمل تلقائيًا** عند فقدانها (وتخطي BOOTSTRAP.md).

التدفق الموصى به (ملف تعريف التطوير + تهيئة التطوير):

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

إذا لم يكن لديك تثبيت عام بعد، قم بتشغيل واجهة سطر الأوامر عبر `pnpm openclaw ...`. ما يفعله هذا:

1.  **عزل ملف التعريف** (`--dev` العام)
    -   `OPENCLAW_PROFILE=dev`
    -   `OPENCLAW_STATE_DIR=~/.openclaw-dev`
    -   `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
    -   `OPENCLAW_GATEWAY_PORT=19001` (يتغير متصفح/لوحة الرسم وفقًا لذلك)
2.  **تهيئة التطوير** (`gateway --dev`)
    -   يكتب تكوينًا بسيطًا إذا كان مفقودًا (`gateway.mode=local`، ربط loopback).
    -   يضبط `agent.workspace` على مساحة عمل التطوير.
    -   يضبط `agent.skipBootstrap=true` (لا يوجد BOOTSTRAP.md).
    -   يملأ ملفات مساحة العمل إذا كانت مفقودة: `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
    -   الهوية الافتراضية: **C3‑PO** (روبوت البروتوكول).
    -   يتخطى مزودي القنوات في وضع التطوير (`OPENCLAW_SKIP_CHANNELS=1`).

تدفق الإعادة (بداية جديدة):

```bash
pnpm gateway:dev:reset
```

ملاحظة: `--dev` هو وسيط **عام** لملف التعريف ويتم "التهامه" من قبل بعض المديرين. إذا كنت بحاجة إلى كتابته صراحة، استخدم صيغة متغير البيئة:

```
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` تمسح التكوين، بيانات الاعتماد، الجلسات، ومساحة عمل التطوير (باستخدام `trash`، وليس `rm`)، ثم تعيد إنشاء إعداد التطوير الافتراضي. نصيحة: إذا كانت بوابة غير للتطوير تعمل بالفعل (launchd/systemd)، أوقفها أولاً:

```bash
openclaw gateway stop
```

## تسجيل البث الخام (OpenClaw)

يمكن لـ OpenClaw تسجيل **بث المساعد الخام** قبل أي تصفية/تنسيق. هذه هي أفضل طريقة لمعرفة ما إذا كان التفكير يصل كدلتا نص عادي (أو ككتل تفكير منفصلة). قم بتمكينها عبر واجهة سطر الأوامر:

```bash
pnpm gateway:watch --raw-stream
```

تجاوز المسار الاختياري:

```bash
pnpm gateway:watch --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

متغيرات البيئة المكافئة:

```
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

الملف الافتراضي: `~/.openclaw/logs/raw-stream.jsonl`

## تسجيل القطع الخام (pi-mono)

للتقاط **قطع OpenAI-compat الخام** قبل تحليلها إلى كتل، يعرض pi-mono مسجلًا منفصلًا:

```
PI_RAW_STREAM=1
```

مسار اختياري:

```
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

الملف الافتراضي: `~/.pi-mono/logs/raw-openai-completions.jsonl`

> ملاحظة: هذا يصدر فقط من العمليات التي تستخدم مزود `openai-completions` الخاص بـ pi-mono.

## ملاحظات الأمان

-   يمكن أن تتضمن سجلات البث الخام المطالبات الكاملة، إخراج الأدوات، وبيانات المستخدم.
-   احتفظ بالسجلات محليًا واحذفها بعد التصحيح.
-   إذا شاركت السجلات، قم بإزالة الأسرار والمعلومات الشخصية أولاً.

[متغيرات البيئة](./environment.md)[الاختبار](./testing.md)