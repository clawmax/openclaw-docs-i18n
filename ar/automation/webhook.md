

  الأتمتة

  
# ويب هوك

يمكن للبوابة تعريض نقطة نهاية ويب هوك HTTP صغيرة للمشغلات الخارجية.

## التمكين

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    // اختياري: تقييد التوجيه الصريح لـ `agentId` إلى قائمة السماح هذه.
    // احذف أو أضف "*" للسماح بأي وكيل.
    // اضبط [] لرفض كل توجيه صريح لـ `agentId`.
    allowedAgentIds: ["hooks", "main"],
  },
}
```

ملاحظات:

-   `hooks.token` مطلوب عندما يكون `hooks.enabled=true`.
-   `hooks.path` الافتراضي هو `/hooks`.

## المصادقة

يجب أن تتضمن كل طلب رمز الويب هوك. يُفضل استخدام الرؤوس:

-   `Authorization: Bearer ` (مُوصى به)
-   `x-openclaw-token: `
-   يتم رفض الرموز في سلسلة الاستعلام (`?token=...` تُرجع `400`).

## نقاط النهاية

### POST /hooks/wake

الحمولة:

```json
{ "text": "System line", "mode": "now" }
```

-   `text` **مطلوب** (سلسلة نصية): وصف الحدث (مثال: "تم استلام بريد إلكتروني جديد").
-   `mode` اختياري (`now` | `next-heartbeat`): ما إذا كان سيتم تشغيل نبضة قلب فورية (الافتراضي `now`) أو الانتظار للفحص الدوري التالي.

التأثير:

-   يُضيف حدث نظام إلى قائمة الانتظار للجلسة **الرئيسية**
-   إذا كان `mode=now`، يُشغل نبضة قلب فورية

### POST /hooks/agent

الحمولة:

```json
{
  "message": "Run this",
  "name": "Email",
  "agentId": "hooks",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

-   `message` **مطلوب** (سلسلة نصية): المطالبة أو الرسالة التي يجب على الوكيل معالجتها.
-   `name` اختياري (سلسلة نصية): اسم مقروء للإنسان للويب هوك (مثال: "GitHub")، يُستخدم كبادئة في ملخصات الجلسة.
-   `agentId` اختياري (سلسلة نصية): توجيه هذا الويب هوك إلى وكيل محدد. تُستخدم الهويات غير المعروفة الوكيل الافتراضي. عند التعيين، يعمل الويب هوك باستخدام مساحة عمل وتكوين الوكيل المُحدد.
-   `sessionKey` اختياري (سلسلة نصية): المفتاح المستخدم لتحديد جلسة الوكيل. بشكل افتراضي، يتم رفض هذا الحقل ما لم يكن `hooks.allowRequestSessionKey=true`.
-   `wakeMode` اختياري (`now` | `next-heartbeat`): ما إذا كان سيتم تشغيل نبضة قلب فورية (الافتراضي `now`) أو الانتظار للفحص الدوري التالي.
-   `deliver` اختياري (منطقي): إذا كان `true`، سيتم إرسال رد الوكيل إلى قناة المراسلة. الافتراضي هو `true`. يتم تخطي الردود التي هي مجرد تأكيدات نبضة قلب تلقائيًا.
-   `channel` اختياري (سلسلة نصية): قناة المراسلة للتسليم. أحد الخيارات: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (المكون الإضافي), `signal`, `imessage`, `msteams`. الافتراضي هو `last`.
-   `to` اختياري (سلسلة نصية): معرف المستلم للقناة (مثال: رقم الهاتف لـ WhatsApp/Signal، معرف الدردشة لـ Telegram، معرف القناة لـ Discord/Slack/Mattermost (المكون الإضافي)، معرف المحادثة لـ MS Teams). الافتراضي هو آخر مستلم في الجلسة الرئيسية.
-   `model` اختياري (سلسلة نصية): تجاوز النموذج (مثال: `anthropic/claude-3-5-sonnet` أو اسم مستعار). يجب أن يكون في قائمة النماذج المسموح بها إذا كانت مقيدة.
-   `thinking` اختياري (سلسلة نصية): تجاوز مستوى التفكير (مثال: `low`, `medium`, `high`).
-   `timeoutSeconds` اختياري (رقم): المدة القصوى لتشغيل الوكيل بالثواني.

التأثير:

-   يُشغل دورة وكيل **معزولة** (مفتاح جلسة خاص)
-   ينشر دائمًا ملخصًا في الجلسة **الرئيسية**
-   إذا كان `wakeMode=now`، يُشغل نبضة قلب فورية

## سياسة مفتاح الجلسة (تغيير غير متوافق)

تجاوزات الحمولة `sessionKey` لـ `/hooks/agent` معطلة بشكل افتراضي.

-   موصى به: اضبط `hooks.defaultSessionKey` ثابتًا وأبْقِ تجاوزات الطلب معطلة.
-   اختياري: اسمح بتجاوزات الطلب فقط عند الحاجة، واقصر البادئات.

التكوين الموصى به:

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

التكوين المتوافق (السلوك القديم):

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // موصى به بشدة
  },
}
```

### POST /hooks/&lt;name&gt; (معين)

يتم حل أسماء الويب هوك المخصصة عبر `hooks.mappings` (انظر التكوين). يمكن للتعيين تحويل الحمولات التعسفية إلى إجراءات `wake` أو `agent`، مع قوالب أو تحويلات كود اختيارية. خيارات التعيين (ملخص):

-   `hooks.presets: ["gmail"]` يُمكن التعيين المدمج لـ Gmail.
-   `hooks.mappings` يتيح لك تعريف `match`, `action`, والقوالب في التكوين.
-   `hooks.transformsDir` + `transform.module` يحمل وحدة JS/TS لمنطق مخصص.
    -   `hooks.transformsDir` (إذا تم تعيينه) يجب أن يبقى ضمن جذر التحويلات تحت دليل تكوين OpenClaw الخاص بك (عادة `~/.openclaw/hooks/transforms`).
    -   `transform.module` يجب أن يحل داخل دليل التحويلات الفعلي (يتم رفض مسارات التنقل/الهروب).
-   استخدم `match.source` للحفاظ على نقطة نهاية استقبال عامة (توجيه مدفوع بالحمولة).
-   تتطلب تحويلات TS محمل TS (مثل `bun` أو `tsx`) أو ملف `.js` مُترجم مسبقًا في وقت التشغيل.
-   اضبط `deliver: true` + `channel`/`to` في التعيينات لتوجيه الردود إلى سطح دردشة (الافتراضي `channel` هو `last` ويرجع إلى WhatsApp).
-   `agentId` يُوجه الويب هوك إلى وكيل محدد؛ الهويات غير المعروفة ترجع إلى الوكيل الافتراضي.
-   `hooks.allowedAgentIds` يقيد التوجيه الصريح لـ `agentId`. احذفه (أو أضف `*`) للسماح بأي وكيل. اضبط `[]` لرفض التوجيه الصريح لـ `agentId`.
-   `hooks.defaultSessionKey` يضبط الجلسة الافتراضية لتشغيلات وكيل الويب هوك عندما لا يتم توفير مفتاح صريح.
-   `hooks.allowRequestSessionKey` يتحكم فيما إذا كانت حمولات `/hooks/agent` يمكنها تعيين `sessionKey` (الافتراضي: `false`).
-   `hooks.allowedSessionKeyPrefixes` يقيد اختياريًا قيم `sessionKey` الصريحة من حمولات الطلب والتعيينات.
-   `allowUnsafeExternalContent: true` يعطل غلاف أمان المحتوى الخارجي لذلك الويب هوك (خطير؛ فقط للمصادر الداخلية الموثوقة).
-   `openclaw webhooks gmail setup` يكتب تكوين `hooks.gmail` لـ `openclaw webhooks gmail run`. انظر [Gmail Pub/Sub](./gmail-pubsub.md) لسلسلة مراقبة Gmail الكاملة.

## الردود

-   `200` لـ `/hooks/wake`
-   `200` لـ `/hooks/agent` (تم قبول التشغيل غير المتزامن)
-   `401` عند فشل المصادقة
-   `429` بعد فشل متكرر في المصادقة من نفس العميل (تحقق من `Retry-After`)
-   `400` عند حمولة غير صالحة
-   `413` عند حمولات كبيرة الحجم

## أمثلة

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### استخدام نموذج مختلف

أضف `model` إلى حمولة الوكيل (أو التعيين) لتجاوز النموذج لتلك الدورة:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

إذا فرضت `agents.defaults.models`، تأكد من تضمين النموذج المتجاوز هناك.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## الأمان

-   احتفظ بنقاط نهاية الويب هوك خلف loopback، أو tailnet، أو وكيل عكسي موثوق.
-   استخدم رمز ويب هوك مخصص؛ لا تعيد استخدام رموز مصادقة البوابة.
-   يتم تحديد معدل فشل المصادقة المتكررة لكل عنوان عميل لإبطاء محاولات القوة الغاشمة.
-   إذا كنت تستخدم توجيه متعدد الوكلاء، اضبط `hooks.allowedAgentIds` للحد من اختيار `agentId` الصريح.
-   أبقِ `hooks.allowRequestSessionKey=false` ما لم تكن بحاجة إلى جلسات محددة من قبل المتصل.
-   إذا مكنت `sessionKey` للطلب، قيد `hooks.allowedSessionKeyPrefixes` (على سبيل المثال، `["hook:"]`).
-   تجنب تضمين الحمولات الأولية الحساسة في سجلات الويب هوك.
-   يتم التعامل مع حمولات الويب هوك على أنها غير موثوقة ويتم تغليفها بحدود أمان بشكل افتراضي. إذا كان يجب تعطيل هذا لويب هوك محدد، اضبط `allowUnsafeExternalContent: true` في تعيين ذلك الويب هوك (خطير).

[استكشاف أخطاء الأتمتة وإصلاحها](./troubleshooting.md)[Gmail PubSub](./gmail-pubsub.md)