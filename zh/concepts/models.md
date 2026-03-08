

  模型概念

  
# 模型 CLI

关于认证配置文件轮换、冷却时间及其与回退方案的交互，请参阅 [/concepts/model-failover](./model-failover.md)。提供商快速概览及示例：[/concepts/model-providers](./model-providers.md)。

## 模型选择工作原理

OpenClaw 按以下顺序选择模型：

1.  **主**模型 (`agents.defaults.model.primary` 或 `agents.defaults.model`)。
2.  `agents.defaults.model.fallbacks` 中的**回退**模型（按顺序）。
3.  **提供商认证故障转移**发生在切换到下一个模型之前，在提供商内部进行。

相关内容：

-   `agents.defaults.models` 是 OpenClaw 可以使用的模型（及别名）的允许列表/目录。
-   `agents.defaults.imageModel` **仅当**主模型无法接受图像时使用。
-   每个代理的默认设置可以通过 `agents.list[].model` 以及绑定来覆盖 `agents.defaults.model`（参见 [/concepts/multi-agent](./multi-agent.md)）。

## 快速模型策略

-   将您的主模型设置为对您可用的、性能最强的、最新一代的模型。
-   对于成本/延迟敏感的任务和较低风险的聊天，使用回退模型。
-   对于启用工具的代理或不受信任的输入，避免使用较旧/较弱的模型层级。

## 设置向导（推荐）

如果您不想手动编辑配置，请运行入门向导：

```bash
openclaw onboard
```

它可以为常见提供商设置模型和认证，包括 **OpenAI Code (Codex) 订阅** (OAuth) 和 **Anthropic** (API 密钥或 `claude setup-token`)。

## 配置键（概述）

-   `agents.defaults.model.primary` 和 `agents.defaults.model.fallbacks`
-   `agents.defaults.imageModel.primary` 和 `agents.defaults.imageModel.fallbacks`
-   `agents.defaults.models` (允许列表 + 别名 + 提供商参数)
-   `models.providers` (写入 `models.json` 的自定义提供商)

