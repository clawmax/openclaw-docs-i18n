

  الأساسيات

  
# هندسة تكامل Pi

يصف هذا المستند كيفية تكامل OpenClaw مع [وكيل الترميز pi](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) والحزم المرتبطة به (`pi-ai`, `pi-agent-core`, `pi-tui`) لتمكين قدرات وكيل الذكاء الاصطناعي.

## نظرة عامة

يستخدم OpenClaw حزمة تطوير برامج (SDK) pi لتضمين وكيل ترميز ذكي اصطناعيًا في هندسة بوابة المراسلة الخاصة به. بدلاً من تشغيل pi كعملية فرعية أو استخدام وضع RPC، يقوم OpenClaw باستيراد وإنشاء مثيل لـ `AgentSession` الخاص بـ pi مباشرةً عبر `createAgentSession()`. يوفر هذا النهج المضمن:

-   تحكمًا كاملاً في دورة حياة الجلسة ومعالجة الأحداث
-   حقن أدوات مخصصة (المراسلة، الحماية، إجراءات محددة للقناة)
-   تخصيص موجه النظام لكل قناة/سياق
-   استمرارية الجلسة مع دعم التفرع/الضغط
-   تناوب ملف تعريف المصادقة متعدد الحسابات مع الانتقال للبديل الاحتياطي
-   تبديل النموذج المستقل عن المزود

## تبعيات الحزم

```json
{
  "@mariozechner/pi-agent-core": "0.49.3",
  "@mariozechner/pi-ai": "0.49.3",
  "@mariozechner/pi-coding-agent": "0.49.3",
  "@mariozechner/pi-tui": "0.49.3"
}
```

| الحزمة | الغرض |
| --- | --- |
| `pi-ai` | تجريدات LLM الأساسية: `Model`, `streamSimple`, أنواع الرسائل، واجهات برمجة تطبيقات المزود |
| `pi-agent-core` | حلقة الوكيل، تنفيذ الأداة، أنواع `AgentMessage` |
| `pi-coding-agent` | SDK عالي المستوى: `createAgentSession`, `SessionManager`, `AuthStorage`, `ModelRegistry`, الأدوات المدمجة |
| `pi-tui` | مكونات واجهة المستخدم الطرفية (تُستخدم في وضع TUI المحلي لـ OpenClaw) |

## هيكل الملفات

