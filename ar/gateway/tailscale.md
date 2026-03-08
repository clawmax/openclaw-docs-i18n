

  الوصول البعيد

  
# Tailscale

يمكن لـ OpenClaw تكوين **Serve** (tailnet) أو **Funnel** (عام) تلقائيًا في Tailscale للوحة تحكم البوابة ومنفذ WebSocket. هذا يحافظ على ارتباط البوابة بـ loopback بينما يوفر Tailscale HTTPS والتوجيه و (لـ Serve) رؤوس هوية.

## الأوضاع

-   `serve`: Serve داخل الشبكة الذيلية فقط عبر `tailscale serve`. تبقى البوابة على `127.0.0.1`.
-   `funnel`: HTTPS عام عبر `tailscale funnel`. يتطلب OpenClaw كلمة مرور مشتركة.
-   `off`: الوضع الافتراضي (بدون أتمتة Tailscale).

## المصادقة

اضبط `gateway.auth.mode` للتحكم في عملية المصافحة:

-   `token` (الافتراضي عند تعيين `OPENCLAW_GATEWAY_TOKEN`)
-   `password` (سر مشترك عبر `OPENCLAW_GATEway_PASSWORD` أو التكوين)

عندما يكون `tailscale.mode = "serve"` و `gateway.auth.allowTailscale` هو `true`، يمكن لواجهة تحكم المستخدم/WebSocket استخدام رؤوس هوية Tailscale (`tailscale-user-login`) دون تقديم رمز/كلمة مرور. يتحقق OpenClaw من الهوية عن طريق حل عنوان `x-forwarded-for` عبر برنامج Tailscale المحلي (`tailscale whois`) ومطابقته مع الرأس قبل قبوله. يعامل OpenClaw الطلب على أنه من Serve فقط عندما يصل من loopback مع رؤوس Tailscale `x-forwarded-for` و `x-forwarded-proto` و `x-forwarded-host`. لا تزال نقاط نهاية واجهة برمجة التطبيقات HTTP (على سبيل المثال `/v1/*` و `/tools/invoke` و `/api/channels/*`) تتطلب مصادقة بالرمز/كلمة المرور. يفترض هذا التدفق بدون رمز أن مضيف البوابة موثوق به. إذا كان هناك رمز محلي غير موثوق قد يعمل على نفس المضيف، قم بتعطيل `gateway.auth.allowTailscale` واشترط مصادقة بالرمز/كلمة مرور بدلاً من ذلك. لاشتراط بيانات اعتماد صريحة، اضبط `gateway.auth.allowTailscale: false` أو فرض `gateway.auth.mode: "password"`.

## أمثلة التكوين

### داخل الشبكة الذيلية فقط (Serve)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

افتح: `https:///` (أو `gateway.controlUi.basePath` الذي قمت بتكوينه)

### داخل الشبكة الذيلية فقط (ربط بعنوان IP الشبكة الذيلية)

استخدم هذا عندما تريد أن تستمع البوابة مباشرة على عنوان IP الشبكة الذيلية (بدون Serve/Funnel).

```json
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

الاتصال من جهاز آخر في الشبكة الذيلية:

-   واجهة تحكم المستخدم: `http://<tailscale-ip>:18789/`
-   WebSocket: `ws://<tailscale-ip>:18789`

ملاحظة: loopback (`http://127.0.0.1:18789`) **لن** يعمل في هذا الوضع.

### الإنترنت العام (Funnel + كلمة مرور مشتركة)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

يفضل استخدام `OPENCLAW_GATEWAY_PASSWORD` عن وضع كلمة مرور على القرص.

## أمثلة سطر الأوامر

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## ملاحظات

-   يتطلب Tailscale Serve/Funnel تثبيت وتسجيل دخول أداة سطر الأوامر `tailscale`.
-   يرفض `tailscale.mode: "funnel"` البدء ما لم يكن وضع المصادقة هو `password` لتجنب التعرض العام.
-   اضبط `gateway.tailscale.resetOnExit` إذا كنت تريد أن يقوم OpenClaw بإلغاء تكوين `tailscale serve` أو `tailscale funnel` عند الإغلاق.
-   `gateway.bind: "tailnet"` هو ربط مباشر بالشبكة الذيلية (بدون HTTPS، بدون Serve/Funnel).
-   `gateway.bind: "auto"` يفضل loopback؛ استخدم `tailnet` إذا كنت تريد الشبكة الذيلية فقط.
-   يعرض Serve/Funnel فقط **واجهة تحكم البوابة + WS**. تتصل العقد عبر نفس نقطة نهاية WS للبوابة، لذا يمكن أن يعمل Serve للوصول إلى العقد.

## التحكم عبر المتصفح (بوابة بعيدة + متصفح محلي)

إذا قمت بتشغيل البوابة على جهاز واحد ولكنك تريد تشغيل متصفح على جهاز آخر، قم بتشغيل **مضيف عقدة** على جهاز المتصفح واحتفظ بكليهما على نفس الشبكة الذيلية. ستقوم البوابة بتمرير إجراءات المتصفح إلى العقدة؛ لا حاجة لخادم تحكم منفصل أو عنوان URL لـ Serve. تجنب استخدام Funnel للتحكم بالمتصفح؛ عالج إقران العقدة مثل وصول المشغل.

## متطلبات وقيود Tailscale

-   يتطلب Serve تمكين HTTPS لشبكتك الذيلية؛ ستطلب الأداة ذلك إذا كان مفقودًا.
-   يحقن Serve رؤوس هوية Tailscale؛ بينما لا يفعل Funnel ذلك.
-   يتطلب Funnel Tailscale إصدار v1.38.3+، و MagicDNS، و HTTPS مفعل، وسمة عقدة funnel.
-   يدعم Funnel المنافذ `443` و `8443` و `10000` فقط عبر TLS.
-   يتطلب Funnel على macOS متغير تطبيق Tailscale مفتوح المصدر.

## تعلم المزيد

-   نظرة عامة على Tailscale Serve: [https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
-   أمر `tailscale serve`: [https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
-   نظرة عامة على Tailscale Funnel: [https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
-   أمر `tailscale funnel`: [https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)

[إعداد بوابة بعيدة](./remote-gateway-readme.md)[التحقق الرسمي (نماذج الأمان)](../security/formal-verification.md)