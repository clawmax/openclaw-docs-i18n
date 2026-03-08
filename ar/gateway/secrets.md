

  التكوين والعمليات

  
# إدارة الأسرار

يدعم OpenClaw إضافة SecretRefs حتى لا تحتاج بيانات الاعتماد المدعومة إلى التخزين كنص عادي في التكوين. النص العادي لا يزال يعمل. استخدام SecretRefs اختياري لكل بيانات اعتماد.

## الأهداف ونموذج وقت التشغيل

يتم تحويل الأسرار إلى لقطة وقت تشغيل في الذاكرة.

-   التحويل يتم بفارغ الصبر أثناء التفعيل، وليس بكسل على مسارات الطلبات.
-   فشل بدء التشغيل بسرعة عندما لا يمكن حل SecretRef نشط بشكل فعال.
-   إعادة التحميل تستخدم التبديل الذري: نجاح كامل، أو الاحتفاظ بلقطة آخر حالة جيدة معروفة.
-   طلبات وقت التشغيل تقرأ فقط من اللقطة النشطة في الذاكرة.

هذا يبقي أعطال موفري الأسرار بعيدًا عن مسارات الطلبات الساخنة.

## تصفية السطح النشط

يتم التحقق من صحة SecretRefs فقط على الأسطح النشطة بشكل فعال.

-   الأسطح الممكّنة: المراجع غير المحلولة تمنع بدء التشغيل/إعادة التحميل.
-   الأسطح غير النشطة: المراجع غير المحلولة لا تمنع بدء التشغيل/إعادة التحميل.
-   المراجع غير النشطة تصدر تشخيصات غير قاتلة بالرمز `SECRETS_REF_IGNORED_INACTIVE_SURFACE`.

أمثلة على الأسطح غير النشطة:

-   إدخالات القناة/الحساب المعطلة.
-   بيانات اعتماد القناة على مستوى عالٍ لا يرثها أي حساب ممكّن.
-   أسطح الأداة/الميزة المعطلة.
-   المفاتيح الخاصة بمزود البحث على الويب التي لم يتم تحديدها بواسطة `tools.web.search.provider`. في الوضع التلقائي (عندما لا يتم تعيين المزود)، تكون المفاتيح الخاصة بالمزود نشطة أيضًا للكشف التلقائي عن المزود.
-   SecretRefs الخاصة بـ `gateway.remote.token` / `gateway.remote.password` تكون نشطة (عندما لا تكون `gateway.remote.enabled` هي `false`) إذا كان أحد هذه الشروط صحيحًا:
    -   `gateway.mode=remote`
    -   تم تكوين `gateway.remote.url`
    -   `gateway.tailscale.mode` هو `serve` أو `funnel` في الوضع المحلي بدون تلك الأسطح البعيدة:
    -   `gateway.remote.token` يكون نشطًا عندما يمكن أن يفوز مصادقة الرمز المميز ولا يتم تكوين رمز مميز بيئي/مصادقة.
    -   `gateway.remote.password` يكون نشطًا فقط عندما يمكن أن تفوز مصادقة كلمة المرور ولا يتم تكوين كلمة مرور بيئية/مصادقة.
-   SecretRef الخاص بـ `gateway.auth.token` يكون غير نشط لحل المصادقة عند بدء التشغيل عندما يتم تعيين `OPENCLAW_GATEWAY_TOKEN` (أو `CLAWDBOT_GATEWAY_TOKEN`)، لأن إدخال الرمز المميز البيئي يفوز لذلك التشغيل.

## تشخيصات سطح مصادقة البوابة

عند تكوين SecretRef على `gateway.auth.token`، أو `gateway.auth.password`، أو `gateway.remote.token`، أو `gateway.remote.password`، يسجل بدء تشغيل/إعادة تحميل البوابة حالة السطح بشكل صريح:

-   `active`: يكون SecretRef جزءًا من سطح المصادقة الفعال ويجب حله.
-   `inactive`: يتم تجاهل SecretRef لهذا التشغيل لأن سطح مصادقة آخر يفوز، أو لأن المصادقة البعيدة معطلة/غير نشطة.

