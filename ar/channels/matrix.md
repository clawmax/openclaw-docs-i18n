

  منصات المراسلة

  
# المصفوفة (Matrix)

Matrix هو بروتوكول مراسلة مفتوح ولا مركزي. يتصل OpenClaw كـ **مستخدم** Matrix على أي خادم منزلي (homeserver)، لذا تحتاج إلى حساب Matrix للبوت. بمجرد تسجيل الدخول، يمكنك مراسلة البوت مباشرة أو دعوته إلى الغرف ("المجموعات" في Matrix). يُعد Beeper خيار عميل صالح أيضًا، ولكنه يتطلب تمكين التشفير من طرف إلى طرف (E2EE). الحالة: مدعوم عبر الإضافة (@vector-im/matrix-bot-sdk). الرسائل المباشرة، الغرف، السلاسل النصية (threads)، الوسائط، التفاعلات (reactions)، الاستطلاعات (إرسال + بدء الاستطلاع كنص)، الموقع الجغرافي، والتشفير من طرف إلى طرف (E2EE) (بدعم التشفير).

## الإضافة مطلوبة

يتم توزيع Matrix كإضافة وليست مضمنة في التثبيت الأساسي. قم بالتثبيت عبر سطر الأوامر (CLI) (من سجل npm):

```bash
openclaw plugins install @openclaw/matrix
```

التثبيت من نسخة محلية (عند التشغيل من مستودع git):

```bash
openclaw plugins install ./extensions/matrix
```

إذا اخترت Matrix أثناء التهيئة/البدء وتم اكتشاف نسخة git محلية، فسيقترح OpenClaw مسار التثبيت المحلي تلقائيًا. التفاصيل: [الإضافات](../tools/plugin.md)

## الإعداد

1.  قم بتثبيت إضافة Matrix:
    -   من npm: `openclaw plugins install @openclaw/matrix`
    -   من نسخة محلية: `openclaw plugins install ./extensions/matrix`
2.  أنشئ حساب Matrix على خادم منزلي (homeserver):
    -   تصفح خيارات الاستضافة على [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
    -   أو استضفه بنفسك.
3.  احصل على رمز وصول (access token) لحساب البوت:
    
    -   استخدم واجهة برمجة تطبيقات تسجيل دخول Matrix مع `curl` على خادمك المنزلي:
    
    نسخ
    
    ```bash
    curl --request POST \
      --url https://matrix.example.org/_matrix/client/v3/login \
      --header 'Content-Type: application/json' \
      --data '{
      "type": "m.login.password",
      "identifier": {
        "type": "m.id.user",
        "user": "your-user-name"
      },
      "password": "your-password"
    }'
    ```
    
    -   استبدل `matrix.example.org` بعنوان URL لخادمك المنزلي.
    -   أو قم بتعيين `channels.matrix.userId` + `channels.matrix.password`: سيتصل OpenClaw بنفس نقطة نهاية تسجيل الدخول، ويخزن رمز الوصول في `~/.openclaw/credentials/matrix/credentials.json`، ويعيد استخدامه عند البدء التالي.
4.  قم بتكوين بيانات الاعتماد:
    -   متغيرات البيئة (Env): `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (أو `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
    -   أو التكوين: `channels.matrix.*`
    -   إذا تم تعيين كليهما، فإن التكوين له الأولوية.
    -   مع رمز الوصول: يتم جلب معرف المستخدم تلقائيًا عبر `/whoami`.
    -   عند التعيين، يجب أن يكون `channels.matrix.userId` هو معرف Matrix الكامل (مثال: `@bot:example.org`).
5.  أعد تشغيل البوابة (gateway) (أو أكمل عملية البدء).
6.  ابدأ محادثة مباشرة (DM) مع البوت أو ادعُه إلى غرفة من أي عميل Matrix (Element، Beeper، إلخ؛ انظر [https://matrix.org/ecosystem/clients/](https://matrix.org/ecosystem/clients/)). يتطلب Beeper تشفير E2EE، لذا قم بتعيين `channels.matrix.encryption: true` وتحقق من الجهاز.

الحد الأدنى للتكوين (رمز الوصول، يتم جلب معرف المستخدم تلقائيًا):

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" },
    },
  },
}
```

تكوين E2EE (تمكين التشفير من طرف إلى طرف):

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" },
    },
  },
}
```

