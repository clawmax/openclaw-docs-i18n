

  概述

  
# 工具

OpenClaw 为浏览器、画布、节点和定时任务提供了**一流的代理工具**。这些工具取代了旧的 `openclaw-*` 技能：工具是类型化的，无需 shell 调用，代理应直接依赖它们。

## 禁用工具

你可以通过 `openclaw.json` 中的 `tools.allow` / `tools.deny` 全局允许/拒绝工具（拒绝优先）。这可以防止不允许的工具被发送给模型提供商。

```json
{
  tools: { deny: ["browser"] },
}
```

注意：

-   匹配不区分大小写。
-   支持 `*` 通配符（`"*"` 表示所有工具）。
-   如果 `tools.allow` 仅引用了未知或未加载的插件工具名称，OpenClaw 会记录警告并忽略允许列表，以便核心工具保持可用。

## 工具配置文件（基础允许列表）

`tools.profile` 在 `tools.allow`/`tools.deny` 之前设置一个**基础工具允许列表**。可按代理覆盖：`agents.list[].tools.profile`。配置文件：

-   `minimal`: 仅 `session_status`
-   `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
-   `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
-   `full`: 无限制（与未设置相同）

示例（默认仅消息传递，同时允许 Slack + Discord 工具）：

```json
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

示例（编码配置文件，但全局拒绝 exec/process）：

```json
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

示例（全局编码配置文件，仅消息传递的支持代理）：

```json
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] },
      },
    ],
  },
}
```

## 特定于提供商的工具策略

使用 `tools.byProvider` 来**进一步限制**特定提供商（或单个 `provider/model`）的工具，而无需更改全局默认设置。可按代理覆盖：`agents.list[].tools.byProvider`。此策略在基础工具配置文件**之后**、允许/拒绝列表**之前**应用，因此它只能缩小工具集。提供商键接受 `provider`（例如 `google-antigravity`）或 `provider/model`（例如 `openai/gpt-5.2`）。示例（保持全局编码配置文件，但对 Google Antigravity 使用最小工具集）：

```json
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

示例（针对不稳定端点的特定 provider/model 允许列表）：

```json
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

示例（针对单个提供商的代理特定覆盖）：

```json
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] },
          },
        },
      },
    ],
  },
}
```

## 工具组（简写）

工具策略（全局、代理、沙箱）支持 `group:*` 条目，这些条目会扩展为多个工具。在 `tools.allow` / `tools.deny` 中使用这些组。可用组：

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:web`: `web_search`, `web_fetch`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: 所有内置的 OpenClaw 工具（不包括提供商插件）

示例（仅允许文件工具 + 浏览器）：

```json
{
  tools: {
    allow: ["group:fs", "browser"],
  },
}
```

## 插件 + 工具

插件可以注册**额外的工具**（和 CLI 命令），超出核心集。有关安装和配置，请参阅[插件](./tools/plugin.md)；有关工具使用指南如何注入提示，请参阅[技能](./tools/skills.md)。一些插件附带自己的技能和工具（例如，语音通话插件）。可选插件工具：

-   [Lobster](./tools/lobster.md): 带有可恢复审批的类型化工作流运行时（需要在网关主机上安装 Lobster CLI）。
-   [LLM Task](./tools/llm-task.md): 用于结构化工作流输出的仅 JSON LLM 步骤（可选模式验证）。
-   [Diffs](./tools/diffs.md): 用于前后文本或统一补丁的只读差异查看器和 PNG 或 PDF 文件渲染器。

## 工具清单

### apply\_patch

跨一个或多个文件应用结构化补丁。用于多块编辑。实验性功能：通过 `tools.exec.applyPatch.enabled` 启用（仅限 OpenAI 模型）。`tools.exec.applyPatch.workspaceOnly` 默认为 `true`（仅限工作空间内）。仅在有意希望 `apply_patch` 在工作空间目录之外写入/删除时，才将其设置为 `false`。

### exec

在工作空间中运行 shell 命令。核心参数：

-   `command`（必需）
-   `yieldMs`（超时后自动后台运行，默认 10000）
-   `background`（立即后台运行）
-   `timeout`（秒；超过则终止进程，默认 1800）
-   `elevated`（布尔值；如果启用了提升模式/允许，则在主机上运行；仅当代理处于沙箱中时才改变行为）
-   `host` (`sandbox | gateway | node`)
-   `security` (`deny | allowlist | full`)
-   `ask` (`off | on-miss | always`)
-   `node`（用于 `host=node` 的节点 ID/名称）
-   需要真正的 TTY？设置 `pty: true`。

注意：

-   后台运行时返回 `status: "running"` 和 `sessionId`。
-   使用 `process` 来轮询/记录/写入/终止/清除后台会话。
-   如果 `process` 被禁止，`exec` 将同步运行并忽略 `yieldMs`/`background`。
-   `elevated` 受 `tools.elevated` 以及任何 `agents.list[].tools.elevated` 覆盖的控制（两者都必须允许），并且是 `host=gateway` + `security=full` 的别名。
-   仅当代理处于沙箱中时，`elevated` 才会改变行为（否则为无操作）。
-   `host=node` 可以定位 macOS 伴侣应用或无头节点主机（`openclaw node run`）。
-   网关/节点审批和允许列表：[Exec 审批](./tools/exec-approvals.md)。

### process

管理后台 exec 会话。核心操作：

-   `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

