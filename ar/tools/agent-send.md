title: "أمر إرسال الوكيل لتنسيق وكيل OpenClaw"
description: "تعلم كيفية استخدام أمر openclaw agent send لتشغيل دورة وكيل واحدة، وإدارة الجلسات، وتسليم الردود إلى قنوات مختلفة."
keywords: ["وكيل openclaw", "تنسيق الوكيل", "أمر واجهة سطر الأوامر", "إرسال الوكيل", "إدارة الجلسات", "تسليم الرسائل", "وقت التشغيل المضمن"]
---

  تنسيق الوكيل

  
# إرسال الوكيل

`openclaw agent` يشغل دورة وكيل واحدة دون الحاجة إلى رسالة دردشة واردة. بشكل افتراضي، يمر **عبر البوابة**؛ أضف `--local` لإجبار وقت التشغيل المضمن على الجهاز الحالي.

## السلوك

-   مطلوب: `--message <نص>`
-   اختيار الجلسة:
    -   `--to <الوجهة>` تستنتج مفتاح الجلسة (الأهداف الجماعية/القنوية تحافظ على العزل؛ الدردشات المباشرة تنهار إلى `main`)، **أو**
    -   `--session-id <المعرف>` يعيد استخدام جلسة موجودة بواسطة المعرف، **أو**
    -   `--agent <المعرف>` يستهدف وكيلًا مُهيأً مباشرةً (يستخدم مفتاح الجلسة `main` لذلك الوكيل)
-   يشغل وقت تشغيل الوكيل المضمن نفسه كما في الردود الواردة العادية.
-   أعلام التفكير/التفصيل تبقى مخزنة في مخزن الجلسات.
-   المخرجات:
    -   الافتراضي: يطبع نص الرد (بالإضافة إلى أسطر `MEDIA:<رابط>`)
    -   `--json`: يطبع الحمولة المنظمة + البيانات الوصفية
-   تسليم اختياري إلى قناة باستخدام `--deliver` + `--channel` (تطابق تنسيقات الهدف `openclaw message --target`).
-   استخدم `--reply-channel`/`--reply-to`/`--reply-account` لتجاوز التسليم دون تغيير الجلسة.

إذا كانت البوابة غير قابلة للوصول، فإن واجهة سطر الأوامر **ترجع** إلى التشغيل المحلي المضمن.

## أمثلة

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## الأعلام

-   `--local`: التشغيل محليًا (يتطلب مفاتيح واجهة برمجة التطبيقات لمزود النموذج في محطتك)
-   `--deliver`: إرسال الرد إلى القناة المختارة
-   `--channel`: قناة التسليم (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`، الافتراضي: `whatsapp`)
-   `--reply-to`: تجاوز هدف التسليم
-   `--reply-channel`: تجاوز قناة التسليم
-   `--reply-account`: تجاوز معرف حساب التسليم
-   `--thinking <off|minimal|low|medium|high|xhigh>`: الحفاظ على مستوى التفكير (نماذج GPT-5.2 + Codex فقط)
-   `--verbose <on|full|off>`: الحفاظ على مستوى التفصيل
-   `--timeout <ثوانٍ>`: تجاوز مهلة الوكيل
-   `--json`: إخراج JSON منظم

[استكشاف أخطاء المتصفح وإصلاحها](./browser-linux-troubleshooting.md)[الوكلاء الفرعيون](./subagents.md)

---