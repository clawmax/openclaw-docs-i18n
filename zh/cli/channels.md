

  CLI 命令

  
# channels

管理网关上的聊天频道账户及其运行时状态。相关文档：

-   频道指南：[Channels](../channels/index.md)
-   网关配置：[Configuration](../gateway/configuration.md)

## 常用命令

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

## 添加 / 移除账户

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

提示：`openclaw channels add --help` 显示每个频道的特定标志（令牌、应用令牌、signal-cli 路径等）。当您不带标志运行 `openclaw channels add` 时，交互式向导可以提示：

-   所选频道的账户 ID
-   这些账户的可选显示名称
-   `现在将配置的频道账户绑定到代理吗？`

如果您确认立即绑定，向导会询问哪个代理应拥有每个已配置的频道账户，并写入账户作用域的路由绑定。您也可以稍后使用 `openclaw agents bindings`、`openclaw agents bind` 和 `openclaw agents unbind` 管理相同的路由规则（参见 [agents](./agents.md)）。当您向一个仍在使用单账户顶级设置（尚无 `channels..accounts` 条目）的频道添加非默认账户时，OpenClaw 会将账户作用域的单账户顶级值移动到 `channels..accounts.default` 中，然后写入新账户。这保留了原始账户行为，同时迁移到多账户结构。路由行为保持一致：

-   现有的仅频道绑定（无 `accountId`）继续匹配默认账户。
-   在非交互模式下，`channels add` 不会自动创建或重写绑定。
-   交互式设置可以选择性地添加账户作用域的绑定。

如果您的配置已处于混合状态（存在命名账户、缺少 `default`，并且仍设置了顶级单账户值），请运行 `openclaw doctor --fix` 将账户作用域的值移动到 `accounts.default` 中。

## 登录 / 登出（交互式）

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

## 故障排除

-   运行 `openclaw status --deep` 进行广泛探测。
-   使用 `openclaw doctor` 进行引导式修复。
-   `openclaw channels list` 打印 `Claude: HTTP 403 ... user:profile` → 使用情况快照需要 `user:profile` 权限。使用 `--no-usage`，或提供 claude.ai 会话密钥（`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`），或通过 Claude Code CLI 重新授权。
-   当网关无法访问时，`openclaw channels status` 会回退到仅配置摘要。如果通过 SecretRef 配置了支持的频道凭据，但在当前命令路径中不可用，它会将该账户报告为已配置但带有降级说明，而不是显示为未配置。

## 能力探测

获取提供商能力提示（可用的意图/权限范围）以及静态功能支持：

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

注意：

-   `--channel` 是可选的；省略它以列出每个频道（包括扩展）。
-   `--target` 接受 `channel:` 或原始数字频道 ID，仅适用于 Discord。
-   探测是提供商特定的：Discord 意图 + 可选的频道权限；Slack 机器人 + 用户权限范围；Telegram 机器人标志 + webhook；Signal 守护进程版本；MS Teams 应用令牌 + Graph 角色/权限范围（在已知处标注）。没有探测的频道报告 `Probe: unavailable`。

## 将名称解析为 ID

使用提供商目录将频道/用户名称解析为 ID：

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

注意：

-   使用 `--kind user|group|auto` 强制指定目标类型。
-   当多个条目共享相同名称时，解析优先选择活跃匹配项。
-   `channels resolve` 是只读的。如果所选账户通过 SecretRef 配置，但该凭据在当前命令路径中不可用，命令将返回带有说明的降级未解析结果，而不是中止整个运行。

[browser](./browser.md)[clawbot](./clawbot.md)