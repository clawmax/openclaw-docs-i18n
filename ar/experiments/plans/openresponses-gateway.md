

  تجارب

  
# خطة بوابة OpenResponses

## السياق

تعرض بوابة OpenClaw حاليًا نقطة نهاية دنيا متوافقة مع OpenAI لـ Chat Completions على المسار `/v1/chat/completions` (انظر [OpenAI Chat Completions](../../gateway/openai-http-api.md)). Open Responses هو معيار استدلال مفتوح يعتمد على OpenAI Responses API. وهو مصمم لسير العمل الوكيلية ويستخدم مدخلات قائمة على العناصر بالإضافة إلى أحداث البث الدلالي. يحدد مواصفات OpenResponses المسار `/v1/responses`، وليس `/v1/chat/completions`.

## الأهداف

-   إضافة نقطة نهاية `/v1/responses` تلتزم بدلالات OpenResponses.
-   الحفاظ على Chat Completents كطبقة توافق يسهل تعطيلها وإزالتها في النهاية.
-   توحيد التحقق والتحليل باستخدام مخططات معزولة وقابلة لإعادة الاستخدام.

## غير الأهداف

-   تحقيق تكافؤ كامل لميزات OpenResponses في المرحلة الأولى (الصور، الملفات، الأدوات المستضافة).
-   استبدال منطق تنفيذ الوكيل الداخلي أو تنسيق الأدوات.
-   تغيير سلوك `/v1/chat/completions` الحالي خلال المرحلة الأولى.

## ملخص البحث

المصادر: OpenResponses OpenAPI، موقع مواصفات OpenResponses، ومنشور مدونة Hugging Face. النقاط الرئيسية المستخلصة:

-   `POST /v1/responses` يقبل حقول `CreateResponseBody` مثل `model`، `input` (سلسلة نصية أو `ItemParam[]`)، `instructions`، `tools`، `tool_choice`، `stream`، `max_output_tokens`، و `max_tool_calls`.
-   `ItemParam` هو اتحاد مميز من:
    -   عناصر `message` بأدوار `system`، `developer`، `user`، `assistant`
    -   `function_call` و `function_call_output`
    -   `reasoning`
    -   `item_reference`
-   الردود الناجحة تُرجع `ResponseResource` مع `object: "response"`، `status`، وعناصر `output`.
-   يستخدم البث المتدفق أحداثًا دلالية مثل:
    -   `response.created`، `response.in_progress`، `response.completed`، `response.failed`
    -   `response.output_item.added`، `response.output_item.done`
    -   `response.content_part.added`، `response.content_part.done`
    -   `response.output_text.delta`، `response.output_text.done`
-   تتطلب المواصفات:
    -   `Content-Type: text/event-stream`
    -   يجب أن يتطابق `event:` مع حقل JSON `type`
    -   الحدث النهائي يجب أن يكون حرفيًا `[DONE]`
-   قد تعرض عناصر التفكير `content`، `encrypted_content`، و `summary`.
-   تتضمن أمثلة HF `OpenResponses-Version: latest` في الطلبات (رأس اختياري).

## البنية المقترحة

-   إضافة `src/gateway/open-responses.schema.ts` يحتوي فقط على مخططات Zod (بدون استيرادات من البوابة).
-   إضافة `src/gateway/openresponses-http.ts` (أو `open-responses-http.ts`) للمسار `/v1/responses`.
-   الحفاظ على `src/gateway/openai-http.ts` كما هو كمحول توافق قديم.
-   إضافة إعداد `gateway.http.endpoints.responses.enabled` (القيمة الافتراضية `false`).
-   الحفاظ على `gateway.http.endpoints.chatCompletions.enabled` مستقلًا؛ السماح بتبديل نقطتي النهاية بشكل منفصل.
-   إصدار تحذير عند بدء التشغيل عندما يكون Chat Completions مفعلاً للإشارة إلى حالته القديمة.

## مسار إيقاف استخدام Chat Completions

-   الحفاظ على حدود وحدات صارمة: لا توجد أنواع مخططات مشتركة بين responses و chat completions.
-   جعل Chat Completions اختياريًا عبر الإعدادات بحيث يمكن تعطيله دون تغييرات في الكود.
-   تحديث الوثائق لوضع علامة على Chat Completions على أنه قديم بمجرد استقرار `/v1/responses`.
-   خطوة مستقبلية اختيارية: تعيين طلبات Chat Completions إلى معالج Responses لمسار إزالة أبسط.

## مجموعة الدعم للمرحلة الأولى

-   قبول `input` كسلسلة نصية أو `ItemParam[]` مع أدوار الرسائل و `function_call_output`.
-   استخراج رسائل النظام والمطور إلى `extraSystemPrompt`.
-   استخدام أحدث `user` أو `function_call_output` كالرسالة الحالية لتشغيلات الوكيل.
-   رفض أجزاء المحتوى غير المدعومة (الصورة/الملف) مع `invalid_request_error`.
-   إرجاع رسالة مساعد واحدة مع محتوى `output_text`.
-   إرجاع `usage` بقيم صفرية حتى يتم توصيل محاسبة الرموز المميزة.

## استراتيجية التحقق (بدون SDK)

-   تنفيذ مخططات Zod للمجموعة المدعومة من:
    -   `CreateResponseBody`
    -   `ItemParam` + اتحادات أجزاء محتوى الرسالة
    -   `ResponseResource`
    -   أشكال أحداث البث المستخدمة من قبل البوابة
-   الاحتفاظ بالمخططات في وحدة واحدة معزولة لتجنب الانحراف والسماح بتوليد الكود في المستقبل.

## تنفيذ البث المتدفق (المرحلة الأولى)

-   أسطر SSE مع كل من `event:` و `data:`.
-   التسلسل المطلوب (الحد الأدنى القابل للتطبيق):
    -   `response.created`
    -   `response.output_item.added`
    -   `response.content_part.added`
    -   `response.output_text.delta` (تكرار حسب الحاجة)
    -   `response.output_text.done`
    -   `response.content_part.done`
    -   `response.completed`
    -   `[DONE]`

## خطة الاختبارات والتحقق

-   إضافة تغطية شاملة من البداية إلى النهاية لـ `/v1/responses`:
    -   المصادقة مطلوبة
    -   شكل الرد غير المتدفق
    -   ترتيب أحداث البث و `[DONE]`
    -   توجيه الجلسة مع الرؤوس و `user`
-   الحفاظ على `src/gateway/openai-http.test.ts` دون تغيير.
-   يدويًا: استخدام curl إلى `/v1/responses` مع `stream: true` والتحقق من ترتيب الأحداث والحدث النهائي `[DONE]`.

## تحديثات الوثائق (متابعة)

-   إضافة صفحة وثائق جديدة لاستخدام `/v1/responses` والأمثلة.
-   تحديث `/gateway/openai-http-api` بملاحظة قديمة وإشارة إلى `/v1/responses`.

[إعادة هيكلة Browser Evaluate CDP](./browser-evaluate-cdp-refactor.md)[خطة PTY والإشراف على العمليات](./pty-process-supervision.md)