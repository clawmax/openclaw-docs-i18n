title: "أوامر OpenClaw CLI للنماذج لاكتشاف النماذج وتكوينها"
description: "تعلم استخدام أوامر OpenClaw CLI للنماذج للمسح، وتعيين النماذج الافتراضية، وتكوين النماذج الاحتياطية، وإدارة ملفات تعريف المصادقة."
keywords: ["openclaw cli", "تكوين النموذج", "اكتشاف النموذج", "أوامر cli", "ملفات تعريف المصادقة", "أسماء مستعارة للنماذج", "نماذج احتياطية", "نماذج openclaw"]
---

  أوامر CLI

  
# نماذج

اكتشاف النماذج، ومسحها، وتكوينها (النموذج الافتراضي، النماذج الاحتياطية، ملفات تعريف المصادقة). مواضيع ذات صلة:

-   مقدمي الخدمة + النماذج: [النماذج](../providers/models.md)
-   إعداد مصادقة مقدم الخدمة: [بدء الاستخدام](../start/getting-started.md)

## الأوامر الشائعة

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

يعرض `openclaw models status` النموذج الافتراضي/الاحتياطيات المحللة بالإضافة إلى نظرة عامة على المصادقة. عندما تكون لقطات استخدام مقدم الخدمة متاحة، يتضمن قسم حالة OAuth/الرمز الرمزي رؤوس استخدام مقدم الخدمة. أضف `--probe` لتشغيل اختبارات مصادقة حية ضد كل ملف تعريف لمقدم خدمة مُكون. الاختبارات هي طلبات حقيقية (قد تستهلك الرموز وتُطلق حدود المعدل). استخدم `--agent ` لفحص حالة النموذج/المصادقة لعامل مُكون. عند حذفه، يستخدم الأمر `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` إذا تم تعيينه، وإلا العامل الافتراضي المُكون. ملاحظات:

-   يقبل `models set <model-or-alias>` `provider/model` أو اسمًا مستعارًا.
-   يتم تحليل مراجع النماذج عن طريق التقسيم على **أول** `/`. إذا كان معرف النموذج يتضمن `/` (على طراز OpenRouter)، قم بتضمين بادئة مقدم الخدمة (مثال: `openrouter/moonshotai/kimi-k2`).
-   إذا حذفت مقدم الخدمة، تعامل OpenClaw مع الإدخال كاسم مستعار أو نموذج لـ **مقدم الخدمة الافتراضي** (يعمل فقط عندما لا يوجد `/` في معرف النموذج).
-   قد يُظهر `models status` `marker()` في إخراج المصادقة للعناصر النائبة غير السرية (على سبيل المثال `OPENAI_API_KEY`, `secretref-managed`, `minimax-oauth`, `qwen-oauth`, `ollama-local`) بدلاً من إخفائها كأسرار.

### models status

خيارات:

-   `--json`
-   `--plain`
-   `--check` (الخروج 1=منتهي/مفقود، 2=على وشك الانتهاء)
-   `--probe` (اختبار حي لملفات تعريف المصادقة المُكونة)
-   `--probe-provider ` (اختبار مقدم خدمة واحد)
-   `--probe-profile ` (كرر أو معرفات ملفات تعريف مفصولة بفواصل)
-   `--probe-timeout `
-   `--probe-concurrency `
-   `--probe-max-tokens `
-   `--agent ` (معرف العامل المُكون؛ يتجاوز `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`)

## الأسماء المستعارة + النماذج الاحتياطية

```bash
openclaw models aliases list
openclaw models fallbacks list
```

## ملفات تعريف المصادقة

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

يشغل `models auth login` سير عمل مصادقة مكون إضافي لمقدم الخدمة (OAuth/مفتاح API). استخدم `openclaw plugins list` لمعرفة مقدمي الخدمة المثبتين. ملاحظات:

-   `setup-token` يطلب قيمة رمز الإعداد (انشئه باستخدام `claude setup-token` على أي جهاز).
-   `paste-token` يقبل سلسلة رمز تم إنشاؤها في مكان آخر أو من الأتمتة.
-   ملاحظة سياسة Anthropic: دعم رمز الإعداد هو توافق تقني. قامت Anthropic بحظر بعض استخدامات الاشتراك خارج Claude Code في الماضي، لذا تحقق من الشروط الحالية قبل استخدامه على نطاق واسع.

[message](./message.md)[node](./node.md)

---