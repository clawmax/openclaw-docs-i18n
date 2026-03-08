

  代理协调

  
# ACP 代理

[代理客户端协议 (ACP)](https://agentclientprotocol.com/) 会话允许 OpenClaw 通过 ACP 后端插件运行外部编码工具（例如 Pi、Claude Code、Codex、OpenCode 和 Gemini CLI）。如果您用自然语言要求 OpenClaw“在 Codex 中运行这个”或“在主题中启动 Claude Code”，OpenClaw 应该将该请求路由到 ACP 运行时（而非原生子代理运行时）。

## 快速操作流程

当您需要一个实用的 `/acp` 操作手册时使用此流程：

1.  生成会话：
    -   `/acp spawn codex --mode persistent --thread auto`
2.  在绑定的主题中工作（或显式指定该会话键）。
3.  检查运行时状态：
    -   `/acp status`
4.  根据需要调整运行时选项：
    -   `/acp model <provider/model>`
    -   `/acp permissions `
    -   `/acp timeout `
5.  在不替换上下文的情况下引导活动会话：
    -   `/acp steer tighten logging and continue`
6.  停止工作：
    -   `/acp cancel`（停止当前轮次），或
    -   `/acp close`（关闭会话 + 移除绑定）

## 人类快速入门

自然请求示例：

-   “在此处主题中启动一个持久的 Codex 会话并保持其专注。”
-   “将此作为一次性 Claude Code ACP 会话运行并总结结果。”
-   “在主题中使用 Gemini CLI 处理此任务，然后将后续跟进保留在同一主题中。”

OpenClaw 应执行的操作：

1.  选择 `runtime: "acp"`。
2.  解析请求的工具目标（`agentId`，例如 `codex`）。
3.  如果请求了主题绑定且当前频道支持，则将 ACP 会话绑定到主题。
4.  将后续主题消息路由到同一 ACP 会话，直到失焦/关闭/过期。

## ACP 与子代理对比

当您需要外部工具运行时，使用 ACP。当您需要 OpenClaw 原生委托运行时，使用子代理。

| 领域 | ACP 会话 | 子代理运行 |
| --- | --- | --- |
| 运行时 | ACP 后端插件（例如 acpx） | OpenClaw 原生子代理运行时 |
| 会话键 | `agent::acp:` | `agent::subagent:` |
| 主要命令 | `/acp ...` | `/subagents ...` |
| 生成工具 | `sessions_spawn` 附带 `runtime:"acp"` | `sessions_spawn`（默认运行时） |

另请参阅[子代理](./subagents.md)。

## 主题绑定会话（频道无关）

当频道适配器启用主题绑定时，ACP 会话可以绑定到主题：

-   OpenClaw 将主题绑定到目标 ACP 会话。
-   该主题中的后续消息将路由到绑定的 ACP 会话。
-   ACP 输出将传回同一主题。
-   失焦/关闭/归档/空闲超时或最长有效期到期将移除绑定。

主题绑定支持是适配器特定的。如果活动频道适配器不支持主题绑定，OpenClaw 将返回明确的不支持/不可用消息。主题绑定 ACP 所需的功能标志：

-   `acp.enabled=true`
-   `acp.dispatch.enabled` 默认开启（设置为 `false` 可暂停 ACP 分发）
-   频道适配器 ACP 主题生成标志已启用（适配器特定）
    -   Discord：`channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram：`channels.telegram.threadBindings.spawnAcpSessions=true`

### 支持主题的频道

-   任何公开会话/主题绑定能力的频道适配器。
-   当前内置支持：
    -   Discord 主题/频道
    -   Telegram 话题（群组/超级群组中的论坛话题和私信话题）
-   插件频道可以通过相同的绑定接口添加支持。

## 频道特定设置

对于非临时工作流，请在顶层 `bindings[]` 条目中配置持久的 ACP 绑定。

### 绑定模型

-   `bindings[].type="acp"` 标记一个持久的 ACP 对话绑定。
-   `bindings[].match` 标识目标对话：
    -   Discord 频道或主题：`match.channel="discord"` + `match.peer.id=""`
    -   Telegram 论坛话题：`match.channel="telegram"` + `match.peer.id=":topic:"`
-   `bindings[].agentId` 是所属的 OpenClaw 代理 ID。
-   可选的 ACP 覆盖项位于 `bindings[].acp` 下：
    -   `mode`（`persistent` 或 `oneshot`）
    -   `label`
    -   `cwd`
    -   `backend`

### 每个代理的运行时默认值

使用 `agents.list[].runtime` 为每个代理定义一次 ACP 默认值：

-   `agents.list[].runtime.type="acp"`
-   `agents.list[].runtime.acp.agent`（工具 ID，例如 `codex` 或 `claude`）
-   `agents.list[].runtime.acp.backend`
-   `agents.list[].runtime.acp.mode`
-   `agents.list[].runtime.acp.cwd`

ACP 绑定会话的覆盖优先级：

1.  `bindings[].acp.*`
2.  `agents.list[].runtime.acp.*`
3.  全局 ACP 默认值（例如 `acp.backend`）

示例：

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

行为：

-   OpenClaw 确保在使用前配置的 ACP 会话存在。
-   该频道或话题中的消息将路由到配置的 ACP 会话。
-   在绑定对话中，`/new` 和 `/reset` 会原地重置相同的 ACP 会话键。
-   临时运行时绑定（例如由主题聚焦流创建的）在存在时仍然适用。

## 启动 ACP 会话（接口）

### 从 sessions\_spawn

使用 `runtime: "acp"` 从代理轮次或工具调用启动 ACP 会话。

```json
{
  "task": "Open the repo and summarize failing tests",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
```

注意：

-   `runtime` 默认为 `subagent`，因此对于 ACP 会话需显式设置 `runtime: "acp"`。
-   如果省略 `agentId`，OpenClaw 将在配置时使用 `acp.defaultAgent`。
-   `mode: "session"` 需要 `thread: true` 以保持持久的绑定对话。

接口详情：

-   `task`（必需）：发送到 ACP 会话的初始提示。
-   `runtime`（ACP 必需）：必须为 `"acp"`。
-   `agentId`（可选）：ACP 目标工具 ID。如果已设置，则回退到 `acp.defaultAgent`。
-   `thread`（可选，默认 `false`）：在支持的地方请求主题绑定流。
-   `mode`（可选）：`run`（一次性）或 `session`（持久）。
    -   默认为 `run`
    -   如果 `thread: true` 且省略 mode，OpenClaw 可能根据运行时路径默认采用持久行为
    -   `mode: "session"` 需要 `thread: true`
-   `cwd`（可选）：请求的运行时工作目录（由后端/运行时策略验证）。
-   `label`（可选）：操作员面向的标签，用于会话/横幅文本。
-   `streamTo`（可选）：`"parent"` 将初始 ACP 运行进度摘要流式传输回请求者会话作为系统事件。
    -   当可用时，接受的响应包括指向会话范围的 JSONL 日志（`.acp-stream.jsonl`）的 `streamLogPath`，您可以跟踪该日志以获取完整的转发历史记录。

## 沙盒兼容性

ACP 会话当前在主机运行时上运行，不在 OpenClaw 沙盒内。当前限制：

-   如果请求者会话已沙盒化，则阻止 ACP 生成。
    -   错误：`Sandboxed sessions cannot spawn ACP sessions because runtime="acp" runs on the host. Use runtime="subagent" from sandboxed sessions.`
-   带有 `runtime: "acp"` 的 `sessions_spawn` 不支持 `sandbox: "require"`。
    -   错误：`sessions_spawn sandbox="require" is unsupported for runtime="acp" because ACP sessions run outside the sandbox. Use runtime="subagent" or sandbox="inherit".`

当您需要沙盒强制执行的执行时，请使用 `runtime: "subagent"`。

### 从 /acp 命令

需要时，使用 `/acp spawn` 从聊天中进行显式操作员控制。

```bash
/acp spawn codex --mode persistent --thread auto
/acp spawn codex --mode oneshot --thread off
/acp spawn codex --thread here
```

关键标志：

-   `--mode persistent|oneshot`
-   `--thread auto|here|off`
-   `--cwd <absolute-path>`
-   `--label `

请参阅[斜杠命令](./slash-commands.md)。

## 会话目标解析

大多数 `/acp` 操作接受可选的会话目标（`session-key`、`session-id` 或 `session-label`）。解析顺序：

1.  显式目标参数（或 `/acp steer` 的 `--session`）
    -   尝试键
    -   然后是 UUID 格式的会话 ID
    -   然后是标签
2.  当前主题绑定（如果此对话/主题绑定到 ACP 会话）
3.  当前请求者会话回退

如果未解析到任何目标，OpenClaw 将返回明确的错误（`Unable to resolve session target: ...`）。

## 生成主题模式

`/acp spawn` 支持 `--thread auto|here|off`。

| 模式 | 行为 |
| --- | --- |
| `auto` | 在活动主题中：绑定该主题。在主题外：在支持时创建/绑定子主题。 |
| `here` | 要求当前活动主题；如果不在主题中则失败。 |
| `off` | 无绑定。会话以未绑定状态启动。 |

注意：

-   在非主题绑定表面上，默认行为实际上是 `off`。
-   主题绑定生成需要频道策略支持：
    -   Discord：`channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram：`channels.telegram.threadBindings.spawnAcpSessions=true`

## ACP 控制

可用的命令族：

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

`/acp status` 显示有效的运行时选项，并在可用时显示运行时级别和后端级别的会话标识符。某些控制取决于后端能力。如果后端不支持某个控制，OpenClaw 将返回明确的不支持控制错误。

## ACP 命令速查表

| 命令 | 功能 | 示例 |
| --- | --- | --- |
| `/acp spawn` | 创建 ACP 会话；可选主题绑定。 | `/acp spawn codex --mode persistent --thread auto --cwd /repo` |
| `/acp cancel` | 取消目标会话的进行中轮次。 | `/acp cancel agent:codex:acp:` |
| `/acp steer` | 向正在运行的会话发送引导指令。 | `/acp steer --session support inbox prioritize failing tests` |
| `/acp close` | 关闭会话并解除主题目标绑定。 | `/acp close` |
| `/acp status` | 显示后端、模式、状态、运行时选项、能力。 | `/acp status` |
| `/acp set-mode` | 为目标会话设置运行时模式。 | `/acp set-mode plan` |
| `/acp set` | 通用运行时配置选项写入。 | `/acp set model openai/gpt-5.2` |
| `/acp cwd` | 设置运行时工作目录覆盖。 | `/acp cwd /Users/user/Projects/repo` |
| `/acp permissions` | 设置批准策略配置文件。 | `/acp permissions strict` |
| `/acp timeout` | 设置运行时超时（秒）。 | `/acp timeout 120` |
| `/acp model` | 设置运行时模型覆盖。 | `/acp model anthropic/claude-opus-4-5` |
| `/acp reset-options` | 移除会话运行时选项覆盖。 | `/acp reset-options` |
| `/acp sessions` | 列出存储中的最近 ACP 会话。 | `/acp sessions` |
| `/acp doctor` | 后端健康状态、能力、可操作的修复。 | `/acp doctor` |
| `/acp install` | 打印确定性安装和启用步骤。 | `/acp install` |

## 运行时选项映射

`/acp` 有便捷命令和通用设置器。等效操作：

-   `/acp model ` 映射到运行时配置键 `model`。
-   `/acp permissions ` 映射到运行时配置键 `approval_policy`。
-   `/acp timeout ` 映射到运行时配置键 `timeout`。
-   `/acp cwd ` 直接更新运行时 cwd 覆盖。
-   `/acp set  ` 是通用路径。
    -   特殊情况：`key=cwd` 使用 cwd 覆盖路径。
-   `/acp reset-options` 清除目标会话的所有运行时覆盖。

## acpx 工具支持（当前）

当前 acpx 内置工具别名：

-   `pi`
-   `claude`
-   `codex`
-   `opencode`
-   `gemini`
-   `kimi`

当 OpenClaw 使用 acpx 后端时，除非您的 acpx 配置定义了自定义代理别名，否则优先使用这些值作为 `agentId`。直接的 acpx CLI 使用也可以通过 `--agent ` 定位任意适配器，但该原始转义机制是 acpx CLI 功能（不是正常的 OpenClaw `agentId` 路径）。

## 必需配置

核心 ACP 基线：

```json
{
  acp: {
    enabled: true,
    // 可选。默认为 true；设置为 false 可在保持 /acp 控制的同时暂停 ACP 分发。
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

主题绑定配置是频道适配器特定的。Discord 示例：

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

如果主题绑定 ACP 生成不工作，请首先验证适配器功能标志：

-   Discord：`channels.discord.threadBindings.spawnAcpSessions=true`

请参阅[配置参考](../gateway/configuration-reference.md)。

## acpx 后端插件设置

安装并启用插件：

```bash
openclaw plugins install acpx
openclaw config set plugins.entries.acpx.enabled true
```

开发期间的本地工作区安装：

```bash
openclaw plugins install ./extensions/acpx
```

然后验证后端健康状态：

```bash
/acp doctor
```

### acpx 命令和版本配置

默认情况下，acpx 插件（发布为 `@openclaw/acpx`）使用插件本地固定的二进制文件：

1.  命令默认为 `extensions/acpx/node_modules/.bin/acpx`。
2.  预期版本默认为扩展固定版本。
3.  启动时立即将 ACP 后端注册为未就绪。
4.  后台确保作业验证 `acpx --version`。
5.  如果插件本地二进制文件缺失或不匹配，则运行：`npm install --omit=dev --no-save acpx@` 并重新验证。

您可以在插件配置中覆盖命令/版本：

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

注意：

-   `command` 接受绝对路径、相对路径或命令名称（`acpx`）。
-   相对路径从 OpenClaw 工作区目录解析。
-   `expectedVersion: "any"` 禁用严格的版本匹配。
-   当 `command` 指向自定义二进制文件/路径时，插件本地自动安装被禁用。
-   OpenClaw 启动在后端健康检查运行时保持非阻塞。

请参阅[插件](./plugin.md)。

## 权限配置

ACP 会话以非交互方式运行——没有 TTY 来批准或拒绝文件写入和 shell 执行权限提示。acpx 插件提供两个配置键来控制权限处理方式：

### permissionMode

控制工具代理可以在不提示的情况下执行哪些操作。

| 值 | 行为 |
| --- | --- |
| `approve-all` | 自动批准所有文件写入和 shell 命令。 |
| `approve-reads` | 仅自动批准读取；写入和执行需要提示。 |
| `deny-all` | 拒绝所有权限提示。 |

### nonInteractivePermissions

控制在显示权限提示但没有交互式 TTY 可用时发生的情况（ACP 会话始终如此）。

| 值 | 行为 |
| --- | --- |
| `fail` | 以 `AcpRuntimeError` 中止会话。**（默认）** |
| `deny` | 静默拒绝权限并继续（优雅降级）。 |

### 配置

通过插件配置设置：

```bash
openclaw config set plugins.entries.acpx.config.permissionMode approve-all
openclaw config set plugins.entries.acpx.config.nonInteractivePermissions fail
```

更改这些值后重启网关。

> **重要：** OpenClaw 当前默认为 `permissionMode=approve-reads` 和 `nonInteractivePermissions=fail`。在非交互式 ACP 会话中，任何触发权限提示的写入或执行都可能失败，并显示 `AcpRuntimeError: Permission prompt unavailable in non-interactive mode`。如果您需要限制权限，请将 `nonInteractivePermissions` 设置为 `deny`，以便会话优雅降级而不是崩溃。

## 故障排除

| 症状 | 可能原因 | 修复方法 |
| --- | --- | --- |
| `ACP runtime backend is not configured` | 后端插件缺失或禁用。 | 安装并启用后端插件，然后运行 `/acp doctor`。 |
| `ACP is disabled by policy (acp.enabled=false)` | ACP 全局禁用。 | 设置 `acp.enabled=true`。 |
| `ACP dispatch is disabled by policy (acp.dispatch.enabled=false)` | 来自正常主题消息的分发已禁用。 | 设置 `acp.dispatch.enabled=true`。 |
| `ACP agent "" is not allowed by policy` | 代理不在允许列表中。 | 使用允许的 `agentId` 或更新 `acp.allowedAgents`。 |
| `Unable to resolve session target: ...` | 错误的键/ID/标签令牌。 | 运行 `/acp sessions`，复制确切的键/标签，重试。 |
| `--thread here requires running /acp spawn inside an active ... thread` | 在主题上下文外使用了 `--thread here`。 | 移动到目标主题或使用 `--thread auto`/`off`。 |
| `Only <user-id> can rebind this thread.` | 其他用户拥有主题绑定。 | 作为所有者重新绑定或使用不同的主题。 |
| `Thread bindings are unavailable for .` | 适配器缺乏主题绑定能力。 | 使用 `--thread off` 或移动到支持的适配器/频道。 |
| `Sandboxed sessions cannot spawn ACP sessions ...` | ACP 运行时在主机端；请求者会话已沙盒化。 | 从沙盒化会话使用 `runtime="subagent"`，或从非沙盒化会话运行 ACP 生成。 |
| `sessions_spawn sandbox="require" is unsupported for runtime="acp" ...` | 为 ACP 运行时请求了 `sandbox="require"`。 | 对于必需的沙盒化使用 `runtime="subagent"`，或从非沙盒化会话使用带有 `sandbox="inherit"` 的 ACP。 |
| 绑定会话缺少 ACP 元数据 | 过时/已删除的 ACP 会话元数据。 | 使用 `/acp spawn` 重新创建，然后重新绑定/聚焦主题。 |
| `AcpRuntimeError: Permission prompt unavailable in non-interactive mode` | `permissionMode` 在非交互式 ACP 会话中阻止写入/执行。 | 将 `plugins.entries.acpx.config.permissionMode` 设置为 `approve-all` 并重启网关。请参阅[权限配置](#permission-configuration)。 |
| ACP 会话早期失败且输出很少 | 权限提示被 `permissionMode`/`nonInteractivePermissions` 阻止。 | 检查网关日志中的 `AcpRuntimeError`。对于完全权限，设置 `permissionMode=approve-all`；对于优雅降级，设置 `nonInteractivePermissions=deny`。 |
| ACP 会话在工作完成后无限期停滞 | 工具进程已完成但 ACP 会话未报告完成。 | 使用 `ps aux \| grep acpx` 监控；手动终止陈旧进程。 |

[子代理](./subagents.md)[多代理沙盒与工具](./multi-agent-sandbox-tools.md)