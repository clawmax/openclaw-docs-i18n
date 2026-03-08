

  指南

  
# CLI 引导参考

本页面是 `openclaw onboard` 的完整参考。简短指南请参见 [引导向导 (CLI)](./wizard.md)。

## 向导的功能

本地模式（默认）将引导您完成：

-   模型和认证设置（OpenAI Code 订阅 OAuth、Anthropic API 密钥或设置令牌，以及 MiniMax、GLM、Moonshot 和 AI 网关选项）
-   工作区位置和引导文件
-   网关设置（端口、绑定、认证、tailscale）
-   频道和提供商（Telegram、WhatsApp、Discord、Google Chat、Mattermost 插件、Signal）
-   守护进程安装（LaunchAgent 或 systemd 用户单元）
-   健康检查
-   技能设置

远程模式配置此机器以连接到其他位置的网关。

> **ℹ️** 远程模式不会在远程主机上安装或修改任何内容。

## 本地流程详情

### 步骤 1：现有配置检测

-   如果 `~/.openclaw/openclaw.json` 存在，选择保留、修改或重置。
-   重新运行向导不会清除任何内容，除非您明确选择重置（或传递 `--reset`）。
-   CLI `--reset` 默认为 `config+creds+sessions`；使用 `--reset-scope full` 以同时移除工作区。
-   如果配置无效或包含遗留键，向导将停止并要求您在继续之前运行 `openclaw doctor`。
-   重置使用 `trash` 并提供范围选项：
    -   仅配置
    -   配置 + 凭据 + 会话
    -   完全重置（同时移除工作区）

### 步骤 2：模型和认证

