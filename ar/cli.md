title: "مرجع OpenClaw CLI: الأوامر، الأعلام، ودليل الاستخدام"
description: "مرجع كامل لأوامر OpenClaw CLI، الأعلام العامة، تنسيق الإخراج، والاستخدام. تعلم إعداد، تكوين، وإدارة الوكلاء، الإضافات، الأمان، والذاكرة."
keywords: ["openclaw cli", "أوامر cli", "مرجع cli", "أوامر الوكيل", "إعداد cli", "تكوين cli", "أدوات المطور", "واجهة سطر الأوامر"]
---

  أوامر CLI

  
# مرجع CLI

تصف هذه الصفحة سلوك CLI الحالي. إذا تغيرت الأوامر، قم بتحديث هذا المستند.

## صفحات الأوامر

-   [`setup`](./cli/setup.md)
-   [`onboard`](./cli/onboard.md)
-   [`configure`](./cli/configure.md)
-   [`config`](./cli/config.md)
-   [`completion`](./cli/completion.md)
-   [`doctor`](./cli/doctor.md)
-   [`dashboard`](./cli/dashboard.md)
-   [`reset`](./cli/reset.md)
-   [`uninstall`](./cli/uninstall.md)
-   [`update`](./cli/update.md)
-   [`message`](./cli/message.md)
-   [`agent`](./cli/agent.md)
-   [`agents`](./cli/agents.md)
-   [`acp`](./cli/acp.md)
-   [`status`](./cli/status.md)
-   [`health`](./cli/health.md)
-   [`sessions`](./cli/sessions.md)
-   [`gateway`](./cli/gateway.md)
-   [`logs`](./cli/logs.md)
-   [`system`](./cli/system.md)
-   [`models`](./cli/models.md)
-   [`memory`](./cli/memory.md)
-   [`directory`](./cli/directory.md)
-   [`nodes`](./cli/nodes.md)
-   [`devices`](./cli/devices.md)
-   [`node`](./cli/node.md)
-   [`approvals`](./cli/approvals.md)
-   [`sandbox`](./cli/sandbox.md)
-   [`tui`](./cli/tui.md)
-   [`browser`](./cli/browser.md)
-   [`cron`](./cli/cron.md)
-   [`dns`](./cli/dns.md)
-   [`docs`](./cli/docs.md)
-   [`hooks`](./cli/hooks.md)
-   [`webhooks`](./cli/webhooks.md)
-   [`pairing`](./cli/pairing.md)
-   [`qr`](./cli/qr.md)
-   [`plugins`](./cli/plugins.md) (أوامر الإضافات)
-   [`channels`](./cli/channels.md)
-   [`security`](./cli/security.md)
-   [`secrets`](./cli/secrets.md)
-   [`skills`](./cli/skills.md)
-   [`daemon`](./cli/daemon.md) (اسم مستعار قديم لأوامر خدمة البوابة)
-   [`clawbot`](./cli/clawbot.md) (مساحة اسم مستعار قديمة)
-   [`voicecall`](./cli/voicecall.md) (إضافة؛ إذا كانت مثبتة)

## الأعلام العامة

-   `--dev`: عزل الحالة تحت `~/.openclaw-dev` وتحويل المنافذ الافتراضية.
-   `--profile `: عزل الحالة تحت `~/.openclaw-`.
-   `--no-color`: تعطيل ألوان ANSI.
-   `--update`: اختصار لـ `openclaw update` (التثبيتات من المصدر فقط).
-   `-V`, `--version`, `-v`: طباعة الإصدار والخروج.

## تنسيق الإخراج

-   ألوان ANSI ومؤشرات التقدم تُعرض فقط في جلسات TTY.
-   الروابط التشعبية OSC-8 تظهر كروابط قابلة للنقر في الطرفيات المدعومة؛ وإلا نعود إلى عناوين URL عادية.
-   `--json` (و `--plain` حيث يكون مدعومًا) يعطل التنسيق لإخراج نظيف.
-   `--no-color` يعطل تنسيق ANSI؛ `NO_COLOR=1` محترم أيضًا.
-   الأوامر طويلة المدى تُظهر مؤشر تقدم (OSC 9;4 عندما يكون مدعومًا).

## لوحة الألوان

يستخدم OpenClaw لوحة ألوان جراد البحر لإخراج CLI.

