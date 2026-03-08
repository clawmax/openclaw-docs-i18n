title: "دليل اختيار نماذج OpenClaw وتهيئتها وإدارتها"
description: "تعلّم كيف يختار OpenClaw النماذج، ويهيئ النماذج الاحتياطية، ويدرّم مزودي الخدمة. قم بإعداد النماذج الأساسية، وتعامل مع الفشل التلقائي، واستخدم أوامر CLI للتحكم في النماذج."
keywords: ["نماذج openclaw", "اختيار النموذج", "النموذج الاحتياطي", "تهيئة النموذج", "openrouter scan", "أوامر cli للنماذج", "مزودو النماذج", "models.json"]
---

  مفاهيم النماذج

  
# أوامر CLI للنماذج

راجع [/concepts/model-failover](./model-failover.md) لمعرفة تدوير ملفات تعريف المصادقة، وفترات التهدئة، وكيفية تفاعل ذلك مع النماذج الاحتياطية. نظرة عامة سريعة على المزودين + أمثلة: [/concepts/model-providers](./model-providers.md).

## كيف يعمل اختيار النموذج

يختار OpenClaw النماذج بهذا الترتيب:

1.  النموذج **الأساسي** (`agents.defaults.model.primary` أو `agents.defaults.model`).
2.  **النماذج الاحتياطية** في `agents.defaults.model.fallbacks` (بالترتيب).
3.  **الفشل التلقائي لمصادقة المزود** يحدث داخل مزود قبل الانتقال إلى النموذج التالي.

ما يتعلق بالموضوع:

-   `agents.defaults.models` هي القائمة المسموح بها/الكتالوج للنماذج التي يمكن لـ OpenClaw استخدامها (بالإضافة إلى الأسماء المستعارة).
-   `agents.defaults.imageModel` يُستخدم **فقط عندما** لا يستطيع النموذج الأساسي قبول الصور.
-   يمكن للإعدادات الافتراضية لكل وكيل تجاوز `agents.defaults.model` عبر `agents.list[].model` بالإضافة إلى الربط (انظر [/concepts/multi-agent](./multi-agent.md)).

## سياسة النماذج السريعة

-   اضبط نموذجك الأساسي على أقوى نموذج من الجيل الأخير متاح لك.
-   استخدم النماذج الاحتياطية للمهام الحساسة للتكلفة/وقت الاستجابة والدردشة ذات الأهمية الأقل.
-   بالنسبة للوكلاء المدعومين بالأدوات أو المدخلات غير الموثوقة، تجنب مستويات النماذج الأقدم/الأضعف.

## معالج الإعداد (موصى به)

إذا كنت لا تريد تحرير الإعداد يدويًا، قم بتشغيل معالج التعريف:

```bash
openclaw onboard
```

يمكنه إعداد النموذج + المصادقة لمزودين شائعين، بما في ذلك **اشتراك OpenAI Code (Codex)** (OAuth) و **Anthropic** (مفتاح API أو `claude setup-token`).

## مفاتيح التهيئة (نظرة عامة)

-   `agents.defaults.model.primary` و `agents.defaults.model.fallbacks`
-   `agents.defaults.imageModel.primary` و `agents.defaults.imageModel.fallbacks`
-   `agents.defaults.models` (القائمة المسموح بها + الأسماء المستعارة + معلمات المزود)
-   `models.providers` (مزودون مخصصون مكتوبون في `models.json`)

