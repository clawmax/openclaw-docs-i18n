

  منصات المراسلة

  
# Signal

الحالة: تكامل CLI خارجي. تتحدث البوابة إلى `signal-cli` عبر HTTP JSON-RPC + SSE.

## المتطلبات الأساسية

-   OpenClaw مثبت على خادمك (تم اختبار تدفق Linux أدناه على Ubuntu 24).
-   `signal-cli` متاح على المضيف الذي يعمل عليه البوابة.
-   رقم هاتف يمكنه استقبال رسالة نصية تحقق واحدة (لمسار التسجيل عبر SMS).
-   وصول متصفح لـ captcha الخاص بـ Signal (`signalcaptchas.org`) أثناء التسجيل.

## الإعداد السريع (للمبتدئين)

1.  استخدم **رقم Signal منفصل** للبوت (موصى به).
2.  قم بتثبيت `signal-cli` (Java مطلوب إذا كنت تستخدم بناء JVM).
3.  اختر مسار إعداد واحد:
    -   **المسار أ (رابط QR):** `signal-cli link -n "OpenClaw"` ثم امسح QR ضوئيًا باستخدام Signal.
    -   **المسار ب (التسجيل عبر SMS):** سجل رقمًا مخصصًا مع captcha + التحقق عبر SMS.
4.  قم بتكوين OpenClaw وأعد تشغيل البوابة.
5.  أرسل أول رسالة مباشرة ووافق على الإقران (`openclaw pairing approve signal `).

التهيئة الدنيا:

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

مرجع الحقول:

| الحقل | الوصف |
| --- | --- |
| `account` | رقم هاتف البوت بتنسيق E.164 (`+15551234567`) |
| `cliPath` | المسار إلى `signal-cli` (`signal-cli` إذا كان على `PATH`) |
| `dmPolicy` | سياسة الوصول للرسائل المباشرة (`pairing` موصى به) |
| `allowFrom` | أرقام الهواتف أو قيم `uuid:` المسموح لها بإرسال رسائل مباشرة |

## ما هو

-   قناة Signal عبر `signal-cli` (ليست libsignal مضمنة).
-   التوجيه الحتمي: الردود تعود دائمًا إلى Signal.
-   تشارك الرسائل المباشرة الجلسة الرئيسية للوكيل؛ المجموعات معزولة (`agent::signal:group:`).

## كتابات التكوين

افتراضيًا، يُسمح لـ Signal بكتابة تحديثات التكوين التي يتم تشغيلها بواسطة `/config set|unset` (يتطلب `commands.config: true`). قم بتعطيله باستخدام:

```json
{
  channels: { signal: { configWrites: false } },
}
```

## نموذج الرقم (مهم)

-   تتصل البوابة بـ **جهاز Signal** (حساب `signal-cli`).
-   إذا قمت بتشغيل البوت على **حساب Signal الشخصي الخاص بك**، فسيتجاهل رسائلك الخاصة (حماية من الحلقات).
-   لـ "أنا أرسل نصًا للبوت ويقوم بالرد"، استخدم **رقم بوت منفصل**.

## مسار الإعداد أ: ربط حساب Signal موجود (QR)

1.  قم بتثبيت `signal-cli` (بناء JVM أو أصلي).
2.  ربط حساب بوت:
    -   `signal-cli link -n "OpenClaw"` ثم امسح QR ضوئيًا في Signal.
3.  قم بتكوين Signal وابدأ تشغيل البوابة.

مثال:

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

دعم الحسابات المتعددة: استخدم `channels.signal.accounts` مع تكوين لكل حساب واسم اختياري. راجع [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) للنمط المشترك.

## مسار الإعداد ب: تسجيل رقم بوت مخصص (SMS، Linux)

استخدم هذا عندما تريد رقم بوت مخصص بدلاً من ربط حساب تطبيق Signal موجود.

