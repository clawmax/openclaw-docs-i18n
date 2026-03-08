title: "Lobster Workflow Shell لسلاسل أدوات OpenClaw متعددة الخطوات"
description: "تعلم كيف يقوم Lobster بتشغيل سلاسل أدوات متعددة الخطوات حتمية مع موافقات مدمجة وحالة قابلة للاستئناف. قلل من استدعاءات LLM ونظم سير العمل في استدعاء واحد."
keywords: ["lobster", "openclaw", "أتمتة سير العمل", "خطوط الأنابيب الحتمية", "سير عمل الموافقات", "حالة قابلة للاستئناف", "أدوات الذكاء الاصطناعي", "أتمتة سطر الأوامر"]
---

  أدوات مدمجة

  
# Lobster

Lobster هو غلاف سير عمل يسمح لـ OpenClaw بتشغيل سلاسل أدوات متعددة الخطوات كعملية واحدة حتمية مع نقاط تفتيش موافقة صريحة.

## الفكرة

يمكن لمساعدك بناء الأدوات التي تدير نفسه. اطلب سير عمل، وبعد 30 دقيقة لديك واجهة سطر أوامر بالإضافة إلى خطوط أنابيب تعمل كاستدعاء واحد. Lobster هو القطعة المفقودة: خطوط الأنابيب الحتمية، الموافقات الصريحة، والحالة القابلة للاستئناف.

## لماذا

اليوم، تتطلب سير العمل المعقدة العديد من استدعاءات الأدوات ذهابًا وإيابًا. كل استدعاء يكلف رموزًا (tokens)، ويجب على LLM تنظيم كل خطوة. يقوم Lobster بنقل هذا التنظيم إلى وقت تشغيل مكتوب (typed runtime):

-   **استدعاء واحد بدلاً من العديد**: يقوم OpenClaw باستدعاء أداة Lobster واحدة ويحصل على نتيجة منظمة.
-   **موافقات مدمجة**: الآثار الجانبية (إرسال بريد إلكتروني، نشر تعليق) توقف سير العمل حتى يتم الموافقة عليه صراحة.
-   **قابل للاستئناف**: سير العمل المتوقف يعيد رمزًا (token); وافق واستأنف دون إعادة تشغيل كل شيء.

## لماذا لغة تعريفية (DSL) بدلاً من برامج عادية؟

Lobster صغير عمدًا. الهدف ليس "لغة جديدة"، بل هو مواصفات خط أنابيب متوقع وصديق للذكاء الاصطناعي مع موافقات ورموز استئناف من الدرجة الأولى.

-   **الموافقة/الاستئناف مدمجة**: يمكن للبرنامج العادي أن يطلب من الإنسان، لكنه لا يستطيع *التوقف والاستئناف* برمز دائم (durable token) دون أن تخترع وقت التشغيل هذا بنفسك.
-   **الحتمية + إمكانية التدقيق**: خطوط الأنابيب هي بيانات، لذا يسهل تسجيلها، ومقارنة الفروق فيها، وإعادة تشغيلها، ومراجعتها.
-   **مساحة محدودة للذكاء الاصطناعي**: قواعد نحوية صغيرة + توصيل JSON يقلل من مسارات الكود "الإبداعية" ويجعل التحقق واقعيًا.
-   **سياسة السلامة مدمجة**: المهلات الزمنية، حدود المخرجات، فحوصات الحماية (sandbox)، والقوائم المسموحة يتم فرضها بواسطة وقت التشغيل، وليس كل نص برمجي.
-   **لا يزال قابلًا للبرمجة**: يمكن لكل خطوة استدعاء أي واجهة سطر أوامر أو نص برمجي. إذا كنت تريد JS/TS، فقم بإنشاء ملفات `.lobster` من الكود.

## كيف يعمل

يقوم OpenClaw بتشغيل واجهة سطر أوامر `lobster` المحلية في **وضع الأداة** ويحلل مغلف JSON من stdout. إذا توقف خط الأنابيب للموافقة، تعيد الأداة `resumeToken` حتى تتمكن من المتابعة لاحقًا.

