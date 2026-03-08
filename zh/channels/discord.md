

  消息平台

  
# Discord

状态：已准备就绪，可通过官方 Discord 网关进行私信和公会频道通信。

## 快速设置

您需要创建一个带有机器人的新应用程序，将机器人添加到您的服务器，并将其与 OpenClaw 配对。我们建议将您的机器人添加到您自己的私人服务器。如果您还没有私人服务器，请[先创建一个](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server)（选择 **创建我自己的 > 为我和我的朋友**）。

### 步骤 1：创建 Discord 应用程序和机器人

前往 [Discord 开发者门户](https://discord.com/developers/applications) 并点击 **新建应用程序**。将其命名为类似“OpenClaw”的名称。点击侧边栏的 **机器人**。将 **用户名** 设置为您为 OpenClaw 代理起的任何名称。

### 步骤 2：启用特权意图

仍在 **机器人** 页面，向下滚动到 **特权网关意图** 并启用：

-   **消息内容意图**（必需）
-   **服务器成员意图**（推荐；角色白名单和名称到 ID 匹配所需）
-   **在线状态意图**（可选；仅在线状态更新时需要）

### 步骤 3：复制您的机器人令牌

在 **机器人** 页面上向上滚动并点击 **重置令牌**。

> **ℹ️** 尽管名称如此，这会生成您的第一个令牌——没有任何东西被“重置”。

复制令牌并保存在某处。这是您的 **机器人令牌**，您很快会需要它。

### 步骤 4：生成邀请 URL 并将机器人添加到您的服务器

点击侧边栏的 **OAuth2**。您将生成一个具有正确权限的邀请 URL，以便将机器人添加到您的服务器。向下滚动到 **OAuth2 URL 生成器** 并启用：

-   `bot`
-   `applications.commands`

下方将出现 **机器人权限** 部分。启用：

-   查看频道
-   发送消息
-   读取消息历史记录
-   嵌入链接
-   附加文件
-   添加反应（可选）

复制底部生成的 URL，将其粘贴到浏览器中，选择您的服务器，然后点击 **继续** 进行连接。您现在应该能在 Discord 服务器中看到您的机器人。

### 步骤 5：启用开发者模式并收集您的 ID

回到 Discord 应用程序，您需要启用开发者模式以便复制内部 ID。

1.  点击 **用户设置**（头像旁边的齿轮图标）→ **高级** → 打开 **开发者模式**
2.  右键点击侧边栏的 **服务器图标** → **复制服务器 ID**
3.  右键点击 **您自己的头像** → **复制用户 ID**

将您的 **服务器 ID** 和 **用户 ID** 与机器人令牌一起保存——您将在下一步中将这三者发送给 OpenClaw。

### 步骤 6：允许来自服务器成员的私信

为了使配对工作，Discord 需要允许您的机器人向您发送私信。右键点击 **服务器图标** → **隐私设置** → 打开 **私信**。这允许服务器成员（包括机器人）向您发送私信。如果您想使用 Discord 私信与 OpenClaw 交互，请保持此设置启用。如果您只计划使用公会频道，可以在配对后禁用私信。

### 步骤 7：步骤 0：安全设置您的机器人令牌（请勿在聊天中发送）

您的 Discord 机器人令牌是机密（类似于密码）。在运行 OpenClaw 的机器上设置它，然后再向您的代理发送消息。

```bash
openclaw config set channels.discord.token '"YOUR_BOT_TOKEN"' --json
openclaw config set channels.discord.enabled true --json
openclaw gateway
```

如果 OpenClaw 已作为后台服务运行，请改用 `openclaw gateway restart`。

### 步骤 8：配置 OpenClaw 并进行配对

在任何现有频道（例如 Telegram）上与您的 OpenClaw 代理聊天并告诉它。如果 Discord 是您的第一个频道，请改用 CLI / 配置选项卡。

> “我已在配置中设置了 Discord 机器人令牌。请使用用户 ID `<user_id>` 和服务器 ID `<server_id>` 完成 Discord 设置。”

```json
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

### 步骤 9：批准首次私信配对

等待网关运行，然后在 Discord 中向您的机器人发送私信。它将回复一个配对码。

将配对码发送到您现有频道上的代理：

> “批准此 Discord 配对码：``”

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

配对码在 1 小时后过期。您现在应该能够通过私信在 Discord 中与您的代理聊天了。

 

> **ℹ️** 令牌解析是账户感知的。配置令牌值优先于环境变量回退。`DISCORD_BOT_TOKEN` 仅用于默认账户。

## 推荐：设置公会工作区

私信工作后，您可以将 Discord 服务器设置为完整的工作区，其中每个频道都获得自己的代理会话及其自己的上下文。这推荐用于只有您和您的机器人的私人服务器。

### 步骤 1：将您的服务器添加到公会白名单

这使您的代理能够在您服务器上的任何频道中响应，而不仅仅是私信。

> “将我的 Discord 服务器 ID `<server_id>` 添加到公会白名单”

```json
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: true,
          users: ["YOUR_USER_ID"],
        },
      },
    },
  },
}
```

### 步骤 2：允许无需 @提及 的响应

默认情况下，您的代理仅在公会频道中被 @提及 时才会响应。对于私人服务器，您可能希望它对每条消息都做出响应。

> “允许我的代理在此服务器上响应而无需被 @提及”

```json
{
  channels: {
    discord: {
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: false,
        },
      },
    },
  },
}
```

### 步骤 3：为公会频道规划内存

默认情况下，长期记忆（MEMORY.md）仅在私信会话中加载。公会频道不会自动加载 MEMORY.md。

> “当我在 Discord 频道中提问时，如果需要来自 MEMORY.md 的长期上下文，请使用 memory\_search 或 memory\_get。”

如果您需要在每个频道中共享上下文，请将稳定的指令放在 `AGENTS.md` 或 `USER.md` 中（它们会注入到每个会话中）。将长期笔记保存在 `MEMORY.md` 中，并根据需要使用内存工具访问它们。

 现在在您的 Discord 服务器上创建一些频道并开始聊天。您的代理可以看到频道名称，并且每个频道都有自己独立的会话——因此您可以设置 `#coding`、`#home`、`#research` 或任何适合您工作流程的频道。

