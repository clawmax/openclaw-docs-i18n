

  Aperçu

  
# Fournisseurs de modèles

OpenClaw peut utiliser de nombreux fournisseurs de LLM. Choisissez un fournisseur, authentifiez-vous, puis définissez le modèle par défaut comme `fournisseur/modèle`. Vous cherchez la documentation des canaux de discussion (WhatsApp/Telegram/Discord/Slack/Mattermost (plugin)/etc.) ? Voir [Canaux](./channels.md).

## Démarrage rapide

1.  Authentifiez-vous auprès du fournisseur (généralement via `openclaw onboard`).
2.  Définissez le modèle par défaut :

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Documentation des fournisseurs

-   [Amazon Bedrock](./providers/bedrock.md)
-   [Anthropic (API + CLI Claude Code)](./providers/anthropic.md)
-   [Cloudflare AI Gateway](./providers/cloudflare-ai-gateway.md)
-   [Modèles GLM](./providers/glm.md)
-   [Hugging Face (Inférence)](./providers/huggingface.md)
-   [Kilocode](./providers/kilocode.md)
-   [LiteLLM (passerelle unifiée)](./providers/litellm.md)
-   [MiniMax](./providers/minimax.md)
-   [Mistral](./providers/mistral.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./providers/moonshot.md)
-   [NVIDIA](./providers/nvidia.md)
-   [Ollama (modèles locaux)](./providers/ollama.md)
-   [OpenAI (API + Codex)](./providers/openai.md)
-   [OpenCode Zen](./providers/opencode.md)
-   [OpenRouter](./providers/openrouter.md)
-   [Qianfan](./providers/qianfan.md)
-   [Qwen (OAuth)](./providers/qwen.md)
-   [Together AI](./providers/together.md)
-   [Vercel AI Gateway](./providers/vercel-ai-gateway.md)
-   [Venice (Venice AI, axé sur la confidentialité)](./providers/venice.md)
-   [vLLM (modèles locaux)](./providers/vllm.md)
-   [Xiaomi](./providers/xiaomi.md)
-   [Z.AI](./providers/zai.md)

## Fournisseurs de transcription

-   [Deepgram (transcription audio)](./providers/deepgram.md)

## Outils communautaires

-   [Claude Max API Proxy](./providers/claude-max-api-proxy.md) - Proxy communautaire pour les identifiants d'abonnement Claude (vérifiez la politique/les conditions d'Anthropic avant utilisation)

Pour le catalogue complet des fournisseurs (xAI, Groq, Mistral, etc.) et la configuration avancée, voir [Fournisseurs de modèles](./concepts/model-providers.md).

[Démarrage rapide des fournisseurs de modèles](./providers/models.md)

---