注意：

-   `poll` 在完成时返回新输出和退出状态。
-   `log` 支持基于行的 `offset`/`limit`（省略 `offset` 以获取最后 N 行）。
-   `process` 按代理作用域；其他代理的会话不可见。

### loop-detection（工具调用循环防护）

OpenClaw 跟踪最近的工具调用历史记录，并在检测到重复的无进展循环时阻止或警告。通过 `tools.loopDetection.enabled: true` 启用（默认为 `false`）。

```json
{
  tools: {
    loopDetection: {
      enabled: true,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      historySize: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

-   `genericRepeat`: 重复的相同工具 + 相同参数调用模式。
-   `knownPollNoProgress`: 重复的轮询类工具且输出相同。
-   `pingPong`: 交替的 `A/B/A/B` 无进展模式。
-   可按代理覆盖：`agents.list[].tools.loopDetection`。

### web\_search

使用 Perplexity、Brave、Gemini、Grok 或 Kimi 搜索网络。核心参数：

-   `query`（必需）
-   `count`（1–10；默认来自 `tools.web.search.maxResults`）

注意：

-   需要所选提供商的 API 密钥（推荐：`openclaw configure --section web`）。
-   通过 `tools.web.search.enabled` 启用。
-   响应会被缓存（默认 15 分钟）。
-   有关设置，请参阅[网络工具](./tools/web.md)。

### web\_fetch

从 URL 获取并提取可读内容（HTML → markdown/文本）。核心参数：

-   `url`（必需）
-   `extractMode` (`markdown` | `text`)
-   `maxChars`（截断长页面）

注意：

-   通过 `tools.web.fetch.enabled` 启用。
-   `maxChars` 受 `tools.web.fetch.maxCharsCap` 限制（默认 50000）。
-   响应会被缓存（默认 15 分钟）。
-   对于 JS 密集型网站，建议使用浏览器工具。
-   有关设置，请参阅[网络工具](./tools/web.md)。
-   有关可选的防机器人回退，请参阅 [Firecrawl](./tools/firecrawl.md)。

### browser

控制 OpenClaw 管理的专用浏览器。核心操作：

-   `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
-   `snapshot` (aria/ai)
-   `screenshot`（返回图像块 + `MEDIA:`）
-   `act`（UI 操作：点击/输入/按键/悬停/拖拽/选择/填充/调整大小/等待/评估）
-   `navigate`, `console`, `pdf`, `upload`, `dialog`

配置文件管理：

-   `profiles` — 列出所有浏览器配置文件及其状态
-   `create-profile` — 创建具有自动分配端口（或 `cdpUrl`）的新配置文件
-   `delete-profile` — 停止浏览器，删除用户数据，从配置中移除（仅限本地）
-   `reset-profile` — 终止配置文件端口上的孤立进程（仅限本地）

通用参数：

