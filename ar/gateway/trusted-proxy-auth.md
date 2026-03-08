title: "دليل تكوين مصادقة الوكيل الموثوق في OpenClaw Gateway"
description: "تعلّم كيفية تكوين OpenClaw Gateway بشكل آمن مع مصادقة الوكيل الموثوق للوكلاء العكسية الواعية بالهوية مثل Pomerium وCaddy وnginx."
keywords: ["مصادقة الوكيل الموثوق", "openclaw gateway", "مصادقة الوكيل العكسي", "pomerium openclaw", "caddy oauth", "nginx oauth2-proxy", "وكيل واعٍ بالهوية", "مصادقة websocket"]
---

  التكوين والعمليات

  
# مصادقة الوكيل الموثوق

> ⚠️ **ميزة حساسة أمنياً.** يفوض هذا الوضع المصادقة بالكامل إلى الوكيل العكسي الخاص بك. يمكن أن يؤدي التكوين الخاطئ إلى تعرض Gateway الخاص بك للوصول غير المصرح به. اقرأ هذه الصفحة بعناية قبل التمكين.

## متى تستخدم

استخدم وضع المصادقة `trusted-proxy` عندما:

-   تقوم بتشغيل OpenClaw خلف **وكيل عكسي واعٍ بالهوية** (Pomerium، Caddy + OAuth، nginx + oauth2-proxy، Traefik + forward auth)
-   يتعامل الوكيل الخاص بك مع جميع المصادقة ويمرر هوية المستخدم عبر الرؤوس (headers)
-   أنت في بيئة Kubernetes أو حاوية حيث يكون الوكيل هو المسار الوحيد إلى Gateway
-   تواجه أخطاء WebSocket `1008 unauthorized` لأن المتصفحات لا تستطيع تمرير الرموز (tokens) في حمولة WS

## متى لا تستخدم

-   إذا كان الوكيل العكسي الخاص بك لا يقوم بمصادقة المستخدمين (مجرد مُنهي TLS أو موزع حمل)
-   إذا كان هناك أي مسار إلى Gateway يتجاوز الوكيل (ثغرات في الجدار الناري، وصول الشبكة الداخلية)
-   إذا لم تكن متأكداً مما إذا كان الوكيل الخاص بك يقوم بإزالة/استبدال الرؤوس المُمررة بشكل صحيح
-   إذا كنت تحتاج فقط إلى وصول شخصي لمستخدم واحد (فكر في استخدام Tailscale Serve + loopback لإعداد أبسط)

## آلية العمل

1.  يقوم الوكيل العكسي الخاص بك بمصادقة المستخدمين (OAuth، OIDC، SAML، إلخ.)
2.  يضيف الوكيل رأساً يحتوي على هوية المستخدم المصادق عليه (مثال: `x-forwarded-user: nick@example.com`)
3.  يتحقق OpenClaw من أن الطلب جاء من **عنوان IP للوكيل الموثوق** (تم تكوينه في `gateway.trustedProxies`)
4.  يستخرج OpenClaw هوية المستخدم من الرأس المُكون
5.  إذا تم التحقق من كل شيء، يتم تفويض الطلب

## سلوك إقران واجهة التحكم (Control UI)

عندما يكون `gateway.auth.mode = "trusted-proxy"` نشطاً ويمر الطلب فحوصات الوكيل الموثوق، يمكن لجلسات WebSocket لواجهة التحكم الاتصال دون هوية إقران الجهاز. الآثار المترتبة:

-   لم يعد الإقران هو البوابة الأساسية للوصول إلى واجهة التحكم في هذا الوضع.
-   تصبح سياسة المصادقة للوكيل العكسي الخاص بك و `allowUsers` هي عنصر التحكم الفعلي في الوصول.
-   حافظ على تقييد دخول Gateway لعناوين IP الوكيل الموثوق فقط (`gateway.trustedProxies` + الجدار الناري).

## التكوين

