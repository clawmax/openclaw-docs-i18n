

  配置

  
# 群组消息

目标：让 Clawd 驻留在 WhatsApp 群组中，仅在收到 ping 时唤醒，并保持该线程与个人私聊会话分离。注意：`agents.list[].groupChat.mentionPatterns` 现在也被 Telegram/Discord/Slack/iMessage 使用；本文档重点介绍 WhatsApp 特定的行为。对于多代理设置，请为每个代理设置 `agents.list[].groupChat.mentionPatterns`（或使用 `messages.groupChat.mentionPatterns` 作为全局回退）。

## 已实现的功能 (2025-12-03)

-   激活模式：`mention`（默认）或 `always`。`mention` 需要 ping（通过 `mentionedJids` 的真实 WhatsApp @-提及、正则表达式模式或文本中任意位置的机器人 E.164 号码）。`always` 会在每条消息上唤醒代理，但仅当它能提供有意义的回复时才应回复；否则返回静默令牌 `NO_REPLY`。默认值可在配置中设置（`channels.whatsapp.groups`），并通过 `/activation` 按群组覆盖。当设置了 `channels.whatsapp.groups` 时，它也充当群组允许列表（包含 `"*"` 以允许所有群组）。
-   群组策略：`channels.whatsapp.groupPolicy` 控制是否接受群组消息（`open|disabled|allowlist`）。`allowlist` 使用 `channels.whatsapp.groupAllowFrom`（回退：显式的 `channels.whatsapp.allowFrom`）。默认为 `allowlist`（在添加发送者之前被阻止）。
-   按群组会话：会话键类似于 `agent::whatsapp:group:`，因此诸如 `/verbose on` 或 `/think high`（作为独立消息发送）等命令的作用域仅限于该群组；个人私聊状态不受影响。群组线程跳过心跳检测。
-   上下文注入：**仅待处理**的群组消息（默认 50 条）*未*触发运行的，会以 `[自您上次回复以来的聊天消息 - 供上下文参考]` 为前缀，触发行则以 `[当前消息 - 请回复此消息]` 为前缀。已在会话中的消息不会被重新注入。
-   发送者信息展示：每个群组批次现在都以 `[来自: 发送者姓名 (+E164)]` 结尾，以便 Pi 知道谁在发言。
-   阅后即焚/单次查看消息：我们在提取文本/提及之前会解包这些消息，因此其中的 ping 仍然可以触发。
-   群组系统提示：在群组会话的第一个回合（以及每当 `/activation` 更改模式时），我们会向系统提示中注入一个简短的说明，例如 `您正在 WhatsApp 群组 "<群组主题>" 中回复。群组成员：Alice (+44...), Bob (+43...), … 激活模式：仅触发 … 请回复消息上下文中注明的特定发送者。` 如果元数据不可用，我们仍会告知代理这是一个群组聊天。

## 配置示例 (WhatsApp)

在 `~/.openclaw/openclaw.json` 中添加一个 `groupChat` 块，以便即使 WhatsApp 在文本正文中去除了可见的 `@`，显示名称的 ping 也能正常工作：

```json
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: ["@?openclaw", "\\+?15555550123"],
        },
      },
    ],
  },
}
```

注意事项：

-   正则表达式不区分大小写；它们覆盖了像 `@openclaw` 这样的显示名称 ping 以及带或不带 `+`/空格的原始号码。
-   当有人点击联系人时，WhatsApp 仍会通过 `mentionedJids` 发送规范的提及，因此很少需要号码回退，但它是一个有用的安全网。

### 激活命令（仅所有者可用）

使用群聊命令：

-   `/activation mention`
-   `/activation always`

只有所有者号码（来自 `channels.whatsapp.allowFrom`，或未设置时来自机器人自身的 E.164）可以更改此设置。在群组中作为独立消息发送 `/status` 以查看当前的激活模式。

## 使用方法

1.  将您的 WhatsApp 账户（运行 OpenClaw 的那个）添加到群组中。
2.  发送 `@openclaw …`（或包含号码）。除非您设置 `groupPolicy: "open"`，否则只有允许列表中的发送者才能触发它。
3.  代理提示将包含最近的群组上下文以及尾随的 `[来自: …]` 标记，以便它可以回复正确的人。
4.  会话级别的指令（`/verbose on`、`/think high`、`/new` 或 `/reset`、`/compact`）仅适用于该群组的会话；将它们作为独立消息发送以便注册。您的个人私聊会话保持独立。

## 测试 / 验证

-   手动冒烟测试：
    -   在群组中发送一个 `@openclaw` ping，并确认回复中引用了发送者姓名。
    -   发送第二个 ping，并验证历史块是否被包含，然后在下一个回合被清除。
-   检查网关日志（使用 `--verbose` 运行）以查看 `inbound web message` 条目，其中显示 `from: ` 和 `[来自: …]` 后缀。

## 已知注意事项

-   为群组有意跳过了心跳检测，以避免产生噪音广播。
-   回声抑制使用组合的批次字符串；如果您发送两次没有提及的相同文本，只有第一次会得到回复。
-   会话存储条目将显示为 `agent::whatsapp:group:`，位于会话存储中（默认在 `~/.openclaw/agents//sessions/sessions.json`）；缺少条目仅意味着该群组尚未触发运行。
-   群组中的输入指示遵循 `agents.defaults.typingMode`（默认：未提及时为 `message`）。

[配对](./pairing.md)[群组](./groups.md)