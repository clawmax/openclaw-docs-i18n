

  配置

  
# 频道路由

OpenClaw 将回复**路由回消息来源的频道**。模型不会选择频道；路由是确定性的，由主机配置控制。

## 关键术语

-   **频道**: `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`。
-   **AccountId**: 每个频道的账户实例（当支持时）。
-   可选的频道默认账户: `channels..defaultAccount` 用于在出站路径未指定 `accountId` 时选择使用哪个账户。
    -   在多账户设置中，当配置了两个或更多账户时，请设置明确的默认值（`defaultAccount` 或 `accounts.default`）。如果没有设置，回退路由可能会选择第一个规范化的账户 ID。
-   **AgentId**: 一个独立的工作区 + 会话存储（“大脑”）。
-   **SessionKey**: 用于存储上下文和控制并发性的桶键。

## 会话密钥结构（示例）

直接消息合并到代理的**主**会话：

-   `agent::` (默认: `agent:main:main`)

群组和频道在每个频道中保持独立：

-   群组: `agent:::group:`
-   频道/房间: `agent:::channel:`

线程：

-   Slack/Discord 线程在基础键后追加 `:thread:`。
-   Telegram 论坛主题将 `:topic:` 嵌入到群组键中。

示例：

-   `agent:main:telegram:group:-1001234567890:topic:42`
-   `agent:main:discord:channel:123456:thread:987654`

## 主私信路由固定

当 `session.dmScope` 为 `main` 时，直接消息可能共享一个主会话。为了防止非所有者的私信覆盖会话的 `lastRoute`，OpenClaw 在满足以下所有条件时，会从 `allowFrom` 推断出一个固定的所有者：

-   `allowFrom` 恰好有一个非通配符条目。
-   该条目可以规范化为该频道的一个具体发送者 ID。
-   入站私信发送者与该固定所有者不匹配。

在不匹配的情况下，OpenClaw 仍会记录入站会话元数据，但会跳过更新主会话的 `lastRoute`。

## 路由规则（如何选择代理）

路由为每条入站消息选择**一个代理**：

1.  **精确对等体匹配**（`bindings` 中带有 `peer.kind` + `peer.id`）。
2.  **父对等体匹配**（线程继承）。
3.  **服务器 + 角色匹配**（Discord）通过 `guildId` + `roles`。
4.  **服务器匹配**（Discord）通过 `guildId`。
5.  **团队匹配**（Slack）通过 `teamId`。
6.  **账户匹配**（频道上的 `accountId`）。
7.  **频道匹配**（该频道上的任何账户，`accountId: "*"`）。
8.  **默认代理**（`agents.list[].default`，否则列表第一个条目，回退到 `main`）。

当一个绑定包含多个匹配字段（`peer`、`guildId`、`teamId`、`roles`）时，**所有提供的字段都必须匹配**，该绑定才会生效。匹配到的代理决定了使用哪个工作区和会话存储。

## 广播群组（运行多个代理）

广播群组允许您为同一个对等体运行**多个代理**，**当 OpenClaw 通常会回复时**（例如：在 WhatsApp 群组中，经过提及/激活门控之后）。配置：

```json
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"],
  },
}
```

参见：[广播群组](./broadcast-groups.md)。

## 配置概览

-   `agents.list`: 命名的代理定义（工作区、模型等）。
-   `bindings`: 将入站频道/账户/对等体映射到代理。

示例：

```json
{
  agents: {
    list: [{ id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }],
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" },
  ],
}
```

## 会话存储

会话存储位于状态目录下（默认 `~/.openclaw`）：

-   `~/.openclaw/agents//sessions/sessions.json`
-   JSONL 对话记录与存储文件位于同一目录

您可以通过 `session.store` 和 `{agentId}` 模板来覆盖存储路径。

## WebChat 行为

WebChat 附加到**选定的代理**，并默认使用该代理的主会话。因此，WebChat 允许您在一个地方查看该代理的跨频道上下文。

## 回复上下文

入站回复包含：

-   可用的 `ReplyToId`、`ReplyToBody` 和 `ReplyToSender`。
-   引用的上下文作为 `[Replying to ...]` 块附加到 `Body`。

这在所有频道中都是一致的。

[广播群组](./broadcast-groups.md)[频道位置解析](./location.md)

---