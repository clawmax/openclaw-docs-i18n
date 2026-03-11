

  التكوين والعمليات

  
# التكوين

يقرأ OpenClaw ملف تكوين **JSON5** اختياريًا من `~/.openclaw/openclaw.json`. إذا كان الملف مفقودًا، يستخدم OpenClaw إعدادات افتراضية آمنة. الأسباب الشائعة لإضافة تكوين:

-   توصيل القنوات والتحكم في من يمكنه مراسلة البوت
-   تعيين النماذج، الأدوات، العزل، أو الأتمتة (وظائف مجدولة، خطافات)
-   ضبط الجلسات، الوسائط، الشبكات، أو واجهة المستخدم

راجع [المرجع الكامل](./configuration-reference.md) لكل حقل متاح.

> **💡** **جديد في التكوين؟** ابدأ بـ `openclaw onboard` للإعداد التفاعلي، أو تحقق من دليل [أمثلة التكوين](./configuration-examples.md) للحصول على تكوينات كاملة قابلة للنسخ واللصق.

## التكوين الأدنى

```
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## تحرير التكوين

```bash
openclaw onboard       # معالج الإعداد الكامل
openclaw configure     # معالج التكوين
```

```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```

افتح [http://127.0.0.1:18789](http://127.0.0.1:18789) واستخدم علامة التبويب **Config**. تعرض واجهة التحكم نموذجًا من مخطط التكوين، مع محرر **Raw JSON** كملاذ أخير.

قم بتحرير `~/.openclaw/openclaw.json` مباشرة. تراقب البوابة الملف وتطبق التغييرات تلقائيًا (انظر [إعادة التحميل الساخن](#config-hot-reload)).

## التحقق الصارم

> **⚠️** يقبل OpenClaw فقط التكوينات التي تطابق المخطط بالكامل. المفاتيح غير المعروفة، أو الأنواع غير الصحيحة، أو القيم غير الصالحة تتسبب في **رفض البوابة البدء**. الاستثناء الوحيد على مستوى الجذر هو `$schema` (سلسلة نصية)، حتى يتمكن المحررون من إرفاق بيانات تعريف مخطط JSON.

 عند فشل التحقق:

-   لا يتم تشغيل البوابة
-   تعمل أوامر التشخيص فقط (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
-   قم بتشغيل `openclaw doctor` لرؤية المشكلات بالضبط
-   قم بتشغيل `openclaw doctor --fix` (أو `--yes`) لتطبيق الإصلاحات

## المهام الشائعة

لكل قناة قسم تكوين خاص بها تحت `channels.`. راجع صفحة القناة المخصصة لخطوات الإعداد:

-   [WhatsApp](../channels/whatsapp.md) — `channels.whatsapp`
-   [Telegram](../channels/telegram.md) — `channels.telegram`
-   [Discord](../channels/discord.md) — `channels.discord`
-   [Slack](../channels/slack.md) — `channels.slack`
-   [Signal](../channels/signal.md) — `channels.signal`
-   [iMessage](../channels/imessage.md) — `channels.imessage`
-   [Google Chat](../channels/googlechat.md) — `channels.googlechat`
-   [Mattermost](../channels/mattermost.md) — `channels.mattermost`
-   [MS Teams](../channels/msteams.md) — `channels.msteams`

تشارك جميع القنوات نفس نمط سياسة الرسائل المباشرة:

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",   // pairing | allowlist | open | disabled
      allowFrom: ["tg:123"], // only for allowlist/open
    },
  },
}
```

