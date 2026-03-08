

  البروتوكولات وواجهات برمجة التطبيقات

  
# واجهات سطر الأوامر الخلفية

يمكن لـ OpenClaw تشغيل **واجهات سطر الأوامر المحلية للذكاء الاصطناعي** كـ **نسخ احتياطي نصي فقط** عندما تكون موفري واجهات برمجة التطبيقات معطلين، أو محدودي السرعة، أو يعانون من مشاكل مؤقتة. هذا متعمد للحفاظ على الحذر:

-   **الأدوات معطلة** (لا استدعاءات للأدوات).
-   **نص يدخل → نص يخرج** (موثوق).
-   **الجلسات مدعومة** (لتبقى الأدوار المتتابعة متماسكة).
-   **يمكن تمرير الصور** إذا كانت واجهة سطر الأوامر تقبل مسارات الصور.

هذا مصمم كـ **شبكة أمان** وليس كمسار أساسي. استخدمه عندما تريد ردود نصية "تعمل دائمًا" دون الاعتماد على واجهات برمجة التطبيقات الخارجية.

## بداية سريعة للمبتدئين

يمكنك استخدام واجهة سطر أوامر Claude Code **دون أي إعداد** (يأتي OpenClaw بإعداد افتراضي مدمج):

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.6
```

واجهة سطر أوامر Codex تعمل أيضًا مباشرة:

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.4
```

إذا كان البواب الخاص بك يعمل تحت launchd/systemd وكان PATH محدودًا، أضف فقط مسار الأمر:

```json
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
      },
    },
  },
}
```

هذا كل شيء. لا حاجة لمفاتيح، ولا حاجة لإعداد مصادقة إضافي بخلاف واجهة سطر الأوامر نفسها.

## استخدامها كنسخ احتياطي

