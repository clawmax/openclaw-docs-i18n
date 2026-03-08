title: "خطة إشراف عملية PTY لتنفيذ أوامر موثوق"
description: "تعرف على خطة PTY والإشراف على العمليات لدورة حياة تنفيذ أوامر طويلة الأمد موثوقة مع ملكية متوقعة وتنظيف."
keywords: ["إشراف عملية pty", "دورة حياة تنفيذ الأمر", "مشرف العمليات", "إنهاء شجرة العمليات", "عداء وكيل سطر الأوامر", "عملية فرعية موثوقة", "إلغاء العملية", "تجارب openclaw"]
---

  التجارب

  
# خطة PTY والإشراف على العمليات

## 1\. المشكلة والهدف

نحتاج إلى دورة حياة موثوقة واحدة لتنفيذ أوامر طويلة الأمد عبر:

-   عمليات `exec` الأمامية
-   عمليات `exec` الخلفية
-   إجراءات المتابعة لـ `process` (`poll`, `log`, `send-keys`, `paste`, `submit`, `kill`, `remove`)
-   العمليات الفرعية لعداء وكيل سطر الأوامر (CLI)

الهدف ليس فقط دعم PTY. الهدف هو ملكية متوقعة، وإلغاء، وانتهاء مهلة، وتنظيف دون استخدام استدلالات مطابقة عمليات غير آمنة.

## 2\. النطاق والحدود

-   احتفظ بالتنفيذ داخليًا في `src/process/supervisor`.
-   لا تنشئ حزمة جديدة لهذا.
-   احتفظ بتوافق السلوك الحالي حيثما كان ذلك عمليًا.
-   لا توسع النطاق لتشمل إعادة تشغيل الطرفية أو استمرارية جلسات نمط tmux.

## 3\. المنفذ في هذا الفرع

### خط الأساس للمشرف موجود بالفعل

-   وحدة المشرف موجودة تحت `src/process/supervisor/*`.
-   وقت تشغيل Exec وعداء CLI يتم توجيههما بالفعل عبر إنشاء وانتظار المشرف.
-   إنهاء السجل (Registry finalization) هو عملية لا رجعة فيها (idempotent).

### ما تم إكماله في هذه المرحلة

1.  عقد أمر PTY صريح

-   `SpawnInput` أصبح الآن اتحادًا مميزًا (discriminated union) في `src/process/supervisor/types.ts`.
-   عمليات PTY تتطلب `ptyCommand` بدلاً من إعادة استخدام `argv` العام.
-   المشرف لم يعد يعيد بناء سلاسل أوامر PTY من وصلات argv في `src/process/supervisor/supervisor.ts`.
-   وقت تشغيل Exec يمرر الآن `ptyCommand` مباشرة في `src/agents/bash-tools.exec-runtime.ts`.

2.  فصل الأنواع في طبقة العملية

-   أنواع المشرف لم تعد تستورد `SessionStdin` من الوكلاء (agents).
-   عقد stdin المحلي للعملية موجود في `src/process/supervisor/types.ts` (`ManagedRunStdin`).
-   المحولات (Adapters) تعتمد الآن فقط على أنواع مستوى العملية:
    -   `src/process/supervisor/adapters/child.ts`
    -   `src/process/supervisor/adapters/pty.ts`

3.  تحسين ملكية دورة حياة أداة العملية

-   `src/agents/bash-tools.process.ts` يطلب الآن الإلغاء عبر المشرف أولاً.
-   `process kill/remove` تستخدم الآن إنهاء شجرة العمليات كخيار احتياطي عندما يفشل البحث عن المشرف.
-   `remove` تحتفظ بسلوك الإزالة الحتمي من خلال إسقاط إدخالات الجلسة قيد التشغيل فور طلب الإنهاء.

4.  مصدر واحد للقيم الافتراضية لمراقب النظام (watchdog)

-   تمت إضافة القيم الافتراضية المشتركة في `src/agents/cli-watchdog-defaults.ts`.
-   `src/agents/cli-backends.ts` تستهلك القيم الافتراضية المشتركة.
-   `src/agents/cli-runner/reliability.ts` تستهلك نفس القيم الافتراضية المشتركة.

