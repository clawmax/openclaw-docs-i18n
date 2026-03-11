

  Protocoles et APIs

  
# Modèles locaux

Le local est faisable, mais OpenClaw nécessite un grand contexte et des défenses solides contre l'injection de prompt. Les petites cartes tronquent le contexte et compromettent la sécurité. Visez haut : **≥2 Mac Studios maxés ou une configuration GPU équivalente (~30k$+)**. Un seul GPU de **24 Go** ne fonctionne que pour des prompts plus légers avec une latence plus élevée. Utilisez la **variante de modèle la plus grande / pleine taille que vous pouvez exécuter** ; les checkpoints fortement quantifiés ou "petits" augmentent le risque d'injection de prompt (voir [Sécurité](./security.md)).

## Recommandé : LM Studio + MiniMax M2.5 (API Responses, pleine taille)

La meilleure pile locale actuelle. Chargez MiniMax M2.5 dans LM Studio, activez le serveur local (par défaut `http://127.0.0.1:1234`), et utilisez l'API Responses pour garder le raisonnement séparé du texte final.

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**Liste de contrôle d'installation**

-   Installez LM Studio : [https://lmstudio.ai](https://lmstudio.ai)
-   Dans LM Studio, téléchargez la **version la plus grande de MiniMax M2.5 disponible** (évitez les variantes "petites"/fortement quantifiées), démarrez le serveur, confirmez que `http://127.0.0.1:1234/v1/models` la liste.
-   Gardez le modèle chargé ; un chargement à froid ajoute de la latence au démarrage.
-   Ajustez `contextWindow`/`maxTokens` si votre version de LM Studio diffère.
-   Pour WhatsApp, restez sur l'API Responses afin que seul le texte final soit envoyé.

Gardez les modèles hébergés configurés même lors de l'exécution locale ; utilisez `models.mode: "merge"` pour que les secours restent disponibles.

### Configuration hybride : hébergé en primaire, local en secours

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.5-gs32", "anthropic/claude-opus-4-6"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.5-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### Local en premier avec filet de sécurité hébergé

Inversez l'ordre du primaire et du secours ; gardez le même bloc de fournisseurs et `models.mode: "merge"` pour pouvoir basculer vers Sonnet ou Opus lorsque la machine locale est indisponible.

### Hébergement régional / routage des données

-   Les variantes hébergées de MiniMax/Kimi/GLM existent également sur OpenRouter avec des points de terminaison régionaux (par ex., hébergés aux États-Unis). Choisissez la variante régionale là-bas pour maintenir le trafic dans votre juridiction choisie tout en utilisant `models.mode: "merge"` pour les secours Anthropic/OpenAI.
-   Le mode local uniquement reste le chemin le plus fort pour la confidentialité ; le routage régional hébergé est le compromis lorsque vous avez besoin des fonctionnalités des fournisseurs mais souhaitez contrôler le flux de données.

## Autres proxys locaux compatibles OpenAI

vLLM, LiteLLM, OAI-proxy, ou des passerelles personnalisées fonctionnent si elles exposent un point de terminaison `/v1` de style OpenAI. Remplacez le bloc de fournisseur ci-dessus par votre point de terminaison et votre ID de modèle :

```json
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Modèle local",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Gardez `models.mode: "merge"` pour que les modèles hébergés restent disponibles en secours.

## Dépannage

-   La passerelle peut-elle atteindre le proxy ? `curl http://127.0.0.1:1234/v1/models`.
-   Le modèle LM Studio est-il déchargé ? Rechargez-le ; le démarrage à froid est une cause fréquente de "blocage".
-   Des erreurs de contexte ? Baissez `contextWindow` ou augmentez la limite de votre serveur.
-   Sécurité : les modèles locaux ignorent les filtres côté fournisseur ; gardez les agents restreints et la compaction activée pour limiter la surface d'attaque de l'injection de prompt.

[Backends CLI](./cli-backends.md)[Modèle réseau](./network-model.md)