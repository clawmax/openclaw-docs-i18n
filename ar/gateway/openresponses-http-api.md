title: "واجهة برمجة تطبيقات HTTP لـ OpenResponses لتكوين واستخدام بوابة OpenClaw"
description: "تعلم كيفية تمكين واستخدام نقطة النهاية المتوافقة مع OpenResponses POST /v1/responses في بوابة OpenClaw. قم بتكوين المصادقة، الأمان، الوكلاء، الأدوات، ومعالجة الملفات."
keywords: ["واجهة برمجة تطبيقات openresponses", "بوابة openclaw", "واجهة برمجة تطبيقات http", "تكامل الوكيل", "مصادقة api", "أدوات العميل", "رفع الملفات", "بث sse"]
---

  البروتوكولات وواجهات برمجة التطبيقات

  
# واجهة برمجة تطبيقات OpenResponses

يمكن لبوابة OpenClaw تقديم نقطة نهاية متوافقة مع OpenResponses `POST /v1/responses`. هذه النقطة **معطلة افتراضيًا**. قم بتمكينها في التكوين أولاً.

-   `POST /v1/responses`
-   نفس منفذ البوابة (WS + HTTP متعدد الإرسال): `http://<gateway-host>:/v1/responses`

تحت الغطاء، يتم تنفيذ الطلبات كتشغيل عادي لوكيل البوابة (نفس مسار الكود مثل `openclaw agent`)، لذا فإن التوجيه/الأذونات/التكوين يتطابق مع بوابتك.

## المصادقة

يستخدم تكوين مصادقة البوابة. أرسل رمز Bearer:

-   `Authorization: Bearer `

ملاحظات:

-   عندما يكون `gateway.auth.mode="token"`، استخدم `gateway.auth.token` (أو `OPENCLAW_GATEWAY_TOKEN`).
-   عندما يكون `gateway.auth.mode="password"`، استخدم `gateway.auth.password` (أو `OPENCLAW_GATEWAY_PASSWORD`).
-   إذا تم تكوين `gateway.auth.rateLimit` وحدثت العديد من حالات فشل المصادقة، ترجع نقطة النهاية `429` مع `Retry-After`.

## حدود الأمان (مهم)

عامل نقطة النهاية هذه على أنها سطح وصول **كامل المشغل** لنموذج البوابة.

-   مصادقة HTTP bearer هنا ليست نموذج نطاق ضيق لكل مستخدم.
-   يجب التعامل مع رمز/كلمة مرور بوابة صالحة لهذه النقطة النهاية مثل بيانات اعتماد المالك/المشغل.
-   تمر الطلبات عبر نفس مسار وكيل مستوى التحكم مثل إجراءات المشغل الموثوق به.
-   لا يوجد حد أداة منفصل لغير المالك/لكل مستخدم على هذه النقطة النهاية؛ بمجرد أن يجتاز المتصل مصادقة البوابة هنا، تعامل OpenClaw مع هذا المتصل كمشغل موثوق به لهذه البوابة.
-   إذا سمحت سياسة الوكيل المستهدف بأدوات حساسة، يمكن لهذه النقطة النهاية استخدامها.
-   احتفظ بهذه النقطة النهاية على loopback/tailnet/مدخل خاص فقط؛ لا تعرضها مباشرة للإنترنت العام.

راجع [الأمان](./security.md) و[الوصول عن بعد](./remote.md).

## اختيار وكيل

لا توجد رؤوس مخصصة مطلوبة: قم بتشفير معرف الوكيل في حقل OpenResponses `model`:

-   `model: "openclaw:"` (مثال: `"openclaw:main"`, `"openclaw:beta"`)
-   `model: "agent:"` (اسم مستعار)

أو استهدف وكيل OpenClaw محددًا عن طريق الرأس:

-   `x-openclaw-agent-id: ` (الافتراضي: `main`)

متقدم:

-   `x-openclaw-session-key: ` للتحكم الكامل في توجيه الجلسة.

