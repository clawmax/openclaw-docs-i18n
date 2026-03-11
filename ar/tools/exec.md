

  أدوات مدمجة

  
# أداة Exec

شغل أوامر shell في مساحة العمل. تدعم التنفيذ في المقدمة + الخلفية عبر `process`. إذا كان `process` غير مسموح، يعمل `exec` بشكل متزامن ويتجاهل `yieldMs`/`background`. الجلسات الخلفية محددة لكل وكيل؛ `process` يرى فقط الجلسات من نفس الوكيل.

## المعلمات

-   `command` (مطلوب)
-   `workdir` (الافتراضي هو cwd)
-   `env` (تجاوزات مفتاح/قيمة)
-   `yieldMs` (الافتراضي 10000): الانتقال التلقائي للخلفية بعد التأخير
-   `background` (منطقي): الانتقال للخلفية فوراً
-   `timeout` (ثواني، الافتراضي 1800): إنهاء عند انتهاء المهلة
-   `pty` (منطقي): التشغيل في طرفية وهمية عند توفرها (واجهات سطر الأوامر التي تعمل فقط على TTY، ووكلاء البرمجة، وواجهات المستخدم الطرفية)
-   `host` (`sandbox | gateway | node`): مكان التنفيذ
-   `security` (`deny | allowlist | full`): وضع الإنفاذ لـ `gateway`/`node`
-   `ask` (`off | on-miss | always`): مطالبات الموافقة لـ `gateway`/`node`
-   `node` (سلسلة نصية): معرف/اسم العقدة لـ `host=node`
-   `elevated` (منطقي): طلب الوضع المرفوع (مضيف gateway)؛ `security=full` يُفرض فقط عندما يحل `elevated` إلى `full`

ملاحظات:

-   `host` الافتراضي هو `sandbox`.
-   `elevated` يتم تجاهله عند إيقاف الحجرة الرملية (exec يعمل بالفعل على المضيف).
-   موافقات `gateway`/`node` يتم التحكم بها بواسطة `~/.openclaw/exec-approvals.json`.
-   `node` تتطلب عقدة مقترنة (تطبيق مرافق أو مضيف عقدة بدون واجهة).
-   إذا كانت هناك عقد متعددة متاحة، عيّن `exec.node` أو `tools.exec.node` لاختيار واحدة.
-   على المضيفين غير Windows، يستخدم exec `SHELL` عندما يكون مضبوطاً؛ إذا كان `SHELL` هو `fish`، فإنه يفضل `bash` (أو `sh`) من `PATH` لتجنب النصوص غير المتوافقة مع fish، ثم يعود إلى `SHELL` إذا لم يكن أي منهما موجوداً.
-   على مضيفي Windows، يفضل exec اكتشاف PowerShell 7 (`pwsh`) (Program Files، ProgramW6432، ثم PATH)، ثم يعود إلى Windows PowerShell 5.1.
-   تنفيذ المضيف (`gateway`/`node`) يرفض `env.PATH` وتجاوزات المحمل (`LD_*`/`DYLD_*`) لمنع اختطاف الملفات الثنائية أو حقن التعليمات البرمجية.
-   يضبط OpenClaw `OPENCLAW_SHELL=exec` في بيئة الأمر المنشأ (بما في ذلك تنفيذ PTY والحجرة الرملية) حتى تتمكن قواعد shell/الملف الشخصي من اكتشاف سياق أداة exec.
-   مهم: الحجرة الرملية **معطلة افتراضياً**. إذا كانت الحجرة الرملية معطلة وتم تكوين/طلب `host=sandbox` بشكل صريح، فإن exec يفشل الآن بشكل مغلق بدلاً من التشغيل بصمت على مضيف gateway. فعّل الحجرة الرملية أو استخدم `host=gateway` مع الموافقات.
-   فحوصات ما قبل التشغيل للنصوص (لأخطاء بناء جملة shell الشائعة في Python/Node) تفحص فقط الملفات داخل حدود `workdir` الفعالة. إذا كان مسار النص يحل خارج `workdir`، يتم تخطي الفحص المسبق لذلك الملف.

## التكوين

