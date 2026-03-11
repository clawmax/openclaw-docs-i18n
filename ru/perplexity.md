

  Встроенные инструменты

  
# Perplexity Sonar

OpenClaw может использовать Perplexity Sonar для инструмента `web_search`. Вы можете подключиться через прямой API Perplexity или через OpenRouter.

## Варианты API

### Perplexity (прямой)

-   Базовый URL: [https://api.perplexity.ai](https://api.perplexity.ai)
-   Переменная окружения: `PERPLEXITY_API_KEY`

### OpenRouter (альтернатива)

-   Базовый URL: [https://openrouter.ai/api/v1](https://openrouter.ai/api/v1)
-   Переменная окружения: `OPENROUTER_API_KEY`
-   Поддерживает предоплату/криптовалютные кредиты.

## Пример конфигурации

```json
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

## Переход с Brave

```json
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
        },
      },
    },
  },
}
```

Если установлены обе переменные `PERPLEXITY_API_KEY` и `OPENROUTER_API_KEY`, укажите `tools.web.search.perplexity.baseUrl` (или `tools.web.search.perplexity.apiKey`), чтобы устранить неоднозначность. Если базовый URL не задан, OpenClaw выбирает значение по умолчанию на основе источника ключа API:

-   `PERPLEXITY_API_KEY` или `pplx-...` → прямой Perplexity (`https://api.perplexity.ai`)
-   `OPENROUTER_API_KEY` или `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
-   Неизвестный формат ключа → OpenRouter (безопасный запасной вариант)

## Модели

-   `perplexity/sonar` — быстрые вопросы и ответы с веб-поиском
-   `perplexity/sonar-pro` (по умолчанию) — многошаговые рассуждения + веб-поиск
-   `perplexity/sonar-reasoning-pro` — глубокое исследование

См. [Веб-инструменты](./tools/web.md) для полной конфигурации web\_search.

[Поиск Brave](./brave-search.md)[Различия](./tools/diffs.md)