## تمكين نقطة النهاية

اضبط `gateway.http.endpoints.responses.enabled` على `true`:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true },
      },
    },
  },
}
```

## تعطيل نقطة النهاية

اضبط `gateway.http.endpoints.responses.enabled` على `false`:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false },
      },
    },
  },
}
```

## سلوك الجلسة

افتراضيًا، تكون نقطة النهاية **بدون حالة لكل طلب** (يتم إنشاء مفتاح جلسة جديد في كل استدعاء). إذا كان الطلب يتضمن سلسلة OpenResponses `user`، فإن البوابة تستمد مفتاح جلسة ثابتًا منه، بحيث يمكن للمكالمات المتكررة مشاركة جلسة وكيل.

## شكل الطلب (المدعوم)

يتبع الطلب واجهة برمجة تطبيقات OpenResponses مع الإدخال القائم على العناصر. الدعم الحالي:

-   `input`: سلسلة أو مصفوفة من كائنات العناصر.
-   `instructions`: يتم دمجها في موجه النظام.
-   `tools`: تعريفات أدوات العميل (أدوات الدالة).
-   `tool_choice`: تصفية أو طلب أدوات العميل.
-   `stream`: يمكّن بث SSE.
-   `max_output_tokens`: حد إخراج بأفضل جهد (يعتمد على المزود).
-   `user`: توجيه جلسة ثابت.

مقبولة ولكن **يتم تجاهلها حاليًا**:

-   `max_tool_calls`
-   `reasoning`
-   `metadata`
-   `store`
-   `previous_response_id`
-   `truncation`

## العناصر (الإدخال)

### message

الأدوار: `system`, `developer`, `user`, `assistant`.

-   يتم إلحاق `system` و `developer` بموجه النظام.
-   يصبح أحدث عنصر `user` أو `function_call_output` هو "الرسالة الحالية".
-   يتم تضمين رسائل user/assistant السابقة كتاريخ للسياق.

### function\_call\_output (أدوات قائمة على الأدوار)

أرسل نتائج الأداة إلى النموذج:

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```

### reasoning و item\_reference

مقبولة لتوافق المخطط ولكن يتم تجاهلها عند بناء الموجه.

## الأدوات (أدوات الدالة من جانب العميل)

قدم أدوات باستخدام `tools: [{ type: "function", function: { name, description?, parameters? } }]`. إذا قرر الوكيل استدعاء أداة، فإن الاستجابة ترجع عنصر إخراج `function_call`. ثم تقوم بإرسال طلب متابعة مع `function_call_output` لمواصلة الدور.

## الصور (input\_image)

يدعم مصادر base64 أو URL:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

أنواع MIME المسموح بها (حاليًا): `image/jpeg`, `image/png`, `image/gif`, `image/webp`, `image/heic`, `image/heif`. الحد الأقصى للحجم (حاليًا): 10 ميجابايت.

## الملفات (input\_file)

يدعم مصادر base64 أو URL:

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

أنواع MIME المسموح بها (حاليًا): `text/plain`, `text/markdown`, `text/html`, `text/csv`, `application/json`, `application/pdf`. الحد الأقصى للحجم (حاليًا): 5 ميجابايت. السلوك الحالي:

-   يتم فك تشفير محتوى الملف وإضافته إلى **موجه النظام**، وليس رسالة المستخدم، لذلك يبقى عابرًا (غير مخزن في تاريخ الجلسة).
-   يتم تحليل ملفات PDF للحصول على النص. إذا تم العثور على نص قليل، يتم تحويل الصفحات الأولى إلى صور وتمريرها إلى النموذج.

يستخدم تحليل PDF بناء `pdfjs-dist` القديم المتوافق مع Node (بدون worker). يتوقع بناء PDF.js الحديث عمال المتصفح/متغيرات DOM العامة، لذلك لا يتم استخدامه في البوابة. إعدادات جلب URL الافتراضية:

-   `files.allowUrl`: `true`
-   `images.allowUrl`: `true`
-   `maxUrlParts`: `8` (إجمالي أجزاء `input_file` + `input_image` القائمة على URL لكل طلب)
-   الطلبات محمية (حل DNS، منع IP الخاص، حدود إعادة التوجيه، مهلات).
-   قوائم السماح الاختيارية لأسماء المضيف مدعومة حسب نوع الإدخال (`files.urlAllowlist`, `images.urlAllowlist`).
    -   المضيف الدقيق: `"cdn.example.com"`
    -   النطاقات الفرعية ذات الأحرف البدل: `"*.assets.example.com"` (لا تطابق القمة)

## حدود الملفات + الصور (التكوين)

يمكن ضبط الإعدادات الافتراضية تحت `gateway.http.endpoints.responses`:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          maxUrlParts: 8,
          files: {
            allowUrl: true,
            urlAllowlist: ["cdn.example.com", "*.assets.example.com"],
            allowedMimes: [
              "text/plain",
              "text/markdown",
              "text/html",
              "text/csv",
              "application/json",
              "application/pdf",
            ],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200,
            },
          },
          images: {
            allowUrl: true,
            urlAllowlist: ["images.example.com"],
            allowedMimes: [
              "image/jpeg",
              "image/png",
              "image/gif",
              "image/webp",
              "image/heic",
              "image/heif",
            ],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000,
          },
        },
      },
    },
  },
}
```

