

  技术参考

  
# 入门向导参考

这是 `openclaw onboard` CLI 向导的完整参考。有关高级概述，请参阅 [入门向导](../start/wizard.md)。

## 流程详情（本地模式）

### 步骤 1：现有配置检测

-   如果 `~/.openclaw/openclaw.json` 存在，选择 **保留 / 修改 / 重置**。
-   重新运行向导**不会**清除任何内容，除非您明确选择**重置**（或传递 `--reset`）。
-   CLI `--reset` 默认为 `config+creds+sessions`；使用 `--reset-scope full` 以同时移除工作区。
-   如果配置无效或包含遗留键，向导将停止并要求您在继续之前运行 `openclaw doctor`。
-   重置使用 `trash`（从不使用 `rm`）并提供范围选项：
    -   仅配置
    -   配置 + 凭据 + 会话
    -   完全重置（同时移除工作区）

### 步骤 2：模型/身份验证

-   **Anthropic API 密钥**：如果存在则使用 `ANTHROPIC_API_KEY`，否则提示输入密钥，然后保存以供守护进程使用。
-   **Anthropic OAuth (Claude Code CLI)**：在 macOS 上，向导检查钥匙串项“Claude Code-credentials”（选择“始终允许”以便 launchd 启动时不会阻塞）；在 Linux/Windows 上，如果存在则重用 `~/.claude/.credentials.json`。
-   **Anthropic 令牌（粘贴 setup-token）**：在任何机器上运行 `claude setup-token`，然后粘贴令牌（您可以为其命名；空白 = 默认）。
-   **OpenAI Code (Codex) 订阅 (Codex CLI)**：如果 `~/.codex/auth.json` 存在，向导可以重用。
-   **OpenAI Code (Codex) 订阅 (OAuth)**：浏览器流程；粘贴 `code#state`。
    -   当模型未设置或为 `openai/*` 时，将 `agents.defaults.model` 设置为 `openai-codex/gpt-5.2`。
