

  تنسيق الوكلاء

  
# وكلاء ACP

تسمح جلسات [بروتوكول العميل الوكيل (ACP)](https://agentclientprotocol.com/) لـ OpenClaw بتشغيل أدوات برمجية خارجية (على سبيل المثال Pi، Claude Code، Codex، OpenCode، و Gemini CLI) من خلال مكون إضافي للخلفية يدعم ACP. إذا طلبت من OpenClaw بلغة عادية "تشغيل هذا في Codex" أو "بدء Claude Code في سلسلة محادثة"، فيجب أن يقوم OpenClaw بتوجيه هذا الطلب إلى وقت تشغيل ACP (وليس وقت تشغيل الوكلاء الفرعيين الأصليين).

## سير العمل السريع للمشغل

استخدم هذا عندما تريد دليل تشغيل `/acp` عمليًا:

1.  إنشاء جلسة:
    -   `/acp spawn codex --mode persistent --thread auto`
2.  اعمل في سلسلة المحادثة المقيدة (أو استهدف مفتاح تلك الجلسة صراحةً).
3.  تحقق من حالة وقت التشغيل:
    -   `/acp status`
4.  اضبط خيارات وقت التشغيل حسب الحاجة:
    -   `/acp model <provider/model>`
    -   `/acp permissions `
    -   `/acp timeout `
5.  توجيه جلسة نشطة دون استبدال السياق:
    -   `/acp steer tighten logging and continue`
6.  أوقف العمل:
    -   `/acp cancel` (أوقف الدور الحالي)، أو
    -   `/acp close` (أغلق الجلسة + أزل الربط)

## بداية سريعة للمستخدمين

أمثلة على الطلبات الطبيعية:

-   "ابدأ جلسة Codex دائمة في سلسلة محادثة هنا وحافظ على تركيزها."
-   "شغّل هذا كجلسة ACP لمرة واحدة باستخدام Claude Code و لخص النتيجة."
-   "استخدم Gemini CLI لهذه المهمة في سلسلة محادثة، ثم احتفظ بالمتابعات في نفس السلسلة."

ما يجب أن يفعله OpenClaw:

1.  اختيار `runtime: "acp"`.
2.  تحديد هدف الأداة المطلوبة (`agentId`، على سبيل المثال `codex`).
3.  إذا تم طلب ربط سلسلة محادثة وقناة الاتصال الحالية تدعم ذلك، قم بربط جلسة ACP بسلسلة المحادثة.
4.  توجيه رسائل المتابعة في سلسلة المحادثة إلى نفس جلسة ACP حتى يتم إلغاء التركيز/الإغلاق/انتهاء الصلاحية.

## ACP مقابل الوكلاء الفرعيين

استخدم ACP عندما تريد وقت تشغيل لأداة خارجية. استخدم الوكلاء الفرعيين عندما تريد عمليات تفويض أصلية من OpenClaw.

| المجال | جلسة ACP | تشغيل وكيل فرعي |
| --- | --- | --- |
| وقت التشغيل | مكون إضافي للخلفية يدعم ACP (مثل acpx) | وقت تشغيل الوكلاء الفرعيين الأصلي في OpenClaw |
| مفتاح الجلسة | `agent::acp:` | `agent::subagent:` |
| الأوامر الرئيسية | `/acp ...` | `/subagents ...` |
| أداة الإنشاء | `sessions_spawn` مع `runtime:"acp"` | `sessions_spawn` (وقت التشغيل الافتراضي) |

انظر أيضًا [الوكلاء الفرعيون](./subagents.md).

## الجلسات المقيدة بسلاسل المحادثات (مستقلة عن القناة)

عند تمكين الربط بسلاسل المحادثات لمحول قناة اتصال، يمكن ربط جلسات ACP بسلاسل المحادثات:

-   يقوم OpenClaw بربط سلسلة محادثة بجلسة ACP مستهدفة.
-   يتم توجيه رسائل المتابعة في تلك السلسلة إلى جلسة ACP المقيدة.
-   يتم إرجاع مخرجات ACP إلى نفس سلسلة المحادثة.
-   يؤدي إلغاء التركيز/الإغلاق/الأرشفة/انتهاء مهلة الخمول أو انتهاء الصلاحية بحد أقصى إلى إزالة الربط.

دعم الربط بسلاسل المحادثات خاص بالمحول. إذا كان محول القناة النشط لا يدعم ربط سلاسل المحادثات، يعيد OpenClaw رسالة واضحة تفيد بعدم الدعم/التوفر. أعلام الميزات المطلوبة لـ ACP المقيد بسلاسل المحادثات:

-   `acp.enabled=true`
-   `acp.dispatch.enabled` مفعل افتراضيًا (اضبط `false` لإيقاف إرسال ACP مؤقتًا)
-   علم إنشاء جلسات ACP لربط سلاسل المحادثات في محول القناة مفعل (خاص بالمحول)
    -   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

### القنوات الداعمة لسلاسل المحادثات

-   أي محول قناة يكشف عن قدرة الربط بين الجلسة/سلسلة المحادثة.
-   الدعم المدمج الحالي:
    -   سلاسل المحادثات/القنوات في Discord
    -   مواضيع Telegram (المواضيع في المنتديات داخل المجموعات/المجموعات الفائقة ومواضيع الدردشة الخاصة)
-   يمكن للقنوات المكونة من مكونات إضافية إضافة الدعم من خلال نفس واجهة الربط.

## إعدادات خاصة بالقناة

لعمليات سير العمل غير المؤقتة، قم بتكوين ارتباطات ACP دائمة في إدخالات `bindings[]` عالية المستوى.

### نموذج الربط

-   `bindings[].type="acp"` يحدد ارتباط محادثة ACP دائم.
-   `bindings[].match` يحدد المحادثة المستهدفة:
    -   قناة أو سلسلة محادثة في Discord: `match.channel="discord"` + `match.peer.id=""`
    -   موضوع منتدى في Telegram: `match.channel="telegram"` + `match.peer.id=":topic:"`
-   `bindings[].agentId` هو معرف وكيل OpenClaw المالك.
-   تجاوزات ACP الاختيارية موجودة تحت `bindings[].acp`:
    -   `mode` (`persistent` أو `oneshot`)
    -   `label`
    -   `cwd`
    -   `backend`

### الإعدادات الافتراضية لوقت التشغيل لكل وكيل

استخدم `agents.list[].runtime` لتعريف إعدادات ACP الافتراضية مرة واحدة لكل وكيل:

-   `agents.list[].runtime.type="acp"`
-   `agents.list[].runtime.acp.agent` (معرف الأداة، على سبيل المثال `codex` أو `claude`)
-   `agents.list[].runtime.acp.backend`
-   `agents.list[].runtime.acp.mode`
-   `agents.list[].runtime.acp.cwd`

أولوية التجاوز للجلسات المقيدة بـ ACP:

1.  `bindings[].acp.*`
2.  `agents.list[].runtime.acp.*`
3.  الإعدادات الافتراضية العالمية لـ ACP (مثل `acp.backend`)

مثال:

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
      {
        id: "claude",
        runtime: {
          type: "acp",
          acp: { agent: "claude", backend: "acpx", mode: "persistent" },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
    {
      type: "acp",
      agentId: "claude",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
      acp: { cwd: "/workspace/repo-b" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "discord", accountId: "default" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "telegram", accountId: "default" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": { requireMention: false },
          },
        },
      },
    },
    telegram: {
      groups: {
        "-1001234567890": {
          topics: { "42": { requireMention: false } },
        },
      },
    },
  },
}
```

السلوك:

-   يتأكد OpenClaw من وجود جلسة ACP المكونة قبل الاستخدام.
-   يتم توجيه الرسائل في تلك القناة أو الموضوع إلى جلسة ACP المكونة.
-   في المحادثات المقيدة، تقوم `/new` و `/reset` بإعادة تعيين نفس مفتاح جلسة ACP في مكانه.
-   لا تزال ارتباطات وقت التشغيل المؤقتة (التي تم إنشاؤها بواسطة سير عمل التركيز على سلسلة المحادثة) سارية حيثما تكون موجودة.

## بدء جلسات ACP (واجهات)

### من sessions\_spawn

استخدم `runtime: "acp"` لبدء جلسة ACP من دور وكيل أو استدعاء أداة.

```json
{
  "task": "Open the repo and summarize failing tests",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
```

ملاحظات:

-   `runtime` الافتراضي هو `subagent`، لذا اضبط `runtime: "acp"` صراحةً لجلسات ACP.
-   إذا تم حذف `agentId`، يستخدم OpenClaw `acp.defaultAgent` عند التكوين.
-   `mode: "session"` يتطلب `thread: true` للحفاظ على محادثة مقيدة دائمة.

تفاصيل الواجهة:

-   `task` (مطلوب): المطالبة الأولية المرسلة إلى جلسة ACP.
-   `runtime` (مطلوب لـ ACP): يجب أن يكون `"acp"`.
-   `agentId` (اختياري): معرف أداة ACP المستهدفة. يتراجع إلى `acp.defaultAgent` إذا تم تعيينه.
-   `thread` (اختياري، افتراضي `false`): طلب سير عمل ربط سلسلة المحادثة حيثما يكون مدعومًا.
-   `mode` (اختياري): `run` (لمرة واحدة) أو `session` (دائم).
    -   الافتراضي هو `run`
    -   إذا كان `thread: true` وتم حذف mode، فقد يقوم OpenClaw بالتراجع إلى السلوك الدائم حسب مسار وقت التشغيل
    -   `mode: "session"` يتطلب `thread: true`
-   `cwd` (اختياري): دليل العمل المطلوب لوقت التشغيل (يتم التحقق منه بواسطة سياسة الخلفية/وقت التشغيل).
-   `label` (اختياري): تسمية موجهة للمشغل تُستخدم في نص الجلسة/الشعار.
-   `streamTo` (اختياري): `"parent"` يقوم ببث ملخصات تقدم تشغيل ACP الأولية مرة أخرى إلى جلسة الطالب كأحداث نظام.
    -   عند التوفر، تتضمن الردود المقبولة `streamLogPath` الذي يشير إلى سجل JSONL محدد بالنطاق للجلسة (`.acp-stream.jsonl`) يمكنك متابعته للحصول على تاريخ النقل الكامل.

## التوافق مع بيئة الحماية

تعمل جلسات ACP حاليًا على وقت تشغيل المضيف، وليس داخل بيئة الحماية الخاصة بـ OpenClaw. القيود الحالية:

-   إذا كانت جلسة الطالب محمية ببيئة حماية، يتم حظر إنشاء جلسات ACP.
    -   خطأ: `لا يمكن للجلسات المحمية ببيئة الحماية إنشاء جلسات ACP لأن runtime="acp" يعمل على المضيف. استخدم runtime="subagent" من الجلسات المحمية ببيئة الحماية.`
-   `sessions_spawn` مع `runtime: "acp"` لا يدعم `sandbox: "require"`.
    -   خطأ: `sessions_spawn sandbox="require" غير مدعوم لـ runtime="acp" لأن جلسات ACP تعمل خارج بيئة الحماية. استخدم runtime="subagent" أو sandbox="inherit".`

استخدم `runtime: "subagent"` عندما تحتاج إلى تنفيذ مفروض ببيئة حماية.

### من أمر /acp

استخدم `/acp spawn` للتحكم الصريح من قبل المشغل من الدردشة عند الحاجة.

```bash
/acp spawn codex --mode persistent --thread auto
/acp spawn codex --mode oneshot --thread off
/acp spawn codex --thread here
```

الأعلام الرئيسية:

-   `--mode persistent|oneshot`
-   `--thread auto|here|off`
-   `--cwd <absolute-path>`
-   `--label `

انظر [أوامر الشرطة المائلة](./slash-commands.md).

## تحديد هدف الجلسة

تقبل معظم إجراءات `/acp` هدف جلسة اختياري (`session-key`، `session-id`، أو `session-label`). ترتيب التحديد:

1.  وسيط هدف صريح (أو `--session` لـ `/acp steer`)
    -   يحاول المفتاح أولاً
    -   ثم معرف الجلسة على شكل UUID
    -   ثم التسمية
2.  ربط سلسلة المحادثة الحالية (إذا كانت هذه المحادثة/السلسلة مربوطة بجلسة ACP)
3.  التراجع إلى جلسة الطالب الحالية

إذا لم يتم تحديد أي هدف، يعيد OpenClaw خطأ واضح (`Unable to resolve session target: ...`).

## أوضاع إنشاء سلسلة المحادثة

يدعم `/acp spawn` `--thread auto|here|off`.

| الوضع | السلوك |
| --- | --- |
| `auto` | في سلسلة محادثة نشطة: اربط تلك السلسلة. خارج سلسلة محادثة: أنشئ/اربط سلسلة محادثة فرعية عند الدعم. |
| `here` | اشترط وجود سلسلة محادثة نشطة حالية؛ فشل إذا لم تكن في واحدة. |
| `off` | لا يوجد ربط. تبدأ الجلسة غير مقيدة. |

ملاحظات:

-   على الأسطح غير الداعمة لربط سلاسل المحادثات، السلوك الافتراضي هو `off` بشكل فعال.
-   يتطلب الإنشاء المقيد بسلسلة محادثة دعم سياسة القناة:
    -   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

## عناصر تحكم ACP

عائلة الأوامر المتاحة:

-   `/acp spawn`
-   `/acp cancel`
-   `/acp steer`
-   `/acp close`
-   `/acp status`
-   `/acp set-mode`
-   `/acp set`
-   `/acp cwd`
-   `/acp permissions`
-   `/acp timeout`
-   `/acp model`
-   `/acp reset-options`
-   `/acp sessions`
-   `/acp doctor`
-   `/acp install`

يظهر `/acp status` خيارات وقت التشغيل الفعالة، وعند التوفر، معرفات الجلسة على مستوى وقت التشغيل ومستوى الخلفية. تعتمد بعض عناصر التحكم على إمكانيات الخلفية. إذا كانت الخلفية لا تدعم عنصر تحكم معين، يعيد OpenClaw خطأ واضحًا يفيد بعدم دعم عنصر التحكم.

## كتاب وصفات أوامر ACP

| الأمر | ما يفعله | مثال |
| --- | --- | --- |
| `/acp spawn` | إنشاء جلسة ACP؛ ربط سلسلة محادثة اختياري. | `/acp spawn codex --mode persistent --thread auto --cwd /repo` |
| `/acp cancel` | إلغاء الدور قيد التنفيذ للجلسة المستهدفة. | `/acp cancel agent:codex:acp:` |
| `/acp steer` | إرسال تعليمات توجيهية إلى الجلسة قيد التشغيل. | `/acp steer --session support inbox prioritize failing tests` |
| `/acp close` | إغلاق الجلسة وإلغاء ربط أهداف سلسلة المحادثة. | `/acp close` |
| `/acp status` | عرض الخلفية، الوضع، الحالة، خيارات وقت التشغيل، الإمكانيات. | `/acp status` |
| `/acp set-mode` | تعيين وضع وقت التشغيل للجلسة المستهدفة. | `/acp set-mode plan` |
| `/acp set` | كتابة تكوين وقت تشغيل عام. | `/acp set model openai/gpt-5.2` |
| `/acp cwd` | تعيين تجاوز دليل العمل لوقت التشغيل. | `/acp cwd /Users/user/Projects/repo` |
| `/acp permissions` | تعيين ملف تعريف سياسة الموافقة. | `/acp permissions strict` |
| `/acp timeout` | تعيين مهلة وقت التشغيل (بالثواني). | `/acp timeout 120` |
| `/acp model` | تعيين تجاوز نموذج وقت التشغيل. | `/acp model anthropic/claude-opus-4-5` |
| `/acp reset-options` | إزالة تجاوزات خيارات وقت التشغيل للجلسة. | `/acp reset-options` |
| `/acp sessions` | سرد جلسات ACP الحديثة من المخزن. | `/acp sessions` |
| `/acp doctor` | صحة الخلفية، الإمكانيات، الإصلاحات القابلة للتنفيذ. | `/acp doctor` |
| `/acp install` | طباعة خطوات التثبيت والتمكين الحتمية. | `/acp install` |

## تعيين خيارات وقت التشغيل

يحتوي `/acp` على أوامر ملائمة وواضعة عامة. العمليات المكافئة:

-   `/acp model ` يُعيّن لمفتاح تكوين وقت التشغيل `model`.
-   `/acp permissions ` يُعيّن لمفتاح تكوين وقت التشغيل `approval_policy`.
-   `/acp timeout ` يُعيّن لمفتاح تكوين وقت التشغيل `timeout`.
-   `/acp cwd ` يقوم بتحديث تجاوز دليل العمل لوقت التشغيل مباشرةً.
-   `/acp set  ` هو المسار العام.
    -   حالة خاصة: `key=cwd` يستخدم مسار تجاوز دليل العمل.
-   `/acp reset-options` يمسح جميع تجاوزات وقت التشغيل للجلسة المستهدفة.

## دعم أدوات acpx (الحالي)

أسماء مستعارة للأدوات المدمجة في acpx الحالية:

-   `pi`
-   `claude`
-   `codex`
-   `opencode`
-   `gemini`
-   `kimi`

عندما يستخدم OpenClaw خلفية acpx، يُفضل استخدام هذه القيم لـ `agentId` إلا إذا كان تكوين acpx الخاص بك يعرّف أسماء مستعارة مخصصة للوكلاء. يمكن أيضًا لاستخدام سطر أوامر acpx المباشر استهداف محولات عشوائية عبر `--agent `، ولكن هذا الهروب الخام هو ميزة في سطر أوامر acpx (وليس مسار `agentId` العادي في OpenClaw).

## التكوين المطلوب

الأساسيات الأساسية لـ ACP:

```json
{
  acp: {
    enabled: true,
    // اختياري. الافتراضي هو true؛ اضبط false لإيقاف إرسال ACP مؤقتًا مع الاحتفاظ بعناصر تحكم /acp.
    dispatch: { enabled: true },
    backend: "acpx",
    defaultAgent: "codex",
    allowedAgents: ["pi", "claude", "codex", "opencode", "gemini", "kimi"],
    maxConcurrentSessions: 8,
    stream: {
      coalesceIdleMs: 300,
      maxChunkChars: 1200,
    },
    runtime: {
      ttlMinutes: 120,
    },
  },
}
```

تكوين الربط بسلاسل المحادثات خاص بمحول القناة. مثال لـ Discord:

```json
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        spawnAcpSessions: true,
      },
    },
  },
}
```

إذا لم يعمل إنشاء ACP المقيد بسلسلة محادثة، تحقق أولاً من علم ميزة المحول:

-   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`

