title: "دليل استكشاف أخطاء بوابة OpenClaw وإصلاحها"
description: "حل مشاكل بوابة OpenClaw الشائعة: الخدمة لا تعمل، لا توجد ردود، أخطاء 429، مشاكل اتصال لوحة التحكم، ومشاكل تدفق الرسائل مع أوامر خطوة بخطوة."
keywords: ["استكشاف أخطاء openclaw", "البوابة لا تعمل", "خطأ 429 من anthropic", "اتصال لوحة التحكم", "مشاكل تدفق الرسائل", "إقران القناة", "نبض cron", "فشل أداة المتصفح"]
---

  التكوين والعمليات

  
# استكشاف الأخطاء وإصلاحها

هذه الصفحة هي دليل التشغيل العميق. ابدأ من [/help/troubleshooting](../help/troubleshooting.md) إذا كنت تريد تدفق التصنيف السريع أولاً.

## سلم الأوامر

شغّل هذه أولاً، بهذا الترتيب:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

الإشارات الصحية المتوقعة:

-   `openclaw gateway status` تظهر `Runtime: running` و `RPC probe: ok`.
-   `openclaw doctor` لا تبلغ عن أي مشاكل في التكوين/الخدمة.
-   `openclaw channels status --probe` تظهر قنوات متصلة/جاهزة.

## Anthropic 429 استخدام إضافي مطلوب للسياق الطويل

استخدم هذا عندما تحتوي السجلات/الأخطاء على: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`.

```bash
openclaw logs --follow
openclaw models status
openclaw config get agents.defaults.models
```

ابحث عن:

-   نموذج Anthropic Opus/Sonnet المحدد لديه `params.context1m: true`.
-   بيانات اعتماد Anthropic الحالية غير مؤهلة للاستخدام طويل السياق.
-   تفشل الطلبات فقط في جلسات/تشغيلات النماذج الطويلة التي تحتاج إلى مسار 1M التجريبي.

خيارات الإصلاح:

1.  تعطيل `context1m` لذلك النموذج للعودة إلى نافذة السياق العادية.
2.  استخدم مفتاح Anthropic API مع الفوترة، أو فعّل "الاستخدام الإضافي" (Extra Usage) على حساب الاشتراك.
3.  تكوين نماذج احتياطية (fallback) لاستمرار التشغيل عند رفض طلبات Anthropic طويلة السياق.

ذات صلة:

-   [/providers/anthropic](../providers/anthropic.md)
-   [/reference/token-use](../reference/token-use.md)
-   [/help/faq#why-am-i-seeing-http-429-ratelimiterror-from-anthropic](../help/faq.md#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)

## لا توجد ردود

إذا كانت القنوات نشطة ولكن لا شيء يجيب، تحقق من التوجيه والسياسة قبل إعادة توصيل أي شيء.

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw config get channels
openclaw logs --follow
```

ابحث عن:

-   إقران معلق لمرسلي الرسائل المباشرة (DM).
-   بوابات الإشارة في المجموعات (`requireMention`, `mentionPatterns`).
-   عدم تطابق قوائم السماح للقناة/المجموعة.

التوقيعات الشائعة:

-   `drop guild message (mention required` → تم تجاهل رسالة المجموعة حتى الإشارة.
-   `pairing request` → يحتاج المرسل إلى الموافقة.
-   `blocked` / `allowlist` → تم تصفية المرسل/القناة بواسطة السياسة.

ذات صلة:

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)
-   [/channels/groups](../channels/groups.md)

## اتصال واجهة تحكم لوحة التحكم

عندما لا تتصل لوحة التحكم/واجهة التحكم، تحقق من صحة عنوان URL، ووضع المصادقة، وافتراضات السياق الآمن.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

ابحث عن:

-   عنوان URL الصحيح للفحص وعنوان URL للوحة التحكم.
-   عدم تطابق وضع/رمز المصادقة بين العميل والبوابة.
-   استخدام HTTP حيث تكون هوية الجهاز مطلوبة.

التوقيعات الشائعة:

-   `device identity required` → سياق غير آمن أو مصادقة جهاز مفقودة.
-   `device nonce required` / `device nonce mismatch` → العميل لا يكمل تدفق مصادقة الجهاز القائم على التحدي (`connect.challenge` + `device.nonce`).
-   `device signature invalid` / `device signature expired` → وقّع العميل على الحمولة الخاطئة (أو الطابع الزمني القديم) لمصافحة اليد الحالية.
-   `unauthorized` / حلقة إعادة الاتصال → عدم تطابق الرمز/كلمة المرور.
-   `gateway connect failed:` → هدف مضيف/منفذ/عنوان URL خاطئ.

