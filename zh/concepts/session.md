

  会话与记忆

  
# 会话管理

OpenClaw 将**每个代理的一个直接聊天会话**视为主会话。直接聊天会合并到 `agent::`（默认为 `main`），而群组/频道聊天则拥有自己的键。`session.mainKey` 会被遵循。使用 `session.dmScope` 来控制**私信**的分组方式：

-   `main`（默认）：所有私信共享主会话以实现连续性。
-   `per-peer`：按发送者 ID 跨频道隔离。
-   `per-channel-peer`：按频道 + 发送者隔离（推荐用于多用户收件箱）。
-   `per-account-channel-peer`：按账户 + 频道 + 发送者隔离（推荐用于多账户收件箱）。使用 `session.identityLinks` 将带有提供商前缀的对等方 ID 映射到一个规范身份，这样当使用 `per-peer`、`per-channel-peer` 或 `per-account-channel-peer` 时，同一个人可以跨频道共享一个私信会话。

## 安全私信模式（推荐用于多用户设置）

> **安全警告：** 如果你的代理可以接收来自**多个用户**的私信，你应强烈考虑启用安全私信模式。若不启用，所有用户将共享相同的对话上下文，这可能导致用户间的私人信息泄露。

**默认设置下可能出现的问题示例：**

-   爱丽丝（`<SENDER_A>`）向你的代理发送关于私人话题的消息（例如，医疗预约）
-   鲍勃（`<SENDER_B>`）向你的代理发送消息询问“我们之前聊了什么？”
-   由于两个私信共享同一个会话，模型可能会使用爱丽丝之前的上下文来回答鲍勃。

**解决方案：** 设置 `dmScope` 以按用户隔离会话：

```
// ~/.openclaw/openclaw.json
{
  session: {
    // 安全私信模式：按频道 + 发送者隔离私信上下文。
    dmScope: "per-channel-peer",
  },
}
```

**何时启用此模式：**

-   你为多个发送者设置了配对批准
-   你使用了包含多个条目的私信允许列表
-   你设置了 `dmPolicy: "open"`
-   多个电话号码或账户可以向你的代理发送消息

注意事项：

-   默认值为 `dmScope: "main"` 以实现连续性（所有私信共享主会话）。这对于单用户设置是可以的。
-   本地 CLI 引导程序在未设置时默认写入 `session.dmScope: "per-channel-peer"`（已有的显式值会被保留）。
-   对于同一频道的多账户收件箱，建议使用 `per-account-channel-peer`。
-   如果同一个人通过多个频道联系你，请使用 `session.identityLinks` 将他们的私信会话合并到一个规范身份下。
-   你可以使用 `openclaw security audit` 验证你的私信设置（参见[安全](../cli/security.md)）。

## 网关是唯一可信源

所有会话状态都**由网关**（“主”OpenClaw）拥有。UI 客户端（macOS 应用、WebChat 等）必须向网关查询会话列表和令牌计数，而不是读取本地文件。

-   在**远程模式**下，你关心的会话存储位于远程网关主机上，而不是你的 Mac 上。
-   UI 中显示的令牌计数来自网关的存储字段（`inputTokens`、`outputTokens`、`totalTokens`、`contextTokens`）。客户端不会解析 JSONL 记录来“修正”总数。

## 状态存储位置

-   在**网关主机**上：
    -   存储文件：`~/.openclaw/agents//sessions/sessions.json`（按代理）。
-   记录文件：`~/.openclaw/agents//sessions/.jsonl`（Telegram 话题会话使用 `.../-topic-.jsonl`）。
-   存储是一个映射 `sessionKey -> { sessionId, updatedAt, ... }`。删除条目是安全的；它们会在需要时重新创建。
-   群组条目可能包含 `displayName`、`channel`、`subject`、`room` 和 `space`，以便在 UI 中标记会话。
-   会话条目包含 `origin` 元数据（标签 + 路由提示），以便 UI 可以解释会话的来源。
-   OpenClaw **不**读取旧的 Pi/Tau 会话文件夹。

## 维护

OpenClaw 应用会话存储维护，以保持 `sessions.json` 和记录文件随时间推移保持在可控范围内。

### 默认值

-   `session.maintenance.mode`: `warn`
-   `session.maintenance.pruneAfter`: `30d`
-   `session.maintenance.maxEntries`: `500`
-   `session.maintenance.rotateBytes`: `10mb`
-   `session.maintenance.resetArchiveRetention`: 默认为 `pruneAfter` (`30d`)
-   `session.maintenance.maxDiskBytes`: 未设置（禁用）
-   `session.maintenance.highWaterBytes`: 启用预算时默认为 `maxDiskBytes` 的 `80%`

