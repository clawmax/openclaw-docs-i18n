

  Протоколы и API

  
# Локальные модели

Локальный запуск возможен, но OpenClaw ожидает большой контекст и сильную защиту от инъекций в промпт. Малые модели обрезают контекст и ослабляют безопасность. Цельтесь высоко: **≥2 максимально укомплектованных Mac Studio или эквивалентная GPU-система (~$30k+)**. Одна видеокарта на **24 ГБ** работает только для более легких промптов с высокой задержкой. Используйте **самый большой / полноразмерный вариант модели, который вы можете запустить**; агрессивно квантованные или «маленькие» чекпоинты повышают риск инъекции в промпт (см. [Безопасность](./security.md)).

## Рекомендуется: LM Studio + MiniMax M2.5 (Responses API, полноразмерная)

Лучший текущий локальный стек. Загрузите MiniMax M2.5 в LM Studio, включите локальный сервер (по умолчанию `http://127.0.0.1:1234`) и используйте Responses API, чтобы отделить рассуждения от финального текста.

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

**Чеклист настройки**

-   Установите LM Studio: [https://lmstudio.ai](https://lmstudio.ai)
-   В LM Studio скачайте **самый большой доступный билд MiniMax M2.5** (избегайте «маленьких»/сильно квантованных вариантов), запустите сервер, убедитесь, что `http://127.0.0.1:1234/v1/models` его отображает.
-   Держите модель загруженной; холодная загрузка добавляет задержку при старте.
-   Отрегулируйте `contextWindow`/`maxTokens`, если ваш билд LM Studio отличается.
-   Для WhatsApp придерживайтесь Responses API, чтобы отправлялся только финальный текст.

Держите облачные модели сконфигурированными даже при локальном запуске; используйте `models.mode: "merge"`, чтобы резервные варианты оставались доступными.

### Гибридная конфигурация: облачная основная, локальная резервная

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

### Локальная основная с облачным резервом

Поменяйте порядок основной и резервной модели; оставьте тот же блок провайдеров и `models.mode: "merge"`, чтобы можно было переключиться на Sonnet или Opus, когда локальный сервер недоступен.

### Региональный хостинг / маршрутизация данных

-   Облачные варианты MiniMax/Kimi/GLM также существуют на OpenRouter с привязанными к регионам эндпоинтами (например, размещенные в США). Выберите региональный вариант там, чтобы трафик оставался в выбранной вами юрисдикции, при этом используя `models.mode: "merge"` для резервных вариантов от Anthropic/OpenAI.
-   Только локальный запуск остается самым надежным путем для конфиденциальности; региональная маршрутизация через облако — это компромисс, когда вам нужны функции провайдера, но вы хотите контролировать поток данных.

## Другие локальные прокси, совместимые с OpenAI

vLLM, LiteLLM, OAI-proxy или пользовательские шлюзы работают, если они предоставляют эндпоинт в стиле OpenAI `/v1`. Замените блок провайдера выше на ваш эндпоинт и ID модели:

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
            name: "Local Model",
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

Сохраняйте `models.mode: "merge"`, чтобы облачные модели оставались доступными в качестве резервных.

## Устранение неполадок

-   Шлюз может связаться с прокси? `curl http://127.0.0.1:1234/v1/models`.
-   Модель в LM Studio выгружена? Перезагрузите; холодный старт — частая причина «зависания».
-   Ошибки контекста? Уменьшите `contextWindow` или увеличьте лимит на вашем сервере.
-   Безопасность: локальные модели пропускают фильтры на стороне провайдера; держите агентов узконаправленными и включите сжатие, чтобы ограничить радиус поражения при инъекции в промпт.

[Бэкенды CLI](./cli-backends.md)[Сетевая модель](./network-model.md)