فحص ترقية مصادقة الجهاز الإصدار 2:

```bash
openclaw --version
openclaw doctor
openclaw gateway status
```

إذا أظهرت السجلات أخطاء nonce/توقيع، قم بتحديث العميل المتصل وتحقق من أنه:

1.  ينتظر `connect.challenge`
2.  يوقّع على الحمولة المرتبطة بالتحدي
3.  يرسل `connect.params.device.nonce` بنفس nonce التحدي

ذات صلة:

-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/remote](./remote.md)

## خدمة البوابة لا تعمل

استخدم هذا عندما تكون الخدمة مثبتة ولكن العملية لا تبقى قيد التشغيل.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

ابحث عن:

-   `Runtime: stopped` مع تلميحات خروج.
-   عدم تطابق تكوين الخدمة (`Config (cli)` مقابل `Config (service)`).
-   تعارضات المنفذ/المستمع.

التوقيعات الشائعة:

-   `Gateway start blocked: set gateway.mode=local` → وضع البوابة المحلي غير مفعّل. الإصلاح: اضبط `gateway.mode="local"` في تكوينك (أو شغّل `openclaw configure`). إذا كنت تشغّل OpenClaw عبر Podman باستخدام المستخدم المخصص `openclaw`، فإن التكوين موجود في `~openclaw/.openclaw/openclaw.json`.
-   `refusing to bind gateway ... without auth` → ربط غير loopback بدون رمز/كلمة مرور.
-   `another gateway instance is already listening` / `EADDRINUSE` → تعارض في المنفذ.

ذات صلة:

-   [/gateway/background-process](./background-process.md)
-   [/gateway/configuration](./configuration.md)
-   [/gateway/doctor](./doctor.md)

## القناة متصلة ولكن الرسائل لا تتدفق

إذا كانت حالة القناة متصلة ولكن تدفق الرسائل متوقف، ركز على السياسة، الأذونات، وقواعد التسليم الخاصة بالقناة.

```bash
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

ابحث عن:

-   سياسة الرسائل المباشرة (DM) (`pairing`, `allowlist`, `open`, `disabled`).
-   قائمة السماح للمجموعة ومتطلبات الإشارة.
-   أذونات/نطاقات واجهة برمجة التطبيقات (API) للقناة المفقودة.

التوقيعات الشائعة:

-   `mention required` → تم تجاهل الرسالة بواسطة سياسة إشارة المجموعة.
-   `pairing` / آثار موافقة معلقة → المرسل غير معتمد.
-   `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → مشكلة في مصادقة/أذونات القناة.

ذات صلة:

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/whatsapp](../channels/whatsapp.md)
-   [/channels/telegram](../channels/telegram.md)
-   [/channels/discord](../channels/discord.md)

## تسليم Cron ونبض القلب

إذا لم يعمل cron أو نبض القلب أو لم يتم تسليمه، تحقق من حالة المجدول أولاً، ثم هدف التسليم.

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

ابحث عن:

-   cron مفعّل ويوجد استيقاظ تالي.
-   حالة تاريخ تشغيل المهمة (`ok`, `skipped`, `error`).
-   أسباب تخطي نبض القلب (`quiet-hours`, `requests-in-flight`, `alerts-disabled`).

التوقيعات الشائعة:

-   `cron: scheduler disabled; jobs will not run automatically` → cron معطل.
-   `cron: timer tick failed` -> فشل نبض المؤقت للمجدول؛ تحقق من أخطاء الملف/السجل/وقت التشغيل.
-   `heartbeat skipped` مع `reason=quiet-hours` → خارج نافذة الساعات النشطة.
-   `heartbeat: unknown accountId` → معرف حساب غير صالح لهدف تسليم نبض القلب.
-   `heartbeat skipped` مع `reason=dm-blocked` → تم حل هدف نبض القلب إلى وجهة من نوع الرسائل المباشرة (DM) بينما تم تعيين `agents.defaults.heartbeat.directPolicy` (أو التجاوز لكل وكيل) على `block`.

ذات صلة:

-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/automation/cron-jobs](../automation/cron-jobs.md)
-   [/gateway/heartbeat](./heartbeat.md)