模型引用被规范化为小写。提供商别名如 `z.ai/*` 规范化为 `zai/*`。提供商配置示例（包括 OpenCode Zen）位于 [/gateway/configuration](../gateway/configuration.md#opencode-zen-multi-model-proxy)。

## “模型不被允许”（以及回复停止的原因）

如果设置了 `agents.defaults.models`，它将成为 `/model` 和会话覆盖的**允许列表**。当用户选择了一个不在该允许列表中的模型时，OpenClaw 会返回：

```bash
Model "provider/model" is not allowed. Use /model to list available models.
```

这发生在生成正常回复**之前**，因此消息可能会让人感觉“没有响应”。修复方法是：
-   将该模型添加到 `agents.defaults.models`，或
-   清除允许列表（移除 `agents.defaults.models`），或
-   从 `/model list` 中选择一个模型。

允许列表配置示例：

```json
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-6": { alias: "Opus" },
    },
  },
}
```

## 在聊天中切换模型 (/model)

您可以在不重启的情况下为当前会话切换模型：

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

注意：

-   `/model`（和 `/model list`）是一个紧凑的、带编号的选择器（模型系列 + 可用提供商）。
-   在 Discord 上，`/model` 和 `/models` 会打开一个交互式选择器，包含提供商和模型下拉列表以及一个提交步骤。
-   `/model <#>` 从该选择器中选择。
-   `/model status` 是详细视图（认证候选者，以及配置后提供商的端点 `baseUrl` + `api` 模式）。
-   模型引用通过**第一个** `/` 进行分割解析。输入 `/model ` 时使用 `provider/model`。
-   如果模型 ID 本身包含 `/`（OpenRouter 风格），您必须包含提供商前缀（例如：`/model openrouter/moonshotai/kimi-k2`）。
-   如果省略提供商，OpenClaw 会将输入视为别名或**默认提供商**的模型（仅当模型 ID 中没有 `/` 时有效）。

完整命令行为/配置：[斜杠命令](../tools/slash-commands.md)。

## CLI 命令

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models`（无子命令）是 `models status` 的快捷方式。

### models list

默认显示已配置的模型。有用的标志：

-   `--all`: 完整目录
-   `--local`: 仅本地提供商
-   `--provider `: 按提供商过滤
-   `--plain`: 每行一个模型
-   `--json`: 机器可读输出

### models status

显示已解析的主模型、回退模型、图像模型以及已配置提供商的认证概览。它还会显示在认证存储中找到的配置文件的 OAuth 过期状态（默认在 24 小时内警告）。`--plain` 仅打印已解析的主模型。OAuth 状态始终显示（并包含在 `--json` 输出中）。如果已配置的提供商没有凭据，`models status` 会打印一个**缺少认证**部分。JSON 包含 `auth.oauth`（警告窗口 + 配置文件）和 `auth.providers`（每个提供商的有效认证）。使用 `--check` 进行自动化（当缺少/过期时退出 `1`，即将过期时退出 `2`）。认证选择取决于提供商/账户。对于始终在线的网关主机，API 密钥通常是最可预测的；也支持订阅令牌流程。示例（Anthropic setup-token）：

```bash
claude setup-token
openclaw models status
```

## 扫描（OpenRouter 免费模型）

`openclaw models scan` 检查 OpenRouter 的**免费模型目录**，并可选择性地探测模型对工具和图像的支持。关键标志：

-   `--no-probe`: 跳过实时探测（仅元数据）
-   `--min-params `: 最小参数大小（十亿）
-   `--max-age-days `: 跳过较旧的模型
-   `--provider `: 提供商前缀过滤器
-   `--max-candidates `: 回退列表大小
-   `--set-default`: 将 `agents.defaults.model.primary` 设置为第一个选择
-   `--set-image`: 将 `agents.defaults.imageModel.primary` 设置为第一个图像选择

探测需要 OpenRouter API 密钥（来自认证配置文件或 `OPENROUTER_API_KEY`）。没有密钥时，使用 `--no-probe` 仅列出候选模型。扫描结果按以下顺序排名：

1.  图像支持
2.  工具延迟
3.  上下文大小
4.  参数数量

输入

-   OpenRouter `/models` 列表（过滤器 `:free`）
-   需要来自认证配置文件或 `OPENROUTER_API_KEY` 的 OpenRouter API 密钥（参见 [/environment](../help/environment.md)）
-   可选过滤器：`--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
-   探测控制：`--timeout`, `--concurrency`

在 TTY 中运行时，您可以交互式地选择回退模型。在非交互模式下，传递 `--yes` 以接受默认值。

## 模型注册表 (models.json)

`models.providers` 中的自定义提供商会写入代理目录下的 `models.json` 文件中（默认 `~/.openclaw/agents//models.json`）。除非 `models.mode` 设置为 `replace`，否则默认会合并此文件。匹配提供商 ID 的合并模式优先级：

-   代理 `models.json` 中已存在的非空 `baseUrl` 优先。
-   代理 `models.json` 中的非空 `apiKey` 仅当该提供商在当前配置/认证配置文件上下文中不受 SecretRef 管理时才优先。
-   受 SecretRef 管理的提供商 `apiKey` 值会从源标记（环境引用为 `ENV_VAR_NAME`，文件/执行引用为 `secretref-managed`）刷新，而不是持久化已解析的密钥。
-   代理 `apiKey`/`baseUrl` 为空或缺失时，回退到配置中的 `models.providers`。
-   其他提供商字段会从配置和规范化的目录数据中刷新。

每当 OpenClaw 重新生成 `models.json` 时，都会应用这种基于标记的持久化，包括命令驱动的路径，如 `openclaw agent`。

[模型提供商快速入门](../providers/models.md)[模型提供商](./model-providers.md)