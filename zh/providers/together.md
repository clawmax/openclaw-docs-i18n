

  提供商

  
# Together

[Together AI](https://together.ai) 通过统一的 API 提供对领先开源模型的访问，包括 Llama、DeepSeek、Kimi 等。

-   提供商：`together`
-   认证：`TOGETHER_API_KEY`
-   API：OpenAI 兼容

## 快速开始

1.  设置 API 密钥（推荐：为网关存储它）：

```bash
openclaw onboard --auth-choice together-api-key
```

2.  设置一个默认模型：

```json
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## 非交互式示例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

这将设置 `together/moonshotai/Kimi-K2.5` 为默认模型。

## 环境说明

如果网关作为守护进程（launchd/systemd）运行，请确保 `TOGETHER_API_KEY` 对该进程可用（例如，在 `~/.openclaw/.env` 中或通过 `env.shellEnv`）。

## 可用模型

Together AI 提供对许多流行开源模型的访问：

-   **GLM 4.7 Fp8** - 默认模型，具有 200K 上下文窗口
-   **Llama 3.3 70B Instruct Turbo** - 快速、高效的指令跟随
-   **Llama 4 Scout** - 具有图像理解能力的视觉模型
-   **Llama 4 Maverick** - 高级视觉与推理模型
-   **DeepSeek V3.1** - 强大的编码和推理模型
-   **DeepSeek R1** - 高级推理模型
-   **Kimi K2 Instruct** - 高性能模型，具有 262K 上下文窗口

所有模型都支持标准聊天补全，并且与 OpenAI API 兼容。

[合成](./synthetic.md)[Vercel AI 网关](./vercel-ai-gateway.md)

---