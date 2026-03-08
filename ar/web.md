title: "واجهات ويب OpenClaw وواجهة التحكم وWebhooks والأمان"
description: "تعلم كيفية تكوين واجهة ويب بوابة OpenClaw، وواجهة التحكم، وwebhooks، والوصول عبر Tailscale. قم بإعداد الأمان، والمصادقة، وطرق النشر."
keywords: ["واجهة ويب openclaw", "واجهة تحكم البوابة", "تكامل tailscale", "تكوين webhooks", "أمان البوابة", "ربط loopback", "نشر funnel", "رمز المصادقة"]
---

  واجهات الويب

  
# الويب

تخدم البوابة **واجهة تحكم صغيرة للمتصفح** (Vite + Lit) من نفس منفذ WebSocket الخاص بالبوابة:

-   الافتراضي: `http://:18789/`
-   بادئة اختيارية: اضبط `gateway.controlUi.basePath` (مثال: `/openclaw`)

توجد الإمكانيات في [واجهة التحكم](./web/control-ui.md). تركز هذه الصفحة على أوضاع الربط، والأمان، والأسطح المواجهة للويب.

## Webhooks

عندما يكون `hooks.enabled=true`، تعرض البوابة أيضًا نقطة نهاية صغيرة لـ webhook على نفس خادم HTTP. راجع [تكوين البوابة](./gateway/configuration.md) → `hooks` للمصادقة + الحمولات.

## التكوين (مفعل افتراضيًا)

**واجهة التحكم مفعلة افتراضيًا** عندما تكون الأصول موجودة (`dist/control-ui`). يمكنك التحكم بها عبر التكوين:

```json
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath اختياري
  },
}
```

## الوصول عبر Tailscale

### النشر المتكامل (مُوصى به)

احتفظ بالبوابة على loopback ودع Tailscale Serve يعمل كوكيل لها:

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

ثم ابدأ البوابة:

```bash
openclaw gateway
```

افتح:

-   `https:///` (أو `gateway.controlUi.basePath` الذي قمت بتكوينه)

### الربط على Tailnet + رمز

```json
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

ثم ابدأ البوابة (مطلوب رمز للربط على غير loopback):

```bash
openclaw gateway
```

افتح:

-   `http://<tailscale-ip>:18789/` (أو `gateway.controlUi.basePath` الذي قمت بتكوينه)

### الإنترنت العام (Funnel)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // أو OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## ملاحظات أمنية

-   مطلوب مصادقة البوابة افتراضيًا (رمز/كلمة مرور أو رؤوس هوية Tailscale).
-   عمليات الربط على غير loopback **تتطلب** أيضًا رمز/كلمة مرور مشتركة (`gateway.auth` أو متغير بيئة).
-   يولد المعالج رمز بوابة افتراضيًا (حتى على loopback).
-   ترسل واجهة المستخدم `connect.params.auth.token` أو `connect.params.auth.password`.
-   لعمليات نشر واجهة التحكم على غير loopback، عيّن `gateway.controlUi.allowedOrigins` بشكل صريح (أصول كاملة). بدونها، يتم رفض بدء تشغيل البوابة افتراضيًا.
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` يُفعّل وضع السقوط الاحتياطي للأصل بناءً على رأس Host، ولكنه تخفيض خطير للأمان.
-   مع Serve، يمكن لرؤوس هوية Tailscale تلبية مصادقة واجهة التحكم/WebSocket عندما يكون `gateway.auth.allowTailscale` هو `true` (لا حاجة لرمز/كلمة مرور). نقاط نهاية واجهة برمجة التطبيقات HTTP لا تزال تتطلب رمز/كلمة مرور. اضبط `gateway.auth.allowTailscale: false` لتطلب بيانات اعتماد صريحة. راجع [Tailscale](./gateway/tailscale.md) و [الأمان](./gateway/security.md). يفترض هذا التدفق الخالي من الرموز أن مضيف البوابة موثوق به.
-   يتطلب `gateway.tailscale.mode: "funnel"` `gateway.auth.mode: "password"` (كلمة مرور مشتركة).

## بناء واجهة المستخدم

تخدم البوابة ملفات ثابتة من `dist/control-ui`. ابنها باستخدام:

```bash
pnpm ui:build # يقوم بتثبيت تبعيات واجهة المستخدم تلقائيًا في أول تشغيل
```

[نموذج التهديد للمساهمة](./security/CONTRIBUTING-THREAT-MODEL.md)[واجهة التحكم](./web/control-ui.md)