## التشفير (E2EE)

التشفير من طرف إلى طرف **مدعوم** عبر Rust crypto SDK. قم بتمكينه باستخدام `channels.matrix.encryption: true`:

-   إذا تم تحميل وحدة التشفير، فسيتم فك تشفير الغرف المشفرة تلقائيًا.
-   يتم تشفير الوسائط الصادرة عند الإرسال إلى الغرف المشفرة.
-   عند الاتصال الأول، يطلب OpenClaw التحقق من الجهاز من جلساتك الأخرى.
-   تحقق من الجهاز في عميل Matrix آخر (Element، إلخ.) لتمكين مشاركة المفاتيح.
-   إذا تعذر تحميل وحدة التشفير، يتم تعطيل E2EE ولن يتم فك تشفير الغرف المشفرة؛ يسجل OpenClaw تحذيرًا.
-   إذا رأيت أخطاء وحدة تشفير مفقودة (على سبيل المثال، `@matrix-org/matrix-sdk-crypto-nodejs-*`)، اسمح بتنفيذ نصوص البناء لـ `@matrix-org/matrix-sdk-crypto-nodejs` وقم بتشغيل `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` أو احصل على الملف الثنائي باستخدام `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`.

يتم تخزين حالة التشفير لكل حساب + رمز وصول في `~/.openclaw/matrix/accounts//__/<token-hash>/crypto/` (قاعدة بيانات SQLite). تعيش حالة المزامنة (Sync state) بجانبها في `bot-storage.json`. إذا تغير رمز الوصول (الجهاز)، يتم إنشاء مخزن جديد ويجب إعادة التحقق من البوت للغرف المشفرة. **التحقق من الجهاز:** عند تمكين E2EE، سيطالب البوت بالتحقق من جلساتك الأخرى عند بدء التشغيل. افتح Element (أو عميل آخر) ووافق على طلب التحقق لإنشاء الثقة. بمجرد التحقق، يمكن للبوت فك تشفير الرسائل في الغرف المشفرة.

## حسابات متعددة

دعم حسابات متعددة: استخدم `channels.matrix.accounts` مع بيانات اعتماد لكل حساب واسم اختياري `name`. راجع [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) للنمط المشترك. يعمل كل حساب كمستخدم Matrix منفصل على أي خادم منزلي. يرث تكوين كل حساب الإعدادات من المستوى الأعلى `channels.matrix` ويمكنه تجاوز أي خيار (سياسة الرسائل المباشرة، المجموعات، التشفير، إلخ.).

```json
{
  channels: {
    matrix: {
      enabled: true,
      dm: { policy: "pairing" },
      accounts: {
        assistant: {
          name: "المساعد الرئيسي",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_assistant_***",
          encryption: true,
        },
        alerts: {
          name: "بوت التنبيهات",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_alerts_***",
          dm: { policy: "allowlist", allowFrom: ["@admin:example.org"] },
        },
      },
    },
  },
}
```

ملاحظات:

-   يتم بدء تشغيل الحسابات بشكل تسلسلي لتجنب ظروف السباق (race conditions) مع استيراد الوحدات المتزامن.
-   تنطبق متغيرات البيئة (`MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`، إلخ.) فقط على الحساب **الافتراضي**.
-   تنطبق إعدادات القناة الأساسية (سياسة الرسائل المباشرة، سياسة المجموعات، التحكم في الإشارات، إلخ.) على جميع الحسابات ما لم يتم تجاوزها لكل حساب.
-   استخدم `bindings[].match.accountId` لتوجيه كل حساب إلى وكيل (agent) مختلف.
-   يتم تخزين حالة التشفير لكل حساب + رمز وصول (مخازن مفاتيح منفصلة لكل حساب).

## نموذج التوجيه

-   تذهب الردود دائمًا إلى Matrix.
-   تشترك الرسائل المباشرة (DMs) في الجلسة الرئيسية للوكيل؛ تُرسم الغرف إلى جلسات المجموعة.

## التحكم في الوصول (الرسائل المباشرة)

-   الافتراضي: `channels.matrix.dm.policy = "pairing"`. يحصل المرسلون غير المعروفين على رمز إقران (pairing code).
-   الموافقة عبر:
    -   `openclaw pairing list matrix`
    -   `openclaw pairing approve matrix `
