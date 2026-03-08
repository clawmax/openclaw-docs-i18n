

  自动化

  
# 钩子

钩子提供了一个可扩展的事件驱动系统，用于在响应代理命令和事件时自动执行操作。钩子会自动从目录中发现，并可以通过 CLI 命令进行管理，类似于 OpenClaw 中技能的工作方式。

## 快速了解

钩子是当某些事件发生时运行的小脚本。有两种类型：

-   **钩子**（本页面）：当代理事件触发时在网关内部运行，例如 `/new`、`/reset`、`/stop` 或生命周期事件。
-   **Webhooks**：外部 HTTP webhook，允许其他系统在 OpenClaw 中触发工作。请参阅 [Webhook 钩子](./webhook.md) 或使用 `openclaw webhooks` 获取 Gmail 辅助命令。

钩子也可以捆绑在插件内部；请参阅 [插件](../tools/plugin.md#plugin-hooks)。常见用途：

-   重置会话时保存记忆快照
-   保留命令的审计跟踪以进行故障排除或合规性检查
-   会话开始或结束时触发后续自动化
-   事件触发时将文件写入代理工作区或调用外部 API

如果你能编写一个小的 TypeScript 函数，你就能编写一个钩子。钩子会被自动发现，你可以通过 CLI 启用或禁用它们。

## 概述

钩子系统允许你：

-   发出 `/new` 命令时将会话上下文保存到记忆
-   记录所有命令以供审计
-   在代理生命周期事件上触发自定义自动化
-   无需修改核心代码即可扩展 OpenClaw 的行为

## 开始使用

### 捆绑的钩子

OpenClaw 附带了四个捆绑的钩子，它们会被自动发现：

-   **💾 session-memory**：当你发出 `/new` 时，将会话上下文保存到你的代理工作区（默认 `~/.openclaw/workspace/memory/`）
-   **📎 bootstrap-extra-files**：在 `agent:bootstrap` 期间，从配置的 glob/路径模式注入额外的工作区引导文件
-   **📝 command-logger**：将所有命令事件记录到 `~/.openclaw/logs/commands.log`
-   **🚀 boot-md**：网关启动时运行 `BOOT.md`（需要启用内部钩子）

列出可用钩子：

```bash
openclaw hooks list
```

启用一个钩子：

```bash
openclaw hooks enable session-memory
```

检查钩子状态：

```bash
openclaw hooks check
```

获取详细信息：

```bash
openclaw hooks info session-memory
```

### 入门引导

在入门引导期间（`openclaw onboard`），系统会提示你启用推荐的钩子。向导会自动发现符合条件的钩子并呈现供你选择。

## 钩子发现

钩子会自动从三个目录中发现（按优先级顺序）：

1.  **工作区钩子**：`/hooks/`（每个代理，最高优先级）
2.  **托管钩子**：`~/.openclaw/hooks/`（用户安装，跨工作区共享）
3.  **捆绑钩子**：`/dist/hooks/bundled/`（随 OpenClaw 一起提供）

托管钩子目录可以是**单个钩子**或**钩子包**（包目录）。每个钩子是一个包含以下内容的目录：

```
my-hook/
├── HOOK.md          # 元数据 + 文档
└── handler.ts       # 处理器实现
```

## 钩子包 (npm/归档文件)

钩子包是标准的 npm 包，通过 `package.json` 中的 `openclaw.hooks` 导出一个或多个钩子。使用以下命令安装它们：

```bash
openclaw hooks install <path-or-spec>
```

Npm 规范仅限注册表（包名 + 可选的确切版本或分发标签）。Git/URL/文件规范和 semver 范围会被拒绝。裸规范和 `@latest` 保持在稳定轨道上。如果 npm 将其中任何一个解析为预发布版本，OpenClaw 会停止并要求你使用预发布标签（如 `@beta`/`@rc`）或确切的预发布版本明确选择加入。示例 `package.json`：

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

每个条目指向一个包含 `HOOK.md` 和 `handler.ts`（或 `index.ts`）的钩子目录。钩子包可以附带依赖项；它们将被安装在 `~/.openclaw/hooks/` 下。每个 `openclaw.hooks` 条目在符号链接解析后必须保留在包目录内；逃逸的条目会被拒绝。安全说明：`openclaw hooks install` 使用 `npm install --ignore-scripts` 安装依赖项（无生命周期脚本）。保持钩子包依赖树为“纯 JS/TS”，避免依赖 `postinstall` 构建的包。

## 钩子结构

### HOOK.md 格式

`HOOK.md` 文件包含 YAML 前言中的元数据以及 Markdown 文档：

```
---
name: my-hook
description: "此钩子功能的简短描述"
homepage: https://docs.openclaw.ai/automation/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# 我的钩子

详细文档放在这里...

## 功能

- 监听 `/new` 命令
- 执行某些操作
- 记录结果

## 要求

- 必须安装 Node.js

## 配置

无需配置。
```

### 元数据字段

`metadata.openclaw` 对象支持：

-   **`emoji`**：用于 CLI 的显示表情符号（例如 `"💾"`）
-   **`events`**：要监听的事件数组（例如 `["command:new", "command:reset"]`）
-   **`export`**：要使用的命名导出（默认为 `"default"`）
-   **`homepage`**：文档 URL
-   **`requires`**：可选要求
    -   **`bins`**：PATH 上必需的二进制文件（例如 `["git", "node"]`）
    -   **`anyBins`**：这些二进制文件中至少有一个必须存在
    -   **`env`**：必需的环境变量
    -   **`config`**：必需的配置路径（例如 `["workspace.dir"]`）
    -   **`os`**：必需的平台（例如 `["darwin", "linux"]`）
-   **`always`**：绕过资格检查（布尔值）
-   **`install`**：安装方法（对于捆绑钩子：`[{"id":"bundled","kind":"bundled"}]`）

### 处理器实现

`handler.ts` 文件导出一个 `HookHandler` 函数：

```typescript
const myHandler = async (event) => {
  // 仅在 'new' 命令时触发
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // 你的自定义逻辑放在这里

  // 可选地向用户发送消息
  event.messages.push("✨ My hook executed!");
};

export default myHandler;
```

#### 事件上下文

每个事件包含：

```json
{
  type: 'command' | 'session' | 'agent' | 'gateway' | 'message',
  action: string,              // 例如 'new', 'reset', 'stop', 'received', 'sent'
  sessionKey: string,          // 会话标识符
  timestamp: Date,             // 事件发生的时间
  messages: string[],          // 将消息推送到这里以发送给用户
  context: {
    // 命令事件：
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // 例如 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig,
    // 消息事件（完整详情请参阅消息事件部分）：
    from?: string,             // message:received
    to?: string,               // message:sent
    content?: string,
    channelId?: string,
    success?: boolean,         // message:sent
  }
}
```

## 事件类型

### 命令事件

当发出代理命令时触发：

-   **`command`**：所有命令事件（通用监听器）
-   **`command:new`**：当发出 `/new` 命令时
-   **`command:reset`**：当发出 `/reset` 命令时
-   **`command:stop`**：当发出 `/stop` 命令时

### 会话事件

-   **`session:compact:before`**：在压缩开始总结历史之前
-   **`session:compact:after`**：压缩完成后，包含摘要元数据

内部钩子负载将这些事件作为 `type: "session"` 和 `action: "compact:before"` / `action: "compact:after"` 发出；监听器使用上述组合键订阅。特定的处理器注册使用字面键格式 `${type}:${action}`。对于这些事件，请注册 `session:compact:before` 和 `session:compact:after`。

### 代理事件

-   **`agent:bootstrap`**：在工作区引导文件注入之前（钩子可以修改 `context.bootstrapFiles`）

### 网关事件

网关启动时触发：

-   **`gateway:startup`**：通道启动且钩子加载后

### 消息事件

当消息被接收或发送时触发：

-   **`message`**：所有消息事件（通用监听器）
-   **`message:received`**：当从任何通道接收到入站消息时。在处理早期触发，在媒体理解之前。内容可能包含原始占位符，如用于尚未处理的媒体附件的 `<media:audio>`。
-   **`message:transcribed`**：当消息被完全处理时，包括音频转录和链接理解。此时，`transcript` 包含音频消息的完整转录文本。当你需要访问转录的音频内容时，请使用此钩子。
-   **`message:preprocessed`**：在所有媒体 + 链接理解完成后，为每条消息触发，让钩子在代理看到之前访问完全丰富的正文（转录、图像描述、链接摘要）。
-   **`message:sent`**：当出站消息成功发送时

#### 消息事件上下文

消息事件包含有关消息的丰富上下文：

```
// message:received 上下文
{
  from: string,           // 发送者标识符（电话号码、用户 ID 等）
  content: string,        // 消息内容
  timestamp?: number,     // 接收时的 Unix 时间戳
  channelId: string,      // 通道（例如 "whatsapp", "telegram", "discord"）
  accountId?: string,     // 多账户设置中的提供商账户 ID
  conversationId?: string, // 聊天/会话 ID
  messageId?: string,     // 来自提供商的消息 ID
  metadata?: {            // 额外的提供商特定数据
    to?: string,
    provider?: string,
    surface?: string,
    threadId?: string,
    senderId?: string,
    senderName?: string,
    senderUsername?: string,
    senderE164?: string,
  }
}

// message:sent 上下文
{
  to: string,             // 接收者标识符
  content: string,        // 已发送的消息内容
  success: boolean,       // 发送是否成功
  error?: string,         // 发送失败时的错误消息
  channelId: string,      // 通道（例如 "whatsapp", "telegram", "discord"）
  accountId?: string,     // 提供商账户 ID
  conversationId?: string, // 聊天/会话 ID
  messageId?: string,     // 提供商返回的消息 ID
  isGroup?: boolean,      // 此出站消息是否属于群组/频道上下文
  groupId?: string,       // 用于与 message:received 关联的群组/频道标识符
}

// message:transcribed 上下文
{
  body?: string,          // 丰富前的原始入站正文
  bodyForAgent?: string,  // 代理可见的丰富后正文
  transcript: string,     // 音频转录文本
  channelId: string,      // 通道（例如 "telegram", "whatsapp"）
  conversationId?: string,
  messageId?: string,
}

// message:preprocessed 上下文
{
  body?: string,          // 原始入站正文
  bodyForAgent?: string,  // 媒体/链接理解后的最终丰富正文
  transcript?: string,    // 存在音频时的转录
  channelId: string,      // 通道（例如 "telegram", "whatsapp"）
  conversationId?: string,
  messageId?: string,
  isGroup?: boolean,
  groupId?: string,
}
```

#### 示例：消息记录器钩子

```typescript
const isMessageReceivedEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "received";
const isMessageSentEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "sent";

const handler = async (event) => {
  if (isMessageReceivedEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Received from ${event.context.from}: ${event.context.content}`);
  } else if (isMessageSentEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Sent to ${event.context.to}: ${event.context.content}`);
  }
};

export default handler;
```

### 工具结果钩子（插件 API）

这些钩子不是事件流监听器；它们允许插件在 OpenClaw 持久化之前同步调整工具结果。

-   **`tool_result_persist`**：在工具结果写入会话记录之前转换它们。必须是同步的；返回更新后的工具结果负载或 `undefined` 以保持原样。请参阅 [代理循环](../concepts/agent-loop.md)。

### 插件钩子事件

通过插件钩子运行器暴露的压缩生命周期钩子：

-   **`before_compaction`**：在压缩之前运行，包含计数/令牌元数据
-   **`after_compaction`**：在压缩之后运行，包含压缩摘要元数据

### 未来事件

计划中的事件类型：

-   **`session:start`**：新会话开始时
-   **`session:end`**：会话结束时
-   **`agent:error`**：代理遇到错误时

## 创建自定义钩子

### 1. 选择位置

-   **工作区钩子**（`/hooks/`）：每个代理，最高优先级
-   **托管钩子**（`~/.openclaw/hooks/`）：跨工作区共享

### 2. 创建目录结构

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3. 创建 HOOK.md

```
---
name: my-hook
description: "做一些有用的事情"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# 我的自定义钩子

当你发出 `/new` 时，此钩子会做一些有用的事情。
```

### 4. 创建 handler.ts

```typescript
const handler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Running!");
  // 你的逻辑放在这里
};

export default handler;
```

### 5. 启用并测试

```bash
# 验证钩子是否被发现
openclaw hooks list

# 启用它
openclaw hooks enable my-hook

# 重启你的网关进程（macOS 上的菜单栏应用重启，或重启你的开发进程）

# 触发事件
# 通过你的消息通道发送 /new
```

## 配置

### 新配置格式（推荐）

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### 每个钩子的配置

钩子可以有自定义配置：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### 额外目录

从额外目录加载钩子：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### 旧配置格式（仍支持）

旧的配置格式为了向后兼容仍然有效：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

注意：`module` 必须是相对于工作区的路径。绝对路径和遍历到工作区之外的路径会被拒绝。**迁移**：对于新钩子，请使用新的基于发现的系统。旧处理器在基于目录的钩子之后加载。

## CLI 命令

### 列出钩子

```bash
# 列出所有钩子
openclaw hooks list

# 仅显示符合条件的钩子
openclaw hooks list --eligible

# 详细输出（显示缺失的要求）
openclaw hooks list --verbose

# JSON 输出
openclaw hooks list --json
```

### 钩子信息

```bash
# 显示钩子的详细信息
openclaw hooks info session-memory

# JSON 输出
openclaw hooks info session-memory --json
```

### 检查资格

```bash
# 显示资格摘要
openclaw hooks check

# JSON 输出
openclaw hooks check --json
```

### 启用/禁用

```bash
# 启用一个钩子
openclaw hooks enable session-memory

# 禁用一个钩子
openclaw hooks disable command-logger
```

## 捆绑钩子参考

### session-memory

当你发出 `/new` 时，将会话上下文保存到记忆。**事件**：`command:new` **要求**：必须配置 `workspace.dir` **输出**：`/memory/YYYY-MM-DD-slug.md`（默认为 `~/.openclaw/workspace`）**功能**：

1.  使用重置前的会话条目来定位正确的记录
2.  提取最后 15 行对话
3.  使用 LLM 生成描述性文件名 slug
4.  将会话元数据保存到带日期的记忆文件中

**示例输出**：

```bash
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**文件名示例**：

-   `2026-01-16-vendor-pitch.md`
-   `2026-01-16-api-design.md`
-   `2026-01-16-1430.md`（如果 slug 生成失败，则回退到时间戳）

**启用**：

```bash
openclaw hooks enable session-memory
```

### bootstrap-extra-files

在 `agent:bootstrap` 期间注入额外的引导文件（例如 monorepo 本地的 `AGENTS.md` / `TOOLS.md`）。**事件**：`agent:bootstrap` **要求**：必须配置 `workspace.dir` **输出**：不写入文件；仅在内存中修改引导上下文。**配置**：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "bootstrap-extra-files": {
          "enabled": true,
          "paths": ["packages/*/AGENTS.md", "packages/*/TOOLS.md"]
        }
      }
    }
  }
}
```

**注意**：

-   路径相对于工作区解析。
-   文件必须保留在工作区内（经过真实路径检查）。
-   仅加载已识别的引导文件名。
-   保留子代理白名单（仅 `AGENTS.md` 和 `TOOLS.md`）。

**启用**：

```bash
openclaw hooks enable bootstrap-extra-files
```

### command-logger

将所有命令事件记录到集中审计文件。**事件**：`command` **要求**：无 **输出**：`~/.openclaw/logs/commands.log` **功能**：

1.  捕获事件详情（命令操作、时间戳、会话键、发送者 ID、来源）
2.  以 JSONL 格式追加到日志文件
3.  在后台静默运行

**示例日志条目**：

```json
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**查看日志**：

```bash
# 查看最近的命令
tail -n 20 ~/.openclaw/logs/commands.log

# 使用 jq 美化打印
cat ~/.openclaw/logs/commands.log | jq .

# 按操作过滤
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**启用**：

```bash
openclaw hooks enable command-logger
```

### boot-md

网关启动时运行 `BOOT.md`（通道启动后）。必须启用内部钩子才能运行。**事件**：`gateway:startup` **要求**：必须配置 `workspace.dir` **功能**：

1.  从你的工作区读取 `BOOT.md`
2.  通过代理运行器运行指令
3.  通过消息工具发送任何请求的出站消息

**启用**：

```bash
openclaw hooks enable boot-md
```

## 最佳实践

### 保持处理器快速

钩子在命令处理期间运行。保持它们轻量级：

```
// ✓ 良好 - 异步工作，立即返回
const handler: HookHandler = async (event) => {
  void processInBackground(event); // 触发后不管
};

// ✗ 不好 - 阻塞命令处理
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### 优雅地处理错误

始终包装有风险的操作：

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Failed:", err instanceof Error ? err.message : String(err));
    // 不要抛出 - 让其他处理器运行
  }
};
```

### 尽早过滤事件

如果事件不相关，尽早返回：

```typescript
const handler: HookHandler = async (event) => {
  // 仅处理 'new' 命令
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // 你的逻辑放在这里
};
```

### 使用特定的事件键

尽可能在元数据中指定确切的事件：

```
metadata: { "openclaw": { "events": ["command:new"] } } # 特定
```

而不是：

```
metadata: { "openclaw": { "events": ["command"] } } # 通用 - 开销更大
```

## 调试

### 启用钩子日志记录

网关在启动时记录钩子加载：

```bash
Registered hook: session-memory -> command:new
Registered hook: bootstrap-extra-files -> agent:bootstrap
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### 检查发现

列出所有发现的钩子：

```bash
openclaw hooks list --verbose
```

### 检查注册

在你的处理器中，记录它被调用的时间：

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // 你的逻辑
};
```

### 验证资格

检查钩子不符合条件的原因：

```bash
openclaw hooks info my-hook
```

在输出中查找缺失的要求。

## 测试

### 网关日志

监控网关日志以查看钩子执行情况：

```bash
# macOS
./scripts/clawlog.sh -f

# 其他平台
tail -f ~/.openclaw/gateway.log
```

### 直接测试钩子

在隔离环境中测试你的处理器：

```typescript
import { test } from "vitest";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = {
    type: "command",
    action: "new",
    sessionKey: "test-session",
    timestamp: new Date(),
    messages: [],
    context: { foo: "bar" },
  };

  await myHandler(event);

  // 断言副作用
});
```

## 架构

### 核心组件

-   **`src/hooks/types.ts`**：类型定义
-   **`src/hooks/workspace.ts`**：目录扫描和加载
-   **`src/hooks/frontmatter.ts`**：HOOK.md 元数据解析
-   **`src/hooks/config.ts`**：资格检查
-   **`src/hooks/hooks-status.ts`**：状态报告
-   **`src/hooks/loader.ts`**：动态模块加载器
-   **`src/cli/hooks-cli.ts`**：CLI 命令
-   **`src/gateway/server-startup.ts`**：在网关启动时加载钩子
-   **`src/auto-reply/reply/commands-core.ts`**：触发命令事件

### 发现流程

```
网关启动
    ↓
扫描目录（工作区 → 托管 → 捆绑）
    ↓
解析 HOOK.md 文件
    ↓
检查资格（二进制文件、环境变量、配置、操作系统）
    ↓
从符合条件的钩子加载处理器
    ↓
为事件注册处理器
```

### 事件流程

```
用户发送 /new
    ↓
命令验证
    ↓
创建钩子事件
    ↓
触发钩子（所有已注册的处理器）
    ↓
命令处理继续
    ↓
会话重置
```

## 故障排除

### 钩子未被发现

1.  检查目录结构：
    
    ```bash
    ls -la ~/.openclaw/hooks/my-hook/
    # 应显示：HOOK.md, handler.ts
    ```
    
2.  验证 HOOK.md 格式：
    
    ```bash
    cat ~/.openclaw/hooks/my-hook/HOOK.md
    # 应具有包含名称和元数据的 YAML 前言
    ```
    
3.  列出所有发现的钩子：
    
    ```bash
    openclaw hooks list
    ```
    

### 钩子不符合条件

检查要求：

```bash
openclaw hooks info my-hook
```

查找缺失的：

-   二进制文件（检查 PATH）
-   环境变量
-   配置值
-   操作系统兼容性

### 钩子未执行

1.  验证钩子已启用：
    
    ```bash
    openclaw hooks list
    # 应在启用的钩子旁边显示 ✓
    ```
    
2.  重启你的网关进程以便钩子重新加载。
3.  检查网关日志中的错误：
    
    ```bash
    ./scripts/clawlog.sh | grep hook
    ```
    

### 处理器错误

检查 TypeScript/导入错误：

```bash
# 直接测试导入
node -e "import('./path/to/handler.ts').then(console.log)"
```

## 迁移指南

### 从旧配置迁移到发现机制

**之前**：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**之后**：

1.  创建钩子目录：
    
    ```bash
    mkdir -p ~/.openclaw/hooks/my-hook
    mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
    ```
    
2.  创建 HOOK.md：
    
    ```
    ---
    name: my-hook
    description: "我的自定义钩子"
    metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
    ---
    
    # 我的钩子
    
    做一些有用的事情。
    ```
    
3.  更新配置：
    
    ```json
    {
      "hooks": {
        "internal": {
          "enabled": true,
          "entries": {
            "my-hook": { "enabled": true }
          }
        }
      }
    }
    ```
    
4.  验证并重启你的网关进程：
    
    ```bash
    openclaw hooks list
    # 应显示：🎯 my-hook ✓
    ```
    

**迁移的好处**：

-   自动发现
-   CLI 管理
-   资格检查
-   更好的文档
-   一致的结构

## 另请参阅

-   [CLI 参考：hooks](../cli/hooks.md)
-   [捆绑钩子 README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
-   [Webhook 钩子](./webhook.md)
-   [配置](../gateway/configuration.md#hooks)

[OpenProse](../prose.md)[Cron 作业](./cron-jobs.md)