

  الأمان والسندبة

  
# الأمان

> \[!WARNING\] **نموذج ثقة المساعد الشخصي:** يفترض هذا التوجيه وجود حد ثقة واحد لكل بوابة (نموذج مستخدم واحد/مساعد شخصي). OpenClaw **ليس** حد أمان متعدد مستأجرين معادٍ لمستخدمين متعددين متعارضين يتشاركون وكيل/بوابة واحدة. إذا كنت بحاجة إلى تشغيل مختلط الثقة أو مستخدمين معاديين، قم بتقسيم حدود الثقة (بوابة منفصلة + بيانات اعتماد، ويفضل مستخدمين/مضيفين نظام تشغيل منفصلين).

## النطاق أولاً: نموذج أمان المساعد الشخصي

يفترض توجيه أمان OpenClaw نشر **مساعد شخصي**: حد مشغل موثوق واحد، وربما العديد من الوكلاء.

-   وضع الأمان المدعوم: مستخدم/حد ثقة واحد لكل بوابة (يفضل مستخدم/مضيف/VPS نظام تشغيل واحد لكل حد).
-   ليس حد أمان مدعومًا: بوابة/وكيل مشترك واحد يستخدمه مستخدمون غير موثوق بهم أو معاديين بشكل متبادل.
-   إذا كان عزل المستخدم المعادي مطلوبًا، قم بالتقسيم حسب حد الثقة (بوابة منفصلة + بيانات اعتماد، ويفضل مستخدمين/مضيفين نظام تشغيل منفصلين).
-   إذا كان بإمكان مستخدمين غير موثوق بهم إرسال رسائل إلى وكيل واحد مدعوم بالأدوات، فاعتبرهم يتشاركون نفس سلطة الأداة المفوضة لذلك الوكيل.

تشرح هذه الصفحة التصلب **ضمن ذلك النموذج**. لا تدعي عزلًا متعدد مستأجرين معاديين على بوابة مشتركة واحدة.

## فحص سريع: تدقيق أمان openclaw

انظر أيضًا: [التحقق الرسمي (نماذج الأمان)](../security/formal-verification.md) قم بتشغيل هذا بانتظام (خاصة بعد تغيير التكوين أو تعرض أسطح الشبكة):

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

يقوم بوضع علامة على المشاكل الشائعة (تعرض مصادقة البوابة، تعرض تحكم المتصفح، قوائم السماح المرتفعة، أذونات نظام الملفات). OpenClaw هو منتج وتجربة في نفس الوقت: أنت تقوم بتوصيل سلوك النموذج المتقدم بأسطح مراسلة حقيقية وأدوات حقيقية. **لا يوجد إعداد "آمن تمامًا".** الهدف هو أن تكون متعمدًا بشأن:

-   من يمكنه التحدث إلى بوتك
-   أين يُسمح للبوت بالتصرف
-   ما الذي يمكن للبوت لمسه

ابدأ بأصغر وصول لا يزال يعمل، ثم وسعه كلما اكتسبت الثقة.

## افتراض النشر (مهم)

يفترض OpenClaw أن المضيف وحد التكوين موثوقان:

-   إذا كان بإمكان شخص ما تعديل حالة/تكوين مضيف البوابة (`~/.openclaw`، بما في ذلك `openclaw.json`)، فاعتبره مشغلًا موثوقًا.
-   تشغيل بوابة واحدة لمشغلين متعددين غير موثوق بهم/معاديين بشكل متبادل **ليس إعدادًا موصى به**.
-   للفرق ذات الثقة المختلطة، قم بتقسيم حدود الثقة بوابات منفصلة (أو على الأقل مستخدمين/مضيفين نظام تشغيل منفصلين).
-   يمكن لـ OpenClaw تشغيل مثيلات بوابات متعددة على جهاز واحد، لكن العمليات الموصى بها تفضل فصل حدود الثقة بوضوح.
-   الإعداد الافتراضي الموصى به: مستخدم واحد لكل جهاز/مضيف (أو VPS)، بوابة واحدة لذلك المستخدم، وواحد أو أكثر من الوكلاء في تلك البوابة.
-   إذا أراد عدة مستخدمين OpenClaw، استخدم VPS/مضيف واحد لكل مستخدم.

