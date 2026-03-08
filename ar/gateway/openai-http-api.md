title: "واجهة برمجة تطبيقات HTTP المتوافقة مع OpenAI لإكمالات المحادثة في OpenClaw Gateway"
description: "تعلم كيفية تمكين واستخدام نقطة نهاية واجهة برمجة تطبيقات HTTP المتوافقة مع OpenAI لإكمالات المحادثة في OpenClaw Gateway، بما في ذلك المصادقة، الأمان، والأمثلة."
keywords: ["واجهة برمجة تطبيقات openai", "إكمالات المحادثة", "بوابة openclaw", "واجهة برمجة تطبيقات http", "وكيل الذكاء الاصطناعي", "التوافق مع openai", "تكامل واجهة برمجة التطبيقات", "مصادقة حامل الرمز"]
---

  البروتوكولات وواجهات برمجة التطبيقات

  
# إكمالات المحادثة من OpenAI

يمكن لبوابة OpenClaw تقديم نقطة نهاية صغيرة لإكمالات المحادثة متوافقة مع OpenAI. هذه النقطة **معطلة بشكل افتراضي**. قم بتمكينها في الإعدادات أولاً.

-   `POST /v1/chat/completions`
-   نفس منفذ البوابة (WS + HTTP متعدد الإرسال): `http://<gateway-host>:/v1/chat/completions`

تحت الغطاء، يتم تنفيذ الطلبات كتشغيل عادي لوكيل البوابة (نفس مسار الكود مثل `openclaw agent`)، لذا فإن التوجيه/الأذونات/الإعدادات تتطابق مع بوابتك.

## المصادقة

يستخدم إعدادات مصادقة البوابة. أرسل رمز حامل:

-   `Authorization: Bearer `

ملاحظات:

-   عندما يكون `gateway.auth.mode="token"`، استخدم `gateway.auth.token` (أو `OPENCLAW_GATEWAY_TOKEN`).
-   عندما يكون `gateway.auth.mode="password"`، استخدم `gateway.auth.password` (أو `OPENCLAW_GATEWAY_PASSWORD`).
-   إذا تم تكوين `gateway.auth.rateLimit` وحدثت العديد من حالات فشل المصادقة، فإن نقطة النهاية ترجع `429` مع `Retry-After`.

## حدود الأمان (مهم)

عامل نقطة النهاية هذه على أنها سطح وصول **كامل للمشغل** لمثيل البوابة.

-   مصادقة حامل الرمز HTTP هنا ليست نموذج نطاق ضيق لكل مستخدم.
-   يجب التعامل مع رمز/كلمة مرور البوابة الصالحة لهذه النقطة النهائية كبيانات اعتماد مالك/مشغل.
-   تمر الطلبات عبر نفس مسار وكيل مستوى التحكم مثل إجراءات المشغل الموثوق به.
-   لا يوجد حد أداة منفصل لغير المالك/لكل مستخدم على نقطة النهاية هذه؛ بمجرد أن يجتاز المتصل مصادقة البوابة هنا، تعامل OpenClaw مع هذا المتصل كمشغل موثوق لهذه البوابة.
-   إذا سمحت سياسة الوكيل الهدف بأدوات حساسة، فيمكن لهذه النقطة النهائية استخدامها.
-   احتفظ بهذه النقطة النهائية على loopback/tailnet/مدخل خاص فقط؛ لا تعرضها مباشرة للإنترنت العام.

راجع [الأمان](./security.md) و[الوصول عن بعد](./remote.md).

## اختيار وكيل

لا توجد رؤوس مخصصة مطلوبة: قم بتشفير معرف الوكيل في حقل OpenAI `model`:

-   `model: "openclaw:"` (مثال: `"openclaw:main"`, `"openclaw:beta"`)
-   `model: "agent:"` (اسم مستعار)

أو استهدف وكيل OpenClaw محددًا عبر الرأس:

-   `x-openclaw-agent-id: ` (الافتراضي: `main`)

متقدم:

-   `x-openclaw-session-key: ` للتحكم الكامل في توجيه الجلسة.

## تمكين نقطة النهاية

اضبط `gateway.http.endpoints.chatCompletions.enabled` على `true`:

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## تعطيل نقطة النهاية

اضبط `gateway.http.endpoints.chatCompletions.enabled` على `false`:

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## سلوك الجلسة

بشكل افتراضي، تكون نقطة النهاية **بدون حالة لكل طلب** (يتم إنشاء مفتاح جلسة جديد في كل استدعاء). إذا كان الطلب يتضمن سلسلة `user` من OpenAI، فإن البوابة تستنتج مفتاح جلسة ثابتًا منها، بحيث يمكن للمكالمات المتكررة مشاركة جلسة وكيل.

## البث (SSE)

اضبط `stream: true` لتلقي أحداث مرسلة من الخادم (SSE):

-   `Content-Type: text/event-stream`
-   كل سطر حدث هو `data: `
-   ينتهي البث بـ `data: [DONE]`

## أمثلة

بدون بث:

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

مع بث:

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```

[بروتوكول الجسر](./bridge-protocol.md)[واجهة برمجة تطبيقات OpenResponses](./openresponses-http-api.md)