-   الرسائل المباشرة العامة: `channels.matrix.dm.policy="open"` بالإضافة إلى `channels.matrix.dm.allowFrom=["*"]`.
-   يقبل `channels.matrix.dm.allowFrom` معرفات مستخدم Matrix كاملة (مثال: `@user:server`). يحلل المعالج (wizard) أسماء العرض إلى معرفات المستخدم عندما يجد بحث الدليل تطابقًا واحدًا دقيقًا.
-   لا تستخدم أسماء العرض أو الأجزاء المحلية المجردة (مثال: `"Alice"` أو `"alice"`). فهي غامضة ويتم تجاهلها لمطابقة القائمة المسموح بها (allowlist). استخدم معرفات `@user:server` الكاملة.

## الغرف (المجموعات)

-   الافتراضي: `channels.matrix.groupPolicy = "allowlist"` (مقيد بالإشارة). استخدم `channels.defaults.groupPolicy` لتجاوز الافتراضي عندما لا يكون مضبوطًا.
-   ملاحظة وقت التشغيل: إذا كان `channels.matrix` مفقودًا تمامًا، فإن وقت التشغيل يتراجع إلى `groupPolicy="allowlist"` للتحقق من الغرف (حتى إذا تم تعيين `channels.defaults.groupPolicy`).
-   أضف الغرف إلى القائمة المسموح بها باستخدام `channels.matrix.groups` (معرفات الغرف أو الأسماء المستعارة؛ يتم تحويل الأسماء إلى معرفات عندما يجد بحث الدليل تطابقًا واحدًا دقيقًا):

```json
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
      groupAllowFrom: ["@owner:example.org"],
    },
  },
}
```

-   `requireMention: false` يمكّن الرد التلقائي في تلك الغرفة.
-   يمكن لـ `groups."*"` تعيين الإعدادات الافتراضية للتحكم في الإشارات عبر الغرف.
-   `groupAllowFrom` يحدد أي المرسلين يمكنهم تشغيل البوت في الغرف (معرفات مستخدم Matrix كاملة).
-   يمكن لقوائم `users` المسموح بها لكل غرفة تقييد المرسلين بشكل أكبر داخل غرفة محددة (استخدم معرفات مستخدم Matrix كاملة).
-   يطالب معالج التهيئة بقوائم الغرف المسموح بها (معرفات الغرف، الأسماء المستعارة، أو الأسماء) ويحلل الأسماء فقط عند وجود تطابق فريد ودقيق.
-   عند بدء التشغيل، يحلل OpenClaw أسماء الغرف/المستخدمين في القوائم المسموح بها إلى معرفات ويسجل التعيين؛ يتم تجاهل الإدخالات غير المحلولة لمطابقة القائمة المسموح بها.
-   يتم الانضمام إلى الدعوات تلقائيًا افتراضيًا؛ تحكم بـ `channels.matrix.autoJoin` و `channels.matrix.autoJoinAllowlist`.
-   للسماح **بلا غرف**، عيّن `channels.matrix.groupPolicy: "disabled"` (أو احتفظ بقائمة مسموح بها فارغة).
-   المفتاح القديم: `channels.matrix.rooms` (بنفس شكل `groups`).

## السلاسل النصية (Threads)

-   الردود في سلاسل نصية مدعومة.
-   يتحكم `channels.matrix.threadReplies` في ما إذا كانت الردود تبقى في السلاسل النصية:
    -   `off`, `inbound` (الافتراضي), `always`
-   يتحكم `channels.matrix.replyToMode` في بيانات وصفية الرد (reply-to metadata) عند عدم الرد في سلسلة نصية:
    -   `off` (الافتراضي), `first`, `all`

## الإمكانيات

| الميزة | الحالة |
| --- | --- |
| الرسائل المباشرة | ✅ مدعوم |
| الغرف | ✅ مدعوم |
| السلاسل النصية | ✅ مدعوم |
| الوسائط | ✅ مدعوم |
| التشفير من طرف إلى طرف (E2EE) | ✅ مدعوم (مطلوب وحدة تشفير) |
| التفاعلات (Reactions) | ✅ مدعوم (إرسال/قراءة عبر الأدوات) |
| الاستطلاعات | ✅ الإرسال مدعوم؛ يتم تحويل بدايات الاستطلاعات الواردة إلى نص (يتم تجاهل الردود/النهايات) |
| الموقع الجغرافي | ✅ مدعوم (رابط جغرافي URI؛ يتم تجاهل الارتفاع) |
| الأوامر الأصلية | ✅ مدعوم |

