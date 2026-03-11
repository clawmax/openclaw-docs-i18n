

  أوامر CLI

  
# qr

توليد رمز QR للإقران مع iOS ورمز إعداد من تكوين Gateway الحالي.

## الاستخدام

```bash
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws --token '<token>'
```

## الخيارات

-   `--remote`: استخدام `gateway.remote.url` بالإضافة إلى رمز/كلمة مرور البعيد من التكوين
-   `--url `: تجاوز عنوان URL لـ Gateway المستخدم في الحمولة
-   `--public-url `: تجاوز عنوان URL العام المستخدم في الحمولة
-   `--token `: تجاوز رمز Gateway للحمولة
-   `--password `: تجاوز كلمة مرور Gateway للحمولة
-   `--setup-code-only`: طباعة رمز الإعداد فقط
-   `--no-ascii`: تخطي عرض رمز QR ASCII
-   `--json`: إخراج JSON (`setupCode`, `gatewayUrl`, `auth`, `urlSource`)

## ملاحظات

-   `--token` و `--password` متنافيان.
-   مع `--remote`، إذا كانت بيانات الاعتماد البعيدة الفعالة مهيأة كـ SecretRefs ولم تمرر `--token` أو `--password`، فإن الأمر يحلها من لقطة Gateway النشطة. إذا كان Gateway غير متاح، يفشل الأمر بسرعة.
-   بدون `--remote`، يتم حل SecretRefs مصادقة Gateway المحلية عندما لا يتم تمرير تجاوز مصادقة عبر CLI:
    -   `gateway.auth.token` يتم حلها عندما تفوز مصادقة الرمز (وضع صريح `gateway.auth.mode="token"` أو وضع مستنتج حيث لا يفوز مصدر كلمة مرور).
    -   `gateway.auth.password` يتم حلها عندما تفوز مصادقة كلمة المرور (وضع صريح `gateway.auth.mode="password"` أو وضع مستنتج بدون رمز فائز من المصادقة/البيئة).
-   إذا تم تكوين كل من `gateway.auth.token` و `gateway.auth.password` (بما في ذلك SecretRefs) وكان `gateway.auth.mode` غير مضبوط، فإن حل رمز الإعداد يفشل حتى يتم ضبط الوضع بشكل صريح.
-   ملاحظة حول اختلاف إصدار Gateway: يتطلب مسار هذا الأمر Gateway يدعم `secrets.resolve`؛ تَرد Gateways الأقدم خطأ طريقة غير معروفة.
-   بعد المسح، قم بالموافقة على إقران الجهاز باستخدام:
    -   `openclaw devices list`
    -   `openclaw devices approve `

[plugins](./plugins.md)[reset](./reset.md)