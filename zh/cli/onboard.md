

  CLI 命令

  
# onboard

交互式引导向导（本地或远程网关设置）。

## 相关指南

-   CLI 引导中心：[引导向导 (CLI)](../start/wizard.md)
-   引导概述：[引导概述](../start/onboarding-overview.md)
-   CLI 引导参考：[CLI 引导参考](../start/wizard-cli-reference.md)
-   CLI 自动化：[CLI 自动化](../start/wizard-cli-automation.md)
-   macOS 引导：[引导 (macOS 应用)](../start/onboarding.md)

## 示例

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url wss://gateway-host:18789
```

对于纯文本私有网络 `ws://` 目标（仅限受信任网络），请在引导进程环境中设置 `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1`。非交互式自定义提供商：

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --secret-input-mode plaintext \
  --custom-compatibility openai
```

`--custom-api-key` 在非交互模式下是可选的。如果省略，引导将检查 `CUSTOM_API_KEY`。将提供商密钥存储为引用而非纯文本：

```bash
openclaw onboard --non-interactive \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

使用 `--secret-input-mode ref` 时，引导会写入基于环境的引用，而非纯文本密钥值。对于基于身份验证配置文件的提供商，这会写入 `keyRef` 条目；对于自定义提供商，这会写入 `models.providers..apiKey` 作为环境引用（例如 `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`）。非交互式 `ref` 模式约定：

-   在引导进程环境中设置提供商环境变量（例如 `OPENAI_API_KEY`）。
-   除非该环境变量也已设置，否则不要传递内联密钥标志（例如 `--openai-api-key`）。
-   如果传递了内联密钥标志但未设置所需的环境变量，引导将快速失败并提供指导。

非交互模式下的网关令牌选项：

-   `--gateway-auth token --gateway-token ` 存储纯文本令牌。
-   `--gateway-auth token --gateway-token-ref-env ` 将 `gateway.auth.token` 存储为环境 SecretRef。
-   `--gateway-token` 和 `--gateway-token-ref-env` 是互斥的。
-   `--gateway-token-ref-env` 要求引导进程环境中存在非空的环境变量。
-   使用 `--install-daemon` 时，当令牌身份验证需要令牌时，SecretRef 管理的网关令牌会被验证，但不会作为已解析的纯文本持久化到监督服务环境元数据中。
-   使用 `--install-daemon` 时，如果令牌模式需要令牌且配置的令牌 SecretRef 未解析，引导将失败关闭并提供修复指导。
-   使用 `--install-daemon` 时，如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password` 且 `gateway.auth.mode` 未设置，引导将阻止安装，直到显式设置模式。

示例：

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN \
  --accept-risk
```

引用模式下的交互式引导行为：

-   出现提示时选择 **使用密钥引用**。
-   然后选择以下任一选项：
    -   环境变量
    -   已配置的密钥提供商（`file` 或 `exec`）
-   引导在保存引用前执行快速预检验证。
    -   如果验证失败，引导会显示错误并允许您重试。

非交互式 Z.AI 端点选择：注意：`--auth-choice zai-api-key` 现在会根据您的密钥自动检测最佳的 Z.AI 端点（优先选择带有 `zai/glm-5` 的通用 API）。如果您特别需要 GLM Coding Plan 端点，请选择 `zai-coding-global` 或 `zai-coding-cn`。

```bash
# 无提示端点选择
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# 其他 Z.AI 端点选择：
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

非交互式 Mistral 示例：

```bash
openclaw onboard --non-interactive \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY"
```

流程说明：

-   `quickstart`：最少的提示，自动生成网关令牌。
-   `manual`：完整的端口/绑定/身份验证提示（`advanced` 的别名）。
-   本地引导 DM 范围行为：[CLI 引导参考](../start/wizard-cli-reference.md#outputs-and-internals)。
-   最快的首次聊天：`openclaw dashboard`（控制 UI，无需通道设置）。
-   自定义提供商：连接任何 OpenAI 或 Anthropic 兼容的端点，包括未列出的托管提供商。使用 Unknown 进行自动检测。

## 常用后续命令

```bash
openclaw configure
openclaw agents add <name>
```

> **ℹ️** `--json` 并不意味着非交互模式。脚本请使用 `--non-interactive`。

[nodes](./nodes.md)[pairing](./pairing.md)

---