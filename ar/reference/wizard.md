

  مرجع تقني

  
# مرجع معالج الإعداد

هذا هو المرجع الكامل لمعالج `openclaw onboard` عبر سطر الأوامر. للحصول على نظرة عامة عالية المستوى، راجع [معالج الإعداد](../start/wizard.md).

## تفاصيل سير العمل (الوضع المحلي)

### الخطوة 1: اكتشاف الإعدادات الحالية

-   إذا كان الملف `~/.openclaw/openclaw.json` موجودًا، اختر **الاحتفاظ / التعديل / إعادة التعيين**.
-   إعادة تشغيل المعالج **لا** تمسح أي شيء إلا إذا اخترت **إعادة التعيين** صراحةً (أو مررت `--reset`).
-   `--reset` في سطر الأوامر يطبق افتراضيًا على `config+creds+sessions`؛ استخدم `--reset-scope full` لإزالة مساحة العمل أيضًا.
-   إذا كان الإعداد غير صالح أو يحتوي على مفاتيح قديمة، يتوقف المعالج ويطلب منك تشغيل `openclaw doctor` قبل المتابعة.
-   يستخدم إعادة التعيين `trash` (وليس `rm` أبدًا) ويعرض نطاقات:
    -   الإعدادات فقط
    -   الإعدادات + بيانات الاعتماد + الجلسات
    -   إعادة التعيين الكاملة (تزيل مساحة العمل أيضًا)

### الخطوة 2: النموذج/المصادقة

-   **مفتاح Anthropic API**: يستخدم `ANTHROPIC_API_KEY` إذا كان موجودًا أو يطلب مفتاحًا، ثم يحفظه لاستخدام الخلفية.
-   **Anthropic OAuth (Claude Code CLI)**: على نظام macOS، يتحقق المعالج من عنصر Keychain "Claude Code-credentials" (اختر "Always Allow" حتى لا تمنع عمليات بدء launchd)؛ على Linux/Windows يعيد استخدام `~/.claude/.credentials.json` إذا كان موجودًا.
-   **رمز Anthropic (لصق setup-token)**: شغل `claude setup-token` على أي جهاز، ثم الصق الرمز (يمكنك تسميته؛ فارغ = افتراضي).
-   **اشتراك OpenAI Code (Codex) (Codex CLI)**: إذا كان `~/.codex/auth.json` موجودًا، يمكن للمعالج إعادة استخدامه.
-   **اشتراك OpenAI Code (Codex) (OAuth)**: تدفق عبر المتصفح؛ الصق `code#state`.
    -   يضبط `agents.defaults.model` إلى `openai-codex/gpt-5.2` عندما يكون النموذج غير مضبوط أو `openai/*`.