-   `profile`（可选；默认为 `browser.defaultProfile`）
-   `target` (`sandbox` | `host` | `node`)
-   `node`（可选；选择特定的节点 ID/名称）注意：
-   需要 `browser.enabled=true`（默认为 `true`；设置为 `false` 以禁用）。
-   所有操作都接受可选的 `profile` 参数以支持多实例。
-   省略 `profile` 时，使用 `browser.defaultProfile`（默认为“chrome”）。
-   配置文件名称：仅限小写字母数字 + 连字符（最多 64 个字符）。
-   端口范围：18800-18899（最多约 100 个配置文件）。
-   远程配置文件仅支持附加（无启动/停止/重置）。
-   如果连接了支持浏览器的节点，工具可能会自动路由到该节点（除非你固定了 `target`）。
-   安装了 Playwright 时，`snapshot` 默认为 `ai`；使用 `aria` 获取可访问性树。
-   `snapshot` 还支持角色快照选项（`interactive`, `compact`, `depth`, `selector`），这些选项返回像 `e12` 这样的引用。
-   `act` 需要来自 `snapshot` 的 `ref`（来自 AI 快照的数字 `12`，或来自角色快照的 `e12`）；对于罕见的 CSS 选择器需求，请使用 `evaluate`。
-   默认避免 `act` → `wait`；仅在特殊情况下使用（没有可靠的 UI 状态可等待）。
-   `upload` 可以选择传递 `ref` 以在准备后自动点击。
-   `upload` 还支持 `inputRef`（aria 引用）或 `element`（CSS 选择器）来直接设置 ``。

### canvas

驱动节点画布（呈现、评估、快照、A2UI）。核心操作：

-   `present`, `hide`, `navigate`, `eval`
-   `snapshot`（返回图像块 + `MEDIA:`）
-   `a2ui_push`, `a2ui_reset`

注意：

-   底层使用网关 `node.invoke`。
-   如果未提供 `node`，工具会选择默认值（单个连接的节点或本地 mac 节点）。
-   A2UI 仅限 v0.8（无 `createSurface`）；CLI 会拒绝带有行错误的 v0.9 JSONL。
-   快速测试：`openclaw nodes canvas a2ui push --node  --text "Hello from A2UI"`。

### nodes

发现并定位配对的节点；发送通知；捕获摄像头/屏幕。核心操作：

-   `status`, `describe`
-   `pending`, `approve`, `reject`（配对）
-   `notify`（macOS `system.notify`）
-   `run`（macOS `system.run`）
-   `camera_list`, `camera_snap`, `camera_clip`, `screen_record`
-   `location_get`, `notifications_list`, `notifications_action`
-   `device_status`, `device_info`, `device_permissions`, `device_health`

注意：

