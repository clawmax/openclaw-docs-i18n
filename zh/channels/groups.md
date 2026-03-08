

  配置

  
# 群组

OpenClaw 在各种平台上对群聊的处理保持一致：WhatsApp、Telegram、Discord、Slack、Signal、iMessage、Microsoft Teams、Zalo。

## 初学者介绍（2分钟）

OpenClaw “运行”在您自己的消息账户上。没有独立的 WhatsApp 机器人用户。如果 **您** 在一个群组中，OpenClaw 就能看到该群组并在那里响应。默认行为：

-   群组是受限的 (`groupPolicy: "allowlist"`)。
-   回复需要提及，除非您明确禁用提及门控。

翻译：允许列表中的发送者可以通过提及 OpenClaw 来触发它。

> 太长不看版
> 
> -   **私信访问** 由 `*.allowFrom` 控制。
> -   **群组访问** 由 `*.groupPolicy` + 允许列表 (`*.groups`, `*.groupAllowFrom`) 控制。
> -   **回复触发** 由提及门控 (`requireMention`, `/activation`) 控制。

快速流程（群组消息的处理过程）：

```
groupPolicy? disabled -> 丢弃
groupPolicy? allowlist -> 群组允许？否 -> 丢弃
requireMention? 是 -> 被提及？否 -> 仅存储为上下文
否则 -> 回复
```

![群组消息流程](../images/channels-groups-flow.svg.md) 如果您想…

| 目标 | 设置内容 |
| --- | --- |
| 允许所有群组，但仅在 @提及 时回复 | `groups: { "*": { requireMention: true } }` |
| 禁用所有群组回复 | `groupPolicy: "disabled"` |
| 仅限特定群组 | `groups: { "<group-id>": { ... } }` (无 `"*"` 键) |
| 仅您可以在群组中触发 | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

## 会话密钥

-   群组会话使用 `agent:::group:` 会话密钥（房间/频道使用 `agent:::channel:`）。
-   Telegram 论坛主题会在群组 ID 后添加 `:topic:`，因此每个主题都有自己的会话。
-   私聊使用主会话（或按发送者配置的会话）。
-   群组会话会跳过心跳检测。

## 模式：个人私信 + 公共群组（单代理）

是的——如果您的“个人”流量是 **私信**，而“公共”流量是 **群组**，这很有效。原因：在单代理模式下，私信通常进入 **主** 会话密钥 (`agent:main:main`)，而群组始终使用 **非主** 会话密钥 (`agent:main::group:`)。如果您启用沙箱并设置 `mode: "non-main"`，这些群组会话将在 Docker 中运行，而您的主私信会话则保留在主机上。这为您提供了一个代理“大脑”（共享工作空间 + 内存），但两种执行姿态：

-   **私信**：完整工具（主机）
-   **群组**：沙箱 + 受限工具（Docker）

> 如果您需要真正独立的工作空间/角色（“个人”和“公共”绝不能混合），请使用第二个代理 + 绑定。参见 [多代理路由](../concepts/multi-agent.md)。

示例（私信在主机，群组沙箱化 + 仅消息工具）：

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // 群组/频道是非主会话 -> 沙箱化
        scope: "session", // 最强隔离（每个群组/频道一个容器）
        workspaceAccess: "none",
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        // 如果 allow 非空，则其他所有内容都被阻止（deny 仍然优先）。
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"],
      },
    },
  },
}
```

想要“群组只能看到文件夹 X”而不是“无主机访问”？保持 `workspaceAccess: "none"` 并仅将允许列表中的路径挂载到沙箱：

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
            // hostPath:containerPath:mode
            "/home/user/FriendsShared:/data:ro",
          ],
        },
      },
    },
  },
}
```

相关：

