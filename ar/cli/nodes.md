

  أوامر CLI

  
# nodes

إدارة العقد (الأجهزة) المقترنة واستدعاء قدرات العقد. ذات صلة:

-   نظرة عامة على العقد: [العقد](../nodes.md)
-   الكاميرا: [عقد الكاميرا](../nodes/camera.md)
-   الصور: [عقد الصور](../nodes/images.md)

الخيارات الشائعة:

-   `--url`, `--token`, `--timeout`, `--json`

## الأوامر الشائعة

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` تطبع جداول العقد المعلقة/المقترنة. تتضمن صفوف العقد المقترنة عمر آخر اتصال (Last Connect). استخدم `--connected` لعرض العقد المتصلة حالياً فقط. استخدم `--last-connected ` لتصفية العقد التي اتصلت خلال مدة زمنية (مثال: `24h`, `7d`).

## استدعاء / تشغيل

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

أعلام الاستدعاء:

-   `--params `: سلسلة كائن JSON (الافتراضي `{}`).
-   `--invoke-timeout `: مهلة استدعاء العقدة (الافتراضي `15000`).
-   `--idempotency-key `: مفتاح عدم التكرار اختياري.

### الإعدادات الافتراضية على نمط التنفيذ

`nodes run` تعكس سلوك exec للنموذج (الإعدادات الافتراضية + الموافقات):

-   تقرأ `tools.exec.*` (بالإضافة إلى تجاوزات `agents.list[].tools.exec.*`).
-   تستخدم موافقات exec (`exec.approval.request`) قبل استدعاء `system.run`.
-   يمكن حذف `--node` عندما يكون `tools.exec.node` مضبوطاً.
-   تتطلب عقدة تعلن عن `system.run` (تطبيق Companion لنظام macOS أو مضيف عقدة headless).

الأعلام:

-   `--cwd `: دليل العمل.
-   `--env <key=val>`: تجاوز متغير البيئة (قابل للتكرار). ملاحظة: مضيفو العقد يتجاهلون تجاوزات `PATH` (ولا يتم تطبيق `tools.exec.pathPrepend` على مضيفي العقد).
-   `--command-timeout `: مهلة الأمر.
-   `--invoke-timeout `: مهلة استدعاء العقدة (الافتراضي `30000`).
-   `--needs-screen-recording`: تتطلب إذن تسجيل الشاشة.
-   `--raw `: تشغيل سلسلة shell (`/bin/sh -lc` أو `cmd.exe /c`). في وضع القائمة المسموحة على مضيفي عقد Windows، يتطلب تشغيل غلاف shell `cmd.exe /c` موافقة (الإدخال في القائمة المسموحة وحده لا يسمح تلقائياً بصيغة الغلاف).
-   `--agent `: موافقات/قوائم مسموحة محددة بالوكيل (الافتراضي هو الوكيل المضبوط).
-   `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: تجاوزات.

[node](./node.md)[onboard](./onboard.md)

---