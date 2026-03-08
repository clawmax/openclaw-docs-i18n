title: "Comment utiliser les modèles GLM dans OpenClaw via le fournisseur Z.AI"
description: "Apprenez à configurer et paramétrer les modèles GLM comme glm-5 dans OpenClaw en utilisant le fournisseur Z.AI. Inclut la configuration CLI et des exemples de configuration."
keywords: ["modèles glm", "fournisseur openclaw zai", "configuration glm-5", "clé api zai", "configuration openclaw", "famille de modèles glm", "plateforme zai", "fournisseurs openclaw"]
---

  Fournisseurs

  
# Modèles GLM

GLM est une **famille de modèles** (et non une entreprise) disponible via la plateforme Z.AI. Dans OpenClaw, les modèles GLM sont accessibles via le fournisseur `zai` et des identifiants de modèle comme `zai/glm-5`.

## Configuration CLI

```bash
openclaw onboard --auth-choice zai-api-key
```

## Extrait de configuration

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Notes

-   Les versions et la disponibilité des modèles GLM peuvent changer ; consultez la documentation de Z.AI pour les dernières informations.
-   Les exemples d'identifiants de modèle incluent `glm-5`, `glm-4.7` et `glm-4.6`.
-   Pour les détails sur le fournisseur, consultez [/providers/zai](./zai.md).

[Litellm](./litellm.md)[MiniMax](./minimax.md)