يتم تسجيل هذه الإدخالات بـ `SECRETS_GATEWAY_AUTH_SURFACE` وتتضمن السبب المستخدم من قبل سياسة السطح النشط، حتى تتمكن من رؤية سبب معاملة بيانات الاعتماد على أنها نشطة أو غير نشطة.

## التحقق المسبق للمرجع عند الإعداد

عند تشغيل الإعداد في الوضع التفاعلي واختيار تخزين SecretRef، يقوم OpenClaw بإجراء التحقق المسبق قبل الحفظ:

-   مراجع المتغيرات البيئية: يتحقق من صحة اسم المتغير البيئي ويؤكد أن قيمة غير فارغة مرئية أثناء الإعداد.
-   مراجع المزودين (`file` أو `exec`): يتحقق من اختيار المزود، ويحل `id`، ويفحص نوع القيمة المحلولة.
-   مسار إعادة استخدام البدء السريع: عندما يكون `gateway.auth.token` بالفعل SecretRef، يحله الإعداد قبل بدء تشغيل المسبار/لوحة التحكم (لمراجع `env`، و`file`، و`exec`) باستخدام بوابة الفشل السريع نفسها.

إذا فشل التحقق، يعرض الإعداد الخطأ ويسمح لك بإعادة المحاولة.

## عقد SecretRef

استخدم شكل كائن واحد في كل مكان:

```json
{ source: "env" | "file" | "exec", provider: "default", id: "..." }
```

### source: "env"

```json
{ source: "env", provider: "default", id: "OPENAI_API_KEY" }
```

التحقق من الصحة:

-   يجب أن يتطابق `provider` مع `^[a-z][a-z0-9_-]{0,63}$`
-   يجب أن يتطابق `id` مع `^[A-Z][A-Z0-9_]{0,127}$`

### source: "file"

```json
{ source: "file", provider: "filemain", id: "/providers/openai/apiKey" }
```

التحقق من الصحة:

-   يجب أن يتطابق `provider` مع `^[a-z][a-z0-9_-]{0,63}$`
-   يجب أن يكون `id` مؤشر JSON مطلق (`/...`)
-   الهروب RFC6901 في المقاطع: `~` => `~0`, `/` => `~1`

### source: "exec"

```json
{ source: "exec", provider: "vault", id: "providers/openai/apiKey" }
```

التحقق من الصحة:

-   يجب أن يتطابق `provider` مع `^[a-z][a-z0-9_-]{0,63}$`
-   يجب أن يتطابق `id` مع `^[A-Za-z0-9][A-Za-z0-9._:/-]{0,255}$`

## تكوين المزود

حدد المزودين تحت `secrets.providers`:

```json
{
  secrets: {
    providers: {
      default: { source: "env" },
      filemain: {
        source: "file",
        path: "~/.openclaw/secrets.json",
        mode: "json", // or "singleValue"
      },
      vault: {
        source: "exec",
        command: "/usr/local/bin/openclaw-vault-resolver",
        args: ["--profile", "prod"],
        passEnv: ["PATH", "VAULT_ADDR"],
        jsonOnly: true,
      },
    },
    defaults: {
      env: "default",
      file: "filemain",
      exec: "vault",
    },
    resolution: {
      maxProviderConcurrency: 4,
      maxRefsPerProvider: 512,
      maxBatchBytes: 262144,
    },
  },
}
```

### مزود المتغيرات البيئية

-   قائمة السماح الاختيارية عبر `allowlist`.
-   قيم المتغيرات البيئية المفقودة/الفارغة تفشل في الحل.

### مزود الملف

-   يقرأ الملف المحلي من `path`.
-   `mode: "json"` يتوقع حمولة كائن JSON ويحل `id` كمؤشر.
-   `mode: "singleValue"` يتوقع معرف المرجع `"value"` ويعيد محتويات الملف.
-   يجب أن يمر المسار بفحوصات الملكية/الصلاحيات.
-   ملاحظة إغلاق الفشل لنظام Windows: إذا كان التحقق من ACL غير متاح لمسار ما، يفشل الحل. للمسارات الموثوقة فقط، عيّن `allowInsecurePath: true` على ذلك المزود لتجاوز فحوصات أمان المسار.