انظر [مرجع التكوين](../gateway/configuration-reference.md).

## إعداد المكون الإضافي لخلفية acpx

قم بتثبيت وتمكين المكون الإضافي:

```bash
openclaw plugins install acpx
openclaw config set plugins.entries.acpx.enabled true
```

التثبيت المحلي في مساحة العمل أثناء التطوير:

```bash
openclaw plugins install ./extensions/acpx
```

ثم تحقق من صحة الخلفية:

```bash
/acp doctor
```

### تكوين أمر وإصدار acpx

بشكل افتراضي، يستخدم مكون acpx الإضافي (المنشور باسم `@openclaw/acpx`) الثنائي المثبت محليًا مع المكون الإضافي:

1.  الأمر الافتراضي هو `extensions/acpx/node_modules/.bin/acpx`.
2.  الإصدار المتوقع الافتراضي هو الإصدار المثبت مع الامتداد.
3.  بدء التشغيل يسجل خلفية ACP على الفور على أنها غير جاهزة.
4.  مهمة ضمان في الخلفية تتحقق من `acpx --version`.
5.  إذا كان الثنائي المثبت مع المكون الإضافي مفقودًا أو غير متطابق، فإنه يشغل: `npm install --omit=dev --no-save acpx@` ويعيد التحقق.

