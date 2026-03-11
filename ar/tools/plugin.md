

  المهارات

  
# الإضافات

## البدء السريع (جديد في الإضافات؟)

الإضافة هي مجرد **وحدة كود صغيرة** تمدّد OpenClaw بميزات إضافية (أوامر، أدوات، و Gateway RPC). في معظم الأحيان، ستستخدم الإضافات عندما تريد ميزة غير مدمجة في نواة OpenClaw بعد (أو تريد إبقاء الميزات الاختيارية خارج التثبيت الرئيسي). المسار السريع:

1.  انظر ما هو محمّل بالفعل:

```bash
openclaw plugins list
```

2.  ثبّت إضافة رسمية (مثال: Voice Call):

```bash
openclaw plugins install @openclaw/voice-call
```

مواصفات Npm هي **للسجل فقط** (اسم الحزمة + **إصدار محدد** اختياري أو **dist-tag**). المواصفات من نوع Git/URL/ملف ونطاقات semver مرفوضة. المواصفات العارية و `@latest` تبقى على المسار المستقر. إذا حلّت npm أيًا من هذين إلى إصدار ما قبل النشر، يتوقف OpenClaw ويطلب منك الموافقة صراحةً باستخدام علامة ما قبل النشر مثل `@beta`/`@rc` أو إصدار محدد ما قبل النشر.

3.  أعد تشغيل Gateway، ثم قم بالتكوين تحت `plugins.entries..config`.

انظر [Voice Call](../plugins/voice-call.md) لمثال إضافة ملموس. تبحث عن قوائم الطرف الثالث؟ انظر [إضافات المجتمع](../plugins/community.md).

## الإضافات المتاحة (الرسمية)

-   Microsoft Teams أصبحت إضافة فقط اعتبارًا من 2026.1.15؛ ثبّت `@openclaw/msteams` إذا كنت تستخدم Teams.
-   الذاكرة (النواة) — إضافة بحث الذاكرة المدمجة (مفعّلة افتراضيًا عبر `plugins.slots.memory`)
-   الذاكرة (LanceDB) — إضافة الذاكرة طويلة الأمد المدمجة (استرجاع/التقاط تلقائي؛ عيّن `plugins.slots.memory = "memory-lancedb"`)
-   [Voice Call](../plugins/voice-call.md) — `@openclaw/voice-call`
-   [Zalo Personal](../plugins/zalouser.md) — `@openclaw/zalouser`
-   [Matrix](../channels/matrix.md) — `@openclaw/matrix`
-   [Nostr](../channels/nostr.md) — `@openclaw/nostr`
-   [Zalo](../channels/zalo.md) — `@openclaw/zalo`
-   [Microsoft Teams](../channels/msteams.md) — `@openclaw/msteams`
-   Google Antigravity OAuth (مصادقة المزود) — مدمجة كـ `google-antigravity-auth` (معطّلة افتراضيًا)
-   Gemini CLI OAuth (مصادقة المزود) — مدمجة كـ `google-gemini-cli-auth` (معطّلة افتراضيًا)
-   Qwen OAuth (مصادقة المزود) — مدمجة كـ `qwen-portal-auth` (معطّلة افتراضيًا)
-   Copilot Proxy (مصادقة المزود) — جسر VS Code Copilot Proxy المحلي؛ متميز عن تسجيل الدخول المدمج `github-copilot` للجهاز (مدمج، معطّل افتراضيًا)

إضافات OpenClaw هي **وحدات TypeScript** تُحمّل في وقت التشغيل عبر jiti. **التحقق من صحة التكوين لا ينفّذ كود الإضافة**؛ بل يستخدم بيان الإضافة و JSON Schema بدلاً من ذلك. انظر [بيان الإضافة](../plugins/manifest.md). يمكن للإضافات تسجيل:

-   طرق Gateway RPC
-   مسارات HTTP للـ Gateway
-   أدوات الوكيل
-   أوامر CLI
-   خدمات الخلفية
-   محركات السياق
-   التحقق الاختياري من صحة التكوين
-   **المهارات** (عن طريق سرد أدلة `skills` في بيان الإضافة)
-   **أوامر الرد التلقائي** (تنفّذ دون استدعاء وكيل الذكاء الاصطناعي)

