

  أوامر CLI

  
# channels

إدارة حسابات قنوات الدردشة وحالة تشغيلها على البوابة. وثائق ذات صلة:

-   أدلة القنوات: [القنوات](../channels/index.md)
-   تهيئة البوابة: [التهيئة](../gateway/configuration.md)

## الأوامر الشائعة

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

## إضافة / إزالة الحسابات

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

نصيحة: `openclaw channels add --help` يعرض علامات خاصة بكل قناة (الرمز المميز، رمز التطبيق، مسارات signal-cli، إلخ). عند تشغيل `openclaw channels add` بدون علامات، يمكن للبرنامج التفاعلي أن يطلب:

-   معرفات الحسابات لكل قناة محددة
-   أسماء عرض اختيارية لتلك الحسابات
-   `ربط حسابات القناة المهيئة بالوكلاء الآن؟`

إذا قمت بتأكيد الربط الآن، يسأل البرنامج التفاعلي أي وكيل يجب أن يمتلك كل حساب قناة مهيء ويكتب قواعد توجيه محددة بالنطاق الحسابي. يمكنك أيضًا إدارة نفس قواعد التوجيه لاحقًا باستخدام `openclaw agents bindings`، و `openclaw agents bind`، و `openclaw agents unbind` (انظر [الوكلاء](./agents.md)). عند إضافة حساب غير افتراضي إلى قناة لا تزال تستخدم إعدادات المستوى الأعلى لحساب واحد (لا توجد إدخالات `channels..accounts` بعد)، يقوم OpenClaw بنقل القيم ذات النطاق الحسابي من المستوى الأعلى إلى `channels..accounts.default`، ثم يكتب الحساب الجديد. هذا يحافظ على سلوك الحساب الأصلي مع الانتقال إلى شكل الحسابات المتعددة. يظل سلوك التوجيه متسقًا:

-   عمليات الربط الحالية للقناة فقط (بدون `accountId`) تستمر في مطابقة الحساب الافتراضي.
-   `channels add` لا ينشئ تلقائيًا ولا يعيد كتابة عمليات الربط في الوضع غير التفاعلي.
-   يمكن للإعداد التفاعلي إضافة عمليات ربط محددة بالنطاق الحسابي بشكل اختياري.

إذا كان تكوينك في حالة مختلطة بالفعل (حسابات مسماة موجودة، `default` مفقود، وقيم حساب واحد من المستوى الأعلى لا تزال مضبوطة)، قم بتشغيل `openclaw doctor --fix` لنقل القيم محددة النطاق الحسابي إلى `accounts.default`.

## تسجيل الدخول / الخروج (تفاعلي)

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

## استكشاف الأخطاء وإصلاحها

-   قم بتشغيل `openclaw status --deep` لفحص شامل.
-   استخدم `openclaw doctor` للإصلاحات الموجهة.
-   `openclaw channels list` تطبع `Claude: HTTP 403 ... user:profile` → لقطة الاستخدام تحتاج إلى نطاق `user:profile`. استخدم `--no-usage`، أو قدم مفتاح جلسة claude.ai (`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`)، أو أعد المصادقة عبر Claude Code CLI.
-   `openclaw channels status` يتراجع إلى ملخصات التكوين فقط عندما تكون البوابة غير قابلة للوصول. إذا تم تكوين بيانات اعتماد قناة مدعومة عبر SecretRef ولكنها غير متاحة في مسار الأمر الحالي، فإنه يبلغ عن هذا الحساب كمهيء مع ملاحظات متدهورة بدلاً من إظهاره على أنه غير مهيء.

## فحص القدرات

جلب تلميحات قدرات المزود (النوايا/النطاقات حيثما كانت متاحة) بالإضافة إلى دعم الميزات الثابتة:

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

ملاحظات:

-   `--channel` اختياري؛ احذفه لسرد كل قناة (بما في ذلك الامتدادات).
-   `--target` يقبل `channel:` أو معرف قناة رقمي خام وينطبق فقط على Discord.
-   الفحوصات خاصة بالمزود: نوايا Discord + أذونات القناة الاختيارية؛ نطاقات بوت Slack + المستخدم؛ أعلام بوت Telegram + webhook؛ إصدار خادم Signal؛ رمز تطبيق MS Teams + أدوار/نطاقات Graph (مع تعليقات توضيحية حيثما كانت معروفة). القنوات التي لا تحتوي على فحوصات تبلغ `Probe: unavailable`.

## تحويل الأسماء إلى معرفات

تحويل أسماء القنوات/المستخدمين إلى معرفات باستخدام دليل المزود:

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

ملاحظات:

-   استخدم `--kind user|group|auto` لإجبار نوع الهدف.
-   يفضل التحويل المطابقات النشطة عندما تتشارك إدخالات متعددة نفس الاسم.
-   `channels resolve` للقراءة فقط. إذا تم تكوين حساب محدد عبر SecretRef ولكن بيانات الاعتماد هذه غير متاحة في مسار الأمر الحالي، فإن الأمر يعيد نتائج غير محللة متدهورة مع ملاحظات بدلاً من إلغاء التشغيل بالكامل.

[browser](./browser.md)[clawbot](./clawbot.md)