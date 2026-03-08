title: "دليل الإعداد والميزات والبدء السريع لتطبيق OpenClaw لنظام iOS"
description: "تعلم كيفية إعداد تطبيق OpenClaw لنظام iOS، والاتصال بالبوابة، واستخدام ميزات مثل Canvas و Voice Wake وأوامر العقد. يتضمن المتطلبات واستكشاف الأخطاء وإصلاحها."
keywords: ["openclaw ios", "إعداد تطبيق ios", "اتصال البوابة", "canvas a2ui", "استدعاء العقدة", "إيقاظ صوتي", "اكتشاف bonjour", "إعداد tailnet"]
---

  نظرة عامة على المنصات

  
# تطبيق iOS

التوفر: معاينة داخلية. تطبيق iOS غير متاح للجمهور بعد.

## ما يفعله

-   يتصل ببوابة عبر WebSocket (شبكة محلية LAN أو شبكة ذيل tailnet).
-   يعرض إمكانيات العقدة: Canvas، ولقطة الشاشة، والتقاط الكاميرا، والموقع، ووضع التحدث، والإيقاظ الصوتي.
-   يتلقى أوامر `node.invoke` ويبلغ عن أحداث حالة العقدة.

## المتطلبات

-   بوابة تعمل على جهاز آخر (macOS، أو Linux، أو Windows عبر WSL2).
-   مسار الشبكة:
    -   نفس الشبكة المحلية عبر Bonjour، **أو**
    -   شبكة ذيل عبر DNS-SD أحادي البث (مثال النطاق: `openclaw.internal.`)، **أو**
    -   المضيف/المنفذ يدوياً (خيار احتياطي).

## البدء السريع (إقران + اتصال)

1.  ابدأ تشغيل البوابة:

```bash
openclaw gateway --port 18789
```

2.  في تطبيق iOS، افتح الإعدادات واختر بوابة مكتشفة (أو فعّل "المضيف اليدوي" وأدخل المضيف/المنفذ).
3.  وافق على طلب الإقران على مضيف البوابة:

```bash
openclaw devices list
openclaw devices approve <requestId>
```

4.  تحقق من الاتصال:

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

## مسارات الاكتشاف

### Bonjour (شبكة محلية LAN)

تعلن البوابة عن `_openclaw-gw._tcp` على `local.`. يعرض تطبيق iOS هذه تلقائياً.

### شبكة الذيل Tailnet (عبر الشبكات)

إذا كان mDNS محظوراً، استخدم منطقة DNS-SD أحادية البث (اختر نطاقاً؛ مثال: `openclaw.internal.`) و DNS المقسم لـ Tailscale. راجع [Bonjour](../gateway/bonjour.md) للحصول على مثال CoreDNS.

### المضيف/المنفذ يدوياً

في الإعدادات، فعّل **المضيف اليدوي** وأدخل مضيف البوابة + المنفذ (الافتراضي `18789`).

## Canvas + A2UI

تعرض عقدة iOS لوحة رسم WKWebView. استخدم `node.invoke` لقيادتها:

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18789/__openclaw__/canvas/"}'
```

ملاحظات:

-   يخدم مضيف لوحة رسم البوابة `/__openclaw__/canvas/` و `/__openclaw__/a2ui/`.
-   يتم تقديمه من خادم HTTP الخاص بالبوابة (نفس منفذ `gateway.port`، الافتراضي `18789`).
-   تنتقل عقدة iOS تلقائياً إلى A2UI عند الاتصال عند الإعلان عن عنوان URL لمضيف لوحة الرسم.
-   للعودة إلى السقالة المدمجة، استخدم `canvas.navigate` مع `{"url":""}`.

### تقييم Canvas / لقطة

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

## الإيقاظ الصوتي + وضع التحدث

-   الإيقاظ الصوتي ووضع التحدث متاحان في الإعدادات.
-   قد يعلق iOS الصوت في الخلفية؛ تعامل مع ميزات الصوت على أنها "أفضل جهد" عندما لا يكون التطبيق نشطاً.

## أخطاء شائعة

-   `NODE_BACKGROUND_UNAVAILABLE`: اجلب تطبيق iOS إلى الواجهة الأمامية (أوامر canvas/الكاميرا/الشاشة تتطلب ذلك).
-   `A2UI_HOST_NOT_CONFIGURED`: لم تعلن البوابة عن عنوان URL لمضيف لوحة الرسم؛ تحقق من `canvasHost` في [تكوين البوابة](../gateway/configuration.md).
-   مطالبة الإقران لا تظهر أبداً: شغّل `openclaw devices list` ووافق يدوياً.
-   فشل إعادة الاتصال بعد إعادة التثبيت: تم مسح رمز الإقران من Keychain؛ أعد إقران العقدة.

## وثائق ذات صلة

-   [الإقران](../channels/pairing.md)
-   [الاكتشاف](../gateway/discovery.md)
-   [Bonjour](../gateway/bonjour.md)

[تطبيق Android](./android.md)[DigitalOcean](./digitalocean.md)

---