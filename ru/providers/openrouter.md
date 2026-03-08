title: "Настройка и конфигурация провайдера OpenRouter для OpenClaw AI"
description: "Узнайте, как настроить и сконфигурировать провайдер OpenRouter в OpenClaw AI. Используйте единый API для доступа к множеству моделей ИИ с одним ключом API и OpenAI-совместимой конечной точкой."
keywords: ["openrouter", "openclaw ai", "api провайдер", "единый api", "маршрутизация моделей", "openai совместимый", "claude sonnet", "конфигурация ии"]
---

  Провайдеры

  
# OpenRouter

OpenRouter предоставляет **единый API**, который маршрутизирует запросы ко многим моделям через одну конечную точку и один ключ API. Он совместим с OpenAI, поэтому большинство SDK OpenAI работают после смены базового URL.

## Настройка через CLI

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## Фрагмент конфигурации

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

## Примечания

-   Ссылки на модели имеют формат `openrouter/<провайдер>/<модель>`.
-   Больше вариантов моделей и провайдеров смотрите в разделе [/concepts/model-providers](../concepts/model-providers.md).
-   OpenRouter использует Bearer-токен с вашим ключом API под капотом.

[OpenCode Zen](./opencode.md)[Qianfan](./qianfan.md)