يتم توحيد مراجع النماذج إلى أحرف صغيرة. الأسماء المستعارة للمزودين مثل `z.ai/*` تُوحّد إلى `zai/*`. أمثلة تهيئة المزودين (بما في ذلك OpenCode Zen) موجودة في [/gateway/configuration](../gateway/configuration.md#opencode-zen-multi-model-proxy).

## "النموذج غير مسموح به" (ولماذا تتوقف الردود)

إذا تم تعيين `agents.defaults.models`، فإنها تصبح **القائمة المسموح بها** لـ `/model` وللتجاوزات الخاصة بالجلسة. عندما يختار المستخدم نموذجًا غير موجود في تلك القائمة المسموح بها، يعيد OpenClaw:

```bash
Model "provider/model" is not allowed. Use /model to list available models.
```

يحدث هذا **قبل** إنشاء رد عادي، لذا قد تشعر أن الرسالة "لم ترد". الإصلاح هو إما:

-   إضافة النموذج إلى `agents.defaults.models`، أو
-   مسح القائمة المسموح بها (إزالة `agents.defaults.models`)، أو
-   اختيار نموذج من `/model list`.

مثال على تهيئة القائمة المسموح بها:

```json
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-6": { alias: "Opus" },
    },
  },
}
```

## تبديل النماذج في الدردشة (/model)

يمكنك تبديل النماذج للجلسة الحالية دون إعادة التشغيل:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

ملاحظات:

-   `/model` (و `/model list`) هو منتقي مضغوط ومرقم (عائلة النموذج + المزودون المتاحون).
-   على Discord، يفتح `/model` و `/models` منتقيًا تفاعليًا مع قوائم منسدلة للمزود والنموذج بالإضافة إلى خطوة إرسال.
-   `/model <#>` يختار من ذلك المنتقي.
-   `/model status` هو العرض التفصيلي (مرشحو المصادقة وعند التهيئة، نقطة نهاية المزود `baseUrl` + وضع `api`).
-   يتم تحليل مراجع النماذج عن طريق التقسيم على **أول** `/`. استخدم `provider/model` عند كتابة `/model `.
-   إذا كان معرف النموذج نفسه يحتوي على `/` (على طراز OpenRouter)، يجب عليك تضمين بادئة المزود (مثال: `/model openrouter/moonshotai/kimi-k2`).
-   إذا حذفت المزود، يعامل OpenClaw المدخلات كاسم مستعار أو نموذج لـ **المزود الافتراضي** (يعمل فقط عندما لا يوجد `/` في معرف النموذج).

سلوك/تهيئة الأمر الكامل: [أوامر الشرطة المائلة](../tools/slash-commands.md).

## أوامر CLI

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models` (بدون أمر فرعي) هو اختصار لـ `models status`.

### models list

يعرض النماذج المهيأة افتراضيًا. أعلام مفيدة:

-   `--all`: الكتالوج الكامل
-   `--local`: المزودون المحليون فقط
-   `--provider `: التصفية حسب المزود
-   `--plain`: نموذج واحد في كل سطر
-   `--json`: مخرجات قابلة للقراءة الآلية

### models status

يعرض النموذج الأساسي المُحلّل، والنماذج الاحتياطية، ونموذج الصور، ونظرة عامة على المصادقة للمزودين المهيئين. كما يكشف عن حالة انتهاء صلاحية OAuth لملفات التعريف الموجودة في مخزن المصادقة (يُحذّر خلال 24 ساعة افتراضيًا). `--plain` يطبع فقط النموذج الأساسي المُحلّل. تُظهر حالة OAuth دائمًا (ويتم تضمينها في مخرجات `--json`). إذا لم يكن لمزود مهيأ أي بيانات اعتماد، فإن `models status` يطبع قسم **Missing auth**. يتضمن JSON `auth.oauth` (نافذة التحذير + ملفات التعريف) و `auth.providers` (المصادقة الفعالة لكل مزود). استخدم `--check` للأتمتة (الخروج بـ `1` عند الفقدان/انتهاء الصلاحية، `2` عند انتهاء الصلاحية قريبًا). يعتمد اختيار المصادقة على المزود/الحساب. بالنسبة لخوادم البوابة العاملة دائمًا، تكون مفاتيح API هي الأكثر قابلية للتنبؤ عادةً؛ كما يتم دعم تدفقات رمز الاشتراك. مثال (إعداد رمز Anthropic):

```bash
claude setup-token
openclaw models status
```

## المسح (نماذج OpenRouter المجانية)

`openclaw models scan` يفحص **كتالوج النماذج المجانية** لـ OpenRouter ويمكنه اختياريًا فحص النماذج للتحقق من دعم الأدوات والصور. أعلام رئيسية:

-   `--no-probe`: تخطي الفحص المباشر (البيانات الوصفية فقط)
-   `--min-params `: الحد الأدنى لحجم المعلمات (بالمليارات)
-   `--max-age-days `: تخطي النماذج الأقدم
-   `--provider `: عامل تصفية بادئة المزود
-   `--max-candidates `: حجم قائمة الاحتياطيات
-   `--set-default`: تعيين `agents.defaults.model.primary` إلى الاختيار الأول
-   `--set-image`: تعيين `agents.defaults.imageModel.primary` إلى اختيار الصورة الأول

يتطلب الفحص مفتاح OpenRouter API (من ملفات تعريف المصادقة أو `OPENROUTER_API_KEY`). بدون مفتاح، استخدم `--no-probe` لإدراج المرشحين فقط. يتم ترتيب نتائج المسح حسب:

1.  دعم الصور
2.  زمن انتقال الأداة
3.  حجم السياق
4.  عدد المعلمات

المدخلات

-   قائمة OpenRouter `/models` (عامل التصفية `:free`)
-   يتطلب مفتاح OpenRouter API من ملفات تعريف المصادقة أو `OPENROUTER_API_KEY` (انظر [/environment](../help/environment.md))
-   عوامل تصفية اختيارية: `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
-   ضوابط الفحص: `--timeout`, `--concurrency`

عند التشغيل في TTY، يمكنك اختيار النماذج الاحتياطية بشكل تفاعلي. في الوضع غير التفاعلي، مرر `--yes` لقبول الإعدادات الافتراضية.

## سجل النماذج (models.json)

يتم كتابة المزودين المخصصين في `models.providers` إلى `models.json` تحت دليل الوكيل (افتراضيًا `~/.openclaw/agents//models.json`). يتم دمج هذا الملف افتراضيًا ما لم يتم تعيين `models.mode` إلى `replace`. أولوية وضع الدمج لمطابقة معرفات المزودين:

-   `baseUrl` غير الفارغ الموجود بالفعل في `models.json` للوكيل يفوز.
-   `apiKey` غير الفارغ في `models.json` للوكيل يفوز فقط عندما لا يكون ذلك المزود مُدارًا بواسطة SecretRef في سياق الإعداد/ملف تعريف المصادقة الحالي.
-   يتم تحديث قيم `apiKey` للمزود المُدار بواسطة SecretRef من علامات المصدر (`ENV_VAR_NAME` لمراجع البيئة، `secretref-managed` لمراجع الملف/التنفيذ) بدلاً من استمرار الأسرار المُحلّلة.
-   `apiKey`/`baseUrl` الفارغ أو المفقود في الوكيل يتراجع إلى `models.providers` في الإعداد.
-   يتم تحديث حقول المزود الأخرى من الإعداد وبيانات الكتالوج الموحدة.

ينطبق هذا الاستمرار القائم على العلامات كلما أعاد OpenClaw إنشاء `models.json`، بما في ذلك المسارات التي تعمل بالأوامر مثل `openclaw agent`.

[بدء سريع لمزود النموذج](../providers/models.md)[مزودو النماذج](./model-providers.md)