يمكنك تجاوز الأمر/الإصدار في تكوين المكون الإضافي:

```json
{
  "plugins": {
    "entries": {
      "acpx": {
        "enabled": true,
        "config": {
          "command": "../acpx/dist/cli.js",
          "expectedVersion": "any"
        }
      }
    }
  }
}
```

ملاحظات:

-   `command` يقبل مسارًا مطلقًا، مسارًا نسبيًا، أو اسم أمر (`acpx`).
-   يتم حل المسارات النسبية من دليل مساحة عمل OpenClaw.
-   `expectedVersion: "any"` يعطل المطابقة الصارمة للإصدار.
-   عندما يشير `command` إلى ثنائي/مسار مخصص، يتم تعطيل التثبيت التلقائي الخاص بالمكون الإضافي.
-   يظل بدء تشغيل OpenClaw غير معيق أثناء تشغيل فحص صحة الخلفية.

انظر [المكونات الإضافية](./plugin.md).

## تكوين الأذونات

تعمل جلسات ACP بشكل غير تفاعلي — لا يوجد طرفية TTY للموافقة على مطالبات أذونات كتابة الملفات وتنفيذ الأوامر أو رفضها. يوفر مكون acpx الإضافي مفتاحي تكوين يتحكمان في كيفية معالجة الأذونات:

### permissionMode

يتحكم في العمليات التي يمكن لوكيل الأداة تنفيذها دون مطالبة.

| القيمة | السلوك |
| --- | --- |
| `approve-all` | الموافقة التلقائية على جميع كتابات الملفات وأوامر shell. |
| `approve-reads` | الموافقة التلقائية على عمليات القراءة فقط؛ تتطلب الكتابات والتنفيذ مطالبات. |
| `deny-all` | رفض جميع مطالبات الأذونات. |

### nonInteractivePermissions

يتحكم فيما يحدث عندما يتم عرض مطالبة إذن ولكن لا يوجد طرفية TTY تفاعلية متاحة (وهو الحال دائمًا مع جلسات ACP).

| القيمة | السلوك |
| --- | --- |
| `fail` | إحباط الجلسة مع `AcpRuntimeError`. **(الافتراضي)** |
| `deny` | رفض الإذن بصمت والمتابعة (تدهور سلس). |

### التكوين

اضبط عبر تكوين المكون الإضافي:

```bash
openclaw config set plugins.entries.acpx.config.permissionMode approve-all
openclaw config set plugins.entries.acpx.config.nonInteractivePermissions fail
```

أعد تشغيل البوابة بعد تغيير هذه القيم.

> **مهم:** OpenClaw حاليًا يستخدم افتراضيًا `permissionMode=approve-reads` و `nonInteractivePermissions=fail`. في جلسات ACP غير التفاعلية، قد تفشل أي كتابة أو تنفيذ يؤدي إلى مطالبة إذن مع `AcpRuntimeError: Permission prompt unavailable in non-interactive mode`. إذا كنت بحاجة إلى تقييد الأذونات، اضبط `nonInteractivePermissions` على `deny` حتى تتدهور الجلسات بسلاسة بدلاً من التعطل.

