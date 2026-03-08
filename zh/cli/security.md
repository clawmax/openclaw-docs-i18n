

  CLI 命令

  
# security

安全工具（审计 + 可选修复）。相关：

-   安全指南：[Security](../gateway/security.md)

## 审计

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

当多个 DM 发送者共享主会话时，审计会发出警告，并建议使用**安全 DM 模式**：对于共享收件箱，设置 `session.dmScope="per-channel-peer"`（或对于多账户频道，设置为 `per-account-channel-peer`）。这适用于协作/共享收件箱的加固。不建议由互不信任/敌对的多个操作者共享单个网关；应使用独立的网关（或独立的操作系统用户/主机）来划分信任边界。当配置暗示可能存在共享用户入口时（例如开放的 DM/群组策略、配置的群组目标或通配符发送者规则），它还会发出 `security.trust_model.multi_user_heuristic` 警告，并提醒您 OpenClaw 默认是个人助手信任模型。对于有意的共享用户设置，审计指导是沙箱化所有会话，将文件系统访问限制在工作区范围内，并且不要在该运行时中存放个人/私人身份或凭据。当使用小型模型（`<=300B`）且未启用沙箱并启用了网页/浏览器工具时，它也会发出警告。对于 Webhook 入口，当 `hooks.defaultSessionKey` 未设置、请求 `sessionKey` 覆盖被启用，以及覆盖被启用但未设置 `hooks.allowedSessionKeyPrefixes` 时，它会发出警告。当沙箱 Docker 设置已配置但沙箱模式关闭时，当 `gateway.nodes.denyCommands` 使用了无效的模式类/未知条目时（仅精确匹配节点命令名称，而非 shell 文本过滤），当 `gateway.nodes.allowCommands` 显式启用了危险的节点命令时，当全局 `tools.profile="minimal"` 被代理工具配置文件覆盖时，当开放的群组暴露运行时/文件系统工具而没有沙箱/工作区防护时，以及当已安装的扩展插件工具可能在宽松的工具策略下可访问时，它也会发出警告。它还会标记 `gateway.allowRealIpFallback=true`（如果代理配置错误，存在头部欺骗风险）和 `discovery.mdns.mode="full"`（通过 mDNS TXT 记录可能导致元数据泄漏）。当沙箱浏览器使用 Docker `bridge` 网络但未设置 `sandbox.browser.cdpSourceRange` 时，它也会发出警告。它还会标记危险的沙箱 Docker 网络模式（包括 `host` 和 `container:*` 命名空间加入）。当现有的沙箱浏览器 Docker 容器缺少/过时的哈希标签时（例如，迁移前的容器缺少 `openclaw.browserConfigEpoch`），它会发出警告，并建议执行 `openclaw sandbox recreate --browser --all`。当基于 npm 的插件/钩子安装记录未固定、缺少完整性元数据或与当前安装的软件包版本存在漂移时，它会发出警告。当频道允许列表依赖于可变的名称/电子邮件/标签而非稳定的 ID 时（Discord、Slack、Google Chat、MS Teams、Mattermost、IRC 范围，如适用），它会发出警告。当 `gateway.auth.mode="none"` 使得网关 HTTP API 可在没有共享密钥的情况下访问时（`/tools/invoke` 加上任何启用的 `/v1/*` 端点），它会发出警告。以 `dangerous`/`dangerously` 为前缀的设置是明确的应急操作员覆盖；启用它们本身并不构成安全漏洞报告。有关完整的危险参数清单，请参阅 [Security](../gateway/security.md) 中的“不安全或危险标志摘要”部分。

## JSON 输出

使用 `--json` 进行 CI/策略检查：

```bash
openclaw security audit --json | jq '.summary'
openclaw security audit --deep --json | jq '.findings[] | select(.severity=="critical") | .checkId'
```

如果同时使用 `--fix` 和 `--json`，输出将包含修复操作和最终报告：

```bash
openclaw security audit --fix --json | jq '{fix: .fix.ok, summary: .report.summary}'
```

## --fix 会更改什么

`--fix` 应用安全、确定性的修复措施：

-   将常见的 `groupPolicy="open"` 更改为 `groupPolicy="allowlist"`（包括支持的频道中的账户变体）
-   将 `logging.redactSensitive` 从 `"off"` 设置为 `"tools"`
-   收紧状态/配置和常见敏感文件（`credentials/*.json`、`auth-profiles.json`、`sessions.json`、会话 `*.jsonl`）的权限

`--fix` **不会**：

-   轮换令牌/密码/API 密钥
-   禁用工具（`gateway`、`cron`、`exec` 等）
-   更改网关绑定/认证/网络暴露选择
-   移除或重写插件/技能

[secrets](./secrets.md)[sessions](./sessions.md)