### مزود التنفيذ

-   يشغل مسار الملف الثنائي المطلق المكون، بدون shell.
-   افتراضيًا، يجب أن يشير `command` إلى ملف عادي (ليس رابطًا رمزيًا).
-   عيّن `allowSymlinkCommand: true` للسماح بمسارات أوامر الروابط الرمزية (على سبيل المثال، واجهات Homebrew). يقوم OpenClaw بالتحقق من صحة مسار الهدف المحلول.
-   زوّد `allowSymlinkCommand` مع `trustedDirs` لمسارات مدير الحزم (على سبيل المثال `["/opt/homebrew"]`).
-   يدعم المهلة، مهلة عدم الإخراج، حدود بايت الإخراج، قائمة السماح البيئية، والمجلدات الموثوقة.
-   ملاحظة إغلاق الفشل لنظام Windows: إذا كان التحقق من ACL غير متاح لمسار الأمر، يفشل الحل. للمسارات الموثوقة فقط، عيّن `allowInsecurePath: true` على ذلك المزود لتجاوز فحوصات أمان المسار.

حمولة الطلب (stdin):

```json
{ "protocolVersion": 1, "provider": "vault", "ids": ["providers/openai/apiKey"] }
```

حمولة الاستجابة (stdout):

```json
{ "protocolVersion": 1, "values": { "providers/openai/apiKey": "<openai-api-key>" } } // pragma: allowlist secret
```

أخطاء لكل معرف اختيارية:

```json
{
  "protocolVersion": 1,
  "values": {},
  "errors": { "providers/openai/apiKey": { "message": "not found" } }
}
```

## أمثلة تكامل التنفيذ

### 1Password CLI

```json
{
  secrets: {
    providers: {
      onepassword_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/op",
        allowSymlinkCommand: true, // required for Homebrew symlinked binaries
        trustedDirs: ["/opt/homebrew"],
        args: ["read", "op://Personal/OpenClaw QA API Key/password"],
        passEnv: ["HOME"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "onepassword_openai", id: "value" },
      },
    },
  },
}
```

### HashiCorp Vault CLI

```json
{
  secrets: {
    providers: {
      vault_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/vault",
        allowSymlinkCommand: true, // required for Homebrew symlinked binaries
        trustedDirs: ["/opt/homebrew"],
        args: ["kv", "get", "-field=OPENAI_API_KEY", "secret/openclaw"],
        passEnv: ["VAULT_ADDR", "VAULT_TOKEN"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "vault_openai", id: "value" },
      },
    },
  },
}
```

### sops

```json
{
  secrets: {
    providers: {
      sops_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/sops",
        allowSymlinkCommand: true, // required for Homebrew symlinked binaries
        trustedDirs: ["/opt/homebrew"],
        args: ["-d", "--extract", '["providers"]["openai"]["apiKey"]', "/path/to/secrets.enc.json"],
        passEnv: ["SOPS_AGE_KEY_FILE"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "sops_openai", id: "value" },
      },
    },
  },
}
```

## سطح بيانات الاعتماد المدعوم

يتم سرد بيانات الاعتماد المدعومة وغير المدعومة بشكل أساسي في:

-   [سطح بيانات اعتماد SecretRef](../reference/secretref-credential-surface.md)

يتم استبعاد بيانات الاعتماد التي يتم إنشاؤها أثناء التشغيل أو المتغيرة ومواد تحديث OAuth عمدًا من حل SecretRef للقراءة فقط.

## السلوك المطلوب والأولوية

-   حقل بدون مرجع: بدون تغيير.
-   حقل بمرجع: مطلوب على الأسطح النشطة أثناء التفعيل.
-   إذا كان النص العادي والمرجع موجودين معًا، يأخذ المرجع الأولوية على مسارات الأولوية المدعومة.

إشارات التحذير والتدقيق:

