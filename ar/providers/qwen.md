title: "إعداد موفر Qwen في OpenClaw: المصادقة ومعرفات النماذج"
description: "تعلم كيفية تمكين موفر Qwen في OpenClaw والمصادقة عليه واستخدامه للوصول المجاني إلى نماذج Qwen Coder و Vision."
keywords: ["qwen", "openclaw", "موفر النماذج", "المصادقة", "oauth", "qwen coder", "qwen vision", "تكامل واجهة برمجة التطبيقات"]
---

  الموفرون

  
# Qwen

يوفر Qwen تدفق OAuth مجانيًا لنماذج Qwen Coder و Qwen Vision (2000 طلب/يوم، وفقًا لحدود معدل Qwen).

## تمكين الإضافة

```bash
openclaw plugins enable qwen-portal-auth
```

أعد تشغيل البوابة بعد التمكين.

## المصادقة

```bash
openclaw models auth login --provider qwen-portal --set-default
```

هذا الأمر ينفذ تدفق OAuth باستخدام رمز الجهاز من Qwen ويكتب إدخال الموفر في ملف `models.json` الخاص بك (بالإضافة إلى اسم مستعار `qwen` للتبديل السريع).

## معرفات النماذج

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

قم بالتبديل بين النماذج باستخدام:

```bash
openclaw models set qwen-portal/coder-model
```

## إعادة استخدام تسجيل دخول Qwen Code CLI

إذا كنت قد سجلت الدخول مسبقًا باستخدام Qwen Code CLI، فسوف يقوم OpenClaw بمزامنة بيانات الاعتماد من `~/.qwen/oauth_creds.json` عند تحميل مخزن المصادقة. لا يزال يتعين عليك وجود إدخال `models.providers.qwen-portal` (استخدم أمر تسجيل الدخول أعلاه لإنشاء واحد).

## ملاحظات

-   يتم تجديد الرموز المميزة تلقائيًا؛ أعد تشغيل أمر تسجيل الدخول إذا فشل التجديد أو تم إلغاء الوصول.
-   عنوان URL الأساسي الافتراضي: `https://portal.qwen.ai/v1` (يمكن تجاوزه باستخدام `models.providers.qwen-portal.baseUrl` إذا قدم Qwen نقطة نهاية مختلفة).
-   راجع [موفرو النماذج](../concepts/model-providers.md) للاطلاع على القواعد العامة للموفرين.

[Qianfan](./qianfan.md)[Synthetic](./synthetic.md)

---