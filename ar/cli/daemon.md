

  أوامر CLI

  
# daemon

اسم مستعار قديم لأوامر إدارة خدمة البوابة. `openclaw daemon ...` تشير إلى نفس واجهة التحكم في الخدمة مثل أوامر `openclaw gateway ...` للخدمة.

## الاستخدام

```bash
openclaw daemon status
openclaw daemon install
openclaw daemon start
openclaw daemon stop
openclaw daemon restart
openclaw daemon uninstall
```

## الأوامر الفرعية

-   `status`: عرض حالة تثبيت الخدمة وفحص صحة البوابة
-   `install`: تثبيت الخدمة (`launchd`/`systemd`/`schtasks`)
-   `uninstall`: إزالة الخدمة
-   `start`: بدء الخدمة
-   `stop`: إيقاف الخدمة
-   `restart`: إعادة تشغيل الخدمة

## الخيارات الشائعة

-   `status`: `--url`, `--token`, `--password`, `--timeout`, `--no-probe`, `--deep`, `--json`
-   `install`: `--port`, `--runtime <node|bun>`, `--token`, `--force`, `--json`
-   دورة الحياة (`uninstall|start|stop|restart`): `--json`

ملاحظات:

-   `status` يحل مراجع الأسرار (SecretRefs) المُهيأة للمصادقة عند الفحص عندما يكون ذلك ممكنًا.
-   عندما تتطلب المصادقة باستخدام الرمز المميز (token) وجود رمز وكان `gateway.auth.token` مُدارًا عبر SecretRef، فإن `install` يتحقق من إمكانية حل المرجع ولكنه لا يحفظ الرمز المُحل في بيانات وصف بيئة الخدمة.
-   إذا كانت المصادقة باستخدام الرمز المميز تتطلب رمزًا وكان المرجع المُهيأ SecretRef غير محلول، فإن التثبيت يفشل بشكل مغلق.
-   إذا تم تكوين كل من `gateway.auth.token` و `gateway.auth.password` ولم يتم تعيين `gateway.auth.mode`، فإن التثبيت يتم حظره حتى يتم تعيين الوضع بشكل صريح.

## يُفضل استخدام

استخدم [`openclaw gateway`](./gateway.md) للوثائق والأمثلة الحالية.

[cron](./cron.md)[dashboard](./dashboard.md)

---