-   `SECRETS_REF_OVERRIDES_PLAINTEXT` (تحذير وقت التشغيل)
-   `REF_SHADOWED` (نتيجة تدقيق عندما تأخذ بيانات اعتماد `auth-profiles.json` الأولوية على مراجع `openclaw.json`)

سلوك توافق Google Chat:

-   يأخذ `serviceAccountRef` الأولوية على النص العادي `serviceAccount`.
-   يتم تجاهل القيمة النصية العادية عند تعيين المرجع الشقيق.

## محفزات التفعيل

يتم تشغيل تفعيل السري على:

-   بدء التشغيل (التحقق المسبق بالإضافة إلى التفعيل النهائي)
-   مسار التطبيق الساخن لإعادة تحميل التكوين
-   مسار فحص إعادة التشغيل لإعادة تحميل التكوين
-   إعادة التحميل اليدوي عبر `secrets.reload`

عقد التفعيل:

-   النجاح يبدل اللقطة ذريًا.
-   فشل بدء التشغيل يلغي بدء تشغيل البوابة.
-   فشل إعادة تحميل وقت التشغيل يحتفظ بلقطة آخر حالة جيدة معروفة.

## إشارات التدهور والاستعادة

عند فشل تفعيل وقت إعادة التحميل بعد حالة صحية، يدخل OpenClaw حالة أسرار متدهورة. حدث نظام لمرة واحدة ورموز السجل:

-   `SECRETS_RELOADER_DEGRADED`
-   `SECRETS_RELOADER_RECOVERED`

السلوك:

-   متدهور: يحتفظ وقت التشغيل بلقطة آخر حالة جيدة معروفة.
-   تم الاستعادة: يتم إصداره مرة واحدة بعد التفعيل الناجح التالي.
-   الفشل المتكرر أثناء التدهور بالفعل يسجل تحذيرات ولكن لا يرسل أحداثًا متكررة.
-   فشل بدء التشغيل السريع لا يصدر أحداث تدهور لأن وقت التشغيل لم يصبح نشطًا أبدًا.

## حل مسار الأمر

يمكن لمسارات الأمر اختيار حل SecretRef المدعوم عبر RPC لقطة البوابة. هناك سلوكان واسعان:

-   مسارات الأمر الصارمة (على سبيل المثال مسارات الذاكرة البعيدة `openclaw memory` و `openclaw qr --remote`) تقرأ من اللقطة النشطة وتفشل بسرعة عندما يكون SecretRef مطلوبًا غير متاح.
-   مسارات الأمر للقراءة فقط (على سبيل المثال `openclaw status`، `openclaw status --all`، `openclaw channels status`، `openclaw channels resolve`، وتدفقات إصلاح الطبيب/التكوين للقراءة فقط) تفضل أيضًا اللقطة النشطة، ولكن تتدهور بدلاً من الإلغاء عندما يكون SecretRef مستهدفًا غير متاح في مسار الأمر ذلك.

سلوك القراءة فقط:

-   عندما تعمل البوابة، تقرأ هذه الأوامر من اللقطة النشطة أولاً.
-   إذا كان حل البوابة غير مكتمل أو البوابة غير متاحة، يحاولون التراجع المحلي المستهدف للسطح المحدد للأمر.
-   إذا كان SecretRef مستهدفًا لا يزال غير متاح، يستمر الأمر بإخراج قراءة فقط متدهور وتشخيصات صريحة مثل "مكون ولكن غير متاح في مسار الأمر هذا".
-   هذا السلوك المتدهور محلي للأمر فقط. لا يضعف بدء تشغيل وقت التشغيل، أو إعادة التحميل، أو مسارات الإرسال/المصادقة.

ملاحظات أخرى:

-   تحديث اللقطة بعد تدوير السر الخلفي يتم معالجته بواسطة `openclaw secrets reload`.
-   طريقة RPC للبوابة المستخدمة بواسطة مسارات الأمر هذه: `secrets.resolve`.

## سير عمل التدقيق والتكوين

تدفق المشغل الافتراضي:

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

### secrets audit

تشمل النتائج:

-   قيم نصية عادية في حالة السكون (`openclaw.json`، `auth-profiles.json`، `.env`، و `agents/*/agent/models.json` المُنشأة)
-   بقايا رأس مزود حساسة نصية عادية في إدخالات `models.json` المُنشأة
-   مراجع غير محلولة
-   تظليل الأولوية (أخذ `auth-profiles.json` الأولوية على مراجع `openclaw.json`)
-   بقايا قديمة (`auth.json`، تذكيرات OAuth)

ملاحظة بقايا الرأس:

-   اكتشاف رأس مزود حساس يعتمد على اسم استدلالي (أسماء رؤوس المصادقة/بيانات الاعتماد الشائعة ومقاطع مثل `authorization`، `x-api-key`، `token`، `secret`، `password`، و `credential`).

### secrets configure

مساعد تفاعلي يقوم بـ:

-   تكوين `secrets.providers` أولاً (`env`/`file`/`exec`، إضافة/تحرير/إزالة)
-   يسمح لك بتحديد الحقول الحاملة للأسرار المدعومة في `openclaw.json` بالإضافة إلى `auth-profiles.json` لنطاق وكيل واحد
-   يمكنه إنشاء تعيين `auth-profiles.json` جديد مباشرة في منتقي الهدف
-   يلتقط تفاصيل SecretRef (`source`، `provider`، `id`)
-   يقوم بالتحقق المسبق للحل
-   يمكنه التطبيق فورًا

أوضاع مفيدة:

-   `openclaw secrets configure --providers-only`
-   `openclaw secrets configure --skip-provider-setup`
-   `openclaw secrets configure --agent `

افتراضيات تطبيق `configure`:

-   مسح بيانات الاعتماد الثابتة المطابقة من `auth-profiles.json` للمزودين المستهدفين
-   مسح إدخالات `api_key` الثابتة القديمة من `auth.json`
-   مسح الأسطر السرية المعروفة المطابقة من `<config-dir>/.env`

### secrets apply

تطبيق خطة محفوظة:

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
```

للحصول على تفاصيل عقد الهدف/المسار الصارمة وقواعد الرفض الدقيقة، انظر:

-   [عقد خطة تطبيق الأسرار](./secrets-plan-contract.md)

## سياسة الأمان أحادية الاتجاه

لا يقوم OpenClaw عمدًا بكتابة نسخ احتياطية للتراجع تحتوي على قيم أسرار نصية عادية تاريخية. نموذج الأمان:

-   يجب أن ينجح التحقق المسبق قبل وضع الكتابة
-   يتم التحقق من تفعيل وقت التشغيل قبل الالتزام
-   يطبق التحديثات على الملفات باستبدال الملف الذري واستعادة بأقصى جهد عند الفشل

## ملاحظات توثق المصادقة القديمة

لبيانات الاعتماد الثابتة، لم يعد وقت التشغيل يعتمد على تخزين المصادقة القديمة النصي العادي.

-   مصدر بيانات اعتماد وقت التشغيل هو اللقطة المحلولة في الذاكرة.
-   يتم مسح إدخالات `api_key` الثابتة القديمة عند اكتشافها.
-   يبقى سلوك التوافق المتعلق بـ OAuth منفصلاً.

## ملاحظة واجهة المستخدم على الويب

بعض اتحادات SecretInput أسهل في التكوين في وضع المحرر الخام منه في وضع النموذج.

## وثائق ذات صلة

-   أوامر CLI: [secrets](../cli/secrets.md)
-   تفاصيل عقد الخطة: [عقد خطة تطبيق الأسرار](./secrets-plan-contract.md)
-   سطح بيانات الاعتماد: [سطح بيانات اعتماد SecretRef](../reference/secretref-credential-surface.md)
-   إعداد المصادقة: [المصادقة](./authentication.md)
-   وضع الأمان: [الأمان](./security.md)
-   أولوية البيئة: [المتغيرات البيئية](../help/environment.md)

[دلالات بيانات اعتماد المصادقة](../auth-credential-semantics.md)[عقد خطة تطبيق الأسرار](./secrets-plan-contract.md)