-   **OpenAI API 密钥**：如果存在则使用 `OPENAI_API_KEY`，否则提示输入密钥，然后将其存储在身份验证配置文件中。
-   **xAI (Grok) API 密钥**：提示输入 `XAI_API_KEY` 并将 xAI 配置为模型提供商。
-   **OpenCode Zen (多模型代理)**：提示输入 `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`，在 [https://opencode.ai/auth](https://opencode.ai/auth) 获取）。
-   **API 密钥**：为您存储密钥。
-   **Vercel AI Gateway (多模型代理)**：提示输入 `AI_GATEWAY_API_KEY`。
-   更多详情：[Vercel AI Gateway](../providers/vercel-ai-gateway.md)
-   **Cloudflare AI Gateway**：提示输入账户 ID、网关 ID 和 `CLOUDFLARE_AI_GATEWAY_API_KEY`。
-   更多详情：[Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md)
-   **MiniMax M2.5**：配置自动写入。
-   更多详情：[MiniMax](../providers/minimax.md)
-   **Synthetic (Anthropic 兼容)**：提示输入 `SYNTHETIC_API_KEY`。
-   更多详情：[Synthetic](../providers/synthetic.md)
-   **Moonshot (Kimi K2)**：配置自动写入。
-   **Kimi Coding**：配置自动写入。
-   更多详情：[Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md)
-   **跳过**：暂不配置身份验证。
-   从检测到的选项中选择一个默认模型（或手动输入提供商/模型）。为了获得最佳质量并降低提示注入风险，请选择您的提供商堆栈中可用的最强最新一代模型。
-   向导运行模型检查，如果配置的模型未知或缺少身份验证，则会发出警告。
-   API 密钥存储模式默认为纯文本身份验证配置文件值。使用 `--secret-input-mode ref` 改为存储基于环境变量的引用（例如 `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`）。
-   OAuth 凭据位于 `~/.openclaw/credentials/oauth.json`；身份验证配置文件位于 `~/.openclaw/agents//agent/auth-profiles.json`（API 密钥 + OAuth）。
-   更多详情：[/concepts/oauth](../concepts/oauth.md)

> **ℹ️** 无头/服务器提示：在具有浏览器的机器上完成 OAuth，然后将 `~/.openclaw/credentials/oauth.json`（或 `$OPENCLAW_STATE_DIR/credentials/oauth.json`）复制到网关主机。

### 步骤 3：工作区

-   默认 `~/.openclaw/workspace`（可配置）。
-   为代理引导仪式植入所需的工作区文件。
-   完整的工作区布局 + 备份指南：[代理工作区](../concepts/agent-workspace.md)

### 步骤 4：网关

-   端口、绑定、身份验证模式、Tailscale 暴露。
-   身份验证建议：即使对于环回地址也保持**令牌**模式，以便本地 WebSocket 客户端必须进行身份验证。
-   在令牌模式下，交互式入门提供：
    -   **生成/存储纯文本令牌**（默认）
    -   **使用 SecretRef**（可选）
    -   Quickstart 在入门探测/仪表板引导过程中跨 `env`、`file` 和 `exec` 提供商重用现有的 `gateway.auth.token` SecretRefs。
    -   如果该 SecretRef 已配置但无法解析，入门将提前失败并显示清晰的修复消息，而不是在运行时静默降级身份验证。
-   在密码模式下，交互式入门也支持纯文本或 SecretRef 存储。
-   非交互式令牌 SecretRef 路径：`--gateway-token-ref-env <ENV_VAR>`。
    -   要求入门进程环境中存在非空的环境变量。
    -   不能与 `--gateway-token` 结合使用。
-   仅当您完全信任每个本地进程时才禁用身份验证。
-   非环回绑定仍然需要身份验证。

### 步骤 5：频道

-   [WhatsApp](../channels/whatsapp.md)：可选的二维码登录。
-   [Telegram](../channels/telegram.md)：机器人令牌。
-   [Discord](../channels/discord.md)：机器人令牌。
-   [Google Chat](../channels/googlechat.md)：服务账户 JSON + webhook 受众。
-   [Mattermost](../channels/mattermost.md)（插件）：机器人令牌 + 基础 URL。
-   [Signal](../channels/signal.md)：可选的 `signal-cli` 安装 + 账户配置。
-   [BlueBubbles](../channels/bluebubbles.md)：**推荐用于 iMessage**；服务器 URL + 密码 + webhook。
-   [iMessage](../channels/imessage.md)：遗留的 `imsg` CLI 路径 + 数据库访问。
-   私信安全：默认是配对。首次私信发送一个代码；通过 `openclaw pairing approve  ` 批准或使用允许列表。

### 步骤 6：网络搜索

-   选择一个提供商：Perplexity、Brave、Gemini、Grok 或 Kimi（或跳过）。
-   粘贴您的 API 密钥（QuickStart 自动从环境变量或现有配置中检测密钥）。
-   使用 `--skip-search` 跳过。
-   稍后配置：`openclaw configure --section web`。

### 步骤 7：守护进程安装

-   macOS：LaunchAgent
    -   需要已登录的用户会话；对于无头模式，使用自定义 LaunchDaemon（未提供）。
-   Linux（以及通过 WSL2 的 Windows）：systemd 用户单元
    -   向导尝试通过 `loginctl enable-linger ` 启用 lingering，以便网关在注销后保持运行。
    -   可能会提示输入 sudo（写入 `/var/lib/systemd/linger`）；它首先尝试不使用 sudo。
-   **运行时选择：** Node（推荐；WhatsApp/Telegram 必需）。Bun **不推荐**。
-   如果令牌身份验证需要令牌且 `gateway.auth.token` 由 SecretRef 管理，守护进程安装会验证它，但不会将解析后的纯文本令牌值持久化到监督服务环境元数据中。
-   如果令牌身份验证需要令牌且配置的令牌 SecretRef 未解析，守护进程安装将被阻止并给出可操作的指导。
-   如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password` 且 `gateway.auth.mode` 未设置，守护进程安装将被阻止，直到明确设置模式。

### 步骤 8：健康检查

-   启动网关（如果需要）并运行 `openclaw health`。
-   提示：`openclaw status --deep` 将网关健康探测添加到状态输出中（需要可访问的网关）。

### 步骤 9：技能（推荐）

-   读取可用技能并检查要求。
-   让您选择一个节点管理器：**npm / pnpm**（不推荐 bun）。
-   安装可选依赖项（某些在 macOS 上使用 Homebrew）。

### 步骤 10：完成

-   摘要 + 后续步骤，包括用于额外功能的 iOS/Android/macOS 应用程序。

 

> **ℹ️** 如果未检测到 GUI，向导将打印用于控制 UI 的 SSH 端口转发说明，而不是打开浏览器。如果控制 UI 资源缺失，向导将尝试构建它们；回退方案是 `pnpm ui:build`（自动安装 UI 依赖项）。

## 非交互模式

使用 `--non-interactive` 来自动化或编写入门脚本：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

添加 `--json` 以获取机器可读的摘要。非交互模式下的网关令牌 SecretRef：

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN
```

`--gateway-token` 和 `--gateway-token-ref-env` 是互斥的。

> **ℹ️** `--json` **不**意味着非交互模式。对于脚本，请使用 `--non-interactive`（和 `--workspace`）。

 

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

### 添加代理（非交互式）

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## 网关向导 RPC

网关通过 RPC（`wizard.start`、`wizard.next`、`wizard.cancel`、`wizard.status`）公开向导流程。客户端（macOS 应用程序、控制 UI）可以渲染步骤而无需重新实现入门逻辑。

## Signal 设置 (signal-cli)

向导可以从 GitHub 发布版安装 `signal-cli`：

-   下载相应的发布资产。
-   将其存储在 `~/.openclaw/tools/signal-cli//` 下。
-   将 `channels.signal.cliPath` 写入您的配置。

注意：

-   JVM 构建需要 **Java 21**。
-   当可用时使用原生构建。
-   Windows 使用 WSL2；signal-cli 安装遵循 WSL 内的 Linux 流程。

## 向导写入的内容

`~/.openclaw/openclaw.json` 中的典型字段：

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers`（如果选择了 Minimax）
-   `tools.profile`（本地入门在未设置时默认为 `"coding"`；现有的显式值将被保留）
-   `gateway.*`（模式、绑定、身份验证、tailscale）
-   `session.dmScope`（行为详情：[CLI 入门参考](../start/wizard-cli-reference.md#outputs-and-internals)）
-   `channels.telegram.botToken`、`channels.discord.token`、`channels.signal.*`、`channels.imessage.*`
-   在提示期间选择加入时的频道允许列表（Slack/Discord/Matrix/Microsoft Teams）（名称在可能时解析为 ID）。
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` 写入 `agents.list[]` 和可选的 `bindings`。WhatsApp 凭据位于 `~/.openclaw/credentials/whatsapp//` 下。会话存储在 `~/.openclaw/agents//sessions/` 下。某些频道作为插件交付。当您在入门过程中选择一个时，向导将提示在配置之前安装它（npm 或本地路径）。

## 相关文档

-   向导概述：[入门向导](../start/wizard.md)
-   macOS 应用程序入门：[入门](../start/onboarding.md)
-   配置参考：[网关配置](../gateway/configuration.md)
-   提供商：[WhatsApp](../channels/whatsapp.md)、[Telegram](../channels/telegram.md)、[Discord](../channels/discord.md)、[Google Chat](../channels/googlechat.md)、[Signal](../channels/signal.md)、[BlueBubbles](../channels/bluebubbles.md) (iMessage)、[iMessage](../channels/imessage.md) (遗留)
-   技能：[技能](../tools/skills.md)、[技能配置](../tools/skills-config.md)

[USER](./templates/USER.md)[令牌使用与成本](./token-use.md)