## النمط: واجهة سطر أوامر صغيرة + توصيل JSON + موافقات

أنشئ أوامر صغيرة تتحدث JSON، ثم ربطها في استدعاء Lobster واحد. (أسماء الأوامر في المثال أدناه — استبدلها بأوامرك الخاصة.)

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

إذا طلب خط الأنابيب موافقة، استأنف باستخدام الرمز (token):

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

يُطلق الذكاء الاصطناعي سير العمل؛ يقوم Lobster بتنفيذ الخطوات. بوابات الموافقة تحافظ على الآثار الجانبية صريحة وقابلة للتدقيق. مثال: تعيين عناصر الإدخال إلى استدعاءات الأدوات:

```
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

## خطوات LLM باستخدام JSON فقط (llm-task)

لسير العمل التي تحتاج إلى **خطوة LLM منظمة**، قم بتمكين أداة الإضافة الاختيارية `llm-task` واستدعها من Lobster. هذا يحافظ على حتمية سير العمل مع السماح لك بالتصنيف/التلخيص/الصياغة باستخدام نموذج. مكن الأداة:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

استخدمها في خط أنابيب:

```
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

راجع [LLM Task](./llm-task.md) للتفاصيل وخيارات التكوين.

## ملفات سير العمل (.lobster)

يمكن لـ Lobster تشغيل ملفات سير العمل YAML/JSON مع الحقول `name`, `args`, `steps`, `env`, `condition`, و `approval`. في استدعاءات أداة OpenClaw، عيّن `pipeline` إلى مسار الملف.

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

ملاحظات:

-   `stdin: $step.stdout` و `stdin: $step.json` تمرر مخرجات خطوة سابقة.
-   يمكن لـ `condition` (أو `when`) التحكم في الخطوات بناءً على `$step.approved`.

## تثبيت Lobster

