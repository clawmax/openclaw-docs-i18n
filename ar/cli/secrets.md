title: "دليل أوامر إدارة الأسرار في OpenClaw CLI"
description: "تعلم كيفية استخدام أوامر openclaw secrets لمراجعة وتكوين وتطبيق إدارة الأسرار، وإعادة تحميل اللقطات، وتأمين وقت التشغيل."
keywords: ["openclaw secrets", "إدارة الأسرار عبر سطر الأوامر", "secretref", "مراجعة الأسرار", "تكوين الأسرار", "تطبيق خطة الأسرار", "إعادة تحميل لقطة وقت التشغيل", "مراجعة أمنية"]
---

  أوامر CLI

  
# secrets

استخدم `openclaw secrets` لإدارة SecretRefs والحفاظ على صحة لقطة وقت التشغيل النشطة. أدوار الأوامر:

-   `reload`: نداء RPC للبوابة (`secrets.reload`) يعيد حل المراجع ويبدل لقطة وقت التشغيل فقط عند النجاح الكامل (بدون كتابة في التكوين).
-   `audit`: مسح للقراءة فقط لمخازن التكوين/المصادقة/النماذج المُنشأة والمخلفات القديمة للبحث عن النص العادي، والمراجع غير المحلولة، وانحراف الأسبقية.
-   `configure`: مخطط تفاعلي لإعداد المزود، وتعيين الهدف، والفحص المسبق (يتطلب TTY).
-   `apply`: تنفيذ خطة محفوظة (`--dry-run` للتحقق فقط)، ثم مسح مخلفات النص العادي المستهدفة.

