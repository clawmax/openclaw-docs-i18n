

  提供商

  
# OpenCode Zen

OpenCode Zen 是 OpenCode 团队为编程代理推荐的**精选模型列表**。它是一个可选的、托管的模型访问路径，使用 API 密钥和 `opencode` 提供商。Zen 目前处于测试阶段。

## CLI 设置

```bash
openclaw onboard --auth-choice opencode-zen
# 或非交互式
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## 配置片段

```json
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## 注意事项

-   也支持 `OPENCODE_ZEN_API_KEY`。
-   您需要登录 Zen，添加账单信息，并复制您的 API 密钥。
-   OpenCode Zen 按请求计费；请查看 OpenCode 仪表板了解详情。

[OpenAI](./openai.md)[OpenRouter](./openrouter.md)

---