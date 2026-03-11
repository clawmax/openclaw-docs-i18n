

  配置

  
# 频道故障排除（troubleshooting）

当你的频道已经成功连接，但响应行为不符合预期时，本页面将帮助你快速定位并解决问题。

## 诊断命令清单

遇到问题时，请先按顺序执行以下命令，建立诊断基线：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

健康的系统应该满足以下条件：

-   `Runtime: running` —— 运行时正常运行
-   `RPC probe: ok` —— RPC 探测通过
-   频道探测显示 `connected`（已连接）或 `ready`（就绪）

## WhatsApp

### WhatsApp 常见故障表现

| 故障现象 | 快速排查方法 | 解决方案 |
| --- | --- | --- |
| 已连接但私信无回复 | `openclaw pairing list whatsapp` | 批准发送者身份，或调整私信策略/允许列表 |
| 群组消息被忽略 | 检查配置中的 `requireMention` 及提及模式 | 在群组中 @ 机器人，或为该群组放宽提及要求 |
| 频繁断开连接、反复登录 | `openclaw channels status --probe` + 查看日志 | 重新登录并检查凭据目录是否完好 |

详细排查指南：[/channels/whatsapp#troubleshooting-quick](./whatsapp.md#troubleshooting-quick)

## Telegram

### Telegram 常见故障表现

| 故障现象 | 快速排查方法 | 解决方案 |
| --- | --- | --- |
| 发送 `/start` 后无回复流程 | `openclaw pairing list telegram` | 批准配对请求，或调整私信策略 |
| 机器人在线但群组内静默 | 检查提及要求和机器人隐私模式设置 | 关闭隐私模式以获取群组消息可见性，或在群组中 @ 机器人 |
| 发送失败并报网络错误 | 查看日志中的 Telegram API 调用失败记录 | 检查并修复到 `api.telegram.org` 的 DNS/IPv6/代理路由问题 |
| 升级后被允许列表拦截 | `openclaw security audit` 检查配置中的允许列表 | 运行 `openclaw doctor --fix`，或将 `@username` 替换为数字格式的发送者 ID |

详细排查指南：[/channels/telegram#troubleshooting](./telegram.md#troubleshooting)

## Discord

### Discord 常见故障表现

| 故障现象 | 快速排查方法 | 解决方案 |
| --- | --- | --- |
| 机器人在线但服务器内无回复 | `openclaw channels status --probe` | 允许该服务器/频道，并确认已启用消息内容意图（Message Content Intent） |
| 群组消息被忽略 | 查看日志中的提及过滤记录 | 在群组中 @ 机器人，或将该服务器/频道的 `requireMention` 设为 `false` |
| 私信无回复 | `openclaw pairing list discord` | 批准私信配对，或调整私信策略 |

详细排查指南：[/channels/discord#troubleshooting](./discord.md#troubleshooting)

## Slack

### Slack 常见故障表现

| 故障现象 | 快速排查方法 | 解决方案 |
| --- | --- | --- |
| Socket 模式已连接但无响应 | `openclaw channels status --probe` | 核验 App Token 和 Bot Token 及所需权限范围（scopes） |
| 私信被拦截 | `openclaw pairing list slack` | 批准配对请求，或放宽私信策略 |
| 频道消息被忽略 | 检查 `groupPolicy` 及频道允许列表 | 将该频道加入允许列表，或将策略改为 `open` |

详细排查指南：[/channels/slack#troubleshooting](./slack.md#troubleshooting)

## iMessage 和 BlueBubbles

### iMessage 和 BlueBubbles 常见故障表现

| 故障现象 | 快速排查方法 | 解决方案 |
| --- | --- | --- |
| 无入站事件（收不到消息） | 检查 Webhook/服务器可达性及应用权限 | 修复 Webhook URL，或检查 BlueBubbles 服务器状态 |
| macOS 上能发送但无法接收 | 检查 macOS 隐私设置中的"信息"自动化权限 | 重新授予 TCC 权限并重启频道进程 |
| 私信发送者被拦截 | `openclaw pairing list imessage` 或 `openclaw pairing list bluebubbles` | 批准配对请求，或更新允许列表 |

详细排查指南：

-   [/channels/imessage#troubleshooting-macos-privacy-and-security-tcc](./imessage.md#troubleshooting-macos-privacy-and-security-tcc)
-   [/channels/bluebubbles#troubleshooting](./bluebubbles.md#troubleshooting)

## Signal

### Signal 常见故障表现

| 故障现象 | 快速排查方法 | 解决方案 |
| --- | --- | --- |
| 守护进程可达但机器人无响应 | `openclaw channels status --probe` | 核验 `signal-cli` 守护进程 URL/账户配置及接收模式 |
| 私信被拦截 | `openclaw pairing list signal` | 批准发送者身份，或调整私信策略 |
| 群组内回复不触发 | 检查群组允许列表及提及模式配置 | 添加发送者/群组到允许列表，或放宽触发条件 |

详细排查指南：[/channels/signal#troubleshooting](./signal.md#troubleshooting)

## Matrix

### Matrix 常见故障表现

| 故障现象 | 快速排查方法 | 解决方案 |
| --- | --- | --- |
| 已登录但忽略房间消息 | `openclaw channels status --probe` | 检查 `groupPolicy` 及房间允许列表配置 |
| 私信不处理 | `openclaw pairing list matrix` | 批准发送者身份，或调整私信策略 |
| 加密房间通信失败 | 核验加密模块及加密设置 | 启用加密支持，并尝试重新加入/同步房间 |

详细排查指南：[/channels/matrix#troubleshooting](./matrix.md#troubleshooting)

[频道位置解析](./location.md)