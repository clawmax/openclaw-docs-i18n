

  Провайдеры

  
# Xiaomi MiMo

Xiaomi MiMo — это API-платформа для моделей **MiMo**. Она предоставляет REST API, совместимые с форматами OpenAI и Anthropic, и использует для аутентификации API-ключи. Создайте свой API-ключ в [консоли Xiaomi MiMo](https://platform.xiaomimimo.com/#/console/api-keys). OpenClaw использует провайдер `xiaomi` с API-ключом Xiaomi MiMo.

## Обзор модели

-   **mimo-v2-flash**: контекстное окно на 262144 токена, совместимо с Anthropic Messages API.
-   Базовый URL: `https://api.xiaomimimo.com/anthropic`
-   Авторизация: `Bearer $XIAOMI_API_KEY`

## Настройка через CLI

```bash
openclaw onboard --auth-choice xiaomi-api-key
# или неинтерактивно
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

## Фрагмент конфигурации

```json
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Примечания

-   Ссылка на модель: `xiaomi/mimo-v2-flash`.
-   Провайдер внедряется автоматически, когда установлена переменная `XIAOMI_API_KEY` (или существует профиль аутентификации).
-   См. [/concepts/model-providers](../concepts/model-providers.md) для правил работы с провайдерами.

[vLLM](./vllm.md)[Z.AI](./zai.md)