```
src/agents/
├── pi-embedded-runner.ts          # يعيد التصدير من pi-embedded-runner/
├── pi-embedded-runner/
│   ├── run.ts                     # نقطة الدخول الرئيسية: runEmbeddedPiAgent()
│   ├── run/
│   │   ├── attempt.ts             # منطق المحاولة الفردية مع إعداد الجلسة
│   │   ├── params.ts              # نوع RunEmbeddedPiAgentParams
│   │   ├── payloads.ts            # إنشاء حمولات الاستجابة من نتائج التشغيل
│   │   ├── images.ts              # حقن صور نموذج الرؤية
│   │   └── types.ts               # EmbeddedRunAttemptResult
│   ├── abort.ts                   # اكتشاف خطأ الإلغاء
│   ├── cache-ttl.ts               # تتبع مدة صلاحية ذاكرة التخزين المؤقت لتقليم السياق
│   ├── compact.ts                 # منطق الضغط اليدوي/التلقائي
│   ├── extensions.ts              # تحميل امتدادات pi للتشغيلات المضمنة
│   ├── extra-params.ts            # معلمات البث الخاصة بالمزود
│   ├── google.ts                  # إصلاحات ترتيب الأدوار لـ Google/Gemini
│   ├── history.ts                 # تحديد سجل المحادثة (DM مقابل المجموعة)
│   ├── lanes.ts                   # مسارات الجلسة/الأوامر العامة
│   ├── logger.ts                  # مسجل النظام الفرعي
│   ├── model.ts                   # تحديد النموذج عبر ModelRegistry
│   ├── runs.ts                    # تتبع التشغيل النشط، الإلغاء، قائمة الانتظار
│   ├── sandbox-info.ts            # معلومات الحماية لموجه النظام
│   ├── session-manager-cache.ts   # تخزين مثيلات SessionManager مؤقتًا
│   ├── session-manager-init.ts    # تهيئة ملف الجلسة
│   ├── system-prompt.ts           # منشئ موجه النظام
│   ├── tool-split.ts              # تقسيم الأدوات إلى مدمجة مقابل مخصصة
│   ├── types.ts                   # EmbeddedPiAgentMeta، EmbeddedPiRunResult
│   └── utils.ts                   # تعيين ThinkLevel، وصف الخطأ
├── pi-embedded-subscribe.ts       # اشتراك/إرسال أحداث الجلسة
├── pi-embedded-subscribe.types.ts # SubscribeEmbeddedPiSessionParams
├── pi-embedded-subscribe.handlers.ts # مصنع معالج الأحداث
├── pi-embedded-subscribe.handlers.lifecycle.ts
├── pi-embedded-subscribe.handlers.types.ts
├── pi-embedded-block-chunker.ts   # تجزئة رد الكتلة المتدفق
├── pi-embedded-messaging.ts       # تتبع أدوات المراسلة المرسلة
├── pi-embedded-helpers.ts         # تصنيف الأخطاء، التحقق من صحة الدور
├── pi-embedded-helpers/           # وحدات المساعدة
├── pi-embedded-utils.ts           # أدوات التنسيق
├── pi-tools.ts                    # createOpenClawCodingTools()
├── pi-tools.abort.ts              # تغليف AbortSignal للأدوات
├── pi-tools.policy.ts             # سياسة القائمة المسموح بها/الممنوعة للأدوات
├── pi-tools.read.ts               # تخصيصات أداة القراءة
├── pi-tools.schema.ts             # تطبيع مخطط الأداة
├── pi-tools.types.ts              # اسم نوع AnyAgentTool
├── pi-tool-definition-adapter.ts  # محول AgentTool -> ToolDefinition
├── pi-settings.ts                 # تجاوزات الإعدادات
├── pi-extensions/                 # امتدادات pi المخصصة
│   ├── compaction-safeguard.ts    # امتداد الحماية
│   ├── compaction-safeguard-runtime.ts
│   ├── context-pruning.ts         # امتداد تقليم السياق بناءً على مدة التخزين المؤقت
│   └── context-pruning/
├── model-auth.ts                  # تحديد ملف تعريف المصادقة
├── auth-profiles.ts               # مخزن الملفات الشخصية، التهدئة، الانتقال للبديل
├── model-selection.ts             # تحديد النموذج الافتراضي
├── models-config.ts               # إنشاء models.json
├── model-catalog.ts               # ذاكرة تخزين مؤقت لفهرس النماذج
├── context-window-guard.ts        # التحقق من صحة نافذة السياق
├── failover-error.ts              # فئة FailoverError
├── defaults.ts                    # DEFAULT_PROVIDER، DEFAULT_MODEL
├── system-prompt.ts               # buildAgentSystemPrompt()
├── system-prompt-params.ts        # تحديد معلمات موجه النظام
├── system-prompt-report.ts        # إنشاء تقرير التصحيح
├── tool-summaries.ts              # ملخصات وصف الأداة
├── tool-policy.ts                 # تحديد سياسة الأداة
├── transcript-policy.ts           # سياسة التحقق من صحة النص
├── skills.ts                      # لقطة المهارات/بناء الموجه
├── skills/                        # النظام الفرعي للمهارات
├── sandbox.ts                     # تحديد سياق الحماية
├── sandbox/                       # النظام الفرعي للحماية
├── channel-tools.ts               # حقن أدوات خاصة بالقناة
├── openclaw-tools.ts              # أدوات خاصة بـ OpenClaw
├── bash-tools.ts                  # أدوات exec/process
├── apply-patch.ts                 # أداة apply_patch (OpenAI)
├── tools/                         # تنفيذات الأدوات الفردية
│   ├── browser-tool.ts
│   ├── canvas-tool.ts
│   ├── cron-tool.ts
│   ├── discord-actions*.ts
│   ├── gateway-tool.ts
│   ├── image-tool.ts
│   ├── message-tool.ts
│   ├── nodes-tool.ts
│   ├── session*.ts
│   ├── slack-actions.ts
│   ├── telegram-actions.ts
│   ├── web-*.ts
│   └── whatsapp-actions.ts
└── ...
```