حلقة التشغيل الموصى بها:

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets audit --check
openclaw secrets reload
```

ملاحظة حول رمز الخروج لـ CI/البوابات:

-   `audit --check` يُرجع `1` عند وجود نتائج.
-   المراجع غير المحلولة تُرجع `2`.

مواضيع ذات صلة:

-   دليل الأسرار: [إدارة الأسرار](../gateway/secrets.md)
-   سطح بيانات الاعتماد: [سطح بيانات اعتماد SecretRef](../reference/secretref-credential-surface.md)
-   دليل الأمان: [الأمان](../gateway/security.md)

## إعادة تحميل لقطة وقت التشغيل

أعد حل مراجع الأسرار وبدل لقطة وقت التشغيل بشكل ذري.

```bash
openclaw secrets reload
openclaw secrets reload --json
```

ملاحظات:

-   يستخدم طريقة RPC للبوابة `secrets.reload`.
-   إذا فشل الحل، تحتفظ البوابة بآخر لقطة جيدة معروفة وتُرجع خطأ (بدون تنشيط جزئي).
-   استجابة JSON تتضمن `warningCount`.

## مراجعة

امسح حالة OpenClaw للبحث عن:

-   تخزين أسرار كنص عادي
-   مراجع غير محلولة
-   انحراف الأسبقية (بيانات اعتماد `auth-profiles.json` تحجب مراجع `openclaw.json`)
-   مخلفات في `agents/*/agent/models.json` المُنشأة (قيم `apiKey` للمزود وعناوين المزود الحساسة)
-   مخلفات قديمة (إدخالات مخزن مصادقة قديم، تذكيرات OAuth)

ملاحظة حول مخلفات العناوين:

-   كشف عنوان المزود الحساس يعتمد على اسم إرشادي (أسماء عناوين المصادقة/بيانات الاعتماد الشائعة وأجزاء منها مثل `authorization`, `x-api-key`, `token`, `secret`, `password`, و `credential`).

```bash
openclaw secrets audit
openclaw secrets audit --check
openclaw secrets audit --json
```

سلوك الخروج:

-   `--check` يخرج برمز غير صفري عند وجود نتائج.
-   المراجع غير المحلولة تخرج برمز غير صفري ذي أولوية أعلى.

أبرز شكل التقرير:

-   `status`: `clean | findings | unresolved`
-   `summary`: `plaintextCount`, `unresolvedRefCount`, `shadowedRefCount`, `legacyResidueCount`
-   رموز النتائج:
    -   `PLAINTEXT_FOUND`
    -   `REF_UNRESOLVED`
    -   `REF_SHADOWED`
    -   `LEGACY_RESIDUE`

## تكوين (مساعد تفاعلي)

أنشئ تغييرات المزود و SecretRef بشكل تفاعلي، وقم بالفحص المسبق، وطبقها اختيارياً:

```bash
openclaw secrets configure
openclaw secrets configure --plan-out /tmp/openclaw-secrets-plan.json
openclaw secrets configure --apply --yes
openclaw secrets configure --providers-only
openclaw secrets configure --skip-provider-setup
openclaw secrets configure --agent ops
openclaw secrets configure --json
```

التدفق:

-   أولاً إعداد المزود (`add/edit/remove` لـ `secrets.providers` aliases).
-   ثانياً تعيين بيانات الاعتماد (اختر الحقول وعيّن مراجع `{source, provider, id}`).
-   أخيراً الفحص المسبق والتطبيق الاختياري.

الأعلام:

-   `--providers-only`: قم بتكوين `secrets.providers` فقط، تخطى تعيين بيانات الاعتماد.
-   `--skip-provider-setup`: تخطى إعداد المزود وقم بتعيين بيانات الاعتماد إلى المزودين الموجودين.
-   `--agent `: حدد نطاق اكتشاف الهدف وكتابة `auth-profiles.json` لمخزن وكيل واحد.

ملاحظات:

-   يتطلب TTY تفاعلي.
-   لا يمكنك الجمع بين `--providers-only` و `--skip-provider-setup`.
-   يستهدف `configure` الحقول الحاملة للأسرار في `openclaw.json` بالإضافة إلى `auth-profiles.json` لنطاق الوكيل المحدد.
-   يدعم `configure` إنشاء تعيينات جديدة لـ `auth-profiles.json` مباشرة في تدفق منتقي الخيارات.
-   السطح المدعوم الأساسي: [سطح بيانات اعتماد SecretRef](../reference/secretref-credential-surface.md).
-   يقوم بالفحص المسبق للحل قبل التطبيق.
-   الخطط المُنشأة تفترض خيارات المسح بشكل افتراضي (`scrubEnv`, `scrubAuthProfilesForProviderTargets`, `scrubLegacyAuthJson` مفعلة جميعها).
-   مسار التطبيق باتجاه واحد لقيم النص العادي الممسوحة.
-   بدون `--apply`، لا يزال CLI يطالب بـ `Apply this plan now?` بعد الفحص المسبق.
-   مع `--apply` (وبدون `--yes`)، يطالب CLI بتأكيد إضافي لا رجعة فيه.

ملاحظة أمان مزود Exec:

-   غالباً ما تعرض التثبيتات عبر Homebrew ملفات ثنائية مرتبطة رمزياً تحت `/opt/homebrew/bin/*`.
-   عيّن `allowSymlinkCommand: true` فقط عند الحاجة لمسارات مدير الحزم الموثوقة، وقم بإقرانها بـ `trustedDirs` (مثال `["/opt/homebrew"]`).
-   على Windows، إذا لم يكن التحقق من ACL متاحاً لمسار مزود، فإن OpenClaw يفشل مغلقاً. للمسارات الموثوقة فقط، عيّن `allowInsecurePath: true` على ذلك المزود لتجاوز فحوصات أمان المسار.

## تطبيق خطة محفوظة

طبق أو قم بالفحص المسبق لخطة مُنشأة مسبقاً:

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --json
```

تفاصيل عقد الخطة (مسارات الهدف المسموحة، قواعد التحقق، ودلالات الفشل):

-   [عقد خطة تطبيق الأسرار](../gateway/secrets-plan-contract.md)

ما قد يقوم `apply` بتحديثه:

-   `openclaw.json` (أهداف SecretRef + عمليات إدراج/حذف المزود)
-   `auth-profiles.json` (مسح الهدف الخاص بالمزود)
-   مخلفات `auth.json` القديمة
-   `~/.openclaw/.env` مفاتيح الأسرار المعروفة التي تم نقل قيمها

## لماذا لا توجد نسخ احتياطية للتراجع

`secrets apply` لا يكتب عمداً نسخاً احتياطية للتراجع تحتوي على قيم النص العادي القديمة. يأتي الأمان من الفحص المسبق الصارم + التطبيق شبه الذري مع استعادة قصوى الجهد في الذاكرة عند الفشل.

## مثال

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

إذا كان `audit --check` لا يزال يُبلغ عن نتائج نص عادي، قم بتحديث مسارات الهدف المتبقية المُبلغ عنها وأعد تشغيل المراجعة.

[CLI Sandbox](./sandbox.md)[security](./security.md)