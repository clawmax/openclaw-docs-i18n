title: "Configurer le fournisseur Moonshot AI et Kimi Coding pour OpenClaw"
description: "Apprenez à configurer le fournisseur Moonshot AI et Kimi Coding dans OpenClaw. Configurez les clés API, les points de terminaison et les références de modèles comme moonshot/kimi-k2.5."
keywords: ["moonshot ai", "api kimi", "fournisseur openclaw", "kimi coding", "kimi k2.5", "api compatible openai", "configuration api", "configuration modèle"]
---

  Fournisseurs

  
# Moonshot AI

Moonshot fournit l'API Kimi avec des points de terminaison compatibles OpenAI. Configurez le fournisseur et définissez le modèle par défaut sur `moonshot/kimi-k2.5`, ou utilisez Kimi Coding avec `kimi-coding/k2p5`. Identifiants actuels des modèles Kimi K2 :

-   `kimi-k2.5`
-   `kimi-k2-0905-preview`
-   `kimi-k2-turbo-preview`
-   `kimi-k2-thinking`
-   `kimi-k2-thinking-turbo`

```bash
openclaw onboard --auth-choice moonshot-api-key
```

Kimi Coding :

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

Note : Moonshot et Kimi Coding sont des fournisseurs distincts. Les clés ne sont pas interchangeables, les points de terminaison diffèrent et les références de modèles diffèrent (Moonshot utilise `moonshot/...`, Kimi Coding utilise `kimi-coding/...`).

## Extrait de configuration (API Moonshot)

```json
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: {
        // moonshot-kimi-k2-aliases:start
        "moonshot/kimi-k2.5": { alias: "Kimi K2.5" },
        "moonshot/kimi-k2-0905-preview": { alias: "Kimi K2" },
        "moonshot/kimi-k2-turbo-preview": { alias: "Kimi K2 Turbo" },
        "moonshot/kimi-k2-thinking": { alias: "Kimi K2 Thinking" },
        "moonshot/kimi-k2-thinking-turbo": { alias: "Kimi K2 Thinking Turbo" },
        // moonshot-kimi-k2-aliases:end
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          // moonshot-kimi-k2-models:start
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-0905-preview",
            name: "Kimi K2 0905 Preview",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-turbo-preview",
            name: "Kimi K2 Turbo",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-thinking",
            name: "Kimi K2 Thinking",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-thinking-turbo",
            name: "Kimi K2 Thinking Turbo",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          // moonshot-kimi-k2-models:end
        ],
      },
    },
  },
}
```

## Kimi Coding

```json
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: {
        "kimi-coding/k2p5": { alias: "Kimi K2.5" },
      },
    },
  },
}
```

## Notes

-   Les références de modèles Moonshot utilisent `moonshot/`. Les références de modèles Kimi Coding utilisent `kimi-coding/`.
-   Remplacez les tarifs et les métadonnées de contexte dans `models.providers` si nécessaire.
-   Si Moonshot publie des limites de contexte différentes pour un modèle, ajustez `contextWindow` en conséquence.
-   Utilisez `https://api.moonshot.ai/v1` pour le point de terminaison international, et `https://api.moonshot.cn/v1` pour le point de terminaison Chine.

[MiniMax](./minimax.md)[Mistral](./mistral.md)