-   `accent` (#FF5A2D): العناوين، التسميات، التمييزات الأساسية.
-   `accentBright` (#FF7A3D): أسماء الأوامر، التأكيد.
-   `accentDim` (#D14A22): نص التمييز الثانوي.
-   `info` (#FF8A5B): القيم المعلوماتية.
-   `success` (#2FBF71): حالات النجاح.
-   `warn` (#FFB020): التحذيرات، الاحتياطات، الانتباه.
-   `error` (#E23D2D): الأخطاء، الإخفاقات.
-   `muted` (#8B7F77): التقليل من التركيز، البيانات الوصفية.

مصدر الحقيقة للوحة: `src/terminal/palette.ts` (المعروفة أيضًا باسم "lobster seam").

## شجرة الأوامر

```bash
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  completion
  doctor
  dashboard
  security
    audit
  secrets
    reload
    migrate
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  directory
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  daemon
    status
    install
    uninstall
    start
    stop
    restart
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  qr
  clawbot
    qr
  docs
  dns
    setup
  tui
```

ملاحظة: يمكن للإضافات إضافة أوامر رئيسية إضافية (على سبيل المثال `openclaw voicecall`).

## الأمان

-   `openclaw security audit` — تدقيق التكوين + الحالة المحلية لمواطن ضعف الأمان الشائعة.
-   `openclaw security audit --deep` — فحص حي للبوابة بأقصى جهد ممكن.
-   `openclaw security audit --fix` — تشديد الإعدادات الافتراضية الآمنة وتغيير أذونات chmod للحالة/التكوين.

## الأسرار

-   `openclaw secrets reload` — إعادة حل المراجع وتبديل لقطة وقت التشغيل بشكل ذري.
-   `openclaw secrets audit` — فحص بقايا النص العادي، المراجع غير المحلولة، وانحراف الأولوية.
-   `openclaw secrets configure` — مساعد تفاعلي لإعداد المزود + تعيين SecretRef + التحقق المسبق/التطبيق.
-   `openclaw secrets apply --from <plan.json>` — تطبيق خطة تم إنشاؤها مسبقًا (`--dry-run` مدعوم).

## الإضافات

إدارة الامتدادات وتكوينها:

-   `openclaw plugins list` — اكتشاف الإضافات (استخدم `--json` للإخراج الآلي).
-   `openclaw plugins info ` — عرض تفاصيل إضافة.
-   `openclaw plugins install <path|.tgz|npm-spec>` — تثبيت إضافة (أو إضافة مسار إضافة إلى `plugins.load.paths`).
-   `openclaw plugins enable ` / `disable ` — تبديل `plugins.entries..enabled`.
-   `openclaw plugins doctor` — الإبلاغ عن أخطاء تحميل الإضافات.

معظم تغييرات الإضافات تتطلب إعادة تشغيل البوابة. انظر [/plugin](./tools/plugin.md).

## الذاكرة

بحث متجه عبر `MEMORY.md` + `memory/*.md`:

-   `openclaw memory status` — عرض إحصائيات الفهرس.
-   `openclaw memory index` — إعادة فهرسة ملفات الذاكرة.
-   `openclaw memory search ""` (أو `--query ""`) — بحث دلالي عبر الذاكرة.

## أوامر الشرطة المائلة للدردشة

تدعم رسائل الدردشة أوامر `/...` (نصية وأصلية). انظر [/tools/slash-commands](./tools/slash-commands.md). أبرز النقاط:

-   `/status` للتشخيص السريع.
-   `/config` لتغييرات التكوين المستمرة.
-   `/debug` لتجاوزات التكوين الخاصة بوقت التشغيل فقط (ذاكرة، وليس قرصًا؛ تتطلب `commands.debug: true`).

## الإعداد + التعريف

### setup

تهيئة التكوين + مساحة العمل. الخيارات:

-   `--workspace `: مسار مساحة عمل الوكيل (الافتراضي `~/.openclaw/workspace`).
-   `--wizard`: تشغيل معالج التعريف.
-   `--non-interactive`: تشغيل المعالج دون مطالبات.
-   `--mode <local|remote>`: وضع المعالج.
-   `--remote-url `: عنوان URL للبوابة البعيدة.
-   `--remote-token `: رمز البوابة البعيدة.

يعمل المعالج تلقائيًا عند وجود أي أعلام للمعالج (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

### onboard

معالج تفاعلي لإعداد البوابة، مساحة العمل، والمهارات. الخيارات:

-   `--workspace `
-   `--reset` (إعادة تعيين التكوين + بيانات الاعتماد + الجلسات قبل المعالج)
-   `--reset-scope <config|config+creds+sessions|full>` (الافتراضي `config+creds+sessions`؛ استخدم `full` لإزالة مساحة العمل أيضًا)
-   `--non-interactive`
-   `--mode <local|remote>`
-   `--flow <quickstart|advanced|manual>` (يدوي هو اسم مستعار لمتقدم)
-   `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|moonshot-api-key-cn|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|mistral-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|custom-api-key|skip>`
-   `--token-provider ` (غير تفاعلي؛ يستخدم مع `--auth-choice token`)
-   `--token ` (غير تفاعلي؛ يستخدم مع `--auth-choice token`)
-   `--token-profile-id ` (غير تفاعلي؛ الافتراضي: `:manual`)
-   `--token-expires-in ` (غير تفاعلي؛ على سبيل المثال `365d`, `12h`)
-   `--secret-input-mode <plaintext|ref>` (الافتراضي `plaintext`؛ استخدم `ref` لتخزين مراجع البيئة الافتراضية للمزود بدلاً من مفاتيح النص العادي)
-   `--anthropic-api-key `
-   `--openai-api-key `
-   `--mistral-api-key `
-   `--openrouter-api-key `
-   `--ai-gateway-api-key `
-   `--moonshot-api-key `
-   `--kimi-code-api-key `
-   `--gemini-api-key `
-   `--zai-api-key `
-   `--minimax-api-key `
-   `--opencode-zen-api-key `
-   `--custom-base-url ` (غير تفاعلي؛ يستخدم مع `--auth-choice custom-api-key`)
-   `--custom-model-id ` (غير تفاعلي؛ يستخدم مع `--auth-choice custom-api-key`)
-   `--custom-api-key ` (غير تفاعلي؛ اختياري؛ يستخدم مع `--auth-choice custom-api-key`؛ يعود إلى `CUSTOM_API_KEY` عند حذفه)
-   `--custom-provider-id ` (غير تفاعلي؛ اختياري؛ معرف مزود مخصص)
-   `--custom-compatibility <openai|anthropic>` (غير تفاعلي؛ اختياري؛ الافتراضي `openai`)
-   `--gateway-port `
-   `--gateway-bind <loopback|lan|tailnet|auto|custom>`
-   `--gateway-auth <token|password>`
-   `--gateway-token `
-   `--gateway-token-ref-env ` (غير تفاعلي؛ تخزين `gateway.auth.token` كـ SecretRef للبيئة؛ يتطلب تعيين متغير البيئة هذا؛ لا يمكن دمجه مع `--gateway-token`)
-   `--gateway-password `
-   `--remote-url `
-   `--remote-token `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--install-daemon`
-   `--no-install-daemon` (اسم مستعار: `--skip-daemon`)
-   `--daemon-runtime <node|bun>`
-   `--skip-channels`
-   `--skip-skills`
-   `--skip-health`
-   `--skip-ui`
-   `--node-manager <npm|pnpm|bun>` (يوصى بـ pnpm؛ لا يوصى بـ bun لوقت تشغيل البوابة)
-   `--json`

### configure

معالج التكوين التفاعلي (النماذج، القنوات، المهارات، البوابة).

### config

مساعدات التكوين غير التفاعلية (get/set/unset/file/validate). تشغيل `openclaw config` بدون أمر فرعي يطلق المعالج. الأوامر الفرعية:

-   `config get `: طباعة قيمة تكوين (مسار نقطة/قوس).
-   `config set  `: تعيين قيمة (JSON5 أو سلسلة نصية خام).
-   `config unset `: إزالة قيمة.
-   `config file`: طباعة مسار ملف التكوين النشط.
-   `config validate`: التحقق من صحة التكوين الحالي مقابل المخطط دون بدء البوابة.
-   `config validate --json`: إخراج JSON قابل للقراءة آليًا.

### doctor

فحوصات الصحة + الإصلاحات السريعة (التكوين + البوابة + الخدمات القديمة). الخيارات:

-   `--no-workspace-suggestions`: تعطيل تلميحات ذاكرة مساحة العمل.
-   `--yes`: قبول الإعدادات الافتراضية دون مطالبة (بدون واجهة).
-   `--non-interactive`: تخطي المطالبات؛ تطبيق الهجرات الآمنة فقط.
-   `--deep`: فحص خدمات النظام بحثًا عن تثبيتات بوابة إضافية.

## مساعدات القنوات

### channels

إدارة حسابات قنوات الدردشة (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (إضافة)/Signal/iMessage/MS Teams). الأوامر الفرعية:

-   `channels list`: عرض القنوات المكونة وملفات تعريف المصادقة.
-   `channels status`: التحقق من إمكانية الوصول إلى البوابة وصحة القناة (`--probe` يقوم بفحوصات إضافية؛ استخدم `openclaw health` أو `openclaw status --deep` لفحوصات صحة البوابة).
-   تلميح: `channels status` يطبع تحذيرات مع اقتراحات إصلاح عندما يمكنه اكتشاف الأخطاء الشائعة في التكوين (ثم يشير لك إلى `openclaw doctor`).
-   `channels logs`: عرض سجلات القناة الحديثة من ملف سجل البوابة.
-   `channels add`: إعداد على نمط المعالج عندما لا يتم تمرير أعلام؛ الأعلام تتحول إلى الوضع غير التفاعلي.
    -   عند إضافة حساب غير افتراضي إلى قناة لا تزال تستخدم تكوينًا رئيسيًا أحادي الحساب، يقوم OpenClaw بنقل القيم ذات النطاق الحسابي إلى `channels..accounts.default` قبل كتابة الحساب الجديد.
    -   `channels add` غير التفاعلي لا ينشئ/يحدث الربط تلقائيًا؛ تستمر عمليات الربط الخاصة بالقناة فقط في مطابقة الحساب الافتراضي.
-   `channels remove`: تعطيل بشكل افتراضي؛ مرر `--delete` لإزالة إدخالات التكوين دون مطالبات.
-   `channels login`: تسجيل دخول تفاعلي للقناة (WhatsApp Web فقط).
-   `channels logout`: تسجيل الخروج من جلسة قناة (إذا كانت مدعومة).

الخيارات المشتركة:

-   `--channel `: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
-   `--account `: معرف حساب القناة (الافتراضي `default`)
-   `--name `: اسم العرض للحساب

خيارات `channels login`:

-   `--channel ` (الافتراضي `whatsapp`؛ يدعم `whatsapp`/`web`)
-   `--account `
-   `--verbose`

خيارات `channels logout`:

-   `--channel ` (الافتراضي `whatsapp`)
-   `--account `

خيارات `channels list`:

-   `--no-usage`: تخطي لقطات الاستخدام/الحصة لمزود النموذج (OAuth/API فقط).
-   `--json`: إخراج JSON (يشمل الاستخدام ما لم يتم تعيين `--no-usage`).

خيارات `channels logs`:

-   `--channel <name|all>` (الافتراضي `all`)
-   `--lines ` (الافتراضي `200`)
-   `--json`

مزيد من التفاصيل: [/concepts/oauth](./concepts/oauth.md) أمثلة:

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

### skills

سرد وفحص المهارات المتاحة بالإضافة إلى معلومات الجاهزية. الأوامر الفرعية:

-   `skills list`: سرد المهارات (الافتراضي عند عدم وجود أمر فرعي).
-   `skills info `: عرض تفاصيل مهارة واحدة.
-   `skills check`: ملخص للمهارات الجاهزة مقابل المتطلبات المفقودة.

الخيارات:

-   `--eligible`: عرض المهارات الجاهزة فقط.
-   `--json`: إخراج JSON (بدون تنسيق).
-   `-v`, `--verbose`: تضمين تفاصيل المتطلبات المفقودة.

تلميح: استخدم `npx clawhub` للبحث، التثبيت، ومزامنة المهارات.

### pairing

الموافقة على طلبات الاقتران عبر الرسائل المباشرة عبر القنوات. الأوامر الفرعية:

-   `pairing list [channel] [--channel ] [--account ] [--json]`
-   `pairing approve   [--account ] [--notify]`
-   `pairing approve --channel  [--account ]  [--notify]`

### devices

إدارة إدخالات اقتران أجهزة البوابة ورموز الجهاز لكل دور. الأوامر الفرعية:

-   `devices list [--json]`
-   `devices approve [requestId] [--latest]`
-   `devices reject `
-   `devices remove `
-   `devices clear --yes [--pending]`
-   `devices rotate --device  --role  [--scope <scope...>]`
-   `devices revoke --device  --role `

### webhooks gmail

إعداد خطاف Gmail Pub/Sub + عداء. انظر [/automation/gmail-pubsub](./automation/gmail-pubsub.md). الأوامر الفرعية:

-   `webhooks gmail setup` (يتطلب `--account `؛ يدعم `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
-   `webhooks gmail run` (تجاوزات وقت التشغيل لنفس الأعلام)

### dns setup

مساعد DNS لاكتشاف المنطقة الواسعة (CoreDNS + Tailscale). انظر [/gateway/discovery](./gateway/discovery.md). الخيارات:

-   `--apply`: تثبيت/تحديث تكوين CoreDNS (يتطلب sudo؛ macOS فقط).

## المراسلة + الوكيل

### message

مراسلة صادرة موحدة + إجراءات القناة. انظر: [/cli/message](./cli/message.md) الأوامر الفرعية:

-   `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
-   `message thread <create|list|reply>`
-   `message emoji <list|upload>`
-   `message sticker <send|upload>`
-   `message role <info|add|remove>`
-   `message channel <info|list>`
-   `message member info`
-   `message voice status`
-   `message event <list|create>`

أمثلة:

-   `openclaw message send --target +15555550123 --message "مرحبًا"`
-   `openclaw message poll --channel discord --target channel:123 --poll-question "وجبة خفيفة؟" --poll-option بيتزا --poll-option سوشي`

### agent

تشغيل دورة وكيل واحدة عبر البوابة (أو `--local` مضمن). مطلوب:

-   `--message `

الخيارات:

-   `--to ` (لمفتاح الجلسة والتسليم الاختياري)
-   `--session-id `
-   `--thinking <off|minimal|low|medium|high|xhigh>` (نماذج GPT-5.2 + Codex فقط)
-   `--verbose <on|full|off>`
-   `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
-   `--local`
-   `--deliver`
-   `--json`
-   `--timeout `

### agents

إدارة الوكلاء المعزولين (مساحات العمل + المصادقة + التوجيه).

#### agents list

سرد الوكلاء المكونين. الخيارات:

-   `--json`
-   `--bindings`

#### agents add \[name\]

إضافة وكيل معزول جديد. يقوم بتشغيل المعالج الموجه ما لم يتم تمرير أعلام (أو `--non-interactive`)؛ `--workspace` مطلوب في الوضع غير التفاعلي. الخيارات:

-   `--workspace `
-   `--model `
-   `--agent-dir `
-   `--bind <channel[:accountId]>` (قابل للتكرار)
-   `--non-interactive`
-   `--json`

مواصفات الربط تستخدم `channel[:accountId]`. عند حذف `accountId`، قد يحل OpenClaw نطاق الحساب عبر الإعدادات الافتراضية/خطافات الإضافة للقناة؛ وإلا فهو ربط قناة بدون نطاق حساب صريح.

#### agents bindings

سرد عمليات الربط. الخيارات:

-   `--agent `
-   `--json`

#### agents bind

إضافة عمليات ربط توجيه لوكيل. الخيارات:

-   `--agent `
-   `--bind <channel[:accountId]>` (قابل للتكرار)
-   `--json`

#### agents unbind

إزالة عمليات ربط توجيه لوكيل. الخيارات:

-   `--agent `
-   `--bind <channel[:accountId]>` (قابل للتكرار)
-   `--all`
-   `--json`

#### agents delete &lt;id&gt;

حذف وكيل وتقليم مساحة عمله + حالته. الخيارات:

-   `--force`
-   `--json`

### acp

تشغيل جسر ACP الذي يربط بيئات التطوير المتكاملة بالبوابة. انظر [`acp`](./cli/acp.md) للحصول على الخيارات والأمثلة الكاملة.

### status

عرض صحة الجلسة المرتبطة والمستلمين الحديثين. الخيارات:

-   `--json`
-   `--all` (تشخيص كامل؛ للقراءة فقط، قابل للنسخ)
-   `--deep` (فحص القنوات)
-   `--usage` (عرض استخدام/حصة مزود النموذج)
-   `--timeout `
-   `--verbose`
-   `--debug` (اسم مستعار لـ `--verbose`)

ملاحظات:

-   النظرة العامة تشمل حالة خدمة مضيف البوابة + العقدة عند توفرها.

### تتبع الاستخدام

يمكن لـ OpenClaw عرض استخدام/حصة المزود عندما تكون بيانات اعتماد OAuth/API متاحة. يعرض:

-   `/status` (يضيف سطر استخدام موجز للمزود عند التوفر)
-   `openclaw status --usage` (يطبع تفصيل كامل للمزود)
-   شريط قائمة macOS (قسم الاستخدام تحت السياق)

ملاحظات:

-   تأتي البيانات مباشرة من نقاط نهاية استخدام المزود (لا توجد تقديرات).
-   المزودون: Anthropic، GitHub Copilot، OpenAI Codex OAuth، بالإضافة إلى Gemini CLI/Antigravity عند تمكين إضافات تلك المزودين.
-   إذا لم توجد بيانات اعتماد مطابقة، يتم إخفاء الاستخدام.
-   التفاصيل: انظر [تتبع الاستخدام](./concepts/usage-tracking.md).

### health

جلب الحالة من البوابة قيد التشغيل. الخيارات:

-   `--json`
-   `--timeout `
-   `--verbose`

### sessions

سرد جلسات المحادثة المخزنة. الخيارات:

-   `--json`
-   `--verbose`
-   `--store `
-   `--active `

## إعادة التعيين / إلغاء التثبيت

### reset

إعادة تعيين التكوين/الحالة المحلية (يبقى CLI مثبتًا). الخيارات:

-   `--scope <config|config+creds+sessions|full>`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

ملاحظات:

-   `--non-interactive` يتطلب `--scope` و `--yes`.

### uninstall

إلغاء تثبيت خدمة البوابة + البيانات المحلية (يبقى CLI). الخيارات:

-   `--service`
-   `--state`
-   `--workspace`
-   `--app`
-   `--all`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

ملاحظات:

-   `--non-interactive` يتطلب `--yes` ونطاقات صريحة (أو `--all`).

## البوابة

### gateway

تشغيل بوابة WebSocket. الخيارات:

-   `--port `
-   `--bind <loopback|tailnet|lan|auto|custom>`
-   `--token `
-   `--auth <token|password>`
-   `--password `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--allow-unconfigured`
-   `--dev`
-   `--reset` (إعادة تعيين تكوين dev + بيانات الاعتماد + الجلسات + مساحة العمل)
-   `--force` (قتل المستمع الحالي على المنفذ)
-   `--verbose`
-   `--claude-cli-logs`
-   `--ws-log <auto|full|compact>`
-   `--compact` (اسم مستعار لـ `--ws-log compact`)
-   `--raw-stream`
-   `--raw-stream-path `

### gateway service

إدارة خدمة البوابة (launchd/systemd/schtasks). الأوامر الفرعية:

-   `gateway status` (يفحص RPC البوابة بشكل افتراضي)
-   `gateway install` (تثبيت الخدمة)
-   `gateway uninstall`
-   `gateway start`
-   `gateway stop`
-   `gateway restart`

ملاحظات:

-   `gateway status` يفحص RPC البوابة بشكل افتراضي باستخدام المنفذ/التكوين المحلول للخدمة (تجاوز باستخدام `--url/--token/--password`).
-   `gateway status` يدعم `--no-probe`, `--deep`, و `--json` للبرمجة النصية.
-   `gateway status` يعرض أيضًا خدمات البوابة القديمة أو الإضافية عندما يمكنه اكتشافها (`--deep` يضيف فحوصات على مستوى النظام). خدمات OpenClaw المسماة بالملف الشخصي تعامل كخدمات من الدرجة الأولى ولا يتم وضع علامة عليها كـ "إضافية".
-   `gateway status` يطبع مسار التكوين الذي يستخدمه CLI مقابل التكوين الذي تستخدمه الخدمة على الأرجح (بيئة الخدمة)، بالإضافة إلى عنوان URL المستهدف للفحص المحلول.
-   `gateway install|uninstall|start|stop|restart` تدعم `--json` للبرمجة النصية (يبقى الإخراج الافتراضي ملائمًا للبشر).
-   `gateway install` الافتراضي هو وقت تشغيل Node؛ **لا يوصى بـ** bun (أخطاء WhatsApp/Telegram).
-   خيارات `gateway install`: `--port`, `--runtime`, `--token`, `--force`, `--json`.

### logs

تتبع سجلات ملف البوابة عبر RPC. ملاحظات:

-   جلسات TTY تعرض عرضًا منظمًا ملونًا؛ غير TTY يعود إلى نص عادي.
-   `--json` يصدر JSON مفصول بأسطر (حدث سجل واحد