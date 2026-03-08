

  配置与运维

  
# Doctor

`openclaw doctor` 是 OpenClaw 的修复与迁移工具。它能修复过时的配置/状态、检查健康状况，并提供可操作的修复步骤。

## 快速开始

```bash
openclaw doctor
```

### 无头模式 / 自动化

```bash
openclaw doctor --yes
```

在不提示的情况下接受默认选项（包括在适用时重启/服务/沙箱修复步骤）。

```bash
openclaw doctor --repair
```

在不提示的情况下应用推荐的修复（包括安全的修复和重启操作）。

```bash
openclaw doctor --repair --force
```

同时应用激进修复（会覆盖自定义的守护进程配置）。

```bash
openclaw doctor --non-interactive
```

无提示运行，仅应用安全的迁移（配置规范化 + 磁盘状态移动）。跳过需要人工确认的重启/服务/沙箱操作。检测到遗留状态迁移时会自动运行。

```bash
openclaw doctor --deep
```

扫描系统服务以查找额外的网关安装（launchd/systemd/schtasks）。如果你想在写入前查看更改，请先打开配置文件：

```bash
cat ~/.openclaw/openclaw.json
```

## 功能概述

-   针对 Git 安装的可选预检更新（仅交互模式）。
-   UI 协议新鲜度检查（当协议模式更新时重建控制界面）。
-   健康检查 + 重启提示。
-   技能状态摘要（可用/缺失/被阻止）。
-   遗留值的配置规范化。
-   OpenCode Zen 提供程序覆盖警告 (`models.providers.opencode`)。
-   遗留磁盘状态迁移（会话/代理目录/WhatsApp 认证）。
-   状态完整性和权限检查（会话、转录、状态目录）。
-   本地运行时配置文件权限检查（chmod 600）。
-   模型认证健康：检查 OAuth 过期情况，可刷新即将过期的令牌，并报告认证配置文件的冷却/禁用状态。
-   额外工作空间目录检测 (`~/openclaw`)。
-   启用沙箱时的沙箱镜像修复。
-   遗留服务迁移和额外网关检测。
-   网关运行时检查（服务已安装但未运行；缓存的 launchd 标签）。
-   通道状态警告（从正在运行的网关探测）。
-   守护进程配置审计（launchd/systemd/schtasks）及可选修复。
-   网关运行时最佳实践检查（Node 与 Bun，版本管理器路径）。
-   网关端口冲突诊断（默认 `18789`）。
-   开放 DM 策略的安全警告。
-   本地令牌模式的网关认证检查（当不存在令牌源时提供令牌生成选项；不会覆盖令牌 SecretRef 配置）。
-   Linux 上的 systemd linger 检查。
-   源码安装检查（pnpm 工作空间不匹配、缺少 UI 资源、缺少 tsx 二进制文件）。
-   写入更新后的配置 + 向导元数据。

## 详细行为与原理

### 0) 可选更新（Git 安装）

如果这是一个 Git 检出目录且 doctor 以交互模式运行，它会在运行 doctor 之前提供更新（fetch/rebase/build）选项。

### 1) 配置规范化

如果配置包含遗留值结构（例如，没有通道特定覆盖的 `messages.ackReaction`），doctor 会将它们规范化为当前模式。

### 2) 遗留配置键迁移

当配置包含已弃用的键时，其他命令会拒绝运行并提示你运行 `openclaw doctor`。Doctor 将：

-   解释发现了哪些遗留键。
-   显示其应用的迁移。
-   使用更新后的模式重写 `~/.openclaw/openclaw.json`。

网关在启动时检测到遗留配置格式时也会自动运行 doctor 迁移，因此过时的配置无需手动干预即可修复。当前迁移包括：

-   `routing.allowFrom` → `channels.whatsapp.allowFrom`
-   `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
-   `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
-   `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
-   `routing.queue` → `messages.queue`
-   `routing.bindings` → 顶级 `bindings`
-   `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
-   `routing.agentToAgent` → `tools.agentToAgent`
-   `routing.transcribeAudio` → `tools.media.audio.models`
-   `bindings[].match.accountID` → `bindings[].match.accountId`
-   对于具有命名 `accounts` 但缺少 `accounts.default` 的通道，将账户范围的顶级单账户通道值移动到 `channels..accounts.default`（如果存在）
-   `identity` → `agents.list[].identity`
-   `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
-   `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks` → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`
-   `browser.ssrfPolicy.allowPrivateNetwork` → `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`

Doctor 警告还包括针对多账户通道的账户默认指导：

