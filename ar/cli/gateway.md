

  أوامر CLI

  
# gateway

البوابة هي خادم WebSocket الخاص بـ OpenClaw (القنوات، العقد، الجلسات، الخطافات). الأوامر الفرعية في هذه الصفحة تندرج تحت `openclaw gateway …`. الوثائق ذات الصلة:

-   [/gateway/bonjour](../gateway/bonjour.md)
-   [/gateway/discovery](../gateway/discovery.md)
-   [/gateway/configuration](../gateway/configuration.md)

## تشغيل البوابة

قم بتشغيل عملية بوابة محلية:

```bash
openclaw gateway
```

اسم مستعار للتشغيل في المقدمة:

```bash
openclaw gateway run
```

ملاحظات:

-   بشكل افتراضي، ترفض البوابة البدء ما لم يتم تعيين `gateway.mode=local` في `~/.openclaw/openclaw.json`. استخدم `--allow-unconfigured` للتشغيل المؤقت/التطويري.
-   الربط خارج النطاق المحلي بدون مصادقة محظور (إجراء أمان احترازي).
-   `SIGUSR1` يؤدي إلى إعادة تشغيل داخلية للعملية عند التفويض (`commands.restart` مفعل افتراضيًا؛ عيّن `commands.restart: false` لمنع إعادة التشغيل يدويًا، بينما يظل تطبيق/تحديث أداة/إعدادات البوابة مسموحًا به).
-   معالجات `SIGINT`/`SIGTERM` توقف عملية البوابة، لكنها لا تستعيد أي حالة طرفية مخصصة. إذا قمت بتغليف CLI بـ TUI أو إدخال بالوضع الخام، قم باستعادة الطرفية قبل الخروج.

### الخيارات

-   `--port `: منفذ WebSocket (الافتراضي يأتي من الإعدادات/البيئة؛ عادةً `18789`).
-   `--bind <loopback|lan|tailnet|auto|custom>`: وضع ربط المستمع.
-   `--auth <token|password>`: تجاوز وضع المصادقة.
-   `--token `: تجاوز الرمز المميز (يضبط أيضًا `OPENCLAW_GATEWAY_TOKEN` للعملية).
-   `--password `: تجاوز كلمة المرور (يضبط أيضًا `OPENCLAW_GATEWAY_PASSWORD` للعملية).
-   `--tailscale <off|serve|funnel>`: عرض البوابة عبر Tailscale.
-   `--tailscale-reset-on-exit`: إعادة تعيين إعدادات Tailscale serve/funnel عند الإغلاق.
-   `--allow-unconfigured`: السماح ببدء البوابة بدون `gateway.mode=local` في الإعدادات.
-   `--dev`: إنشاء إعدادات تطوير + مساحة عمل إذا كانت مفقودة (يتخطى BOOTSTRAP.md).
-   `--reset`: إعادة تعيين إعدادات التطوير + بيانات الاعتماد + الجلسات + مساحة العمل (يتطلب `--dev`).
-   `--force`: إنهاء أي مستمع موجود على المنفذ المحدد قبل البدء.
-   `--verbose`: سجلات تفصيلية.
-   `--claude-cli-logs`: عرض سجلات claude-cli فقط في وحدة التحكم (وتمكين stdout/stderr الخاصة بها).
-   `--ws-log <auto|full|compact>`: نمط سجل websocket (الافتراضي `auto`).
-   `--compact`: اسم مستعار لـ `--ws-log compact`.
-   `--raw-stream`: تسجيل أحداث دفق النموذج الخام إلى jsonl.
-   `--raw-stream-path `: مسار jsonl لدفق البيانات الخام.

## استعلام بوابة قيد التشغيل

جميع أوامر الاستعلام تستخدم WebSocket RPC. أوضاع الإخراج:

-   الافتراضي: قابل للقراءة البشرية (ملون في TTY).
-   `--json`: JSON قابل للقراءة الآلية (بدون تنسيق/مؤشر تدوير).
-   `--no-color` (أو `NO_COLOR=1`): تعطيل ANSI مع الحفاظ على التخطيط البشري.

الخيارات المشتركة (حيث تكون مدعومة):

-   `--url `: عنوان URL لـ WebSocket الخاص بالبوابة.
-   `--token `: رمز البوابة المميز.
-   `--password `: كلمة مرور البوابة.
-   `--timeout `: المهلة/الميزانية (تختلف حسب الأمر).
-   `--expect-final`: انتظار استجابة "نهائية" (مكالمات الوكيل).

ملاحظة: عند تعيين `--url`، لا يتراجع CLI إلى بيانات اعتماد الإعدادات أو البيئة. قم بتمرير `--token` أو `--password` صراحةً. عدم وجود بيانات اعتماد صريحة يعتبر خطأ.

### gateway health

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### gateway status

`gateway status` يعرض خدمة البوابة (launchd/systemd/schtasks) بالإضافة إلى مسبار RPC اختياري.

```bash
openclaw gateway status
openclaw gateway status --json
```

الخيارات:

-   `--url `: تجاوز عنوان URL للمسبار.
-   `--token `: مصادقة الرمز المميز للمسبار.
-   `--password `: مصادقة كلمة المرور للمسبار.
-   `--timeout `: مهلة المسبار (الافتراضي `10000`).
-   `--no-probe`: تخطي مسبار RPC (عرض الخدمة فقط).
-   `--deep`: فحص خدمات مستوى النظام أيضًا.

ملاحظات:

-   `gateway status` يحل SecretRefs المصادقة المُعدة لمصادقة المسبار عندما يكون ذلك ممكنًا.
-   إذا كان SecretRef المصادقة المطلوب غير محلول في مسار هذا الأمر، فقد تفشل مصادقة المسبار؛ قم بتمرير `--token`/`--password` صراحةً أو قم بحل مصدر السر أولاً.

### gateway probe

`gateway probe` هو أمر "تصحيح كل شيء". يقوم دائمًا بفحص:

-   البوابة البعيدة المُعدة لديك (إذا تم تعيينها)، و
-   localhost (loopback) **حتى إذا تم تكوين بوابة بعيدة**.

إذا كانت هناك بوابات متعددة يمكن الوصول إليها، فإنه يطبعها جميعًا. يتم دعم البوابات المتعددة عند استخدام ملفات تعريف/منافذ معزولة (مثل، بوت الإنقاذ)، لكن معظم التثبيتات لا تزال تشغل بوابة واحدة.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

#### عن بعد عبر SSH (تكافؤ تطبيق Mac)

يستخدم وضع "عن بعد عبر SSH" في تطبيق macOS إعادة توجيه منفذ محلي بحيث تصبح البوابة البعيدة (والتي قد تكون مقيدة بـ loopback فقط) قابلة للوصول عند `ws://127.0.0.1:`. المعادل في CLI:

```bash
openclaw gateway probe --ssh user@gateway-host
```

الخيارات:

-   `--ssh `: `user@host` أو `user@host:port` (المنفذ الافتراضي `22`).
-   `--ssh-identity `: ملف الهوية.
-   `--ssh-auto`: اختيار أول مضيف بوابة تم اكتشافه كهدف SSH (LAN/WAB فقط).

الإعدادات (اختياري، تُستخدم كافتراضيات):

-   `gateway.remote.sshTarget`
-   `gateway.remote.sshIdentity`

### gateway call &lt;method&gt;

مساعد RPC منخفض المستوى.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

## إدارة خدمة البوابة

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

ملاحظات:

-   `gateway install` يدعم `--port`, `--runtime`, `--token`, `--force`, `--json`.
-   عندما تتطلب مصادقة الرمز المميز رمزًا وكان `gateway.auth.token` مُدارًا بواسطة SecretRef، فإن `gateway install` يتحقق من إمكانية حل SecretRef ولكنه لا يحفظ الرمز المميز المحلول في بيانات تعريف بيئة الخدمة.
-   إذا كانت مصادقة الرمز المميز تتطلب رمزًا وكان SecretRef للرمز المميز المُعد غير محلول، فإن التثبيت يفشل مغلقًا بدلاً من حفظ نص عادي بديل.
-   في وضع المصادقة المُستنتج، لا يخفف `OPENCLAW_GATEWAY_PASSWORD`/`CLAWDBOT_GATEWAY_PASSWORD` المقتصر على shell متطلبات رمز التثبيت؛ استخدم إعدادات دائمة (`gateway.auth.password` أو `env` في الإعدادات) عند تثبيت خدمة مُدارة.
-   إذا تم تكوين كل من `gateway.auth.token` و `gateway.auth.password` ولم يتم تعيين `gateway.auth.mode`، يتم حظر التثبيت حتى يتم تعيين الوضع صراحةً.
-   أوامر دورة الحياة تقبل `--json` للبرمجة النصية.

## اكتشاف البوابات (Bonjour)

`gateway discover` يفحص بحثًا عن منارات البوابة (`_openclaw-gw._tcp`).

-   Multicast DNS-SD: `local.`
-   Unicast DNS-SD (Wide-Area Bonjour): اختر نطاقًا (مثال: `openclaw.internal.`) وقم بإعداد DNS منقسم + خادم DNS؛ انظر [/gateway/bonjour](../gateway/bonjour.md)

فقط البوابات التي تم تمكين اكتشاف Bonjour فيها (الافتراضي) تعلن عن المنارة. سجلات الاكتشاف واسعة النطاق تتضمن (TXT):

-   `role` (تلميح دور البوابة)
-   `transport` (تلميح النقل، مثل `gateway`)
-   `gatewayPort` (منفذ WebSocket، عادةً `18789`)
-   `sshPort` (منفذ SSH؛ الافتراضي `22` إذا لم يكن موجودًا)
-   `tailnetDns` (اسم مضيف MagicDNS، عند التوفر)
-   `gatewayTls` / `gatewayTlsSha256` (تمكين TLS + بصمة الشهادة)
-   `cliPath` (تلميح اختياري للتثبيتات البعيدة)

### gateway discover

```bash
openclaw gateway discover
```

الخيارات:

-   `--timeout `: المهلة لكل أمر (تصفح/حل)؛ الافتراضي `2000`.
-   `--json`: إخراج قابل للقراءة الآلية (يعطل أيضًا التنسيق/مؤشر التدوير).

أمثلة:

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```

[doctor](./doctor.md)[health](./health.md)