### النتيجة العملية (حد ثقة المشغل)

داخل مثيل بوابة واحد، الوصول المصادق للمشغل هو دور مستوى تحكم موثوق، وليس دور مستأجر لكل مستخدم.

-   يمكن للمشغلين الذين لديهم وصول قراءة/مستوى تحكم تصميمًا فحص بيانات تعريف/تاريخ جلسة البوابة.
-   معرفات الجلسة (`sessionKey`، معرفات الجلسة، التسميات) هي محددات توجيه، وليست رموز تفويض.
-   مثال: توقع عزل لكل مشغل لطرق مثل `sessions.list`، `sessions.preview`، أو `chat.history` هو خارج هذا النموذج.
-   إذا كنت بحاجة إلى عزل مستخدم معادي، قم بتشغيل بوابات منفصلة لكل حد ثقة.
-   من الممكن تقنيًا وجود بوابات متعددة على جهاز واحد، لكنها ليست الأساس الموصى به للعزل متعدد المستخدمين.

## نموذج المساعد الشخصي (ليس حافلة متعددة المستأجرين)

تم تصميم OpenClaw كنموذج أمان للمساعد الشخصي: حد مشغل موثوق واحد، وربما العديد من الوكلاء.

-   إذا كان بإمكان عدة أشخاص إرسال رسائل إلى وكيل واحد مدعوم بالأدوات، يمكن لكل منهم توجيه نفس مجموعة الأذونات.
-   يساعد عزل الجلسة/الذاكرة لكل مستخدم في الخصوصية، لكنه لا يحول الوكيل المشترك إلى تفويض مضيف لكل مستخدم.
-   إذا كان المستخدمون قد يكونون معاديين لبعضهم البعض، قم بتشغيل بوابات منفصلة (أو مستخدمين/مضيفين نظام تشغيل منفصلين) لكل حد ثقة.

### مساحة عمل Slack المشتركة: خطر حقيقي

إذا كان "الجميع في Slack يمكنهم مراسلة البوت"، فإن الخطر الأساسي هو سلطة الأداة المفوضة:

-   يمكن لأي مرسل مسموح به تحفيز استدعاءات الأدوات (`exec`، متصفح، أدوات شبكة/ملف) ضمن سياسة الوكيل؛
-   يمكن لحقن المطالبة/المحتوى من مرسل واحد أن يتسبب في إجراءات تؤثر على الحالة المشتركة، الأجهزة، أو المخرجات؛
-   إذا كان لدى وكيل مشترك واحد بيانات اعتماد/ملفات حساسة، يمكن لأي مرسل مسموح به محتملًا قيادة تسريب عبر استخدام الأداة.

استخدم وكلاء/بوابات منفصلة بأدوات دنيا لسير عمل الفريق؛ احتفظ بوكلاء البيانات الشخصية خاصة.

### الوكيل المشترك للشركة: نمط مقبول

هذا مقبول عندما يكون كل من يستخدم ذلك الوكيل في نفس حد الثقة (على سبيل المثال فريق شركة واحد) ويكون الوكيل محصورًا بشكل صارم في نطاق العمل.

-   قم بتشغيله على جهاز/آلة افتراضية/حاوية مخصصة؛
-   استخدم مستخدم نظام تشغيل مخصص + متصفح/ملف تعريف/حسابات مخصصة لذلك وقت التشغيل؛
-   لا تقم بتسجيل دخول ذلك وقت التشغيل إلى حسابات Apple/Google الشخصية أو ملفات تعريف مدير كلمات المرور/المتصفح الشخصية.

إذا قمت بخلط الهويات الشخصية والشركات على نفس وقت التشغيل، فإنك تدمج الفصل وتزيد من خطر تعرض البيانات الشخصية.

## مفهوم ثقة البوابة والعقدة