5.  تنظيف المساعدين الميتين

-   تمت إزالة مسار المساعد غير المستخدم `killSession` من `src/agents/bash-tools.shared.ts`.

6.  إضافة اختبارات المسار المباشر للمشرف

-   تمت إضافة `src/agents/bash-tools.process.supervisor.test.ts` لتغطية توجيه kill و remove عبر إلغاء المشرف.

7.  إصلاحات فجوات الموثوقية المكتملة

-   `src/agents/bash-tools.process.ts` يستخدم الآن إنهاء عملية على مستوى نظام التشغيل الحقيقي كخيار احتياطي عندما يفشل البحث عن المشرف.
-   `src/process/supervisor/adapters/child.ts` يستخدم الآن دلالات إنهاء شجرة العمليات لمسارات القتل الافتراضية للإلغاء/انتهاء المهلة.
-   تمت إضافة أداة شجرة العمليات المشتركة في `src/process/kill-tree.ts`.

8.  إضافة تغطية لحالات حافة عقد PTY

-   تمت إضافة `src/process/supervisor/supervisor.pty-command.test.ts` لإعادة توجيه أمر PTY حرفيًا ورفض الأمر الفارغ.
-   تمت إضافة `src/process/supervisor/adapters/child.test.ts` لسلوك قتل شجرة العمليات في إلغاء محول الطفل (child adapter).

## 4\. الفجوات والقرارات المتبقية

### حالة الموثوقية

تم إغلاق فجوتي الموثوقية المطلوبتين لهذه المرحلة الآن:

-   `process kill/remove` لديها الآن خيار إنهاء حقيقي على مستوى نظام التشغيل كخيار احتياطي عندما يفشل البحث عن المشرف.
-   إلغاء/انتهاء مهلة الطفل يستخدم الآن دلالات قتل شجرة العمليات للمسار الافتراضي للقتل.
-   تمت إضافة اختبارات انحدار لكلا السلوكين.

### المتانة والمصالحة عند بدء التشغيل

تم تعريف سلوك إعادة التشغيل الآن صراحةً على أنه دورة حياة في الذاكرة فقط.

-   `reconcileOrphans()` تبقى بدون تأثير (no-op) في `src/process/supervisor/supervisor.ts` عن قصد.
-   لا يتم استعادة العمليات النشطة بعد إعادة تشغيل العملية.
-   هذا الحد مقصود في مرحلة التنفيذ هذه لتجنب مخاطر الاستمرارية الجزئية.

### متابعات قابلية الصيانة

1.  `runExecProcess` في `src/agents/bash-tools.exec-runtime.ts` لا تزال تتعامل مع مسؤوليات متعددة ويمكن تقسيمها إلى مساعدين مركزين في متابعة لاحقة.

## 5\. خطة التنفيذ

مرحلة التنفيذ للعناصر المطلوبة من الموثوقية والعقد اكتملت. تم إكمال:

-   خيار إنهاء حقيقي احتياطي لـ `process kill/remove`
-   إلغاء شجرة العمليات للمسار الافتراضي لقتل محول الطفل
-   اختبارات انحدار للقتل الاحتياطي ومسار قتل محول الطفل
-   اختبارات حالات حافة أمر PTY تحت `ptyCommand` الصريح
-   حد إعادة التشغيل الصريح في الذاكرة مع `reconcileOrphans()` بدون تأثير عن قصد

متابعة اختيارية:

-   تقسيم `runExecProcess` إلى مساعدين مركزين دون انحراف في السلوك

## 6\. خريطة الملفات

### مشرف العمليات

-   `src/process/supervisor/types.ts` تم تحديثه بإدخال إنشاء مميز وعقد stdin محلي للعملية.
-   `src/process/supervisor/supervisor.ts` تم تحديثه لاستخدام `ptyCommand` صريح.
-   `src/process/supervisor/adapters/child.ts` و `src/process/supervisor/adapters/pty.ts` تم فصلهما عن أنواع الوكلاء (agents).
-   `src/process/supervisor/registry.ts` إنهاء لا رجعة فيه (idempotent finalize) لم يتغير وتم الاحتفاظ به.

