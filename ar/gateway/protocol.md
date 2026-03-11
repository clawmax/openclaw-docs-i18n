

  البروتوكولات وواجهات برمجة التطبيقات

  
# بروتوكول البوابة

بروتوكول Gateway WS هو **مستوى التحكم الوحيد + نقل العقدة** لـ OpenClaw. جميع العملاء (CLI، واجهة الويب، تطبيق macOS، عقد iOS/Android، العقد بلا واجهة) يتصلون عبر WebSocket ويعلنون عن **دورهم** + **نطاقهم** في وقت المصافحة.

## النقل

-   WebSocket، إطارات نصية مع حمولات JSON.
-   **يجب** أن تكون الإطارة الأولى طلب `connect`.

## المصافحة (connect)

البوابة → العميل (تحدي ما قبل الاتصال):

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

العميل → البوابة:

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

البوابة → العميل:

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

عند إصدار رمز جهاز، يتضمن `hello-ok` أيضًا:

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```

### مثال للعقدة

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

## التأطير

-   **الطلب**: `{type:"req", id, method, params}`
-   **الاستجابة**: `{type:"res", id, ok, payload|error}`
-   **الحدث**: `{type:"event", event, payload, seq?, stateVersion?}`

تتطلب الطرق ذات الأثر الجانبي **مفاتيح عدم التكرار** (انظر المخطط).

## الأدوار + النطاقات

### الأدوار

-   `operator` = عميل مستوى التحكم (CLI/UI/الأتمتة).
-   `node` = مضيف القدرات (الكاميرا/الشاشة/اللوحة/system.run).

### النطاقات (المشغل)

النطاقات الشائعة:

-   `operator.read`
-   `operator.write`
-   `operator.admin`
-   `operator.approvals`
-   `operator.pairing`

نطاق الطريقة هو البوابة الأولى فقط. بعض أوامر الشرطة المائلة التي يتم الوصول إليها عبر `chat.send` تطبق فحوصات أكثر صرامة على مستوى الأمر فوق ذلك. على سبيل المثال، تتطلب عمليات الكتابة المستمرة `/config set` و `/config unset` نطاق `operator.admin`.

### القدرات/الأوامر/الأذونات (العقدة)

تعلن العقد عن مطالبات القدرات في وقت الاتصال:

-   `caps`: فئات القدرات عالية المستوى.
-   `commands`: قائمة السماح بالأوامر للاستدعاء.
-   `permissions`: مفاتيح تفصيلية (مثل `screen.record`، `camera.capture`).

تعامل البوابة هذه على أنها **مطالبات** وتفرض قوائم السماح من جانب الخادم.

## الحضور

-   `system-presence` تُرجع إدخالات مفتاحها هوية الجهاز.
-   تتضمن إدخالات الحضور `deviceId`، و `roles`، و `scopes` حتى تتمكن واجهات المستخدم من عرض صف واحد لكل جهاز حتى عندما يتصل كـ **مشغل** و **عقدة** معًا.

### طرق المساعدة للعقدة

-   قد تستدعي العقد `skills.bins` لجلب القائمة الحالية للملفات القابلة للتنفيذ للمهارات لفحوصات السماح التلقائي.

### طرق المساعدة للمشغل

-   قد يستدعي المشغلون `tools.catalog` (`operator.read`) لجلب كتالوج أدوات وقت التشغيل للوكيل. تتضمن الاستجابة الأدوات المجمعة وبيانات المصدر:
    -   `source`: `core` أو `plugin`
    -   `pluginId`: مالك المكون الإضافي عندما يكون `source="plugin"`
    -   `optional`: ما إذا كانت أداة المكون الإضافي اختيارية

## موافقات التنفيذ

-   عندما يحتاج طلب التنفيذ إلى موافقة، تبث البوابة `exec.approval.requested`.
-   يحلها عملاء المشغل عن طريق استدعاء `exec.approval.resolve` (يتطلب نطاق `operator.approvals`).
-   بالنسبة لـ `host=node`، يجب أن يتضمن `exec.approval.request` `systemRunPlan` (`argv`/`cwd`/`rawCommand`/بيانات جلسة العمل الأساسية). يتم رفض الطلبات التي تفتقد `systemRunPlan`.

## التحكم بالإصدارات

-   `PROTOCOL_VERSION` موجود في `src/gateway/protocol/schema.ts`.
-   يرسل العملاء `minProtocol` + `maxProtocol`؛ يرفض الخادم التطابقات غير المتطابقة.
-   يتم إنشاء المخططات + النماذج من تعريفات TypeBox:
    -   `pnpm protocol:gen`
    -   `pnpm protocol:gen:swift`
    -   `pnpm protocol:check`

## المصادقة

-   إذا تم تعيين `OPENCLAW_GATEWAY_TOKEN` (أو `--token`)، **يجب** أن يتطابق `connect.params.auth.token` أو يتم إغلاق المقبس.
-   بعد الاقتران، تصدر البوابة **رمز جهاز** محدد النطاق لدور الاتصال + النطاقات. يتم إرجاعه في `hello-ok.auth.deviceToken` ويجب على العميل الاحتفاظ به لاتصالات مستقبلية.
-   يمكن تدوير/إلغاء رموز الجهاز عبر `device.token.rotate` و `device.token.revoke` (يتطلب نطاق `operator.pairing`).

## هوية الجهاز + الاقتران

-   يجب أن تتضمن العقد هوية جهاز مستقرة (`device.id`) مشتقة من بصمة زوج المفاتيح.
-   تصدر البوابات رموزًا لكل جهاز + دور.
-   مطلوب موافقات الاقتران لهويات الأجهزة الجديدة ما لم يتم تمكين الموافقة التلقائية المحلية.
-   تتضمن الاتصالات **المحلية** loopback وعنوان tailnet الخاص بمضيف البوابة نفسه (حتى يمكن لربطات tailnet على نفس المضيف الموافقة تلقائيًا).
-   يجب على جميع عملاء WS تضمين هوية `device` أثناء `connect` (المشغل + العقدة). يمكن لواجهة التحكم حذفها **فقط** عندما يتم تمكين `gateway.controlUi.dangerouslyDisableDeviceAuth` للاستخدام في حالات الطوارئ.
-   يجب على جميع الاتصالات توقيع nonce `connect.challenge` المقدم من الخادم.

### تشخيصات ترقية مصادقة الجهاز

للعملاء القديمين الذين لا يزالون يستخدمون سلوك التوقيع السابق للتحدي، يُرجع `connect` الآن رموز تفاصيل `DEVICE_AUTH_*` تحت `error.details.code` مع `error.details.reason` ثابت. حالات الفشل الشائعة أثناء الترقية:

| الرسالة | details.code | details.reason | المعنى |
| --- | --- | --- | --- |
| `device nonce required` | `DEVICE_AUTH_NONCE_REQUIRED` | `device-nonce-missing` | العميل حذف `device.nonce` (أو أرسل فارغًا). |
| `device nonce mismatch` | `DEVICE_AUTH_NONCE_MISMATCH` | `device-nonce-mismatch` | وقع العميل باستخدام nonce قديم/خاطئ. |
| `device signature invalid` | `DEVICE_AUTH_SIGNATURE_INVALID` | `device-signature` | الحمولة الموقعة لا تطابق حمولة v2. |
| `device signature expired` | `DEVICE_AUTH_SIGNATURE_EXPIRED` | `device-signature-stale` | الطابع الزمني الموقع خارج النطاق المسموح به. |
| `device identity mismatch` | `DEVICE_AUTH_DEVICE_ID_MISMATCH` | `device-id-mismatch` | `device.id` لا يطابق بصمة المفتاح العام. |
| `device public key invalid` | `DEVICE_AUTH_PUBLIC_KEY_INVALID` | `device-public-key` | فشل تنسيق/توحيد المفتاح العام. |

هدف الترقية:

-   انتظر دائمًا `connect.challenge`.
-   وقع حمولة v2 التي تتضمن nonce الخادم.
-   أرسل نفس nonce في `connect.params.device.nonce`.
-   الحمولة الموقعة المفضلة هي `v3`، والتي تربط `platform` و `deviceFamily` بالإضافة إلى حقول الجهاز/العميل/الدور/النطاقات/الرمز/الـ nonce.
-   لا تزال تواقيع `v2` القديمة مقبولة من أجل التوافق، لكن تثبيت بيانات الجهاز المقترن لا يزال يتحكم في سياسة الأمر عند إعادة الاتصال.

## TLS + التثبيت

-   TLS مدعوم لاتصالات WS.
-   قد يثبت العملاء اختياريًا بصمة شهادة البوابة (انظر إعداد `gateway.tls` بالإضافة إلى `gateway.remote.tlsFingerprint` أو `--tls-fingerprint` في CLI).

## النطاق

يعرض هذا البروتوكول **واجهة برمجة تطبيقات البوابة الكاملة** (الحالة، القنوات، النماذج، الدردشة، الوكيل، الجلسات، العقد، الموافقات، إلخ). يتم تحديد السطح الدقيق بواسطة مخططات TypeBox في `src/gateway/protocol/schema.ts`.

[Sandbox vs Tool Policy vs Elevated](./sandbox-vs-tool-policy-vs-elevated.md)[Bridge Protocol](./bridge-protocol.md)