اعتبر البوابة والعقدة كمجال ثقة مشغل واحد، بأدوار مختلفة:

-   **البوابة** هي مستوى التحط والسياسة (`gateway.auth`، سياسة الأداة، التوجيه).
-   **العقدة** هي سطح تنفيذ بعيد مقترن بتلك البوابة (أوامر، إجراءات الجهاز، إمكانيات محلية للمضيف).
-   المتصل المصادق للبوابة موثوق به في نطاق البوابة. بعد الاقتران، إجراءات العقدة هي إجراءات مشغل موثوق على تلك العقدة.
-   `sessionKey` هو اختيار توجيه/سياق، وليس مصادقة لكل مستخدم.
-   موافقات التنفيذ (قائمة السماح + سؤال) هي حواجز لحماية نية المشغل، وليست عزلًا متعدد مستأجرين معاديين.

إذا كنت بحاجة إلى عزل مستخدم معادي، قم بتقسيم حدود الثقة حسب مستخدم/مضيف نظام التشغيل وقم بتشغيل بوابات منفصلة.

## مصفوفة حدود الثقة

استخدم هذا كنموذج سريع عند فرز المخاطر:

| الحد أو التحكم | ماذا يعني | سوء الفهم الشائع |
| --- | --- | --- |
| `gateway.auth` (رمز/كلمة مرور/مصادقة جهاز) | يصادق المتصلين بواجهات برمجة تطبيقات البوابة | "يحتاج إلى توقيعات لكل رسالة على كل إطار ليكون آمنًا" |
| `sessionKey` | مفتاح توجيه لاختيار السياق/الجلسة | "مفتاح الجلسة هو حد تفويض مستخدم" |
| حواجز المطالبة/المحتوى | تقليل خطر إساءة استخدام النموذج | "حقن المطالبة وحده يثبت تجاوز المصادقة" |
| `canvas.eval` / تقييم المتصفح | قدرة مشغل مقصودة عند تمكينها | "أي بدائية تقييم JS هي تلقائيًا ثغرة في نموذج الثقة هذا" |
| Shell المحلي TUI `!` | تنفيذ محلي مطلق بواسطة المشغل | "أمر shell المحلي المريح هو حقن بعيد" |
| اقتران العقدة وأوامر العقدة | تنفيذ بعيد على مستوى المشغل على الأجهزة المقترنة | "يجب معاملة التحكم في الجهاز البعيد كوصول مستخدم غير موثوق افتراضيًا" |

## ليست ثغرات حسب التصميم

هذه الأنماط يتم الإبلاغ عنها بشكل شائع وعادة ما يتم إغلاقها كـ "لا إجراء" ما لم يتم عرض تجاوز حد حقيقي:

-   سلاسل حقن مطالبة فقط بدون تجاوز سياسة/مصادقة/سندبة.
-   ادعاءات تفترض تشغيل مستأجرين معاديين متعددين على مضيف/تكوين مشترك واحد.
-   ادعاءات تصنف وصول مسار قراءة المشغل العادي (على سبيل المثال `sessions.list`/`sessions.preview`/`chat.history`) كـ IDOR في إعداد بوابة مشتركة.
-   نتائج نشر localhost فقط (على سبيل المثال HSTS على بوابة loopback فقط).
-   نتائج توقيع webhook الوارد لـ Discord لمسارات واردة غير موجودة في هذا المستودع.
-   نتائج "تفويض مفقود لكل مستخدم" تعامل `sessionKey` كرمز مصادقة.

## قائمة التحقق المسبق للباحث

قبل فتح GHSA، تحقق من كل هذه:

1.  لا يزال إعادة الإنتاج يعمل على أحدث `main` أو أحدث إصدار.
2.  التقرير يتضمن مسار كود دقيق (`file`، دالة، نطاق أسطر) وإصدار/commit تم اختباره.
3.  التأثير يتجاوز حد ثقة موثق (ليس مجرد حقن مطالبة).
4.  الادعاء غير مدرج في [خارج النطاق](https://github.com/openclaw/openclaw/blob/main/SECURITY.md#out-of-scope).
5.  تم فحص الإخطارات الحالية بحثًا عن تكرارات (إعادة استخدام GHSA أساسي عند التطبيق).
6.  افتراضات النشر صريحة (loopback/محلي مقابل معرض، مشغلين موثوقين مقابل غير موثوقين).

## خط أساس متصلب في 60 ثانية

استخدم هذا الخط الأساسي أولاً، ثم أعد تمكين الأدوات بشكل انتقائي لكل وكيل موثوق:

```json
{
  gateway: {
    mode: "local",
    bind: "loopback",
    auth: { mode: "token", token: "replace-with-long-random-token" },
  },
  session: {
    dmScope: "per-channel-peer",
  },
  tools: {
    profile: "messaging",
    deny: ["group:automation", "group:runtime", "group:fs", "sessions_spawn", "sessions_send"],
    fs: { workspaceOnly: true },
    exec: { security: "deny", ask: "always" },
    elevated: { enabled: false },
  },
  channels: {
    whatsapp: { dmPolicy: "pairing", groups: { "*": { requireMention: true } } },
  },
}
```

هذا يحافظ على البوابة محلية فقط، يعزل الرسائل المباشرة، ويعطل أدوات مستوى التحكم/وقت التشغيل افتراضيًا.

## قاعدة سريعة للصندوق الوارد المشترك

إذا كان بإمكان أكثر من شخص إرسال رسائل مباشرة إلى بوتك:

-   عيّن `session.dmScope: "per-channel-peer"` (أو `"per-account-channel-peer"` للقنوات متعددة الحسابات).
-   احتفظ بـ `dmPolicy: "pairing"` أو قوائم سماح صارمة.
-   لا تجمع أبدًا بين الرسائل المباشرة المشتركة ووصول واسع للأدوات.
-   هذا يقوي صناديق الوارد التعاونية/المشتركة، لكنه غير مصمم كعزل مستأجرين معاديين عندما يتشارك المستخدمون وصول كتابة المضيف/التكوين.

### ما يتحقق منه التدقيق (مستوى عالٍ)

-   **الوصول الوارد** (سياسات الرسائل المباشرة، سياسات المجموعات، قوائم السماح): هل يمكن للغرباء تحفيز البوت؟
-   **نصف قطر انفجار الأداة** (أدوات مرتفعة + غرف مفتوحة): هل يمكن أن يتحول حقن المطالبة إلى إجراءات shell/ملف/شبكة؟
-   **التعرض للشبكة** (ربط/مصادقة البوابة، Tailscale Serve/Funnel، رموز مصادقة ضعيفة/قصيرة).
-   **تعرض تحكم المتصفح** (عقد بعيدة، منافذ relay، نقاط نهاية CDP بعيدة).
-   **نظافة القرص المحلي** (الأذونات، الروابط الرمزية، تضمينات التكوين، مسارات "المجلدات المتزامنة").
-   **الإضافات** (توجد امتدادات بدون قائمة سماح صريحة).
-   **انحراف/سوء تكوين السياسة** (إعدادات docker للسندبة مهيئة لكن وضع السندبة معطل؛ أنماط `gateway.nodes.denyCommands` غير فعالة لأن المطابقة هي اسم الأمر الدقيق فقط (على سبيل المثال `system.run`) ولا تفحص نص shell؛ إدخالات `gateway.nodes.allowCommands` خطيرة؛ `tools.profile="minimal"` عام تم تجاوزه بواسطة ملفات تعريف لكل وكيل؛ أدوات إضافة الامتداد قابلة للوصول تحت سياسة أداة متساهلة).
-   **انحراف توقعات وقت التشغيل** (على سبيل المثال `tools.exec.host="sandbox"` بينما وضع السندبة معطل، مما يعمل مباشرة على مضيف البوابة).
-   **نظافة النموذج** (تحذير عندما تبدو النماذج المهيئة قديمة؛ ليس حظرًا صارمًا).

إذا قمت بتشغيل `--deep`، يحاول OpenClaw أيضًا فحص بوابة حية بأفضل جهد.

## خريطة تخزين بيانات الاعتماد

استخدم هذا عند تدقيق الوصول أو اتخاذ قرار بشأن ما يجب نسخه احتياطيًا:

-   **WhatsApp**: `~/.openclaw/credentials/whatsapp//creds.json`
-   **رمز بوت Telegram**: تكوين/env أو `channels.telegram.tokenFile`
-   **رمز بوت Discord**: تكوين/env أو SecretRef (مزودي env/file/exec)
-   **رموز Slack**: تكوين/env (`channels.slack.*`)
-   **قوائم السماح للاقتران**:
    -   `~/.openclaw/credentials/-allowFrom.json` (الحساب الافتراضي)
    -   `~/.openclaw/credentials/--allowFrom.json` (الحسابات غير الافتراضية)
-   **ملفات تعريف مصادقة النموذج**: `~/.openclaw/agents//agent/auth-profiles.json`
-   **حمولة أسرار مدعومة بملف (اختياري)**: `~/.openclaw/secrets.json`
-   **استيراد OAuth القديم**: `~/.openclaw/credentials/oauth.json`

## قائمة تدقيق الأمان

عندما يطبع التدقيق النتائج، عالج هذا كترتيب أولوية:

1.  **أي شيء "مفتوح" + أدوات ممكّنة**: قفل الرسائل المباشرة/المجموعات أولاً (اقتران/قوائم سماح)، ثم شدد سياسة الأداة/السندبة.
2.  **التعرض للشبكة العامة** (ربط LAN، Funnel، مصادقة مفقودة): أصلح فورًا.
3.  **تعرض تحكم المتصفح البعيد**: عالجه كوصول مشغل (tailnet فقط، اقترن بالعقد عمدًا، تجنب التعرض العام).
4.  **الأذونات**: تأكد من أن الحالة/التكوين/بيانات الاعتماد/المصادقة ليست قابلة للقراءة للمجموعة/العالم.
5.  **الإضافات/الامتدادات**: قم بتحميل ما تثق به صراحة فقط.
6.  **اختيار النموذج**: فضل النماذج الحديثة المقواة بالتوجيه لأي بوت لديه أدوات.

## مسرد تدقيق الأمان

قيم `checkId` عالية الإشارة من المرجح أن تراها في عمليات النشر الحقيقية (ليست شاملة):

| `checkId` | الشدة | لماذا يهم | مفتاح/مسار الإصلاح الأساسي | إصلاح تلقائي |
| --- | --- | --- | --- | --- |
| `fs.state_dir.perms_world_writable` | حرج | يمكن لمستخدمين/عمليات أخرى تعديل حالة OpenClaw الكاملة | أذونات نظام الملفات على `~/.openclaw` | نعم |
| `fs.config.perms_writable` | حرج | يمكن للآخرين تغيير سياسة/تكوين المصادقة/الأداة | أذونات نظام الملفات على `~/.openclaw/openclaw.json` | نعم |
| `fs.config.perms_world_readable` | حرج | يمكن للتكوين كشف الرموز/الإعدادات | أذونات نظام الملفات على ملف التكوين | نعم |
| `gateway.bind_no_auth` | حرج | ربط بعيد بدون سر مشترك | `gateway.bind`, `gateway.auth.*` | لا |
| `gateway.loopback_no_auth` | حرج | loopback المعكوس الوكيل قد يصبح غير مصادق | `gateway.auth.*`, إعداد الوكيل | لا |
| `gateway.http.no_auth` | تحذير/حرج | واجهات برمجة تطبيقات HTTP للبوابة قابلة للوصول بـ `auth.mode="none"` | `gateway.auth.mode`, `gateway.http.endpoints.*` | لا |
| `gateway.tools_invoke_http.dangerous_allow` | تحذير/حرج | يعيد تمكين أدوات خطيرة عبر واجهة برمجة تطبيقات HTTP | `gateway.tools.allow` | لا |
| `gateway.nodes.allow_commands_dangerous` | تحذير/حرج | يمكّن أوامر عقدة عالية التأثير (كاميرا/شاشة/جهات اتصال/تقويم/SMS) | `gateway.nodes.allowCommands` | لا |
| `gateway.tailscale_funnel` | حرج | تعرض للإنترنت العام | `gateway.tailscale.mode` | لا |
| `gateway.control_ui.allowed_origins_required` | حرج | واجهة تحكم غير loopback بدون قائمة سماح أصل متصفح صريحة | `gateway.controlUi.allowedOrigins` | لا |
| `gateway.control_ui.host_header_origin_fallback` | تحذير/حرج | يمكّن استرجاع أصل رأس المضيف (تخفيض تقوية إعادة ربط DNS) | `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` | لا |
| `gateway.control_ui.insecure_auth` | تحذير | تبديل توافق المصادقة غير الآمنة ممكّن | `gateway.controlUi.allowInsecureAuth` | لا |
| `gateway.control_ui.device_auth_disabled` | حرج | يعطل فحص هوية الجهاز | `gateway.controlUi.dangerouslyDisableDeviceAuth` | لا |
| `gateway.real_ip_fallback_enabled` | تحذير/حرج | الثقة في استرجاع `X-Real-IP` يمكن أن يمكّن انتحال IP المصدر عبر سوء تكوين الوكيل | `gateway.allowRealIpFallback`, `gateway.trustedProxies` | لا |
| `discovery.mdns_full_mode` | تحذير/حرج | وضع mDNS الكامل يعلن عن بيانات تعريف `cliPath`/`sshPort` على الشبكة المحلية | `discovery.mdns.mode`, `gateway.bind` | لا |
| `config.insecure_or_dangerous_flags` | تحذير | أي أعلام تصحيح غير آمنة/خطيرة ممكّنة | مفاتيح متعددة (انظر تفاصيل النتيجة) | لا |
| `hooks.token_too_short` | تحذير | أسهل قوة غاشمة على مدخل hook | `hooks.token` | لا |
| `hooks.request_session_key_enabled` | تحذير/حرج | المتصل الخارجي يمكنه اختيار sessionKey | `hooks.allowRequestSessionKey` | لا |
| `hooks.request_session_key_prefixes_missing` | تحذير/حرج | لا حدود على أشكال مفتاح الجلسة الخارجية | `hooks.allowedSessionKeyPrefixes` | لا |
| `logging.redact_off` | تحذير | تسرب قيم حساسة إلى السجلات/الحالة | `logging.redactSensitive` | نعم |
| `sandbox.docker_config_mode_off` | تحذير | تكوين Docker للسندبة موجود لكن غير نشط | `agents.*.sandbox.mode` | لا |
| `sandbox.dangerous_network_mode` | حرج | شبكة Docker للسندبة تستخدم وضع `host` أو `container:*` للانضمام إلى النطاق | `agents.*.sandbox.docker.network` | لا |
| `tools.exec.host_sandbox_no_sandbox_defaults` | تحذير | `exec host=sandbox` يحل إلى exec مضيف عندما تكون السندبة معطلة | `tools.exec.host`, `agents.defaults.sandbox.mode` | لا |
| `tools.exec.host_sandbox_no_sandbox_agents` | تحذير | `exec host=sandbox` لكل وكيل يحل إلى exec مضيف عندما تكون السندبة معطلة | `agents.list[].tools.exec.host`, `agents.list[].sandbox.mode` | لا |
| `tools.exec.safe_bins_interpreter_unprofiled` | تحذير | bins مترجم/وقت تشغيل في `safeBins` بدون ملفات تعريف صريحة توسع خطر exec | `tools.exec.safeBins`, `tools.exec.safeBinProfiles`, `agents.list[].tools.exec.*` | لا |
| `skills.workspace.symlink_escape` | تحذير | مساحة العمل `skills/**/SKILL.md` تحل خارج جذر مساحة العمل (انحراف سلسلة رابط رمزي) | حالة نظام الملفات `skills/**` لمساحة العمل | لا |
| `security.exposure.open_groups_with_elevated` | حرج | مجموعات مفتوحة + أدوات مرتفعة تخلق مسارات حقن مطالبة عالية التأثير | `channels.*.groupPolicy`, `tools.elevated.*` | لا |
| `security.exposure.open_groups_with_runtime_or_fs` | حرج/تحذير | يمكن للمجموعات المفتوحة الوصول إلى أدوات ملف/أمر بدون حواجز سندبة/مساحة عمل | `channels.*.groupPolicy`, `tools.profile/deny`, `tools.fs.workspaceOnly`, `agents.*.sandbox.mode` | لا |
| `security.trust_model.multi_user_heuristic` | تحذير | التكوين يبدو متعدد المستخدمين بينما نموذج ثقة البوابة هو مساعد شخصي | تقسيم حدود الثقة، أو تقوية مستخدم مشترك (`sandbox.mode`، نطاق رفض/مساحة عمل الأداة) | لا |
| `tools.profile_minimal_overridden` | تحذير | تجاوزات الوكيل تتجاوز ملف التعريف الأدنى العام | `agents.list[].tools.profile` | لا |
| `plugins.tools_reachable_permissive_policy` | تحذير | أدوات الامتداد قابلة للوصول في سياقات متساهلة | `tools.profile` + السماح/الرفض للأداة | لا |
| `models.small_params` | حرج/معلومات | نماذج صغيرة + أسطح أداة غير آمنة تزيد خطر الحقن | اختيار النموذج + سياسة الأداة/السندبة | لا |

## واجهة التحكم عبر HTTP

تحتاج واجهة التحكم إلى **سياق آمن** (HTTPS أو localhost) لتوليد هوية الجهاز. `gateway.controlUi.allowInsecureAuth` **لا** تتجاوز فحوصات السياق الآمن، هوية الجهاز، أو فحوصات اقتران الجهاز. فضل HTTPS (Tailscale Serve) أو افتح الواجهة على `127.0.0.1`. فقط لسيناريوهات كسر الزجاج، `gateway.controlUi.dangerouslyDisableDeviceAuth` يعطل فحوصات هوية الجهاز تمامًا. هذا تخفيض أمني شديد؛ أبقيه معطلًا ما لم تكن تقوم بتصحيح أخطاء بنشاط ويمكنك التراجع بسرعة. `openclaw security audit` يحذر عندما يكون هذا الإعداد ممكّنًا.

## ملخص الأعلام غير الآمنة أو الخطيرة

يتضمن `openclaw security audit` `config.insecure_or_dangerous_flags` عندما تكون مفاتيح التبديل المعروفة غير الآمنة/الخطيرة ممكّنة. هذا الفحص يجمع حاليًا:

-   `gateway.controlUi.allowInsecureAuth=true`
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true`
-   `gateway.controlUi.dangerouslyDisableDeviceAuth=true`
-   `hooks.gmail.allowUnsafeExternalContent=true`
-   `hooks.mappings[].allowUnsafeExternalContent=true`
-   `tools.exec.applyPatch.workspaceOnly=false`

مفاتيح التكوين `dangerous*` / `dangerously*` الكاملة المعرفة في مخطط OpenClaw:

-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback`
-   `gateway.controlUi.dangerouslyDisableDeviceAuth`
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`
-   `channels.discord.dangerouslyAllowNameMatching`
-   `channels.discord.accounts..dangerouslyAllowNameMatching`
-   `channels.slack.dangerouslyAllowNameMatching`
-   `channels.slack.accounts..dangerouslyAllowNameMatching`
-   `channels.googlechat.dangerouslyAllowNameMatching`
-   `channels.googlechat.accounts..dangerouslyAllowNameMatching`
-   `channels.msteams.dangerouslyAllowNameMatching`
-   `channels.irc.dangerouslyAllowNameMatching` (قناة امتداد)
-   `channels.irc.accounts..dangerouslyAllowNameMatching` (قناة امتداد)
-   `channels.mattermost.dangerouslyAllowNameMatching` (قناة امتداد)
-   `channels.mattermost.accounts..dangerouslyAllowNameMatching` (قناة امتداد)
-   `agents.defaults.sandbox.docker.dangerouslyAllowReservedContainerTargets`
-   `agents.defaults.sandbox.docker.dangerouslyAllowExternalBindSources`
-   `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin`
-   `agents.list[].sandbox.docker.dangerouslyAllowReservedContainerTargets`
-   `agents.list[].sandbox.docker.dangerouslyAllowExternalBindSources`
-   `agents.list[].sandbox.docker.dangerouslyAllowContainerNamespaceJoin`

## تكوين الوكيل العكسي

إذا قمت بتشغيل البوابة خلف وكيل عكسي (nginx، Caddy، Traefik، إلخ.)، يجب عليك تكوين `gateway.trustedProxies` لاكتشاف IP العميل المناسب. عندما تكتشف البوابة رؤوس الوكيل من عنوان **ليس** في `trustedProxies`، فإنها **لن** تعامل الاتصالات كعملاء محليين. إذا كانت مصادقة البوابة معطلة، يتم رفض تلك الاتصالات. هذا يمنع تجاوز المصادقة حيث قد تظهر الاتصالات المعكوسة الوكيل وكأنها قادمة من localhost وتتلقى ثقة تلقائية.

```
gateway:
  trustedProxies:
    - "127.0.0.1" # إذا كان وكيلك يعمل على localhost
  # اختياري. افتراضي خطأ.
  # قم بتمكين فقط إذا كان وكيلك لا يمكنه توفير X-Forwarded-For.
  allowRealIpFallback: false
  auth:
    mode: password
    password: ${OPENCLAW_GATEWAY_PASSWORD}
```

عند تكوين `trustedProxies`، تستخدم البوابة `X-Forwarded-For` لتحديد IP العميل. يتم تجاهل `X-Real-IP` افتراضيًا ما لم يتم تعيين `gateway.allowRealIpFallback: true` صراحة. سلوك الوكيل العكسي الجيد (كتابة رؤوس التوجيه الواردة):

```bash
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Real-IP $remote_addr;
```

سلوك الوكيل العكسي السيئ (إلحاق/الحفاظ على رؤوس توجيه غير موثوقة):

```bash
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

## ملاحظات HSTS والأصل

-   بوابة OpenClaw محلية/loopback أولاً. إذا قمت بإنهاء TLS عند وكيل عكسي، قم بتعيين HSTS على مجال HTTPS المواجه للوكيل هناك.
-   إذا أنهت البوابة نفسها HTTPS، يمكنك تعيين `gateway.http.securityHeaders.strictTransportSecurity` لإصدار رأس HSTS من استجابات OpenClaw.
-   التوجيه التفصيلي للنشر موجود في [مصادقة الوكيل الموثوق](./trusted-proxy-auth.md#tls-termination-and-hsts).
-   لنشر واجهة تحكم غير loopback، `gateway.controlUi.allowedOrigins` مطلوب افتراضيًا.
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` يمكّن وضع استرجاع أصل رأس المضيف؛ عالجه كسياسة خطيرة مختارة من قبل المشغل.
-   عالج إعادة ربط DNS وسلوك رأس مضيف الوكيل كمخاوف تقوية للنشر؛ حافظ على `trustedProxies` ضيقة وتجنب تعريض البوابة مباشرة للإنترنت العام.

## سجلات الجلسة المحلية تعيش على القرص

يخزن OpenClaw نصوص الجلسات على القرص تحت `~/.openclaw/agents//sessions/*.jsonl`. هذا مطلوب لاستمرارية الجلسة و (اختياريًا) فهرسة ذاكرة الجلسة، لكنه يعني أيضًا **أي عملية/مستخدم لديه وصول نظام ملفات يمكنه قراءة تلك السجلات**. عالج وصول القرص كحد الثقة وقفل الأذونات على `~/.openclaw` (انظر قسم التدقيق أدناه). إذا كنت بحاجة إلى عزل أقوى بين الوكلاء، قم بتش