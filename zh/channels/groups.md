

  配置

  
# 群组

无论你使用 WhatsApp、Telegram、Discord、Slack、Signal、iMessage、Microsoft Teams 还是 Zalo，OpenClaw 对群聊的处理方式都保持一致。

## 两分钟快速入门

OpenClaw 直接「运行」在你自己的消息账号上——没有独立的机器人用户。只要 **你** 在某个群组里，OpenClaw 就能看到这个群组并在其中响应。默认配置如下：

- 群组处于受限状态（`groupPolicy: "allowlist"`）
- 必须提及才能触发回复，除非你显式关闭提及门控

换句话说：只有白名单中的发送者才能通过 @提及 来唤醒 OpenClaw。

> 一句话总结
> 
> -   **私聊访问** 由 `*.allowFrom` 控制
> -   **群组访问** 由 `*.groupPolicy` + 白名单（`*.groups`、`*.groupAllowFrom`）控制
> -   **回复触发** 由提及门控（`requireMention`、`/activation`）控制

群组消息的处理流程：

```
groupPolicy? disabled -> 丢弃
groupPolicy? allowlist -> 群组在白名单？否 -> 丢弃
requireMention? 是 -> 被提及？否 -> 仅保存为上下文
否则 -> 回复
```

![群组消息流程](../images/channels-groups-flow.svg.md) 想实现某种效果？

| 目标 | 如何配置 |
| --- | --- |
| 允许所有群组，但只在被 @提及 时回复 | `groups: { "*": { requireMention: true } }` |
| 完全禁用群组回复 | `groupPolicy: "disabled"` |
| 只允许特定群组 | `groups: { "<group-id>": { ... } }`（不设置 `"*"` 键） |
| 只有你能触发群组回复 | `groupPolicy: "allowlist"`，`groupAllowFrom: ["+1555..."]` |

## 会话密钥

- 群组会话使用 `agent:::group:` 格式的会话密钥（房间/频道则使用 `agent:::channel:`）
- Telegram 论坛主题会在群组 ID 后追加 `:topic:`，让每个主题拥有独立会话
- 私聊使用主会话（或按发送者配置的会话）
- 群组会话不执行心跳检测

## 典型模式：私聊归私聊，群组归群组（单个智能体）

如果你的「个人」流量集中在 **私聊**，「公共」流量集中在 **群组**，这个模式非常适合你。原因是：在单智能体模式下，私聊消息通常进入 **主会话**（`agent:main:main`），而群组始终使用 **非主会话**（`agent:main::group:`）。如果你启用沙箱并设置 `mode: "non-main"`，群组会话会在 Docker 中运行，而主私聊会话留在主机上。这样你就拥有了一个智能体「大脑」（共享工作空间 + 记忆），同时具备两种执行姿态：

-   **私聊**：完整工具权限（主机）
-   **群组**：沙箱隔离 + 受限工具（Docker）

> 如果你需要完全独立的工作空间或角色（「个人」和「公共」绝不能混用），请使用第二个智能体 + 绑定。参见 [多智能体路由](../concepts/multi-agent.md)。

配置示例（私聊在主机，群组沙箱化 + 仅消息工具）：

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // 群组/频道属于非主会话 -> 沙箱化
        scope: "session", // 最强隔离（每个群组/频道一个容器）
        workspaceAccess: "none",
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        // allow 非空时，其他工具都被阻断（deny 优先级最高）
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"],
      },
    },
  },
}
```

想要「群组只能看到 X 文件夹」而不是「完全禁止主机访问」？保持 `workspaceAccess: "none"`，只把白名单路径挂载进沙箱：

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // 主机路径:容器路径:模式
            "/home/user/FriendsShared:/data:ro",
          ],
        },
      },
    },
  },
}
```

延伸阅读：