-   **مفتاح OpenAI API**: يستخدم `OPENAI_API_KEY` إذا كان موجودًا أو يطلب مفتاحًا، ثم يخزنه في ملفات تعريف المصادقة.
-   **مفتاح xAI (Grok) API**: يطلب `XAI_API_KEY` ويهيئ xAI كمزود نموذج.
-   **OpenCode Zen (وكيل متعدد النماذج)**: يطلب `OPENCODE_API_KEY` (أو `OPENCODE_ZEN_API_KEY`، احصل عليه من [https://opencode.ai/auth](https://opencode.ai/auth)).
-   **مفتاح API**: يخزن المفتاح لك.
-   **Vercel AI Gateway (وكيل متعدد النماذج)**: يطلب `AI_GATEWAY_API_KEY`.
-   مزيد من التفاصيل: [Vercel AI Gateway](../providers/vercel-ai-gateway.md)
-   **Cloudflare AI Gateway**: يطلب معرف الحساب، ومعرف البوابة، و `CLOUDFLARE_AI_GATEWAY_API_KEY`.
-   مزيد من التفاصيل: [Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md)
-   **MiniMax M2.5**: يتم كتابة الإعداد تلقائيًا.
-   مزيد من التفاصيل: [MiniMax](../providers/minimax.md)
-   **Synthetic (متوافق مع Anthropic)**: يطلب `SYNTHETIC_API_KEY`.
-   مزيد من التفاصيل: [Synthetic](../providers/synthetic.md)
-   **Moonshot (Kimi K2)**: يتم كتابة الإعداد تلقائيًا.
-   **Kimi Coding**: يتم كتابة الإعداد تلقائيًا.
-   مزيد من التفاصيل: [Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md)
-   **تخطي**: لا يتم تكوين مصادقة بعد.
-   اختر نموذجًا افتراضيًا من الخيارات المكتشفة (أو أدخل مزود/نموذج يدويًا). للحصول على أفضل جودة وتقليل مخاطر حقن الأوامر، اختر أقوى نموذج من الجيل الأحدث المتاح في مكدس المزود الخاص بك.
-   يقوم المعالج بفحص النموذج وينذر إذا كان النموذج المضبوط غير معروف أو يفتقر إلى مصادقة.
-   وضع تخزين مفتاح API الافتراضي هو قيم ملف تعريف المصادقة كنص عادي. استخدم `--secret-input-mode ref` لتخزين مراجع مدعومة بالبيئة بدلاً من ذلك (على سبيل المثال `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`).
-   بيانات اعتماد OAuth موجودة في `~/.openclaw/credentials/oauth.json`؛ ملفات تعريف المصادقة موجودة في `~/.openclaw/agents//agent/auth-profiles.json` (مفاتيح API + OAuth).
-   مزيد من التفاصيل: [/concepts/oauth](../concepts/oauth.md)

> **ℹ️** نصيحة للخوادم/بدون واجهة: أكمل OAuth على جهاز به متصفح، ثم انسخ `~/.openclaw/credentials/oauth.json` (أو `$OPENCLAW_STATE_DIR/credentials/oauth.json`) إلى مضيف البوابة.

### الخطوة 3: مساحة العمل

-   افتراضيًا `~/.openclaw/workspace` (قابل للتعديل).
-   يجهز ملفات مساحة العمل اللازمة لطقوس بدء تشغيل الوكيل.
-   تخطيط مساحة العمل الكامل + دليل النسخ الاحتياطي: [مساحة عمل الوكيل](../concepts/agent-workspace.md)

### الخطوة 4: البوابة

-   المنفذ، الربط، وضع المصادقة، التعريض عبر Tailscale.
-   توصية المصادقة: احتفظ بـ **الرمز** حتى للربط المحلي حتى يجب على عملاء WS المحليين المصادقة.
-   في وضع الرمز، يعرض الإعداد التفاعلي:
    -   **إنشاء/تخزين رمز نص عادي** (افتراضي)
    -   **استخدام SecretRef** (اختياري)
    -   يعيد Quickstart استخدام SecretRefs الحالية لـ `gateway.auth.token` عبر موفري `env` و `file` و `exec` لبدء تشغيل المسبار/لوحة التحكم.
    -   إذا تم تكوين هذا SecretRef ولكن لا يمكن حله، يفشل الإعداد مبكرًا برسالة إصلاح واضحة بدلاً من التدهور الصامت لمصادقة وقت التشغيل.
-   في وضع كلمة المرور، يدعم الإعداد التفاعلي أيضًا تخزين النص العادي أو SecretRef.
-   مسار SecretRef للرمز في الوضع غير التفاعلي: `--gateway-token-ref-env <ENV_VAR>`.
    -   يتطلب متغير بيئة غير فارغ في بيئة عملية الإعداد.
    -   لا يمكن دمجه مع `--gateway-token`.
-   عطل المصادقة فقط إذا كنت تثق تمامًا بكل عملية محلية.
-   عمليات الربط غير المحلية لا تزال تتطلب مصادقة.

### الخطوة 5: القنوات

-   [WhatsApp](../channels/whatsapp.md): تسجيل دخول اختياري برمز QR.
-   [Telegram](../channels/telegram.md): رمز بوت.
-   [Discord](../channels/discord.md): رمز بوت.
-   [Google Chat](../channels/googlechat.md): حساب خدمة JSON + جمهور webhook.
-   [Mattermost](../channels/mattermost.md) (إضافة): رمز بوت + عنوان URL أساسي.
-   [Signal](../channels/signal.md): تثبيت اختياري لـ `signal-cli` + تكوين الحساب.
-   [BlueBubbles](../channels/bluebubbles.md): **موصى به لـ iMessage**؛ عنوان URL للخادم + كلمة المرور + webhook.
-   [iMessage](../channels/imessage.md): مسار `imsg` CLI قديم + وصول إلى قاعدة البيانات.
-   أمان المراسلة المباشرة: الافتراضي هو الاقتران. ترسل أول رسالة مباشرة رمزًا؛ وافق عبر `openclaw pairing approve  ` أو استخدم قوائم السماح.

### الخطوة 6: البحث على الويب

-   اختر مزودًا: Perplexity أو Brave أو Gemini أو Grok أو Kimi (أو تخطى).
-   الصق مفتاح API الخاص بك (يكتشف QuickStart المفاتيح تلقائيًا من متغيرات البيئة أو الإعدادات الحالية).
-   تخطى باستخدام `--skip-search`.
-   اضبط لاحقًا: `openclaw configure --section web`.

### الخطوة 7: تثبيت الخلفية

-   macOS: LaunchAgent
    -   يتطلب جلسة مستخدم مسجل الدخول؛ للخوادم بدون واجهة، استخدم LaunchDaemon مخصصًا (غير مشمول).
-   Linux (و Windows عبر WSL2): وحدة systemd للمستخدم
    -   يحاول المعالج تمكين التمهل عبر `loginctl enable-linger ` حتى تبقى البوابة قيد التشغيل بعد تسجيل الخروج.
    -   قد يطلب صلاحيات sudo (يكتب في `/var/lib/systemd/linger`)؛ يحاول أولاً بدون sudo.
-   **اختيار وقت التشغيل:** Node (موصى به؛ مطلوب لـ WhatsApp/Telegram). Bun **غير موصى به**.
-   إذا تطلبت مصادقة الرمز رمزًا وكان `gateway.auth.token` مُدارًا عبر SecretRef، فإن تثبيت الخلفية يتحقق منه ولكنه لا يحفظ قيم الرمز النصية العادية المحلولة في بيانات وصفية لبيئة خدمة المشرف.
-   إذا تطلبت مصادقة الرمز رمزًا وكان SecretRef المضبوط غير محلول، يتم منع تثبيت الخلفية بتوجيهات قابلة للتنفيذ.
-   إذا تم تكوين كل من `gateway.auth.token` و `gateway.auth.password` ولم يتم ضبط `gateway.auth.mode`، يتم منع تثبيت الخلفية حتى يتم ضبط الوضع صراحةً.

### الخطوة 8: فحص الصحة

-   يبدأ تشغيل البوابة (إذا لزم الأمر) ويشغل `openclaw health`.
-   نصيحة: `openclaw status --deep` يضيف فحوصات صحة البوابة إلى إخراج الحالة (يتطلب بوابة يمكن الوصول إليها).

### الخطوة 9: المهارات (موصى بها)

-   يقرأ المهارات المتاحة ويتحقق من المتطلبات.
-   يتيح لك اختيار مدير العقد: **npm / pnpm** (bun غير موصى به).
-   يثبت التبعيات الاختيارية (يستخدم بعضها Homebrew على macOS).

### الخطوة 10: الانتهاء

-   ملخص + الخطوات التالية، بما في ذلك تطبيقات iOS/Android/macOS لميزات إضافية.

 

> **ℹ️** إذا لم يتم اكتشاف واجهة مستخدم رسومية، يطبع المعالج تعليمات توجيه منفذ SSH لواجهة التحكم بدلاً من فتح متصفح. إذا كانت أصول واجهة التحكم مفقودة، يحاول المعالج بنائها؛ الاحتياطي هو `pnpm ui:build` (يثبت تبعيات واجهة المستخدم تلقائيًا).

## الوضع غير التفاعلي

استخدم `--non-interactive` لأتمتة أو كتابة سيناريو للإعداد:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

أضف `--json` للحصول على ملخص قابل للقراءة آليًا. SecretRef لرمز البوابة في الوضع غير التفاعلي:

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN
```

`--gateway-token` و `--gateway-token-ref-env` متنافيان. 

> **ℹ️** `--json` **لا** تعني الوضع غير التفاعلي. استخدم `--non-interactive` (و `--workspace`) للسيناريوهات.

 

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

### إضافة وكيل (غير تفاعلي)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## معالج البوابة عبر RPC

تعرض البوابة سير عمل المعالج عبر RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`). يمكن للعملاء (تطبيق macOS، واجهة التحكم) عرض الخطوات دون إعادة تنفيذ منطق الإعداد.

## إعداد Signal (signal-cli)

يمكن للمعالج تثبيت `signal-cli` من إصدارات GitHub:

-   يقوم بتنزيل أصول الإصدار المناسبة.
-   يخزنها تحت `~/.openclaw/tools/signal-cli//`.
-   يكتب `channels.signal.cliPath` في إعداداتك.

ملاحظات:

-   تتطلب إصدارات JVM **Java 21**.
-   تستخدم الإصدارات الأصلية عند توفرها.
-   يستخدم Windows نظام WSL2؛ يتبع تثبيت signal-cli سير عمل Linux داخل WSL.

## ما يكتبه المعالج

الحقول النموذجية في `~/.openclaw/openclaw.json`:

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers` (إذا تم اختيار Minimax)
-   `tools.profile` (يضبط الإعداد المحلي الافتراضي على `"coding"` عندما يكون غير مضبوط؛ يتم الاحتفاظ بالقيم الصريحة الحالية)
-   `gateway.*` (الوضع، الربط، المصادقة، tailscale)
-   `session.dmScope` (تفاصيل السلوك: [مرجع إعداد سطر الأوامر](../start/wizard-cli-reference.md#outputs-and-internals))
-   `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
-   قوائم السماح للقنوات (Slack/Discord/Matrix/Microsoft Teams) عندما توافق أثناء المطالبات (تتحول الأسماء إلى معرفات عندما يكون ذلك ممكنًا).
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

يكتب `openclaw agents add` في `agents.list[]` و `bindings` الاختيارية. تذهب بيانات اعتماد WhatsApp تحت `~/.openclaw/credentials/whatsapp//`. يتم تخزين الجلسات تحت `~/.openclaw/agents//sessions/`. يتم تسليم بعض القنوات كإضافات. عندما تختار واحدة أثناء الإعداد، سيطالبك المعالج بتثبيتها (npm أو مسار محلي) قبل أن يمكن تكوينها.

## وثائق ذات صلة

-   نظرة عامة على المعالج: [معالج الإعداد](../start/wizard.md)
-   إعداد تطبيق macOS: [الإعداد](../start/onboarding.md)
-   مرجع الإعدادات: [تهيئة البوابة](../gateway/configuration.md)
-   المزودون: [WhatsApp](../channels/whatsapp.md), [Telegram](../channels/telegram.md), [Discord](../channels/discord.md), [Google Chat](../channels/googlechat.md), [Signal](../channels/signal.md), [BlueBubbles](../channels/bluebubbles.md) (iMessage), [iMessage](../channels/imessage.md) (قديم)
-   المهارات: [المهارات](../tools/skills.md), [إعداد المهارات](../tools/skills-config.md)

[USER](./templates/USER.md)[استخدام الرمز والتكاليف](./token-use.md)

---