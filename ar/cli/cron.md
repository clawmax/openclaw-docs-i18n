title: "إدارة مهام Cron لجدولة OpenClaw Gateway"
description: "تعلم كيفية إضافة وتحرير وإدارة مهام cron لـ OpenClaw CLI. قم بتكوين التسليم والاحتفاظ والسياق خفيف الوزن للمهام الآلية."
keywords: ["openclaw cron", "مهام cron", "جدولة gateway", "أتمتة cli", "جدولة المهام", "أتمتة المهام", "إدارة cron", "مهام معزولة"]
---

  أوامر CLI

  
# cron

إدارة مهام cron لجدولة Gateway. ذات صلة:

-   مهام Cron: [مهام Cron](../automation/cron-jobs.md)

نصيحة: قم بتشغيل `openclaw cron --help` للحصول على سطح الأمر الكامل. ملاحظة: المهام المعزولة `cron add` تستخدم افتراضيًا تسليم `--announce`. استخدم `--no-deliver` للحفاظ على الناتج داخليًا. يبقى `--deliver` كاسم مستعار مهمل لـ `--announce`. ملاحظة: المهام لمرة واحدة (`--at`) تحذف بعد النجاح افتراضيًا. استخدم `--keep-after-run` للاحتفاظ بها. ملاحظة: المهام المتكررة تستخدم الآن التراجع الأسي بعد الأخطاء المتتالية (30s → 1m → 5m → 15m → 60m)، ثم تعود إلى الجدول الطبيعي بعد التشغيل الناجح التالي. ملاحظة: يتم التحكم في الاحتفاظ/التقليم في التكوين:

-   `cron.sessionRetention` (الافتراضي `24h`) يقوم بتقليم جلسات التشغيل المعزولة المكتملة.
-   `cron.runLog.maxBytes` + `cron.runLog.keepLines` يقومان بتقليم `~/.openclaw/cron/runs/.jsonl`.

## التعديلات الشائعة

تحديث إعدادات التسليم دون تغيير الرسالة:

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

تعطيل التسليم لمهمة معزولة:

```bash
openclaw cron edit <job-id> --no-deliver
```

تمكين سياق التمهيد خفيف الوزن لمهمة معزولة:

```bash
openclaw cron edit <job-id> --light-context
```

الإعلان إلى قناة محددة:

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```

إنشاء مهمة معزولة مع سياق تمهيد خفيف الوزن:

```bash
openclaw cron add \
  --name "تقرير الصباح خفيف الوزن" \
  --cron "0 7 * * *" \
  --session isolated \
  --message "لخص التحديثات الليلية." \
  --light-context \
  --no-deliver
```

`--light-context` ينطبق على المهام المعزولة للوكيل فقط. بالنسبة لتشغيلات cron، يحتفض الوضع خفيف الوزن بسياق التمهيد فارغًا بدلاً من حقن مجموعة التمهيد الكاملة لمساحة العمل.

[configure](./configure.md)[daemon](./daemon.md)

---