-   如果配置了两个或更多 `channels..accounts` 条目，但没有设置 `channels..defaultAccount` 或 `accounts.default`，doctor 会警告回退路由可能选择意外的账户。
-   如果 `channels..defaultAccount` 设置为未知的账户 ID，doctor 会警告并列出已配置的账户 ID。

### 2b) OpenCode Zen 提供程序覆盖

如果你手动添加了 `models.providers.opencode`（或 `opencode-zen`），它会覆盖来自 `@mariozechner/pi-ai` 的内置 OpenCode Zen 目录。这可能导致所有模型都强制使用单一 API 或成本归零。Doctor 会发出警告，以便你可以移除覆盖并恢复按模型的 API 路由和成本计算。

### 3) 遗留状态迁移（磁盘布局）

Doctor 可以将较旧的磁盘布局迁移到当前结构：

-   会话存储 + 转录：
    -   从 `~/.openclaw/sessions/` 迁移到 `~/.openclaw/agents//sessions/`
-   代理目录：
    -   从 `~/.openclaw/agent/` 迁移到 `~/.openclaw/agents//agent/`
-   WhatsApp 认证状态 (Baileys)：
    -   从遗留的 `~/.openclaw/credentials/*.json`（`oauth.json` 除外）
    -   迁移到 `~/.openclaw/credentials/whatsapp//...`（默认账户 ID：`default`）

这些迁移是尽力而为且幂等的；当 doctor 留下任何遗留文件夹作为备份时，它会发出警告。网关/CLI 也会在启动时自动迁移遗留会话和代理目录，因此历史记录/认证/模型会进入每个代理的路径，无需手动运行 doctor。WhatsApp 认证特意仅通过 `openclaw doctor` 迁移。

### 4) 状态完整性检查（会话持久性、路由和安全性）

状态目录是操作的核心。如果它消失，你将丢失会话、凭证、日志和配置（除非你在其他地方有备份）。Doctor 检查：

-   **状态目录缺失**：警告灾难性的状态丢失，提示重新创建目录，并提醒无法恢复丢失的数据。
-   **状态目录权限**：验证可写性；提供修复权限的选项（并在检测到所有者/组不匹配时发出 `chown` 提示）。
-   **macOS 云同步状态目录**：当状态目录位于 iCloud Drive (`~/Library/Mobile Documents/com~apple~CloudDocs/...`) 或 `~/Library/CloudStorage/...` 下时发出警告，因为同步支持的路径可能导致较慢的 I/O 和锁/同步竞争。
-   **Linux SD 或 eMMC 状态目录**：当状态目录解析到 `mmcblk*` 挂载源时发出警告，因为 SD 或 eMMC 支持的随机 I/O 在会话和凭证写入下可能较慢且磨损更快。
-   **会话目录缺失**：`sessions/` 和会话存储目录是持久化历史和避免 `ENOENT` 崩溃所必需的。
-   **转录不匹配**：当最近的会话条目缺少转录文件时发出警告。
-   **主会话“单行 JSONL”**：标记主转录文件只有一行的情况（历史记录未累积）。
-   **多个状态目录**：当多个 `~/.openclaw` 文件夹存在于不同主目录下，或当 `OPENCLAW_STATE_DIR` 指向其他地方时发出警告（历史记录可能在安装之间分裂）。
-   **远程模式提醒**：如果 `gateway.mode=remote`，doctor 会提醒你在远程主机上运行它（状态位于那里）。
-   **配置文件权限**：如果 `~/.openclaw/openclaw.json` 对组/其他用户可读，则发出警告并提供收紧至 `600` 的选项。

### 5) 模型认证健康（OAuth 过期）

Doctor 检查认证存储中的 OAuth 配置文件，在令牌即将过期/已过期时发出警告，并可在安全时刷新它们。如果 Anthropic Claude Code 配置文件已过时，它会建议运行 `claude setup-token`（或粘贴设置令牌）。刷新提示仅在交互式（TTY）运行时出现；`--non-interactive` 会跳过刷新尝试。Doctor 还会报告因以下原因暂时不可用的认证配置文件：

-   短期冷却（速率限制/超时/认证失败）
-   长期禁用（计费/信用失败）

### 6) Hooks 模型验证

如果设置了 `hooks.gmail.model`，doctor 会根据目录和允许列表验证模型引用，并在无法解析或被禁止时发出警告。

### 7) 沙箱镜像修复

启用沙箱时，doctor 检查 Docker 镜像，并在当前镜像缺失时提供构建或切换到遗留名称的选项。

### 8) 网关服务迁移和清理提示