## تدفق التكامل الأساسي

### 1\. تشغيل وكيل مضمن

نقطة الدخول الرئيسية هي `runEmbeddedPiAgent()` في `pi-embedded-runner/run.ts`:

```typescript
import { runEmbeddedPiAgent } from "./agents/pi-embedded-runner.js";

const result = await runEmbeddedPiAgent({
  sessionId: "user-123",
  sessionKey: "main:whatsapp:+1234567890",
  sessionFile: "/path/to/session.jsonl",
  workspaceDir: "/path/to/workspace",
  config: openclawConfig,
  prompt: "Hello, how are you?",
  provider: "anthropic",
  model: "claude-sonnet-4-20250514",
  timeoutMs: 120_000,
  runId: "run-abc",
  onBlockReply: async (payload) => {
    await sendToChannel(payload.text, payload.mediaUrls);
  },
});
```

### 2\. إنشاء الجلسة

داخل `runEmbeddedAttempt()` (التي تستدعيها `runEmbeddedPiAgent()`)، يتم استخدام SDK الخاص بـ pi:

```typescript
import {
  createAgentSession,
  DefaultResourceLoader,
  SessionManager,
  SettingsManager,
} from "@mariozechner/pi-coding-agent";

const resourceLoader = new DefaultResourceLoader({
  cwd: resolvedWorkspace,
  agentDir,
  settingsManager,
  additionalExtensionPaths,
});
await resourceLoader.reload();

const { session } = await createAgentSession({
  cwd: resolvedWorkspace,
  agentDir,
  authStorage: params.authStorage,
  modelRegistry: params.modelRegistry,
  model: params.model,
  thinkingLevel: mapThinkingLevel(params.thinkLevel),
  tools: builtInTools,
  customTools: allCustomTools,
  sessionManager,
  settingsManager,
  resourceLoader,
});

applySystemPromptOverrideToSession(session, systemPromptOverride);
```

### 3\. الاشتراك في الأحداث

تقوم `subscribeEmbeddedPiSession()` بالاشتراك في أحداث `AgentSession` الخاصة بـ pi:

```typescript
const subscription = subscribeEmbeddedPiSession({
  session: activeSession,
  runId: params.runId,
  verboseLevel: params.verboseLevel,
  reasoningMode: params.reasoningLevel,
  toolResultFormat: params.toolResultFormat,
  onToolResult: params.onToolResult,
  onReasoningStream: params.onReasoningStream,
  onBlockReply: params.onBlockReply,
  onPartialReply: params.onPartialReply,
  onAgentEvent: params.onAgentEvent,
});
```

تشمل الأحداث التي يتم معالجتها:

-   `message_start` / `message_end` / `message_update` (النص/التفكير المتدفق)
-   `tool_execution_start` / `tool_execution_update` / `tool_execution_end`
-   `turn_start` / `turn_end`
-   `agent_start` / `agent_end`
-   `auto_compaction_start` / `auto_compaction_end`

### 4\. الإرسال

بعد الإعداد، يتم إرسال الموجه إلى الجلسة:

```typescript
await session.prompt(effectivePrompt, { images: imageResult.images });
```

تتعامل SDK مع حلقة الوكيل الكاملة: الإرسال إلى LLM، تنفيذ استدعاءات الأداة، بث الردود. حقن الصور محلي للموجه: يقوم OpenClaw بتحميل مراجع الصور من الموجه الحالي ويمررها عبر `images` لتلك الجولة فقط. لا يعيد فحص أدوار السجل القديمة لإعادة حقن حمولات الصور.

## هندسة الأداة

### خط أنابيب الأداة

1.  **الأدوات الأساسية**: `codingTools` الخاصة بـ pi (قراءة، bash، تحرير، كتابة)
2.  **البدائل المخصصة**: يستبدل OpenClaw bash بـ `exec`/`process`، ويخصص القراءة/التحرير/الكتابة للحماية
3.  **أدوات OpenClaw**: المراسلة، المتصفح، اللوحة القماشية، الجلسات، cron، البوابة، إلخ.
4.  **أدوات القناة**: أدوات الإجراءات الخاصة بـ Discord/Telegram/Slack/WhatsApp
5.  **التصفية حسب السياسة**: تصفية الأدوات حسب ملف التعريف، المزود، الوكيل، المجموعة، سياسات الحماية
6.  **تطبيع المخطط**: تنظيف المخططات لتناسب خصوصيات Gemini/OpenAI
7.  **تغليف AbortSignal**: تغليف الأدوات لاحترام إشارات الإلغاء

