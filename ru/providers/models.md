

  Обзор

  
# Быстрый старт с провайдерами моделей

OpenClaw может использовать множество провайдеров LLM. Выберите одного, пройдите аутентификацию, затем установите модель по умолчанию как `провайдер/модель`.

## Быстрый старт (два шага)

1.  Пройдите аутентификацию у провайдера (обычно через `openclaw onboard`).
2.  Установите модель по умолчанию:

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Поддерживаемые провайдеры (стартовый набор)

-   [OpenAI (API + Codex)](./openai.md)
-   [Anthropic (API + Claude Code CLI)](./anthropic.md)
-   [OpenRouter](./openrouter.md)
-   [Vercel AI Gateway](./vercel-ai-gateway.md)
-   [Cloudflare AI Gateway](./cloudflare-ai-gateway.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./moonshot.md)
-   [Mistral](./mistral.md)
-   [Synthetic](./synthetic.md)
-   [OpenCode Zen](./opencode.md)
-   [Z.AI](./zai.md)
-   [GLM models](./glm.md)
-   [MiniMax](./minimax.md)
-   [Venice (Venice AI)](./venice.md)
-   [Amazon Bedrock](./bedrock.md)
-   [Qianfan](./qianfan.md)

Полный каталог провайдеров (xAI, Groq, Mistral и др.) и расширенную конфигурацию смотрите в разделе [Провайдеры моделей](../concepts/model-providers.md).

[Провайдеры моделей](../providers.md)[CLI для моделей](../concepts/models.md)

---