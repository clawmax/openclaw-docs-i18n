title: "Настройка провайдера Moonshot AI и Kimi Coding для OpenClaw"
description: "Узнайте, как настроить провайдер Moonshot AI и Kimi Coding в OpenClaw. Настройте API-ключи, конечные точки и ссылки на модели, такие как moonshot/kimi-k2.5."
keywords: ["moonshot ai", "kimi api", "провайдер openclaw", "kimi coding", "kimi k2.5", "openai-совместимый api", "настройка api", "настройка модели"]
---

  Провайдеры

  
# Moonshot AI

Moonshot предоставляет Kimi API с OpenAI-совместимыми конечными точками. Настройте провайдер и установите модель по умолчанию `moonshot/kimi-k2.5` или используйте Kimi Coding с `kimi-coding/k2p5`. Текущие идентификаторы моделей Kimi K2:

-   `kimi-k2.5`
-   `kimi-k2-0905-preview`
-   `kimi-k2-turbo-preview`
-   `kimi-k2-thinking`
-   `kimi-k2-thinking-turbo`

```bash
openclaw onboard --auth-choice moonshot-api-key
```

Kimi Coding:

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

Примечание: Moonshot и Kimi Coding — это отдельные провайдеры. Ключи не взаимозаменяемы, конечные точки различаются, и ссылки на модели различаются (Moonshot использует `moonshot/...`, Kimi Coding использует `kimi-coding/...`).

## Фрагмент конфигурации (Moonshot API)

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

## Примечания

-   Ссылки на модели Moonshot используют `moonshot/`. Ссылки на модели Kimi Coding используют `kimi-coding/`.
-   При необходимости переопределите стоимость и метаданные контекста в `models.providers`.
-   Если Moonshot публикует другие ограничения контекста для модели, скорректируйте `contextWindow` соответствующим образом.
-   Используйте `https://api.moonshot.ai/v1` для международной конечной точки и `https://api.moonshot.cn/v1` для китайской конечной точки.

[MiniMax](./minimax.md)[Mistral](./mistral.md)