-   摄像头/屏幕命令需要节点应用在前台运行。
-   图像返回图像块 + `MEDIA:`。
-   视频返回 `FILE:` (mp4)。
-   位置返回 JSON 负载（纬度/经度/精度/时间戳）。
-   `run` 参数：`command` argv 数组；可选的 `cwd`, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording`。

示例（`run`）：

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

### image

使用配置的图像模型分析图像。核心参数：

-   `image`（必需路径或 URL）
-   `prompt`（可选；默认为“描述图像。”）
-   `model`（可选覆盖）
-   `maxBytesMb`（可选大小限制）

注意：

-   仅当配置了 `agents.defaults.imageModel`（主要或备用）时可用，或者当可以从默认模型 + 配置的身份验证推断出隐式图像模型时（尽力配对）。
-   直接使用图像模型（独立于主聊天模型）。

### pdf

分析一个或多个 PDF 文档。有关完整行为、限制、配置和示例，请参阅 [PDF 工具](./tools/pdf.md)。

### message

跨 Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams 发送消息和频道操作。核心操作：

-   `send`（文本 + 可选媒体；MS Teams 还支持 `card` 用于自适应卡片）
-   `poll`（WhatsApp/Discord/MS Teams 投票）
-   `react` / `reactions` / `read` / `edit` / `delete`
-   `pin` / `unpin` / `list-pins`
-   `permissions`
-   `thread-create` / `thread-list` / `thread-reply`
-   `search`
-   `sticker`
-   `member-info` / `role-info`
-   `emoji-list` / `emoji-upload` / `sticker-upload`
-   `role-add` / `role-remove`
-   `channel-info` / `channel-list`
-   `voice-status`
-   `event-list` / `event-create`
-   `timeout` / `kick` / `ban`

注意：

-   `send` 通过网关路由 WhatsApp；其他频道直接发送。
-   `poll` 对 WhatsApp 和 MS Teams 使用网关；Discord 投票直接发送。
-   当消息工具调用绑定到活动聊天会话时，发送被限制在该会话的目标，以避免跨上下文泄露。

### cron

管理网关定时任务和唤醒。核心操作：

-   `status`, `list`
-   `add`, `update`, `remove`, `run`, `runs`
-   `wake`（入队系统事件 + 可选立即心跳）

注意：

-   `add` 需要一个完整的定时任务对象（与 `cron.add` RPC 相同的模式）。
-   `update` 使用 `{ jobId, patch }`（为兼容性接受 `id`）。

### gateway

重新启动或应用更新到正在运行的网关进程（原地）。核心操作：

-   `restart`（授权并发送 `SIGUSR1` 进行进程内重启；`openclaw gateway` 原地重启）
-   `config.schema.lookup`（一次检查一个配置路径，而无需将完整模式加载到提示上下文中）
-   `config.get`
-   `config.apply`（验证 + 写入配置 + 重启 + 唤醒）
-   `config.patch`（合并部分更新 + 重启 + 唤醒）
-   `update.run`（运行更新 + 重启 + 唤醒）

注意：

-   `config.schema.lookup` 需要一个目标配置路径，例如 `gateway.auth` 或 `agents.list.*.heartbeat`。
-   当寻址 `plugins.entries.` 时，路径可能包含斜杠分隔的插件 ID，例如 `plugins.entries.pack/one.config`。
-   使用 `delayMs`（默认为 2000）以避免中断正在进行的回复。
-   `config.schema` 仍可用于内部控制 UI 流程，并且不会通过代理 `gateway` 工具公开。
-   `restart` 默认启用；设置 `commands.restart: false` 以禁用它。

### sessions\_list / sessions\_history / sessions\_send / sessions\_spawn / session\_status

列出会话、检查转录历史记录或发送到另一个会话。核心参数：

-   `sessions_list`: `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = 无)
-   `sessions_history`: `sessionKey`（或 `sessionId`）, `limit?`, `includeTools?`
-   `sessions_send`: `sessionKey`（或 `sessionId`）, `message`, `timeoutSeconds?` (0 = 即发即弃)
-   `sessions_spawn`: `task`, `label?`, `runtime?`, `agentId?`, `model?`, `thinking?`, `cwd?`, `runTimeoutSeconds?`, `thread?`, `mode?`, `cleanup?`, `sandbox?`, `streamTo?`, `attachments?`, `attachAs?`
-   `session_status`: `sessionKey?`（默认为当前；接受 `sessionId`）, `model?` (`default` 清除覆盖)

注意：

-   `main` 是规范的直接聊天键；全局/未知会话被隐藏。
-   `messageLimit > 0` 获取每个会话的最后 N 条消息（工具消息被过滤）。
-   会话目标由 `tools.sessions.visibility` 控制（默认 `tree`：当前会话 + 生成的子代理会话）。如果你为多个用户运行共享代理，请考虑设置 `tools.sessions.visibility: "self"` 以防止跨会话浏览。
-   `sessions_send` 在 `timeoutSeconds > 0` 时等待最终完成。
-   交付/通知在完成后进行，并且是尽力而为的；`status: "ok"` 确认代理运行完成，而不是通知已送达。
-   `sessions_spawn` 支持 `runtime: "subagent" | "acp"`（默认为 `subagent`）。有关 ACP 运行时行为，请参阅 [ACP 代理](./tools/acp-agents.md)。
-   对于 ACP 运行时，`streamTo: "parent"` 将初始运行进度摘要作为系统事件路由回请求者会话，而不是直接传递给子会话。
-   `sessions_spawn` 启动子代理运行，并将通知回复发布回请求者聊天。
    -   支持一次性模式（`mode: "run"`）和持久线程绑定模式（`mode: "session"` 且 `thread: true`）。
    -   如果 `thread: true` 且省略 `mode`，则模式默认为 `session`。
    -   `mode: "session"` 需要 `thread: true`。
    -   如果省略 `runTimeoutSeconds`，OpenClaw 在设置时使用 `agents.defaults.subagents.runTimeoutSeconds`；否则超时默认为 `0`（无超时）。
    -   Discord 线程绑定流程依赖于 `session.threadBindings.*` 和 `channels.discord.threadBindings.*`。
    -   回复格式包括 `Status`、`Result` 和紧凑统计信息。
    -   `Result` 是助手完成文本；如果缺失，则使用最新的 `toolResult` 作为回退。
