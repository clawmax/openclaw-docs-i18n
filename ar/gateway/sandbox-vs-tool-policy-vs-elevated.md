

  الأمان والتجريد

  
# الصندوق الرملي مقابل سياسة الأدوات مقابل الوضع المتقدم

يحتوي OpenClaw على ثلاثة ضوابط مرتبطة (لكن مختلفة):

1.  **الصندوق الرملي** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) يقرر **مكان تشغيل الأدوات** (Docker مقابل المضيف).
2.  **سياسة الأدوات** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) تقرر **الأدوات المتاحة/المسموح بها**.
3.  **الوضع المتقدم** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) هو **منفذ هروب يعمل بـ exec فقط** للتشغيل على المضيف عندما تكون في الصندوق الرملي.

## تصحيح سريع

استخدم المفتش لمعرفة ما يفعله OpenClaw *فعليًا*:

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

يطبع:

-   وضع/نطاق/وصول مساحة العمل الفعال للصندوق الرملي
-   ما إذا كانت الجلسة حاليًا في الصندوق الرملي (رئيسية مقابل غير رئيسية)
-   القائمة المسموح بها/المرفوضة للأدوات في الصندوق الرملي الفعال (وما إذا كان مصدرها من الوكيل/العام/الافتراضي)
-   بوابات الوضع المتقدم ومسارات مفاتيح الإصلاح

## الصندوق الرملي: مكان تشغيل الأدوات

يتم التحكم في التجريد بواسطة `agents.defaults.sandbox.mode`:

-   `"off"`: كل شيء يعمل على المضيف.
-   `"non-main"`: يتم تجريد الجلسات غير الرئيسية فقط (شائع في "المفاجآت" للمجموعات/القنوات).
-   `"all"`: يتم تجريد كل شيء.

راجع [التجريد](./sandboxing.md) للحصول على المصفوفة الكاملة (النطاق، نقاط تركيب مساحة العمل، الصور).

### نقاط الربط (فحص أمني سريع)

-   `docker.binds` *يخترق* نظام ملفات الصندوق الرملي: أي شيء تقوم بتركيبه يكون مرئيًا داخل الحاوية بالوضع الذي قمت بتعيينه (`:ro` أو `:rw`).
-   الافتراضي هو القراءة والكتابة إذا حذفت الوضع؛ يُفضل `:ro` للمصادر/الأسرار.
-   `scope: "shared"` يتجاهل نقاط الربط الخاصة بكل وكيل (تطبق فقط نقاط الربط العامة).
-   ربط `/var/run/docker.sock` يسلم فعليًا تحكم المضيف إلى الصندوق الرملي؛ افعل هذا فقط عن قصد.
-   الوصول إلى مساحة العمل (`workspaceAccess: "ro"`/`"rw"`) مستقل عن أوضاع الربط.

## سياسة الأدوات: الأدوات الموجودة/القابلة للاستدعاء

هناك طبقتان مهمتان:

-   **ملف تعريف الأداة**: `tools.profile` و `agents.list[].tools.profile` (القائمة المسموح بها الأساسية)
-   **ملف تعريف أداة المزود**: `tools.byProvider[provider].profile` و `agents.list[].tools.byProvider[provider].profile`
-   **سياسة الأداة العامة/لكل وكيل**: `tools.allow`/`tools.deny` و `agents.list[].tools.allow`/`agents.list[].tools.deny`
-   **سياسة أداة المزود**: `tools.byProvider[provider].allow/deny` و `agents.list[].tools.byProvider[provider].allow/deny`
-   **سياسة أداة الصندوق الرملي** (تطبق فقط عند التجريد): `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` و `agents.list[].tools.sandbox.tools.*`

قواعد عامة:

-   `deny` تفوز دائمًا.
-   إذا كانت `allow` غير فارغة، يتم التعامل مع كل شيء آخر على أنه محظور.
-   سياسة الأداة هي التوقف الصارم: لا يمكن لـ `/exec` تجاوز أداة `exec` مرفوضة.
-   `/exec` يغير فقط الإعدادات الافتراضية للجلسة للمرسلين المصرح لهم؛ لا يمنح وصولاً للأدوات. تقبل مفاتيح أداة المزود إما `provider` (مثل `google-antigravity`) أو `provider/model` (مثل `openai/gpt-5.2`).

### مجموعات الأدوات (اختصارات)

تدعم سياسات الأدوات (العامة، الوكيل، الصندوق الرملي) إدخالات `group:*` التي تتوسع إلى أدوات متعددة:

```json
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"],
      },
    },
  },
}
```

المجموعات المتاحة:

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: جميع أدوات OpenClaw المدمجة (تستثني إضافات المزود)

## الوضع المتقدم: "التشغيل على المضيف" بـ exec فقط

الوضع المتقدم **لا** يمنح أدوات إضافية؛ يؤثر فقط على `exec`.

-   إذا كنت في الصندوق الرملي، فإن `/elevated on` (أو `exec` مع `elevated: true`) يعمل على المضيف (قد لا تزال الموافقات تنطبق).
-   استخدم `/elevated full` لتخطي موافقات exec للجلسة.
-   إذا كنت تعمل مباشرة بالفعل، فإن الوضع المتقدم لا يؤثر فعليًا (لا يزال محميًا بالبوابات).
-   الوضع المتقدم **ليس** محددًا بالمهارة و**لا** يتجاوز السماح/الرفض للأداة.
-   `/exec` منفصل عن الوضع المتقدم. يقوم فقط بتعديل الإعدادات الافتراضية لـ exec لكل جلسة للمرسلين المصرح لهم.

البوابات:

-   التمكين: `tools.elevated.enabled` (واختياريًا `agents.list[].tools.elevated.enabled`)
-   قوائم السماح للمرسلين: `tools.elevated.allowFrom.` (واختياريًا `agents.list[].tools.elevated.allowFrom.`)

راجع [الوضع المتقدم](../tools/elevated.md).

## إصلاحات شائعة لـ "سجن الصندوق الرملي"

### "تم حظر الأداة X بواسطة سياسة أداة الصندوق الرملي"

مفاتيح الإصلاح (اختر واحدة):

-   تعطيل الصندوق الرملي: `agents.defaults.sandbox.mode=off` (أو لكل وكيل `agents.list[].sandbox.mode=off`)
-   السماح بالأداة داخل الصندوق الرملي:
    -   إزالتها من `tools.sandbox.tools.deny` (أو لكل وكيل `agents.list[].tools.sandbox.tools.deny`)
    -   أو إضافتها إلى `tools.sandbox.tools.allow` (أو السماح لكل وكيل)

### "اعتقدت أن هذه جلسة رئيسية، لماذا هي في الصندوق الرملي؟"

في وضع `"non-main"`، مفاتيح المجموعة/القناة *ليست* رئيسية. استخدم مفتاح الجلسة الرئيسية (الذي يظهره `sandbox explain`) أو غيّر الوضع إلى `"off"`.

[التجريد](./sandboxing.md)[بروتوكول البوابة](./protocol.md)