-   完整选项矩阵见 [认证和模型选项](#auth-and-model-options)。

### 步骤 3：工作区

-   默认 `~/.openclaw/workspace`（可配置）。
-   为首次运行引导仪式所需的工作区文件进行种子填充。
-   工作区布局：[代理工作区](../concepts/agent-workspace.md)。

### 步骤 4：网关

-   提示输入端口、绑定、认证模式和 tailscale 暴露。
-   推荐：即使对于环回地址也保持令牌认证启用，以便本地 WebSocket 客户端必须进行身份验证。
-   在令牌模式下，交互式引导提供：
    -   **生成/存储明文令牌**（默认）
    -   **使用 SecretRef**（可选）
-   在密码模式下，交互式引导也支持明文或 SecretRef 存储。
-   非交互式令牌 SecretRef 路径：`--gateway-token-ref-env <ENV_VAR>`。
    -   要求引导进程环境中存在非空的环境变量。
    -   不能与 `--gateway-token` 结合使用。
-   仅当您完全信任每个本地进程时才禁用认证。
-   非环回绑定仍然需要认证。

### 步骤 5：频道

-   [WhatsApp](../channels/whatsapp.md)：可选的二维码登录
-   [Telegram](../channels/telegram.md)：机器人令牌
-   [Discord](../channels/discord.md)：机器人令牌
-   [Google Chat](../channels/googlechat.md)：服务账户 JSON + webhook 受众
-   [Mattermost](../channels/mattermost.md) 插件：机器人令牌 + 基础 URL
-   [Signal](../channels/signal.md)：可选的 `signal-cli` 安装 + 账户配置
-   [BlueBubbles](../channels/bluebubbles.md)：推荐用于 iMessage；服务器 URL + 密码 + webhook
-   [iMessage](../channels/imessage.md)：遗留的 `imsg` CLI 路径 + 数据库访问
-   DM 安全：默认是配对。首次 DM 发送一个代码；通过 `openclaw pairing approve  ` 批准或使用允许列表。

### 步骤 6：守护进程安装

-   macOS：LaunchAgent
    -   需要已登录的用户会话；对于无头环境，使用自定义 LaunchDaemon（未提供）。
-   Linux 和通过 WSL2 的 Windows：systemd 用户单元
    -   向导尝试 `loginctl enable-linger ` 以便网关在注销后保持运行。
    -   可能提示需要 sudo（写入 `/var/lib/systemd/linger`）；它会先尝试不使用 sudo。
-   运行时选择：Node（推荐；WhatsApp 和 Telegram 必需）。不推荐 Bun。

### 步骤 7：健康检查

-   启动网关（如果需要）并运行 `openclaw health`。
-   `openclaw status --deep` 在状态输出中添加网关健康探测。

### 步骤 8：技能

-   读取可用技能并检查要求。
-   让您选择节点管理器：npm 或 pnpm（不推荐 bun）。
-   安装可选依赖项（某些在 macOS 上使用 Homebrew）。

### 步骤 9：完成

-   摘要和后续步骤，包括 iOS、Android 和 macOS 应用选项。

 

> **ℹ️** 如果未检测到 GUI，向导将打印用于控制 UI 的 SSH 端口转发说明，而不是打开浏览器。如果控制 UI 资源缺失，向导会尝试构建它们；回退方案是 `pnpm ui:build`（自动安装 UI 依赖项）。

## 远程模式详情

远程模式配置此机器以连接到其他位置的网关。

> **ℹ️** 远程模式不会在远程主机上安装或修改任何内容。

 您需要设置：

-   远程网关 URL (`ws://...`)
-   如果远程网关需要认证（推荐），则提供令牌

> **ℹ️** -   如果网关仅限环回地址，请使用 SSH 隧道或 tailnet。
> -   发现提示：
>     -   macOS：Bonjour (`dns-sd`)
>     -   Linux：Avahi (`avahi-browse`)

## 认证和模型选项

如果存在 `ANTHROPIC_API_KEY` 则使用，否则提示输入密钥，然后保存以供守护进程使用。

-   macOS：检查钥匙串项 “Claude Code-credentials”
-   Linux 和 Windows：如果存在则重用 `~/.claude/.credentials.json`

在 macOS 上，选择“始终允许”以便 launchd 启动时不会阻塞。

在任何机器上运行 `claude setup-token`，然后粘贴令牌。您可以为其命名；留空则使用默认名称。

如果 `~/.codex/auth.json` 存在，向导可以重用。

浏览器流程；粘贴 `code#state`。当模型未设置或为 `openai/*` 时，将 `agents.defaults.model` 设置为 `openai-codex/gpt-5.4`。

如果存在 `OPENAI_API_KEY` 则使用，否则提示输入密钥，然后将凭据存储在认证配置文件中。当模型未设置、为 `openai/*` 或 `openai-codex/*` 时，将 `agents.defaults.model` 设置为 `openai/gpt-5.1-codex`。

提示输入 `XAI_API_KEY` 并将 xAI 配置为模型提供商。

提示输入 `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`）。设置 URL：[opencode.ai/auth](https://opencode.ai/auth)。

为您存储密钥。

提示输入 `AI_GATEWAY_API_KEY`。更多详情：[Vercel AI Gateway](../providers/vercel-ai-gateway.md)。

提示输入账户 ID、网关 ID 和 `CLOUDFLARE_AI_GATEWAY_API_KEY`。更多详情：[Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md)。

配置自动写入。更多详情：[MiniMax](../providers/minimax.md)。

提示输入 `SYNTHETIC_API_KEY`。更多详情：[Synthetic](../providers/synthetic.md)。

Moonshot (Kimi K2) 和 Kimi Coding 配置自动写入。更多详情：[Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md)。

适用于 OpenAI 兼容和 Anthropic 兼容的端点。交互式引导支持与其他提供商 API 密钥流程相同的 API 密钥存储选择：

-   **现在粘贴 API 密钥**（明文）
-   **使用密钥引用**（环境变量引用或配置的提供商引用，带有预检验证）

非交互式标志：

-   `--auth-choice custom-api-key`
-   `--custom-base-url`
-   `--custom-model-id`
-   `--custom-api-key`（可选；回退到 `CUSTOM_API_KEY`）
-   `--custom-provider-id`（可选）
-   `--custom-compatibility <openai|anthropic>`（可选；默认 `openai`）

保持认证未配置。

 模型行为：

-   从检测到的选项中选择默认模型，或手动输入提供商和模型。
-   向导运行模型检查，如果配置的模型未知或缺少认证，则发出警告。

凭据和配置文件路径：

-   OAuth 凭据：`~/.openclaw/credentials/oauth.json`
-   认证配置文件（API 密钥 + OAuth）：`~/.openclaw/agents//agent/auth-profiles.json`

凭据存储模式：

-   默认引导行为将 API 密钥作为明文值保存在认证配置文件中。
-   `--secret-input-mode ref` 启用引用模式，而不是明文密钥存储。在交互式引导中，您可以选择：
    -   环境变量引用（例如 `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`）
    -   配置的提供商引用（`file` 或 `exec`），带有提供商别名 + id
-   交互式引用模式在保存前运行快速预检验证。
    -   环境变量引用：验证当前引导环境中的变量名称 + 非空值。
    -   提供商引用：验证提供商配置并解析请求的 id。
    -   如果预检失败，引导会显示错误并让您重试。
-   在非交互式模式下，`--secret-input-mode ref` 仅支持环境变量。
    -   在引导进程环境中设置提供商环境变量。
    -   内联密钥标志（例如 `--openai-api-key`）要求设置该环境变量；否则引导快速失败。
    -   对于自定义提供商，非交互式 `ref` 模式将 `models.providers..apiKey` 存储为 `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`。
    -   在自定义提供商的情况下，`--custom-api-key` 要求设置 `CUSTOM_API_KEY`；否则引导快速失败。
-   网关认证凭据在交互式引导中支持明文和 SecretRef 选择：
    -   令牌模式：**生成/存储明文令牌**（默认）或**使用 SecretRef**。
    -   密码模式：明文或 SecretRef。
-   非交互式令牌 SecretRef 路径：`--gateway-token-ref-env <ENV_VAR>`。
-   现有的明文设置继续正常工作。

> **ℹ️** 无头服务器提示：在带有浏览器的机器上完成 OAuth，然后将 `~/.openclaw/credentials/oauth.json`（或 `$OPENCLAW_STATE_DIR/credentials/oauth.json`）复制到网关主机。

## 输出和内部机制

`~/.openclaw/openclaw.json` 中的典型字段：

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers`（如果选择了 Minimax）
-   `tools.profile`（本地引导在未设置时默认为 `"coding"`；现有的显式值会被保留）
-   `gateway.*`（模式、绑定、认证、tailscale）
-   `session.dmScope`（本地引导在未设置时默认为 `per-channel-peer`；现有的显式值会被保留）
-   `channels.telegram.botToken`、`channels.discord.token`、`channels.signal.*`、`channels.imessage.*`
-   频道允许列表（Slack、Discord、Matrix、Microsoft Teams），当您在提示中选择加入时（名称在可能的情况下解析为 ID）
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` 写入 `agents.list[]` 和可选的 `bindings`。WhatsApp 凭据存储在 `~/.openclaw/credentials/whatsapp//` 下。会话存储在 `~/.openclaw/agents//sessions/` 下。

> **ℹ️** 某些频道以插件形式提供。在引导过程中选择时，向导会在频道配置之前提示安装插件（npm 或本地路径）。

 网关向导 RPC：

-   `wizard.start`
-   `wizard.next`
-   `wizard.cancel`
-   `wizard.status`

客户端（macOS 应用和控制 UI）可以渲染步骤而无需重新实现引导逻辑。Signal 设置行为：

-   下载相应的发布资源
-   将其存储在 `~/.openclaw/tools/signal-cli//` 下
-   在配置中写入 `channels.signal.cliPath`
-   JVM 构建需要 Java 21
-   原生构建在可用时使用
-   Windows 使用 WSL2 并在 WSL 内遵循 Linux signal-cli 流程

## 相关文档

-   引导中心：[引导向导 (CLI)](./wizard.md)
-   自动化和脚本：[CLI 自动化](./wizard-cli-automation.md)
-   命令参考：[`openclaw onboard`](../cli/onboard.md)

[个人助理设置](./openclaw.md)[CLI 自动化](./wizard-cli-automation.md)