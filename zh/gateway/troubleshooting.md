

  配置与操作

  
# 故障排除

本页是深度操作手册。如果您想先进行快速分类流程，请从 [/help/troubleshooting](../help/troubleshooting.md) 开始。

## 命令阶梯

首先按此顺序运行这些命令：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

预期的健康信号：

-   `openclaw gateway status` 显示 `Runtime: running` 和 `RPC probe: ok`。
-   `openclaw doctor` 报告没有阻塞性的配置/服务问题。
-   `openclaw channels status --probe` 显示已连接/就绪的频道。

## Anthropic 429 长上下文需要额外使用量

当日志/错误包含以下内容时使用此方法：`HTTP 429: rate_limit_error: Extra usage is required for long context requests`。

```bash
openclaw logs --follow
openclaw models status
openclaw config get agents.defaults.models
```

查找：

-   选定的 Anthropic Opus/Sonnet 模型具有 `params.context1m: true`。
-   当前的 Anthropic 凭证不符合长上下文使用资格。
-   请求仅在需要 1M beta 路径的长会话/模型运行时失败。

修复选项：

1.  为该模型禁用 `context1m` 以回退到正常上下文窗口。
2.  使用具有计费的 Anthropic API 密钥，或在订阅账户上启用 Anthropic 额外使用量。
3.  配置备用模型，以便在 Anthropic 长上下文请求被拒绝时继续运行。

相关：

-   [/providers/anthropic](../providers/anthropic.md)
-   [/reference/token-use](../reference/token-use.md)
-   [/help/faq#why-am-i-seeing-http-429-ratelimiterror-from-anthropic](../help/faq.md#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)

## 无回复

如果频道已启动但无任何回复，请在重新连接任何内容之前检查路由和策略。

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw config get channels
openclaw logs --follow
```

查找：

-   私信发送者的配对待处理。
-   群组提及门控（`requireMention`、`mentionPatterns`）。
-   频道/群组允许列表不匹配。

常见特征：

-   `drop guild message (mention required` → 群组消息在被提及前被忽略。
-   `pairing request` → 发送者需要批准。
-   `blocked` / `allowlist` → 发送者/频道被策略过滤。

相关：

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)
-   [/channels/groups](../channels/groups.md)

## 仪表板控制界面连接性

当仪表板/控制界面无法连接时，验证 URL、认证模式和安全上下文假设。

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

查找：

-   正确的探测 URL 和仪表板 URL。
-   客户端与网关之间的认证模式/令牌不匹配。
-   需要设备身份验证时使用了 HTTP。

常见特征：

-   `device identity required` → 非安全上下文或缺少设备认证。
-   `device nonce required` / `device nonce mismatch` → 客户端未完成基于挑战的设备认证流程（`connect.challenge` + `device.nonce`）。
-   `device signature invalid` / `device signature expired` → 客户端为当前握手签错了负载（或时间戳过时）。
-   `unauthorized` / 重连循环 → 令牌/密码不匹配。
-   `gateway connect failed:` → 错误的主机/端口/URL 目标。

设备认证 v2 迁移检查：

```bash
openclaw --version
openclaw doctor
openclaw gateway status
```

如果日志显示 nonce/签名错误，请更新连接的客户端并验证其：

1.  等待 `connect.challenge`
2.  对绑定挑战的负载进行签名
3.  发送带有相同挑战 nonce 的 `connect.params.device.nonce`

相关：

-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/remote](./remote.md)

## 网关服务未运行

当服务已安装但进程无法保持运行时使用此方法。

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

查找：

-   `Runtime: stopped` 并带有退出提示。
-   服务配置不匹配（`Config (cli)` 与 `Config (service)`）。
-   端口/监听器冲突。

常见特征：

-   `Gateway start blocked: set gateway.mode=local` → 本地网关模式未启用。修复：在配置中设置 `gateway.mode="local"`（或运行 `openclaw configure`）。如果您通过 Podman 使用专用的 `openclaw` 用户运行 OpenClaw，配置位于 `~openclaw/.openclaw/openclaw.json`。
-   `refusing to bind gateway ... without auth` → 非环回绑定缺少令牌/密码。
-   `another gateway instance is already listening` / `EADDRINUSE` → 端口冲突。

相关：

-   [/gateway/background-process](./background-process.md)
-   [/gateway/configuration](./configuration.md)
-   [/gateway/doctor](./doctor.md)

## 频道已连接但消息不流动

如果频道状态为已连接但消息流停滞，请关注策略、权限和频道特定的传递规则。

```bash
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

查找：

-   私信策略（`pairing`、`allowlist`、`open`、`disabled`）。
-   群组允许列表和提及要求。
-   缺少频道 API 权限/范围。

常见特征：

-   `mention required` → 消息被群组提及策略忽略。
-   `pairing` / 待批准跟踪 → 发送者未获批准。
-   `missing_scope`、`not_in_channel`、`Forbidden`、`401/403` → 频道认证/权限问题。

相关：

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/whatsapp](../channels/whatsapp.md)
-   [/channels/telegram](../channels/telegram.md)
-   [/channels/discord](../channels/discord.md)

## Cron 和心跳传递

如果 cron 或心跳未运行或未传递，请先验证调度器状态，然后检查传递目标。

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

查找：

-   Cron 已启用且存在下一次唤醒。
-   作业运行历史状态（`ok`、`skipped`、`error`）。
-   心跳跳过原因（`quiet-hours`、`requests-in-flight`、`alerts-disabled`）。

常见特征：

-   `cron: scheduler disabled; jobs will not run automatically` → cron 已禁用。
-   `cron: timer tick failed` → 调度器计时器滴答失败；检查文件/日志/运行时错误。
-   `heartbeat skipped` 且 `reason=quiet-hours` → 在活动时间窗口之外。
-   `heartbeat: unknown accountId` → 心跳传递目标账户 ID 无效。
-   `heartbeat skipped` 且 `reason=dm-blocked` → 心跳目标解析为私信类目的地，而 `agents.defaults.heartbeat.directPolicy`（或每个代理的覆盖）设置为 `block`。

相关：

-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/automation/cron-jobs](../automation/cron-jobs.md)
-   [/gateway/heartbeat](./heartbeat.md)