قم بتعيين النموذج الأساسي والنماذج الاحتياطية الاختيارية:

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5.2"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "openai/gpt-5.2": { alias: "GPT" },
      },
    },
  },
}
```

-   `agents.defaults.models` يحدد كتالوج النماذج ويعمل كقائمة السماح لـ `/model`.
-   تستخدم مراجع النماذج تنسيق `provider/model` (مثال: `anthropic/claude-opus-4-6`).
-   `agents.defaults.imageMaxDimensionPx` يتحكم في تقليص حجم الصور في النصوص والأدوات (الافتراضي `1200`)؛ تقلل القيم الأصغر عادةً من استخدام الرموز البصرية في عمليات التشغيل الغنية بلقطات الشاشة.
-   راجع [واجهة سطر أوامر النماذج](../concepts/models.md) لتبديل النماذج في الدردشة و [النموذج الاحتياطي](../concepts/model-failover.md) لتدوير المصادقة وسلوك الاحتياطي.
-   لمزودي مخصصين/مستضافين ذاتيًا، راجع [مزودون مخصصون](./configuration-reference.md#custom-providers-and-base-urls) في المرجع.

يتم التحكم في الوصول للرسائل المباشرة لكل قناة عبر `dmPolicy`:

-   `"pairing"` (الافتراضي): يحصل المرسلون غير المعروفين على رمز اقتران لمرة واحدة للموافقة
-   `"allowlist"`: فقط المرسلون الموجودون في `allowFrom` (أو مخزن السماح المقترن)
-   `"open"`: السماح بجميع الرسائل المباشرة الواردة (يتطلب `allowFrom: ["*"]`)
-   `"disabled"`: تجاهل جميع الرسائل المباشرة

للمجموعات، استخدم `groupPolicy` + `groupAllowFrom` أو قوائم السماح الخاصة بالقناة. راجع [المرجع الكامل](./configuration-reference.md#dm-and-group-access) للتفاصيل لكل قناة.

الرسائل الجماعية افتراضيًا **تتطلب الإشارة**. قم بتكوين الأنماط لكل وكيل:

```json
{
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw"],
        },
      },
    ],
  },
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

-   **إشارات البيانات الوصفية**: إشارات @-الأصلية (النقر للإشارة في WhatsApp، @bot في Telegram، إلخ)
-   **أنماط النص**: أنماط regex في `mentionPatterns`
-   راجع [المرجع الكامل](./configuration-reference.md#group-chat-mention-gating) لتجاوزات كل قناة ووضع الدردشة الذاتية.

تتحكم الجلسات في استمرارية المحادثة وعزلها:

```json
{
  session: {
    dmScope: "per-channel-peer",  // موصى به لمتعدد المستخدمين
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
  },
}
```

-   `dmScope`: `main` (مشترك) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
-   `threadBindings`: الإعدادات الافتراضية العالمية للتوجيه المرتبط بالخيط (يدعم Discord `/focus`, `/unfocus`, `/agents`, `/session idle`, و `/session max-age`).
-   راجع [إدارة الجلسات](../concepts/session.md) للتحديد، روابط الهوية، وسياسة الإرسال.
-   راجع [المرجع الكامل](./configuration-reference.md#session) لجميع الحقول.

قم بتشغيل جلسات الوكيل في حاويات Docker معزولة:

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",  // off | non-main | all
        scope: "agent",    // session | agent | shared
      },
    },
  },
}
```

قم ببناء الصورة أولاً: `scripts/sandbox-setup.sh` راجع [العزل](./sandboxing.md) للحصول على الدليل الكامل و [المرجع الكامل](./configuration-reference.md#sandbox) لجميع الخيارات.

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
  },
}
```

-   `every`: سلسلة مدة (`30m`, `2h`). اضبط `0m` لتعطيل.
-   `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
-   `directPolicy`: `allow` (الافتراضي) أو `block` لأهداف نبضات القلب من نوع الرسائل المباشرة
-   راجع [نبضات القلب](./heartbeat.md) للحصول على الدليل الكامل.

```json
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h",
    runLog: {
      maxBytes: "2mb",
      keepLines: 2000,
    },
  },
}
```

-   `sessionRetention`: تقليم جلسات التشغيل المعزولة المكتملة من `sessions.json` (الافتراضي `24h`؛ اضبط `false` لتعطيل).
-   `runLog`: تقليم `cron/runs/.jsonl` حسب الحجم والخطوط المحتفظ بها.
-   راجع [الوظائف المجدولة](../automation/cron-jobs.md) للحصول على نظرة عامة على الميزة وأمثلة واجهة سطر الأوامر.

قم بتمكين نقاط نهاية خطافات HTTP على البوابة:

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "main",
        deliver: true,
      },
    ],
  },
}
```

ملاحظة أمنية:

-   عالج كل محتوى حمولة الخطاف/الخطاف الويب كمدخلات غير موثوقة.
-   حافظ على تعطيل أعلام تجاوز المحتوى غير الآمن (`hooks.gmail.allowUnsafeExternalContent`, `hooks.mappings[].allowUnsafeExternalContent`) ما لم تكن تقوم بتصحيح أخطاء محدودة النطاق.
-   لوكلاء مدفوعين بالخطافات، فضل مستويات النماذج الحديثة القوية وسياسة الأدوات الصارمة (على سبيل المثال المراسلة فقط بالإضافة إلى العزل حيثما أمكن).

