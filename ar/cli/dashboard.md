title: "استخدام وأوامر لوحة تحكم OpenClaw CLI"
description: "تعلم كيفية فتح لوحة تحكم واجهة التحكم OpenClaw عبر CLI، وإدارة أمان الرمز المميز، واستخدام علم --no-open. يتضمن ملاحظات حول التعامل مع SecretRef."
keywords: ["openclaw dashboard", "لوحة تحكم cli", "واجهة التحكم", "رمز مصادقة gateway", "secretref", "أمان cli", "openclaw cli"]
---

  أوامر CLI

  
# dashboard

افتح واجهة التحكم باستخدام مصادقتك الحالية.

```bash
openclaw dashboard
openclaw dashboard --no-open
```

ملاحظات:

-   `dashboard` يحل تكوينات `gateway.auth.token` من نوع SecretRef عندما يكون ذلك ممكنًا.
-   للرموز المميزة المدارة بواسطة SecretRef (المحلولة أو غير المحلولة)، يقوم `dashboard` بطباعة/نسخ/فتح عنوان URL غير مميز لتجنب كشف الأسرار الخارجية في ناتج الطرفية، أو سجل الحافظة، أو وسائط تشغيل المتصفح.
-   إذا كان `gateway.auth.token` مدار بواسطة SecretRef ولكنه غير محلول في مسار هذا الأمر، فإن الأمر يطبع عنوان URL غير مميز وتوجيهات علاجية صريحة بدلاً من تضمين عنصر نائب لرمز مميز غير صالح.

[daemon](./daemon.md)[devices](./devices.md)