## 运行时模型

-   网关拥有 Discord 连接。
-   回复路由是确定性的：Discord 入站消息回复回 Discord。
-   默认情况下（`session.dmScope=main`），直接聊天共享代理主会话（`agent:main:main`）。
-   公会频道是独立的会话键（`agent::discord:channel:`）。
-   默认忽略群组私信（`channels.discord.dm.groupEnabled=false`）。
-   原生斜杠命令在独立的命令会话中运行（`agent::discord:slash:`），同时仍将 `CommandTargetSessionKey` 携带到路由的对话会话。

## 论坛频道

Discord 论坛和媒体频道仅接受帖子回复。OpenClaw 支持两种创建方式：

-   向论坛父级（`channel:`）发送消息以自动创建帖子。帖子标题使用您消息的第一行非空行。
-   使用 `openclaw message thread create` 直接创建帖子。对于论坛频道，不要传递 `--message-id`。

示例：发送到论坛父级以创建帖子

```bash
openclaw message send --channel discord --target channel:<forumId> \
  --message "Topic title\nBody of the post"
```

示例：显式创建论坛帖子

```bash
openclaw message thread create --channel discord --target channel:<forumId> \
  --thread-name "Topic title" --message "Body of the post"
```

论坛父级不接受 Discord 组件。如果您需要组件，请发送到帖子本身（`channel:`）。

## 交互式组件

OpenClaw 支持 Discord 组件 v2 容器用于代理消息。使用带有 `components` 负载的消息工具。交互结果作为正常的入站消息路由回代理，并遵循现有的 Discord `replyToMode` 设置。支持的块：

-   `text`、`section`、`separator`、`actions`、`media-gallery`、`file`
-   操作行最多允许 5 个按钮或单个选择菜单
-   选择类型：`string`、`user`、`role`、`mentionable`、`channel`

默认情况下，组件是单次使用的。设置 `components.reusable=true` 以允许按钮、选择菜单和表单在过期前多次使用。要限制谁可以点击按钮，请在该按钮上设置 `allowedUsers`（Discord 用户 ID、标签或 `*`）。配置后，不匹配的用户会收到一个临时拒绝消息。`/model` 和 `/models` 斜杠命令会打开一个交互式模型选择器，包含提供商和模型下拉列表以及一个提交步骤。选择器回复是临时的，只有调用用户可以使用它。文件附件：