-   手动完成模式的生成首先直接发送，具有队列回退和临时失败重试（`status: "ok"` 表示运行完成，而不是通知已送达）。
-   `sessions_spawn` 支持仅用于子代理运行时的内联文件附件（ACP 拒绝它们）。每个附件都有 `name`、`content` 和可选的 `encoding`（`utf8` 或 `base64`）以及 `mimeType`。文件被具体化到子工作空间的 `.openclaw/attachments//` 目录中，并附带一个 `.manifest.json` 元数据文件。该工具返回一个包含 `count`、`totalBytes`、每个文件的 `sha256` 和 `relDir` 的回执。附件内容会自动从转录持久化中删除。
    -   通过 `tools.sessions_spawn.attachments` 配置限制（`enabled`, `maxTotalBytes`, `maxFiles`, `maxFileBytes`, `retainOnSessionKeep`）。
    -   `attachAs.mountPath` 是为未来挂载实现保留的提示。
-   `sessions_spawn` 是非阻塞的，并立即返回 `status: "accepted"`。
-   ACP `streamTo: "parent"` 响应可能包含 `streamLogPath`（会话作用域的 `*.acp-stream.jsonl`）用于跟踪进度历史记录。
-   `sessions_send` 运行回复乒乓（回复 `REPLY_SKIP` 以停止；最大轮次通过 `session.agentToAgent.maxPingPongTurns`，0–5）。
-   乒乓之后，目标代理运行一个**通知步骤**；回复 `ANNOUNCE_SKIP` 以抑制通知。
-   沙箱限制：当当前会话处于沙箱中且 `agents.defaults.sandbox.sessionToolsVisibility: "spawned"` 时，OpenClaw 将 `tools.sessions.visibility` 限制为 `tree`。

### agents\_list

列出当前会话可以通过 `sessions_spawn` 定位的代理 ID。注意：

-   结果受限于每个代理的允许列表（`agents.list[].subagents.allowAgents`）。
-   当配置了 `["*"]` 时，该工具包含所有已配置的代理并标记 `allowAny: true`。

## 参数（通用）

网关支持的工具（`canvas`, `nodes`, `cron`）：

-   `gatewayUrl`（默认 `ws://127.0.0.1:18789`）
-   `gatewayToken`（如果启用了身份验证）
-   `timeoutMs`

注意：设置 `gatewayUrl` 时，请显式包含 `gatewayToken`。工具不会继承配置或环境凭据进行覆盖，缺少显式凭据会导致错误。浏览器工具：

-   `profile`（可选；默认为 `browser.defaultProfile`）
-   `target` (`sandbox` | `host` | `node`)
-   `node`（可选；固定特定的节点 ID/名称）

## 推荐的代理流程

浏览器自动化：

1.  `browser` → `status` / `start`
2.  `snapshot` (ai 或 aria)
3.  `act` (click/type/press)
4.  如果需要视觉确认，使用 `screenshot`

画布渲染：

1.  `canvas` → `present`
2.  `a2ui_push`（可选）
3.  `snapshot`

节点定位：

1.  `nodes` → `status`
2.  在所选节点上 `describe`
3.  `notify` / `run` / `camera_snap` / `screen_record`

## 安全性

-   避免直接使用 `system.run`；仅在获得用户明确同意时使用 `nodes` → `run`。
-   尊重用户对摄像头/屏幕捕获的同意。
-   在调用媒体命令之前，使用 `status/describe` 确保权限。

## 工具如何呈现给代理

工具通过两个并行渠道暴露：

1.  **系统提示文本**：人类可读的列表 + 指南。
2.  **工具模式**：发送给模型 API 的结构化函数定义。

这意味着代理既看到“存在哪些工具”，也看到“如何调用它们”。如果一个工具没有出现在系统提示或模式中，模型就无法调用它。

[apply\_patch 工具](./tools/apply-patch.md)