### 工作原理

维护在会话存储写入期间运行，你也可以使用 `openclaw sessions cleanup` 按需触发。

-   `mode: "warn"`：报告哪些内容将被驱逐，但不修改条目/记录文件。
-   `mode: "enforce"`：按以下顺序应用清理：
    1.  清理早于 `pruneAfter` 的陈旧条目
    2.  将条目数量限制在 `maxEntries` 以内（最旧的优先）
    3.  为已移除且不再被引用的条目归档记录文件
    4.  根据保留策略清理旧的 `*.deleted.` 和 `*.reset.` 归档文件
    5.  当 `sessions.json` 超过 `rotateBytes` 时进行轮换
    6.  如果设置了 `maxDiskBytes`，则强制执行磁盘预算，目标为 `highWaterBytes`（最旧的工件优先，然后是最旧的会话）

### 大型存储的性能注意事项

大型会话存储在高流量设置中很常见。维护工作是写入路径上的工作，因此非常大的存储可能会增加写入延迟。最增加成本的因素：

-   非常高的 `session.maintenance.maxEntries` 值
-   很长的 `pruneAfter` 窗口，导致陈旧条目长期保留
-   `~/.openclaw/agents//sessions/` 目录中有大量记录/归档工件
-   启用磁盘预算（`maxDiskBytes`）但没有合理的清理/上限限制

应对措施：

-   在生产环境中使用 `mode: "enforce"`，以便自动限制增长
-   同时设置时间和数量限制（`pruneAfter` + `maxEntries`），而不是只设置一个
-   在大型部署中设置 `maxDiskBytes` + `highWaterBytes` 作为硬性上限
-   将 `highWaterBytes` 保持在显著低于 `maxDiskBytes` 的水平（默认为 80%）
-   配置更改后运行 `openclaw sessions cleanup --dry-run --json`，以在执行前验证预期影响
-   对于频繁活动的会话，在运行手动清理时传递 `--active-key`

### 自定义示例

使用保守的强制执行策略：

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "45d",
      maxEntries: 800,
      rotateBytes: "20mb",
      resetArchiveRetention: "14d",
    },
  },
}
```

为会话目录启用硬磁盘预算：

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      maxDiskBytes: "1gb",
      highWaterBytes: "800mb",
    },
  },
}
```