-   `file` 块必须指向附件引用（`attachment://`）
-   通过 `media`/`path`/`filePath` 提供附件（单个文件）；使用 `media-gallery` 处理多个文件
-   使用 `filename` 在需要匹配附件引用时覆盖上传名称

模态表单：

-   添加 `components.modal`，最多包含 5 个字段
-   字段类型：`text`、`checkbox`、`radio`、`select`、`role-select`、`user-select`
-   OpenClaw 会自动添加一个触发按钮

示例：

```json
{
  channel: "discord",
  action: "send",
  to: "channel:123456789012345678",
  message: "Optional fallback text",
  components: {
    reusable: true,
    text: "Choose a path",
    blocks: [
      {
        type: "actions",
        buttons: [
          {
            label: "Approve",
            style: "success",
            allowedUsers: ["123456789012345678"],
          },
          { label: "Decline", style: "danger" },
        ],
      },
      {
        type: "actions",
        select: {
          type: "string",
          placeholder: "Pick an option",
          options: [
            { label: "Option A", value: "a" },
            { label: "Option B", value: "b" },
          ],
        },
      },
    ],
    modal: {
      title: "Details",
      triggerLabel: "Open form",
      fields: [
        { type: "text", label: "Requester" },
        {
          type: "select",
          label: "Priority",
          options: [
            { label: "Low", value: "low" },
            { label: "High", value: "high" },
          ],
        },
      ],
    },
  },
}
```

## 访问控制和路由

`channels.discord.dmPolicy` 控制私信访问（旧版：`channels.discord.dm.policy`）：

-   `pairing`（默认）
-   `allowlist`
-   `open`（需要 `channels.discord.allowFrom` 包含 `"*"`；旧版：`channels.discord.dm.allowFrom`）
-   `disabled`

如果私信策略不是 open，未知用户将被阻止（或在 `pairing` 模式下提示配对）。多账户优先级：

-   `channels.discord.accounts.default.allowFrom` 仅适用于 `default` 账户。
-   命名账户在其自身的 `allowFrom` 未设置时继承 `channels.discord.allowFrom`。
-   命名账户不继承 `channels.discord.accounts.default.allowFrom`。

用于传递的私信目标格式：

-   `user:`
-   `<@id>` 提及

裸数字 ID 是模糊的，除非提供了明确的用户/频道目标类型，否则会被拒绝。

```json
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          ignoreOtherMentions: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

公会消息默认需要提及。提及检测包括：

-   显式的机器人提及
-   配置的提及模式（`agents.list[].groupChat.mentionPatterns`，回退 `messages.groupChat.mentionPatterns`）
-   在支持的案例中隐式的回复给机器人行为

`requireMention` 按公会/频道配置（`channels.discord.guilds...`）。`ignoreOtherMentions` 可选地丢弃提及了其他用户/角色但未提及机器人的消息（不包括 @everyone/@here）。群组私信：

-   默认：忽略（`dm.groupEnabled=false`）
-   可选的白名单通过 `dm.groupChannels`（频道 ID 或别名）

### 基于角色的代理路由

使用 `bindings[].match.roles` 通过角色 ID 将 Discord 公会成员路由到不同的代理。基于角色的绑定仅接受角色 ID，并在对等或父对等绑定之后、仅公会绑定之前进行评估。如果绑定还设置了其他匹配字段（例如 `peer` + `guildId` + `roles`），则所有配置的字段都必须匹配。

```json
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## 开发者门户设置

1.  Discord 开发者门户 -> **应用程序** -> **新建应用程序**
2.  **机器人** -> **添加机器人**
3.  复制机器人令牌

在 **机器人 -> 特权网关意图** 中，启用：

-   消息内容意图
-   服务器成员意图（推荐）

在线状态意图是可选的，仅当您希望接收在线状态更新时才需要。设置机器人状态（`setPresence`）不需要为成员启用在线状态更新。

OAuth URL 生成器：

-   范围：`bot`、`applications.commands`

典型的基线权限：