### محول تعريف الأداة

يحتوي `AgentTool` الخاص بـ pi-agent-core على توقيع `execute` مختلف عن `ToolDefinition` الخاص بـ pi-coding-agent. يقوم المحول في `pi-tool-definition-adapter.ts` بالربط بينهما:

```bash
export function toToolDefinitions(tools: AnyAgentTool[]): ToolDefinition[] {
  return tools.map((tool) => ({
    name: tool.name,
    label: tool.label ?? name,
    description: tool.description ?? "",
    parameters: tool.parameters,
    execute: async (toolCallId, params, onUpdate, _ctx, signal) => {
      // توقيع pi-coding-agent يختلف عن pi-agent-core
      return await tool.execute(toolCallId, params, signal, onUpdate);
    },
  }));
}
```

### استراتيجية تقسيم الأداة

تمرر `splitSdkTools()` جميع الأدوات عبر `customTools`:

```bash
export function splitSdkTools(options: { tools: AnyAgentTool[]; sandboxEnabled: boolean }) {
  return {
    builtInTools: [], // فارغ. نتجاوز كل شيء
    customTools: toToolDefinitions(options.tools),
  };
}
```

يضمن هذا بقاء تصفية السياسة الخاصة بـ OpenClaw، وتكامل الحماية، ومجموعة الأدوات الموسعة متسقة عبر جميع المزودين.

## بناء موجه النظام

يتم بناء موجه النظام في `buildAgentSystemPrompt()` (`system-prompt.ts`). يقوم بتجميع موجه كامل بأقسام تشمل الأدوات، أسلوب استدعاء الأداة، حواجز الأمان، مرجع OpenClaw CLI، المهارات، المستندات، مساحة العمل، الحماية، المراسلة، علامات الرد، الصوت، الردود الصامتة، نبضات القلب، بيانات وصفية وقت التشغيل، بالإضافة إلى الذاكرة والتفاعلات عند تمكينها، ومحتوى موجه النظام الإضافي وملفات السياق الاختيارية. يتم تقليم الأقسام لوضع الموجه الأدنى المستخدم بواسطة الوكلاء الفرعيين. يتم تطبيق الموجه بعد إنشاء الجلسة عبر `applySystemPromptOverrideToSession()`:

```typescript
const systemPromptOverride = createSystemPromptOverride(appendPrompt);
applySystemPromptOverrideToSession(session, systemPromptOverride);
```

## إدارة الجلسات

### ملفات الجلسة

الجلسات هي ملفات JSONL ذات بنية شجرية (ربط id/parentId). يتعامل `SessionManager` الخاص بـ pi مع الاستمرارية:

```typescript
const sessionManager = SessionManager.open(params.sessionFile);
```

يقوم OpenClaw بتغليف هذا بـ `guardSessionManager()` لسلامة نتائج الأداة.

### تخزين الجلسات مؤقتًا

تقوم `session-manager-cache.ts` بتخزين مثيلات SessionManager مؤقتًا لتجنب تحليل الملفات المتكرر:

```typescript
await prewarmSessionFile(params.sessionFile);
sessionManager = SessionManager.open(params.sessionFile);
trackSessionManagerAccess(params.sessionFile);
```

### تحديد السجل

تقوم `limitHistoryTurns()` بقص سجل المحادثة بناءً على نوع القناة (DM مقابل المجموعة).

### الضغط

يتم تشغيل الضغط التلقائي عند تجاوز سعة السياق. يتعامل `compactEmbeddedPiSessionDirect()` مع الضغط اليدوي:

```typescript
const compactResult = await compactEmbeddedPiSessionDirect({
  sessionId, sessionFile, provider, model, ...
});
```

## المصادقة وتحديد النموذج

### ملفات تعريف المصادقة