## 节点已配对但工具失败

如果节点已配对但工具失败，请隔离前台、权限和批准状态。

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

查找：

-   节点在线并具有预期能力。
-   摄像头/麦克风/位置/屏幕的 OS 权限授予。
-   执行批准和允许列表状态。

常见特征：

-   `NODE_BACKGROUND_UNAVAILABLE` → 节点应用必须在前台运行。
-   `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → 缺少 OS 权限。
-   `SYSTEM_RUN_DENIED: approval required` → 执行批准待处理。
-   `SYSTEM_RUN_DENIED: allowlist miss` → 命令被允许列表阻止。

相关：

-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/nodes/index](../nodes/index.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

## 浏览器工具失败

当浏览器工具操作失败，但网关本身健康时使用此方法。

```bash
openclaw browser status
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

查找：

-   有效的浏览器可执行文件路径。
-   CDP 配置文件可达性。
-   `profile="chrome"` 的扩展中继标签页附加。

常见特征：

-   `Failed to start Chrome CDP on port` → 浏览器进程启动失败。
-   `browser.executablePath not found` → 配置的路径无效。
-   `Chrome extension relay is running, but no tab is connected` → 扩展中继未附加。
-   `Browser attachOnly is enabled ... not reachable` → 仅附加配置文件没有可达目标。

相关：

-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)
-   [/tools/browser](../tools/browser.md)

## 如果您升级后某些功能突然损坏

大多数升级后的损坏是由于配置漂移或现在强制执行了更严格的默认设置。

### 1) 认证和 URL 覆盖行为已更改

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

需要检查的内容：

-   如果 `gateway.mode=remote`，CLI 调用可能针对远程网关，而您的本地服务正常。
-   显式的 `--url` 调用不会回退到存储的凭证。

常见特征：

-   `gateway connect failed:` → 错误的 URL 目标。
-   `unauthorized` → 端点可达但认证错误。

### 2) 绑定和认证防护措施更严格

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

需要检查的内容：

-   非环回绑定（`lan`、`tailnet`、`custom`）需要配置认证。
-   旧键如 `gateway.token` 不会替换 `gateway.auth.token`。

常见特征：

-   `refusing to bind gateway ... without auth` → 绑定+认证不匹配。
-   `RPC probe: failed` 但运行时正在运行 → 网关存活但使用当前认证/URL 无法访问。

### 3) 配对和设备身份状态已更改

```bash
openclaw devices list
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
openclaw doctor
```

需要检查的内容：

-   仪表板/节点的待处理设备批准。
-   策略或身份更改后的待处理私信配对批准。

常见特征：

-   `device identity required` → 设备认证未满足。
-   `pairing required` → 发送者/设备必须获得批准。

如果检查后服务配置和运行时仍然不一致，请从相同的配置文件/状态目录重新安装服务元数据：

```bash
openclaw gateway install --force
openclaw gateway restart
```

相关：

-   [/gateway/pairing](./pairing.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/background-process](./background-process.md)

[多个网关](./multiple-gateways.md)[安全性](./security.md)