-   查看频道
-   发送消息
-   读取消息历史记录
-   嵌入链接
-   附加文件
-   添加反应（可选）

除非明确需要，否则避免使用 `Administrator`。

启用 Discord 开发者模式，然后复制：

-   服务器 ID
-   频道 ID
-   用户 ID

在 OpenClaw 配置中优先使用数字 ID 以进行可靠的审计和探测。

## 原生命令和命令认证

-   `commands.native` 默认为 `"auto"` 并为 Discord 启用。
-   每频道覆盖：`channels.discord.commands.native`。
-   `commands.native=false` 显式清除先前注册的 Discord 原生命令。
-   原生命令认证使用与正常消息处理相同的 Discord 白名单/策略。
-   命令可能仍然对未经授权的用户在 Discord UI 中可见；执行时仍会强制执行 OpenClaw 认证并返回“未授权”。

有关命令目录和行为，请参阅[斜杠命令](../tools/slash-commands.md)。默认斜杠命令设置：

-   `ephemeral: true`

## 功能详情

Discord 支持代理输出中的回复标签：

-   `[[reply_to_current]]`
-   `[[reply_to:]]`

由 `channels.discord.replyToMode` 控制：

-   `off`（默认）
-   `first`
-   `all`

注意：`off` 禁用隐式回复线程。显式的 `[[reply_to_*]]` 标签仍然有效。消息 ID 在上下文/历史记录中公开，以便代理可以定位特定消息。

OpenClaw 可以通过发送临时消息并在文本到达时编辑它来流式传输草稿回复。

-   `channels.discord.streaming` 控制预览流式传输（`off` | `partial` | `block` | `progress`，默认：`off`）。
-   为跨频道一致性接受 `progress`，并在 Discord 上映射到 `partial`。
-   `channels.discord.streamMode` 是旧版别名，会自动迁移。
-   `partial` 在令牌到达时编辑单个预览消息。
-   `block` 发出草稿大小的块（使用 `draftChunk` 调整大小和断点）。

示例：

```json
{
  channels: {
    discord: {
      streaming: "partial",
    },
  },
}
```

`block` 模式分块默认值（限制在 `channels.discord.textChunkLimit` 内）：

```json
{
  channels: {
    discord: {
      streaming: "block",
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph",
      },
    },
  },
}
```

预览流式传输仅为文本；媒体回复回退到正常传递。注意：预览流式传输与块流式传输是分开的。当为 Discord 显式启用块流式传输时，OpenClaw 会跳过预览流以避免双重流式传输。

公会历史记录上下文：

-   `channels.discord.historyLimit` 默认 `20`
-   回退：`messages.groupChat.historyLimit`
-   `0` 禁用

私信历史记录控制：

-   `channels.discord.dmHistoryLimit`
-   `channels.discord.dms["<user_id>"].historyLimit`

帖子行为：

-   Discord 帖子作为频道会话路由
-   父帖子元数据可用于父会话链接
-   帖子配置继承父频道配置，除非存在帖子特定条目

频道主题作为**不受信任的**上下文注入（不作为系统提示）。

Discord 可以将帖子绑定到会话目标，以便该帖子中的后续消息继续路由到同一会话（包括子代理会话）。命令：

-   `/focus ` 将当前/新帖子绑定到子代理/会话目标
-   `/unfocus` 移除当前帖子绑定
-   `/agents` 显示活动运行和绑定状态
-   `/session idle <duration|off>` 检查/更新已绑定帖子的不活动自动解绑
-   `/session max-age <duration|off>` 检查/更新已绑定帖子的硬性最大存在时间