### تكامل Exec والعملية

-   `src/agents/bash-tools.exec-runtime.ts` تم تحديثه لتمرير أمر PTY صراحة والاحتفاظ بالمسار الاحتياطي.
-   `src/agents/bash-tools.process.ts` تم تحديثه للإلغاء عبر المشرف مع إنهاء شجرة العمليات الحقيقي كخيار احتياطي.
-   `src/agents/bash-tools.shared.ts` تمت إزالة مسار المساعد المباشر للقتل.

### موثوقية سطر الأوامر (CLI)

-   `src/agents/cli-watchdog-defaults.ts` تمت إضافته كخط أساس مشترك.
-   `src/agents/cli-backends.ts` و `src/agents/cli-runner/reliability.ts` تستهلكان الآن نفس القيم الافتراضية.

## 7\. تشغيل التحقق في هذه المرحلة

اختبارات الوحدة:

-   `pnpm vitest src/process/supervisor/registry.test.ts`
-   `pnpm vitest src/process/supervisor/supervisor.test.ts`
-   `pnpm vitest src/process/supervisor/supervisor.pty-command.test.ts`
-   `pnpm vitest src/process/supervisor/adapters/child.test.ts`
-   `pnpm vitest src/agents/cli-backends.test.ts`
-   `pnpm vitest src/agents/bash-tools.exec.pty-cleanup.test.ts`
-   `pnpm vitest src/agents/bash-tools.process.poll-timeout.test.ts`
-   `pnpm vitest src/agents/bash-tools.process.supervisor.test.ts`
-   `pnpm vitest src/process/exec.test.ts`

أهداف الاختبار الشامل (E2E):

-   `pnpm vitest src/agents/cli-runner.test.ts`
-   `pnpm vitest run src/agents/bash-tools.exec.pty-fallback.test.ts src/agents/bash-tools.exec.background-abort.test.ts src/agents/bash-tools.process.send-keys.test.ts`

ملاحظة فحص الأنواع:

-   استخدم `pnpm build` (و `pnpm check` للبوابة الكاملة للتدقيق/المستندات) في هذا المستودع. الملاحظات القديمة التي تذكر `pnpm tsgo` أصبحت قديمة.

## 8\. الضمانات التشغيلية المحفوظة

-   سلوك تعزيز بيئة Exec لم يتغير.
-   تدفق الموافقة والقائمة المسموح بها لم يتغير.
-   تعقيم الإخراج وحدود الإخراج لم تتغير.
-   محول PTY لا يزال يضمن تسوية الانتظار على القتل القسري والتخلص من المستمعين.

## 9\. تعريف الإكمال

1.  المشرف هو مالك دورة الحياة للعمليات المدارة.
2.  إنشاء PTY يستخدم عقد أمر صريح دون إعادة بناء argv.
3.  طبقة العملية ليس لديها اعتماد نوعي على طبقة الوكيل لعقود stdin الخاصة بالمشرف.
4.  القيم الافتراضية لمراقب النظام (watchdog) هي من مصدر واحد.
5.  الاختبارات المستهدفة للوحدة والشاملة (e2e) تبقى ناجحة.
6.  حد متانة إعادة التشغيل موثق صراحة أو منفذ بالكامل.

## 10\. الملخص

الفرع لديه الآن شكل إشراف متماسك وأكثر أمانًا:

-   عقد PTY صريح
-   طبقات عملية أنظف
-   مسار إلغاء مدفوع بالمشرف لعمليات العملية
-   إنهاء حقيقي احتياطي عندما يفشل البحث عن المشرف
-   إلغاء شجرة العمليات لمسارات القتل الافتراضية لتشغيل الطفل
-   قيم افتراضية موحدة لمراقب النظام
-   حد إعادة تشغيل صريح في الذاكرة (لا مصالحة للعمليات اليتيمة عبر إعادة التشغيل في هذه المرحلة)

[خطة بوابة الردود المفتوحة](./openresponses-gateway.md)[خطة ربط الجلسة المستقلة عن القناة](./session-binding-channel-agnostic.md)