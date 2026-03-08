

  Descripción general

  
# Inicio Rápido de Proveedores de Modelos

OpenClaw puede utilizar muchos proveedores de LLM. Elige uno, autentícate y luego establece el modelo predeterminado como `proveedor/modelo`.

## Inicio rápido (dos pasos)

1.  Autentícate con el proveedor (generalmente mediante `openclaw onboard`).
2.  Establece el modelo predeterminado:

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Proveedores soportados (conjunto inicial)

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

Para el catálogo completo de proveedores (xAI, Groq, Mistral, etc.) y configuración avanzada, consulta [Proveedores de modelos](../concepts/model-providers.md).

[Proveedores de Modelos](../providers.md)[CLI de Modelos](../concepts/models.md)

---