配置：

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
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // 选择启用
      },
    },
  },
}
```

注意：

-   `session.threadBindings.*` 设置全局默认值。
-   `channels.discord.threadBindings.*` 覆盖 Discord 行为。
-   `spawnSubagentSessions` 必须为 true 才能为 `sessions_spawn({ thread: true })` 自动创建/绑定帖子。
-   `spawnAcpSessions` 必须为 true 才能为 ACP（`/acp spawn ... --thread ...` 或 `sessions_spawn({ runtime: "acp", thread: true })`）自动创建/绑定帖子。
-   如果帖子的账户绑定被禁用，则 `/focus` 和相关的帖子绑定操作不可用。

请参阅[子代理](../tools/subagents.md)、[ACP 代理](../tools/acp-agents.md)和[配置参考](../gateway/configuration-reference.md)。

对于稳定的“始终在线”ACP 工作区，配置针对 Discord 对话的顶级类型化 ACP 绑定。配置路径：

-   `bindings[]` 包含 `type: "acp"` 和 `match.channel: "discord"`

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
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

注意：

-   帖子消息可以继承父频道 ACP 绑定。
-   在绑定的频道或帖子中，`/new` 和 `/reset` 会原地重置同一 ACP 会话。
-   临时帖子绑定仍然有效，并且可以在活动时覆盖目标解析。

有关绑定行为的详细信息，请参阅[ACP 代理](../tools/acp-agents.md)。

每公会反应通知模式：

-   `off`
-   `own`（默认）
-   `all`
-   `allowlist`（使用 `guilds..users`）

反应事件被转换为系统事件并附加到路由的 Discord 会话。

`ackReaction` 在 OpenClaw 处理入站消息时发送一个确认表情符号。解析顺序：

-   `channels.discord.accounts..ackReaction`
-   `channels.discord.ackReaction`
-   `messages.ackReaction`
-   代理身份表情符号回退（`agents.list[].identity.emoji`，否则为 ”👀”）

注意：

-   Discord 接受 Unicode 表情符号或自定义表情符号名称。
-   使用 `""` 为频道或账户禁用该反应。

频道发起的配置写入默认启用。这会影响 `/config set|unset` 流程（当命令功能启用时）。禁用：

```json
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

通过 `channels.discord.proxy` 通过 HTTP(S) 代理路由 Discord 网关 WebSocket 流量和启动 REST 查找（应用程序 ID + 白名单解析）。

```json
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

每账户覆盖：

```json
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

启用 PluralKit 解析以将代理消息映射到系统成员身份：

```json
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // 可选；私有系统需要
      },
    },
  },
}
```

注意：

-   白名单可以使用 `pk:`
-   仅当 `channels.discord.dangerouslyAllowNameMatching: true` 时，才按名称/别名匹配成员显示名称
-   查找使用原始消息 ID 并受时间窗口限制
-   如果查找失败，代理消息将被视为机器人消息并丢弃，除非 `allowBots=true`

当您设置状态或活动字段时，或者当您启用自动在线状态时，会应用在线状态更新。仅状态示例：

```json
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

活动示例（自定义状态是默认的活动类型）：

```json
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

流式传输示例：

