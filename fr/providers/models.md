

  Vue d'ensemble

  
# Démarrage rapide des fournisseurs de modèles

OpenClaw peut utiliser de nombreux fournisseurs de LLM. Choisissez-en un, authentifiez-vous, puis définissez le modèle par défaut comme `fournisseur/modèle`.

## Démarrage rapide (deux étapes)

1.  Authentifiez-vous auprès du fournisseur (généralement via `openclaw onboard`).
2.  Définissez le modèle par défaut :

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Fournisseurs pris en charge (sélection de départ)

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
-   [Modèles GLM](./glm.md)
-   [MiniMax](./minimax.md)
-   [Venice (Venice AI)](./venice.md)
-   [Amazon Bedrock](./bedrock.md)
-   [Qianfan](./qianfan.md)

Pour le catalogue complet des fournisseurs (xAI, Groq, Mistral, etc.) et la configuration avancée, consultez [Fournisseurs de modèles](../concepts/model-providers.md).

[Fournisseurs de modèles](../providers.md)[CLI des modèles](../concepts/models.md)