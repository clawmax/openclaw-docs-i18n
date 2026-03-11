

  Extensions

  
# Outils Agent de Plugin

Les plugins OpenClaw peuvent enregistrer des **outils d'agent** (fonctions JSON‑schema) qui sont exposés au LLM lors des exécutions d'agent. Les outils peuvent être **requis** (toujours disponibles) ou **optionnels** (opt‑in). Les outils d'agent sont configurés sous `tools` dans la configuration principale, ou par agent sous `agents.list[].tools`. La politique de liste d'autorisation/refus contrôle les outils que l'agent peut appeler.

## Outil de base

```typescript
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Faire une chose",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```

## Outil optionnel (opt‑in)

Les outils optionnels ne sont **jamais** activés automatiquement. Les utilisateurs doivent les ajouter à une liste d'autorisation d'agent.

```bash
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Exécuter un workflow local",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

Activez les outils optionnels dans `agents.list[].tools.allow` (ou globalement dans `tools.allow`) :

```json
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool", // nom d'outil spécifique
            "workflow", // id de plugin (active tous les outils de ce plugin)
            "group:plugins", // tous les outils de plugin
          ],
        },
      },
    ],
  },
}
```

Autres paramètres de configuration affectant la disponibilité des outils :

-   Les listes d'autorisation qui ne nomment que des outils de plugin sont traitées comme des opt-ins de plugin ; les outils de base restent activés sauf si vous incluez également des outils ou groupes de base dans la liste d'autorisation.
-   `tools.profile` / `agents.list[].tools.profile` (liste d'autorisation de base)
-   `tools.byProvider` / `agents.list[].tools.byProvider` (autorisation/refus spécifique au fournisseur)
-   `tools.sandbox.tools.*` (politique d'outil sandbox en mode isolé)

## Règles + conseils

-   Les noms d'outils ne doivent **pas** entrer en conflit avec les noms d'outils de base ; les outils en conflit sont ignorés.
-   Les ids de plugin utilisés dans les listes d'autorisation ne doivent pas entrer en conflit avec les noms d'outils de base.
-   Préférez `optional: true` pour les outils qui déclenchent des effets de bord ou nécessitent des binaires/identifiants supplémentaires.

[Manifeste de Plugin](./manifest.md)[OpenProse](../prose.md)