```json
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

活动类型映射：

-   0: 正在玩
-   1: 正在直播（需要 `activityUrl`）
-   2: 正在听
-   3: 正在看
-   4: 自定义（使用活动文本作为状态；表情符号可选）
-   5: 正在竞赛

自动在线状态示例（运行时健康信号）：

```json
{
  channels: {
    discord: {
      autoPresence: {
        enabled: true,
        intervalMs: 30000,
        minUpdateIntervalMs: 15000,
        exhaustedText: "token exhausted",
      },
    },
  },
}
```

自动在线状态将运行时可用性映射到 Discord 状态：健康 => 在线，降级或未知 => 空闲，耗尽或不可用 => 请勿打扰。可选的文本覆盖：

-   `autoPresence.healthyText`
-   `autoPresence.degradedText`
-   `autoPresence.exhaustedText`（支持 `{reason}` 占位符）

Discord 支持基于按钮的执行批准私信，并且可以选择在原始频道中发布批准提示。配置路径：

-   `channels.discord.execApprovals.enabled`
-   `channels.discord.execApprovals.approvers`
-   `channels.discord.execApprovals.target`（`dm` | `channel` | `both`，默认：`dm`）
-   `agentFilter`、`sessionFilter`、`cleanupAfterResolve`

当 `target` 为 `channel` 或 `both` 时，批准提示在频道中可见。只有配置的批准者可以使用按钮；其他用户会收到临时拒绝消息。批准提示包含命令文本，因此仅在受信任的频道中启用频道传递。如果无法从会话键派生频道 ID，OpenClaw 将回退到私信传递。如果批准因未知批准 ID 而失败，请验证批准者列表和功能启用情况。相关文档：[执行批准](../tools/exec-approvals.md)

## 工具和操作门控

Discord 消息操作包括消息传递、频道管理、审核、在线状态和元数据操作。核心示例：

-   消息传递：`sendMessage`、`readMessages`、`editMessage`、`deleteMessage`、`threadReply`
-   反应：`react`、`reactions`、`emojiList`
-   审核：`timeout`、`kick`、`ban`
-   在线状态：`setPresence`

操作门控位于 `channels.discord.actions.*` 下。默认门控行为：

| 操作组 | 默认 |
| --- | --- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | 启用 |
| roles | 禁用 |
| moderation | 禁用 |
| presence | 禁用 |

## 组件 v2 UI

OpenClaw 使用 Discord 组件 v2 进行执行批准和跨上下文标记。Discord 消息操作也可以接受 `components` 用于自定义 UI（高级；需要 Carbon 组件实例），而旧版 `embeds` 仍然可用但不推荐。

-   `channels.discord.ui.components.accentColor` 设置 Discord 组件容器使用的强调色（十六进制）。
-   使用 `channels.discord.accounts..ui.components.accentColor` 按账户设置。
-   当存在组件 v2 时，`embeds` 被忽略。

示例：

```json
{
  channels: {
    discord: {
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
    },
  },
}
```

## 语音频道

OpenClaw 可以加入 Discord 语音频道进行实时、连续的对话。这与语音消息附件是分开的。要求：

-   启用原生命令（`commands.native` 或 `channels.discord.commands.native`）。
-   配置 `channels.discord.voice`。
-   机器人在目标语音频道中需要连接 + 说话权限。

使用仅限 Discord 的原生命令 `/vc join|leave|status` 来控制会话。该命令使用账户默认代理，并遵循与其他 Discord 命令相同的白名单和群组策略规则。自动加入示例：

```json
{
  channels: {
    discord: {
      voice: {
        enabled: true,
        autoJoin: [
          {
            guildId: "123456789012345678",
            channelId: "234567890123456789",
          },
        ],
        daveEncryption: true,
        decryptionFailureTolerance: 24,
        tts: {
          provider: "openai",
          openai: { voice: "alloy" },
        },
      },
    },
  },
}
```

注意：

-   `voice.tts` 仅覆盖语音播放的 `messages.tts`。
-   语音转录轮次从 Discord `allowFrom`（或 `dm.allowFrom`）派生所有者状态；非所有者说话者无法访问仅限所有者的工具（例如 `gateway` 和 `cron`）。
-   默认启用语音；设置 `channels.discord.voice.enabled=false` 以禁用它。
-   `voice.daveEncryption` 和 `voice.decryptionFailureTolerance` 传递给 `@discordjs/voice` 加入选项。
-   如果未设置，`@discordjs/voice` 默认值为 `daveEncryption=true` 和 `decryptionFailureTolerance=24`。
-   OpenClaw 还会监视接收解密失败，并在短时间内重复失败后通过离开/重新加入语音频道来自动恢复。
-   如果接收日志重复显示 `DecryptionFailed(UnencryptedWhenPassthroughDisabled)`，这可能是上游 `@discordjs/voice` 接收错误，在 [discord.js #11419](https://github.com/discordjs/discord.js/issues/11419) 中跟踪。

## 语音消息

Discord 语音消息显示波形预览，需要 OGG/Opus 音频和元数据。OpenClaw 会自动生成波形，但需要网关主机上可用的 `ffmpeg` 和 `ffprobe` 来检查和转换音频文件。要求和约束：

-   提供**本地文件路径**（URL 被拒绝）。
-   省略文本内容（Discord 不允许在同一负载中同时包含文本和语音消息）。
-   接受任何音频格式；需要时 OpenClaw 会转换为 OGG/Opus。

示例：

```
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## 故障排除

-   启用消息内容意图
-   当您依赖用户/成员解析时，启用服务器成员意图
-   更改意图后重启网关

-   验证 `groupPolicy`
-   验证 `channels.discord.guilds` 下的公会白名单
-   如果存在公会 `channels` 映射，则仅允许列出的频道
-   验证 `requireMention` 行为和提及模式

有用的检查：

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

常见原因：

