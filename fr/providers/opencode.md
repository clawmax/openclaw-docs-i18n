title: "Configuration d'OpenCode Zen pour les agents de codage IA OpenClaw"
description: "Apprenez à configurer OpenCode Zen, une liste de modèles recommandés pour les agents de codage IA. Configurez votre clé API et commencez à utiliser le fournisseur opencode."
keywords: ["opencode zen", "agents de codage", "fournisseur de modèles", "configuration clé api", "configuration openclaw", "claude opus", "modèles hébergés", "accès bêta"]
---

  Fournisseurs

  
# OpenCode Zen

OpenCode Zen est une **liste de modèles recommandés** par l'équipe OpenCode pour les agents de codage. C'est un accès optionnel et hébergé aux modèles qui utilise une clé API et le fournisseur `opencode`. Zen est actuellement en version bêta.

## Configuration CLI

```bash
openclaw onboard --auth-choice opencode-zen
# ou non-interactif
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## Extrait de configuration

```json
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## Notes

-   `OPENCODE_ZEN_API_KEY` est également pris en charge.
-   Vous vous connectez à Zen, ajoutez vos informations de facturation et copiez votre clé API.
-   OpenCode Zen facture par requête ; consultez le tableau de bord OpenCode pour plus de détails.

[OpenAI](./openai.md)[OpenRouter](./openrouter.md)