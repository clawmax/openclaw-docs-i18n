title: "Configuration et paramétrage du fournisseur OpenRouter pour OpenClaw AI"
description: "Apprenez à configurer et paramétrer le fournisseur OpenRouter dans OpenClaw AI. Utilisez une API unifiée pour accéder à de multiples modèles d'IA avec une seule clé API et un point de terminaison compatible OpenAI."
keywords: ["openrouter", "openclaw ai", "fournisseur api", "api unifiée", "routage de modèles", "compatible openai", "claude sonnet", "configuration ia"]
---

  Fournisseurs

  
# OpenRouter

OpenRouter fournit une **API unifiée** qui achemine les requêtes vers de nombreux modèles derrière un seul point de terminaison et une seule clé API. Elle est compatible OpenAI, donc la plupart des SDK OpenAI fonctionnent en changeant l'URL de base.

## Configuration en ligne de commande

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## Extrait de configuration

```json
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## Notes

-   Les références de modèles sont `openrouter//<modèle>`.
-   Pour plus d'options de modèles/fournisseurs, consultez [/concepts/model-providers](../concepts/model-providers.md).
-   OpenRouter utilise un jeton Bearer avec votre clé API en arrière-plan.

[OpenCode Zen](./opencode.md)[Qianfan](./qianfan.md)

---