title: "إعداد مصادقة بوابة OpenClaw لـ OAuth ومفاتيح API"
description: "تعلم كيفية تكوين المصادقة لبوابة OpenClaw باستخدام مفاتيح API وOAuth ورموز الإعداد. يتضمن خطوات الإعداد، استكشاف الأخطاء وإصلاحها، وإدارة بيانات الاعتماد."
keywords: ["مصادقة openclaw", "مفتاح api للبوابة", "إعداد oauth", "رمز إعداد anthropic", "دلالات بيانات اعتماد المصادقة", "مجس حالة النماذج", "مصادقة secretref", "تدوير مفتاح api"]
---

  التكوين والعمليات

  
# المصادقة

يدعم OpenClaw OAuth ومفاتيح API لمزودي النماذج. بالنسبة لمضيفي البوابة الدائمين التشغيل، تعتبر مفاتيح API عادةً الخيار الأكثر قابلية للتنبؤ. كما يتم دعم عمليات الاشتراك/OAuth عندما تتطابق مع نموذج حساب مزودك. راجع [/concepts/oauth](../concepts/oauth.md) لتدفق OAuth الكامل وتخطيط التخزين. بالنسبة للمصادقة القائمة على SecretRef (مزودي `env`/`file`/`exec`)، راجع [إدارة الأسرار](./secrets.md). لقواعد أهلية بيانات الاعتماد/أسباب الرفض المستخدمة من قبل `models status --probe`، راجع [دلالات بيانات اعتماد المصادقة](../auth-credential-semantics.md).

## الإعداد الموصى به (مفتاح API، لأي مزود)

إذا كنت تشغل بوابة طويلة الأمد، ابدأ بمفتاح API لمزودك المختار. بالنسبة لـ Anthropic تحديدًا، مصادقة مفتاح API هي المسار الآمن ويوصى بها على مصادقة رمز إعداد الاشتراك.

1.  أنشئ مفتاح API في وحدة تحكم مزودك.
2.  ضعه على **مضيف البوابة** (الجهاز الذي يشغل `openclaw gateway`).

```bash
export <PROVIDER>_API_KEY="..."
openclaw models status
```

3.  إذا كانت البوابة تعمل تحت systemd/launchd، يُفضل وضع المفتاح في `~/.openclaw/.env` حتى يتمكن البرنامج الخفي من قراءته:

```bash
cat >> ~/.openclaw/.env <<'EOF'
<PROVIDER>_API_KEY=...
EOF
```

ثم أعد تشغيل البرنامج الخفي (أو أعد تشغيل عملية البوابة الخاصة بك) وتحقق مرة أخرى:

```bash
openclaw models status
openclaw doctor
```

إذا كنت تفضل عدم إدارة متغيرات البيئة بنفسك، يمكن لمعالج الإعداد الأولي تخزين مفاتيح API لاستخدام البرنامج الخفي: `openclaw onboard`. راجع [المساعدة](../help.md) للحصول على تفاصيل حول توريث متغيرات البيئة (`env.shellEnv`, `~/.openclaw/.env`, systemd/launchd).

## Anthropic: رمز الإعداد (مصادقة الاشتراك)

إذا كنت تستخدم اشتراك Claude، فإن تدفق رمز الإعداد مدعوم. شغله على **مضيف البوابة**:

```bash
claude setup-token
```

ثم الصقه في OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

إذا تم إنشاء الرمز على جهاز آخر، قم بلصقه يدويًا:

```bash
openclaw models auth paste-token --provider anthropic
```

إذا رأيت خطأ من Anthropic مثل:

