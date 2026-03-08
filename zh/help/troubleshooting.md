

  帮助

  
# 故障排除

如果你只有2分钟，请将此页面作为分诊入口。

## 前60秒

按顺序运行以下精确命令阶梯：

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw gateway status
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

良好的单行输出：

-   `openclaw status` → 显示已配置的频道且无明显的身份验证错误。
-   `openclaw status --all` → 完整报告存在且可共享。
-   `openclaw gateway probe` → 预期的网关目标可达。
-   `openclaw gateway status` → `Runtime: running` 且 `RPC probe: ok`。
-   `openclaw doctor` → 无阻塞性的配置/服务错误。
-   `openclaw channels status --probe` → 频道报告为 `connected` 或 `ready`。
-   `openclaw logs --follow` → 活动稳定，无重复的致命错误。

## Anthropic 长上下文 429 错误

如果看到：`HTTP 429: rate_limit_error: Extra usage is required for long context requests`，请前往 [/gateway/troubleshooting#anthropic-429-extra-usage-required-for-long-context](../gateway/troubleshooting.md#anthropic-429-extra-usage-required-for-long-context)。

## 插件安装因缺少 openclaw 扩展而失败

如果安装失败并提示 `package.json missing openclaw.extensions`，说明插件包使用了 OpenClaw 不再接受的旧格式。在插件包中修复：

1.  在 `package.json` 中添加 `openclaw.extensions`。
2.  将条目指向构建后的运行时文件（通常是 `./dist/index.js`）。
3.  重新发布插件并再次运行 `openclaw plugins install <npm-spec>`。

示例：

```json
{
  "name": "@openclaw/my-plugin",
  "version": "1.2.3",
  "openclaw": {
    "extensions": ["./dist/index.js"]
  }
}
```

参考：[/tools/plugin#distribution-npm](../tools/plugin.md#distribution-npm)

## 决策树

```bash
openclaw status
openclaw gateway status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
```

良好的输出看起来像：

-   `Runtime: running`
-   `RPC probe: ok`
-   你的频道在 `channels status --probe` 中显示为 connected/ready
-   发送者显示为已批准（或 DM 策略为开放/允许列表）

常见的日志特征：

-   `drop guild message (mention required` → Discord 中的提及门控阻止了消息。
-   `pairing request` → 发送者未获批准，正在等待 DM 配对批准。
-   频道日志中的 `blocked` / `allowlist` → 发送者、房间或群组被过滤。

深入页面：

-   [/gateway/troubleshooting#no-replies](../gateway/troubleshooting.md#no-replies)
-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

良好的输出看起来像：

-   `openclaw gateway status` 中显示 `Dashboard: http://...`
-   `RPC probe: ok`
-   日志中无身份验证循环

常见的日志特征：

-   `device identity required` → HTTP/非安全上下文无法完成设备身份验证。
-   `unauthorized` / 重连循环 → 错误的令牌/密码或身份验证模式不匹配。
-   `gateway connect failed:` → 界面指向错误的 URL/端口或网关不可达。

深入页面：

-   [/gateway/troubleshooting#dashboard-control-ui-connectivity](../gateway/troubleshooting.md#dashboard-control-ui-connectivity)
-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](../gateway/authentication.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

良好的输出看起来像：

-   `Service: ... (loaded)`
-   `Runtime: running`
-   `RPC probe: ok`

常见的日志特征：

-   `Gateway start blocked: set gateway.mode=local` → 网关模式未设置/为远程。
-   `refusing to bind gateway ... without auth` → 非回环绑定但未设置令牌/密码。
-   `another gateway instance is already listening` 或 `EADDRINUSE` → 端口已被占用。

深入页面：

-   [/gateway/troubleshooting#gateway-service-not-running](../gateway/troubleshooting.md#gateway-service-not-running)
-   [/gateway/background-process](../gateway/background-process.md)
-   [/gateway/configuration](../gateway/configuration.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

良好的输出看起来像：

-   频道传输已连接。
-   配对/允许列表检查通过。
-   在需要的地方检测到提及。

常见的日志特征：

-   `mention required` → 群组提及门控阻止了处理。
-   `pairing` / `pending` → DM 发送者尚未获批准。
-   `not_in_channel`, `missing_scope`, `Forbidden`, `401/403` → 频道权限令牌问题。

深入页面：

-   [/gateway/troubleshooting#channel-connected-messages-not-flowing](../gateway/troubleshooting.md#channel-connected-messages-not-flowing)
-   [/channels/troubleshooting](../channels/troubleshooting.md)

```bash
openclaw status
openclaw gateway status
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

良好的输出看起来像：

-   `cron.status` 显示已启用且有下一次唤醒。
-   `cron runs` 显示最近的 `ok` 条目。
-   心跳已启用且未超出活动时间。

常见的日志特征：

-   `cron: scheduler disabled; jobs will not run automatically` → 定时任务已禁用。
-   `heartbeat skipped` 附带 `reason=quiet-hours` → 超出配置的活动时间。
-   `requests-in-flight` → 主通道繁忙；心跳唤醒被推迟。
-   `unknown accountId` → 心跳送达的目标账户不存在。

深入页面：

-   [/gateway/troubleshooting#cron-and-heartbeat-delivery](../gateway/troubleshooting.md#cron-and-heartbeat-delivery)
-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)

```bash
openclaw status
openclaw gateway status
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw logs --follow
```

良好的输出看起来像：

-   节点被列为已连接且已为 `node` 角色配对。
-   存在你要调用的命令所需的能力。
-   工具的权限状态为已授予。

常见的日志特征：

-   `NODE_BACKGROUND_UNAVAILABLE` → 将节点应用切换到前台。
-   `*_PERMISSION_REQUIRED` → 操作系统权限被拒绝/缺失。
-   `SYSTEM_RUN_DENIED: approval required` → 命令执行批准待处理。
-   `SYSTEM_RUN_DENIED: allowlist miss` → 命令不在执行允许列表中。

深入页面：

-   [/gateway/troubleshooting#node-paired-tool-fails](../gateway/troubleshooting.md#node-paired-tool-fails)
-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

```bash
openclaw status
openclaw gateway status
openclaw browser status
openclaw logs --follow
openclaw doctor
```

良好的输出看起来像：

-   浏览器状态显示 `running: true` 以及一个选定的浏览器/配置文件。
-   `openclaw` 配置文件启动或 `chrome` 中继有一个已连接的标签页。

常见的日志特征：

-   `Failed to start Chrome CDP on port` → 本地浏览器启动失败。
-   `browser.executablePath not found` → 配置的二进制路径错误。
-   `Chrome extension relay is running, but no tab is connected` → 扩展未连接。
-   `Browser attachOnly is enabled ... not reachable` → 仅附加配置文件没有活动的 CDP 目标。

深入页面：

-   [/gateway/troubleshooting#browser-tool-fails](../gateway/troubleshooting.md#browser-tool-fails)
-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)

[帮助](../help.md)[常见问题](./faq.md)