Doctor 检测遗留的网关服务（launchd/systemd/schtasks），并提供移除它们并使用当前网关端口安装 OpenClaw 服务的选项。它还可以扫描额外的类似网关的服务并打印清理提示。按配置文件命名的 OpenClaw 网关服务被视为一等公民，不会被标记为“额外”。

### 9) 安全警告

当提供程序对 DM 开放且没有允许列表，或策略配置方式危险时，Doctor 会发出警告。

### 10) systemd linger (Linux)

如果作为 systemd 用户服务运行，doctor 确保启用了 lingering，以便网关在注销后保持活动状态。

### 11) 技能状态

Doctor 为当前工作空间打印一份可用/缺失/被阻止技能的快速摘要。

### 12) 网关认证检查（本地令牌）

Doctor 检查本地网关令牌认证准备情况。

-   如果令牌模式需要令牌且不存在令牌源，doctor 提供生成令牌的选项。
-   如果 `gateway.auth.token` 由 SecretRef 管理但不可用，doctor 会警告并且不会用明文覆盖它。
-   `openclaw doctor --generate-gateway-token` 仅在未配置令牌 SecretRef 时强制生成令牌。

### 12b) 只读 SecretRef 感知的修复

某些修复流程需要在不削弱运行时快速失败行为的情况下检查已配置的凭证。

-   `openclaw doctor --fix` 现在使用与状态系列命令相同的只读 SecretRef 摘要模型进行针对性配置修复。
-   示例：Telegram `allowFrom` / `groupAllowFrom` `@username` 修复尝试在可用时使用已配置的机器人凭证。
-   如果 Telegram 机器人令牌通过 SecretRef 配置但在当前命令路径中不可用，doctor 会报告凭证已配置但不可用，并跳过自动解析，而不是崩溃或错误地报告令牌缺失。

### 13) 网关健康检查 + 重启

Doctor 运行健康检查，并在网关看起来不健康时提供重启选项。

### 14) 通道状态警告

如果网关健康，doctor 运行通道状态探测，并报告带有建议修复方案的警告。

### 15) 守护进程配置审计 + 修复

Doctor 检查已安装的守护进程配置（launchd/systemd/schtasks）是否存在缺失或过时的默认值（例如，systemd 的 network-online 依赖项和重启延迟）。当发现不匹配时，它会建议更新，并可以将服务文件/任务重写为当前默认值。注意：

-   `openclaw doctor` 在重写守护进程配置前会提示。
-   `openclaw doctor --yes` 接受默认的修复提示。
-   `openclaw doctor --repair` 在不提示的情况下应用推荐的修复。
-   `openclaw doctor --repair --force` 覆盖自定义的守护进程配置。
-   如果令牌认证需要令牌且 `gateway.auth.token` 由 SecretRef 管理，doctor 服务安装/修复会验证 SecretRef，但不会将解析出的明文令牌值持久化到守护进程服务环境元数据中。
-   如果令牌认证需要令牌且配置的令牌 SecretRef 未解析，doctor 会阻止安装/修复路径并提供可操作的指导。
-   如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password` 且 `gateway.auth.mode` 未设置，doctor 会阻止安装/修复，直到明确设置模式。
-   你始终可以通过 `openclaw gateway install --force` 强制完全重写。

### 16) 网关运行时 + 端口诊断

Doctor 检查服务运行时（PID，上次退出状态），并在服务已安装但实际未运行时发出警告。它还会检查网关端口（默认 `18789`）上的端口冲突，并报告可能的原因（网关已在运行，SSH 隧道）。

### 17) 网关运行时最佳实践

Doctor 在网关服务运行在 Bun 或版本管理的 Node 路径（`nvm`、`fnm`、`volta`、`asdf` 等）上时发出警告。WhatsApp + Telegram 通道需要 Node，并且版本管理器路径在升级后可能中断，因为服务不会加载你的 shell 初始化脚本。当系统 Node 安装（Homebrew/apt/choco）可用时，doctor 提供迁移选项。

### 18) 配置写入 + 向导元数据

Doctor 持久化任何配置更改，并标记向导元数据以记录 doctor 运行。

### 19) 工作空间提示（备份 + 记忆系统）

当缺失时，Doctor 建议工作空间记忆系统，如果工作空间尚未置于 git 下，则打印备份提示。有关工作空间结构和 git 备份（推荐私有 GitHub 或 GitLab）的完整指南，请参阅 [/concepts/agent-workspace](../concepts/agent-workspace.md)。

[心跳](./heartbeat.md)[日志记录](./logging.md)