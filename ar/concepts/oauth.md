title: "دليل OAuth والمصادقة بالاشتراك في OpenClaw"
description: "تعرف على كيفية عمل OAuth والمصادقة بالاشتراك في OpenClaw لـ OpenAI و Anthropic، بما في ذلك تخزين الرمز المميز، تدفقات التحديث، وإدارة حسابات متعددة."
keywords: ["oauth", "مصادقة openclaw", "anthropic setup-token", "openai codex oauth", "ملفات تعريف المصادقة", "تخزين الرمز المميز", "pkce", "حسابات متعددة"]
---

  الأساسيات

  
# OAuth

يدعم OpenClaw "مصادقة الاشتراك" عبر OAuth لمقدمي الخدمة الذين يقدمونها (أبرزهم **OpenAI Codex (ChatGPT OAuth)**). لاشتراكات Anthropic، استخدم تدفق **setup-token**. تم تقييد استخدام اشتراك Anthropic خارج Claude Code لبعض المستخدمين في الماضي، لذا تعامل معه على أنه مخاطر اختيار المستخدم وتحقق من سياسة Anthropic الحالية بنفسك. OAuth الخاص بـ OpenAI Codex مدعوم صراحةً للاستخدام في الأدوات الخارجية مثل OpenClaw. تشرح هذه الصفحة: بالنسبة لـ Anthropic في الإنتاج، تعد مصادقة مفتاح API المسار الأكثر أمانًا والمُوصى به مقارنة بمصادقة setup-token للاشتراك.

-   كيفية عمل **تبادل الرمز المميز** لـ OAuth (PKCE)
-   أين يتم **تخزين** الرموز المميزة (ولماذا)
-   كيفية التعامل مع **حسابات متعددة** (ملفات تعريف + تجاوزات لكل جلسة)

يدعم OpenClaw أيضًا **إضافات مقدمي الخدمة** التي تشحن تدفقات OAuth أو API‑key الخاصة بها. شغلها عبر:

```bash
openclaw models auth login --provider <id>
```

## مصرف الرموز المميزة (سبب وجوده)

عادةً ما تصك مقدمي خدمة OAuth **رمز تحديث جديد** أثناء عمليات تسجيل الدخول/التحديث. يمكن لبعض المقدمين (أو عملاء OAuth) إبطال رموز التحديث الأقدم عند إصدار رمز جديد لنفس المستخدم/التطبيق. العرض العملي:

-   تقوم بتسجيل الدخول عبر OpenClaw *وأيضًا* عبر Claude Code / Codex CLI → يتم "تسجيل الخروج" لأحدهم بشكل عشوائي لاحقًا

لتقليل ذلك، يعامل OpenClaw `auth-profiles.json` على أنه **مصرف للرموز المميزة**:

-   يقرأ وقت التشغيل بيانات الاعتماد من **مكان واحد**
-   يمكننا الاحتفاظ بملفات تعريف متعددة وتوجيهها بشكل حتمي

## التخزين (أين تعيش الرموز المميزة)

يتم تخزين الأسرار **لكل وكيل**:

-   ملفات تعريف المصادقة (OAuth + مفاتيح API + مراجع اختيارية على مستوى القيمة): `~/.openclaw/agents//agent/auth-profiles.json`
-   ملف التوافق القديم: `~/.openclaw/agents//agent/auth.json` (يتم مسح إدخالات `api_key` الثابتة عند اكتشافها)

ملف الاستيراد القديم فقط (لا يزال مدعومًا، ولكنه ليس المخزن الرئيسي):

-   `~/.openclaw/credentials/oauth.json` (يتم استيراده إلى `auth-profiles.json` عند أول استخدام)