## استكشاف الأخطاء وإصلاحها

| العرض | السبب المحتمل | الإصلاح |
| --- | --- | --- |
| `ACP runtime backend is not configured` | مكون الخلفية الإضافي مفقود أو معطل. | قم بتثبيت وتمكين مكون الخلفية الإضافي، ثم شغل `/acp doctor`. |
| `ACP is disabled by policy (acp.enabled=false)` | ACP معطل عالميًا. | اضبط `acp.enabled=true`. |
| `ACP dispatch is disabled by policy (acp.dispatch.enabled=false)` | الإرسال من رسائل سلسلة المحادثة العادية معطل. | اضبط `acp.dispatch.enabled=true`. |
| `ACP agent "" is not allowed by policy` | الوكيل غير موجود في قائمة السماح. | استخدم `agentId` مسموحًا به أو قم بتحديث `acp.allowedAgents`. |
| `Unable to resolve session target: ...` | رمز مفتاح/معرف/تسمية غير صالح. | شغل `/acp sessions`، انسخ المفتاح/التسمية بالضبط، أعد المحاولة. |
| `--thread here requires running /acp spawn inside an active ... thread` | تم استخدام `--thread here` خارج سياق سلسلة محادثة. | انتقل إلى سلسلة المحادثة المستهدفة أو استخدم `--thread auto`/`off`. |
| `Only <user-id> can rebind this thread.` | مستخدم آخر يملك ربط سلسلة المحادثة. | أعد الربط كمُلك أو استخدم سلسلة محادثة مختلفة. |
| `Thread bindings are unavailable for .` | المحول يفتقر إلى قدرة ربط سلاسل المحادثات. | استخدم `--thread off` أو انتقل إلى محول/قناة مدعومة. |
| `Sandboxed sessions cannot spawn ACP sessions ...` | وقت تشغيل ACP على جانب المضيف؛ جلسة الطالب محمية ببيئة حماية. | استخدم `runtime="subagent"` من الجلسات المحمية ببيئة حماية، أو شغل إنشاء ACP من جلسة غير محمية ببيئة حماية. |
| `sessions_spawn sandbox="require" is unsupported for runtime="acp" ...` | تم طلب `sandbox="require"` لوقت تشغيل ACP. | استخدم `runtime="subagent"` للحماية المطلوبة ببيئة حماية، أو استخدم ACP مع `sandbox="inherit"` من جلسة غير محمية ببيئة حماية. |
| Missing ACP metadata for bound session | بيانات وصفية لجلسة ACP قديمة/محذوفة. | أعد الإنشاء باستخدام `/acp spawn`، ثم أعد ربط/تركيز سلسلة المحادثة. |
| `AcpRuntimeError: Permission prompt unavailable in non-interactive mode` | `permissionMode` يمنع الكتابات/التنفيذ في جلسة ACP غير التفاعلية. | اضبط `plugins.entries.acpx.config.permissionMode` على `approve-all` وأعد تشغيل البوابة. انظر [تكوين الأذونات](#permission-configuration). |
| ACP session fails early with little output | مطالبات الأذونات محظورة بواسطة `permissionMode`/`nonInteractivePermissions`. | تحقق من سجلات البوابة بحثًا عن `AcpRuntimeError`. للأذونات الكاملة، اضبط `permissionMode=approve-all`؛ للتدهور السلس، اضبط `nonInteractivePermissions=deny`. |
| ACP session stalls indefinitely after completing work | انتهت عملية الأداة ولكن جلسة ACP لم تبلغ عن الاكتمال. | راقب باستخدام `ps aux \| grep acpx`؛ أوقف العمليات القديمة يدويًا. |

[الوكلاء الفرعيون](./subagents.md)[بيئة الحماية متعددة الوكلاء والأدوات](./multi-agent-sandbox-tools.md)