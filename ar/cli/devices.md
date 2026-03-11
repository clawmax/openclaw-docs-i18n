

  أوامر CLI

  
# الأجهزة

إدارة طلبات اقتران الأجهزة والرموز المميزة المحددة للأجهزة.

## الأوامر

### openclaw devices list

عرض قائمة طلبات الاقتران المعلقة والأجهزة المقترنة.

```bash
openclaw devices list
openclaw devices list --json
```

### openclaw devices remove &lt;deviceId&gt;

إزالة إدخال جهاز مقترن واحد.

```bash
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### openclaw devices clear --yes \[--pending\]

مسح الأجهزة المقترنة بشكل جماعي.

```bash
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### openclaw devices approve \[requestId\] \[--latest\]

الموافقة على طلب اقتران جهاز معلق. إذا تم حذف `requestId`، فإن OpenClaw يوافق تلقائيًا على أحدث طلب معلق.

```bash
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### openclaw devices reject &lt;requestId&gt;

رفض طلب اقتران جهاز معلق.

```bash
openclaw devices reject <requestId>
```

### openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; \[--scope &lt;scope...&gt;\]

تدوير رمز مميز لجهاز لدور محدد (مع تحديث النطاقات اختياريًا).

```bash
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### openclaw devices revoke --device &lt;id&gt; --role &lt;role&gt;

سحب رمز مميز لجهاز لدور محدد.

```bash
openclaw devices revoke --device <deviceId> --role node
```

## الخيارات الشائعة

-   `--url `: عنوان URL لـ WebSocket للبوابة (الافتراضي هو `gateway.remote.url` عند التكوين).
-   `--token `: الرمز المميز للبوابة (إذا لزم الأمر).
-   `--password `: كلمة مرور البوابة (المصادقة بكلمة المرور).
-   `--timeout `: مهلة RPC.
-   `--json`: إخراج JSON (موصى به للبرمجة النصية).

ملاحظة: عند تعيين `--url`، لا يتراجع CLI إلى بيانات اعتماد التكوين أو البيئة. قم بتمرير `--token` أو `--password` صراحةً. يُعد عدم وجود بيانات اعتماد صريحة خطأ.

## ملاحظات

-   يُرجع تدوير الرمز المميز رمزًا جديدًا (حساس). عالجه كسر.
-   تتطلب هذه الأوامر نطاق `operator.pairing` (أو `operator.admin`).
-   أمر `devices clear` محمي عمدًا بـ `--yes`.
-   إذا كان نطاق الاقتران غير متاح على الاتصال المحلي الدائري (ولم يتم تمرير `--url` صراحةً)، يمكن لـ list/approve استخدام تراجع اقتران محلي.

[لوحة التحكم](./dashboard.md)[الدليل](./directory.md)

---