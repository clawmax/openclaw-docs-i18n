

  技能

  
# 斜杠命令

命令由网关处理。大多数命令必须作为以 `/` 开头的**独立**消息发送。仅限主机的 bash 聊天命令使用 `! `（`/bash ` 是其别名）。有两个相关系统：

-   **命令**：独立的 `/...` 消息。
-   **指令**：`/think`、`/verbose`、`/reasoning`、`/elevated`、`/exec`、`/model`、`/queue`。
    -   指令在模型看到消息之前会被剥离。
    -   在普通聊天消息（非纯指令消息）中，它们被视为“内联提示”，并**不会**持久化会话设置。
    -   在纯指令消息（消息仅包含指令）中，它们会持久化到会话并回复确认信息。
    -   指令仅对**授权发送者**生效。如果设置了 `commands.allowFrom`，则它是唯一使用的允许列表；否则授权来自频道允许列表/配对加上 `commands.useAccessGroups`。未授权的发送者看到的指令会被视为纯文本。

还有一些**内联快捷命令**（仅限允许列表/授权发送者）：`/help`、`/commands`、`/status`、`/whoami`（`/id`）。它们会立即执行，在模型看到消息前被剥离，剩余文本继续通过正常流程处理。

## 配置

```json
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

-   `commands.text`（默认 `true`）启用解析聊天消息中的 `/...`。
    -   在不支持原生命令的平台上（WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams），即使将此设置为 `false`，文本命令仍然有效。
-   `commands.native`（默认 `"auto"`）注册原生命令。
    -   自动：在 Discord/Telegram 上启用；在 Slack 上禁用（直到你添加快捷命令）；对于不支持原生的提供商则忽略。
    -   设置 `channels.discord.commands.native`、`channels.telegram.commands.native` 或 `channels.slack.commands.native` 以按提供商覆盖（布尔值或 `"auto"`）。
    -   `false` 会在启动时清除 Discord/Telegram 上先前注册的命令。Slack 命令在 Slack 应用中管理，不会自动移除。
-   `commands.nativeSkills`（默认 `"auto"`）在支持时原生注册**技能**命令。
    -   自动：在 Discord/Telegram 上启用；在 Slack 上禁用（Slack 要求为每个技能创建斜杠命令）。
    -   设置 `channels.discord.commands.nativeSkills`、`channels.telegram.commands.nativeSkills` 或 `channels.slack.commands.nativeSkills` 以按提供商覆盖（布尔值或 `"auto"`）。
-   `commands.bash`（默认 `false`）启用 `! ` 来运行主机 shell 命令（`/bash ` 是别名；需要 `tools.elevated` 允许列表）。
-   `commands.bashForegroundMs`（默认 `2000`）控制 bash 在切换到后台模式前等待的时间（`0` 表示立即后台运行）。
-   `commands.config`（默认 `false`）启用 `/config`（读取/写入 `openclaw.json`）。
-   `commands.debug`（默认 `false`）启用 `/debug`（仅运行时覆盖）。
-   `commands.allowFrom`（可选）为命令授权设置按提供商的允许列表。配置后，它是命令和指令的唯一授权来源（频道允许列表/配对和 `commands.useAccessGroups` 被忽略）。使用 `"*"` 作为全局默认值；特定于提供商的键会覆盖它。
-   `commands.useAccessGroups`（默认 `true`）在未设置 `commands.allowFrom` 时，强制执行命令的允许列表/策略。

## 命令列表

文本 + 原生（启用时）：

-   `/help`
-   `/commands`
-   `/skill  [input]`（按名称运行技能）
-   `/status`（显示当前状态；包括当前模型提供商的提供商使用情况/配额，如果可用）
-   `/allowlist`（列出/添加/删除允许列表条目）
-   `/approve  allow-once|allow-always|deny`（解决 exec 批准提示）
-   `/context [list|detail|json]`（解释“上下文”；`detail` 显示每个文件 + 每个工具 + 每个技能 + 系统提示的大小）
-   `/export-session [path]`（别名：`/export`）（将当前会话导出为包含完整系统提示的 HTML）
-   `/whoami`（显示你的发送者 ID；别名：`/id`）
-   `/session idle <duration|off>`（管理聚焦线程绑定的非活动自动取消聚焦）
-   `/session max-age <duration|off>`（管理聚焦线程绑定的硬性最大期限自动取消聚焦）
-   `/subagents list|kill|log|info|send|steer|spawn`（检查、控制或为当前会话生成子代理运行）
-   `/acp spawn|cancel|steer|close|status|set-mode|set|cwd|permissions|timeout|model|reset-options|doctor|install|sessions`（检查和控制 ACP 运行时会话）
-   `/agents`（列出此会话的线程绑定代理）
-   `/focus `（Discord：将此线程或新线程绑定到会话/子代理目标）
-   `/unfocus`（Discord：移除当前线程绑定）
-   `/kill <id|#|all>`（立即中止此会话的一个或所有正在运行的子代理；无确认消息）
-   `/steer <id|#> `（立即引导正在运行的子代理：尽可能在运行中引导，否则中止当前工作并根据引导消息重新启动）
-   `/tell <id|#> `（`/steer` 的别名）
-   `/config show|get|set|unset`（将配置持久化到磁盘，仅限所有者；需要 `commands.config: true`）
-   `/debug show|set|unset|reset`（运行时覆盖，仅限所有者；需要 `commands.debug: true`）
-   `/usage off|tokens|full|cost`（每次回复的使用情况页脚或本地成本摘要）
-   `/tts off|always|inbound|tagged|status|provider|limit|summary|audio`（控制 TTS；参见 [/tts](../tts.md)）
    -   Discord：原生命令是 `/voice`（Discord 保留了 `/tts`）；文本 `/tts` 仍然有效。
