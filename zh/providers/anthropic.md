

  提供商

  
# Anthropic

Anthropic 构建了 **Claude** 模型系列，并通过 API 提供访问。在 OpenClaw 中，您可以使用 API 密钥或 **setup-token** 进行身份验证。

## 选项 A: Anthropic API 密钥

**最适合：** 标准 API 访问和基于使用量的计费。在 Anthropic 控制台创建您的 API 密钥。

### CLI 设置

```bash
openclaw onboard
# 选择：Anthropic API key

# 或非交互式
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### 配置片段

```json
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 思考默认值 (Claude 4.6)

-   在 OpenClaw 中，当未设置显式思考级别时，Anthropic Claude 4.6 模型默认使用 `adaptive` 思考。
-   您可以按消息覆盖 (`/think:`) 或在模型参数中覆盖：`agents.defaults.models["anthropic/"].params.thinking`。
-   相关 Anthropic 文档：
    -   [自适应思考](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
    -   [扩展思考](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

## 提示缓存 (Anthropic API)

OpenClaw 支持 Anthropic 的提示缓存功能。这是 **仅限 API** 的；订阅身份验证不遵循缓存设置。

### 配置

在您的模型配置中使用 `cacheRetention` 参数：

| 值 | 缓存持续时间 | 描述 |
| --- | --- | --- |
| `none` | 无缓存 | 禁用提示缓存 |
| `short` | 5 分钟 | API 密钥身份验证的默认值 |
| `long` | 1 小时 | 扩展缓存（需要测试版标志） |

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

### 默认值

当使用 Anthropic API 密钥身份验证时，OpenClaw 会自动为所有 Anthropic 模型应用 `cacheRetention: "short"`（5 分钟缓存）。您可以通过在配置中显式设置 `cacheRetention` 来覆盖此默认值。

### 按代理的 cacheRetention 覆盖

使用模型级参数作为基线，然后通过 `agents.list[].params` 覆盖特定代理。

```json
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" }, // 大多数代理的基线
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } }, // 仅为此代理覆盖
    ],
  },
}
```

缓存相关参数的配置合并顺序：

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params`（匹配 `id`，按键覆盖）

这允许一个代理保持长期缓存，而同一模型上的另一个代理禁用缓存，以避免突发/低复用流量上的写入成本。

### Bedrock Claude 注意事项

-   Bedrock 上的 Anthropic Claude 模型 (`amazon-bedrock/*anthropic.claude*`) 在配置时接受 `cacheRetention` 透传。
-   非 Anthropic 的 Bedrock 模型在运行时被强制设置为 `cacheRetention: "none"`。
-   Anthropic API 密钥智能默认值也会在未设置显式值时，为 Claude-on-Bedrock 模型引用设置 `cacheRetention: "short"`。

### 旧参数

旧的 `cacheControlTtl` 参数仍受支持以保持向后兼容性：

-   `"5m"` 映射到 `short`
-   `"1h"` 映射到 `long`

我们建议迁移到新的 `cacheRetention` 参数。OpenClaw 为 Anthropic API 请求包含 `extended-cache-ttl-2025-04-11` 测试版标志；如果您覆盖了提供商头部，请保留它（参见 [/gateway/configuration](../gateway/configuration.md)）。

## 100 万上下文窗口 (Anthropic 测试版)

Anthropic 的 100 万上下文窗口处于测试版限制中。在 OpenClaw 中，对于支持的 Opus/Sonnet 模型，使用 `params.context1m: true` 按模型启用它。

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { context1m: true },
        },
      },
    },
  },
}
```

OpenClaw 将此映射到 Anthropic 请求上的 `anthropic-beta: context-1m-2025-08-07`。这仅在为该模型显式设置 `params.context1m` 为 `true` 时激活。要求：Anthropic 必须允许该凭证使用长上下文（通常是 API 密钥计费，或启用了额外使用量的订阅账户）。否则 Anthropic 会返回：`HTTP 429: rate_limit_error: Extra usage is required for long context requests`。注意：当使用 OAuth/订阅令牌 (`sk-ant-oat-*`) 时，Anthropic 目前会拒绝 `context-1m-*` 测试版请求。OpenClaw 会自动为 OAuth 身份验证跳过 context1m 测试版头部，并保留必需的 OAuth 测试版。

## 选项 B: Claude setup-token

**最适合：** 使用您的 Claude 订阅。

### 在哪里获取 setup-token

Setup-token 由 **Claude Code CLI** 创建，而非 Anthropic 控制台。您可以在 **任何机器** 上运行此命令：

```bash
claude setup-token
```

将令牌粘贴到 OpenClaw 中（向导：**Anthropic token (paste setup-token)**），或在网关主机上运行：

```bash
openclaw models auth setup-token --provider anthropic
```

如果您在其他机器上生成了令牌，请粘贴它：

```bash
openclaw models auth paste-token --provider anthropic
```

### CLI 设置 (setup-token)

```bash
# 在引导过程中粘贴 setup-token
openclaw onboard --auth-choice setup-token
```

### 配置片段 (setup-token)

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 注意事项

-   使用 `claude setup-token` 生成 setup-token 并粘贴它，或在网关主机上运行 `openclaw models auth setup-token`。
-   如果您在 Claude 订阅上看到“OAuth token refresh failed …”，请使用 setup-token 重新进行身份验证。参见 [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](../gateway/troubleshooting.md#oauth-token-refresh-failed-anthropic-claude-subscription)。
-   身份验证详情 + 重用规则在 [/concepts/oauth](../concepts/oauth.md) 中。

## 故障排除

**401 错误 / 令牌突然失效**

-   Claude 订阅身份验证可能会过期或被撤销。重新运行 `claude setup-token` 并将其粘贴到 **网关主机** 上。
-   如果 Claude CLI 登录位于不同的机器上，请在网关主机上使用 `openclaw models auth paste-token --provider anthropic`。

**未找到提供商“anthropic”的 API 密钥**

-   身份验证是 **按代理** 的。新代理不会继承主代理的密钥。
-   为该代理重新运行引导，或在网关主机上粘贴 setup-token / API 密钥，然后使用 `openclaw models status` 验证。

**未找到配置文件 `anthropic:default` 的凭据**

-   运行 `openclaw models status` 查看哪个身份验证配置文件处于活动状态。
-   重新运行引导，或为该配置文件粘贴 setup-token / API 密钥。

**无可用身份验证配置文件（全部处于冷却/不可用状态）**

-   检查 `openclaw models status --json` 中的 `auth.unusableProfiles`。
-   添加另一个 Anthropic 配置文件或等待冷却结束。

更多信息：[/gateway/troubleshooting](../gateway/troubleshooting.md) 和 [/help/faq](../help/faq.md)。

[模型故障转移](../concepts/model-failover.md)[Amazon Bedrock](./bedrock.md)