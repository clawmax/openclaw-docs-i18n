

  提供商

  
# Anthropic

Anthropic 开发了 **Claude** 模型系列，并通过 API 对外提供服务。在 OpenClaw 中，你可以通过两种方式完成身份验证：使用 API 密钥或使用 **setup-token**。

## 选项 A：Anthropic API 密钥

**适用场景：** 标准的 API 调用和按量计费。请在 Anthropic 控制台中创建你的 API 密钥。

### CLI 设置

```bash
openclaw onboard
# 选择：Anthropic API key

# 或使用非交互模式
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### 配置示例

```json
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 思考模式默认值（Claude 4.6）

- 当未显式设置思考级别时，OpenClaw 会将 Anthropic Claude 4.6 模型的思考模式默认设为 `adaptive`。
- 你可以针对单条消息进行覆盖（使用 `/think:`），也可以在模型参数中配置：`agents.defaults.models["anthropic/"].params.thinking`。
- 相关 Anthropic 文档：
    - [自适应思考](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
    - [扩展思考](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

## 提示缓存（Anthropic API）

OpenClaw 支持 Anthropic 的提示缓存功能。请注意，此功能**仅适用于 API 密钥认证**，订阅令牌方式不支持缓存设置。

### 配置方法

在模型配置中使用 `cacheRetention` 参数：

| 值 | 缓存时长 | 说明 |
| --- | --- | --- |
| `none` | 不缓存 | 禁用提示缓存 |
| `short` | 5 分钟 | API 密钥认证的默认值 |
| `long` | 1 小时 | 扩展缓存（需要启用 beta 标志） |

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

### 默认行为

使用 Anthropic API 密钥认证时，OpenClaw 会自动为所有 Anthropic 模型应用 `cacheRetention: "short"`（5 分钟缓存）。如需调整，只需在配置中显式设置 `cacheRetention` 即可覆盖默认值。

### 按智能体覆盖缓存设置

你可以先在模型级别设置基线参数，再针对特定智能体通过 `agents.list[].params` 进行覆盖：

```json
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" }, // 大多数智能体的基线设置
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } }, // 仅覆盖此智能体
    ],
  },
}
```

缓存相关参数的配置合并顺序：

1. `agents.defaults.models["provider/model"].params`
2. `agents.list[].params`（根据 `id` 匹配，按键名覆盖）

这样设计的好处是：一个智能体可以保持长期缓存，而使用同一模型的另一个智能体则可以禁用缓存——避免在流量突发或复用率低的场景下产生额外的写入成本。

### Bedrock Claude 说明

- Bedrock 上的 Anthropic Claude 模型（`amazon-bedrock/*anthropic.claude*`）在配置后支持 `cacheRetention` 透传。
- 非 Anthropic 的 Bedrock 模型在运行时会被强制设为 `cacheRetention: "none"`。
- 当未显式设置缓存参数时，Anthropic API 密钥的智能默认值也会为 Claude-on-Bedrock 模型引用自动填充 `cacheRetention: "short"`。

### 旧版参数

旧版 `cacheControlTtl` 参数仍受支持，以确保向后兼容：

- `"5m"` 映射为 `short`
- `"1h"` 映射为 `long`

我们建议迁移到新版 `cacheRetention` 参数。OpenClaw 已为 Anthropic API 请求内置 `extended-cache-ttl-2025-04-11` beta 标志；如果你需要覆盖提供商请求头，请保留此标志（详见 [/gateway/configuration](../gateway/configuration.md)）。

## 100 万上下文窗口（Anthropic beta）

Anthropic 的 100 万上下文窗口目前处于 beta 阶段。在 OpenClaw 中，你可以针对支持的 Opus/Sonnet 模型，通过设置 `params.context1m: true` 来启用此功能。

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

OpenClaw 会将此设置映射为 Anthropic 请求中的 `anthropic-beta: context-1m-2025-08-07`。只有当 `params.context1m` 被显式设置为 `true` 时才会生效。

**前提条件：** Anthropic 必须允许该凭证使用长上下文功能（通常是 API 密钥计费账户，或已启用「额外使用量」的订阅账户）。否则，Anthropic 会返回错误：`HTTP 429: rate_limit_error: Extra usage is required for long context requests`。

**注意：** 当使用 OAuth/订阅令牌（`sk-ant-oat-*`）时，Anthropic 目前会拒绝 `context-1m-*` beta 请求。OpenClaw 会自动为 OAuth 认证跳过 context1m beta 头部，并保留 OAuth 所需的其他 beta 标志。

## 选项 B：Claude setup-token

**适用场景：** 使用你的 Claude 订阅账户。

### 如何获取 setup-token

setup-token 需要通过 **Claude Code CLI** 创建，而不是在 Anthropic 控制台中生成。你可以在**任意机器**上运行：

```bash
claude setup-token
```

将生成的令牌粘贴到 OpenClaw 中（向导选项：**Anthropic token (paste setup-token)**），或直接在网关主机上运行：

```bash
openclaw models auth setup-token --provider anthropic
```

如果你是在其他机器上生成的令牌，可以通过以下命令粘贴：

```bash
openclaw models auth paste-token --provider anthropic
```

### CLI 设置（setup-token）

```bash
# 在引导流程中粘贴 setup-token
openclaw onboard --auth-choice setup-token
```

### 配置示例（setup-token）

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 注意事项

- 使用 `claude setup-token` 生成令牌后粘贴，或直接在网关主机上运行 `openclaw models auth setup-token`。
- 如果在 Claude 订阅中遇到「OAuth token refresh failed …」错误，请使用 setup-token 重新认证。详见 [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](../gateway/troubleshooting.md#oauth-token-refresh-failed-anthropic-claude-subscription)。
- 认证详情和复用规则请参考 [/concepts/oauth](../concepts/oauth.md)。

## 故障排除

**401 错误 / 令牌突然失效**

- Claude 订阅认证可能会过期或被撤销。请重新运行 `claude setup-token`，然后将新令牌粘贴到**网关主机**上。
- 如果 Claude CLI 登录位于另一台机器上，请在网关主机上运行 `openclaw models auth paste-token --provider anthropic`。

**未找到提供商「anthropic」的 API 密钥**

- 认证是**按智能体（agent）独立配置**的。新创建的智能体不会继承主智能体的密钥。
- 请为该智能体重新运行引导流程，或在网关主机上粘贴 setup-token / API 密钥，然后用 `openclaw models status` 验证。

**未找到配置文件 `anthropic:default` 的凭据**

- 运行 `openclaw models status` 查看当前活动的认证配置文件。
- 重新运行引导流程，或为该配置文件粘贴 setup-token / API 密钥。

**无可用认证配置文件（全部处于冷却/不可用状态）**

- 检查 `openclaw models status --json` 输出中的 `auth.unusableProfiles` 字段。
- 添加另一个 Anthropic 配置文件，或等待冷却期结束。

更多信息请参考：[/gateway/troubleshooting](../gateway/troubleshooting.md) 和 [/help/faq](../help/faq.md)。

[模型故障转移](../concepts/model-failover.md)[Amazon Bedrock](./bedrock.md)