الإعدادات الافتراضية عند الحذف:

-   `maxBodyBytes`: 20 ميجابايت
-   `maxUrlParts`: 8
-   `files.maxBytes`: 5 ميجابايت
-   `files.maxChars`: 200 ألف
-   `files.maxRedirects`: 3
-   `files.timeoutMs`: 10 ثوانٍ
-   `files.pdf.maxPages`: 4
-   `files.pdf.maxPixels`: 4,000,000
-   `files.pdf.minTextChars`: 200
-   `images.maxBytes`: 10 ميجابايت
-   `images.maxRedirects`: 3
-   `images.timeoutMs`: 10 ثوانٍ
-   مصادر `input_image` من نوع HEIC/HEIF مقبولة ويتم تحويلها إلى JPEG قبل التسليم للمزود.

ملاحظة أمنية:

-   يتم فرض قوائم السماح الخاصة بـ URL قبل الجلب وعند نقاط إعادة التوجيه.
-   إدراج اسم مضيف في قائمة السماح لا يتجاوز منع IP الخاص/الداخلي.
-   للبوابات المعرضة للإنترنت، قم بتطبيق ضوابط خروج الشبكة بالإضافة إلى حراسات مستوى التطبيق. راجع [الأمان](./security.md).

## البث (SSE)

اضبط `stream: true` لتلقي أحداث مرسلة من الخادم (SSE):

-   `Content-Type: text/event-stream`
-   كل سطر حدث هو `event: ` و `data: `
-   ينتهي البث بـ `data: [DONE]`

أنواع الأحداث المنبعثة حاليًا:

-   `response.created`
-   `response.in_progress`
-   `response.output_item.added`
-   `response.content_part.added`
-   `response.output_text.delta`
-   `response.output_text.done`
-   `response.content_part.done`
-   `response.output_item.done`
-   `response.completed`
-   `response.failed` (عند الخطأ)

## الاستخدام

يتم ملء `usage` عندما يبلغ المزود الأساسي عن أعداد الرموز.

## الأخطاء

تستخدم الأخطاء كائن JSON مثل:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

الحالات الشائعة:

-   `401` مصادقة مفقودة/غير صالحة
-   `400` جسم طلب غير صالح
-   `405` طريقة خاطئة

## أمثلة

بدون بث:

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

مع بث:

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```

[OpenAI Chat Completions](./openai-http-api.md)[Tools Invoke API](./tools-invoke-http-api.md)