قم بتثبيت واجهة سطر أوامر Lobster على **نفس المضيف** الذي يعمل عليه OpenClaw Gateway (راجع [مستودع Lobster](https://github.com/openclaw/lobster))، وتأكد من أن `lobster` موجود في `PATH`.

## تمكين الأداة

Lobster هي أداة إضافة **اختيارية** (غير مفعلة افتراضيًا). موصى به (إضافي، آمن):

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

أو لكل وكيل:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

تجنب استخدام `tools.allow: ["lobster"]` إلا إذا كنت تنوي التشغيل في وضع القائمة المسموحة المقيد. ملاحظة: القوائم المسموحة اختيارية للإضافات الاختيارية. إذا كانت قائمتك المسموحة تسمي فقط أدوات الإضافة (مثل `lobster`)، يحتفظ OpenClaw بالأدوات الأساسية مفعلة. لتقييد الأدوات الأساسية، قم بتضمين الأدوات أو المجموعات الأساسية التي تريدها في القائمة المسموحة أيضًا.

## مثال: فرز البريد الإلكتروني

بدون Lobster:

```
المستخدم: "تحقق من بريدي الإلكتروني وصُغ ردودًا"
→ يستدعي openclaw أداة gmail.list
→ يقوم LLM بالتلخيص
→ المستخدم: "صغ ردودًا على #2 و #5"
→ يصوغ LLM الردود
→ المستخدم: "أرسل #2"
→ يستدعي openclaw أداة gmail.send
(كرر يوميًا، لا توجد ذاكرة بما تم فرزه)
```

مع Lobster:

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

يعيد مغلف JSON (مختصر):

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "Send 2 draft replies?",
    "items": [],
    "resumeToken": "..."
  }
}
```

يوافق المستخدم → استئناف:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

سير عمل واحد. حتمي. آمن.

## معلمات الأداة

### run

تشغيل خط أنابيب في وضع الأداة.

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

تشغيل ملف سير عمل مع وسائط (args):

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### resume

متابعة سير عمل متوقف بعد الموافقة.

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### مدخلات اختيارية

-   `cwd`: دليل العمل النسبي لخط الأنابيب (يجب أن يبقى ضمن دليل العمل الحالي للعملية).
-   `timeoutMs`: قتل العملية الفرعية إذا تجاوزت هذه المدة (الافتراضي: 20000).
-   `maxStdoutBytes`: قتل العملية الفرعية إذا تجاوز حجم stdout هذا الحجم (الافتراضي: 512000).
-   `argsJson`: سلسلة JSON يتم تمريرها إلى `lobster run --args-json` (ملفات سير العمل فقط).

## مغلف المخرجات

يعيد Lobster مغلف JSON بإحدى ثلاث حالات:

-   `ok` → انتهى بنجاح
-   `needs_approval` → متوقف؛ `requiresApproval.resumeToken` مطلوب للاستئناف
-   `cancelled` → مرفوض أو ملغي صراحة

تعرض الأداة المغلف في كل من `content` (JSON منسق) و `details` (كائن خام).

## الموافقات

إذا كان `requiresApproval` موجودًا، افحص المطالبة (prompt) وقرر:

-   `approve: true` → استأنف واستمر في الآثار الجانبية
-   `approve: false` → ألغِ وأنهِ سير العمل

استخدم `approve --preview-from-stdin --limit N` لإرفاق معاينة JSON لطلبات الموافقة دون الحاجة إلى لصق jq/heredoc مخصص. رموز الاستئناف أصبحت مضغوطة الآن: يقوم Lobster بتخزين حالة استئناف سير العمل تحت دليل حالته ويعيد مفتاح رمز صغير.

## OpenProse

يتناسب OpenProse جيدًا مع Lobster: استخدم `/prose` لتنظيم تحضير متعدد الوكلاء، ثم شغل خط أنابيب Lobster للموافقات الحتمية. إذا احتاج برنامج Prose إلى Lobster، اسمح بأداة `lobster` للوكلاء الفرعيين عبر `tools.subagents.tools`. راجع [OpenProse](../prose.md).

## السلامة

-   **عملية فرعية محلية فقط** — لا توجد استدعاءات شبكية من الإضافة نفسها.
-   **لا توجد أسرار** — لا يدير Lobster OAuth؛ فهو يستدعي أدوات OpenClaw التي تقوم بذلك.
-   **واعٍ بالحماية (Sandbox-aware)** — معطل عندما يكون سياق الأداة محميًا (sandboxed).
-   **مقوى** — اسم تنفيذي ثابت (`lobster`) على `PATH`؛ يتم فرض المهلات الزمنية وحدود المخرجات.

## استكشاف الأخطاء وإصلاحها

-   **`lobster subprocess timed out`** → زد `timeoutMs`، أو قسّم خط أنابيب طويل.
-   **`lobster output exceeded maxStdoutBytes`** → ارفع `maxStdoutBytes` أو قلل حجم المخرجات.
-   **`lobster returned invalid JSON`** → تأكد من تشغيل خط الأنابيب في وضع الأداة وطباعة JSON فقط.
-   **`lobster failed (code …)`** → شغل نفس خط الأنابيب في طرفية لفحص stderr.

## تعلم المزيد

-   [الإضافات](./plugin.md)
-   [كتابة أدوات الإضافة](../plugins/agent-tools.md)

## دراسة حالة: سير عمل المجتمع

مثال عام واحد: واجهة سطر أوامر "الدماغ الثاني" + خطوط أنابيب Lobster التي تدير ثلاث خزائن Markdown (شخصية، شريك، مشتركة). تصدر واجهة سطر الأوامر JSON للإحصائيات، وعناوين البريد الوارد، وفحوصات البيانات القديمة؛ تربط Lobster تلك الأوامر في سير عمل مثل `weekly-review`, `inbox-triage`, `memory-consolidation`, و `shared-task-sync`، كل منها يحتوي على بوابات موافقة. يتعامل الذكاء الاصطناعي مع الحكم (التصنيف) عند التوفر ويرجع إلى قواعد حتمية عند عدم التوفر.

-   سلسلة التغريدات: [https://x.com/plattenschieber/status/2014508656335770033](https://x.com/plattenschieber/status/2014508656335770033)
-   المستودع: [https://github.com/bloomedai/brain-cli](https://github.com/bloomedai/brain-cli)

[LLM Task](./llm-task.md)[كشف حلقة الأداة](./loop-detection.md)