为大型安装调优（示例）：

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "14d",
      maxEntries: 2000,
      rotateBytes: "25mb",
      maxDiskBytes: "2gb",
      highWaterBytes: "1.6gb",
    },
  },
}
```

从 CLI 预览或强制执行维护：

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

## 会话清理

OpenClaw 默认在 LLM 调用前从内存上下文中修剪**旧的工具结果**。这**不会**重写 JSONL 历史记录。参见 [/concepts/session-pruning](./session-pruning.md)。

## 预压缩内存刷新

当会话接近自动压缩时，OpenClaw 可以运行一个**静默内存刷新**轮次，提醒模型将持久性笔记写入磁盘。这仅在工作区可写时运行。参见[记忆](./memory.md)和[压缩](./compaction.md)。

## 传输 → 会话键映射

-   直接聊天遵循 `session.dmScope`（默认为 `main`）。
    -   `main`: `agent::`（跨设备/频道的连续性）。
        -   多个电话号码和频道可以映射到同一个代理主键；它们充当进入一个对话的传输通道。
    -   `per-peer`: `agent::dm:`。
    -   `per-channel-peer`: `agent:::dm:`。
    -   `per-account-channel-peer`: `agent::::dm:`（accountId 默认为 `default`）。
    -   如果 `session.identityLinks` 匹配到一个带有提供商前缀的对等方 ID（例如 `telegram:123`），则规范键会替换 ``，以便同一个人跨频道共享一个会话。
-   群组聊天隔离状态：`agent:::group:`（房间/频道使用 `agent:::channel:`）。
    -   Telegram 论坛话题会在群组 ID 后附加 `:topic:` 以实现隔离。
    -   为迁移目的，仍识别旧的 `group:` 键。
-   入站上下文可能仍使用 `group:`；频道从 `Provider` 推断并规范化为 `agent:::group:` 形式。
-   其他来源：
    -   定时任务：`cron:<job.id>`
    -   Webhooks：`hook:`（除非由 hook 显式设置）
    -   节点运行：`node-`

## 生命周期

-   重置策略：会话会被重用直到过期，过期时间在收到下一条入站消息时评估。
-   每日重置：默认为**网关主机本地时间凌晨 4:00**。一旦会话的最后更新时间早于最近一次每日重置时间，该会话即被视为陈旧。
-   空闲重置（可选）：`idleMinutes` 添加一个滑动空闲窗口。当同时配置了每日重置和空闲重置时，**先到期的那个**会强制创建新会话。
-   旧版仅空闲模式：如果你设置了 `session.idleMinutes` 而没有任何 `session.reset`/`resetByType` 配置，OpenClaw 会保持仅空闲模式以实现向后兼容性。
-   按类型覆盖（可选）：`resetByType` 允许你覆盖 `direct`、`group` 和 `thread` 会话的策略（thread = Slack/Discord 线程、Telegram 话题、连接器提供的 Matrix 线程）。
-   按频道覆盖（可选）：`resetByChannel` 覆盖某个频道的重置策略（适用于该频道的所有会话类型，优先级高于 `reset`/`resetByType`）。
-   重置触发器：精确的 `/new` 或 `/reset`（以及 `resetTriggers` 中的任何额外内容）会启动一个新的会话 ID，并将消息的其余部分传递过去。`/new ` 接受模型别名、`provider/model` 或提供商名称（模糊匹配）来设置新会话的模型。如果单独发送 `/new` 或 `/reset`，OpenClaw 会运行一个简短的“问候”轮次以确认重置。
-   手动重置：从存储中删除特定键或移除 JSONL 记录文件；下一条消息会重新创建它们。
-   隔离的定时任务每次运行总是生成一个新的 `sessionId`（不进行空闲重用）。

## 发送策略（可选）

阻止特定会话类型的消息传递，而无需列出单个 ID。

```json
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } },
        // 匹配原始会话键（包括 `agent:<id>:` 前缀）。
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ],
      default: "allow",
    },
  },
}
```

运行时覆盖（仅限所有者）：

-   `/send on` → 允许此会话发送
-   `/send off` → 拒绝此会话发送
-   `/send inherit` → 清除覆盖并使用配置规则
    请将这些作为独立消息发送，以便它们被注册。

## 配置（可选重命名示例）

```
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender", // 保持群组键分离
    dmScope: "main", // 私信连续性（对于共享收件箱，设置为 per-channel-peer/per-account-channel-peer）
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      // 默认值：mode=daily, atHour=4（网关主机本地时间）。
      // 如果同时设置了 idleMinutes，则先到期的那个生效。
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  },
}
```

## 检查

-   `openclaw status` — 显示存储路径和最近会话。
-   `openclaw sessions --json` — 转储每个条目（使用 `--active ` 过滤）。
-   `openclaw gateway call sessions.list --params '{}'` — 从正在运行的网关获取会话（使用 `--url`/`--token` 访问远程网关）。
-   在聊天中发送 `/status` 作为独立消息，以查看代理是否可达、会话上下文使用了多少、当前思考/详细模式切换状态，以及你的 WhatsApp Web 凭据上次刷新时间（有助于发现重新链接需求）。
-   发送 `/context list` 或 `/context detail` 以查看系统提示和注入的工作区文件中的内容（以及最大的上下文贡献者）。
-   发送 `/stop`（或独立的终止短语，如 `stop`、`stop action`、`stop run`、`stop openclaw`）以中止当前运行，清除该会话的排队后续操作，并停止由其产生的任何子代理运行（回复中包含已停止的数量）。
-   发送 `/compact`（可选指令）作为独立消息，以总结较旧的上下文并释放窗口空间。参见 [/concepts/compaction](./compaction.md)。
-   可以直接打开 JSONL 记录文件以查看完整轮次。

## 提示

-   保持主键专用于 1:1 流量；让群组保持自己的键。
-   在自动化清理时，删除单个键而不是整个存储，以保留其他地方的上下文。

## 会话来源元数据

每个会话条目都会在 `origin` 中记录其来源（尽力而为）：

-   `label`：人工标签（从对话标签 + 群组主题/频道解析而来）
-   `provider`：规范化的频道 ID（包括扩展）
-   `from`/`to`：来自入站信封的原始路由 ID
-   `accountId`：提供商账户 ID（多账户时）
-   `threadId`：频道支持时的线程/话题 ID
    来源字段会为私信、频道和群组填充。如果连接器仅更新传递路由（例如，以保持私信主会话新鲜），它仍应提供入站上下文，以便会话保留其解释性元数据。扩展可以通过在入站上下文中发送 `ConversationLabel`、`GroupSubject`、`GroupChannel`、`GroupSpace` 和 `SenderName`，并调用 `recordSessionMetaFromInbound`（或将相同的上下文传递给 `updateLastRoute`）来实现这一点。

[引导启动](../start/bootstrapping.md)[会话清理](./session-pruning.md)