```json
{
  gateway: {
    // استخدم loopback لإعدادات الوكيل على نفس المضيف؛ استخدم lan/custom لمضيفات الوكيل البعيدة
    bind: "loopback",

    // حاسم: أضف فقط عنوان (عناوين) IP للوكيل الخاص بك هنا
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // الرأس الذي يحتوي على هوية المستخدم المصادق عليه (مطلوب)
        userHeader: "x-forwarded-user",

        // اختياري: الرؤوس التي يجب أن تكون موجودة (للتحقق من الوكيل)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // اختياري: تقييد لمستخدمين محددين (فارغ = السماح للجميع)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

إذا كان `gateway.bind` هو `loopback`، قم بتضمين عنوان وكيل loopback في `gateway.trustedProxies` (`127.0.0.1`، `::1`، أو نطاق CIDR مكافئ لـ loopback).

### مرجع التكوين

| الحقل | مطلوب | الوصف |
| --- | --- | --- |
| `gateway.trustedProxies` | نعم | مصفوفة عناوين IP الوكيل الموثوقة. يتم رفض الطلبات من عناوين IP أخرى. |
| `gateway.auth.mode` | نعم | يجب أن يكون `"trusted-proxy"` |
| `gateway.auth.trustedProxy.userHeader` | نعم | اسم الرأس الذي يحتوي على هوية المستخدم المصادق عليه |
| `gateway.auth.trustedProxy.requiredHeaders` | لا | رؤوس إضافية يجب أن تكون موجودة ليتم الوثوق بالطلب |
| `gateway.auth.trustedProxy.allowUsers` | لا | قائمة السماح بهويات المستخدمين. الفارغ يعني السماح لجميع المستخدمين المصادق عليهم. |

## إنهاء TLS و HSTS

استخدم نقطة إنهاء TLS واحدة وطبق HSTS هناك.

### النمط الموصى به: إنهاء TLS في الوكيل

عندما يتعامل الوكيل العكسي الخاص بك مع HTTPS لـ `https://control.example.com`، قم بتعيين `Strict-Transport-Security` عند الوكيل لهذا النطاق.

-   مناسب جيداً للنشرات المواجهة للإنترنت.
-   يحتفظ بالشهادة وسياسة تعزيز HTTP في مكان واحد.
-   يمكن أن يبقى OpenClaw على HTTP loopback خلف الوكيل.

مثال لقيمة الرأس:

```yaml
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### إنهاء TLS في Gateway

إذا كان OpenClaw نفسه يخدم HTTPS مباشرة (بدون وكيل مُنهي لـ TLS)، قم بتعيين:

```json
{
  gateway: {
    tls: { enabled: true },
    http: {
      securityHeaders: {
        strictTransportSecurity: "max-age=31536000; includeSubDomains",
      },
    },
  },
}
```

يقبل `strictTransportSecurity` قيمة رأس نصية، أو `false` لتعطيله صراحةً.

### إرشادات النشر

-   ابدأ بعمر أقصى قصير أولاً (على سبيل المثال `max-age=300`) أثناء التحقق من حركة المرور.
-   زد إلى قيم طويلة الأمد (على سبيل المثال `max-age=31536000`) فقط بعد أن تكون الثقة عالية.
-   أضف `includeSubDomains` فقط إذا كان كل نطاق فرعي جاهزاً لـ HTTPS.
-   استخدم preload فقط إذا كنت تستوفي متطلبات preload عمداً لمجموعة النطاق الكاملة الخاصة بك.
-   لا يستفيد التطوير المحلي loopback-only من HSTS.

## أمثلة إعداد الوكيل

### Pomerium

يمرر Pomerium الهوية في `x-pomerium-claim-email` (أو رؤوس المطالبات الأخرى) و JWT في `x-pomerium-jwt-assertion`.

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // عنوان IP لـ Pomerium
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

مقتطف تكوين Pomerium:

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy مع OAuth

يمكن لـ Caddy مع إضافة `caddy-security` مصادقة المستخدمين وتمرير رؤوس الهوية.

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // عنوان IP لـ Caddy (إذا كان على نفس المضيف)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

مقتطف Caddyfile:

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

يقوم oauth2-proxy بمصادقة المستخدمين وتمرير الهوية في `x-auth-request-email`.

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // عنوان IP لـ nginx/oauth2-proxy
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

مقتطف تكوين nginx:

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik مع Forward Auth

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // عنوان IP حاوية Traefik
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## قائمة التحقق الأمنية

قبل تمكين مصادقة الوكيل الموثوق، تحقق من:

-   [ ]  **الوكيل هو المسار الوحيد**: منفذ Gateway محمي بجدار ناري من كل شيء باستثناء الوكيل الخاص بك
-   [ ]  **trustedProxies محدود للغاية**: فقط عناوين IP الوكيل الفعلية، وليس نطاقات فرعية كاملة
-   [ ]  **الوكيل يزيل الرؤوس**: يقوم الوكيل الخاص بك باستبدال (وليس إضافة) رؤوس `x-forwarded-*` من العملاء
-   [ ]  **إنهاء TLS**: يتعامل الوكيل الخاص بك مع TLS؛ يتصل المستخدمون عبر HTTPS
-   [ ]  **تم تعيين allowUsers** (موصى به): قصر الوصول على مستخدمين معروفين بدلاً من السماح لأي شخص مصادق عليه

## التدقيق الأمني

سيقوم `openclaw security audit` بوضع علامة على مصادقة الوكيل الموثوق بدرجة خطورة **حرجة**. هذا مقصود — إنه تذكير بأنك تفوض الأمان إلى إعداد الوكيل الخاص بك. يتحقق التدقيق من:

-   تكوين `trustedProxies` مفقود
-   تكوين `userHeader` مفقود
-   `allowUsers` فارغ (يسمح لأي مستخدم مصادق عليه)

## استكشاف الأخطاء وإصلاحها

### ”trusted\_proxy\_untrusted\_source"

لم يأتِ الطلب من عنوان IP موجود في `gateway.trustedProxies`. تحقق من:

-   هل عنوان IP الوكيل صحيح؟ (يمكن أن تتغير عناوين IP حاويات Docker)
-   هل هناك موزع حمل أمام الوكيل الخاص بك؟
-   استخدم `docker inspect` أو `kubectl get pods -o wide` للعثور على عناوين IP الفعلية

### ”trusted\_proxy\_user\_missing"

كان رأس المستخدم فارغاً أو مفقوداً. تحقق من:

-   هل تم تكوين الوكيل الخاص بك لتمرير رؤوس الهوية؟
-   هل اسم الرأس صحيح؟ (غير حساس لحالة الأحرف، لكن التهجئة مهمة)
-   هل تمت مصادقة المستخدم بالفعل عند الوكيل؟

### “trustedproxy\_missing\_header\*"

لم يكن رأس مطلوب موجوداً. تحقق من:

-   تكوين الوكيل الخاص بك لتلك الرؤوس المحددة
-   ما إذا كانت الرؤوس تُزال في مكان ما في السلسلة

### ”trusted\_proxy\_user\_not\_allowed"

تمت مصادقة المستخدم لكنه غير موجود في `allowUsers`. إما أضفه أو أزل قائمة السماح.

### فشل WebSocket مستمر

تأكد من أن الوكيل الخاص بك:

-   يدعم ترقيات WebSocket (`Upgrade: websocket`، `Connection: upgrade`)
-   يمرر رؤوس الهوية على طلبات ترقية WebSocket (وليس HTTP فقط)
-   لا يحتوي على مسار مصادقة منفصل لاتصالات WebSocket

## الهجرة من مصادقة الرمز (Token Auth)

إذا كنت تنتقل من مصادقة الرمز إلى الوكيل الموثوق:

1.  قم بتكوين الوكيل الخاص بك لمصادقة المستخدمين وتمرير الرؤوس
2.  اختبر إعداد الوكيل بشكل مستقل (curl مع الرؤوس)
3.  قم بتحديث تكوين OpenClaw بمصادقة الوكيل الموثوق
4.  أعد تشغيل Gateway
5.  اختبر اتصالات WebSocket من واجهة التحكم
6.  قم بتشغيل `openclaw security audit` ومراجعة النتائج

## ذات صلة

-   [الأمان](./security.md) — دليل الأمان الكامل
-   [التكوين](./configuration.md) — مرجع التكوين
-   [الوصول عن بُعد](./remote.md) — أنماط الوصول عن بُعد الأخرى
-   [Tailscale](./tailscale.md) — بديل أبسط للوصول المقتصر على tailnet فقط

[Secrets Apply Plan Contract](./secrets-plan-contract.md)[Health Checks](./health.md)