## فشل أداة العقدة المقترنة

إذا كانت العقدة مقترنة ولكن الأدوات تفشل، اعزل حالة المقدمة، الإذن، والموافقة.

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

ابحث عن:

-   العقدة متصلة بالإنترنت مع القدرات المتوقعة.
-   منح إذن نظام التشغيل للكاميرا/الميكروفون/الموقع/الشاشة.
-   موافقات التنفيذ وحالة قائمة السماح.

التوقيعات الشائعة:

-   `NODE_BACKGROUND_UNAVAILABLE` → يجب أن يكون تطبيق العقدة في المقدمة.
-   `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → إذن نظام تشغيل مفقود.
-   `SYSTEM_RUN_DENIED: approval required` → موافقة تنفيذ معلقة.
-   `SYSTEM_RUN_DENIED: allowlist miss` → أمر محظور بواسطة قائمة السماح.

ذات صلة:

-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/nodes/index](../nodes/index.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

## فشل أداة المتصفح

استخدم هذا عندما تفشل إجراءات أداة المتصفح على الرغم من أن البوابة نفسها سليمة.

```bash
openclaw browser status
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

ابحث عن:

-   مسار تنفيذي صالح للمتصفح.
-   إمكانية الوصول إلى ملف تعريف CDP.
-   إرفاق علامة تبويب امتداد الترحيل لـ `profile="chrome"`.

التوقيعات الشائعة:

-   `Failed to start Chrome CDP on port` → فشل تشغيل عملية المتصفح.
-   `browser.executablePath not found` → المسار المُكوّن غير صالح.
-   `Chrome extension relay is running, but no tab is connected` → امتداد الترحيل غير مرفق.
-   `Browser attachOnly is enabled ... not reachable` → ملف تعريف الإرفاق فقط لا يحتوي على هدف يمكن الوصول إليه.

ذات صلة:

-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)
-   [/tools/browser](../tools/browser.md)

## إذا قمت بالترقية وفشل شيء ما فجأة

معظم الأعطال بعد الترقية ناتجة عن انحراف التكوين أو تطبيق الإعدادات الافتراضية الأكثر صرامة الآن.

### 1) تغير سلوك تجاوز المصادقة وعنوان URL

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

ما يجب التحقق منه:

-   إذا كان `gateway.mode=remote`، فقد تستهدف استدعاءات CLI البوابة البعيدة بينما خدمتك المحلية تعمل بشكل جيد.
-   استدعاءات `--url` الصريحة لا تعود إلى بيانات الاعتماد المخزنة.

التوقيعات الشائعة:

-   `gateway connect failed:` → هدف عنوان URL خاطئ.
-   `unauthorized` → نقطة النهاية قابلة للوصول ولكن المصادقة خاطئة.

### 2) حواجز الربط والمصادقة أصبحت أكثر صرامة

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

ما يجب التحقق منه:

-   عمليات الربط غير loopback (`lan`, `tailnet`, `custom`) تحتاج إلى تكوين المصادقة.
-   المفاتيح القديمة مثل `gateway.token` لا تحل محل `gateway.auth.token`.

التوقيعات الشائعة:

-   `refusing to bind gateway ... without auth` → عدم تطابق الربط+المصادقة.
-   `RPC probe: failed` بينما وقت التشغيل قيد التشغيل → البوابة نشطة ولكن لا يمكن الوصول إليها بالمصادقة/عنوان URL الحالي.

### 3) تغيرت حالة إقران الجهاز وهويتة

```bash
openclaw devices list
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
openclaw doctor
```

ما يجب التحقق منه:

-   موافقات جهاز معلقة للوحة التحكم/العقد.
-   موافقات إقران الرسائل المباشرة (DM) معلقة بعد تغييرات السياسة أو الهوية.

التوقيعات الشائعة:

-   `device identity required` → لم يتم استيفاء مصادقة الجهاز.
-   `pairing required` → يجب الموافقة على المرسل/الجهاز.

إذا كان تكوين الخدمة ووقت التشغيل لا يزالان غير متوافقين بعد الفحوصات، أعد تثبيت بيانات وصفية للخدمة من نفس دليل الملف الشخصي/الحالة:

```bash
openclaw gateway install --force
openclaw gateway restart
```

ذات صلة:

-   [/gateway/pairing](./pairing.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/background-process](./background-process.md)

[بوابات متعددة](./multiple-gateways.md)[الأمان](./security.md)