```bash
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

...استخدم مفتاح API من Anthropic بدلاً من ذلك.

> **⚠️** دعم رمز إعداد Anthropic هو توافق تقني فقط. قامت Anthropic بحظر بعض استخدامات الاشتراك خارج Claude Code في الماضي. استخدمه فقط إذا قررت أن مخاطر السياسة مقبولة، وتحقق من شروط Anthropic الحالية بنفسك.

 الإدخال اليدوي للرمز (أي مزود؛ يكتب `auth-profiles.json` + يقوم بتحديث التكوين):

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

كما يتم دعم إشارات ملفات تعريف المصادقة لبيانات الاعتماد الثابتة:

-   يمكن لبيانات اعتماد `api_key` استخدام `keyRef: { source, provider, id }`
-   يمكن لبيانات اعتماد `token` استخدام `tokenRef: { source, provider, id }`

فحص ملائم للأتمتة (يخرج بـ `1` عند انتهاء الصلاحية/الفقدان، `2` عند اقتراب انتهاء الصلاحية):

```bash
openclaw models status --check
```

يتم توثيق نصوص التشغيل الاختيارية (systemd/Termux) هنا: [/automation/auth-monitoring](../automation/auth-monitoring.md)

> يتطلب `claude setup-token` TTY تفاعليًا.

## التحقق من حالة مصادقة النموذج

```bash
openclaw models status
openclaw doctor
```

## سلوك تدوير مفتاح API (البوابة)

يدعم بعض المزودين إعادة محاولة الطلب بمفاتيح بديلة عندما يصطدم استدعاء API بحد معدل المزود.

-   ترتيب الأولوية:
    -   `OPENCLAW_LIVE__KEY` (تجاوز واحد)
    -   `_API_KEYS`
    -   `_API_KEY`
    -   `_API_KEY_*`
-   كما تتضمن مزودي Google `GOOGLE_API_KEY` كخيار احتياطي إضافي.
-   يتم إزالة التكرارات من نفس قائمة المفاتيح قبل الاستخدام.
-   يعيد OpenClaw المحاولة بالمفتاح التالي فقط لأخطاء حد المعدل (على سبيل المثال `429`, `rate_limit`, `quota`, `resource exhausted`).
-   لا يتم إعادة محاولة الأخطاء غير المرتبطة بحد المعدل بمفاتيح بديلة.
-   إذا فشلت جميع المفاتيح، يتم إرجاع الخطأ النهائي من المحاولة الأخيرة.

## التحكم في بيانات الاعتماد المستخدمة

### لكل جلسة (أمر الدردشة)

استخدم `/model <alias-or-id>@` لتثبيت بيانات اعتماد مزود معينة للجلسة الحالية (أمثلة على معرفات الملفات الشخصية: `anthropic:default`, `anthropic:work`). استخدم `/model` (أو `/model list`) لمعرض مختصر؛ استخدم `/model status` للعرض الكامل (المرشحون + ملف تعريف المصادقة التالي، بالإضافة إلى تفاصيل نقطة نهاية المزود عند التكوين).

### لكل وكيل (تجاوز CLI)

قم بتعيين ترتيب تجاوز صريح لملف تعريف المصادقة لوكيل (يتم تخزينه في `auth-profiles.json` الخاص بهذا الوكيل):

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

استخدم `--agent ` لاستهداف وكيل محدد؛ احذفه لاستخدام الوكيل الافتراضي المكون.

## استكشاف الأخطاء وإصلاحها

### "لم يتم العثور على بيانات اعتماد"

إذا كان ملف تعريف رمز Anthropic مفقودًا، شغل `claude setup-token` على **مضيف البوابة**، ثم تحقق مرة أخرى:

```bash
openclaw models status
```

### الرمز على وشك الانتهاء/منتهي الصلاحية

شغل `openclaw models status` لتأكيد أي ملف تعريف على وشك الانتهاء. إذا كان الملف الشخصي مفقودًا، أعد تشغيل `claude setup-token` والصق الرمز مرة أخرى.

## المتطلبات

-   حساب اشتراك Anthropic (لـ `claude setup-token`)
-   تثبيت Claude Code CLI (الأمر `claude` متاح)

[أمثلة التكوين](./configuration-examples.md)[دلالات بيانات اعتماد المصادقة](../auth-credential-semantics.md)

---