## استكشاف الأخطاء وإصلاحها

قم بتشغيل هذا السلم أولاً:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

ثم تأكد من حالة إقران الرسائل المباشرة إذا لزم الأمر:

```bash
openclaw pairing list matrix
```

الأعطال الشائعة:

-   تم تسجيل الدخول ولكن يتم تجاهل رسائل الغرفة: الغرفة محظورة بواسطة `groupPolicy` أو قائمة الغرف المسموح بها.
-   يتم تجاهل الرسائل المباشرة: المرسل في انتظار الموافقة عندما يكون `channels.matrix.dm.policy="pairing"`.
-   فشل الغرف المشفرة: عدم تطابق دعم التشفير أو إعدادات التشفير.

لتدفق التقييم: [/channels/troubleshooting](./troubleshooting.md).

## مرجع التكوين (Matrix)

التكوين الكامل: [التكوين](../gateway/configuration.md) خيارات المزود (Provider):

-   `channels.matrix.enabled`: تمكين/تعطيل بدء تشغيل القناة.
-   `channels.matrix.homeserver`: عنوان URL لخادم المنزل (homeserver).
-   `channels.matrix.userId`: معرف مستخدم Matrix (اختياري مع رمز الوصول).
-   `channels.matrix.accessToken`: رمز الوصول.
-   `channels.matrix.password`: كلمة المرور لتسجيل الدخول (يتم تخزين الرمز).
-   `channels.matrix.deviceName`: اسم عرض الجهاز.
-   `channels.matrix.encryption`: تمكين E2EE (الافتراضي: false).
-   `channels.matrix.initialSyncLimit`: حد المزامنة الأولية.
-   `channels.matrix.threadReplies`: `off | inbound | always` (الافتراضي: inbound).
-   `channels.matrix.textChunkLimit`: حجم تجزئة النص الصادر (بالأحرف).
-   `channels.matrix.chunkMode`: `length` (الافتراضي) أو `newline` للتقسيم على الأسطر الفارغة (حدود الفقرات) قبل تجزئة الطول.
-   `channels.matrix.dm.policy`: `pairing | allowlist | open | disabled` (الافتراضي: pairing).
-   `channels.matrix.dm.allowFrom`: قائمة المسموح بهم للرسائل المباشرة (معرفات مستخدم Matrix كاملة). يتطلب `open` تعيين `"*"`. يحلل المعالج الأسماء إلى معرفات عندما يكون ذلك ممكنًا.
-   `channels.matrix.groupPolicy`: `allowlist | open | disabled` (الافتراضي: allowlist).
-   `channels.matrix.groupAllowFrom`: المرسلون المسموح بهم لرسائل المجموعة (معرفات مستخدم Matrix كاملة).
-   `channels.matrix.allowlistOnly`: فرض قواعد القائمة المسموح بها للرسائل المباشرة + الغرف.
-   `channels.matrix.groups`: قائمة المجموعات المسموح بها + خريطة إعدادات كل غرفة.
-   `channels.matrix.rooms`: قائمة المجموعات المسموح بها/التكوين القديم.
-   `channels.matrix.replyToMode`: وضع الرد (reply-to mode) للسلاسل النصية/العلامات.
-   `channels.matrix.mediaMaxMb`: الحد الأقصى للوسائط الواردة/الصادرة (ميغابايت).
-   `channels.matrix.autoJoin`: معالجة الدعوات (`always | allowlist | off`, الافتراضي: always).
-   `channels.matrix.autoJoinAllowlist`: معرفات الغرف/الأسماء المستعارة المسموح بها للانضمام التلقائي.
-   `channels.matrix.accounts`: تكوين الحسابات المتعددة مفتاحًا بمعرف الحساب (كل حساب يرث إعدادات المستوى الأعلى).
-   `channels.matrix.actions`: التحكم في الأدوات لكل إجراء (تفاعلات/رسائل/تثبيتات/معلومات الأعضاء/معلومات القناة).

[LINE](./line.md)[Mattermost](./mattermost.md)