-   `/stop`
-   `/restart`
-   `/dock-telegram`（别名：`/dock_telegram`）（将回复切换到 Telegram）
-   `/dock-discord`（别名：`/dock_discord`）（将回复切换到 Discord）
-   `/dock-slack`（别名：`/dock_slack`）（将回复切换到 Slack）
-   `/activation mention|always`（仅限群组）
-   `/send on|off|inherit`（仅限所有者）
-   `/reset` 或 `/new [model]`（可选的模型提示；其余部分会传递过去）
-   `/think <off|minimal|low|medium|high|xhigh>`（由模型/提供商动态选择；别名：`/thinking`、`/t`）
-   `/verbose on|full|off`（别名：`/v`）
-   `/reasoning on|off|stream`（别名：`/reason`；启用时，发送单独的消息，前缀为 `Reasoning:`；`stream` = 仅 Telegram 草稿）
-   `/elevated on|off|ask|full`（别名：`/elev`；`full` 跳过 exec 批准）
-   `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=`（发送 `/exec` 以显示当前设置）
-   `/model `（别名：`/models`；或来自 `agents.defaults.models.*.alias` 的 `/`）
-   `/queue `（加上选项如 `debounce:2s cap:25 drop:summarize`；发送 `/queue` 查看当前设置）
-   `/bash `（仅限主机；`! ` 的别名；需要 `commands.bash: true` + `tools.elevated` 允许列表）

仅文本：

-   `/compact [instructions]`（参见 [/concepts/compaction](../concepts/compaction.md)）
-   `! `（仅限主机；一次一个；使用 `!poll` + `!stop` 处理长时间运行的任务）
-   `!poll`（检查输出 / 状态；接受可选的 `sessionId`；`/bash poll` 也有效）
-   `!stop`（停止正在运行的 bash 任务；接受可选的 `sessionId`；`/bash stop` 也有效）

注意：

-   命令在命令和参数之间接受可选的 `:`（例如 `/think: high`、`/send: on`、`/help:`）。
-   `/new ` 接受模型别名、`provider/model` 或提供商名称（模糊匹配）；如果没有匹配项，文本将被视为消息正文。
-   要获取完整的提供商使用情况细分，请使用 `openclaw status --usage`。
-   `/allowlist add|remove` 需要 `commands.config=true` 并遵循频道 `configWrites`。
-   `/usage` 控制每次回复的使用情况页脚；`/usage cost` 从 OpenClaw 会话日志中打印本地成本摘要。
-   `/restart` 默认启用；设置 `commands.restart: false` 以禁用它。
-   仅限 Discord 的原生命令：`/vc join|leave|status` 控制语音频道（需要 `channels.discord.voice` 和原生命令；不作为文本提供）。
-   Discord 线程绑定命令（`/focus`、`/unfocus`、`/agents`、`/session idle`、`/session max-age`）要求有效的线程绑定已启用（`session.threadBindings.enabled` 和/或 `channels.discord.threadBindings.enabled`）。
-   ACP 命令参考和运行时行为：[ACP 代理](./acp-agents.md)。
-   `/verbose` 用于调试和额外可见性；在正常使用中保持其**关闭**。
-   工具失败摘要仍会在相关时显示，但详细的失败文本仅在 `/verbose` 为 `on` 或 `full` 时包含。
-   `/reasoning`（和 `/verbose`）在群组设置中存在风险：它们可能暴露你本不打算公开的内部推理或工具输出。最好保持关闭，尤其是在群聊中。
-   **快速路径**：来自允许列表发送者的纯命令消息会立即处理（绕过队列 + 模型）。
-   **群组提及门控**：来自允许列表发送者的纯命令消息绕过提及要求。
-   **内联快捷命令（仅限允许列表发送者）**：某些命令在嵌入到普通消息中时也有效，并在模型看到剩余文本之前被剥离。
    -   示例：`hey /status` 触发状态回复，剩余文本继续通过正常流程处理。