-   配置键和默认值：[网关配置](../gateway/configuration.md#agentsdefaultssandbox)
-   调试工具被阻止的原因：[沙箱 vs 工具策略 vs 提升权限](../gateway/sandbox-vs-tool-policy-vs-elevated.md)
-   绑定挂载详情：[沙箱化](../gateway/sandboxing.md#custom-bind-mounts)

## 显示标签

-   UI 标签在可用时使用 `displayName`，格式为 `:`。
-   `#room` 保留给房间/频道；群聊使用 `g-`（小写，空格 -> `-`，保留 `#@+._-`）。

## 群组策略

控制每个频道的群组/房间消息处理方式：

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789"], // 数字 Telegram 用户 ID（向导可以解析 @username）
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

| 策略 | 行为 |
| --- | --- |
| `"open"` | 群组绕过允许列表；提及门控仍然适用。 |
| `"disabled"` | 完全阻止所有群组消息。 |
| `"allowlist"` | 仅允许匹配配置的允许列表的群组/房间。 |

注意：

-   `groupPolicy` 与提及门控（需要 @提及）是分开的。
-   WhatsApp/Telegram/Signal/iMessage/Microsoft Teams/Zalo：使用 `groupAllowFrom`（回退：显式 `allowFrom`）。
-   私信配对批准（`*-allowFrom` 存储条目）仅适用于私信访问；群组发送者授权仍明确依赖于群组允许列表。
-   Discord：允许列表使用 `channels.discord.guilds..channels`。
-   Slack：允许列表使用 `channels.slack.channels`。
-   Matrix：允许列表使用 `channels.matrix.groups`（房间 ID、别名或名称）。使用 `channels.matrix.groupAllowFrom` 来限制发送者；也支持每个房间的 `users` 允许列表。
-   群组私信是单独控制的 (`channels.discord.dm.*`, `channels.slack.dm.*`)。
-   Telegram 允许列表可以匹配用户 ID (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) 或用户名 (`"@alice"` 或 `"alice"`)；前缀不区分大小写。
-   默认是 `groupPolicy: "allowlist"`；如果您的群组允许列表为空，群组消息将被阻止。
-   运行时安全：当提供者块完全缺失时（`channels.` 不存在），群组策略会回退到故障关闭模式（通常是 `allowlist`），而不是继承 `channels.defaults.groupPolicy`。

快速心智模型（群组消息的评估顺序）：

1.  `groupPolicy` (open/disabled/allowlist)
2.  群组允许列表 (`*.groups`, `*.groupAllowFrom`, 频道特定的允许列表)
3.  提及门控 (`requireMention`, `/activation`)

## 提及门控（默认）

群组消息需要提及，除非按群组覆盖。默认值位于每个子系统的 `*.groups."*"` 下。回复机器人消息算作隐式提及（当频道支持回复元数据时）。这适用于 Telegram、WhatsApp、Slack、Discord 和 Microsoft Teams。

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

注意：

-   `mentionPatterns` 是不区分大小写的正则表达式。
-   提供显式提及的平台仍然通过；模式是回退方案。
-   每个代理的覆盖：`agents.list[].groupChat.mentionPatterns`（当多个代理共享一个群组时有用）。
-   仅当提及检测可能时（配置了原生提及或 `mentionPatterns`）才强制执行提及门控。
-   Discord 默认值位于 `channels.discord.guilds."*"` 中（可按公会/频道覆盖）。
-   群组历史上下文在各个频道间统一包装，并且是 **仅待处理** 的（因提及门控而跳过的消息）；使用 `messages.groupChat.historyLimit` 作为全局默认值，使用 `channels..historyLimit`（或 `channels..accounts.*.historyLimit`）进行覆盖。设置为 `0` 以禁用。

## 群组/频道工具限制（可选）

某些频道配置支持限制 **特定群组/房间/频道内** 可用的工具。

-   `tools`：允许/拒绝整个群组的工具。
-   `toolsBySender`：群组内按发送者的覆盖。使用显式键前缀：`id:`, `e164:`, `username:`, `name:`，以及 `"*"` 通配符。仍接受旧的无前缀键，并仅匹配为 `id:`。

解析顺序（最具体的优先）：

1.  群组/频道 `toolsBySender` 匹配
2.  群组/频道 `tools`
3.  默认 (`"*"`) `toolsBySender` 匹配
4.  默认 (`"*"`) `tools`

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

注意：

-   群组/频道工具限制是在全局/代理工具策略之外额外应用的（deny 仍然优先）。
-   某些频道对房间/频道使用不同的嵌套（例如，Discord `guilds.*.channels.*`、Slack `channels.*`、MS Teams `teams.*.channels.*`）。

## 群组允许列表

当配置了 `channels.whatsapp.groups`、`channels.telegram.groups` 或 `channels.imessage.groups` 时，键充当群组允许列表。使用 `"*"` 允许所有群组，同时仍设置默认提及行为。常见意图（复制/粘贴）：

1.  禁用所有群组回复

```json
{
  channels: { whatsapp: { groupPolicy: "disabled" } },
}
```

2.  仅允许特定群组（WhatsApp）

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

3.  允许所有群组但需要提及（显式）

```json
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

4.  仅所有者可以在群组中触发（WhatsApp）

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

## 激活（仅限所有者）

群组所有者可以切换每个群组的激活状态：

-   `/activation mention`
-   `/activation always`

所有者由 `channels.whatsapp.allowFrom`（或未设置时的机器人自身 E.164）确定。将命令作为独立消息发送。其他平台目前忽略 `/activation`。

## 上下文字段

群组入站载荷设置：

-   `ChatType=group`
-   `GroupSubject`（如果已知）
-   `GroupMembers`（如果已知）
-   `WasMentioned`（提及门控结果）
-   Telegram 论坛主题还包括 `MessageThreadId` 和 `IsForum`。

代理系统提示在新群组会话的第一个回合中包含群组介绍。它提醒模型要像人类一样回应，避免使用 Markdown 表格，并避免输入字面的 `\n` 序列。

## iMessage 特定说明

-   路由或允许列表时，优先使用 `chat_id:`。
-   列出聊天：`imsg chats --limit 20`。
-   群组回复总是返回到相同的 `chat_id`。

## WhatsApp 特定说明

有关 WhatsApp 特有行为（历史记录注入、提及处理详情），请参阅 [群组消息](./group-messages.md)。

[群组消息](./group-messages.md)[广播群组](./broadcast-groups.md)

---