1.  احصل على رقم يمكنه استقبال رسائل SMS (أو التحقق الصوتي لأرقام الهواتف الأرضية).
    -   استخدم رقم بوت مخصص لتجنب تعارضات الحساب/الجلسة.
2.  قم بتثبيت `signal-cli` على مضيف البوابة:

```
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

إذا كنت تستخدم بناء JVM (`signal-cli-${VERSION}.tar.gz`)، قم بتثبيت JRE 25+ أولاً. حافظ على تحديث `signal-cli`؛ تشير المصادر العلوية إلى أن الإصدارات القديمة قد تتوقف عن العمل مع تغيير واجهات برمجة تطبيقات خادم Signal.

3.  سجل و تحقق من الرقم:

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

إذا كان captcha مطلوبًا:

1.  افتح `https://signalcaptchas.org/registration/generate.html`.
2.  أكمل captcha، انسخ رابط `signalcaptcha://...` من "Open Signal".
3.  قم بتشغيل التسجيل مرة أخرى من نفس عنوان IP الخارجي لجلسة المتصفح عندما يكون ذلك ممكنًا.
4.  قم بتشغيل التسجيل مرة أخرى فورًا (تنتهي صلاحية رموز captcha بسرعة):

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4.  قم بتكوين OpenClaw، أعد تشغيل البوابة، تحقق من القناة:

```bash
# إذا كنت تشغل البوابة كخدمة systemd للمستخدم:
systemctl --user restart openclaw-gateway

# ثم تحقق:
openclaw doctor
openclaw channels status --probe
```

5.  قم بإقران مرسل الرسائل المباشرة الخاص بك:
    -   أرسل أي رسالة إلى رقم البوت.
    -   وافق على الرمز على الخادم: `openclaw pairing approve signal <PAIRING_CODE>`.
    -   احفظ رقم البوت كجهة اتصال على هاتفك لتجنب "جهة اتصال غير معروفة".

مهم: تسجيل حساب برقم هاتف باستخدام `signal-cli` يمكن أن يلغي مصادقة جلسة تطبيق Signal الرئيسية لهذا الرقم. يُفضل استخدام رقم بوت مخصص، أو استخدم وضع رابط QR إذا كنت بحاجة إلى الحفاظ على إعداد تطبيق الهاتف الحالي الخاص بك. مراجع المصادر العلوية:

-   `signal-cli` README: `https://github.com/AsamK/signal-cli`
-   تدفق Captcha: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
-   تدفق الربط: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## وضع Daemon الخارجي (httpUrl)

إذا كنت تريد إدارة `signal-cli` بنفسك (بدايات باردة بطيئة لـ JVM، تهيئة الحاوية، أو وحدات معالجة مركزية مشتركة)، قم بتشغيل الـ daemon بشكل منفصل وأشر OpenClaw إليه:

