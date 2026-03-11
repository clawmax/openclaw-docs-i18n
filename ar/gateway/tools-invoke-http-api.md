

  البروتوكولات وواجهات برمجة التطبيقات

  
# واجهة برمجة تطبيقات استدعاء الأدوات

تعرض بوابة OpenClaw نقطة نهاية HTTP بسيطة لاستدعاء أداة واحدة مباشرة. تكون دائمًا مفعلة، ولكن محمية بمصادقة البوابة وسياسة الأداة.

-   `POST /tools/invoke`
-   نفس منفذ البوابة (WS + HTTP مضغوط): `http://<gateway-host>:/tools/invoke`

الحجم الأقصى الافتراضي للحِمل هو 2 ميجابايت.

## المصادقة

يستخدم إعداد مصادقة البوابة. أرسل رمز Bearer:

-   `Authorization: Bearer `

ملاحظات:

-   عندما يكون `gateway.auth.mode="token"`، استخدم `gateway.auth.token` (أو `OPENCLAW_GATEWAY_TOKEN`).
-   عندما يكون `gateway.auth.mode="password"`، استخدم `gateway.auth.password` (أو `OPENCLAW_GATEWAY_PASSWORD`).
-   إذا تم تكوين `gateway.auth.rateLimit` وحدثت فشل مصادقة كثيرة، ترجع نقطة النهاية `429` مع `Retry-After`.

## جسم الطلب

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

الحقول:

-   `tool` (سلسلة نصية، مطلوبة): اسم الأداة المراد استدعاؤها.
-   `action` (سلسلة نصية، اختيارية): يتم تعيينها في args إذا كان مخطط الأداة يدعم `action` وتم حذف حمل args لها.
-   `args` (كائن، اختيارية): وسائط خاصة بالأداة.
-   `sessionKey` (سلسلة نصية، اختيارية): مفتاح الجلسة المستهدف. إذا حُذف أو كان `"main"`، تستخدم البوابة مفتاح الجلسة الرئيسي المُكون (تحترم `session.mainKey` والوكيل الافتراضي، أو `global` في النطاق العام).
-   `dryRun` (منطقية، اختيارية): محجوز للاستخدام المستقبلي؛ يتم تجاهله حاليًا.

## سلوك السياسة + التوجيه

يتم تصفية توفر الأداة من خلال نفس سلسلة السياسة المستخدمة بواسطة وكلاء البوابة:

-   `tools.profile` / `tools.byProvider.profile`
-   `tools.allow` / `tools.byProvider.allow`
-   `agents..tools.allow` / `agents..tools.byProvider.allow`
-   سياسات المجموعة (إذا كان مفتاح الجلسة يُرسم إلى مجموعة أو قناة)
-   سياسة الوكيل الفرعي (عند الاستدعاء بمفتاح جلسة وكيل فرعي)

إذا لم تكن الأداة مسموحًا بها حسب السياسة، ترجع نقطة النهاية **404**. تطبق بوابة HTTP أيضًا قائمة حظر صارمة افتراضيًا (حتى لو سمحت سياسة الجلسة بالأداة):

-   `sessions_spawn`
-   `sessions_send`
-   `gateway`
-   `whatsapp_login`

يمكنك تخصيص قائمة الحظر هذه عبر `gateway.tools`:

```json
{
  gateway: {
    tools: {
      // أدوات إضافية لحظرها عبر HTTP /tools/invoke
      deny: ["browser"],
      // إزالة أدوات من قائمة الحظر الافتراضية
      allow: ["gateway"],
    },
  },
}
```

لمساعدة سياسات المجموعة في حل السياق، يمكنك تعيين ما يلي اختياريًا:

-   `x-openclaw-message-channel: ` (مثال: `slack`, `telegram`)
-   `x-openclaw-account-id: ` (عند وجود حسابات متعددة)

## الردود

-   `200` → `{ ok: true, result }`
-   `400` → `{ ok: false, error: { type, message } }` (طلب غير صالح أو خطأ في إدخال الأداة)
-   `401` → غير مصرح
-   `429` → تجاوز حد معدل المصادقة (تم تعيين `Retry-After`)
-   `404` → الأداة غير متاحة (غير موجودة أو غير مدرجة في قائمة السماح)
-   `405` → الطريقة غير مسموح بها
-   `500` → `{ ok: false, error: { type, message } }` (خطأ غير متوقع في تنفيذ الأداة؛ رسالة مُنقحة)

## مثال

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```

[واجهة برمجة تطبيقات OpenResponses](./openresponses-http-api.md)[واجهات CLI الخلفية](./cli-backends.md)