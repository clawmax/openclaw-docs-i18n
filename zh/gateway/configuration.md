

  配置与运维

  
# 配置

OpenClaw 从 `~/.openclaw/openclaw.json` 读取一个可选的 **JSON5** 配置文件。如果文件不存在，OpenClaw 会使用安全的默认值。通常需要添加配置的原因包括：

-   连接频道并控制谁可以向机器人发送消息
-   设置模型、工具、沙箱或自动化功能（定时任务、钩子）
-   调整会话、媒体、网络或用户界面

查看[完整参考文档](./configuration-reference.md)了解所有可用字段。

> **💡** **初次接触配置？** 可以从 `openclaw onboard` 开始进行交互式设置，或者查看[配置示例](./configuration-examples.md)指南获取完整的可复制粘贴配置。

## 最小配置

```
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## 编辑配置

```bash
openclaw onboard       # 完整设置向导
openclaw configure     # 配置向导
```

```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```

打开 [http://127.0.0.1:18789](http://127.0.0.1:18789) 并使用 **配置** 标签页。控制界面会根据配置模式渲染表单，并提供一个 **原始 JSON** 编辑器作为备用方案。

直接编辑 `~/.openclaw/openclaw.json`。网关会监视该文件并自动应用更改（参见[热重载](#config-hot-reload)）。

## 严格验证

> **⚠️** OpenClaw 只接受完全符合模式的配置。未知的键、格式错误的类型或无效的值会导致网关**拒绝启动**。唯一的根级别例外是 `$schema`（字符串），以便编辑器可以附加 JSON 模式元数据。

 当验证失败时：

-   网关不会启动
-   只有诊断命令有效（`openclaw doctor`、`openclaw logs`、`openclaw health`、`openclaw status`）
-   运行 `openclaw doctor` 查看具体问题
-   运行 `openclaw doctor --fix`（或 `--yes`）来应用修复

## 常见任务

每个频道在 `channels.` 下都有自己的配置部分。查看专用频道页面获取设置步骤：

-   [WhatsApp](../channels/whatsapp.md) — `channels.whatsapp`
-   [Telegram](../channels/telegram.md) — `channels.telegram`
-   [Discord](../channels/discord.md) — `channels.discord`
-   [Slack](../channels/slack.md) — `channels.slack`
-   [Signal](../channels/signal.md) — `channels.signal`
-   [iMessage](../channels/imessage.md) — `channels.imessage`
-   [Google Chat](../channels/googlechat.md) — `channels.googlechat`
-   [Mattermost](../channels/mattermost.md) — `channels.mattermost`
-   [MS Teams](../channels/msteams.md) — `channels.msteams`

所有频道共享相同的私信策略模式：

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

设置主模型和可选的备用模型：

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

-   `agents.defaults.models` 定义了模型目录，并作为 `/model` 命令的允许列表。
-   模型引用使用 `provider/model` 格式（例如 `anthropic/claude-opus-4-6`）。
-   `agents.defaults.imageMaxDimensionPx` 控制转录/工具图像的降尺度（默认 `1200`）；较低的值通常可以减少截图密集型运行时的视觉令牌使用量。
-   查看[模型 CLI](../concepts/models.md) 了解如何在聊天中切换模型，以及[模型故障转移](../concepts/model-failover.md)了解身份验证轮换和备用模型行为。
-   对于自定义/自托管提供商，请参考参考文档中的[自定义提供商](./configuration-reference.md#custom-providers-and-base-urls)。

私信访问权限通过每个频道的 `dmPolicy` 控制：

-   `"pairing"`（默认）：未知发送者会收到一次性配对码进行批准
-   `"allowlist"`：只允许 `allowFrom` 中的发送者（或已配对的允许存储）
-   `"open"`：允许所有入站私信（需要 `allowFrom: ["*"]`）
-   `"disabled"`：忽略所有私信

对于群组，使用 `groupPolicy` + `groupAllowFrom` 或特定频道的允许列表。查看[完整参考文档](./configuration-reference.md#dm-and-group-access)了解每个频道的详细信息。

群组消息默认**需要提及**。为每个智能体配置模式：

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

-   **元数据提及**：原生的 @-提及（WhatsApp 点击提及、Telegram @bot 等）
-   **文本模式**：`mentionPatterns` 中的正则表达式模式
-   查看[完整参考文档](./configuration-reference.md#group-chat-mention-gating)了解每个频道的覆盖规则和自聊天模式。

会话控制对话的连续性和隔离性：

```json
{
  session: {
    dmScope: "per-channel-peer",  // 多用户推荐使用
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

-   `dmScope`：`main`（共享）| `per-peer` | `per-channel-peer` | `per-account-channel-peer`
-   `threadBindings`：线程绑定会话路由的全局默认值（Discord 支持 `/focus`、`/unfocus`、`/agents`、`/session idle` 和 `/session max-age`）。
-   查看[会话管理](../concepts/session.md)了解作用域、身份链接和发送策略。
-   查看[完整参考文档](./configuration-reference.md#session)了解所有字段。

在隔离的 Docker 容器中运行智能体会话：

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

首先构建镜像：`scripts/sandbox-setup.sh`。查看[沙箱](./sandboxing.md)获取完整指南，以及[完整参考文档](./configuration-reference.md#sandbox)获取所有选项。

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

-   `every`：持续时间字符串（`30m`、`2h`）。设置为 `0m` 以禁用。
-   `target`：`last` | `whatsapp` | `telegram` | `discord` | `none`
-   `directPolicy`：私信风格心跳目标的 `allow`（默认）或 `block`
-   查看[心跳](./heartbeat.md)获取完整指南。

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

-   `sessionRetention`：从 `sessions.json` 中清理已完成的隔离运行会话（默认 `24h`；设置为 `false` 以禁用）。
-   `runLog`：按大小和保留行数清理 `cron/runs/.jsonl`。
-   查看[定时任务](../automation/cron-jobs.md)获取功能概述和 CLI 示例。

在网关上启用 HTTP webhook 端点：

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

安全说明：

-   将所有钩子/webhook 有效负载内容视为不可信输入。
-   除非进行严格限定范围的调试，否则保持不安全内容绕过标志处于禁用状态（`hooks.gmail.allowUnsafeExternalContent`、`hooks.mappings[].allowUnsafeExternalContent`）。
-   对于钩子驱动的智能体，建议使用强大的现代模型层级和严格的工具策略（例如，仅限消息传递，并在可能的情况下使用沙箱）。

查看[完整参考文档](./configuration-reference.md#hooks)获取所有映射选项和 Gmail 集成信息。

运行多个具有独立工作空间和会话的隔离智能体：

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

查看[多智能体](../concepts/multi-agent.md)和[完整参考文档](./configuration-reference.md#multi-agent-routing)了解绑定规则和每个智能体的访问配置文件。

使用 `$include` 来组织大型配置：

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

-   **单个文件**：替换包含的对象
-   **文件数组**：按顺序深度合并（后出现的优先）
-   **同级键**：在包含之后合并（覆盖已包含的值）
-   **嵌套包含**：支持最多 10 层深度
-   **相对路径**：相对于包含文件解析
-   **错误处理**：对缺失文件、解析错误和循环包含提供清晰的错误信息

## 配置热重载

网关监视 `~/.openclaw/openclaw.json` 并自动应用更改 —— 大多数设置无需手动重启。

### 重载模式

| 模式 | 行为 |
| --- | --- |
| **`hybrid`**（默认） | 即时热应用安全更改。对于关键更改自动重启。 |
| **`hot`** | 仅热应用安全更改。当需要重启时记录警告 —— 由您手动处理。 |
| **`restart`** | 任何配置更改（无论是否安全）都会重启网关。 |
| **`off`** | 禁用文件监视。更改在下一次手动重启时生效。 |

```json
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### 可热应用与需要重启的内容

大多数字段可以热应用而无需停机。在 `hybrid` 模式下，需要重启的更改会自动处理。

| 类别 | 字段 | 需要重启？ |
| --- | --- | --- |
| 频道 | `channels.*`、`web`（WhatsApp）—— 所有内置和扩展频道 | 否 |
| 智能体与模型 | `agent`、`agents`、`models`、`routing` | 否 |
| 自动化 | `hooks`、`cron`、`agent.heartbeat` | 否 |
| 会话与消息 | `session`、`messages` | 否 |
| 工具与媒体 | `tools`、`browser`、`skills`、`audio`、`talk` | 否 |
| 用户界面与杂项 | `ui`、`logging`、`identity`、`bindings` | 否 |
| 网关服务器 | `gateway.*`（端口、绑定、身份验证、tailscale、TLS、HTTP） | **是** |
| 基础设施 | `discovery`、`canvasHost`、`plugins` | **是** |

> **ℹ️** `gateway.reload` 和 `gateway.remote` 是例外 —— 更改它们**不会**触发重启。

## 配置 RPC（编程式更新）

> **ℹ️** 控制平面写入 RPC（`config.apply`、`config.patch`、`update.run`）的速率限制为每个 `deviceId+clientIp` **每 60 秒 3 个请求**。当被限制时，RPC 返回 `UNAVAILABLE` 并附带 `retryAfterMs`。

 

验证 + 写入完整配置并一步重启网关。

`config.apply` 替换**整个配置**。对于部分更新，请使用 `config.patch`，或者对于单个键，使用 `openclaw config set`。

参数：

-   `raw`（字符串）—— 整个配置的 JSON5 有效负载
-   `baseHash`（可选）—— 来自 `config.get` 的配置哈希（当配置存在时必需）
-   `sessionKey`（可选）—— 用于重启后唤醒 ping 的会话键
-   `note`（可选）—— 重启哨兵的备注
-   `restartDelayMs`（可选）—— 重启前的延迟（默认 2000）

当已经有一个重启请求待处理/进行中时，重启请求会被合并，并且在重启周期之间有 30 秒的冷却时间。

```bash
openclaw gateway call config.get --params '{}'  # 捕获 payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
  "baseHash": "<hash>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123"
}'
```

将部分更新合并到现有配置中（JSON 合并补丁语义）：

-   对象递归合并
-   `null` 删除键
-   数组替换

参数：

-   `raw`（字符串）—— 仅包含要更改的键的 JSON5
-   `baseHash`（必需）—— 来自 `config.get` 的配置哈希
-   `sessionKey`、`note`、`restartDelayMs` —— 与 `config.apply` 相同

重启行为与 `config.apply` 匹配：合并待处理的重启，并且在重启周期之间有 30 秒的冷却时间。

```bash
openclaw gateway call config.patch --params '{
  "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
  "baseHash": "<hash>"
}'
```

## 环境变量

OpenClaw 从父进程读取环境变量，以及：

-   当前工作目录中的 `.env` 文件（如果存在）
-   `~/.openclaw/.env`（全局回退）

这两个文件都不会覆盖已存在的环境变量。您也可以在配置中设置内联环境变量：

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

如果启用且预期的键未设置，OpenClaw 会运行您的登录 shell 并仅导入缺失的键：

```json
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

环境变量等效项：`OPENCLAW_LOAD_SHELL_ENV=1`

 

使用 `${VAR_NAME}` 在任何配置字符串值中引用环境变量：

```json
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

规则：

-   仅匹配大写名称：`[A-Z_][A-Z0-9_]*`
-   缺失/空的变量在加载时抛出错误
-   使用 `$${VAR}` 进行转义以输出字面值
-   在 `$include` 文件内部有效
-   内联替换：`"${BASE}/v1"` → `"https://api.example.com/v1"`

 

对于支持 SecretRef 对象的字段，您可以使用：

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

SecretRef 详细信息（包括用于 `env`/`file`/`exec` 的 `secrets.providers`）请参见[密钥管理](./secrets.md)。支持的凭据路径列在[SecretRef 凭据范围](../reference/secretref-credential-surface.md)中。

 查看[环境](../help/environment.md)了解完整的优先级和来源。

## 完整参考

要获取完整的字段参考，请参见**[配置参考文档](./configuration-reference.md)**。

* * *

*相关链接：[配置示例](./configuration-examples.md) · [配置参考文档](./configuration-reference.md) · [Doctor](./doctor.md)*

[网关运行手册](../gateway.md)[配置参考文档](./configuration-reference.md)