تعمل الإضافات **ضمن العملية** مع Gateway، لذا عالجها ككود موثوق. دليل كتابة الأدوات: [أدوات وكيل الإضافة](../plugins/agent-tools.md).

## مساعدات وقت التشغيل

يمكن للإضافات الوصول إلى مساعدات النواة المحددة عبر `api.runtime`. لتحويل النص إلى كلام للهاتف:

```typescript
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

ملاحظات:

-   يستخدم تكوين `messages.tts` الأساسي (OpenAI أو ElevenLabs).
-   يُرجع مخزن مؤقت للصوت PCM + معدل العينة. يجب على الإضافات إعادة العينة/التشفير للمزودين.
-   Edge TTS غير مدعوم للهاتف.

لـ STT/النسخ، يمكن للإضافات استدعاء:

```typescript
const { text } = await api.runtime.stt.transcribeAudioFile({
  filePath: "/tmp/inbound-audio.ogg",
  cfg: api.config,
  // اختياري عندما لا يمكن الاستدلال على MIME بشكل موثوق:
  mime: "audio/ogg",
});
```

ملاحظات:

-   يستخدم تكوين الصوت لفهم الوسائط الأساسي (`tools.media.audio`) وترتيب التراجع للمزود.
-   يُرجع `{ text: undefined }` عندما لا يتم إنتاج ناتج نسخ (على سبيل المثال، مدخلات تم تخطيها/غير مدعومة).

## مسارات HTTP للـ Gateway

يمكن للإضافات كشف نقاط نهاية HTTP مع `api.registerHttpRoute(...)`.

```
api.registerHttpRoute({
  path: "/acme/webhook",
  auth: "plugin",
  match: "exact",
  handler: async (_req, res) => {
    res.statusCode = 200;
    res.end("ok");
    return true;
  },
});
```

حقول المسار:

-   `path`: مسار المسار تحت خادم HTTP للـ gateway.
-   `auth`: مطلوب. استخدم `"gateway"` لتطلب مصادقة gateway العادية، أو `"plugin"` للمصادقة المدارة بواسطة الإضافة/التحقق من webhook.
-   `match`: اختياري. `"exact"` (الافتراضي) أو `"prefix"`.
-   `replaceExisting`: اختياري. يسمح للإضافة نفسها باستبدال تسجيل مسارها الحالي.
-   `handler`: أعد `true` عندما يعالج المسار الطلب.

ملاحظات:

-   `api.registerHttpHandler(...)` أصبح قديمًا. استخدم `api.registerHttpRoute(...)`.
-   يجب على مسارات الإضافة الإعلان عن `auth` صراحةً.
-   يتم رفض تعارضات `path + match` الدقيقة ما لم يكن `replaceExisting: true`، ولا يمكن لإضافة استبدال مسار إضافة أخرى.
-   يتم رفض المسارات المتداخلة بمستويات `auth` مختلفة. حافظ على سلاسل التراجع `exact`/`prefix` على نفس مستوى المصادقة فقط.

## مسارات استيراد SDK للإضافة

استخدم المسارات الفرعية لـ SDK بدلاً من استيراد `openclaw/plugin-sdk` الأحادي عند كتابة الإضافات:

-   `openclaw/plugin-sdk/core` لواجهات برمجة تطبيقات الإضافة العامة، أنواع مصادقة المزود، والمساعدات المشتركة.
-   `openclaw/plugin-sdk/compat` لكود الإضافة المدمج/الداخلي الذي يحتاج إلى مساعدات وقت تشغيل مشتركة أوسع من `core`.
-   `openclaw/plugin-sdk/telegram` لإضافات قناة Telegram.
-   `openclaw/plugin-sdk/discord` لإضافات قناة Discord.
-   `openclaw/plugin-sdk/slack` لإضافات قناة Slack.
-   `openclaw/plugin-sdk/signal` لإضافات قناة Signal.
-   `openclaw/plugin-sdk/imessage` لإضافات قناة iMessage.
-   `openclaw/plugin-sdk/whatsapp` لإضافات قناة WhatsApp.
-   `openclaw/plugin-sdk/line` لإضافات قناة LINE.
-   `openclaw/plugin-sdk/msteams` لسطح إضافة Microsoft Teams المدمجة.
-   المسارات الفرعية المحددة للإضافات المدمجة متاحة أيضًا: `openclaw/plugin-sdk/acpx`, `openclaw/plugin-sdk/bluebubbles`, `openclaw/plugin-sdk/copilot-proxy`, `openclaw/plugin-sdk/device-pair`, `openclaw/plugin-sdk/diagnostics-otel`, `openclaw/plugin-sdk/diffs`, `openclaw/plugin-sdk/feishu`, `openclaw/plugin-sdk/google-gemini-cli-auth`, `openclaw/plugin-sdk/googlechat`, `openclaw/plugin-sdk/irc`, `openclaw/plugin-sdk/llm-task`, `openclaw/plugin-sdk/lobster`, `openclaw/plugin-sdk/matrix`, `openclaw/plugin-sdk/mattermost`, `openclaw/plugin-sdk/memory-core`, `openclaw/plugin-sdk/memory-lancedb`, `openclaw/plugin-sdk/minimax-portal-auth`, `openclaw/plugin-sdk/nextcloud-talk`, `openclaw/plugin-sdk/nostr`, `openclaw/plugin-sdk/open-prose`, `openclaw/plugin-sdk/phone-control`, `openclaw/plugin-sdk/qwen-portal-auth`, `openclaw/plugin-sdk/synology-chat`, `openclaw/plugin-sdk/talk-voice`, `openclaw/plugin-sdk/test-utils`, `openclaw/plugin-sdk/thread-ownership`, `openclaw/plugin-sdk/tlon`, `openclaw/plugin-sdk/twitch`, `openclaw/plugin-sdk/voice-call`, `openclaw/plugin-sdk/zalo`, و `openclaw/plugin-sdk/zalouser`.

ملاحظة التوافق:

-   `openclaw/plugin-sdk` يبقى مدعومًا للإضافات الخارجية الحالية.
-   يجب على الإضافات المدمجة الجديدة والمهاجرة استخدام المسارات الفرعية المحددة للقناة أو الامتداد؛ استخدم `core` للأسطح العامة و `compat` فقط عندما تكون هناك حاجة إلى مساعدات مشتركة أوسع.

## فحص القناة للقراءة فقط

إذا سجلت إضافتك قناة، ففضل تنفيذ `plugin.config.inspectAccount(cfg, accountId)` بجانب `resolveAccount(...)`. لماذا:

-   `resolveAccount(...)` هو مسار وقت التشغيل. يُسمح له بافتراض أن بيانات الاعتماد مكتملة المادية ويمكنه الفشل بسرعة عندما تكون الأسرار المطلوبة مفقودة.
-   مسارات الأوامر للقراءة فقط مثل `openclaw status`, `openclaw status --all`, `openclaw channels status`, `openclaw channels resolve`, وتدفقات إصلاح الطبيب/التكوين لا يجب أن تحتاج إلى مادية بيانات اعتماد وقت التشغيل فقط لوصف التكوين.

السلوك الموصى به لـ `inspectAccount(...)`:

-   أعد حالة الحساب الوصفية فقط.
-   احفظ `enabled` و `configured`.
-   أضف حقول مصدر/حالة بيانات الاعتماد عند الصلة، مثل:
    -   `tokenSource`, `tokenStatus`
    -   `botTokenSource`, `botTokenStatus`
    -   `appTokenSource`, `appTokenStatus`
    -   `signingSecretSource`, `signingSecretStatus`
-   لا تحتاج إلى إرجاع قيم الرمز الأولية فقط للإبلاغ عن التوفر للقراءة فقط. إرجاع `tokenStatus: "available"` (وحقل المصدر المطابق) يكفي لأوامر النمط status.
-   استخدم `configured_unavailable` عندما يتم تكوين بيانات الاعتماد عبر SecretRef ولكنها غير متاحة في مسار الأمر الحالي.

هذا يسمح لأوامر القراءة فقط بالإبلاغ عن "مكوّنة ولكن غير متاحة في مسار الأمر هذا" بدلاً من التعطل أو الإبلاغ الخاطئ عن الحساب على أنه غير مكوّن. ملاحظة الأداء:

-   يستخدم اكتشاف الإضافة وبيانات تعريف البيان ذواكر تخزين مؤقت قصيرة داخل العملية لتقليل العمل الاندفاعي عند بدء التشغيل/إعادة التحميل.
-   عيّن `OPENCLAW_DISABLE_PLUGIN_DISCOVERY_CACHE=1` أو `OPENCLAW_DISABLE_PLUGIN_MANIFEST_CACHE=1` لتعطيل ذواكر التخزين المؤقت هذه.
-   اضبط نوافذ ذاكرة التخزين المؤقت باستخدام `OPENCLAW_PLUGIN_DISCOVERY_CACHE_MS` و `OPENCLAW_PLUGIN_MANIFEST_CACHE_MS`.

## الاكتشاف والأولوية

يفحص OpenClaw، بالترتيب:

1.  مسارات التكوين

-   `plugins.load.paths` (ملف أو دليل)

2.  امتدادات مساحة العمل

-   `/.openclaw/extensions/*.ts`
-   `/.openclaw/extensions/*/index.ts`

3.  الامتدادات العامة

-   `~/.openclaw/extensions/*.ts`
-   `~/.openclaw/extensions/*/index.ts`

4.  الامتدادات المدمجة (مرفقة مع OpenClaw، معظمها معطّل افتراضيًا)

-   `/extensions/*`

يجب تفعيل معظم الإضافات المدمجة صراحةً عبر `plugins.entries..enabled` أو `openclaw plugins enable `. استثناءات الإضافات المدمجة المفعلة افتراضيًا:

-   `device-pair`
-   `phone-control`
-   `talk-voice`
-   إضافة فتحة الذاكرة النشطة (الفتحة الافتراضية: `memory-core`)

الإضافات المثبتة مفعلة افتراضيًا، ولكن يمكن تعطيلها بنفس الطريقة. ملاحظات التعزيز:

-   إذا كان `plugins.allow` فارغًا وكانت هناك إضافات غير مدمجة قابلة للاكتشاف، يسجل OpenClaw تحذير بدء التشغيل مع معرفات الإضافات والمصادر.
-   يتم فحص المسارات المرشحة للسلامة قبل قبول الاكتشاف. يمنع OpenClaw المرشحين عندما:
    -   يحل مدخل الامتداد خارج جذر الإضافة (بما في ذلك الهروب من الارتباطات الرمزية/عبور المسار)،
    -   مسار جذر/مصدر الإضافة قابل للكتابة للعالم،
    -   ملكية المسار مشبوهة للإضافات غير المدمجة (المالك POSIX ليس هو uid الحالي ولا root).
-   الإضافات غير المدمجة المحمّلة بدون إثبات التثبيت/مسار التحميل تصدر تحذيرًا حتى تتمكن من تثبيت الثقة (`plugins.allow`) أو تتبع التثبيت (`plugins.installs`).

يجب أن تتضمن كل إضافة ملف `openclaw.plugin.json` في جذرها. إذا أشار مسار إلى ملف، فإن جذر الإضافة هو دليل الملف ويجب أن يحتوي على البيان. إذا حلت إضافات متعددة إلى نفس المعرف، فإن المطابقة الأولى بالترتيب أعلاه تفوز ويتم تجاهل النسخ ذات الأولوية الأقل.

### حزم الحزم

قد يتضمن دليل الإضافة `package.json` مع `openclaw.extensions`:

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

يصبح كل مدخل إضافة. إذا سردت الحزمة امتدادات متعددة، يصبح معرف الإضافة `name/`. إذا استوردت إضافتك تبعيات npm، فثبّتها في ذلك الدليل حتى يكون `node_modules` متاحًا (`npm install` / `pnpm install`). حاجز الأمان: يجب أن يبقى كل مدخل `openclaw.extensions` داخل دليل الإضافة بعد حل الارتباطات الرمزية. يتم رفض المداخل التي تهرب من دليل الحزمة. ملاحظة الأمان: `openclaw plugins install` يثبت تبعيات الإضافة مع `npm install --ignore-scripts` (لا توجد نصوص دورة حياة). حافظ على أشجار تبعيات الإضافة "نقية JS/TS" وتجنب الحزم التي تتطلب بناء `postinstall`.

### بيانات تعريف كتالوج القناة

يمكن للإضافات القناة الإعلان عن بيانات تعريف الإعداد عبر `openclaw.channel` وتلميحات التثبيت عبر `openclaw.install`. هذا يحافظ على بيانات الكتالوج الأساسية خالية من البيانات. مثال:

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (self-hosted)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Self-hosted chat via Nextcloud Talk webhook bots.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

يمكن لـ OpenClaw أيضًا دمج **كتالوجات القناة الخارجية** (على سبيل المثال، تصدير سجل MPM). ضع ملف JSON في أحد:

-   `~/.openclaw/mpm/plugins.json`
-   `~/.openclaw/mpm/catalog.json`
-   `~/.openclaw/plugins/catalog.json`

أو أشر `OPENCLAW_PLUGIN_CATALOG_PATHS` (أو `OPENCLAW_MPM_CATALOG_PATHS`) إلى ملف أو أكثر من ملفات JSON (مفصولة بفاصلة/فاصلة منقوطة/`PATH`). يجب أن يحتوي كل ملف على `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`.

## معرفات الإضافة

معرفات الإضافة الافتراضية:

-   حزم الحزم: `package.json` `name`
-   ملف منفرد: اسم الملف الأساسي (`~/.../voice-call.ts` → `voice-call`)

إذا صدّرت إضافة `id`، يستخدمه OpenClaw ولكن يحذر عندما لا يتطابق مع المعرف المكوّن.

## التكوين

```json
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

الحقول:

-   `enabled`: المفتاح الرئيسي (الافتراضي: true)
-   `allow`: قائمة السماح (اختياري)
-   `deny`: قائمة الرفض (اختياري؛ الرفض يفوز)
-   `load.paths`: ملفات/أدلة إضافية للإضافة
-   `slots`: محددات الفتحات الحصرية مثل `memory` و `contextEngine`
-   `entries.`: مفاتيح تبديل لكل إضافة + تكوين

تغييرات التكوين **تتطلب إعادة تشغيل gateway**. قواعد التحقق من الصحة (صارمة):

-   معرفات الإضافة غير المعروفة في `entries`, `allow`, `deny`, أو `slots` هي **أخطاء**.
-   مفاتيح `channels.` غير المعروفة هي **أخطاء** ما لم يعلن بيان إضافة عن معرف القناة.
-   يتم التحقق من تكوين الإضافة باستخدام JSON Schema المضمن في `openclaw.plugin.json` (`configSchema`).
-   إذا كانت الإضافة معطلة، يتم الاحتفاظ بتكوينها ويتم إصدار **تحذير**.

## فتحات الإضافة (فئات حصرية)

بعض فئات الإضافة **حصرية** (واحدة نشطة فقط في كل مرة). استخدم `plugins.slots` لتحديد الإضافة التي تملك الفتحة:

```json
{
  plugins: {
    slots: {
      memory: "memory-core", // أو "none" لتعطيل إضافات الذاكرة
      contextEngine: "legacy", // أو معرف إضافة مثل "lossless-claw"
    },
  },
}
```

الفتحات الحصرية المدعومة:

-   `memory`: إضافة الذاكرة النشطة (`"none"` تعطل إضافات الذاكرة)
-   `contextEngine`: إضافة محرك السياق النشطة (`"legacy"` هي الافتراضية المدمجة)

إذا أعلنت إضافات متعددة `kind: "memory"` أو `kind: "context-engine"`، فإن الإضافة المحددة فقط هي التي تُحمّل لتلك الفتحة. يتم تعطيل الأخرى مع تشخيصات.

### إضافات محرك السياق

تملك إضافات محرك السياق تنسيق سياق الجلسة للاستيعاب، التجميع، والضغط. سجلها من إضافتك باستخدام `api.registerContextEngine(id, factory)`، ثم حدد المحرك النشط باستخدام `plugins.slots.contextEngine`. استخدم هذا عندما تحتاج إضافتك إلى استبدال أو توسيع خط أنابيب السياق الافتراضي بدلاً من مجرد إضافة بحث ذاكرة أو خطافات.

## واجهة التحكم (المخطط + التسميات)

تستخدم واجهة التحكم `config.schema` (JSON Schema + `uiHints`) لعرض نماذج أفضل. يعزز OpenClaw `uiHints` في وقت التشغيل بناءً على الإضافات المكتشفة:

-   يضيف تسميات لكل إضافة لـ `plugins.entries.` / `.enabled` / `.config`
-   يدمج تلميحات حقول التكوين الاختيارية المقدمة من الإضافة تحت: `plugins.entries..config.`

إذا أردت أن تعرض حقول تكوين إضافتك تسميات/عناصر نائبة جيدة (وتحديد الأسرار كحساسة)، فقدم `uiHints` بجانب JSON Schema الخاص بك في بيان الإضافة. مثال:

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "مفتاح API", "sensitive": true },
    "region": { "label": "المنطقة", "placeholder": "us-east-1" }
  }
}
```

## CLI

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # انسخ ملف/دليل محلي إلى ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # مسار نسبي مقبول
openclaw plugins install ./plugin.tgz           # ثبّت من أرشيف tar محلي
openclaw plugins install ./plugin.zip           # ثبّت من أرشيف zip محلي
openclaw plugins install -l ./extensions/voice-call # رابط (بدون نسخ) للتطوير
openclaw plugins install @openclaw/voice-call # ثبّت من npm
openclaw plugins install @openclaw/voice-call --pin # خزن الاسم@الإصدار المحلول بالضبط
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` تعمل فقط لتثبيتات npm المتعقبة تحت `plugins.installs`. إذا تغيرت بيانات النزاهة المخزنة بين التحديثات، يحذر OpenClaw ويطلب تأكيدًا (استخدم `--yes` العام لتجاوز المطالبات). قد تسجل الإضافات أيضًا أوامرها الخاصة على المستوى الأعلى (مثال: `openclaw voicecall`).

## واجهة برمجة تطبيقات الإضافة (نظرة عامة)

تصدّر الإضافات إما:

-   دالة: `(api) => { ... }`
-   كائن: `{ id, name, configSchema, register(api) { ... } }`

يمكن لإضافات محرك السياق أيضًا تسجيل مدير سياق مملوك لوقت التشغيل:

```bash
export default function (api) {
  api.registerContextEngine("lossless-claw", () => ({
    info: { id: "lossless-claw", name: "Lossless Claw", ownsCompaction: true },
    async ingest() {
      return { ingested: true };
    },
    async assemble({ messages }) {
      return { messages, estimatedTokens: 0 };
    },
    async compact() {
      return { ok: true, compacted: false };
    },
  }));
}
```

ثم فعّلها في التكوين:

```json
{
  plugins: {
    slots: {
      contextEngine: "lossless-claw",
    },
  },
}
```

## خطافات الإضافة

يمكن للإضافات تسجيل خطافات في وقت التشغيل. هذا يسمح للإضافة بحزم الأتمتة القائمة على الأحداث دون تثبيت حزمة خطافات منفصلة.

### مثال

```bash
export default function register(api) {
  api.registerHook(
    "command:new",
    async () => {
      // منطق الخطاف هنا.
    },
    {
      name: "my-plugin.command-new",
      description: "يعمل عند استدعاء /new",
    },
  );
}
```

ملاحظات:

-   سجل الخطافات صراحةً عبر `api.registerHook(...)`.
-   لا تزال قواعد أهلية الخطاف تنطبق (متطلبات نظام التشغيل/الثنائيات/البيئة/التكوين).
-   تظهر الخطافات المدارة بواسطة الإضافة في `openclaw hooks list` مع `plugin:`.
-   لا يمكنك تفعيل/تعطيل الخطافات المدارة بواسطة الإضافة عبر `openclaw hooks`؛ فعّل/عطّل الإضافة بدلاً من ذلك.

### خطافات دورة حياة الوكيل (api.on)

للخطافات المحددة النوع لدورة حياة وقت التشغيل، استخدم `api.on(...)`:

```bash
export default function register(api) {
  api.on(
    "before_prompt_build",
    (event, ctx) => {
      return {
        prependSystemContext: "اتبع دليل أسلوب الشركة.",
      };
    },
    { priority: 10 },
  );
}
```

الخطافات المهمة لبناء المطالبة:

-   `before_model_resolve`: يعمل قبل تحميل الجلسة (`messages` غير متاحة). استخدم هذا لتجاوز `modelOverride` أو `providerOverride` بشكل حتمي.
-   `before_prompt_build`: يعمل بعد تحميل الجلسة (`messages` متاحة). استخدم هذا لتشكيل مدخل المطالبة.
-   `before_agent_start`: خطاف توافق قديم. فضّل الخطافين الصريحين أعلاه.

سياسة الخطاف التي يفرضها النواة:

-   يمكن للمشغلين تعطيل خطافات تحوير المطالبة لكل إضافة عبر `plugins.entries..hooks.allowPromptInjection: false`.
-   عند التعطيل، يمنع OpenClaw `before_prompt_build` ويتجاهل حقول تحوير المطالبة المُرجعة من `before_agent_start` القديم مع الحفاظ على `modelOverride` و `providerOverride` القديمين.

حقول نتيجة `before_prompt_build`:

-   `prependContext`: يضيف نصًا قبل مطالبة المستخدم لهذه الجولة. الأفضل للمحتوى المحدد للدورة أو الديناميكي.
-   `systemPrompt`: تجاوز كامل لمطالبة النظام.
-   `prependSystemContext`: يضيف نصًا قبل مطالبة النظام الحالية.
-   `appendSystemContext`: يضيف نصًا بعد مطالبة النظام الحالية.

ترتيب بناء المطالبة في وقت التشغيل المدمج:

1.  طبق `prependContext` على مطالبة المستخدم.
2.  طبق تجاوز `systemPrompt` عند توفيره.
3.  طبق `prependSystemContext + مطالبة النظام الحالية + appendSystemContext`.

ملاحظات الدمج والأولوية:

-   تعمل معالجات الخطاف حسب الأولوية (الأعلى أولاً).
-   لحقول السياق المدمجة، يتم تسلسل القيم بترتيب التنفيذ.
-   تطبق قيم `before_prompt_build` قبل قيم التراجع `before_agent_start` القديمة.

توجيهات الهجرة:

-   انقل التوجيهات الثابتة من `prependContext` إلى `prependSystemContext` (أو `appendSystemContext`) حتى يتمكن المزودون من تخزين محتوى بادئة النظام المستقر مؤقتًا.
-   احتفظ بـ `prependContext` لسياق ديناميكي محدد لكل دورة يجب أن يبقى مرتبطًا برسالة المستخدم.

## إضافات المزود (مصادقة النموذج)

يمكن للإضافات تسجيل تدفقات **مصادقة مزود النموذج** حتى يتمكن المستخدمون من تشغيل إعداد OAuth أو مفتاح API داخل OpenClaw (بدون نصوص خارجية). سجل مزودًا عبر `api.registerProvider(...)`. يعرض كل مزود طريقة مصادقة واحدة أو أكثر (OAuth، مفتاح API، رمز الجهاز، إلخ). تشغل هذه الطرق:

-   `openclaw models auth login --provider  [--method ]`

مثال:

```
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // شغّل تدفق OAuth وأعد ملفات تعريف المصادقة.
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

ملاحظات:

-   يتلقى `run` `ProviderAuthContext` مع مساعدات `prompter`, `runtime`, `openUrl`, و `oauth.createVpsAwareHandlers`.
-   أعد `configPatch` عندما تحتاج إلى إضافة نماذج افتراضية أو تكوين مزود.
-   أعد `defaultModel` حتى يتمكن `--set-default` من تحديث افتراضيات الوكيل.

### تسجيل قناة مراسلة

يمكن للإضافات تسجيل **إضافات قناة** تتصرف مثل القنوات المدمجة (WhatsApp، Telegram، إلخ). يعيش تكوين القناة تحت `channels.` ويتم التحقق منه بواسطة كود إضافة القناة الخاص بك.

```typescript
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "إضافة قناة تجريبية.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

ملاحظ