

  Обзор

  
# Провайдеры моделей

OpenClaw может использовать множество провайдеров LLM. Выберите провайдера, пройдите аутентификацию, затем установите модель по умолчанию как `провайдер/модель`. Ищете документацию по каналам для чата (WhatsApp/Telegram/Discord/Slack/Mattermost (плагин)/и т.д.)? Смотрите [Каналы](./channels.md).

## Быстрый старт

1.  Пройдите аутентификацию у провайдера (обычно через `openclaw onboard`).
2.  Установите модель по умолчанию:

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Документация по провайдерам

-   [Amazon Bedrock](./providers/bedrock.md)
-   [Anthropic (API + Claude Code CLI)](./providers/anthropic.md)
-   [Cloudflare AI Gateway](./providers/cloudflare-ai-gateway.md)
-   [GLM модели](./providers/glm.md)
-   [Hugging Face (Inference)](./providers/huggingface.md)
-   [Kilocode](./providers/kilocode.md)
-   [LiteLLM (унифицированный шлюз)](./providers/litellm.md)
-   [MiniMax](./providers/minimax.md)
-   [Mistral](./providers/mistral.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./providers/moonshot.md)
-   [NVIDIA](./providers/nvidia.md)
-   [Ollama (локальные модели)](./providers/ollama.md)
-   [OpenAI (API + Codex)](./providers/openai.md)
-   [OpenCode Zen](./providers/opencode.md)
-   [OpenRouter](./providers/openrouter.md)
-   [Qianfan](./providers/qianfan.md)
-   [Qwen (OAuth)](./providers/qwen.md)
-   [Together AI](./providers/together.md)
-   [Vercel AI Gateway](./providers/vercel-ai-gateway.md)
-   [Venice (Venice AI, ориентированный на приватность)](./providers/venice.md)
-   [vLLM (локальные модели)](./providers/vllm.md)
-   [Xiaomi](./providers/xiaomi.md)
-   [Z.AI](./providers/zai.md)

## Провайдеры транскрипции

-   [Deepgram (транскрипция аудио)](./providers/deepgram.md)

## Инструменты сообщества

-   [Claude Max API Proxy](./providers/claude-max-api-proxy.md) - Прокси сообщества для учетных данных подписки Claude (перед использованием проверьте политику/условия Anthropic)

Полный каталог провайдеров (xAI, Groq, Mistral и т.д.) и расширенная конфигурация доступны в разделе [Провайдеры моделей](./concepts/model-providers.md).

[Быстрый старт с провайдерами моделей](./providers/models.md)