راجع [المرجع الكامل](./configuration-reference.md#hooks) لجميع خيارات التعيين وتكامل Gmail.

قم بتشغيل وكلاء متعددين معزولين بمساحات عمل وجلسات منفصلة:

```json
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

راجع [متعدد الوكلاء](../concepts/multi-agent.md) و [المرجع الكامل](./configuration-reference.md#multi-agent-routing) لقواعد الربط وملفات تعريف الوصول لكل وكيل.

استخدم `$include` لتنظيم التكوينات الكبيرة:

```
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/a.json5", "./clients/b.json5"],
  },
}
```

-   **ملف واحد**: يستبدل الكائن الحاوي
-   **مصفوفة من الملفات**: يتم دمجها بعمق بالترتيب (الأخير يفوز)
-   **المفاتيح الشقيقة**: يتم دمجها بعد التضمينات (تجاوز القيم المضمنة)
-   **تضمينات متداخلة**: مدعومة حتى 10 مستويات عمق
-   **المسارات النسبية**: يتم حلها بالنسبة للملف المضمن
-   **معالجة الأخطاء**: أخطاء واضحة للملفات المفقودة، أخطاء التحليل، والتضمينات الدائرية

## إعادة التحميل الساخن للتكوين

تراقب البوابة ملف `~/.openclaw/openclaw.json` وتطبق التغييرات تلقائيًا — لا حاجة لإعادة تشغيل يدوية لمعظم الإعدادات.

### أوضاع إعادة التحميل

| الوضع | السلوك |
| --- | --- |
| **`hybrid`** (الافتراضي) | يطبق التغييرات الآمنة على الفور. يعيد التشغيل تلقائيًا للتغييرات الحرجة. |
| **`hot`** | يطبق التغييرات الآمنة فقط. يسجل تحذيرًا عندما تكون هناك حاجة لإعادة تشغيل — عليك التعامل معها. |
| **`restart`** | يعيد تشغيل البوابة عند أي تغيير في التكوين، آمنًا كان أم لا. |
| **`off`** | يعطل مراقبة الملف. التغييرات سارية المفعول عند إعادة التشغيل اليدوية التالية. |

```json
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### ما الذي يتم تطبيقه ساخنًا مقابل ما يحتاج إلى إعادة تشغيل

معظم الحقول تطبق ساخنًا دون توقف. في وضع `hybrid`، يتم التعامل مع التغييرات التي تتطلب إعادة تشغيل تلقائيًا.

| الفئة | الحقول | هل تحتاج إلى إعادة تشغيل؟ |
| --- | --- | --- |
| القنوات | `channels.*`, `web` (WhatsApp) — جميع القنوات المدمجة وامتدادات القنوات | لا |
| الوكيل والنماذج | `agent`, `agents`, `models`, `routing` | لا |
| الأتمتة | `hooks`, `cron`, `agent.heartbeat` | لا |
| الجلسات والرسائل | `session`, `messages` | لا |
| الأدوات والوسائط | `tools`, `browser`, `skills`, `audio`, `talk` | لا |
| واجهة المستخدم ومتنوعات | `ui`, `logging`, `identity`, `bindings` | لا |
| خادم البوابة | `gateway.*` (المنفذ، الربط، المصادقة، tailscale، TLS، HTTP) | **نعم** |
| البنية التحتية | `discovery`, `canvasHost`, `plugins` | **نعم** |

> **ℹ️** `gateway.reload` و `gateway.remote` استثناءات — تغييرها **لا** يؤدي إلى إعادة تشغيل.

## تكوين RPC (تحديثات برمجية)

> **ℹ️** عمليات كتابة RPC لمستوى التحكم (`config.apply`, `config.patch`, `update.run`) محدودة بمعدل **3 طلبات لكل 60 ثانية** لكل `deviceId+clientIp`. عند التحديد، ترجع RPC `UNAVAILABLE` مع `retryAfterMs`.

 

يتحقق + يكتب التكوين الكامل ويعيد تشغيل البوابة في خطوة واحدة.

يستبدل `config.apply` **التكوين بالكامل**. استخدم `config.patch` للتحديثات الجزئية، أو `openclaw config set` للمفاتيح المفردة.

المعلمات:

