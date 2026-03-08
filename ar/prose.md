title: "إضافة OpenProse لسير عمل الذكاء الاصطناعي في OpenClaw"
description: "تعلّم كيفية استخدام OpenProse، وهو تنسيق يعتمد على Markdown لتنسيق جلسات الذكاء الاصطناعي متعددة الوكلاء. قم بتثبيت الإضافة، وتشغيل أوامر الشرطة المائلة، وإدارة الحالة."
keywords: ["openprose", "سير عمل الذكاء الاصطناعي", "متعدد الوكلاء", "إضافة openclaw", "prose.md", "تنسيق markdown", "أمر الشرطة المائلة", "إدارة الحالة"]
---

  الإضافات

  
# OpenProse

OpenProse هو تنسيق سير عمل محمول يعتمد على Markdown لتنسيق جلسات الذكاء الاصطناعي. في OpenClaw يتم توزيعه كإضافة تقوم بتثبيت حزمة مهارات OpenProse بالإضافة إلى أمر الشرطة المائلة `/prose`. تعيش البرامج في ملفات `.prose` ويمكنها إنشاء وكلاء فرعيين متعددين مع تحكم صريح في تدفق العمل. الموقع الرسمي: [https://www.prose.md](https://www.prose.md)

## ما يمكنه فعله

-   بحث وتركيب متعدد الوكلاء مع توازي صريح.
-   سير عمل قابلة للتكرار وآمنة للموافقة (مراجعة الكود، فرز الحوادث، خطوط أنابيب المحتوى).
-   برامج `.prose` قابلة لإعادة الاستخدام يمكنك تشغيلها عبر بيئات تنفيذ الوكلاء المدعومة.

## التثبيت + التمكين

يتم تعطيل الإضافات المضمنة افتراضيًا. لتمكين OpenProse:

```bash
openclaw plugins enable open-prose
```

أعد تشغيل Gateway بعد تمكين الإضافة. للنسخة المحلية/التطويرية: `openclaw plugins install ./extensions/open-prose` الوثائق ذات الصلة: [الإضافات](./tools/plugin.md)، [بيان الإضافة](./plugins/manifest.md)، [المهارات](./tools/skills.md).

## أمر الشرطة المائلة

يسجل OpenProse الأمر `/prose` كأمر مهارة يمكن للمستخدم استدعاؤه. يقوم بتوجيه الأمر إلى تعليمات OpenProse VM ويستخدم أدوات OpenClaw في الخلفية. الأوامر الشائعة:

```bash
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## مثال: ملف .prose بسيط

```bash
# بحث وتركيب بواسطة وكيلين يعملان بشكل متوازٍ.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## مواقع الملفات

يحتفظ OpenProse بالحالة تحت المجلد `.prose/` في مساحة العمل الخاصة بك:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

تعيش الوكلاء الدائمة على مستوى المستخدم في:

```
~/.prose/agents/
```

## أوضاع الحالة

يدعم OpenProse وحدات تخزين خلفية متعددة للحالة:

-   **نظام الملفات** (الافتراضي): `.prose/runs/...`
-   **ضمن السياق**: مؤقت، للبرامج الصغيرة
-   **sqlite** (تجريبي): يتطلب وجود `sqlite3`
-   **postgres** (تجريبي): يتطلب `psql` وسلسلة اتصال

ملاحظات:

-   sqlite/postgres اختيارية وتجريبية.
-   بيانات اعتماد postgres تتدفق إلى سجلات الوكلاء الفرعية؛ استخدم قاعدة بيانات مخصصة بأقل صلاحيات.

## البرامج البعيدة

`/prose run <handle/slug>` يحل إلى `https://p.prose.md//`. يتم جلب الروابط المباشرة كما هي. يستخدم هذا الأداة `web_fetch` (أو `exec` لـ POST).

## تعيين بيئة تنفيذ OpenClaw

تتم تعيين برامج OpenProse إلى العناصر الأساسية في OpenClaw:

| مفهوم OpenProse | أداة OpenClaw |
| --- | --- |
| بدء جلسة / أداة مهمة | `sessions_spawn` |
| قراءة/كتابة ملف | `read` / `write` |
| جلب من الويب | `web_fetch` |

إذا كانت قائمة السماح للأدوات الخاصة بك تمنع هذه الأدوات، فستفشل برامج OpenProse. راجع [تهيئة المهارات](./tools/skills-config.md).

## الأمان + الموافقات

عامل ملفات `.prose` مثل الكود. راجعها قبل التشغيل. استخدم قوائم السماح لأدوات OpenClaw وبوابات الموافقة للتحكم في التأثيرات الجانبية. لسير العمل الحتمية الخاضعة للموافقة، قارن مع [Lobster](./tools/lobster.md).

[أدوات وكيل الإضافة](./plugins/agent-tools.md)[الخطافات](./automation/hooks.md)