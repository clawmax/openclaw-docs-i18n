title: "دليل إعدادات ومهارات OpenClaw"
description: "تعلم كيفية تكوين مهارات الذكاء الاصطناعي OpenClaw. قم بتعيين القوائم المسموح بها، وتحميل أدلة المهارات المخصصة، وإدارة الإعدادات لكل مهارة، وتكوين متغيرات البيئة."
keywords: ["مهارات openclaw", "تكوين المهارات", "مهارات الوكيل", "إعدادات المهارة", "أدلة المهارات", "متغيرات بيئة المهارة", "تجاوزات المهارة", "مهارات معزولة"]
---

  المهارات

  
# تكوين المهارات

جميع إعدادات التكوين المتعلقة بالمهارات موجودة تحت `skills` في `~/.openclaw/openclaw.json`.

```json
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway runtime still Node; bun not recommended)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // or plaintext string
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## الحقول

-   `allowBundled`: قائمة اختيارية للمهارات **المضمنة** فقط. عند تعيينها، فقط المهارات المضمنة في القائمة مؤهلة (المهارات المدارة أو في مساحة العمل غير متأثرة).
-   `load.extraDirs`: أدلة مهارات إضافية للمسح (الأولوية الأدنى).
-   `load.watch`: مراقبة مجلدات المهارات وتحديث لقطة المهارات (الافتراضي: true).
-   `load.watchDebounceMs`: إزالة الارتداد لأحداث مراقب المهارات بالملي ثانية (الافتراضي: 250).
-   `install.preferBrew`: تفضيل مثبتات brew عند توفرها (الافتراضي: true).
-   `install.nodeManager`: تفضيل مدير حزم Node (`npm` | `pnpm` | `yarn` | `bun`، الافتراضي: npm). هذا يؤثر فقط على **تثبيت المهارات**؛ وقت تشغيل Gateway يجب أن يبقى Node (لا يوصى بـ Bun لـ WhatsApp/Telegram).
-   `entries.`: تجاوزات لكل مهارة.

الحقول لكل مهارة:

-   `enabled`: عيّن `false` لتعطيل مهارة حتى لو كانت مضمنة/مثبتة.
-   `env`: متغيرات البيئة المحقونة لتشغيل الوكيل (فقط إذا لم تكن مضبوطة مسبقًا).
-   `apiKey`: اختياري للراحة للمهارات التي تعلن عن متغير بيئة أساسي. يدعم نصًا عاديًا أو كائن SecretRef (`{ source, provider, id }`).

## ملاحظات

-   المفاتيح تحت `entries` ترتبط باسم المهارة افتراضيًا. إذا عرّفت المهارة `metadata.openclaw.skillKey`، استخدم ذلك المفتاح بدلاً من ذلك.
-   يتم التقاط التغييرات في المهارات في دورة الوكيل التالية عندما يكون المراقب مفعلًا.

### المهارات المعزولة + متغيرات البيئة

عندما تكون الجلسة **معزولة**، تعمل عمليات المهارة داخل Docker. الصندوق الرملي **لا** يرث `process.env` من المضيف. استخدم أحد الخيارات:

-   `agents.defaults.sandbox.docker.env` (أو لكل وكيل `agents.list[].sandbox.docker.env`)
-   خبز متغير البيئة في صورتك المعزولة المخصصة

`env` العام و `skills.entries..env/apiKey` ينطبقان على عمليات التشغيل على **المضيف** فقط.

[المهارات](./skills.md)[ClawHub](./clawhub.md)

---