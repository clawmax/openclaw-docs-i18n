title: "Руководство по настройке и конфигурации интеграции OpenClaw с LiteLLM"
description: "Узнайте, как маршрутизировать OpenClaw через LiteLLM для централизованного отслеживания затрат, маршрутизации моделей и логирования. Настройте провайдеров и управляйте виртуальными ключами."
keywords: ["litellm", "openclaw", "маршрутизация моделей", "отслеживание затрат", "llm шлюз", "виртуальные ключи", "openai-совместимый", "интеграция api"]
---

  Провайдеры

  
# Litellm

[LiteLLM](https://litellm.ai) — это шлюз LLM с открытым исходным кодом, предоставляющий унифицированный API для 100+ провайдеров моделей. Маршрутизируйте OpenClaw через LiteLLM, чтобы получить централизованное отслеживание затрат, логирование и гибкость смены бэкендов без изменения конфигурации OpenClaw.

## Зачем использовать LiteLLM с OpenClaw?

-   **Отслеживание затрат** — Точно видите, на что OpenClaw тратит средства для всех моделей
-   **Маршрутизация моделей** — Переключайтесь между Claude, GPT-4, Gemini, Bedrock без изменений конфигурации
-   **Виртуальные ключи** — Создавайте ключи с лимитами расходов для OpenClaw
-   **Логирование** — Полные логи запросов и ответов для отладки
-   **Резервные варианты** — Автоматический переход на запасной вариант, если ваш основной провайдер недоступен

## Быстрый старт

### Через онбординг

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Ручная настройка

1.  Запустите LiteLLM Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2.  Направьте OpenClaw на LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

Всё готово. Теперь OpenClaw маршрутизируется через LiteLLM.

## Конфигурация

### Переменные окружения

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Файл конфигурации

```json
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## Виртуальные ключи

Создайте выделенный ключ для OpenClaw с лимитами расходов:

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

Используйте сгенерированный ключ как `LITELLM_API_KEY`.

## Маршрутизация моделей

LiteLLM может направлять запросы к моделям на разные бэкенды. Настройте в вашем `config.yaml` LiteLLM:

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw продолжает запрашивать `claude-opus-4-6` — LiteLLM обрабатывает маршрутизацию.

## Просмотр использования

Проверьте дашборд или API LiteLLM:

```bash
# Информация о ключе
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Логи расходов
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Примечания

-   LiteLLM по умолчанию работает на `http://localhost:4000`
-   OpenClaw подключается через OpenAI-совместимый эндпоинт `/v1/chat/completions`
-   Все функции OpenClaw работают через LiteLLM — без ограничений

## Смотрите также

-   [Документация LiteLLM](https://docs.litellm.ai)
-   [Провайдеры моделей](../concepts/model-providers.md)

[Kilocode](./kilocode.md)[GLM Models](./glm.md)