

  技能

  
# 插件

## 快速开始（初次接触插件？）

插件只是一个**小型代码模块**，用于通过额外功能（命令、工具和网关 RPC）扩展 OpenClaw。大多数情况下，当你需要一个 OpenClaw 核心尚未内置的功能（或者你想将可选功能排除在主安装之外）时，你会使用插件。快速路径：

1.  查看已加载的内容：

```bash
openclaw plugins list
```

2.  安装一个官方插件（例如：语音通话）：

```bash
openclaw plugins install @openclaw/voice-call
```

Npm 规范是**仅限注册表**（包名 + 可选的**精确版本**或**分发标签**）。Git/URL/文件规范和 semver 范围会被拒绝。裸规范和 `@latest` 保持在稳定轨道上。如果 npm 将其中任何一个解析为预发布版本，OpenClaw 会停止并提示你使用预发布标签（如 `@beta`/`@rc`）或精确的预发布版本明确选择加入。

3.  重启网关，然后在 `plugins.entries..config` 下进行配置。

具体插件示例请参阅[语音通话](../plugins/voice-call.md)。寻找第三方列表？请参阅[社区插件](../plugins/community.md)。

## 可用插件（官方）

-   自 2026.1.15 起，Microsoft Teams 仅作为插件提供；如果你使用 Teams，请安装 `@openclaw/msteams`。
-   记忆（核心）— 捆绑的记忆搜索插件（默认通过 `plugins.slots.memory` 启用）
-   记忆（LanceDB）— 捆绑的长期记忆插件（自动回忆/捕获；设置 `plugins.slots.memory = "memory-lancedb"`）
-   [语音通话](../plugins/voice-call.md) — `@openclaw/voice-call`
-   [Zalo 个人版](../plugins/zalouser.md) — `@openclaw/zalouser`
-   [Matrix](../channels/matrix.md) — `@openclaw/matrix`
-   [Nostr](../channels/nostr.md) — `@openclaw/nostr`
-   [Zalo](../channels/zalo.md) — `@openclaw/zalo`
-   [Microsoft Teams](../channels/msteams.md) — `@openclaw/msteams`
-   Google Antigravity OAuth（提供商认证）— 捆绑为 `google-antigravity-auth`（默认禁用）
-   Gemini CLI OAuth（提供商认证）— 捆绑为 `google-gemini-cli-auth`（默认禁用）
-   Qwen OAuth（提供商认证）— 捆绑为 `qwen-portal-auth`（默认禁用）
-   Copilot 代理（提供商认证）— 本地 VS Code Copilot 代理桥接；区别于内置的 `github-copilot` 设备登录（捆绑，默认禁用）

OpenClaw 插件是**TypeScript 模块**，通过 jiti 在运行时加载。**配置验证不执行插件代码**；它使用插件清单和 JSON Schema 代替。请参阅[插件清单](../plugins/manifest.md)。插件可以注册：

-   网关 RPC 方法
-   网关 HTTP 路由
-   智能体工具
-   CLI 命令
-   后台服务
-   上下文引擎
-   可选的配置验证
-   **技能**（通过在插件清单中列出 `skills` 目录）
-   **自动回复命令**（无需调用 AI 智能体即可执行）

插件与网关**同进程**运行，因此请将其视为受信任的代码。工具编写指南：[插件智能体工具](../plugins/agent-tools.md)。

## 运行时助手

插件可以通过 `api.runtime` 访问选定的核心助手。用于电话 TTS：

