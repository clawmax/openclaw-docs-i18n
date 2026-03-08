

  安全与沙箱

  
# 安全

> \[!WARNING\] **个人助理信任模型：** 本指南假设每个网关有一个受信任的操作员边界（单用户/个人助理模型）。OpenClaw **不是** 为多个敌对用户共享一个代理/网关而设计的敌对多租户安全边界。如果您需要混合信任或敌对用户操作，请分割信任边界（单独的网关 + 凭证，理想情况下是单独的操作系统用户/主机）。

## 范围优先：个人助理安全模型

OpenClaw 安全指南假设采用 **个人助理** 部署：一个受信任的操作员边界，可能包含多个代理。

-   支持的安全态势：每个网关一个用户/信任边界（建议每个边界使用一个操作系统用户/主机/VPS）。
-   不支持的安全边界：一个由互不信任或敌对的用户共享的网关/代理。
-   如果需要敌对用户隔离，请按信任边界进行分割（单独的网关 + 凭证，理想情况下是单独的操作系统用户/主机）。
-   如果多个不受信任的用户可以向一个启用了工具的代理发送消息，则将他们视为共享该代理的相同委托工具权限。

本页解释了 **在该模型内** 的强化措施。它不声称在一个共享网关上实现敌对的的多租户隔离。

## 快速检查：openclaw 安全审计

另请参阅：[形式化验证（安全模型）](../security/formal-verification.md) 定期运行此命令（尤其是在更改配置或暴露网络接口后）：

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

它会标记常见的隐患（网关认证暴露、浏览器控制暴露、提升的允许列表、文件系统权限）。OpenClaw 既是一个产品也是一个实验：您正在将前沿模型的行为连接到真实的消息传递接口和真实工具。**不存在“绝对安全”的设置。** 目标是审慎地决定：

-   谁可以与您的机器人对话
-   机器人被允许在哪里行动
-   机器人可以接触什么

从最小但仍能工作的访问权限开始，然后在获得信心后逐步放宽。

## 部署假设（重要）

OpenClaw 假设主机和配置边界是受信任的：

-   如果有人可以修改网关主机状态/配置（`~/.openclaw`，包括 `openclaw.json`），则将其视为受信任的操作员。
-   为多个互不信任/敌对的操作员运行一个网关 **不是推荐的做法**。
-   对于混合信任的团队，使用单独的网关（或至少是单独的操作系统用户/主机）分割信任边界。
-   OpenClaw 可以在一台机器上运行多个网关实例，但推荐的操作倾向于清晰的信任边界分离。
-   推荐的默认设置：每台机器/主机（或 VPS）一个用户，为该用户运行一个网关，以及该网关中的一个或多个代理。
-   如果多个用户需要使用 OpenClaw，请为每个用户使用一个 VPS/主机。

### 实际后果（操作员信任边界）

在一个网关实例内部，经过身份验证的操作员访问是一个受信任的控制平面角色，而不是每个用户的租户角色。

-   具有读取/控制平面访问权限的操作员可以按设计检查网关会话元数据/历史记录。
-   会话标识符（`sessionKey`、会话 ID、标签）是路由选择器，而不是授权令牌。
-   示例：期望像 `sessions.list`、`sessions.preview` 或 `chat.history` 这样的方法实现每个操作员的隔离，这超出了此模型的范围。
-   如果您需要敌对用户隔离，请为每个信任边界运行单独的网关。
-   在一台机器上运行多个网关在技术上是可行的，但不是多用户隔离的推荐基线。

## 个人助理模型（非多租户总线）

OpenClaw 被设计为个人助理安全模型：一个受信任的操作员边界，可能包含多个代理。

-   如果几个人可以向一个启用了工具的代理发送消息，他们每个人都可以驱动相同的权限集。
-   每个用户的会话/内存隔离有助于隐私，但不会将共享代理转换为每个用户的主机授权。
-   如果用户之间可能存在敌对关系，请为每个信任边界运行单独的网关（或单独的操作系统用户/主机）。

### 共享 Slack 工作区：真实风险

如果“Slack 中的每个人都可以向机器人发送消息”，核心风险在于委托的工具权限：

