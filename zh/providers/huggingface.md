

  提供商

  
# Hugging Face (推理)

[Hugging Face 推理提供商](https://huggingface.co/docs/inference-providers) 通过单一的路由器 API 提供 OpenAI 兼容的聊天补全服务。您只需一个令牌即可访问众多模型（DeepSeek、Llama 等）。OpenClaw 使用 **OpenAI 兼容端点**（仅限聊天补全）；对于文生图、嵌入或语音，请直接使用 [HF 推理客户端](https://huggingface.co/docs/api-inference/quicktour)。

-   提供商：`huggingface`
-   认证：`HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`（细粒度令牌，需具有 **向推理提供商发起调用** 权限）
-   API：OpenAI 兼容 (`https://router.huggingface.co/v1`)
-   计费：单一 HF 令牌；[定价](https://huggingface.co/docs/inference-providers/pricing) 遵循提供商费率，并提供免费层级。

## 快速开始

1.  在 [Hugging Face → 设置 → 令牌](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) 创建一个细粒度令牌，并授予 **向推理提供商发起调用** 权限。
2.  运行引导程序，在提供商下拉菜单中选择 **Hugging Face**，然后根据提示输入您的 API 密钥：

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3.  在 **默认 Hugging Face 模型** 下拉菜单中，选择您想要的模型（当您拥有有效令牌时，列表会从推理 API 加载；否则会显示内置列表）。您的选择将保存为默认模型。
4.  您也可以在配置中稍后设置或更改默认模型：

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## 非交互式示例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

这将把 `huggingface/deepseek-ai/DeepSeek-R1` 设置为默认模型。

## 环境说明

如果网关以守护进程（launchd/systemd）形式运行，请确保 `HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN` 对该进程可用（例如，在 `~/.openclaw/.env` 中或通过 `env.shellEnv` 设置）。

## 模型发现与引导程序下拉菜单

OpenClaw 通过直接调用 **推理端点** 来发现模型：

```bash
GET https://router.huggingface.co/v1/models
```

（可选：发送 `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` 或 `$HF_TOKEN` 以获取完整列表；某些端点未经认证会返回子集。）响应为 OpenAI 风格的 `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`。当您配置 Hugging Face API 密钥（通过引导程序、`HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`）后，OpenClaw 会使用此 GET 请求来发现可用的聊天补全模型。在 **交互式引导** 期间，输入令牌后，您会看到一个 **默认 Hugging Face 模型** 下拉菜单，其中的选项来自该列表（如果请求失败，则使用内置目录）。在运行时（例如网关启动时），当存在密钥时，OpenClaw 会再次调用 **GET** `https://router.huggingface.co/v1/models` 来刷新目录。该列表会与内置目录（包含上下文窗口和成本等元数据）合并。如果请求失败或未设置密钥，则仅使用内置目录。

## 模型名称与可编辑选项

-   **来自 API 的名称：** 当 API 返回 `name`、`title` 或 `display_name` 时，模型显示名称**从 GET /v1/models 获取**；否则将从模型 ID 派生（例如 `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”）。
-   **覆盖显示名称：** 您可以在配置中为每个模型设置自定义标签，以便在 CLI 和 UI 中以您希望的方式显示：

```json
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (快速)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (最便宜)" },
      },
    },
  },
}
```

-   **提供商 / 策略选择：** 在**模型 ID** 后附加后缀以选择路由器如何挑选后端：
    
    -   **`:fastest`** — 最高吞吐量（路由器选择；提供商选择被**锁定** — 无交互式后端选择器）。
    -   **`:cheapest`** — 每个输出令牌成本最低（路由器选择；提供商选择被**锁定**）。
    -   **`:provider`** — 强制使用特定后端（例如 `:sambanova`、`:together`）。
    
    当您选择 **:cheapest** 或 **:fastest**（例如在引导程序的模型下拉菜单中）时，提供商将被锁定：路由器根据成本或速度决定，并且不会显示可选的“偏好特定后端”步骤。您可以将这些作为单独的条目添加到 `models.providers.huggingface.models` 中，或者在 `model.primary` 中设置后缀。您也可以在 [推理提供商设置](https://hf.co/settings/inference-providers) 中设置默认顺序（无后缀 = 使用该顺序）。
-   **配置合并：** 配置合并时，`models.providers.huggingface.models` 中的现有条目（例如在 `models.json` 中）会被保留。因此，您在那里设置的任何自定义 `name`、`alias` 或模型选项都将被保留。

## 模型 ID 与配置示例

模型引用使用 `huggingface//` 形式（Hub 风格 ID）。下表来自 **GET** `https://router.huggingface.co/v1/models`；您的目录可能包含更多模型。**示例 ID（来自推理端点）：**

| 模型 | 引用（前缀为 `huggingface/`） |
| --- | --- |
| DeepSeek R1 | `deepseek-ai/DeepSeek-R1` |
| DeepSeek V3.2 | `deepseek-ai/DeepSeek-V3.2` |
| Qwen3 8B | `Qwen/Qwen3-8B` |
| Qwen2.5 7B Instruct | `Qwen/Qwen2.5-7B-Instruct` |
| Qwen3 32B | `Qwen/Qwen3-32B` |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct` |
| Llama 3.1 8B Instruct | `meta-llama/Llama-3.1-8B-Instruct` |
| GPT-OSS 120B | `openai/gpt-oss-120b` |
| GLM 4.7 | `zai-org/GLM-4.7` |
| Kimi K2.5 | `moonshotai/Kimi-K2.5` |

您可以在模型 ID 后附加 `:fastest`、`:cheapest` 或 `:provider`（例如 `:together`、`:sambanova`）。在 [推理提供商设置](https://hf.co/settings/inference-providers) 中设置您的默认顺序；有关完整列表，请参阅 [推理提供商](https://huggingface.co/docs/inference-providers) 和 **GET** `https://router.huggingface.co/v1/models`。

### 完整配置示例

**主要使用 DeepSeek R1，Qwen 作为后备：**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen 作为默认模型，包含 :cheapest 和 :fastest 变体：**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (最便宜)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (最快)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS 并设置别名：**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**使用 :provider 强制指定特定后端：**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**多个 Qwen 和 DeepSeek 模型，带策略后缀：**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (便宜)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (快速)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```

[GitHub Copilot](./github-copilot.md)[Kilocode](./kilocode.md)