

  منصات المراسلة

  
# Microsoft Teams

> "اترك كل أمل، أيها الداخل هنا."

تم التحديث: 2026-01-21 الحالة: النصوص ومرفقات الرسائل المباشرة مدعومة؛ إرسال الملفات في قنوات/مجموعات الدردشة يتطلب `sharePointSiteId` + أذونات Graph (انظر [إرسال الملفات في دردشات المجموعات](#sending-files-in-group-chats)). يتم إرسال الاستطلاعات عبر بطاقات Adaptive Cards.

## الإضافة مطلوبة

يتم توزيع Microsoft Teams كإضافة وليست مضمنة في التثبيت الأساسي. **تغيير غير متوافق (2026.1.15):** تم نقل MS Teams خارج النواة. إذا كنت تستخدمه، يجب تثبيت الإضافة. السبب: يحافظ على خفة التثبيتات الأساسية ويسمح لتوابع MS Teams بالتحديث بشكل مستقل. التثبيت عبر CLI (npm registry):

```bash
openclaw plugins install @openclaw/msteams
```

التثبيت من نسخة محلية (عند التشغيل من مستودع git):

```bash
openclaw plugins install ./extensions/msteams
```

إذا اخترت Teams أثناء التكوين/البدء وتم اكتشاف نسخة git، سيعرض OpenClaw مسار التثبيت المحلي تلقائياً. التفاصيل: [الإضافات](../tools/plugin.md)

## الإعداد السريع (للمبتدئين)

1.  قم بتثبيت إضافة Microsoft Teams.
2.  أنشئ **Azure Bot** (معرف التطبيق + السر السري + معرف المستأجر).
3.  قم بتكوين OpenClaw باستخدام تلك الاعتمادات.
4.  اعرض `/api/messages` (المنفذ 3978 افتراضياً) عبر عنوان URL عام أو نفق.
5.  قم بتثبيت حزمة تطبيق Teams وابدأ تشغيل البوابة.

التهيئة الدنيا:

```json
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

ملاحظة: دردشات المجموعات محظورة افتراضياً (`channels.msteams.groupPolicy: "allowlist"`). للسماح بالردود الجماعية، عيّن `channels.msteams.groupAllowFrom` (أو استخدم `groupPolicy: "open"` للسماح لأي عضو، مع اشتراط الإشارة @).

## الأهداف

-   التحدث إلى OpenClaw عبر الرسائل المباشرة في Teams، أو دردشات المجموعات، أو القنوات.
-   الحفاظ على التوجيه حاسماً: الردود تعود دائماً إلى القناة التي وصلت منها.
-   الافتراضي إلى سلوك قناة آمن (مطلوب الإشارة @ ما لم يتم تكوين خلاف ذلك).

## كتابات التكوين

افتراضياً، يُسمح لـ Microsoft Teams بكتابة تحديثات التكوين التي يتم تشغيلها بواسطة `/config set|unset` (يتطلب `commands.config: true`). تعطيل باستخدام:

```json
{
  channels: { msteams: { configWrites: false } },
}
```

## التحكم في الوصول (الرسائل المباشرة + المجموعات)

**الوصول للرسائل المباشرة**

-   الافتراضي: `channels.msteams.dmPolicy = "pairing"`. يتم تجاهل المرسلين غير المعروفين حتى الموافقة عليهم.
-   يجب أن يستخدم `channels.msteams.allowFrom` معرفات كائنات AAD الثابتة.
-   أسماء UPN/عرض الأسماء قابلة للتغيير؛ المطابقة المباشرة معطلة افتراضياً ولا يتم تمكينها إلا مع `channels.msteams.dangerouslyAllowNameMatching: true`.
-   يمكن للمعالج السحري حل الأسماء إلى معرفات عبر Microsoft Graph عندما تسمح الاعتمادات.

**الوصول للمجموعات**

-   الافتراضي: `channels.msteams.groupPolicy = "allowlist"` (محظورة ما لم تضيف `groupAllowFrom`). استخدم `channels.defaults.groupPolicy` لتجاوز الافتراضي عندما لا يكون مضبوطاً.
-   يتحكم `channels.msteams.groupAllowFrom` في أي مرسلين يمكنهم التشغيل في دردشات/قنوات المجموعات (يتراجع إلى `channels.msteams.allowFrom`).
-   عيّن `groupPolicy: "open"` للسماح لأي عضو (لا يزال مشروطاً بالإشارة @ افتراضياً).
-   لمنع **أي قنوات**، عيّن `channels.msteams.groupPolicy: "disabled"`.

مثال:

```json
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**قائمة السماح للفرق والقنوات**

-   حدد نطاق الردود الجماعية/القنوية عن طريق سرد الفرق والقنوات تحت `channels.msteams.teams`.
-   يمكن أن تكون المفاتيح معرفات فرق أو أسماء؛ يمكن أن تكون مفاتيح القنوات معرفات محادثات أو أسماء.
-   عندما يكون `groupPolicy="allowlist"` وتوجد قائمة سماح للفرق، يتم قبول الفرق/القنوات المدرجة فقط (مشروطة بالإشارة @).
-   يقبل معالج التكوين إدخالات `Team/Channel` ويخزنها لك.
-   عند بدء التشغيل، يحل OpenClaw أسماء الفرق/القنوات وقوائم السماح للمستخدمين إلى معرفات (عندما تسمح أذونات Graph) ويسجل التعيين؛ يتم الاحتفاظ بالإدخالات غير المحلولة كما هي.

مثال:

```json
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            General: { requireMention: true },
          },
        },
      },
    },
  },
}
```

## كيفية عملها

1.  قم بتثبيت إضافة Microsoft Teams.
2.  أنشئ **Azure Bot** (معرف التطبيق + السر السري + معرف المستأجر).
3.  أنشئ **حزمة تطبيق Teams** التي تشير إلى الروبوت وتتضمن أذونات RSC أدناه.
4.  حمّل/ثبّت تطبيق Teams في فريق (أو نطاق شخصي للرسائل المباشرة).
5.  قم بتكوين `msteams` في `~/.openclaw/openclaw.json` (أو متغيرات البيئة) وابدأ تشغيل البوابة.
6.  تستمع البوابة لحركة مرور webhook الخاصة بـ Bot Framework على `/api/messages` افتراضياً.

## إعداد Azure Bot (المتطلبات الأساسية)

قبل تكوين OpenClaw، تحتاج إلى إنشاء مورد Azure Bot.

### الخطوة 1: إنشاء Azure Bot

1.  انتقل إلى [إنشاء Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2.  املأ علامة التبويب **الأساسيات**:
    
    | الحقل | القيمة |
    | --- | --- |
    | **مقبض الروبوت** | اسم روبوتك، مثلاً `openclaw-msteams` (يجب أن يكون فريداً) |
    | **الاشتراك** | اختر اشتراك Azure الخاص بك |
    | **مجموعة الموارد** | أنشئ مجموعة جديدة أو استخدم موجودة |
    | **مستوى التسعير** | **مجاني** للتطوير/الاختبار |
    | **نوع التطبيق** | **مستأجر واحد** (موصى به - انظر الملاحظة أدناه) |
    | **نوع الإنشاء** | **إنشاء معرف تطبيق Microsoft جديد** |
    

> **إشعار إهمال:** تم إهمال إنشاء روبوتات متعددة المستأجرين جديدة بعد 2025-07-31. استخدم **مستأجر واحد** للروبوتات الجديدة.

3.  انقر على **مراجعة + إنشاء** → **إنشاء** (انتظر ~1-2 دقيقة)

### الخطوة 2: الحصول على الاعتمادات

1.  انتقل إلى مورد Azure Bot الخاص بك → **التكوين**
2.  انسخ **معرف تطبيق Microsoft** → هذا هو `appId` الخاص بك
3.  انقر على **إدارة كلمة المرور** → انتقل إلى تسجيل التطبيق
4.  تحت **الشهادات والأسرار** → **سر عميل جديد** → انسخ **القيمة** → هذا هو `appPassword` الخاص بك
5.  انتقل إلى **نظرة عامة** → انسخ **معرف الدليل (المستأجر)** → هذا هو `tenantId` الخاص بك

### الخطوة 3: تكوين نقطة نهاية المراسلة

1.  في Azure Bot → **التكوين**
2.  عيّن **نقطة نهاية المراسلة** إلى عنوان URL الخاص بـ webhook:
    -   الإنتاج: `https://your-domain.com/api/messages`
    -   التطوير المحلي: استخدم نفقاً (انظر [التطوير المحلي](#local-development-tunneling) أدناه)

### الخطوة 4: تمكين قناة Teams

1.  في Azure Bot → **القنوات**
2.  انقر على **Microsoft Teams** → تكوين → حفظ
3.  اقبل شروط الخدمة

## التطوير المحلي (النفق)

لا يمكن لـ Teams الوصول إلى `localhost`. استخدم نفقاً للتطوير المحلي: **الخيار أ: ngrok**

```bash
ngrok http 3978
# انسخ عنوان URL الآمن، مثلاً https://abc123.ngrok.io
# عيّن نقطة نهاية المراسلة إلى: https://abc123.ngrok.io/api/messages
```

**الخيار ب: Tailscale Funnel**

```bash
tailscale funnel 3978
# استخدم عنوان URL الخاص بـ Tailscale funnel كنقطة نهاية المراسلة
```

## بوابة مطوري Teams (بديل)

بدلاً من إنشاء ملف ZIP للبيان يدوياً، يمكنك استخدام [بوابة مطوري Teams](https://dev.teams.microsoft.com/apps):

1.  انقر على **\+ تطبيق جديد**
2.  املأ المعلومات الأساسية (الاسم، الوصف، معلومات المطور)
3.  انتقل إلى **ميزات التطبيق** → **الروبوت**
4.  اختر **أدخل معرف روبوت يدوياً** والصق معرف تطبيق Azure Bot الخاص بك
5.  تحقق من النطاقات: **شخصي**، **فريق**، **دردشة جماعية**
6.  انقر على **توزيع** → **تنزيل حزمة التطبيق**
7.  في Teams: **التطبيقات** → **إدارة تطبيقاتك** → **تحميل تطبيق مخصص** → اختر ملف ZIP

غالباً ما يكون هذا أسهل من تحرير بيانات JSON يدوياً.

## اختبار الروبوت

**الخيار أ: دردشة الويب في Azure (تحقق من webhook أولاً)**

1.  في بوابة Azure → مورد Azure Bot الخاص بك → **اختبار في دردشة الويب**
2.  أرسل رسالة - يجب أن ترى رداً
3.  هذا يؤكد أن نقطة نهاية webhook تعمل قبل إعداد Teams

**الخيار ب: Teams (بعد تثبيت التطبيق)**

1.  قم بتثبيت تطبيق Teams (تحميل جانبي أو كتالوج المؤسسة)
2.  ابحث عن الروبوت في Teams وأرسل رسالة مباشرة
3.  تحقق من سجلات البوابة للأنشطة الواردة

## الإعداد (الحد الأدنى للنص فقط)

1.  **قم بتثبيت إضافة Microsoft Teams**
    -   من npm: `openclaw plugins install @openclaw/msteams`
    -   من نسخة محلية: `openclaw plugins install ./extensions/msteams`
2.  **تسجيل الروبوت**
    -   أنشئ Azure Bot (انظر أعلاه) ولاحظ:
        -   معرف التطبيق
        -   السر السري للعميل (كلمة مرور التطبيق)
        -   معرف المستأجر (مستأجر واحد)
3.  **بيان تطبيق Teams**
    -   أضف إدخال `bot` مع `botId = `.
    -   النطاقات: `personal`, `team`, `groupChat`.
    -   `supportsFiles: true` (مطلوب لمعالجة الملفات في النطاق الشخصي).
    -   أضف أذونات RSC (أدناه).
    -   أنشئ أيقونات: `outline.png` (32x32) و `color.png` (192x192).
    -   ضغط الملفات الثلاثة معاً: `manifest.json`, `outline.png`, `color.png`.
4.  **تكوين OpenClaw**
    
    انسخ
    
    ```json
    {
      "msteams": {
        "enabled": true,
        "appId": "<APP_ID>",
        "appPassword": "<APP_PASSWORD>",
        "tenantId": "<TENANT_ID>",
        "webhook": { "port": 3978, "path": "/api/messages" }
      }
    }
    ```
    
    يمكنك أيضاً استخدام متغيرات البيئة بدلاً من مفاتيح التكوين:
    -   `MSTEAMS_APP_ID`
    -   `MSTEAMS_APP_PASSWORD`
    -   `MSTEAMS_TENANT_ID`
5.  **نقطة نهاية الروبوت**
    -   عيّن نقطة نهاية مراسلة Azure Bot إلى:
        -   `https://:3978/api/messages` (أو المسار/المنفذ الذي اخترته).
6.  **تشغيل البوابة**
    -   تبدأ قناة Teams تلقائياً عند تثبيت الإضافة ووجود تكوين `msteams` مع الاعتمادات.

## سياق السجل

-   يتحكم `channels.msteams.historyLimit` في عدد رسائل القناة/المجموعة الأخيرة التي يتم تضمينها في المطالبة.
-   يتراجع إلى `messages.groupChat.historyLimit`. عيّن `0` لتعطيل (الافتراضي 50).
-   يمكن تحديد سجل الرسائل المباشرة بـ `channels.msteams.dmHistoryLimit` (دورات المستخدم). تجاوزات لكل مستخدم: `channels.msteams.dms["<user_id>"].historyLimit`.

## أذونات Teams RSC الحالية (البيان)

هذه هي **أذونات resourceSpecific الموجودة** في بيان تطبيق Teams الخاص بنا. تنطبق فقط داخل الفريق/الدردشة التي تم تثبيت التطبيق فيها. **للقنوات (نطاق الفريق):**

-   `ChannelMessage.Read.Group` (تطبيق) - استقبال جميع رسائل القناة دون @mention
-   `ChannelMessage.Send.Group` (تطبيق)
-   `Member.Read.Group` (تطبيق)
-   `Owner.Read.Group` (تطبيق)
-   `ChannelSettings.Read.Group` (تطبيق)
-   `TeamMember.Read.Group` (تطبيق)
-   `TeamSettings.Read.Group` (تطبيق)

**لدردشات المجموعات:**

-   `ChatMessage.Read.Chat` (تطبيق) - استقبال جميع رسائل دردشة المجموعة دون @mention

## مثال على بيان Teams (مختصر)

مثال أدنى صالح مع الحقول المطلوبة. استبدل المعرفات والعناوين.

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

### محاذير البيان (الحقول الإلزامية)

-   `bots[].botId` **يجب** أن يتطابق مع معرف تطبيق Azure Bot.
-   `webApplicationInfo.id` **يجب** أن يتطابق مع معرف تطبيق Azure Bot.
-   `bots[].scopes` يجب أن تتضمن الأسطح التي تخطط لاستخدامها (`personal`, `team`, `groupChat`).
-   `bots[].supportsFiles: true` مطلوب لمعالجة الملفات في النطاق الشخصي.
-   `authorization.permissions.resourceSpecific` يجب أن تتضمن قراءة/إرسال القناة إذا كنت تريد حركة مرور القناة.

### تحديث تطبيق موجود

لتحديث تطبيق Teams مثبت مسبقاً (مثلاً، لإضافة أذونات RSC):

1.  قم بتحديث `manifest.json` الخاص بك بالإعدادات الجديدة
2.  **زد قيمة حقل `version`** (مثلاً `1.0.0` → `1.1.0`)
3.  **أعد ضغط** البيان مع الأيقونات (`manifest.json`, `outline.png`, `color.png`)
4.  حمّل ملف ZIP الجديد:
    -   **الخيار أ (مركز إدارة Teams):** مركز إدارة Teams → تطبيقات Teams → إدارة التطبيقات → ابحث عن تطبيقك → تحميل نسخة جديدة
    -   **الخيار ب (تحميل جانبي):** في Teams → التطبيقات → إدارة تطبيقاتك → تحميل تطبيق مخصص
5.  **لقنوات الفريق:** أعد تثبيت التطبيق في كل فريق لتفعيل الأذونات الجديدة
6.  **أغلق Teams بالكامل وأعِد تشغيله** (ليس مجرد إغلاق النافذة) لمسح بيانات التطبيق المخزنة مؤقتاً

## الإمكانيات: RSC فقط مقابل Graph

### مع Teams RSC فقط (التطبيق مثبت، بدون أذونات Graph API)

يعمل:

-   قراءة محتوى **النص** لرسائل القناة.
-   إرسال محتوى **النص** لرسائل القناة.
-   استقبال **مرفقات الملفات الشخصية (رسائل مباشرة)**.

لا يعمل:

-   محتويات **الصور أو الملفات** في القناة/المجموعة (الحمولة تتضمن فقط HTML stub).
-   تنزيل المرفقات المخزنة في SharePoint/OneDrive.
-   قراءة سجل الرسائل (أبعد من حدث webhook المباشر).

### مع Teams RSC + أذونات تطبيق Microsoft Graph

يضيف:

-   تنزيل المحتويات المستضافة (الصور الملصقة في الرسائل).
-   تنزيل مرفقات الملفات المخزنة في SharePoint/OneDrive.
-   قراءة سجل رسائل القناة/الدردشة عبر Graph.

### RSC مقابل Graph API

| الإمكانية | أذونات RSC | Graph API |
| --- | --- | --- |
| **الرسائل في الوقت الفعلي** | نعم (عبر webhook) | لا (استطلاع فقط) |
| **الرسائل التاريخية** | لا | نعم (يمكن الاستعلام عن السجل) |
| **تعقيد الإعداد** | بيان التطبيق فقط | يتطلب موافقة المسؤول + سير عمل الرمز المميز |
| **يعمل دون اتصال** | لا (يجب أن يكون قيد التشغيل) | نعم (استعلام في أي وقت) |

**الخلاصة:** RSC للاستماع في الوقت الفعلي؛ Graph API للوصول التاريخي. لتعويض الرسائل الفائتة أثناء عدم الاتصال، تحتاج إلى Graph API مع `ChannelMessage.Read.All` (يتطلب موافقة المسؤول).

## الوسائط والبيانات التاريخية الممكّنة بـ Graph (مطلوبة للقنوات)

إذا كنت تحتاج إلى صور/ملفات في **القنوات** أو تريد جلب **سجل الرسائل**، يجب تمكين أذونات Microsoft Graph ومنح موافقة المسؤول.

1.  في Entra ID (Azure AD) **تسجيل التطبيق**، أضف أذونات تطبيق **Microsoft Graph**:
    -   `ChannelMessage.Read.All` (مرفقات القناة + السجل)
    -   `Chat.Read.All` أو `ChatMessage.Read.All` (دردشات المجموعات)
2.  **امنح موافقة المسؤول** للمستأجر.
3.  زد **رقم إصدار** بيان تطبيق Teams، وأعد التحميل، و**أعد تثبيت التطبيق في Teams**.
4.  **أغلق Teams بالكامل وأعِد تشغيله** لمسح بيانات التطبيق المخزنة مؤقتاً.

**إذن إضافي للإشارات @ للمستخدمين:** تعمل الإشارات @ للمستخدمين بشكل افتراضي للمستخدمين في المحادثة. ومع ذلك، إذا كنت تريد البحث الديناميكي والإشارة @ لمستخدمين **ليسوا في المحادثة الحالية**، أضف إذن `User.Read.All` (تطبيق) وامنح موافقة المسؤول.

## القيود المعروفة

### مهلات Webhook

يقوم Teams بتسليم الرسائل عبر webhook HTTP. إذا استغرق المعالجة وقتاً طويلاً (مثلاً، استجابات LLM بطيئة)، قد ترى:

-   مهلات البوابة
-   Teams يعيد محاولة الرسالة (مسبباً تكرارات)
-   ردود مفقودة

يتعامل OpenClaw مع هذا بالرد بسرعة وإرسال الردود بشكل استباقي، ولكن الردود البطيئة جداً قد تسبب مشاكل.

### التنسيق

تنسيق Markdown في Teams أكثر محدودية من Slack أو Discord:

-   التنسيق الأساسي يعمل: **عريض**، *مائل*، `كود`، روابط
-   قد لا يتم عرض تنسيق Markdown المعقد (جداول، قوائم متداخلة) بشكل صحيح
-   بطاقات Adaptive Cards مدعومة للاستطلاعات وإرسال البطاقات التعسفي (انظر أدناه)

## التكوين

الإعدادات الرئيسية (انظر `/gateway/configuration` لأنماط القنوات المشتركة):

-   `channels.msteams.enabled`: تمكين/تعطيل القناة.
-   `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId`: اعتمادات الروبوت.
-   `channels.msteams.webhook.port` (افتراضي `3978`)
-   `channels.msteams.webhook.path` (افتراضي `/api/messages`)
-   `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled` (افتراضي: pairing)
-   `channels.msteams.allowFrom`: قائمة السماح للرسائل المباشرة (يوصى بمعرفات كائنات AAD). يحلل المعالج السحري الأسماء إلى معرفات أثناء الإعداد عندما يكون الوصول إلى Graph متاحاً.
-   `channels.msteams.dangerouslyAllowNameMatching`: تبديل كسر الزجاج لإعادة تمكين مطابقة UPN/اسم العرض القابلة للتغيير.
-   `channels.msteams.textChunkLimit`: حجم جزء النص الصادر.
-   `channels.msteams.chunkMode`: `length` (افتراضي) أو `newline` للتقسيم على أسطر فارغة (حدود الفقرات) قبل التجزئة حسب الطول.
-   `channels.msteams.mediaAllowHosts`: قائمة السماح لمضيفات المرفقات الواردة (تتضمن افتراضياً نطاقات Microsoft/Teams).
-   `channels.msteams.mediaAuthAllowHosts`: قائمة السماح لإرفاق رؤوس Authorization عند إعادة محاولات الوسائط (تتضمن افتراضياً مضيفات Graph + Bot Framework).
-   `channels.msteams.requireMention`: اشتراط @mention في القنوات/المجموعات (افتراضي صحيح).
-   `channels.msteams.replyStyle`: `thread | top-level` (انظر [نمط الرد](#reply-style-threads-vs-posts)).
-   `channels.msteams.teams..replyStyle`: تجاوز لكل فريق.
-   `channels.msteams.teams..requireMention`: تجاوز لكل فريق.
-   `channels.msteams.teams..tools`: تجاوزات سياسة الأدوات الافتراضية لكل فريق (`allow`/`deny`/`alsoAllow`) تُستخدم عندما يكون تجاوز القناة مفقوداً.
-   `channels.msteams.teams..toolsBySender`: تجاوزات سياسة الأدوات لكل مرسل افتراضية لكل فريق (يدعم `"*"` كحرف بدل).
-   `channels.msteams.teams..channels..replyStyle`: تجاوز لكل قناة.
-   `channels.msteams.teams..channels..requireMention`: تجاوز لكل قناة.
-   `channels.msteams.teams..channels..tools`: تجاوزات سياسة الأدوات لكل قناة (`allow`/`deny`/`alsoAllow`).
-   `channels.msteams.teams..channels..toolsBySender`: تجاوزات سياسة الأدوات لكل مرسل لكل قناة (يدعم `"*"` كحرف بدل).
-   يجب أن تستخدم مفاتيح `toolsBySender` بادئات صريحة: `id:`, `e164:`, `username:`, `name:` (المفاتيح غير المسبوقة القديمة لا تزال تُرسم إلى `id:` فقط).
-   `channels.msteams.sharePointSiteId`: معرف موقع SharePoint لتحميل الملفات في دردشات/قنوات المجموعات (انظر [إرسال الملفات في دردشات المجموعات](#sending-files-in-group-chats)).

## التوجيه والجلسات

-   تتبع مفاتيح الجلسة تنسيق الوكيل القياسي (انظر [/concepts/session](../concepts/session.md)):
    -   تشارك الرسائل المباشرة الجلسة الرئيسية (`agent::`).
    -   تستخدم رسائل القناة/المجموعة معرف المحادثة:
        -   `agent::msteams:channel:`
        -   `agent::msteams:group:`

## نمط الرد: سلاسل النقاش مقابل المنشورات

أدخلت Teams مؤخراً نمطين لواجهة المستخدم للقنوات فوق نفس نموذج البيانات الأساسي:

| النمط | الوصف | `replyStyle` الموصى به |
| --- | --- | --- |
| **المنشورات** (كلاسيكي) | تظهر الرسائل كبطاقات مع ردود متسلسلة تحتها | `thread` (افتراضي) |
| **سلاسل النقاش** (مشابه لـ Slack) | تتدفق الرسائل بشكل خطي، أشبه بـ Slack | `top-level` |

**المشكلة:** لا يعرض واجهة برمجة تطبيقات Teams أي نمط واجهة مستخدم تستخدمه القناة. إذا استخدمت `replyStyle` خاطئاً:

-   `thread` في قناة بنمط سلاسل النقاش → تظهر الردود متداخلة بشكل غريب
-   `top-level` في قناة بنمط المنشورات → تظهر الردود كمنشورات منفصلة على المستوى العلوي بدلاً من داخل السلسلة

**الحل:** قم بتكوين `replyStyle` لكل قناة بناءً على كيفية إعداد القناة:

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

## المرفقات والصور

**القيود الحالية:**

-   **الرسائل المباشرة:** تعمل الصور ومرفقات الملفات عبر واجهات برمجة تطبيقات ملفات روبوت Teams.
-   **القنوات/المجموعات:** تعيش المرفقات في تخزين M365 (SharePoint/OneDrive). تحتوي حمولة webhook فقط على HTML stub، وليس بايتات الملف الفعلية. **مطلوب أذونات Graph API** لتنزيل مرفقات القناة.

بدون أذونات Graph، سيتم استقبال رسائل القناة التي تحتوي على صور كنص فقط (محتوى الصورة غير قابل للوصول للروبوت). افتراضياً، يقوم OpenClaw بتنزيل الوسائط فقط من أسماء مضيفات Microsoft/Teams. تجاوز باستخدام `channels.msteams.mediaAllowHosts` (استخدم `["*"]` للسماح بأي مضيف). يتم إرفاق رؤوس Authorization فقط للمضيفات في `channels.msteams.mediaAuthAllowHosts` (تتضمن افتراضياً مضيفات Graph + Bot Framework). حافظ على هذه القائمة صارمة (تجنب اللواحق متعددة المستأجرين).

## إرسال الملفات في دردشات المجموعات

يمكن للروبوتات إرسال الملفات في الرسائل المباشرة باستخدام سير عمل FileConsentCard (مدمج). ومع ذلك، **يتطلب إرسال الملفات في دردشات/قنوات المجموعات** إعداداً إضافياً:

| السياق | كيفية إرسال الملفات | الإعداد المطلوب |
| --- | --- | --- |
| **الرسائل المباشرة** | FileConsentCard → يقبل المستخدم → يرفع الروبوت | يعمل افتراضياً |
| **دردشات/قنوات المجموعات** | التحميل إلى SharePoint → رابط مشاركة | يتطلب `sharePointSiteId` + أذونات Graph |
| **الصور (أي سياق)** | مضمنة مشفرة بـ Base64 | يعمل افتراضياً |

### لماذا تحتاج دردشات المجموعات إلى SharePoint

ليس للروبوتات محرك OneDrive شخصي (نقطة نهاية Graph API `/me/drive` لا تعمل لهويات التطبيق). لإرسال الملفات في دردشات/قنوات المجموعات، يرفع الروبوت إلى **موقع SharePoint** وينشئ رابط مشاركة.

### الإعداد

1.  **أضف أذونات Graph API** في Entra ID (Azure AD) → تسجيل التطبيق:
    -   `Sites.ReadWrite.All` (تطبيق) - رفع الملفات إلى SharePoint
    -   `Chat.Read.All` (تطبيق) - اختياري، يمكّن روابط المشاركة لكل مستخدم
2.  **امنح