-   配置项与默认值：[网关配置](../gateway/configuration.md#agentsdefaultssandbox)
-   调试工具为何被阻断：[沙箱 vs 工具策略 vs 提升权限](../gateway/sandbox-vs-tool-policy-vs-elevated.md)
-   绑定挂载详解：[沙箱机制](../gateway/sandboxing.md#custom-bind-mounts)

## 显示标签

-   界面标签优先使用 `displayName`，格式为 `:`
-   `#room` 格式保留给房间/频道；群聊使用 `g-` 格式（小写，空格转为 `-`，保留 `#@+._-` 字符）

## 群组策略

控制每个频道（channel）如何处理群组/房间消息：

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789"], // 数字 Telegram 用户 ID（向导可解析 @username）
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: { channels: { help: { allow: true } } },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
    },
  },
}
```

| 策略值 | 行为说明 |
| --- | --- |
| `"open"` | 群组绕过白名单检查；提及门控仍然生效 |
| `"disabled"` | 完全阻断所有群组消息 |
| `"allowlist"` | 只允许匹配白名单的群组/房间 |

注意事项：

-   `groupPolicy` 与提及门控（要求 @提及）是两套独立的机制
-   WhatsApp/Telegram/Signal/iMessage/Microsoft Teams/Zalo：使用 `groupAllowFrom`（若未设置则回退到 `allowFrom`）
-   私聊配对审批（`*-allowFrom` 存储条目）仅适用于私聊访问；群组发送者授权需单独配置群组白名单
-   Discord：白名单使用 `channels.discord.guilds..channels`
-   Slack：白名单使用 `channels.slack.channels`
-   Matrix：白名单使用 `channels.matrix.groups`（支持房间 ID、别名或名称）。用 `channels.matrix.groupAllowFrom` 限制发送者；也支持每个房间的 `users` 白名单
-   群组私聊单独控制（`channels.discord.dm.*`、`channels.slack.dm.*`）
-   Telegram 白名单支持用户 ID（`"123456789"`、`"telegram:123456789"`、`"tg:123456789"`）或用户名（`"@alice"` 或 `"alice"`）；前缀不区分大小写
-   默认值为 `groupPolicy: "allowlist"`；如果群组白名单为空，群组消息将被阻断
-   运行时安全：当整个频道配置块缺失（`channels.` 不存在）时，群组策略会回退到故障关闭模式（通常是 `allowlist`），而不是继承 `channels.defaults.groupPolicy`

快速理解（群组消息的评估顺序）：

1.  `groupPolicy`（open/disabled/allowlist）
2.  群组白名单（`*.groups`、`*.groupAllowFrom`、频道特定白名单）
3.  提及门控（`requireMention`、`/activation`）

## 提及门控（默认启用）

默认情况下，群组消息必须包含提及才会触发回复，除非针对特定群组单独覆盖。默认配置位于各子系统的 `*.groups."*"` 下。回复机器人的消息算作隐式提及（当频道支持回复元数据时）。此机制适用于 Telegram、WhatsApp、Slack、Discord 和 Microsoft Teams。

```json
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false },
      },
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false },
      },
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50,
        },
      },
    ],
  },
}
```

注意事项：

-   `mentionPatterns` 是不区分大小写的正则表达式
-   支持原生提及的平台仍然优先使用原生提及；模式只是兜底方案
-   按智能体覆盖：`agents.list[].groupChat.mentionPatterns`（多个智能体共享群组时很有用）
-   仅当提及检测可用时（配置了原生提及或 `mentionPatterns`）才强制执行提及门控
-   Discord 的默认值位于 `channels.discord.guilds."*"`（可按公会/频道覆盖）
-   群组历史上下文在各频道间统一处理，且 **仅保留待处理消息**（因提及门控跳过的消息）；全局默认值用 `messages.groupChat.historyLimit`，覆盖用 `channels..historyLimit`（或 `channels..accounts.*.historyLimit`）。设为 `0` 禁用

## 群组/频道工具限制（可选）

部分频道支持在 **特定群组/房间/频道内** 限制可用工具。

-   `tools`：针对整个群组允许/拒绝工具
-   `toolsBySender`：群组内按发送者覆盖。使用显式键前缀：`id:`、`e164:`、`username:`、`name:`，以及 `"*"` 通配符。旧的无前缀键仍然支持，但只按 `id:` 匹配

解析顺序（最具体优先）：

1.  群组/频道 `toolsBySender` 匹配
2.  群组/频道 `tools`
3.  默认（`"*"`）`toolsBySender` 匹配
4.  默认（`"*"`）`tools`

示例（Telegram）：

```json
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "id:123456789": { alsoAllow: ["exec"] },
          },
        },
      },
    },
  },
}
```

注意事项：

-   群组/频道工具限制在全局/智能体工具策略之外额外应用（deny 优先级最高）
-   部分频道对房间/频道使用不同的嵌套结构（如 Discord `guilds.*.channels.*`、Slack `channels.*`、MS Teams `teams.*.channels.*`）

## 群组白名单

当配置了 `channels.whatsapp.groups`、`channels.telegram.groups` 或 `channels.imessage.groups` 时，其中的键就是群组白名单。使用 `"*"` 可以允许所有群组，同时仍然设置默认的提及行为。常见配置模板（可直接复制）：

1.  禁用所有群组回复

```json
{
  channels: { whatsapp: { groupPolicy: "disabled" } },
}
```

2.  只允许特定群组（WhatsApp）

```json
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false },
      },
    },
  },
}
```

3.  允许所有群组但必须提及（显式配置）

```json
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

4.  只有所有者能在群组中触发（WhatsApp）

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## 激活命令（仅限所有者）

群组所有者可以在群内发送命令切换激活模式：

-   `/activation mention` —— 需要提及才响应
-   `/activation always` —— 始终响应

所有者身份由 `channels.whatsapp.allowFrom` 确定（未设置时使用机器人自身的 E.164 号码）。命令需作为独立消息发送。其他平台目前忽略 `/activation` 命令。

## 上下文字段

群组入站消息会设置以下字段：

-   `ChatType=group`
-   `GroupSubject`（群组名称，如已知）
-   `GroupMembers`（群成员列表，如已知）
-   `WasMentioned`（提及门控检测结果）
-   Telegram 论坛主题还会包含 `MessageThreadId` 和 `IsForum`

智能体的系统提示会在新群组会话的首次交互中插入群组介绍，提醒模型像真人一样回复、避免使用 Markdown 表格、不要输出字面的 `\n` 序列。

## iMessage 特别说明

-   路由或白名单配置中，推荐使用 `chat_id:` 格式
-   列出所有聊天：`imsg chats --limit 20`
-   群组回复总是发回同一个 `chat_id`

## WhatsApp 特别说明

WhatsApp 专有行为（历史记录注入、提及处理细节）请参阅 [群组消息](./group-messages.md)。

[群组消息](./group-messages.md)[广播群组](./broadcast-groups.md)