

  多智能体

  
# 多智能体路由

目标：在一个运行的 Gateway 中，实现多个*隔离*的智能体（独立的工作空间 + `agentDir` + 会话），以及多个频道账户（例如两个 WhatsApp）。入站消息通过绑定路由到智能体。

## 什么是“一个智能体”？

一个**智能体**是一个拥有完整作用域的“大脑”，包含其自身的：

-   **工作空间**（文件、AGENTS.md/SOUL.md/USER.md、本地笔记、角色规则）。
-   **状态目录**（`agentDir`），用于存储认证配置文件、模型注册表和每个智能体的配置。
-   **会话存储**（聊天历史 + 路由状态），位于 `~/.openclaw/agents//sessions`。

认证配置文件是**每个智能体独立**的。每个智能体从其自身的路径读取：

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

主智能体的凭证**不会**自动共享。切勿在智能体之间复用 `agentDir`（这会导致认证/会话冲突）。如果你想共享凭证，请将 `auth-profiles.json` 复制到其他智能体的 `agentDir` 中。技能通过每个工作空间的 `skills/` 文件夹实现每个智能体独立，同时共享技能可从 `~/.openclaw/skills` 获取。参见[技能：每个智能体独立 vs 共享](../tools/skills.md#per-agent-vs-shared-skills)。Gateway 可以托管**一个智能体**（默认）或**多个智能体**并行运行。**工作空间注意：** 每个智能体的工作空间是**默认的当前工作目录**，并非一个硬性的沙箱。相对路径在工作空间内解析，但绝对路径可以访问主机上的其他位置，除非启用了沙箱。参见[沙箱化](../gateway/sandboxing.md)。

## 路径（快速映射）

-   配置：`~/.openclaw/openclaw.json`（或 `OPENCLAW_CONFIG_PATH`）
-   状态目录：`~/.openclaw`（或 `OPENCLAW_STATE_DIR`）
-   工作空间：`~/.openclaw/workspace`（或 `~/.openclaw/workspace-`）
-   智能体目录：`~/.openclaw/agents//agent`（或 `agents.list[].agentDir`）
-   会话：`~/.openclaw/agents//sessions`

### 单智能体模式（默认）

如果你不做任何配置，OpenClaw 将运行单个智能体：

-   `agentId` 默认为 **`main`**。
-   会话的键为 `agent:main:`。
-   工作空间默认为 `~/.openclaw/workspace`（或当设置了 `OPENCLAW_PROFILE` 时为 `~/.openclaw/workspace-`）。
-   状态默认为 `~/.openclaw/agents/main/agent`。

## 智能体助手

使用智能体向导添加一个新的隔离智能体：

```bash
openclaw agents add work
```

然后添加 `bindings`（或让向导完成）以路由入站消息。使用以下命令验证：

```bash
openclaw agents list --bindings
```

## 快速开始

### 步骤 1：创建每个智能体的工作空间

使用向导或手动创建工作空间：

```bash
openclaw agents add coding
openclaw agents add social
```

每个智能体获得其自己的工作空间，包含 `SOUL.md`、`AGENTS.md` 和可选的 `USER.md`，以及一个专用的 `agentDir` 和位于 `~/.openclaw/agents/` 下的会话存储。

### 步骤 2：创建频道账户

在你偏好的频道上为每个智能体创建一个账户：

-   Discord：每个智能体一个机器人，启用消息内容意图，复制每个令牌。
-   Telegram：通过 BotFather 为每个智能体创建一个机器人，复制每个令牌。
-   WhatsApp：为每个账户链接一个电话号码。

```bash
openclaw channels login --channel whatsapp --account work
```

参见频道指南：[Discord](../channels/discord.md)、[Telegram](../channels/telegram.md)、[WhatsApp](../channels/whatsapp.md)。

### 步骤 3：添加智能体、账户和绑定

在 `agents.list` 下添加智能体，在 `channels..accounts` 下添加频道账户，并使用 `bindings` 连接它们（示例如下）。

### 步骤 4：重启并验证

```bash
openclaw gateway restart
openclaw agents list --bindings
openclaw channels status --probe
```

## 多个智能体 = 多个人，多种个性

使用**多个智能体**，每个 `agentId` 成为一个**完全隔离的角色**：

-   **不同的电话号码/账户**（每个频道 `accountId`）。
-   **不同的个性**（每个智能体的工作空间文件，如 `AGENTS.md` 和 `SOUL.md`）。
-   **独立的认证 + 会话**（除非显式启用，否则不会交叉通信）。

这使得**多个人**可以共享一个 Gateway 服务器，同时保持他们的 AI“大脑”和数据隔离。

## 一个 WhatsApp 号码，多个人（私聊分流）

你可以将**不同的 WhatsApp 私聊**路由到不同的智能体，同时保持在**一个 WhatsApp 账户**上。通过发送者的 E.164 号码（如 `+15551234567`）和 `peer.kind: "direct"` 进行匹配。回复仍然来自同一个 WhatsApp 号码（没有每个智能体的发送者身份）。重要细节：私聊会合并到智能体的**主会话键**，因此真正的隔离需要**每个人一个智能体**。示例：

```json
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" },
    ],
  },
  bindings: [
    {
      agentId: "alex",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230001" } },
    },
    {
      agentId: "mia",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230002" } },
    },
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"],
    },
  },
}
```

注意：

-   私聊访问控制是**每个 WhatsApp 账户全局的**（配对/允许列表），而不是每个智能体。
-   对于共享群组，将群组绑定到一个智能体或使用[广播群组](../channels/broadcast-groups.md)。

## 路由规则（消息如何选择智能体）

绑定是**确定性的**且**最具体的优先**：

1.  `peer` 匹配（精确的私聊/群组/频道 ID）
2.  `parentPeer` 匹配（线程继承）
3.  `guildId + roles`（Discord 角色路由）
4.  `guildId`（Discord）
5.  `teamId`（Slack）
6.  `accountId` 匹配（针对某个频道）
7.  频道级别的匹配（`accountId: "*"`）
8.  回退到默认智能体（`agents.list[].default`，否则列表中的第一个条目，默认：`main`）

如果同一层级有多个绑定匹配，配置顺序中的第一个胜出。如果一个绑定设置了多个匹配字段（例如 `peer` + `guildId`），则所有指定的字段都是必需的（`AND` 语义）。重要的账户作用域细节：

-   省略 `accountId` 的绑定仅匹配默认账户。
-   使用 `accountId: "*"` 作为跨所有账户的频道范围回退。
-   如果你稍后为同一个智能体添加具有显式账户 ID 的相同绑定，OpenClaw 会将现有的仅频道绑定升级为账户作用域，而不是复制它。

## 多个账户 / 电话号码

支持**多个账户**的频道（例如 WhatsApp）使用 `accountId` 来标识每个登录。每个 `accountId` 可以路由到不同的智能体，因此一个服务器可以托管多个电话号码而不会混淆会话。如果你想在省略 `accountId` 时设置一个频道范围的默认账户，请设置 `channels..defaultAccount`（可选）。如果未设置，OpenClaw 会回退到 `default`（如果存在），否则是第一个配置的账户 ID（已排序）。支持此模式的常见频道包括：

-   `whatsapp`、`telegram`、`discord`、`slack`、`signal`、`imessage`
-   `irc`、`line`、`googlechat`、`mattermost`、`matrix`、`nextcloud-talk`
-   `bluebubbles`、`zalo`、`zalouser`、`nostr`、`feishu`

## 概念

-   `agentId`：一个“大脑”（工作空间、每个智能体的认证、每个智能体的会话存储）。
-   `accountId`：一个频道账户实例（例如 WhatsApp 账户 `"personal"` 与 `"biz"`）。
-   `binding`：通过 `(channel, accountId, peer)` 以及可选的公会/团队 ID 将入站消息路由到 `agentId`。
-   私聊合并到 `agent::`（每个智能体的“主”会话；`session.mainKey`）。

## 平台示例

### 每个智能体的 Discord 机器人

每个 Discord 机器人账户映射到一个唯一的 `accountId`。将每个账户绑定到一个智能体，并为每个机器人维护允许列表。

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "coding", workspace: "~/.openclaw/workspace-coding" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "discord", accountId: "default" } },
    { agentId: "coding", match: { channel: "discord", accountId: "coding" } },
  ],
  channels: {
    discord: {
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN_MAIN",
          guilds: {
            "123456789012345678": {
              channels: {
                "222222222222222222": { allow: true, requireMention: false },
              },
            },
          },
        },
        coding: {
          token: "DISCORD_BOT_TOKEN_CODING",
          guilds: {
            "123456789012345678": {
              channels: {
                "333333333333333333": { allow: true, requireMention: false },
              },
            },
          },
        },
      },
    },
  },
}
```

注意：

-   邀请每个机器人加入公会并启用消息内容意图。
-   令牌位于 `channels.discord.accounts..token`（默认账户可以使用 `DISCORD_BOT_TOKEN`）。

### 每个智能体的 Telegram 机器人

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "alerts", workspace: "~/.openclaw/workspace-alerts" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "alerts", match: { channel: "telegram", accountId: "alerts" } },
  ],
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "123456:ABC...",
          dmPolicy: "pairing",
        },
        alerts: {
          botToken: "987654:XYZ...",
          dmPolicy: "allowlist",
          allowFrom: ["tg:123456789"],
        },
      },
    },
  },
}
```