```json
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

يتخطى هذا التشغيل التلقائي ووقت الانتظار للبدء داخل OpenClaw. للبدايات البطيئة عند التشغيل التلقائي، قم بتعيين `channels.signal.startupTimeoutMs`.

## التحكم في الوصول (الرسائل المباشرة + المجموعات)

الرسائل المباشرة:

-   الافتراضي: `channels.signal.dmPolicy = "pairing"`.
-   يتلقى المرسلون غير المعروفين رمز إقران؛ يتم تجاهل الرسائل حتى الموافقة (تنتهي رموز الإقران بعد ساعة واحدة).
-   وافق عبر:
    -   `openclaw pairing list signal`
    -   `openclaw pairing approve signal `
-   الإقران هو تبادل الرمز الافتراضي للرسائل المباشرة في Signal. التفاصيل: [الإقران](./pairing.md)
-   يتم تخزين المرسلين باستخدام UUID فقط (من `sourceUuid`) كـ `uuid:` في `channels.signal.allowFrom`.

المجموعات:

-   `channels.signal.groupPolicy = open | allowlist | disabled`.
-   `channels.signal.groupAllowFrom` يتحكم في من يمكنه التشغيل في المجموعات عند تعيين `allowlist`.
-   ملاحظة وقت التشغيل: إذا كان `channels.signal` مفقودًا تمامًا، يعود وقت التشغيل إلى `groupPolicy="allowlist"` للتحقق من المجموعات (حتى إذا تم تعيين `channels.defaults.groupPolicy`).

## كيفية عمله (السلوك)

-   يعمل `signal-cli` كـ daemon؛ تقرأ البوابة الأحداث عبر SSE.
-   يتم توحيد الرسائل الواردة في مغلف القناة المشترك.
-   دائمًا ما يتم توجيه الردود إلى نفس الرقم أو المجموعة.

## الوسائط + الحدود

-   يتم تقسيم النص الصادر إلى `channels.signal.textChunkLimit` (الافتراضي 4000).
-   تقسيم الأسطر الجديدة الاختياري: عيّن `channels.signal.chunkMode="newline"` لتقسيم على الأسطر الفارغة (حدود الفقرات) قبل التقسيم حسب الطول.
-   المرفقات مدعومة (يتم جلبها بـ base64 من `signal-cli`).
-   الحد الأقصى الافتراضي للوسائط: `channels.signal.mediaMaxMb` (الافتراضي 8).
-   استخدم `channels.signal.ignoreAttachments` لتخطي تنزيل الوسائط.
-   سياق تاريخ المجموعة يستخدم `channels.signal.historyLimit` (أو `channels.signal.accounts.*.historyLimit`)، مع التراجع إلى `messages.groupChat.historyLimit`. عيّن `0` لتعطيل (الافتراضي 50).

## الكتابة + إيصالات القراءة

-   **مؤشرات الكتابة**: يرسل OpenClaw إشارات الكتابة عبر `signal-cli sendTyping` ويجددها أثناء تشغيل الرد.
-   **إيصالات القراءة**: عندما يكون `channels.signal.sendReadReceipts` صحيحًا، يقوم OpenClaw بإعادة توجيه إيصالات القراءة للرسائل المباشرة المسموح بها.
-   لا يعرض signal-cli إيصالات القراءة للمجموعات.

## التفاعلات (أداة الرسالة)

-   استخدم `message action=react` مع `channel=signal`.
-   الأهداف: مرسل E.164 أو UUID (استخدم `uuid:` من ناتج الإقران؛ UUID العاري يعمل أيضًا).
-   `messageId` هو الطابع الزمني لـ Signal للرسالة التي تتفاعل معها.
-   تفاعلات المجموعة تتطلب `targetAuthor` أو `targetAuthorUuid`.

أمثلة:

```bash
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

التكوين:

-   `channels.signal.actions.reactions`: تمكين/تعطيل إجراءات التفاعل (الافتراضي صحيح).
-   `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
    -   `off`/`ack` يعطل تفاعلات الوكيل (ستؤدي أداة الرسالة `react` إلى خطأ).
    -   `minimal`/`extensive` يمكّن تفاعلات الوكيل ويضبط مستوى التوجيه.
-   تجاوزات لكل حساب: `channels.signal.accounts..actions.reactions`, `channels.signal.accounts..reactionLevel`.

## أهداف التسليم (CLI/cron)

-   الرسائل المباشرة: `signal:+15551234567` (أو E.164 عادي).
-   الرسائل المباشرة باستخدام UUID: `uuid:` (أو UUID عاري).
-   المجموعات: `signal:group:`.
-   أسماء المستخدمين: `username:` (إذا كان مدعومًا من حساب Signal الخاص بك).

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
openclaw pairing list signal
```

الإخفاقات الشائعة:

-   يمكن الوصول إلى Daemon ولكن لا توجد ردود: تحقق من إعدادات الحساب/daemon (`httpUrl`, `account`) ووضع الاستقبال.
-   يتم تجاهل الرسائل المباشرة: المرسل معلق بانتظار موافقة الإقران.
-   يتم تجاهل رسائل المجموعة: بوابات مرسل/ذكر المجموعة تمنع التسليم.
-   أخطاء التحقق من التكوين بعد التعديلات: قم بتشغيل `openclaw doctor --fix`.
-   Signal مفقود من التشخيصات: تأكد من `channels.signal.enabled: true`.

فحوصات إضافية:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

لتدفق التقييم: [/channels/troubleshooting](./troubleshooting.md).

## ملاحظات الأمان

-   يخزن `signal-cli` مفاتيح الحساب محليًا (عادة `~/.local/share/signal-cli/data/`).
-   قم بنسخ حالة حساب Signal احتياطيًا قبل ترحيل الخادم أو إعادة البناء.
-   حافظ على `channels.signal.dmPolicy: "pairing"` إلا إذا كنت تريد وصولًا أوسع للرسائل المباشرة بشكل صريح.
-   التحقق عبر SMS مطلوب فقط لتدفقات التسجيل أو الاستعادة، لكن فقدان السيطرة على الرقم/الحساب يمكن أن يعقد إعادة التسجيل.

## مرجع التكوين (Signal)

التكوين الكامل: [التكوين](../gateway/configuration.md) خيارات المزود:

-   `channels.signal.enabled`: تمكين/تعطيل بدء تشغيل القناة.
-   `channels.signal.account`: E.164 لحساب البوت.
-   `channels.signal.cliPath`: المسار إلى `signal-cli`.
-   `channels.signal.httpUrl`: عنوان URL الكامل لـ daemon (يتجاوز المضيف/المنفذ).
-   `channels.signal.httpHost`, `channels.signal.httpPort`: ربط daemon (الافتراضي 127.0.0.1:8080).
-   `channels.signal.autoStart`: التشغيل التلقائي لـ daemon (الافتراضي صحيح إذا لم يتم تعيين `httpUrl`).
-   `channels.signal.startupTimeoutMs`: مهلة انتظار البدء بالمللي ثانية (الحد الأقصى 120000).
-   `channels.signal.receiveMode`: `on-start | manual`.
-   `channels.signal.ignoreAttachments`: تخطي تنزيل المرفقات.
-   `channels.signal.ignoreStories`: تجاهل القصص من daemon.
-   `channels.signal.sendReadReceipts`: إعادة توجيه إيصالات القراءة.
-   `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (الافتراضي: pairing).
-   `channels.signal.allowFrom`: قائمة السماح للرسائل المباشرة (E.164 أو `uuid:`). `open` يتطلب `"*"`. Signal لا يحتوي على أسماء مستخدمين؛ استخدم معرفات الهاتف/UUID.
-   `channels.signal.groupPolicy`: `open | allowlist | disabled` (الافتراضي: allowlist).
-   `channels.signal.groupAllowFrom`: قائمة السماح لمرسلي المجموعات.
-   `channels.signal.historyLimit`: الحد الأقصى لرسائل المجموعة المضمنة كسياق (0 يعطل).
-   `channels.signal.dmHistoryLimit`: حد تاريخ الرسائل المباشرة بدورات المستخدم. تجاوزات لكل مستخدم: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
-   `channels.signal.textChunkLimit`: حجم جزء النص الصادر (أحرف).
-   `channels.signal.chunkMode`: `length` (الافتراضي) أو `newline` لتقسيم على الأسطر الفارغة (حدود الفقرات) قبل التقسيم حسب الطول.
-   `channels.signal.mediaMaxMb`: الحد الأقصى للوسائط الواردة/الصادرة (ميغابايت).

الخيارات العامة ذات الصلة:

-   `agents.list[].groupChat.mentionPatterns` (Signal لا يدعم الإشارات الأصلية).
-   `messages.groupChat.mentionPatterns` (التراجع العام).
-   `messages.responsePrefix`.

[Nostr](./nostr.md)[Synology Chat](./synology-chat.md)