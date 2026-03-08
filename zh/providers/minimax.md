

  提供商

  
# MiniMax

MiniMax 是一家构建 **M2/M2.5** 模型系列的 AI 公司。当前专注于编码的版本是 **MiniMax M2.5**（2025年12月23日发布），专为现实世界复杂任务而构建。来源：[MiniMax M2.5 发布说明](https://www.minimax.io/news/minimax-m25)

## 模型概述 (M2.5)

MiniMax 强调了 M2.5 中的这些改进：

-   更强的**多语言编码**能力（Rust、Java、Go、C++、Kotlin、Objective-C、TS/JS）。
-   更好的**Web/应用开发**和美学输出质量（包括原生移动端）。
-   改进了**复合指令**处理能力，适用于办公风格工作流，建立在交错思考和集成约束执行的基础上。
-   **更简洁的响应**，令牌使用量更低，迭代循环更快。
-   更强的**工具/代理框架**兼容性和上下文管理能力（Claude Code、Droid/Factory AI、Cline、Kilo Code、Roo Code、BlackBox）。
-   更高质量的**对话和技术写作**输出。

## MiniMax M2.5 对比 MiniMax M2.5 Highspeed

-   **速度：** `MiniMax-M2.5-highspeed` 是 MiniMax 文档中的官方快速层级。
-   **成本：** MiniMax 定价显示高速版的输入成本相同，但输出成本更高。
-   **兼容性：** OpenClaw 仍接受旧的 `MiniMax-M2.5-Lightning` 配置，但新设置建议使用 `MiniMax-M2.5-highspeed`。

## 选择设置方式

### MiniMax OAuth（编程计划） — 推荐

**适用于：** 通过 OAuth 快速设置 MiniMax 编程计划，无需 API 密钥。启用捆绑的 OAuth 插件并进行身份验证：

```bash
openclaw plugins enable minimax-portal-auth  # 如果已加载则跳过。
openclaw gateway restart  # 如果网关已在运行，则重启
openclaw onboard --auth-choice minimax-portal
```

系统将提示您选择一个端点：

-   **Global** - 国际用户 (`api.minimax.io`)
-   **CN** - 中国用户 (`api.minimaxi.com`)

详情请参阅 [MiniMax OAuth 插件 README](https://github.com/openclaw/openclaw/tree/main/extensions/minimax-portal-auth)。

### MiniMax M2.5（API 密钥）

**适用于：** 使用 Anthropic 兼容 API 的托管 MiniMax。通过 CLI 配置：

-   运行 `openclaw configure`
-   选择 **Model/auth**
-   选择 **MiniMax M2.5**

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.5" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.5",
            name: "MiniMax M2.5",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
          {
            id: "MiniMax-M2.5-highspeed",
            name: "MiniMax M2.5 Highspeed",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### MiniMax M2.5 作为回退（示例）

**适用于：** 将您最强大的最新一代模型设为主要模型，故障时回退到 MiniMax M2.5。下面的示例使用 Opus 作为具体的主要模型；请替换为您偏好的最新一代主要模型。

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "primary" },
        "minimax/MiniMax-M2.5": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.5"],
      },
    },
  },
}
```

### 可选：通过 LM Studio 本地运行（手动）

**适用于：** 使用 LM Studio 进行本地推理。我们已在强大硬件（例如台式机/服务器）上使用 LM Studio 的本地服务器看到了 MiniMax M2.5 的出色效果。通过 `openclaw.json` 手动配置：

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 通过 openclaw configure 配置

使用交互式配置向导设置 MiniMax，无需编辑 JSON：

1.  运行 `openclaw configure`。
2.  选择 **Model/auth**。
3.  选择 **MiniMax M2.5**。
4.  在提示时选择您的默认模型。

## 配置选项

-   `models.providers.minimax.baseUrl`：建议使用 `https://api.minimax.io/anthropic`（Anthropic 兼容）；`https://api.minimax.io/v1` 可选用于 OpenAI 兼容负载。
-   `models.providers.minimax.api`：建议使用 `anthropic-messages`；`openai-completions` 可选用于 OpenAI 兼容负载。
-   `models.providers.minimax.apiKey`：MiniMax API 密钥 (`MINIMAX_API_KEY`)。
-   `models.providers.minimax.models`：定义 `id`、`name`、`reasoning`、`contextWindow`、`maxTokens`、`cost`。
-   `agents.defaults.models`：为希望加入允许列表的模型设置别名。
-   `models.mode`：如果您想将 MiniMax 与内置模型一起添加，请保持 `merge`。

## 注意事项

-   模型引用格式为 `minimax/`。
-   推荐的模型 ID：`MiniMax-M2.5` 和 `MiniMax-M2.5-highspeed`。
-   编程计划使用量 API：`https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains`（需要编程计划密钥）。
-   如果需要精确的成本跟踪，请在 `models.json` 中更新定价值。
-   MiniMax 编程计划推荐链接（9折优惠）：[https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link](https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link)
-   有关提供商规则，请参阅 [/concepts/model-providers](../concepts/model-providers.md)。
-   使用 `openclaw models list` 和 `openclaw models set minimax/MiniMax-M2.5` 进行切换。

## 故障排除

### “未知模型：minimax/MiniMax-M2.5”

这通常意味着 **MiniMax 提供商未配置**（没有提供商条目且未找到 MiniMax 身份验证配置文件/环境密钥）。此检测的修复在 **2026.1.12** 版本中（撰写本文时尚未发布）。通过以下方式修复：

-   升级到 **2026.1.12**（或从 `main` 分支源码运行），然后重启网关。
-   运行 `openclaw configure` 并选择 **MiniMax M2.5**，或
-   手动添加 `models.providers.minimax` 代码块，或
-   设置 `MINIMAX_API_KEY`（或 MiniMax 身份验证配置文件），以便可以注入提供商。

确保模型 ID **区分大小写**：

-   `minimax/MiniMax-M2.5`
-   `minimax/MiniMax-M2.5-highspeed`
-   `minimax/MiniMax-M2.5-Lightning`（旧版）

然后重新检查：

```bash
openclaw models list
```

[GLM 模型](./glm.md)[Moonshot AI](./moonshot.md)