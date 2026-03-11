

  Outils intégrés

  
# Tâche LLM

`llm-task` est un **outil plugin optionnel** qui exécute une tâche LLM en JSON uniquement et renvoie une sortie structurée (optionnellement validée par rapport à un schéma JSON). C'est idéal pour les moteurs de workflow comme Lobster : vous pouvez ajouter une seule étape LLM sans écrire de code OpenClaw personnalisé pour chaque workflow.

## Activer le plugin

1.  Activez le plugin :

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2.  Mettez l'outil en liste autorisée (il est enregistré avec `optional: true`) :

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

## Configuration (optionnelle)

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.4",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.4"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` est une liste autorisée de chaînes `provider/model`. Si elle est définie, toute requête en dehors de la liste est rejetée.

## Paramètres de l'outil

-   `prompt` (chaîne, requis)
-   `input` (n'importe quel type, optionnel)
-   `schema` (objet, schéma JSON optionnel)
-   `provider` (chaîne, optionnel)
-   `model` (chaîne, optionnel)
-   `authProfileId` (chaîne, optionnel)
-   `temperature` (nombre, optionnel)
-   `maxTokens` (nombre, optionnel)
-   `timeoutMs` (nombre, optionnel)

## Sortie

Renvoie `details.json` contenant le JSON analysé (et le valide par rapport au `schema` lorsqu'il est fourni).

## Exemple : étape de workflow Lobster

```
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

## Notes de sécurité

-   L'outil est **uniquement en JSON** et demande au modèle de ne produire que du JSON (pas de blocs de code, pas de commentaires).
-   Aucun outil n'est exposé au modèle pour cette exécution.
-   Traitez la sortie comme non fiable à moins de la valider avec `schema`.
-   Placez des approbations avant toute étape ayant des effets de bord (envoi, publication, exécution).

[Firecrawl](./firecrawl.md)[Lobster](./lobster.md)