注意：

-   使用 BotFather 为每个智能体创建一个机器人并复制每个令牌。
-   令牌位于 `channels.telegram.accounts..botToken`（默认账户可以使用 `TELEGRAM_BOT_TOKEN`）。

### 每个智能体的 WhatsApp 号码

在启动网关之前链接每个账户：

```bash
openclaw channels login --channel whatsapp --account personal
openclaw channels login --channel whatsapp --account biz
```

`~/.openclaw/openclaw.json` (JSON5)：

```json
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // 确定性路由：第一个匹配项胜出（最具体的在前）。
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // 可选的每个对等体覆盖（示例：将特定群组发送到工作智能体）。
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // 默认关闭：智能体到智能体的消息传递必须显式启用 + 加入允许列表。
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // 可选覆盖。默认：~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // 可选覆盖。默认：~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

## 示例：WhatsApp 日常聊天 + Telegram 深度工作

按频道分流：将 WhatsApp 路由到快速的日常智能体，将 Telegram 路由到 Opus 智能体。

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

注意：

-   如果你有一个频道的多个账户，请在绑定中添加 `accountId`（例如 `{ channel: "whatsapp", accountId: "personal" }`）。
-   要将单个私聊/群组路由到 Opus 同时保持其他消息在聊天智能体上，请为该对等体添加一个 `match.peer` 绑定；对等体匹配总是优先于频道范围的规则。

## 示例：同一频道，一个对等体路由到 Opus

将 WhatsApp 保持在快速智能体上，但将一个私聊路由到 Opus：

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    {
      agentId: "opus",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551234567" } },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
  ],
}
```

