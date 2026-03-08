title: "دليل إعداد و مصادقة لوحة تحكم OpenClaw Gateway"
description: "تعلم كيفية الوصول وتأمين لوحة تحكم واجهة التحكم (Control UI) لـ OpenClaw Gateway. قم بتكوين المصادقة، واستخدام localhost أو الوصول عن بُعد، واستكشاف أخطاء الاتصال وإصلاحها."
keywords: ["لوحة تحكم openclaw", "واجهة تحكم البوابة", "رمز المصادقة", "tailscale serve", "واجهة الويب", "سطح الإدارة", "الوصول عبر localhost", "نفق ssh"]
---

  واجهات الويب

  
# لوحة التحكم

لوحة تحكم البوابة هي واجهة تحكم المتصفح (Control UI) التي يتم تقديمها افتراضيًا عند `/` (يمكن تجاوز ذلك باستخدام `gateway.controlUi.basePath`). فتح سريع (بوابة محلية):

-   [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (أو [http://localhost:18789/](http://localhost:18789/))

مراجع رئيسية:

-   [واجهة التحكم (Control UI)](./control-ui.md) للاستخدام وإمكانيات الواجهة.
-   [Tailscale](../gateway/tailscale.md) لأتمتة Serve/Funnel.
-   [أسطح الويب (Web surfaces)](../web.md) لأنماط الربط (bind modes) وملاحظات الأمان.

يتم فرض المصادقة عند مصافحة WebSocket عبر `connect.params.auth` (رمز أو كلمة مرور). راجع `gateway.auth` في [تكوين البوابة (Gateway configuration)](../gateway/configuration.md). ملاحظة أمنية: واجهة التحكم هي **سطح إدارة** (دردشة، تكوين، موافقات تنفيذ). لا تعرضها للعامة. تحتفظ الواجهة برموز URL للوحة التحكم في الذاكرة للتبويب الحالي وتزيلها من عنوان URL بعد التحميل. يُفضل استخدام localhost، أو Tailscale Serve، أو نفق SSH.

## المسار السريع (مُوصى به)

-   بعد الإعداد الأولي (onboarding)، يفتح CLI لوحة التحكم تلقائيًا ويطبع رابطًا نظيفًا (بدون رمز).
-   إعادة الفتح في أي وقت: `openclaw dashboard` (ينسخ الرابط، يفتح المتصفح إن أمكن، يعرض تلميح SSH إذا كان النظام بدون واجهة رسومية (headless)).
-   إذا طلبت الواجهة المصادقة، الصق الرمز من `gateway.auth.token` (أو `OPENCLAW_GATEWAY_TOKEN`) في إعدادات واجهة التحكم.

## أساسيات الرمز (محلي مقابل عن بُعد)

-   **Localhost**: افتح `http://127.0.0.1:18789/`.
-   **مصدر الرمز**: `gateway.auth.token` (أو `OPENCLAW_GATEWAY_TOKEN`)؛ يمكن لـ `openclaw dashboard` تمريره عبر جزء URL (fragment) للتهيئة لمرة واحدة، لكن واجهة التحكم لا تخزن رموز البوابة في localStorage.
-   إذا كان `gateway.auth.token` مُدارًا بواسطة SecretRef، فإن `openclaw dashboard` يطبع/ينسخ/يفتح عنوان URL بدون رمز عن قصد. هذا يتجنب تعريض الرموز المدارة خارجيًا في سجلات الطرفية (shell)، أو سجل الحافظة (clipboard)، أو وسائط تشغيل المتصفح.
-   إذا تم تكوين `gateway.auth.token` كـ SecretRef ولم يتم حله في طرفيتك الحالية، فإن `openclaw dashboard` لا يزال يطبع عنوان URL بدون رمز بالإضافة إلى توجيهات إعداد مصادقة قابلة للتنفيذ.
-   **ليس localhost**: استخدم Tailscale Serve (بدون رمز لواجهة التحكم/WebSocket إذا كان `gateway.auth.allowTailscale: true`، بافتراض أن مضيف البوابة موثوق؛ واجهات برمجة تطبيقات HTTP لا تزال تحتاج إلى رمز/كلمة مرور)، أو tailnet bind مع رمز، أو نفق SSH. راجع [أسطح الويب (Web surfaces)](../web.md).

## إذا رأيت "غير مصرح" / 1008

-   تأكد من إمكانية الوصول إلى البوابة (محليًا: `openclaw status`؛ عن بُعد: نفق SSH `ssh -N -L 18789:127.0.0.1:18789 user@host` ثم افتح `http://127.0.0.1:18789/`).
-   استرجع أو زود الرمز من مضيف البوابة:
    -   تكوين نصي عادي: `openclaw config get gateway.auth.token`
    -   تكوين مُدار بواسطة SecretRef: حل موفر الأسرار الخارجي أو صدّر `OPENCLAW_GATEWAY_TOKEN` في هذه الطرفية، ثم أعد تشغيل `openclaw dashboard`
    -   لا يوجد رمز مُكون: `openclaw doctor --generate-gateway-token`
-   في إعدادات لوحة التحكم، الصق الرمز في حقل المصادقة، ثم اتصل.

[واجهة التحكم (Control UI)](./control-ui.md)[WebChat](./webchat.md)

---