-   `groupPolicy="allowlist"` 但没有匹配的公会/频道白名单
-   `requireMention` 配置在错误的位置（必须在 `channels.discord.guilds` 或频道条目下）
-   发送者被公会/频道 `users` 白名单阻止

典型日志：

-   `Listener DiscordMessageListener timed out after 30000ms for event MESSAGE_CREATE`
-   `Slow listener detected ...`
-   `discord inbound worker timed out after ...`

监听器预算旋钮：

-   single-account: `channels.discord.eventQueue.listenerTimeout`
-   multi-account: `channels.discord.accounts..eventQueue.listenerTimeout`

工作运行超时旋钮：

-   single-account: `channels.discord.inboundWorker.runTimeoutMs`
-   multi-account: `channels.discord.accounts..inboundWorker.runTimeoutMs`
-   default: `1800000`（30 分钟）；设为 `0` 可禁用

推荐基线：

```json
{
  channels: {
    discord: {
      accounts: {
        default: {
          eventQueue: {
            listenerTimeout: 120000,
          },
          inboundWorker: {
            runTimeoutMs: 1800000,
          },
        },
      },
    },
  },
}
```

使用 `eventQueue.listenerTimeout` 配置慢监听器；仅当需要为排队代理轮次设置单独安全阀时才使用 `inboundWorker.runTimeoutMs`。

`channels status --probe` 的权限检查仅对数字频道 ID 有效。若使用 slug 键，运行时匹配仍可工作，但 probe 无法完全验证权限。

-   DM 已禁用：`channels.discord.dm.enabled=false`
-   DM 策略已禁用：`channels.discord.dmPolicy="disabled"`（旧版：`channels.discord.dm.policy`）
-   在 `pairing` 模式下等待配对批准

默认会忽略机器人发出的消息。若设置 `channels.discord.allowBots=true`，请使用严格的提及和白名单规则以避免循环。建议使用 `channels.discord.allowBots="mentions"` 仅接受提及该机器人的机器人消息。

-   保持 OpenClaw 为最新（`openclaw update`），以便使用 Discord 语音接收恢复逻辑
-   确认 `channels.discord.voice.daveEncryption=true`（默认）
-   从 `channels.discord.voice.decryptionFailureTolerance=24`（上游默认）开始，仅在需要时调整
-   在日志中关注：
    -   `discord voice: DAVE decrypt failures detected`
    -   `discord voice: repeated decrypt failures; attempting rejoin`
-   若自动重新加入后仍失败，请收集日志并与 [discord.js #11419](https://github.com/discordjs/discord.js/issues/11419) 对照

## 配置参考要点

主要参考：

-   [配置参考 - Discord](../gateway/configuration-reference.md#discord)

高信号 Discord 字段：

-   startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
-   policy: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
-   command: `commands.native`, `commands.useAccessGroups`, `configWrites`, `slashCommand.*`
-   event queue: `eventQueue.listenerTimeout`（监听器预算）, `eventQueue.maxQueueSize`, `eventQueue.maxConcurrency`
-   inbound worker: `inboundWorker.runTimeoutMs`
-   reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
-   delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
-   streaming: `streaming`（旧别名：`streamMode`）, `draftChunk`, `blockStreaming`, `blockStreamingCoalesce`
-   media/retry: `mediaMaxMb`, `retry`
    -   `mediaMaxMb` 限制 Discord 出站上传（默认：`8MB`）
-   actions: `actions.*`
-   presence: `activity`, `status`, `activityType`, `activityUrl`
-   UI: `ui.components.accentColor`
-   features: `threadBindings`, 顶层 `bindings[]`（`type: "acp"`）, `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## 安全与运维

-   将机器人令牌视为机密（在受管环境中优先使用 `DISCORD_BOT_TOKEN`）。
-   授予最小权限的 Discord 权限。
-   若命令部署/状态过期，请重启网关并用 `openclaw channels status --probe` 重新检查。

## 相关链接

-   [配对](./pairing.md)
-   [频道路由](./channel-routing.md)
-   [多代理路由](../concepts/multi-agent.md)
-   [故障排除](./troubleshooting.md)
-   [斜杠命令](../tools/slash-commands.md)

[BlueBubbles](./bluebubbles.md)[Feishu](./feishu.md)