-   任何允许的发送者都可以在代理的策略范围内诱导工具调用（`exec`、浏览器、网络/文件工具）；
-   来自一个发送者的提示/内容注入可能导致影响共享状态、设备或输出的操作；
-   如果一个共享代理拥有敏感的凭证/文件，任何允许的发送者都可能通过工具使用驱动数据外泄。

为团队工作流使用具有最少工具的单独代理/网关；将个人数据代理保持为私有。

### 公司共享代理：可接受的模式

当使用该代理的每个人都在同一个信任边界内（例如一个公司团队）并且代理严格限定在业务范围内时，这是可以接受的。

-   在专用机器/虚拟机/容器上运行它；
-   为该运行时使用专用的操作系统用户 + 专用的浏览器/配置文件/账户；
-   不要使用该运行时登录个人 Apple/Google 账户或个人密码管理器/浏览器配置文件。

如果您在同一运行时混合个人和公司身份，就会破坏分离并增加个人数据暴露风险。

## 网关和节点信任概念

将网关和节点视为一个操作员信任域，具有不同的角色：

-   **网关** 是控制平面和策略接口（`gateway.auth`、工具策略、路由）。
-   **节点** 是与该网关配对的远程执行接口（命令、设备操作、主机本地能力）。
-   经过网关身份验证的调用者在网关范围内是受信任的。配对后，节点操作就是该节点上受信任的操作员操作。
-   `sessionKey` 是路由/上下文选择器，不是每个用户的认证。
-   执行批准（允许列表 + 询问）是操作员意图的防护栏，不是敌对的的多租户隔离。

如果您需要敌对用户隔离，请按操作系统用户/主机分割信任边界并运行单独的网关。

## 信任边界矩阵

在评估风险时，使用此快速模型：

| 边界或控制 | 含义 | 常见误解 |
| --- | --- | --- |
| `gateway.auth`（令牌/密码/设备认证） | 验证调用者对网关 API 的访问 | “需要在每一帧上都有每条消息的签名才能安全” |
| `sessionKey` | 用于上下文/会话选择的路由键 | “会话密钥是用户认证边界” |
| 提示/内容防护栏 | 降低模型滥用风险 | “仅凭提示注入就证明存在认证绕过” |
| `canvas.eval` / 浏览器评估 | 启用时是故意的操作员能力 | “在此信任模型中，任何 JS eval 原语自动成为漏洞” |
| 本地 TUI `!` shell | 明确由操作员触发的本地执行 | “本地 shell 便捷命令是远程注入” |
| 节点配对和节点命令 | 在配对设备上的操作员级远程执行 | “远程设备控制默认应被视为不受信任的用户访问” |

## 设计上非漏洞

这些模式经常被报告，并且通常在不显示真实边界绕过的情况下被关闭为无需处理：

-   仅提示注入链，没有策略/认证/沙箱绕过。
-   假设在一个共享主机/配置上进行敌对的多租户操作。
-   将正常的操作员读取路径访问（例如 `sessions.list`/`sessions.preview`/`chat.history`）归类为共享网关设置中的 IDOR。
-   仅限本地主机的部署发现（例如仅回环网关上的 HSTS）。
-   针对此仓库中不存在的入站路径的 Discord 入站 Webhook 签名发现。
-   将 `sessionKey` 视为认证令牌的“缺少每个用户授权”发现。

## 研究人员预检清单

在打开 GHSA 之前，请验证所有这些：

