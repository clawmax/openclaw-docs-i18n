title: "Documentación y Guía de Configuración de Proveedores de Modelos de IA de OpenClaw"
description: "Aprende a autenticar y configurar proveedores de LLM como Anthropic, OpenAI y Ollama en OpenClaw. Establece tu modelo predeterminado y explora el catálogo completo de proveedores."
keywords: ["openclaw", "proveedores de modelos", "configuración de llm", "anthropic claude", "openai api", "ollama", "litellm", "bedrock"]
---

  Descripción general

  
# Proveedores de Modelos

OpenClaw puede utilizar muchos proveedores de LLM. Elige un proveedor, autentícate y luego establece el modelo predeterminado como `proveedor/modelo`. ¿Buscas documentación sobre canales de chat (WhatsApp/Telegram/Discord/Slack/Mattermost (plugin)/etc.)? Consulta [Canales](./channels.md).

## Inicio rápido

1.  Autentícate con el proveedor (generalmente mediante `openclaw onboard`).
2.  Establece el modelo predeterminado:

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Documentación de proveedores

-   [Amazon Bedrock](./providers/bedrock.md)
-   [Anthropic (API + Claude Code CLI)](./providers/anthropic.md)
-   [Cloudflare AI Gateway](./providers/cloudflare-ai-gateway.md)
-   [Modelos GLM](./providers/glm.md)
-   [Hugging Face (Inferencia)](./providers/huggingface.md)
-   [Kilocode](./providers/kilocode.md)
-   [LiteLLM (puerta de enlace unificada)](./providers/litellm.md)
-   [MiniMax](./providers/minimax.md)
-   [Mistral](./providers/mistral.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./providers/moonshot.md)
-   [NVIDIA](./providers/nvidia.md)
-   [Ollama (modelos locales)](./providers/ollama.md)
-   [OpenAI (API + Codex)](./providers/openai.md)
-   [OpenCode Zen](./providers/opencode.md)
-   [OpenRouter](./providers/openrouter.md)
-   [Qianfan](./providers/qianfan.md)
-   [Qwen (OAuth)](./providers/qwen.md)
-   [Together AI](./providers/together.md)
-   [Vercel AI Gateway](./providers/vercel-ai-gateway.md)
-   [Venice (Venice AI, centrado en la privacidad)](./providers/venice.md)
-   [vLLM (modelos locales)](./providers/vllm.md)
-   [Xiaomi](./providers/xiaomi.md)
-   [Z.AI](./providers/zai.md)

## Proveedores de transcripción

-   [Deepgram (transcripción de audio)](./providers/deepgram.md)

## Herramientas de la comunidad

-   [Claude Max API Proxy](./providers/claude-max-api-proxy.md) - Proxy comunitario para credenciales de suscripción a Claude (verifica la política/términos de Anthropic antes de usar)

Para el catálogo completo de proveedores (xAI, Groq, Mistral, etc.) y configuración avanzada, consulta [Proveedores de modelos](./concepts/model-providers.md).

[Inicio Rápido de Proveedores de Modelos](./providers/models.md)

---