-   目前：`/help`、`/commands`、`/status`、`/whoami`（`/id`）。
-   未授权的纯命令消息会被静默忽略，内联的 `/...` 标记被视为纯文本。
-   **技能命令**：`user-invocable` 技能会作为斜杠命令公开。名称被清理为 `a-z0-9_`（最多 32 个字符）；冲突会获得数字后缀（例如 `_2`）。
    -   `/skill  [input]` 按名称运行技能（当原生命令限制阻止每个技能命令时很有用）。
    -   默认情况下，技能命令会作为普通请求转发给模型。
    -   技能可以选择声明 `command-dispatch: tool` 以将命令直接路由到工具（确定性的，无需模型）。
    -   示例：`/prose`（OpenProse 插件）—— 参见 [OpenProse](/prose）。
-   **原生命令参数**：Discord 使用自动补全来提供动态选项（当你省略必需参数时，会显示按钮菜单）。Telegram 和 Slack 在命令支持选择且你省略参数时显示按钮菜单。

## 使用情况展示面（何处显示什么）

-   **提供商使用情况/配额**（例如：“Claude 剩余 80%”）在启用使用情况跟踪时，会显示在 `/status` 中，针对当前模型提供商。
-   **每次回复的令牌数/成本** 由 `/usage off|tokens|full` 控制（附加到正常回复后）。
-   `/model status` 是关于**模型/认证/端点**的，而非使用情况。

## 模型选择 (/model)

`/model` 作为指令实现。示例：

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

注意：

-   `/model` 和 `/model list` 显示一个紧凑的、带编号的选择器（模型系列 + 可用提供商）。
-   在 Discord 上，`/model` 和 `/models` 打开一个交互式选择器，包含提供商和模型下拉列表以及一个提交步骤。
-   `/model <#>` 从该选择器中选择（并尽可能优先选择当前提供商）。
-   `/model status` 显示详细视图，包括配置的提供商端点（`baseUrl`）和 API 模式（`api`），如果可用。

## 调试覆盖

`/debug` 允许你设置**仅运行时**的配置覆盖（内存中，非磁盘）。仅限所有者。默认禁用；通过 `commands.debug: true` 启用。示例：

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

注意：

-   覆盖会立即应用于新的配置读取，但**不会**写入 `openclaw.json`。
-   使用 `/debug reset` 清除所有覆盖并返回到磁盘上的配置。

## 配置更新

`/config` 写入你的磁盘配置（`openclaw.json`）。仅限所有者。默认禁用；通过 `commands.config: true` 启用。示例：

```bash
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

注意：

-   配置在写入前会进行验证；无效的更改会被拒绝。
-   `/config` 更新会在重启后持久化。

## 平台注意事项

-   **文本命令** 在正常的聊天会话中运行（私信共享 `main`，群组有自己的会话）。
-   **原生命令** 使用独立的会话：
    -   Discord：`agent::discord:slash:`
    -   Slack：`agent::slack:slash:`（前缀可通过 `channels.slack.slashCommand.sessionPrefix` 配置）
    -   Telegram：`telegram:slash:`（通过 `CommandTargetSessionKey` 定位聊天会话）
-   **`/stop`** 针对活动的聊天会话，以便它可以中止当前运行。
-   **Slack：** `channels.slack.slashCommand` 仍然支持单个 `/openclaw` 风格的命令。如果你启用 `commands.native`，则必须为每个内置命令创建一个 Slack 斜杠命令（名称与 `/help` 等相同）。Slack 的命令参数菜单以临时 Block Kit 按钮形式交付。
    -   Slack 原生例外：注册 `/agentstatus`（而非 `/status`），因为 Slack 保留了 `/status`。文本 `/status` 在 Slack 消息中仍然有效。

[创建技能](./creating-skills.md)[技能](./skills.md)