```typescript
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

注意：

-   使用核心 `messages.tts` 配置（OpenAI 或 ElevenLabs）。
-   返回 PCM 音频缓冲区 + 采样率。插件必须为提供商重新采样/编码。
-   电话场景不支持 Edge TTS。

对于 STT/转录，插件可以调用：

```typescript
const { text } = await api.runtime.stt.transcribeAudioFile({
  filePath: "/tmp/inbound-audio.ogg",
  cfg: api.config,
  // 当 MIME 类型无法可靠推断时可选：
  mime: "audio/ogg",
});
```

注意：

-   使用核心媒体理解音频配置（`tools.media.audio`）和提供商回退顺序。
-   当未产生转录输出时（例如跳过/不支持的输入），返回 `{ text: undefined }`。

## 网关 HTTP 路由

插件可以通过 `api.registerHttpRoute(...)` 暴露 HTTP 端点。

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

路由字段：

-   `path`：网关 HTTP 服务器下的路由路径。
-   `auth`：必需。使用 `"gateway"` 要求正常的网关认证，或使用 `"plugin"` 进行插件管理的认证/webhook 验证。
-   `match`：可选。`"exact"`（默认）或 `"prefix"`。
-   `replaceExisting`：可选。允许同一插件替换其自身现有的路由注册。
-   `handler`：当路由处理了请求时返回 `true`。

注意：

-   `api.registerHttpHandler(...)` 已过时。请使用 `api.registerHttpRoute(...)`。
-   插件路由必须显式声明 `auth`。
-   精确的 `path + match` 冲突会被拒绝，除非 `replaceExisting: true`，并且一个插件不能替换另一个插件的路由。
-   具有不同 `auth` 级别的重叠路由会被拒绝。仅在相同认证级别上保持 `exact`/`prefix` 回退链。

## 插件 SDK 导入路径

编写插件时，请使用 SDK 子路径，而不是整体导入 `openclaw/plugin-sdk`：

-   `openclaw/plugin-sdk/core` 用于通用插件 API、提供商认证类型和共享助手。
-   `openclaw/plugin-sdk/compat` 用于捆绑/内部插件代码，这些代码需要比 `core` 更广泛的共享运行时助手。
-   `openclaw/plugin-sdk/telegram` 用于 Telegram 频道插件。
-   `openclaw/plugin-sdk/discord` 用于 Discord 频道插件。
-   `openclaw/plugin-sdk/slack` 用于 Slack 频道插件。
-   `openclaw/plugin-sdk/signal` 用于 Signal 频道插件。
-   `openclaw/plugin-sdk/imessage` 用于 iMessage 频道插件。
-   `openclaw/plugin-sdk/whatsapp` 用于 WhatsApp 频道插件。
-   `openclaw/plugin-sdk/line` 用于 LINE 频道插件。
-   `openclaw/plugin-sdk/msteams` 用于捆绑的 Microsoft Teams 插件接口。
-   捆绑的扩展特定子路径也可用：`openclaw/plugin-sdk/acpx`, `openclaw/plugin-sdk/bluebubbles`, `openclaw/plugin-sdk/copilot-proxy`, `openclaw/plugin-sdk/device-pair`, `openclaw/plugin-sdk/diagnostics-otel`, `openclaw/plugin-sdk/diffs`, `openclaw/plugin-sdk/feishu`, `openclaw/plugin-sdk/google-gemini-cli-auth`, `openclaw/plugin-sdk/googlechat`, `openclaw/plugin-sdk/irc`, `openclaw/plugin-sdk/llm-task`, `openclaw/plugin-sdk/lobster`, `openclaw/plugin-sdk/matrix`, `openclaw/plugin-sdk/mattermost`, `openclaw/plugin-sdk/memory-core`, `openclaw/plugin-sdk/memory-lancedb`, `openclaw/plugin-sdk/minimax-portal-auth`, `openclaw/plugin-sdk/nextcloud-talk`, `openclaw/plugin-sdk/nostr`, `openclaw/plugin-sdk/open-prose`, `openclaw/plugin-sdk/phone-control`, `openclaw/plugin-sdk/qwen-portal-auth`, `openclaw/plugin-sdk/synology-chat`, `openclaw/plugin-sdk/talk-voice`, `openclaw/plugin-sdk/test-utils`, `openclaw/plugin-sdk/thread-ownership`, `openclaw/plugin-sdk/tlon`, `openclaw/plugin-sdk/twitch`, `openclaw/plugin-sdk/voice-call`, `openclaw/plugin-sdk/zalo`, 和 `openclaw/plugin-sdk/zalouser`。

兼容性说明：

-   `openclaw/plugin-sdk` 对现有外部插件仍然支持。
-   新的和迁移的捆绑插件应使用频道或扩展特定的子路径；通用接口使用 `core`，仅当需要更广泛的共享助手时才使用 `compat`。

## 只读频道检查

如果你的插件注册了一个频道，建议在实现 `resolveAccount(...)` 的同时实现 `plugin.config.inspectAccount(cfg, accountId)`。原因：

-   `resolveAccount(...)` 是运行时路径。它允许假设凭证已完全就绪，并且当必需的密钥缺失时可以快速失败。
-   只读命令路径，如 `openclaw status`、`openclaw status --all`、`openclaw channels status`、`openclaw channels resolve` 以及医生/配置修复流程，不应仅仅为了描述配置而需要实例化运行时凭证。

推荐的 `inspectAccount(...)` 行为：

-   仅返回描述性的账户状态。
-   保留 `enabled` 和 `configured`。
-   在相关时包含凭证来源/状态字段，例如：
    -   `tokenSource`, `tokenStatus`
    -   `botTokenSource`, `botTokenStatus`
    -   `appTokenSource`, `appTokenStatus`
    -   `signingSecretSource`, `signingSecretStatus`
-   你不需要返回原始令牌值来报告只读可用性。返回 `tokenStatus: "available"`（以及匹配的 source 字段）对于状态类命令就足够了。
-   当凭证通过 SecretRef 配置但在当前命令路径中不可用时，使用 `configured_unavailable`。

这允许只读命令报告“已配置但在此命令路径中不可用”，而不是崩溃或错误地报告账户未配置。性能说明：

-   插件发现和清单元数据使用短期的进程内缓存来减少突发性的启动/重载工作。
-   设置 `OPENCLAW_DISABLE_PLUGIN_DISCOVERY_CACHE=1` 或 `OPENCLAW_DISABLE_PLUGIN_MANIFEST_CACHE=1` 来禁用这些缓存。
-   使用 `OPENCLAW_PLUGIN_DISCOVERY_CACHE_MS` 和 `OPENCLAW_PLUGIN_MANIFEST_CACHE_MS` 调整缓存窗口。

## 发现与优先级

OpenClaw 按顺序扫描：

1.  配置路径

-   `plugins.load.paths`（文件或目录）

2.  工作区扩展

-   `/.openclaw/extensions/*.ts`
-   `/.openclaw/extensions/*/index.ts`

3.  全局扩展

-   `~/.openclaw/extensions/*.ts`
-   `~/.openclaw/extensions/*/index.ts`

4.  捆绑扩展（随 OpenClaw 分发，大多默认禁用）

-   `/extensions/*`

大多数捆绑插件必须通过 `plugins.entries..enabled` 或 `openclaw plugins enable ` 显式启用。默认启用的捆绑插件例外：

-   `device-pair`
-   `phone-control`
-   `talk-voice`
-   活动的记忆槽插件（默认槽：`memory-core`）

已安装的插件默认启用，但可以以相同方式禁用。加固说明：

-   如果 `plugins.allow` 为空且发现了非捆绑插件，OpenClaw 会记录一个包含插件 ID 和来源的启动警告。
-   候选路径在准入发现前会进行安全检查。OpenClaw 在以下情况下会阻止候选路径：
    -   扩展入口解析到插件根目录之外（包括符号链接/路径遍历逃逸），
    -   插件根目录/源路径全局可写，
    -   对于非捆绑插件，路径所有权可疑（POSIX 所有者既不是当前 uid 也不是 root）。
-   加载的没有安装/加载路径来源的非捆绑插件会发出警告，以便你可以固定信任（`plugins.allow`）或安装跟踪（`plugins.installs`）。

每个插件必须在其根目录中包含一个 `openclaw.plugin.json` 文件。如果路径指向一个文件，插件根目录是该文件所在的目录，并且必须包含清单。如果多个插件解析到相同的 ID，则按上述顺序第一个匹配的获胜，优先级较低的副本将被忽略。

### 包包

插件目录可以包含一个带有 `openclaw.extensions` 的 `package.json`：

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

每个条目成为一个插件。如果包列出了多个扩展，插件 ID 变为 `name/`。如果你的插件导入 npm 依赖项，请在该目录中安装它们，以便 `node_modules` 可用（`npm install` / `pnpm install`）。安全护栏：每个 `openclaw.extensions` 条目在符号链接解析后必须保持在插件目录内。逃逸包目录的条目会被拒绝。安全说明：`openclaw plugins install` 使用 `npm install --ignore-scripts` 安装插件依赖项（无生命周期脚本）。保持插件依赖树为“纯 JS/TS”，避免需要 `postinstall` 构建的包。

### 频道目录元数据

频道插件可以通过 `openclaw.channel` 和 `openclaw.install` 宣传入门元数据和安装提示。这使核心目录保持无数据。示例：

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk（自托管）",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "通过 Nextcloud Talk webhook 机器人进行自托管聊天。",
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

OpenClaw 也可以合并**外部频道目录**（例如，MPM 注册表导出）。将 JSON 文件放在以下位置之一：

-   `~/.openclaw/mpm/plugins.json`
-   `~/.openclaw/mpm/catalog.json`
-   `~/.openclaw/plugins/catalog.json`

或者将 `OPENCLAW_PLUGIN_CATALOG_PATHS`（或 `OPENCLAW_MPM_CATALOG_PATHS`）指向一个或多个 JSON 文件（逗号/分号/`PATH` 分隔）。每个文件应包含 `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`。

## 插件 ID

默认插件 ID：

-   包包：`package.json` 中的 `name`
-   独立文件：文件基本名（`~/.../voice-call.ts` → `voice-call`）

如果插件导出 `id`，OpenClaw 会使用它，但当其与配置的 ID 不匹配时会发出警告。

## 配置

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

字段：

-   `enabled`：主开关（默认：true）
-   `allow`：允许列表（可选）
-   `deny`：拒绝列表（可选；拒绝优先）
-   `load.paths`：额外的插件文件/目录
-   `slots`：独占槽选择器，如 `memory` 和 `contextEngine`
-   `entries.`：每个插件的开关 + 配置

配置更改**需要重启网关**。验证规则（严格）：

-   `entries`、`allow`、`deny` 或 `slots` 中的未知插件 ID 是**错误**。
-   `channels.` 中的未知键是**错误**，除非插件清单声明了该频道 ID。
-   插件配置使用嵌入在 `openclaw.plugin.json` 中的 JSON Schema（`configSchema`）进行验证。
-   如果插件被禁用，其配置会被保留，并发出**警告**。

## 插件槽（独占类别）

某些插件类别是**独占的**（一次只能有一个处于活动状态）。使用 `plugins.slots` 来选择哪个插件拥有该槽：

```json
{
  plugins: {
    slots: {
      memory: "memory-core", // 或 "none" 以禁用记忆插件
      contextEngine: "legacy", // 或插件 ID，如 "lossless-claw"
    },
  },
}
```

支持的独占槽：

-   `memory`：活动的记忆插件（`"none"` 禁用记忆插件）
-   `contextEngine`：活动的上下文引擎插件（`"legacy"` 是内置默认值）

如果多个插件声明 `kind: "memory"` 或 `kind: "context-engine"`，则只有选定的插件为该槽加载。其他插件会被禁用并带有诊断信息。

### 上下文引擎插件

上下文引擎插件拥有会话上下文的编排，用于摄取、组装和压缩。通过 `api.registerContextEngine(id, factory)` 从你的插件中注册它们，然后使用 `plugins.slots.contextEngine` 选择活动的引擎。当你的插件需要替换或扩展默认的上下文管道，而不仅仅是添加记忆搜索或钩子时，请使用此功能。

## 控制 UI（模式 + 标签）

控制 UI 使用 `config.schema`（JSON Schema + `uiHints`）来渲染更好的表单。OpenClaw 在运行时根据发现的插件增强 `uiHints`：

-   为 `plugins.entries.` / `.enabled` / `.config` 添加每个插件的标签
-   在以下位置合并可选的插件提供的配置字段提示：`plugins.entries..config.`

如果你希望插件配置字段显示良好的标签/占位符（并将密钥标记为敏感），请在插件清单中提供 `uiHints` 以及你的 JSON Schema。示例：

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
    "apiKey": { "label": "API 密钥", "sensitive": true },
    "region": { "label": "区域", "placeholder": "us-east-1" }
  }
}
```

## CLI

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # 将本地文件/目录复制到 ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # 相对路径可以
openclaw plugins install ./plugin.tgz           # 从本地 tarball 安装
openclaw plugins install ./plugin.zip           # 从本地 zip 安装
openclaw plugins install -l ./extensions/voice-call # 链接（不复制）用于开发
openclaw plugins install @openclaw/voice-call # 从 npm 安装
openclaw plugins install @openclaw/voice-call --pin # 存储精确解析的 name@version
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` 仅适用于在 `plugins.installs` 下跟踪的 npm 安装。如果存储的完整性元数据在更新之间发生变化，OpenClaw 会警告并要求确认（使用全局 `--yes` 绕过提示）。插件也可以注册自己的顶级命令（例如：`openclaw voicecall`）。

## 插件 API（概述）

插件导出以下之一：

-   一个函数：`(api) => { ... }`
-   一个对象：`{ id, name, configSchema, register(api) { ... } }`

上下文引擎插件还可以注册一个运行时拥有的上下文管理器：

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

然后在配置中启用它：

```json
{
  plugins: {
    slots: {
      contextEngine: "lossless-claw",
    },
  },
}
```

## 插件钩子

插件可以在运行时注册钩子。这允许插件捆绑事件驱动的自动化，而无需单独安装钩子包。

### 示例

```bash
export default function register(api) {
  api.registerHook(
    "command:new",
    async () => {
      // 钩子逻辑写在这里。
    },
    {
      name: "my-plugin.command-new",
      description: "当 /new 被调用时运行",
    },
  );
}
```

注意：

-   通过 `api.registerHook(...)` 显式注册钩子。
-   钩子资格规则仍然适用（OS/二进制文件/环境/配置要求）。
-   插件管理的钩子会出现在 `openclaw hooks list` 中，并带有 `plugin:`。
-   你不能通过 `openclaw hooks` 启用/禁用插件管理的钩子；而是启用/禁用插件本身。

### 智能体生命周期钩子 (api.on)

对于类型化的运行时生命周期钩子，使用 `api.on(...)`：

```bash
export default function register(api) {
  api.on(
    "before_prompt_build",
    (event, ctx) => {
      return {
        prependSystemContext: "遵循公司风格指南。",
      };
    },
    { priority: 10 },
  );
}
```

提示构建的重要钩子：

-   `before_model_resolve`：在会话加载之前运行（`messages` 不可用）。使用此钩子确定性地覆盖 `modelOverride` 或 `providerOverride`。
-   `before_prompt_build`：在会话加载之后运行（`messages` 可用）。使用此钩子来塑造提示输入。
-   `before_agent_start`：旧版兼容钩子。优先使用上述两个显式钩子。

核心强制执行的钩子策略：

-   操作员可以通过 `plugins.entries..hooks.allowPromptInjection: false` 按插件禁用提示变异钩子。
-   禁用时，OpenClaw 会阻止 `before_prompt_build` 并忽略从旧版 `before_agent_start` 返回的提示变异字段，同时保留旧版的 `modelOverride` 和 `providerOverride`。

`before_prompt_build` 结果字段：

-   `prependContext`：将文本预置到本次运行的用户提示中。最适合特定轮次或动态内容。
-   `systemPrompt`：完整的系统提示覆盖。
-   `prependSystemContext`：将文本预置到当前系统提示。
-   `appendSystemContext`：将文本追加到当前系统提示。

嵌入式运行时中的提示构建顺序：

1.  将 `prependContext` 应用于用户提示。
2.  提供时应用 `systemPrompt` 覆盖。
3.  应用 `prependSystemContext + 当前系统提示 + appendSystemContext`。

合并和优先级说明：

-   钩子处理程序按优先级运行（高优先级先执行）。
-   对于合并的上下文字段，值按执行顺序连接。
-   `before_prompt_build` 的值在旧版 `before_agent_start` 回退值之前应用。

迁移指南：

-   将静态指导从 `prependContext` 移动到 `prependSystemContext`（或 `appendSystemContext`），以便提供商可以缓存稳定的系统前缀内容。
-   将 `prependContext` 保留用于应与用户消息绑定的每轮动态上下文。

## 提供商插件（模型认证）

插件可以注册**模型提供商认证**流程，以便用户可以在 OpenClaw 内部运行 OAuth 或 API 密钥设置（无需外部脚本）。通过 `api.registerProvider(...)` 注册一个提供商。每个提供商暴露一个或多个认证方法（OAuth、API 密钥、设备代码等）。这些方法支持：

-   `openclaw models auth login --provider  [--method ]`

示例：

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
        // 运行 OAuth 流程并返回认证配置文件。
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

注意：

-   `run` 接收一个 `ProviderAuthContext`，其中包含 `prompter`、`runtime`、`openUrl` 和 `oauth.createVpsAwareHandlers` 助手。
-   当你需要添加默认模型或提供商配置时，返回 `configPatch`。
-   返回 `defaultModel` 以便 `--set-default` 可以更新智能体默认值。

### 注册一个消息频道

插件可以注册**频道插件**，其行为类似于内置频道（WhatsApp、Telegram 等）。频道配置位于 `channels.` 下，并由你的频道插件代码验证。

```typescript
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "演示频道插件。",
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

注意：

-   将配置放在 `channels.` 下（而不是 `plugins.entries`）。
-   `meta.label` 用于 CLI/UI 列表中的标签。
-   `meta.aliases` 为规范化和 CLI 输入添加备用 ID。
-   `meta.preferOver` 列出在两者都配置时要跳过自动启用的频道 ID。
-   `meta.detailLabel` 和 `meta.systemImage` 允许 UI 显示更丰富的频道标签/图标。

### 频道入门钩子

频道插件可以在 `plugin.onboarding` 上定义可选的入门钩子：

-   `configure(ctx)` 是基线设置流程。
-   `configureInteractive(ctx)` 可以完全拥有交互式设置，适用于已配置和未配置状态。
-   `configureWhenConfigured(ctx)` 可以仅针对已配置的频道覆盖行为。

向导中的钩子优先级：

1.  `configureInteractive`（如果存在）
2.  `configureWhenConfigured`（仅当频道状态已配置时）
3.  回退到 `configure`

上下文详情：

-   `configureInteractive` 和 `configureWhenConfigured` 接收：
    -   `configured`（`true` 或 `false`）
    -   `label`（提示使用的面向用户的频道名称）
    -   加上共享的 config/runtime/prompter/options 字段
-   返回 `"skip"` 会保持选择和账户跟踪不变。
-   返回 `{ cfg, accountId? }` 会应用配置更新并记录账户选择。

### 编写一个新的消息频道（逐步指南）

当你想要一个**新的聊天界面**（一个“消息频道”）时使用此方法，而不是模型提供商。模型提供商的文档位于 `/providers/*` 下。

1.  选择一个 ID + 配置形状

-   所有频道配置都位于 `channels.` 下。
-   对于多账户设置，优先使用 `channels..accounts.`。

2.  定义频道元数据

-   `meta.label`、`meta.selectionLabel`、`meta.docsPath`、`meta.blurb` 控制 CLI/UI 列表。
-   `meta.docsPath` 应指向一个文档页面，如 `/channels/`。
-   `meta.preferOver` 允许插件替换另一个频道（自动启用时优先选择它）。
-   `meta.detailLabel` 和 `meta.systemImage` 被 UI 用于详细文本/图标。

3.  实现必需的适配器

-   `config.listAccountIds` + `config.resolveAccount`
-   `capabilities`（聊天类型、媒体、线程等）
-   `outbound.deliveryMode` + `outbound.sendText`（用于基本发送）

4.  根据需要添加可选适配器

-   `setup`（向导）、`security`（DM 策略）、`status`（健康/诊断）
-   `gateway`（启动/停止/登录）、`mentions`、`threading`、`streaming`
-   `actions`（消息操作）、`commands`（本地命令行为）

5.  在你的插件中注册频道

-   `api.registerChannel({ plugin })`

最小配置示例：

```json
{
  channels: {
    acmechat: {
      accounts: {
        default: { token: "ACME_TOKEN", enabled: true },
      },
    },
  },
}
```

最小频道插件（仅出站）：

```typescript
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat 消息频道。",
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
    sendText: async ({ text }) => {
      // 在此处将 `text` 传递到你的频道
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

加载插件（扩展目录或 `plugins.load.paths`），重启网关，然后在你的配置中配置 `channels.`。

### 智能体工具

请参阅专用指南：[插件智能体工具](../plugins/agent-tools.md)。

### 注册网关 RPC 方法

```bash
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

### 注册 CLI 命令

```bash
export default function (api) {
  api.registerCli(
    ({ program }) => {
      program.command("mycmd").action(() => {
        console.log("Hello");
      });
    },
    { commands: ["mycmd"] },
  );
}
```

### 注册自动回复命令

插件可以注册自定义斜杠命令，这些命令**无需调用 AI 智能体**即可执行。这对于切换命令、状态检查或不需要 LLM 处理的快速操作很有用。

```bash
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "显示插件状态",
    handler: (ctx) => ({
      text: `插件正在运行！频道：${ctx.channel}`,
    }),
  });
}
```

命令处理程序上下文：

-   `senderId`：发送者的 ID（如果可用）
-   `channel`：发送命令的频道
-   `isAuthorizedSender`：发送者是否为授权用户
-   `args`：命令后传递的参数（如果 `acceptsArgs: true`）
-   `commandBody`：完整的命令文本
-   `config`：当前的 OpenClaw 配置

命令选项：

-   `name`：命令名称（不带前导 `/`）
-   `description`：在命令列表中显示的帮助文本
-   `acceptsArgs`：命令是否接受参数（默认：false）。如果为 false 且提供了参数，命令将不匹配，消息会传递给其他处理程序
-   `requireAuth`：是否需要授权发送者（默认：true）
-   `handler`：返回 `{ text: string }` 的函数（可以是异步的）

带有授权和参数的示例：

```
api.registerCommand({
  name: "setmode",
  description: "设置插件模式",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `模式设置为：${mode}` };
  },
});
```

注意：

-   插件命令在**内置命令和 AI 智能体之前**处理
-   命令全局注册，适用于所有频道
-   命令名称不区分大小写（`/MyStatus` 匹配 `/mystatus`）
-   命令名称必须以字母开头，并且只能包含字母、数字、连字符和下划线
-   保留的命令名称（如 `help`、`status`、`reset` 等）不能被插件覆盖
-   跨插件的重复命令注册将失败并产生诊断错误

### 注册后台服务

```bash
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("ready"),
    stop: () => api