أضف واجهة سطر أوامر خلفية إلى قائمة النسخ الاحتياطي الخاصة بك حتى تعمل فقط عندما تفشل النماذج الأساسية:

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["claude-cli/opus-4.6", "claude-cli/opus-4.5"],
      },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "claude-cli/opus-4.6": {},
        "claude-cli/opus-4.5": {},
      },
    },
  },
}
```

ملاحظات:

-   إذا استخدمت `agents.defaults.models` (القائمة البيضاء)، يجب أن تتضمن `claude-cli/...`.
-   إذا فشل المزود الأساسي (المصادقة، حدود السرعة، المهلات)، ستحاول OpenClaw استخدام واجهة سطر الأوامر الخلفية بعد ذلك.

## نظرة عامة على الإعداد

توجد جميع واجهات سطر الأوامر الخلفية تحت:

```
agents.defaults.cliBackends
```

يتم تعريف كل إدخال بواسطة **معرف مزود** (مثل `claude-cli`, `my-cli`). يصبح معرف المزود الجزء الأيسر من مرجع النموذج الخاص بك:

```
<provider>/<model>
```

### مثال على الإعداد

```json
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-6": "opus",
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet",
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true,
        },
      },
    },
  },
}
```

## كيف تعمل

1.  **تحدد واجهة خلفية** بناءً على بادئة المزود (`claude-cli/...`).
2.  **تبني مطالبة نظام** باستخدام نفس مطالبة OpenClaw + سياق مساحة العمل.
3.  **تنفذ واجهة سطر الأوامر** مع معرف جلسة (إذا كانت مدعومة) لتبقى السجل متسقًا.
4.  **تحلل المخرجات** (JSON أو نص عادي) وتعيد النص النهائي.
5.  **تحافظ على معرفات الجلسات** لكل واجهة خلفية، بحيث تعيد الأدوار المتتابعة استخدام نفس جلسة واجهة سطر الأوامر.

## الجلسات

-   إذا كانت واجهة سطر الأوامر تدعم الجلسات، عيّن `sessionArg` (مثل `--session-id`) أو `sessionArgs` (العنصر النائب `{sessionId}`) عندما يحتاج المعرف إلى إدراجه في أعلام متعددة.
-   إذا كانت واجهة سطر الأوامر تستخدم **أمر فرعي للاستئناف** بأعلام مختلفة، عيّن `resumeArgs` (يحل محل `args` عند الاستئناف) واختياريًا `resumeOutput` (للاستئنافات غير JSON).
-   `sessionMode`:
    -   `always`: إرسال معرف جلسة دائمًا (معرف فريد جديد إذا لم يكن هناك أي معرف مخزن).
    -   `existing`: إرسال معرف جلسة فقط إذا كان هناك معرف مخزن مسبقًا.
    -   `none`: عدم إرسال معرف جلسة أبدًا.

## الصور (التمرير)

إذا كانت واجهة سطر الأوامر الخاصة بك تقبل مسارات الصور، عيّن `imageArg`:

```yaml
imageArg: "--image",
imageMode: "repeat"
```

ستقوم OpenClaw بكتابة صور base64 إلى ملفات مؤقتة. إذا تم تعيين `imageArg`، يتم تمرير هذه المسارات كوسائط لواجهة سطر الأوامر. إذا كان `imageArg` مفقودًا، تقوم OpenClaw بإلحاق مسارات الملفات بالمطالبة (حقن المسار)، وهو ما يكفي لواجهات سطر الأوامر التي تلقائيًا تحمل الملفات المحلية من المسارات العادية (سلوك واجهة سطر أوامر Claude Code).

## المدخلات / المخرجات

-   `output: "json"` (الافتراضي) يحاول تحليل JSON واستخراج النص + معرف الجلسة.
-   `output: "jsonl"` يحلل تدفقات JSONL (واجهة سطر أوامر Codex `--json`) ويستخرج آخر رسالة وكيل بالإضافة إلى `thread_id` عند وجوده.
-   `output: "text"` يعامل stdout كالرد النهائي.

أوضاع الإدخال:

-   `input: "arg"` (الافتراضي) يمرر المطالبة كوسيطة واجهة سطر الأوامر الأخيرة.
-   `input: "stdin"` يرسل المطالبة عبر stdin.
-   إذا كانت المطالبة طويلة جدًا وتم تعيين `maxPromptArgChars`، يتم استخدام stdin.

## الإعدادات الافتراضية (المدمجة)

يأتي OpenClaw بإعداد افتراضي لـ `claude-cli`:

-   `command: "claude"`
-   `args: ["-p", "--output-format", "json", "--permission-mode", "bypassPermissions"]`
-   `resumeArgs: ["-p", "--output-format", "json", "--permission-mode", "bypassPermissions", "--resume", "{sessionId}"]`
-   `modelArg: "--model"`
-   `systemPromptArg: "--append-system-prompt"`
-   `sessionArg: "--session-id"`
-   `systemPromptWhen: "first"`
-   `sessionMode: "always"`

يأتي OpenClaw أيضًا بإعداد افتراضي لـ `codex-cli`:

-   `command: "codex"`
-   `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
-   `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
-   `output: "jsonl"`
-   `resumeOutput: "text"`
-   `modelArg: "--model"`
-   `imageArg: "--image"`
-   `sessionMode: "existing"`

تجاوز الإعدادات فقط إذا لزم الأمر (الشائع: مسار `command` المطلق).

## القيود

-   **لا توجد أدوات OpenClaw** (لا تتلقى واجهة سطر الأوامر الخلفية استدعاءات للأدوات أبدًا). قد لا تزال بعض واجهات سطر الأوامر تشغل أدوات الوكيل الخاصة بها.
-   **لا يوجد تدفق** (يتم جمع مخرجات واجهة سطر الأوامر ثم إرجاعها).
-   **المخرجات المنظمة** تعتمد على تنسيق JSON لواجهة سطر الأوامر.
-   **جلسات واجهة سطر أوامر Codex** تستأنف عبر مخرجات نصية (بدون JSONL)، وهي أقل تنظيمًا من التشغيل الأولي `--json`. لا تزال جلسات OpenClaw تعمل بشكل طبيعي.

## استكشاف الأخطاء وإصلاحها

-   **لم يتم العثور على واجهة سطر الأوامر**: عيّن `command` إلى مسار كامل.
-   **اسم النموذج خاطئ**: استخدم `modelAliases` لتعيين `provider/model` → نموذج واجهة سطر الأوامر.
-   **لا استمرارية للجلسة**: تأكد من تعيين `sessionArg` وأن `sessionMode` ليس `none` (لا يمكن لواجهة سطر أوامر Codex حاليًا الاستئناف مع مخرجات JSON).
-   **يتم تجاهل الصور**: عيّن `imageArg` (وتأكد من أن واجهة سطر الأوامر تدعم مسارات الملفات).

[أدوات استدعاء واجهة برمجة التطبيقات](./tools-invoke-http-api.md)[النماذج المحلية](./local-models.md)