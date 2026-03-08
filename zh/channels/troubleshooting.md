

  配置

  
# 通道故障排除

当通道已连接但行为异常时，请使用此页面。

## 命令阶梯

首先按顺序运行这些命令：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

健康基线：

-   `Runtime: running`
-   `RPC probe: ok`
-   通道探测显示已连接/就绪

## WhatsApp

### WhatsApp 故障特征

| 症状 | 最快检查方法 | 修复方法 |
| --- | --- | --- |
| 已连接但无私信回复 | `openclaw pairing list whatsapp` | 批准发送者或切换私信策略/允许列表。 |
| 群组消息被忽略 | 检查配置中的 `requireMention` 和提及模式 | 提及机器人或为该群组放宽提及策略。 |
| 随机断开连接/重新登录循环 | `openclaw channels status --probe` + 日志 | 重新登录并验证凭据目录是否正常。 |

完整故障排除：[/channels/whatsapp#troubleshooting-quick](./whatsapp.md#troubleshooting-quick)

## Telegram

### Telegram 故障特征

| 症状 | 最快检查方法 | 修复方法 |
| --- | --- | --- |
| 发送 `/start` 但无可用回复流程 | `openclaw pairing list telegram` | 批准配对或更改私信策略。 |
| 机器人在线但群组保持静默 | 验证提及要求和机器人隐私模式 | 为群组可见性禁用隐私模式或提及机器人。 |
| 发送失败并伴随网络错误 | 检查日志中 Telegram API 调用失败信息 | 修复到 `api.telegram.org` 的 DNS/IPv6/代理路由。 |
| 升级后允许列表阻止了你 | `openclaw security audit` 和配置允许列表 | 运行 `openclaw doctor --fix` 或将 `@username` 替换为数字发送者 ID。 |

完整故障排除：[/channels/telegram#troubleshooting](./telegram.md#troubleshooting)

## Discord

### Discord 故障特征

| 症状 | 最快检查方法 | 修复方法 |
| --- | --- | --- |
| 机器人在线但无服务器回复 | `openclaw channels status --probe` | 允许服务器/频道并验证消息内容意图。 |
| 群组消息被忽略 | 检查日志中提及门控丢弃信息 | 提及机器人或将服务器/频道设置为 `requireMention: false`。 |
| 缺少私信回复 | `openclaw pairing list discord` | 批准私信配对或调整私信策略。 |

完整故障排除：[/channels/discord#troubleshooting](./discord.md#troubleshooting)

## Slack

### Slack 故障特征

| 症状 | 最快检查方法 | 修复方法 |
| --- | --- | --- |
| Socket 模式已连接但无响应 | `openclaw channels status --probe` | 验证应用令牌 + 机器人令牌以及所需权限范围。 |
| 私信被阻止 | `openclaw pairing list slack` | 批准配对或放宽私信策略。 |
| 频道消息被忽略 | 检查 `groupPolicy` 和频道允许列表 | 允许该频道或将策略切换为 `open`。 |

完整故障排除：[/channels/slack#troubleshooting](./slack.md#troubleshooting)

## iMessage 和 BlueBubbles

### iMessage 和 BlueBubbles 故障特征

| 症状 | 最快检查方法 | 修复方法 |
| --- | --- | --- |
| 无入站事件 | 验证 Webhook/服务器可达性及应用权限 | 修复 Webhook URL 或 BlueBubbles 服务器状态。 |
| 在 macOS 上可以发送但无法接收 | 检查 macOS 隐私权限以获取“信息”自动化权限 | 重新授予 TCC 权限并重启通道进程。 |
| 私信发送者被阻止 | `openclaw pairing list imessage` 或 `openclaw pairing list bluebubbles` | 批准配对或更新允许列表。 |

完整故障排除：

-   [/channels/imessage#troubleshooting-macos-privacy-and-security-tcc](./imessage.md#troubleshooting-macos-privacy-and-security-tcc)
-   [/channels/bluebubbles#troubleshooting](./bluebubbles.md#troubleshooting)

## Signal

### Signal 故障特征

| 症状 | 最快检查方法 | 修复方法 |
| --- | --- | --- |
| 守护进程可达但机器人静默 | `openclaw channels status --probe` | 验证 `signal-cli` 守护进程 URL/账户和接收模式。 |
| 私信被阻止 | `openclaw pairing list signal` | 批准发送者或调整私信策略。 |
| 群组回复未触发 | 检查群组允许列表和提及模式 | 添加发送者/群组或放宽门控。 |

完整故障排除：[/channels/signal#troubleshooting](./signal.md#troubleshooting)

## Matrix

### Matrix 故障特征

| 症状 | 最快检查方法 | 修复方法 |
| --- | --- | --- |
| 已登录但忽略房间消息 | `openclaw channels status --probe` | 检查 `groupPolicy` 和房间允许列表。 |
| 私信未处理 | `openclaw pairing list matrix` | 批准发送者或调整私信策略。 |
| 加密房间失败 | 验证加密模块和加密设置 | 启用加密支持并重新加入/同步房间。 |

完整故障排除：[/channels/matrix#troubleshooting](./matrix.md#troubleshooting)

[通道位置解析](./location.md)

---