كل ما سبق يحترم أيضًا `$OPENCLAW_STATE_DIR` (تجاوز دليل الحالة). المرجع الكامل: [/gateway/configuration](../gateway/configuration.md#auth-storage-oauth--api-keys) لمراجع الأسرار الثابتة وسلوك تنشيط لقطة وقت التشغيل، راجع [إدارة الأسرار](../gateway/secrets.md).

## Anthropic setup-token (مصادقة الاشتراك)

> **⚠️** دعم Anthropic setup-token هو توافق تقني، وليس ضمانًا للسياسة. قام Anthropic بحظر بعض استخدامات الاشتراك خارج Claude Code في الماضي. قرر بنفسك ما إذا كنت ستستخدم مصادقة الاشتراك، وتحقق من شروط Anthropic الحالية.

 شغل `claude setup-token` على أي جهاز، ثم الصقه في OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

إذا قمت بإنشاء الرمز المميز في مكان آخر، فالصقه يدويًا:

```bash
openclaw models auth paste-token --provider anthropic
```

تحقق:

```bash
openclaw models status
```

## تبادل OAuth (كيف يعمل تسجيل الدخول)

يتم تنفيذ تدفقات تسجيل الدخول التفاعلية لـ OpenClaw في `@mariozechner/pi-ai` وتوصيلها في المعالجات/الأوامر.

### Anthropic setup-token

شكل التدفق:

1.  شغل `claude setup-token`
2.  الصق الرمز المميز في OpenClaw
3.  تخزينه كملف تعريف مصادقة رمز مميز (بدون تحديث)

مسار المعالج هو `openclaw onboard` → خيار المصادقة `setup-token` (Anthropic).

### OpenAI Codex (ChatGPT OAuth)

OAuth الخاص بـ OpenAI Codex مدعوم صراحةً للاستخدام خارج Codex CLI، بما في ذلك سير عمل OpenClaw. شكل التدفق (PKCE):

1.  إنشاء تحقق PKCE/تحدي + `state` عشوائي
2.  افتح `https://auth.openai.com/oauth/authorize?...`
3.  محاولة التقاط رد الاتصال على `http://127.0.0.1:1455/auth/callback`
4.  إذا تعذر ربط رد الاتصال (أو كنت بعيدًا/بدون واجهة)، الصق عنوان URL/الكود لإعادة التوجيه
5.  التبادل على `https://auth.openai.com/oauth/token`
6.  استخراج `accountId` من رمز الوصول وتخزين `{ access, refresh, expires, accountId }`

مسار المعالج هو `openclaw onboard` → خيار المصادقة `openai-codex`.

## التحديث + انتهاء الصلاحية

تقوم ملفات التعريف بتخزين طابع زمني `expires`. في وقت التشغيل:

-   إذا كان `expires` في المستقبل → استخدم رمز الوصول المخزن
-   إذا انتهت صلاحيته → قم بالتحديث (تحت قفل ملف) واستبدال بيانات الاعتماد المخزنة

تدفق التحديث تلقائي؛ بشكل عام لا تحتاج إلى إدارة الرموز المميزة يدويًا.

## حسابات متعددة (ملفات تعريف) + التوجيه

نموذجان:

### 1) مفضل: وكلاء منفصلون

إذا كنت تريد ألا يتفاعل "الشخصي" و"العمل" أبدًا، استخدم وكلاء معزولين (جلسات منفصلة + بيانات اعتماد + مساحة عمل):

```bash
openclaw agents add work
openclaw agents add personal
```

ثم قم بتكوين المصادقة لكل وكيل (المعالج) وقم بتوجيه الدردشات إلى الوكيل الصحيح.

### 2) متقدم: ملفات تعريف متعددة في وكيل واحد

يدعم `auth-profiles.json` معرفات ملفات تعريف متعددة لنفس مقدم الخدمة. اختر ملف التعريف المستخدم:

-   عالميًا عبر ترتيب التكوين (`auth.order`)
-   لكل جلسة عبر `/model ...@`

مثال (تجاوز لكل جلسة):

-   `/model Opus@anthropic:work`

كيفية معرفة معرفات الملفات الشخصية الموجودة:

-   `openclaw channels list --json` (يعرض `auth[]`)

الوثائق ذات الصلة:

-   [/concepts/model-failover](./model-failover.md) (قواعد التناوب + التهدئة)
-   [/tools/slash-commands](../tools/slash-commands.md) (واجهة الأوامر)

[مساحة عمل الوكيل](./agent-workspace.md)[التهيئة الأولية](../start/bootstrapping.md)