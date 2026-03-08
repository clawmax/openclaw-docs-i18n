

  CLI 命令

  
# message

用于发送消息和频道操作（Discord/Google Chat/Slack/Mattermost (插件)/Telegram/WhatsApp/Signal/iMessage/MS Teams）的单一出站命令。

## 用法

```bash
openclaw message <子命令> [标志]
```

频道选择：

-   `--channel` 如果配置了多个频道则为必需。
-   如果只配置了一个频道，它将作为默认频道。
-   值：`whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`（Mattermost 需要插件）

目标格式（`--target`）：

-   WhatsApp：E.164 或群组 JID
-   Telegram：聊天 ID 或 `@用户名`
-   Discord：`channel:` 或 `user:`（或 `<@id>` 提及；原始数字 ID 被视为频道）
-   Google Chat：`spaces/` 或 `users/`
-   Slack：`channel:` 或 `user:`（接受原始频道 ID）
-   Mattermost（插件）：`channel:`、`user:` 或 `@用户名`（裸 ID 被视为频道）
-   Signal：`+E.164`、`group:`、`signal:+E.164`、`signal:group:` 或 `username:`/`u:`
-   iMessage：句柄、`chat_id:`、`chat_guid:` 或 `chat_identifier:`
-   MS Teams：会话 ID（`19:...@thread.tacv2`）或 `conversation:` 或 `user:<aad-object-id>`

名称查找：

-   对于支持的提供商（Discord/Slack 等），像 `Help` 或 `#help` 这样的频道名称会通过目录缓存解析。
-   缓存未命中时，如果提供商支持，OpenClaw 将尝试实时目录查找。

## 通用标志

-   `--channel <名称>`
-   `--account `
-   `--target <目标>`（用于发送/投票/阅读等的目标频道或用户）
-   `--targets <名称>`（可重复；仅用于广播）
-   `--json`
-   `--dry-run`
-   `--verbose`

## 操作

### 核心

-   `send`
    -   频道：WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (插件)/Signal/iMessage/MS Teams
    -   必需：`--target`，加上 `--message` 或 `--media`
    -   可选：`--media`、`--reply-to`、`--thread-id`、`--gif-playback`
    -   仅 Telegram：`--buttons`（需要 `channels.telegram.capabilities.inlineButtons` 允许）
    -   仅 Telegram：`--thread-id`（论坛主题 ID）
    -   仅 Slack：`--thread-id`（线程时间戳；`--reply-to` 使用相同字段）
    -   仅 WhatsApp：`--gif-playback`
-   `poll`
    -   频道：WhatsApp/Telegram/Discord/Matrix/MS Teams
    -   必需：`--target`、`--poll-question`、`--poll-option`（可重复）
    -   可选：`--poll-multi`
    -   仅 Discord：`--poll-duration-hours`、`--silent`、`--message`
    -   仅 Telegram：`--poll-duration-seconds` (5-600)、`--silent`、`--poll-anonymous` / `--poll-public`、`--thread-id`
-   `react`
    -   频道：Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
    -   必需：`--message-id`、`--target`
    -   可选：`--emoji`、`--remove`、`--participant`、`--from-me`、`--target-author`、`--target-author-uuid`
    -   注意：`--remove` 需要 `--emoji`（省略 `--emoji` 以清除自己的反应，在支持的地方；参见 /tools/reactions）
    -   仅 WhatsApp：`--participant`、`--from-me`
    -   Signal 群组反应：需要 `--target-author` 或 `--target-author-uuid`
-   `reactions`
    -   频道：Discord/Google Chat/Slack
    -   必需：`--message-id`、`--target`
    -   可选：`--limit`
-   `read`
    -   频道：Discord/Slack
    -   必需：`--target`
    -   可选：`--limit`、`--before`、`--after`
    -   仅 Discord：`--around`
-   `edit`
    -   频道：Discord/Slack
    -   必需：`--message-id`、`--message`、`--target`
-   `delete`
    -   频道：Discord/Slack/Telegram
    -   必需：`--message-id`、`--target`
