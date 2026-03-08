

  Fournisseurs

  
# Z.AI

Z.AI est la plateforme API pour les modèles **GLM**. Elle fournit des API REST pour GLM et utilise des clés API pour l'authentification. Créez votre clé API dans la console Z.AI. OpenClaw utilise le fournisseur `zai` avec une clé API Z.AI.

## Configuration en CLI

```bash
openclaw onboard --auth-choice zai-api-key
# ou en mode non interactif
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## Extrait de configuration

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Notes

-   Les modèles GLM sont disponibles sous la forme `zai/` (exemple : `zai/glm-5`).
-   `tool_stream` est activé par défaut pour le streaming des appels d'outils Z.AI. Définissez `agents.defaults.models["zai/"].params.tool_stream` sur `false` pour le désactiver.
-   Consultez [/providers/glm](./glm.md) pour un aperçu de la famille de modèles.
-   Z.AI utilise l'authentification Bearer avec votre clé API.

[Xiaomi MiMo](./xiaomi.md)

---