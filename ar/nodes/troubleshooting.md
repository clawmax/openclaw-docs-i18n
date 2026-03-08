

  الوسائط والأجهزة

  
# استكشاف أخطاء العقدة

استخدم هذه الصفحة عندما تكون العقدة ظاهرة في الحالة ولكن أدوات العقدة تفشل.

## سلم الأوامر

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

ثم قم بتشغيل الفحوصات المحددة للعقدة:

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

إشارات الصحة الجيدة:

-   العقدة متصلة ومقترنة لدور `node`.
-   `nodes describe` تتضمن القدرة التي تستدعيها.
-   موافقات التنفيذ تُظهر الوضع/القائمة المسموحة المتوقع.

## متطلبات المقدمة

`canvas.*`، `camera.*`، و `screen.*` تعمل في المقدمة فقط على عقد iOS/Android. فحص سريع وإصلاح:

```bash
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

إذا رأيت `NODE_BACKGROUND_UNAVAILABLE`، أحضر تطبيق العقدة إلى المقدمة وأعد المحاولة.

## مصفوفة الأذونات

| القدرة | iOS | Android | تطبيق عقدة macOS | رمز الفشل النموذجي |
| --- | --- | --- | --- | --- |
| `camera.snap`, `camera.clip` | الكاميرا (+ ميكروفون لصوت المقطع) | الكاميرا (+ ميكروفون لصوت المقطع) | الكاميرا (+ ميكروفون لصوت المقطع) | `*_PERMISSION_REQUIRED` |
| `screen.record` | تسجيل الشاشة (+ ميكروفون اختياري) | مطالبة التقاط الشاشة (+ ميكروفون اختياري) | تسجيل الشاشة | `*_PERMISSION_REQUIRED` |
| `location.get` | أثناء الاستخدام أو دائمًا (يعتمد على الوضع) | الموقع في المقدمة/الخلفية بناءً على الوضع | إذن الموقع | `LOCATION_PERMISSION_REQUIRED` |
| `system.run` | غير متاح (مسار مضيف العقدة) | غير متاح (مسار مضيف العقدة) | مطلوب موافقات تنفيذ | `SYSTEM_RUN_DENIED` |

## الإقران مقابل الموافقات

هذه بوابات مختلفة:

1.  **إقران الجهاز**: هل يمكن لهذه العقدة الاتصال بالبوابة؟
2.  **موافقات التنفيذ**: هل يمكن لهذه العقدة تشغيل أمر محدد في shell؟

فحوصات سريعة:

```bash
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> "/usr/bin/uname"
```

إذا كان الإقران مفقودًا، قم بالموافقة على جهاز العقدة أولاً. إذا كان الإقران جيدًا ولكن `system.run` يفشل، أصلح موافقات/القائمة المسموحة للتنفيذ.

## رموز أخطاء العقدة الشائعة

-   `NODE_BACKGROUND_UNAVAILABLE` → التطبيق في الخلفية؛ أحضره إلى المقدمة.
-   `CAMERA_DISABLED` → تبديل الكاميرا معطل في إعدادات العقدة.
-   `*_PERMISSION_REQUIRED` → إذن نظام التشغيل مفقود/مرفوض.
-   `LOCATION_DISABLED` → وضع الموقع معطل.
-   `LOCATION_PERMISSION_REQUIRED` → وضع الموقع المطلوب غير ممنوح.
-   `LOCATION_BACKGROUND_UNAVAILABLE` → التطبيق في الخلفية ولكن إذن "أثناء الاستخدام" فقط موجود.
-   `SYSTEM_RUN_DENIED: approval required` → طلب التنفيذ يحتاج إلى موافقة صريحة.
-   `SYSTEM_RUN_DENIED: allowlist miss` → الأمر محظور بوضع القائمة المسموحة. على مضيفي عقد Windows، الأشكال المغلفة مثل `cmd.exe /c ...` تُعامل كأخطاء في القائمة المسموحة في وضع القائمة المسموحة ما لم تتم الموافقة عليها عبر سؤال التدفق.

## حلقة الاستعادة السريعة

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

إذا كنت لا تزال عالقًا:

-   أعد الموافقة على إقران الجهاز.
-   أعد فتح تطبيق العقدة (في المقدمة).
-   أعد منح أذونات نظام التشغيل.
-   أعد إنشاء/ضبط سياسة موافقة التنفيذ.

ذات صلة:

-   [/nodes/index](./index.md)
-   [/nodes/camera](./camera.md)
-   [/nodes/location-command](./location-command.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)
-   [/gateway/pairing](../gateway/pairing.md)

[العقد](../nodes.md)[فهم الوسائط](./media-understanding.md)