-   `pin` / `unpin`
    -   频道：Discord/Slack
    -   必需：`--message-id`、`--target`
-   `pins`（列表）
    -   频道：Discord/Slack
    -   必需：`--target`
-   `permissions`
    -   频道：Discord
    -   必需：`--target`
-   `search`
    -   频道：Discord
    -   必需：`--guild-id`、`--query`
    -   可选：`--channel-id`、`--channel-ids`（可重复）、`--author-id`、`--author-ids`（可重复）、`--limit`

### 线程

-   `thread create`
    -   频道：Discord
    -   必需：`--thread-name`、`--target`（频道 ID）
    -   可选：`--message-id`、`--message`、`--auto-archive-min`
-   `thread list`
    -   频道：Discord
    -   必需：`--guild-id`
    -   可选：`--channel-id`、`--include-archived`、`--before`、`--limit`
-   `thread reply`
    -   频道：Discord
    -   必需：`--target`（线程 ID）、`--message`
    -   可选：`--media`、`--reply-to`

### 表情符号

-   `emoji list`
    -   Discord：`--guild-id`
    -   Slack：无需额外标志
-   `emoji upload`
    -   频道：Discord
    -   必需：`--guild-id`、`--emoji-name`、`--media`
    -   可选：`--role-ids`（可重复）

### 贴纸

-   `sticker send`
    -   频道：Discord
    -   必需：`--target`、`--sticker-id`（可重复）
    -   可选：`--message`
-   `sticker upload`
    -   频道：Discord
    -   必需：`--guild-id`、`--sticker-name`、`--sticker-desc`、`--sticker-tags`、`--media`

### 角色 / 频道 / 成员 / 语音

-   `role info` (Discord)：`--guild-id`
-   `role add` / `role remove` (Discord)：`--guild-id`、`--user-id`、`--role-id`
-   `channel info` (Discord)：`--target`
-   `channel list` (Discord)：`--guild-id`
-   `member info` (Discord/Slack)：`--user-id`（+ Discord 需要 `--guild-id`）
-   `voice status` (Discord)：`--guild-id`、`--user-id`

### 活动

-   `event list` (Discord)：`--guild-id`
-   `event create` (Discord)：`--guild-id`、`--event-name`、`--start-time`
    -   可选：`--end-time`、`--desc`、`--channel-id`、`--location`、`--event-type`

### 管理（Discord）

-   `timeout`：`--guild-id`、`--user-id`（可选 `--duration-min` 或 `--until`；两者都省略以清除超时）
-   `kick`：`--guild-id`、`--user-id`（+ `--reason`）
-   `ban`：`--guild-id`、`--user-id`（+ `--delete-days`、`--reason`）
    -   `timeout` 也支持 `--reason`

### 广播

-   `broadcast`
    -   频道：任何已配置的频道；使用 `--channel all` 以定位所有提供商
    -   必需：`--targets`（可重复）
    -   可选：`--message`、`--media`、`--dry-run`

## 示例

发送 Discord 回复：

```bash
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

发送带组件的 Discord 消息：

```bash
openclaw message send --channel discord \
  --target channel:123 --message "Choose:" \
  --components '{"text":"Choose a path","blocks":[{"type":"actions","buttons":[{"label":"Approve","style":"success"},{"label":"Decline","style":"danger"}]}]}'
```

完整模式请参见 [Discord 组件](../channels/discord.md#interactive-components)。创建 Discord 投票：

```bash
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

创建 Telegram 投票（2 分钟后自动关闭）：

```bash
openclaw message poll --channel telegram \
  --target @mychat \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-duration-seconds 120 --silent
```

发送 Teams 主动消息：

```bash
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

创建 Teams 投票：

```bash
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

在 Slack 中反应：

```bash
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

在 Signal 群组中反应：

```bash
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

发送 Telegram 内联按钮：

```bash
openclaw message send --channel telegram --target @mychat --message "Choose:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```

[memory](./memory.md)[models](./models.md)