1.  复现步骤在最新的 `main` 分支或最新版本上仍然有效。
2.  报告包含确切的代码路径（`文件`、函数、行范围）和测试的版本/提交。
3.  影响跨越了文档化的信任边界（不仅仅是提示注入）。
4.  声明未列在[范围之外](https://github.com/openclaw/openclaw/blob/main/SECURITY.md#out-of-scope)。
5.  已检查现有公告是否存在重复（适用时重用规范的 GHSA）。
6.  部署假设是明确的（回环/本地与暴露、受信任与不受信任的操作员）。

## 60 秒内的强化基线

首先使用此基线，然后为受信任的代理选择性地重新启用工具：

```json
{
  gateway: {
    mode: "local",
    bind: "loopback",
    auth: { mode: "token", token: "replace-with-long-random-token" },
  },
  session: {
    dmScope: "per-channel-peer",
  },
  tools: {
    profile: "messaging",
    deny: ["group:automation", "group:runtime", "group:fs", "sessions_spawn", "sessions_send"],
    fs: { workspaceOnly: true },
    exec: { security: "deny", ask: "always" },
    elevated: { enabled: false },
  },
  channels: {
    whatsapp: { dmPolicy: "pairing", groups: { "*": { requireMention: true } } },
  },
}
```

这使网关保持仅限本地，隔离私信，并默认禁用控制平面/运行时工具。

## 共享收件箱快速规则

如果不止一个人可以向您的机器人发送私信：

-   设置 `session.dmScope: "per-channel-peer"`（对于多账户频道，使用 `"per-account-channel-peer"`）。
-   保持 `dmPolicy: "pairing"` 或严格的允许列表。
-   切勿将共享私信与广泛的工具访问权限结合使用。
-   这强化了协作/共享收件箱，但当用户共享主机/配置写入权限时，并非设计为敌对的共租户隔离。

### 审计检查的内容（高级别）

-   **入站访问**（私信策略、群组策略、允许列表）：陌生人能否触发机器人？
-   **工具爆炸半径**（提升的工具 + 开放房间）：提示注入是否会转变为 shell/文件/网络操作？
-   **网络暴露**（网关绑定/认证、Tailscale Serve/Funnel、弱/短的认证令牌）。
-   **浏览器控制暴露**（远程节点、中继端口、远程 CDP 端点）。
-   **本地磁盘卫生**（权限、符号链接、配置包含、“同步文件夹”路径）。
-   **插件**（扩展存在但没有明确的允许列表）。
-   **策略漂移/配置错误**（配置了沙箱 Docker 设置但沙箱模式关闭；`gateway.nodes.denyCommands` 模式无效，因为匹配仅针对确切的命令名称（例如 `system.run`）且不检查 shell 文本；危险的 `gateway.nodes.allowCommands` 条目；全局 `tools.profile="minimal"` 被每个代理的配置文件覆盖；扩展插件工具在宽松的工具策略下可访问）。
-   **运行时期望漂移**（例如 `tools.exec.host="sandbox"` 而沙箱模式关闭，这将在网关主机上直接运行）。
-   **模型卫生**（当配置的模型看起来过时时发出警告；不是硬性阻止）。

如果运行 `--deep`，OpenClaw 还会尝试尽力对网关进行实时探测。

## 凭证存储映射

在审计访问或决定备份内容时使用此映射：

-   **WhatsApp**：`~/.openclaw/credentials/whatsapp//creds.json`
-   **Telegram 机器人令牌**：配置/环境变量或 `channels.telegram.tokenFile`
-   **Discord 机器人令牌**：配置/环境变量或 SecretRef（环境变量/文件/执行提供程序）
-   **Slack 令牌**：配置/环境变量（`channels.slack.*`）
-   **配对允许列表**：
    -   `~/.openclaw/credentials/-allowFrom.json`（默认账户）
    -   `~/.openclaw/credentials/--allowFrom.json`（非默认账户）
-   **模型认证配置文件**：`~/.openclaw/agents//agent/auth-profiles.json`
-   **文件支持的密钥有效负载（可选）**：`~/.openclaw/secrets.json`
-   **旧版 OAuth 导入**：`~/.openclaw/credentials/oauth.json`

## 安全审计清单

当审计打印发现时，请将其视为优先级顺序：

1.  **任何“开放”+ 工具启用**：首先锁定私信/群组（配对/允许列表），然后收紧工具策略/沙箱。
2.  **公共网络暴露**（局域网绑定、Funnel、缺少认证）：立即修复。
3.  **浏览器控制远程暴露**：将其视为操作员访问（仅限 tailnet，审慎配对节点，避免公共暴露）。
4.  **权限**：确保状态/配置/凭证/认证不是组/全局可读的。
5.  **插件/扩展**：仅加载您明确信任的内容。
6.  **模型选择**：对于任何带有工具的机器人，优先选择现代、指令强化的模型。

## 安全审计术语表

您在实际部署中最可能看到的高信号 `checkId` 值（非详尽）：

| `checkId` | 严重性 | 为何重要 | 主要修复键/路径 | 自动修复 |
| --- | --- | --- | --- | --- |
| `fs.state_dir.perms_world_writable` | 严重 | 其他用户/进程可以修改完整的 OpenClaw 状态 | `~/.openclaw` 的文件系统权限 | 是 |
| `fs.config.perms_writable` | 严重 | 其他人可以更改认证/工具策略/配置 | `~/.openclaw/openclaw.json` 的文件系统权限 | 是 |
| `fs.config.perms_world_readable` | 严重 | 配置可能暴露令牌/设置 | 配置文件上的文件系统权限 | 是 |
| `gateway.bind_no_auth` | 严重 | 远程绑定没有共享密钥 | `gateway.bind`, `gateway.auth.*` | 否 |
| `gateway.loopback_no_auth` | 严重 | 反向代理的回环可能变为未认证 | `gateway.auth.*`, 代理设置 | 否 |
| `gateway.http.no_auth` | 警告/严重 | 网关 HTTP API 在 `auth.mode="none"` 时可访问 | `gateway.auth.mode`, `gateway.http.endpoints.*` | 否 |
| `gateway.tools_invoke_http.dangerous_allow` | 警告/严重 | 通过 HTTP API 重新启用危险工具 | `gateway.tools.allow` | 否 |
| `gateway.nodes.allow_commands_dangerous` | 警告/严重 | 启用高影响节点命令（相机/屏幕/联系人/日历/短信） | `gateway.nodes.allowCommands` | 否 |
| `gateway.tailscale_funnel` | 严重 | 公共互联网暴露 | `gateway.tailscale.mode` | 否 |
| `gateway.control_ui.allowed_origins_required` | 严重 | 非回环控制 UI 没有明确的浏览器来源允许列表 | `gateway.controlUi.allowedOrigins` | 否 |
| `gateway.control_ui.host_header_origin_fallback` | 警告/严重 | 启用 Host 头来源回退（DNS 重绑定强化降级） | `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` | 否 |
| `gateway.control_ui.insecure_auth` | 警告 | 启用不安全认证兼容性开关 | `gateway.controlUi.allowInsecureAuth` | 否 |
| `gateway.control_ui.device_auth_disabled` | 严重 | 禁用设备身份检查 | `gateway.controlUi.dangerouslyDisableDeviceAuth` | 否 |
| `gateway.real_ip_fallback_enabled` | 警告/严重 | 信任 `X-Real-IP` 回退可能通过代理配置错误启用源 IP 欺骗 | `gateway.allowRealIpFallback`, `gateway.trustedProxies` | 否 |
| `discovery.mdns_full_mode` | 警告/严重 | mDNS 完整模式在本地网络上广播 `cliPath`/`sshPort` 元数据 | `discovery.mdns.mode`, `gateway.bind` | 否 |
| `config.insecure_or_dangerous_flags` | 警告 | 任何不安全/危险的调试标志已启用 | 多个键（参见发现详情） | 否 |
| `hooks.token_too_short` | 警告 | 更容易对钩子入口进行暴力破解 | `hooks.token` | 否 |
| `hooks.request_session_key_enabled` | 警告/严重 | 外部调用者可以选择 sessionKey | `hooks.allowRequestSessionKey` | 否 |
| `hooks.request_session_key_prefixes_missing` | 警告/严重 | 对外部会话键形状没有限制 | `hooks.allowedSessionKeyPrefixes` | 否 |
| `logging.redact_off` | 警告 | 敏感值泄露到日志/状态 | `logging.redactSensitive` | 是 |
| `sandbox.docker_config_mode_off` | 警告 | 存在沙箱 Docker 配置但未激活 | `agents.*.sandbox.mode` | 否 |
| `sandbox.dangerous_network_mode` | 严重 | 沙箱 Docker 网络使用 `host` 或 `container:*` 命名空间加入模式 | `agents.*.sandbox.docker.network` | 否 |
| `tools.exec.host_sandbox_no_sandbox_defaults` | 警告 | `exec host=sandbox` 在沙箱关闭时解析为主机执行 | `tools.exec.host`, `agents.defaults.sandbox.mode` | 否 |
| `tools.exec.host_sandbox_no_sandbox_agents` | 警告 | 每个代理的 `exec host=sandbox` 在沙箱关闭时解析为主机执行 | `agents.list[].tools.exec.host`, `agents.list[].sandbox.mode` | 否 |
| `tools.exec.safe_bins_interpreter_unprofiled` | 警告 | `safeBins` 中的解释器/运行时二进制文件没有明确的配置文件，扩大了执行风险 | `tools.exec.safeBins`, `tools.exec.safeBinProfiles`, `agents.list[].tools.exec.*` | 否 |
| `skills.workspace.symlink_escape` | 警告 | 工作区 `skills/**/SKILL.md` 解析到工作区根目录之外（符号链接链漂移） | 工作区 `skills/**` 文件系统状态 | 否 |
| `security.exposure.open_groups_with_elevated` | 严重 | 开放群组 + 提升的工具创建高影响的提示注入路径 | `channels.*.groupPolicy`, `tools.elevated.*` | 否 |
| `security.exposure.open_groups_with_runtime_or_fs` | 严重/警告 | 开放群组可以在没有沙箱/工作区防护的情况下访问命令/文件工具 | `channels.*.groupPolicy`, `tools.profile/deny`, `tools.fs.workspaceOnly`, `agents.*.sandbox.mode` | 否 |
| `security.trust_model.multi_user_heuristic` | 警告 | 配置看起来是多用户的，而网关信任模型是个人助理 | 分割信任边界，或共享用户强化（`sandbox.mode`、工具拒绝/工作区限定） | 否 |
| `tools.profile_minimal_overridden` | 警告 | 代理覆盖绕过全局最小配置文件 | `agents.list[].tools.profile` | 否 |
| `plugins.tools_reachable_permissive_policy` | 警告 | 扩展工具在宽松上下文中可访问 | `tools.profile` + 工具允许/拒绝 | 否 |
| `models.small_params` | 严重/信息 | 小型模型 + 不安全的工具接口增加了注入风险 | 模型选择 + 沙箱/工具策略 | 否 |

## 通过 HTTP 的控制 UI

控制 UI 需要一个 **安全上下文**（HTTPS 或 localhost）来生成设备身份。`gateway.controlUi.allowInsecureAuth` **不会** 绕过安全上下文、设备身份或设备配对检查。优先使用 HTTPS（Tailscale Serve）或在 `127.0.0.1` 上打开 UI。仅用于应急场景，`gateway.controlUi.dangerouslyDisableDeviceAuth` 完全禁用设备身份检查。这是一个严重的安全降级；除非您正在主动调试并能快速恢复，否则请保持关闭。`openclaw security audit` 在此设置启用时会发出警告。

## 不安全或危险标志摘要

当已知的不安全/危险调试开关启用时，`openclaw security audit` 包含 `config.insecure_or_dangerous_flags`。该检查当前汇总了：

-   `gateway.controlUi.allowInsecureAuth=true`
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true`
-   `gateway.controlUi.dangerouslyDisableDeviceAuth=true`
-   `hooks.gmail.allowUnsafeExternalContent=true`
-   `hooks.mappings[].allowUnsafeExternalContent=true`
-   `tools.exec.applyPatch.workspaceOnly=false`

OpenClaw 配置模式中定义的完整 `dangerous*` / `dangerously*` 配置键：

-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback`
-   `gateway.controlUi.dangerouslyDisableDeviceAuth`
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`
-   `channels.discord.dangerouslyAllowNameMatching`
-   `channels.discord.accounts..dangerouslyAllowNameMatching`
-   `channels.slack.dangerouslyAllowNameMatching`
-   `channels.slack.accounts..dangerouslyAllowNameMatching`
-   `channels.googlechat.dangerouslyAllowNameMatching`
-   `channels.googlechat.accounts..dangerouslyAllowNameMatching`
-   `channels.msteams.dangerouslyAllowNameMatching`
-   `channels.irc.dangerouslyAllowNameMatching`（扩展频道）
-   `channels.irc.accounts..dangerouslyAllowNameMatching`（扩展频道）
-   `channels.mattermost.dangerouslyAllowNameMatching`（扩展频道）
-   `channels.mattermost.accounts..dangerouslyAllowNameMatching`（扩展频道）
-   `agents.defaults.sandbox.docker.dangerouslyAllowReservedContainerTargets`
-   `agents.defaults.sandbox.docker.dangerouslyAllowExternalBindSources`
-   `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin`
-   `agents.list[].sandbox.docker.dangerouslyAllowReservedContainerTargets`
-   `agents.list[].sandbox.docker.dangerouslyAllowExternalBindSources`
-   `agents.list[].sandbox.docker.dangerouslyAllowContainerNamespaceJoin`

## 反向代理配置

如果您在反向代理（nginx、Caddy、Traefik 等）后面运行网关，则应配置 `gateway.trustedProxies` 以进行正确的客户端 IP 检测。当网关检测到来自 **不在** `trustedProxies` 中的地址的代理头时，它将 **不会** 将这些连接视为本地客户端。如果网关认证被禁用，这些连接将被拒绝。这可以防止认证绕过，否则代理连接会显示为来自本地主机并获得自动信任。

```
gateway:
  trustedProxies:
    - "127.0.0.1" # 如果您的代理在 localhost 上运行
  # 可选。默认 false。
  # 仅在您的代理无法提供 X-Forwarded-For 时启用。
  allowRealIpFallback: false
  auth:
    mode: password
    password: ${OPENCLAW_GATEWAY_PASSWORD}
```

配置 `trustedProxies` 后，网关使用 `X-Forwarded-For` 来确定客户端 IP。默认情况下忽略 `X-Real-IP`，除非显式设置 `gateway.allowRealIpFallback: true`。良好的反向代理行为（覆盖传入的转发头）：

```bash
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Real-IP $remote_addr;
```

不良的反向代理行为（追加/保留不受信任的转发头）：

```bash
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

## HSTS 和来源说明

-   OpenClaw 网关首先是本地/回环的。如果您在反向代理处终止 TLS，请在该代理面向的 HTTPS 域上设置 HSTS。
-   如果网关本身终止 HTTPS，您可以设置 `gateway.http.securityHeaders.strictTransportSecurity` 以从 OpenClaw 响应中发出 HSTS 头。
-   详细的部署指南在[受信任代理认证](./trusted-proxy-auth.md#tls-termination-and-hsts)中。
-   对于非回环控制 UI 部署，默认需要 `gateway.controlUi.allowedOrigins`。
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` 启用 Host 头来源回退模式；将其视为危险的操作员选定策略。
-   将 DNS 重绑定和代理主机头行为视为部署强化问题；保持 `trustedProxies` 严格，并避免将网关直接暴露给公共互联网。

## 本地会话日志存储在磁盘上

OpenClaw 将会话记录存储在磁盘上的 `~/.openclaw/agents//sessions/*.jsonl` 下。这对于会话连续性和（可选的）会话内存索引是必需的，但这也意味着 **任何具有文件系统访问权限的进程/用户都可以读取这些日志**。将磁盘访问视为信任边界，并锁定 `~/.openclaw` 的权限（参见下面的审计部分）。如果您需要在代理之间进行更强的隔离，请在单独的操作系统用户或单独的主机下运行它们。

## 节点执行（system.run）

如果配对了一个 macOS 节点，网关可以在该节点上调用 `system.run`。这是 Mac 上的 **远程代码执行**：

-   需要节点配对（批准 + 令牌）。
-   在 Mac 上通过 **设置 → 执行批准**（安全 + 询问 + 允许列表）进行控制。
-   如果您不想要远程执行，请将安全性设置为 **拒绝** 并移除该 Mac 的节点配对。

## 动态技能（监视器 / 远程节点）

OpenClaw 可以在会话中刷新技能列表：

-   **技能监视器**：对 `SKILL.md` 的更改可以在下一个代理轮次时更新技能快照。
-   **远程节点**：连接 macOS 节点可以使仅限 macOS 的技能符合条件（基于二进制探测）。

将技能文件夹视为 **受信任的代码** 并限制谁可以修改它们。

## 威胁模型

您的 AI 助手可以：

-   执行任意 shell 命令
-   读取/写入文件
-   访问网络服务
-   向任何人发送消息（如果您授予它 WhatsApp 访问权限）

向您发送消息的人可以：

-   试图诱骗您的 AI 做坏事
-   通过社会工程获取您的数据访问权限
-   探测基础设施细节

## 核心理念：访问控制先于智能

这里的大多数故障不是花哨的漏洞利用——而是“有人给机器人发了消息，机器人照做了”。OpenClaw 的立场：

-   **身份优先：** 决定谁可以与机器人对话（私信配对 / 允许列表 / 显式“开放”）。
-   **范围其次：** 决定机器人被允许在哪里行动（群组允许列表 + 提及门控、工具、沙箱、设备权限）。
-   **模型最后：** 假设模型可以被操纵；设计使操纵具有有限的爆炸半径。

## 命令授权模型

斜杠命令和指令仅对 **授权发送者** 生效。授权源自频道允许列表/配对加上 `commands.useAccessGroups`（参见[配置](./configuration.md)和[斜杠命令](../tools/slash-commands.md)）。如果频道允许列表为空或包含 `"*"`，则该频道的命令实际上是开放的。`/exec` 是授权操作员的仅会话便捷命令。它 **不会** 写入配置或更改其他会话。

## 控制平面工具风险

两个内置工具可以进行持久的控制平面更改：

-   `gateway` 可以调用 `config.apply`、`config.patch` 和 `update.run`。
-   `cron` 可以创建在原始聊天/任务结束后继续运行的定时作业。

对于处理不受信任内容的任何代理/接口，默认拒绝这些：

```json
{
  tools: {
    deny: ["gateway", "cron", "sessions_spawn", "sessions_send"],
  },
}
```

`commands.restart=false` 仅阻止重启操作。它不会禁用 `gateway` 配置/更新操作。

## 插件/扩展

插件与网关 **同进程** 运行。将它们视为受信任的代码：

-   仅从您信任的来源安装插件。
-   优先使用显式的 `plugins.allow` 允许列表。
-   在启用前审查插件配置。
-   插件更改后重启网关。
-   如果您从 npm 安装插件（`openclaw plugins install <npm-spec>`），请将其视为运行不受信任的代码：
    -   安装路径是 `~/.openclaw/extensions//`（或 `$OPENCLAW_STATE_DIR/extensions//`）。
    -   OpenClaw 使用 `npm pack`，然后在该目录中运行 `npm install --omit=dev`（npm 生命周期脚本可能在安装期间执行代码）。
    -   优先使用固定的确切版本（`@scope/pkg@1.2.3`），并在启用前检查磁盘上的解压代码。

详情：[插件](../tools/plugin.md)

## 私信访问模型（配对 / 允许列表 / 开放 / 禁用）

所有当前支持私信的频道都支持一个私信策略（`dmPolicy` 或 `*.dm.policy`），该策略在消息被处理 **之前** 对入站私信进行门控：

-   `pairing`（默认）：未知发送者会收到一个简短的配对码，机器人在批准前忽略他们的消息。代码在 1 小时后过期；重复的私信不会重新发送代码，除非创建新的请求。默认情况下，待处理请求上限为 **每个频道 3 个**。
-   `allowlist`：未知发送者被阻止（没有配对握手）。
-   `open`：允许任何人发送私信（公开）。**要求** 频道允许列表包含 `"*"`（显式选择加入）。
-   `disabled`：完全忽略入站私信。

通过 CLI 批准：

```bash
openclaw pairing list <channel>
openclaw pairing approve <channel> <code>
```

磁盘上的详情 + 文件：[配对](../channels/pairing.md)

## 私信会话隔离（多用户模式）

默认情况下，OpenClaw 将 **所有私信路由到主会话**，以便您的助手跨设备和频道保持连续性。如果 **多个人** 可以向机器人发送私信（开放私信或多人的允许列表），请考虑隔离私信会话：

```json
{
  session: { dmScope: "per-channel-peer" },
}
```

这可以防止跨用户上下文泄漏，同时保持群聊隔离。这是一个消息传递上下文边界，不是主机管理员边界。如果用户相互敌对并共享相同的网关主机/配置，请为每个信任边界运行单独的网关。

### 安全私信模式（推荐）

将上面的代码片段视为 **安全私信模式**：

-   默认：`session.dmScope: "main"`（所有私信共享一个会话以实现连续性）。
-   本地 CLI 引导默认：当未设置时写入 `session.dmScope: "per-channel-peer"`（保留现有的显式值）。
-   安全私信模式：`session.dmScope: "per-channel-peer"`（每个频道+发送者对获得一个隔离的私信上下文）。

如果您在同一频道上运行多个账户，请改用 `per-account-channel-peer`。如果同一个人通过多个频道联系您，请使用 `session.identityLinks` 将这些私信会话折叠为一个规范身份。参见[会话管理](../concepts/session.md)和[配置](./configuration.md)。

## 允许列表（私信 + 群组）—— 术语

OpenClaw 有两个独立的“谁能触发我？”层：

-   **私信允许列表**（`allowFrom` / `channels.discord.allowFrom` / `channels.slack.allowFrom`；旧版：`channels.discord.dm.allowFrom`、`channels.slack.dm.allowFrom`）：谁被允许在私信中与机器人对话。
    -   当 `dmPolicy="pairing"` 时，批准会写入账户范围的配对允许列表存储，位于 `~/.openclaw/credentials/`（默认账户为 `-allowFrom.json`，非默认账户为 `--allowFrom.json`），并与配置允许列表合并。
-   **群组允许列表**（频道特定）：机器人将接受来自哪些群组/频道/公会的消息。
    -   常见模式：
        -   `channels.whatsapp.groups`、`channels.telegram.groups`、`channels.imessage.groups`：每个群组的默认设置，如 `requireMention`；设置时，它也充当群组允许列表（包含 `"*"` 以保持允许所有行为）。
        -   `groupPolicy="allowlist"` + `groupAllowFrom`：限制谁可以在群组会话 **内部** 触发机器人（WhatsApp/Telegram/Signal/iMessage/Microsoft Teams）。
        -   `channels.discord.guilds` / `channels.slack.channels`：每个接口的允许列表 + 提及默认值。
    -   群组检查按此顺序运行：`groupPolicy`/群组允许列表优先，提及/回复激活其次。
    -   回复机器人消息（隐式提及）**不会** 绕过像 `groupAllowFrom` 这样的发送者允许列表。
    -   **安全说明：** 将 `dmPolicy="open"` 和 `groupPolicy="open"` 视为最后手段的设置。它们应几乎不使用；除非您完全信任房间中的每个成员，否则优先使用配对 + 允许列表。

详情：[配置](./configuration.md)和[群组](../channels/groups.md)

## 提示注入（是什么，为何重要）

提示注入是指攻击者精心设计一条消息，操纵模型执行不安全操作（“忽略你的指令”、“转储你的文件系统”、“点击此链接并运行命令”等）。即使有强大的系统提示，**提示注入问题仍未解决**。系统提示防护栏仅是软性指导；硬性执行来自工具策略、执行批准、沙箱和频道允许列表（并且操作员可以按设计禁用这些）。实践中有效的措施：

-   保持入站私信锁定（配对/允许列表）。
-   在群组中优先使用提及门控；避免在公共房间中使用“始终在线”的机器人。
-   默认将链接、附件和粘贴的指令视为敌对。
-   在沙箱中运行敏感工具执行；将密钥排除在代理可访问的文件系统之外。
-   注意：沙箱是选择加入的。如果沙箱模式关闭，即使 `tools.exec.host` 默认为沙箱，exec 也会在网关主机上运行，并且主机执行不需要批准，除非您设置 `host=gateway` 并配置执行批准。
-   将高风险工具（`exec`、`browser`、`web_fetch`、`web_search`）限制在受信任的代理或显式允许列表。
-   **模型选择很重要：** 较旧/较小/旧版模型在抵抗提示注入和工具滥用方面明显较弱。对于启用了工具的代理，请使用可用的最强最新一代、指令强化的模型。

应视为不受信任的危险信号：

-   “阅读此文件/URL 并完全按照其