يحتفظ OpenClaw بمخزن لملفات تعريف المصادقة يحتوي على مفاتيح واجهة برمجة تطبيقات متعددة لكل مزود:

```typescript
const authStore = ensureAuthProfileStore(agentDir, { allowKeychainPrompt: false });
const profileOrder = resolveAuthProfileOrder({ cfg, store: authStore, provider, preferredProfile });
```

تتناوب ملفات التعريف عند الفشل مع تتبع فترة التهدئة:

```typescript
await markAuthProfileFailure({ store, profileId, reason, cfg, agentDir });
const rotated = await advanceAuthProfile();
```

### تحديد النموذج

```typescript
import { resolveModel } from "./pi-embedded-runner/model.js";

const { model, error, authStorage, modelRegistry } = resolveModel(
  provider,
  modelId,
  agentDir,
  config,
);

// يستخدم ModelRegistry و AuthStorage الخاص بـ pi
authStorage.setRuntimeApiKey(model.provider, apiKeyInfo.apiKey);
```

### الانتقال للبديل

يؤدي `FailoverError` إلى التراجع للنموذج البديل عند التكوين:

```
if (fallbackConfigured && isFailoverErrorMessage(errorText)) {
  throw new FailoverError(errorText, {
    reason: promptFailoverReason ?? "unknown",
    provider,
    model: modelId,
    profileId,
    status: resolveFailoverStatus(promptFailoverReason),
  });
}
```

## امتدادات Pi

يقوم OpenClaw بتحميل امتدادات pi مخصصة لسلوك متخصص:

### حماية الضغط

يضيف `src/agents/pi-extensions/compaction-safeguard.ts` حواجز حماية للضغط، بما في ذلك ميزانية الرموز التكيفية بالإضافة إلى ملخصات فشل الأداة وعمليات الملفات:

```
if (resolveCompactionMode(params.cfg) === "safeguard") {
  setCompactionSafeguardRuntime(params.sessionManager, { maxHistoryShare });
  paths.push(resolvePiExtensionPath("compaction-safeguard"));
}
```

### تقليم السياق

ينفذ `src/agents/pi-extensions/context-pruning.ts` تقليم السياق بناءً على مدة صلاحية ذاكرة التخزين المؤقت:

```
if (cfg?.agents?.defaults?.contextPruning?.mode === "cache-ttl") {
  setContextPruningRuntime(params.sessionManager, {
    settings,
    contextWindowTokens,
    isToolPrunable,
    lastCacheTouchAt,
  });
  paths.push(resolvePiExtensionPath("context-pruning"));
}
```

## البث وردود الكتلة

### تجزئة الكتلة

يدير `EmbeddedBlockChunker` تحويل النص المتدفق إلى كتل رد منفصلة:

```typescript
const blockChunker = blockChunking ? new EmbeddedBlockChunker(blockChunking) : null;
```

### إزالة علامات التفكير/النهائي

يتم معالجة الإخراج المتدفق لإزالة كتل ``/`` واستخراج محتوى ``:

```typescript
const stripBlockTags = (text: string, state: { thinking: boolean; final: boolean }) => {
  // إزالة محتوى <think>...</think>
  // إذا كان enforceFinalTag، أعد فقط محتوى <final>...</final>
};
```

### توجيهات الرد

يتم تحليل توجيهات الرد مثل `[[media:url]]`، `[[voice]]`، `[[reply:id]]` واستخراجها:

```typescript
const { text: cleanedText, mediaUrls, audioAsVoice, replyToId } = consumeReplyDirectives(chunk);
```

## معالجة الأخطاء

### تصنيف الخطأ

تصنف `pi-embedded-helpers.ts` الأخطاء للمعالجة المناسبة:

```
isContextOverflowError(errorText)     // السياق كبير جدًا
isCompactionFailureError(errorText)   // فشل الضغط
isAuthAssistantError(lastAssistant)   // فشل المصادقة
isRateLimitAssistantError(...)        // تجاوز معدل الطلبات
isFailoverAssistantError(...)         // يجب الانتقال للبديل
classifyFailoverReason(errorText)     // "auth" | "rate_limit" | "quota" | "timeout" | ...
```

### التراجع لمستوى التفكير

