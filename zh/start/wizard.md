

  第一步

  
# 入门向导 (CLI)

入门向导是在 macOS、Linux 或 Windows（通过 WSL2；强烈推荐）上设置 OpenClaw 的**推荐**方式。它在一个引导式流程中配置本地网关或远程网关连接，以及频道、技能和工作区默认值。

```bash
openclaw onboard
```

> **ℹ️** 最快的首次聊天：打开控制界面（无需设置频道）。运行 `openclaw dashboard` 并在浏览器中聊天。文档：[控制面板](../web/dashboard.md)。

 后续重新配置：

```bash
openclaw configure
openclaw agents add <name>
```

> **ℹ️** `--json` 不意味着非交互模式。对于脚本，请使用 `--non-interactive`。

 

> **💡** 入门向导包含一个网络搜索步骤，您可以在其中选择提供商（Perplexity、Brave、Gemini、Grok 或 Kimi）并粘贴您的 API 密钥，以便代理可以使用 `web_search`。您也可以稍后使用 `openclaw configure --section web` 进行配置。文档：[网络工具](../tools/web.md)。

## 快速开始 vs 高级模式

向导以**快速开始**（默认值）与**高级**（完全控制）开始。

-   本地网关（环回）
-   工作区默认值（或现有工作区）
-   网关端口 **18789**
-   网关认证 **令牌**（自动生成，即使在环回地址上）
-   新本地设置的工具策略默认值：`tools.profile: "coding"`（现有的显式配置文件将被保留）
-   DM 隔离默认值：本地入门在未设置时写入 `session.dmScope: "per-channel-peer"`。详情：[CLI 入门参考](./wizard-cli-reference.md#outputs-and-internals)
-   Tailscale 暴露 **关闭**
-   Telegram 和 WhatsApp 私信默认使用 **允许列表**（系统将提示您输入电话号码）

-   暴露每个步骤（模式、工作区、网关、频道、守护进程、技能）。

## 向导配置的内容

**本地模式（默认）** 引导您完成以下步骤：

1.  **模型/认证** — 选择任何支持的提供商/认证流程（API 密钥、OAuth 或设置令牌），包括自定义提供商（OpenAI 兼容、Anthropic 兼容或未知自动检测）。选择一个默认模型。安全提示：如果此代理将运行工具或处理 webhook/hooks 内容，请优先选择可用的最强最新一代模型，并保持工具策略严格。较弱/较旧的层级更容易受到提示注入攻击。对于非交互式运行，`--secret-input-mode ref` 会在认证配置文件中存储环境变量支持的引用，而不是明文 API 密钥值。在非交互式 `ref` 模式下，必须设置提供商环境变量；在没有该环境变量的情况下传递内联密钥标志会快速失败。在交互式运行中，选择密钥引用模式可以让您指向环境变量或已配置的提供商引用（`file` 或 `exec`），并在保存前进行快速预检验证。
2.  **工作区** — 代理文件的位置（默认 `~/.openclaw/workspace`）。植入引导文件。
3.  **网关** — 端口、绑定地址、认证模式、Tailscale 暴露。在交互式令牌模式下，选择默认的明文令牌存储或选择使用 SecretRef。非交互式令牌 SecretRef 路径：`--gateway-token-ref-env <ENV_VAR>`。
4.  **频道** — WhatsApp、Telegram、Discord、Google Chat、Mattermost、Signal、BlueBubbles 或 iMessage。
5.  **守护进程** — 安装 LaunchAgent（macOS）或 systemd 用户单元（Linux/WSL2）。如果令牌认证需要令牌且 `gateway.auth.token` 由 SecretRef 管理，守护进程安装会验证它，但不会将解析后的令牌持久化到监督服务环境元数据中。如果令牌认证需要令牌且配置的令牌 SecretRef 未解析，守护进程安装将被阻止并给出可操作的指导。如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password` 且 `gateway.auth.mode` 未设置，守护进程安装将被阻止，直到显式设置模式。
6.  **健康检查** — 启动网关并验证其正在运行。
7.  **技能** — 安装推荐的技能和可选依赖项。

> **ℹ️** 重新运行向导**不会**清除任何内容，除非您明确选择**重置**（或传递 `--reset`）。CLI `--reset` 默认重置配置、凭据和会话；使用 `--reset-scope full` 以包含工作区。如果配置无效或包含遗留键，向导会要求您先运行 `openclaw doctor`。

 **远程模式** 仅配置本地客户端以连接到其他位置的网关。它**不会**在远程主机上安装或更改任何内容。

## 添加另一个代理

使用 `openclaw agents add ` 创建一个具有独立工作区、会话和认证配置文件的代理。不带 `--workspace` 运行将启动向导。它设置的内容：

-   `agents.list[].name`
-   `agents.list[].workspace`
-   `agents.list[].agentDir`

注意：

-   默认工作区遵循 `~/.openclaw/workspace-`。
-   添加 `bindings` 以路由入站消息（向导可以完成此操作）。
-   非交互式标志：`--model`、`--agent-dir`、`--bind`、`--non-interactive`。

## 完整参考

有关详细的逐步分解、非交互式脚本编写、Signal 设置、RPC API 以及向导写入的配置字段完整列表，请参阅[向导参考](../reference/wizard.md)。

## 相关文档

-   CLI 命令参考：[`openclaw onboard`](../cli/onboard.md)
-   入门概述：[入门概述](./onboarding-overview.md)
-   macOS 应用入门：[入门](./onboarding.md)
-   代理首次运行流程：[代理引导](./bootstrapping.md)

[入门概述](./onboarding-overview.md)[入门：macOS 应用](./onboarding.md)

---