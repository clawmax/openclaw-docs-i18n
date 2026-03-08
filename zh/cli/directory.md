

  CLI 命令

  
# directory

对支持此功能的频道进行目录查找（联系人/对等节点、群组和“我”）。

## 常用标志

-   `--channel `：频道 ID/别名（当配置了多个频道时为必需项；仅配置一个频道时自动选择）
-   `--account `：账户 ID（默认值：频道默认账户）
-   `--json`：输出 JSON 格式

## 说明

-   `directory` 命令旨在帮助您找到可以粘贴到其他命令（特别是 `openclaw message send --target ...`）中的 ID。
-   对于许多频道，结果基于配置（允许列表 / 已配置的群组），而非来自提供商的实时目录。
-   默认输出为 `id`（有时包含 `name`），以制表符分隔；脚本处理时请使用 `--json`。

## 将结果用于消息发送

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## ID 格式（按频道）

-   WhatsApp：`+15551234567`（私聊），`1234567890-1234567890@g.us`（群组）
-   Telegram：`@username` 或数字聊天 ID；群组为数字 ID
-   Slack：`user:U…` 和 `channel:C…`
-   Discord：`user:` 和 `channel:`
-   Matrix (插件)：`user:@user:server`，`room:!roomId:server`，或 `#alias:server`
-   Microsoft Teams (插件)：`user:` 和 `conversation:`
-   Zalo (插件)：用户 ID (Bot API)
-   Zalo Personal / `zalouser` (插件)：来自 `zca` (`me`, `friend list`, `group list`) 的线程 ID（私聊/群组）

## 自身（“我”）

```bash
openclaw directory self --channel zalouser
```

## 对等节点（联系人/用户）

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```

## 群组

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```

[devices](./devices.md)[dns](./dns.md)