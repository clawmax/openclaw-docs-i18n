title: "أوامر ذاكرة OpenClaw CLI للفهرسة الدلالية والبحث"
description: "تعلم كيفية إدارة الذاكرة الدلالية باستخدام أوامر CLI: التحقق من الحالة، إعادة الفهرسة القسرية، والبحث في المحتوى المفهرس باستخدام إضافة الذاكرة."
keywords: ["openclaw cli", "الذاكرة الدلالية", "فهرسة الذاكرة", "بحث الذاكرة", "أوامر cli", "إضافة الذاكرة", "بحث متجهي", "ذاكرة الوكيل"]
---

  أوامر CLI

  
# memory

إدارة فهرسة و بحث الذاكرة الدلالية. يتم توفيرها بواسطة إضافة الذاكرة النشطة (الافتراضي: `memory-core`؛ اضبط `plugins.slots.memory = "none"` لتعطيلها). متعلق:

-   مفهوم الذاكرة: [الذاكرة](../concepts/memory.md)
-   الإضافات: [الإضافات](../tools/plugin.md)

## أمثلة

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory index --force
openclaw memory search "meeting notes"
openclaw memory search --query "deployment" --max-results 20
openclaw memory status --json
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## الخيارات

`memory status` و `memory index`:

-   `--agent `: تحديد نطاق لوكيل واحد. بدونها، تعمل هذه الأوامر لكل وكيل مُهيأ؛ إذا لم يتم تكوين قائمة وكلاء، فإنها تعود إلى الوكيل الافتراضي.
-   `--verbose`: إصدار سجلات مفصلة أثناء عمليات الفحص والفهرسة.

`memory status`:

-   `--deep`: فحص توفر المتجهات والتضمين.
-   `--index`: تشغيل إعادة فهرسة إذا كان المخزن غير نظيف (يتضمن `--deep`).
-   `--json`: طباعة الإخراج بتنسيق JSON.

`memory index`:

-   `--force`: فرض إعادة فهرسة كاملة.

`memory search`:

-   إدخال الاستعلام: مرر إما `[query]` كمعامل موضعي أو `--query `.
-   إذا تم توفير كليهما، فإن `--query` هو الفائز.
-   إذا لم يتم توفير أي منهما، يخرج الأمر مع خطأ.
-   `--agent `: تحديد نطاق لوكيل واحد (الافتراضي: الوكيل الافتراضي).
-   `--max-results `: تحديد عدد النتائج المرجعة.
-   `--min-score `: تصفية المطابقات ذات الدرجة المنخفضة.
-   `--json`: طباعة النتائج بتنسيق JSON.

ملاحظات:

-   `memory index --verbose` تطبع تفاصيل لكل مرحلة (المزود، النموذج، المصادر، نشاط الدفعة).
-   `memory status` تتضمن أي مسارات إضافية تم تكوينها عبر `memorySearch.extraPaths`.
-   إذا تم تكوين حقول مفتاح واجهة برمجة تطبيقات الذاكرة البعيدة الفعالة كـ SecretRefs، فإن الأمر يحل تلك القيم من لقطة البوابة النشطة. إذا كانت البوابة غير متاحة، يفشل الأمر بسرعة.
-   ملاحظة حول اختلاف إصدار البوابة: يتطلب مسار هذا الأمر بوابة تدعم `secrets.resolve`؛ البوابات الأقدم ترجع خطأ طريقة غير معروفة.

[logs](./logs.md)[message](./message.md)

---