-   `raw` (سلسلة نصية) — حمولة JSON5 للتكوين بأكمله
-   `baseHash` (اختياري) — تجزئة التكوين من `config.get` (مطلوب عندما يكون التكوين موجودًا)
-   `sessionKey` (اختياري) — مفتاح الجلسة لتنبيه الاستيقاظ بعد إعادة التشغيل
-   `note` (اختياري) — ملاحظة لحارس إعادة التشغيل
-   `restartDelayMs` (اختياري) — تأخير قبل إعادة التشغيل (الافتراضي 2000)

يتم دمج طلبات إعادة التشغيل أثناء وجود طلب معلق/قيد التنفيذ بالفعل، ويتم تطبيق فترة تهدئة مدتها 30 ثانية بين دورات إعادة التشغيل.

```bash
openclaw gateway call config.get --params '{}'  # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
  "baseHash": "<hash>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123"
}'
```

يدمج تحديثًا جزئيًا في التكوين الحالي (دلالات رقعة دمج JSON):

-   يتم دمج الكائنات بشكل متكرر
-   `null` يحذف مفتاحًا
-   المصفوفات تستبدل

المعلمات:

-   `raw` (سلسلة نصية) — JSON5 يحتوي فقط على المفاتيح المراد تغييرها
-   `baseHash` (مطلوب) — تجزئة التكوين من `config.get`
-   `sessionKey`, `note`, `restartDelayMs` — نفس `config.apply`

سلوك إعادة التشغيل يطابق `config.apply`: دمج عمليات إعادة التشغيل المعلقة بالإضافة إلى فترة تهدئة مدتها 30 ثانية بين دورات إعادة التشغيل.

```bash
openclaw gateway call config.patch --params '{
  "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
  "baseHash": "<hash>"
}'
```

## متغيرات البيئة

يقرأ OpenClaw متغيرات البيئة من العملية الأصل بالإضافة إلى:

-   `.env` من دليل العمل الحالي (إذا كان موجودًا)
-   `~/.openclaw/.env` (الرجوع العالمي)

لا يتجاوز أي ملف متغيرات البيئة الموجودة. يمكنك أيضًا تعيين متغيرات بيئة مضمنة في التكوين:

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

إذا تم تمكينه ولم يتم تعيين المفاتيح المتوقعة، يقوم OpenClaw بتشغيل shell تسجيل الدخول الخاص بك ويستورد فقط المفاتيح المفقودة:

```json
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

ما يعادل متغير البيئة: `OPENCLAW_LOAD_SHELL_ENV=1`

 

قم بالإشارة إلى متغيرات البيئة في أي قيمة سلسلة نصية للتكوين باستخدام `${VAR_NAME}`:

```json
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

القواعد:

-   الأسماء المكتوبة بأحرف كبيرة فقط متطابقة: `[A-Z_][A-Z0-9_]*`
-   المتغيرات المفقودة/الفارغة تسبب خطأ في وقت التحميل
-   الهروب باستخدام `$${VAR}` للإخراج الحرفي
-   يعمل داخل ملفات `$include`
-   الاستبدال المضمن: `"${BASE}/v1"` → `"https://api.example.com/v1"`

 

للحقول التي تدعم كائنات SecretRef، يمكنك استخدام:

```json
{
  models: {
    providers: {
      openai: { apiKey: { source: "env", provider: "default", id: "OPENAI_API_KEY" } },
    },
  },
  skills: {
    entries: {
      "nano-banana-pro": {
        apiKey: {
          source: "file",
          provider: "filemain",
          id: "/skills/entries/nano-banana-pro/apiKey",
        },
      },
    },
  },
  channels: {
    googlechat: {
      serviceAccountRef: {
        source: "exec",
        provider: "vault",
        id: "channels/googlechat/serviceAccount",
      },
    },
  },
}
```

تفاصيل SecretRef (بما في ذلك `secrets.providers` لـ `env`/`file`/`exec`) موجودة في [إدارة الأسرار](./secrets.md). المسارات المدعومة للاعتمادات مدرجة في [سطح اعتماد SecretRef](../reference/secretref-credential-surface.md).

 راجع [البيئة](../help/environment.md) للحصول على الأسبقية والمصادر الكاملة.

## المرجع الكامل

للحصول على المرجع الكامل حقلًا بحقل، راجع **[مرجع التكوين](./configuration-reference.md)**.

* * *

*ذات صلة: [أمثلة التكوين](./configuration-examples.md) · [مرجع التكوين](./configuration-reference.md) · [Doctor](./doctor.md)*

[دفتر تشغيل البوابة](../gateway.md)[مرجع التكوين](./configuration-reference.md)