إذا كان مستوى التفكير غير مدعوم، فإنه يتراجع:

```typescript
const fallbackThinking = pickFallbackThinkingLevel({
  message: errorText,
  attempted: attemptedThinking,
});
if (fallbackThinking) {
  thinkLevel = fallbackThinking;
  continue;
}
```

## تكامل الحماية

عند تمكين وضع الحماية، يتم تقييد الأدوات والمسارات:

```typescript
const sandbox = await resolveSandboxContext({
  config: params.config,
  sessionKey: sandboxSessionKey,
  workspaceDir: resolvedWorkspace,
});

if (sandboxRoot) {
  // استخدام أدوات القراءة/التحرير/الكتابة المحمية
  // يعمل Exec في الحاوية
  // يستخدم المتصفح عنوان URL الجسر
}
```

## المعالجة الخاصة بالمزود

### Anthropic

-   تنظيف سلسلة الرفض السحرية
-   التحقق من صحة الدور للأدوار المتتالية
-   توافق معامل Claude Code

### Google/Gemini

-   إصلاحات ترتيب الأدوار (`applyGoogleTurnOrderingFix`)
-   تعقيم مخطط الأداة (`sanitizeToolsForGoogle`)
-   تعقيم سجل الجلسة (`sanitizeSessionHistory`)

### OpenAI

-   أداة `apply_patch` لنماذج Codex
-   معالجة تخفيض مستوى التفكير

## تكامل TUI

يحتوي OpenClaw أيضًا على وضع TUI محلي يستخدم مكونات pi-tui مباشرة:

```bash
// src/tui/tui.ts
import { ... } from "@mariozechner/pi-tui";
```

يوفر هذا تجربة الطرفية التفاعلية المشابهة للوضع الأصلي لـ pi.

## الاختلافات الرئيسية عن واجهة سطر أوامر Pi

| الجانب | واجهة سطر أوامر Pi | OpenClaw المضمن |
| --- | --- | --- |
| الاستدعاء | أمر `pi` / RPC | SDK عبر `createAgentSession()` |
| الأدوات | أدوات الترميز الافتراضية | مجموعة أدوات OpenClaw المخصصة |
| موجه النظام | AGENTS.md + الموجهات | ديناميكي لكل قناة/سياق |
| تخزين الجلسة | `~/.pi/agent/sessions/` | `~/.openclaw/agents//sessions/` (أو `$OPENCLAW_STATE_DIR/agents//sessions/`) |
| المصادقة | بيانات اعتماد واحدة | متعدد الملفات الشخصية مع التناوب |
| الامتدادات | محملة من القرص | برمجي + مسارات القرص |
| معالجة الأحداث | عرض TUI | قائم على ردود النداء (onBlockReply، إلخ) |

## اعتبارات مستقبلية

مجالات لإعادة العمل المحتملة:

1.  **محاذاة توقيع الأداة**: حاليًا يتم التكيف بين توقيعات pi-agent-core و pi-coding-agent
2.  **تغليف مدير الجلسة**: تضيف `guardSessionManager` الأمان ولكنها تزيد التعقيد
3.  **تحميل الامتداد**: يمكن استخدام `ResourceLoader` الخاص بـ pi بشكل أكثر مباشرة
4.  **تعقيد معالج البث**: أصبح `subscribeEmbeddedPiSession` كبيرًا
5.  **خصوصيات المزود**: العديد من مسارات التعليمات البرمجية الخاصة بالمزود التي يمكن لـ pi التعامل معها

## الاختبارات

يغطي تكامل Pi مجموعات الاختبار التالية:

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-auth-json.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-embedded-helpers*.test.ts`
-   `src/agents/pi-embedded-runner*.test.ts`
-   `src/agents/pi-embedded-runner/**/*.test.ts`
-   `src/agents/pi-embedded-subscribe*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-tool-definition-adapter*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-extensions/**/*.test.ts`

اختبار مباشر/اختياري:

-   `src/agents/pi-embedded-runner-extraparams.live.test.ts` (تمكين `OPENCLAW_LIVE_TEST=1`)

لأوامر التشغيل الحالية، راجع [سير عمل تطوير Pi](./pi-dev.md).

[هندسة البوابة](./concepts/architecture.md)