对等体绑定总是胜出，因此将它们放在频道范围规则之上。

## 绑定到 WhatsApp 群组的家庭智能体

将一个专用的家庭智能体绑定到单个 WhatsApp 群组，并启用提及门控和更严格的工具策略：

```json
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"],
        },
        sandbox: {
          mode: "all",
          scope: "agent",
        },
        tools: {
          allow: [
            "exec",
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"],
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" },
      },
    },
  ],
}
```

注意：

-   工具允许/拒绝列表是**工具**，不是技能。如果一个技能需要运行二进制文件，请确保 `exec` 被允许并且沙箱中存在该二进制文件。
-   为了更严格的门控，设置 `agents.list[].groupChat.mentionPatterns` 并为频道保持群组允许列表启用。

## 每个智能体的沙箱和工具配置

从 v2026.1.6 开始，每个智能体可以拥有自己的沙箱和工具限制：

```json
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // 个人智能体不使用沙箱
        },
        // 无工具限制 - 所有工具可用
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // 始终沙箱化
          scope: "agent",  // 每个智能体一个容器
          docker: {
            // 容器创建后可选的一次性设置命令
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // 仅允许读取工具
          deny: ["exec", "write", "edit", "apply_patch"],    // 拒绝其他工具
        },
      },
    ],
  },
}
```

注意：`setupCommand` 位于 `sandbox.docker` 下，并在容器创建时运行一次。当解析的作用域为 `"shared"` 时，每个智能体的 `sandbox.docker.*` 覆盖将被忽略。**好处：**

-   **安全隔离**：限制不受信任智能体的工具
-   **资源控制**：沙箱化特定智能体，同时让其他智能体在主机上运行
-   **灵活的策略**：每个智能体不同的权限

注意：`tools.elevated` 是**全局的**并且基于发送者；它不能按智能体配置。如果你需要每个智能体的边界，请使用 `agents.list[].tools` 来拒绝 `exec`。对于群组目标，使用 `agents.list[].groupChat.mentionPatterns`，以便 @提及清晰地映射到预期的智能体。参见[多智能体沙箱与工具](../tools/multi-agent-sandbox-tools.md)获取详细示例。

[压缩](./compaction.md)[在线状态](./presence.md)