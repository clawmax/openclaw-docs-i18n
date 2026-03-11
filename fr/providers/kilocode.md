

  Fournisseurs

  
# Kilocode

Kilo Gateway fournit une **API unifiée** qui achemine les requêtes vers de nombreux modèles derrière un seul point de terminaison et une seule clé API. Elle est compatible avec OpenAI, donc la plupart des SDK OpenAI fonctionnent en changeant l'URL de base.

## Obtenir une clé API

1.  Allez sur [app.kilo.ai](https://app.kilo.ai)
2.  Connectez-vous ou créez un compte
3.  Naviguez vers "API Keys" et générez une nouvelle clé

## Configuration en ligne de commande

```bash
openclaw onboard --kilocode-api-key <key>
```

Ou définissez la variable d'environnement :

```bash
export KILOCODE_API_KEY="<your-kilocode-api-key>" # pragma: allowlist secret
```

## Extrait de configuration

```json
{
  env: { KILOCODE_API_KEY: "<your-kilocode-api-key>" }, // pragma: allowlist secret
  agents: {
    defaults: {
      model: { primary: "kilocode/kilo/auto" },
    },
  },
}
```

## Modèle par défaut

Le modèle par défaut est `kilocode/kilo/auto`, un modèle de routage intelligent qui sélectionne automatiquement le meilleur modèle sous-jacent en fonction de la tâche :

-   Les tâches de planification, débogage et orchestration sont acheminées vers Claude Opus
-   Les tâches d'écriture et d'exploration de code sont acheminées vers Claude Sonnet

## Modèles disponibles

OpenClaw découvre dynamiquement les modèles disponibles depuis la passerelle Kilo au démarrage. Utilisez `/models kilocode` pour voir la liste complète des modèles disponibles avec votre compte. Tout modèle disponible sur la passerelle peut être utilisé avec le préfixe `kilocode/` :

```
kilocode/kilo/auto              (par défaut - routage intelligent)
kilocode/anthropic/claude-sonnet-4
kilocode/openai/gpt-5.2
kilocode/google/gemini-3-pro-preview
...et bien d'autres
```

## Notes

-   Les références de modèles sont `kilocode/<model-id>` (par exemple, `kilocode/anthropic/claude-sonnet-4`).
-   Modèle par défaut : `kilocode/kilo/auto`
-   URL de base : `https://api.kilo.ai/api/gateway/`
-   Pour plus d'options de modèles/fournisseurs, consultez [/concepts/model-providers](../concepts/model-providers.md).
-   Kilo Gateway utilise un jeton Bearer avec votre clé API en arrière-plan.

[Hugging Face (Inference)](./huggingface.md)[Litellm](./litellm.md)