-   `tools.exec.notifyOnExit` (الافتراضي: true): عندما يكون true، جلسات exec في الخلفية تضيف حدث نظام وتطلب نبضة قلب عند الخروج.
-   `tools.exec.approvalRunningNoticeMs` (الافتراضي: 10000): إصدار إشعار واحد "قيد التشغيل" عندما يستمر exec الخاضع للموافقة لفترة أطول من هذا (0 يعطله).
-   `tools.exec.host` (الافتراضي: `sandbox`)
-   `tools.exec.security` (الافتراضي: `deny` للحجرة الرملية، `allowlist` لـ gateway + node عندما لا يكون مضبوطاً)
-   `tools.exec.ask` (الافتراضي: `on-miss`)
-   `tools.exec.node` (الافتراضي: غير مضبوط)
-   `tools.exec.pathPrepend`: قائمة الدلائل المطلوب إضافتها في بداية `PATH` لعمليات exec (gateway + الحجرة الرملية فقط).
-   `tools.exec.safeBins`: ملفات ثنائية آمنة للقراءة فقط من stdin يمكنها التشغيل دون إدخالات قائمة السماح الصريحة. للحصول على تفاصيل السلوك، انظر [الملفات الثنائية الآمنة](./exec-approvals.md#safe-bins-stdin-only).
-   `tools.exec.safeBinTrustedDirs`: دلائل إضافية صريحة موثوقة لفحوصات مسار `safeBins`. إدخالات `PATH` ليست موثوقة تلقائياً أبداً. الإعدادات الافتراضية المدمجة هي `/bin` و `/usr/bin`.
-   `tools.exec.safeBinProfiles`: سياسة argv مخصصة اختيارية لكل ملف ثنائي آمن (`minPositional`, `maxPositional`, `allowedValueFlags`, `deniedFlags`).

مثال:

```json
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### معالجة PATH

-   `host=gateway`: يدمج `PATH` من shell تسجيل الدخول الخاص بك في بيئة exec. تجاوزات `env.PATH` مرفوضة لتنفيذ المضيف. الخدمة الخلفية نفسها لا تزال تعمل بأقل `PATH`:
    -   macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
    -   Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
-   `host=sandbox`: يشغل `sh -lc` (shell تسجيل الدخول) داخل الحاوية، لذا قد يعيد `/etc/profile` ضبط `PATH`. يضيف OpenClaw `env.PATH` في البداية بعد تحميل الملف الشخصي عبر متغير بيئة داخلي (بدون استيفاء shell)؛ `tools.exec.pathPrepend` ينطبق هنا أيضاً.
-   `host=node`: فقط تجاوزات env غير المحظورة التي تمررها تُرسل إلى العقدة. تجاوزات `env.PATH` مرفوضة لتنفيذ المضيف ويتم تجاهلها من قبل مضيفي العقد. إذا كنت بحاجة إلى إدخالات PATH إضافية على عقدة، قم بتكوين بيئة خدمة مضيف العقدة (systemd/launchd) أو قم بتثبيت الأدوات في المواقع القياسية.

ربط العقدة لكل وكيل (استخدم فهرس قائمة الوكيل في التكوين):

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

واجهة التحكم: علامة التبويب Nodes تتضمن لوحة صغيرة "ربط عقدة Exec" لنفس الإعدادات.

## تجاوزات الجلسة (/exec)

استخدم `/exec` لتعيين **الافتراضيات لكل جلسة** لـ `host`, `security`, `ask`, و `node`. أرسل `/exec` بدون وسيطات لعرض القيم الحالية. مثال:

```bash
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## نموذج التفويض

يتم احترام `/exec` فقط **للمرسلين المفوضين** (قوائم السماح/الاقتران للقناة بالإضافة إلى `commands.useAccessGroups`). يقوم بتحديث **حالة الجلسة فقط** ولا يكتب التكوين. لتعطيل exec بشكل صارم، ارفضه عبر سياسة الأداة (`tools.deny: ["exec"]` أو لكل وكيل). موافقات المضيف لا تزال تنطبق ما لم تضبط صراحةً `security=full` و `ask=off`.

## موافقات Exec (التطبيق المرافق / مضيف العقدة)

يمكن للوكلاء في الحجرة الرملية أن يتطلبوا موافقة لكل طلب قبل تشغيل `exec` على مضيف gateway أو العقدة. انظر [موافقات Exec](./exec-approvals.md) للحصول على السياسة، وقائمة السماح، وتدفق واجهة المستخدم. عندما تكون الموافقات مطلوبة، تعيد أداة exec فوراً مع `status: "approval-pending"` ومعرف موافقة. بمجرد الموافقة (أو الرفض / انتهاء المهلة)، يصدر Gateway أحداث نظام (`Exec finished` / `Exec denied`). إذا كان الأمر لا يزال قيد التشغيل بعد `tools.exec.approvalRunningNoticeMs`، يتم إصدار إشعار واحد `Exec running`.

## قائمة السماح + الملفات الثنائية الآمنة

إنفاذ قائمة السماح اليدوية يطابق **مسارات الملفات الثنائية المحلولة فقط** (لا يوجد تطابق لأسماء الأساس). عندما يكون `security=allowlist`، يتم السماح تلقائياً لأوامر shell فقط إذا كان كل مقطع في خط الأنابيب مدرجاً في قائمة السماح أو ملف ثنائي آمن. التسلسل (`;`, `&&`, `||`) وإعادة التوجيه مرفوضة في وضع قائمة السماح ما لم يحقق كل مقطع من المستوى الأعلى قائمة السماح (بما في ذلك الملفات الثنائية الآمنة). إعادة التوجيه لا تزال غير مدعومة. `autoAllowSkills` هو مسار راحة منفصل في موافقات exec. إنه ليس نفس إدخالات قائمة السماح اليدوية للمسار. للثقة الصريحة الصارمة، أبقِ `autoAllowSkills` معطلاً. استخدم عنصري التحكم لأعمال مختلفة:

-   `tools.exec.safeBins`: مرشحات تدفق صغيرة للقراءة فقط من stdin.
-   `tools.exec.safeBinTrustedDirs`: دلائل موثوقة إضافية صريحة لمسارات الملفات الثنائية الآمنة.
-   `tools.exec.safeBinProfiles`: سياسة argv صريحة للملفات الثنائية الآمنة المخصصة.
-   قائمة السماح: ثقة صريحة لمسارات الملفات القابلة للتنفيذ.

لا تعامل `safeBins` كقائمة سماح عامة، ولا تضيف ملفات ثنائية للمترجم/وقت التشغيل (على سبيل المثال `python3`, `node`, `ruby`, `bash`). إذا كنت بحاجة إلى تلك، استخدم إدخالات قائمة السماح الصريحة وأبقِ مطالبات الموافقة مفعلة. يحذر `openclaw security audit` عندما تكون إدخالات `safeBins` للمترجم/وقت التشغيل تفتقد إلى ملفات تعريف صريحة، ويمكن لـ `openclaw doctor --fix` إنشاء إدخالات `safeBinProfiles` مخصصة مفقودة. للحصول على تفاصيل السياسة الكاملة والأمثلة، انظر [موافقات Exec](./exec-approvals.md#safe-bins-stdin-only) و [الملفات الثنائية الآمنة مقابل قائمة السماح](./exec-approvals.md#safe-bins-versus-allowlist).

## أمثلة

في المقدمة:

```json
{ "tool": "exec", "command": "ls -la" }
```

الخلفية + الاستطلاع:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

إرسال مفاتيح (نمط tmux):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

إرسال (إرسال CR فقط):

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

لصق (بأقواس افتراضياً):

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply\_patch (تجريبي)

`apply_patch` هي أداة فرعية لـ `exec` للتعديلات المنظمة متعددة الملفات. فعّلها صراحةً:

```json
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

ملاحظات:

-   متاحة فقط لنماذج OpenAI/OpenAI Codex.
-   سياسة الأداة لا تزال تنطبق؛ `allow: ["exec"]` تسمح ضمنياً بـ `apply_patch`.
-   التكوين موجود تحت `tools.exec.applyPatch`.
-   `tools.exec.applyPatch.workspaceOnly` الافتراضي هو `true` (محتوى مساحة العمل). اضبطه على `false` فقط إذا كنت تريد عمداً أن يكتب/يحذف `apply_patch` خارج دليل مساحة العمل.

[الوضع المرفوع](./elevated.md)[موافقات Exec](./exec-approvals.md)