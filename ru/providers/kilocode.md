title: "Настройка провайдера Kilocode в OpenClaw: API-ключ и модели"
description: "Узнайте, как настроить провайдер Kilocode в OpenClaw, получить API-ключ и использовать унифицированный Kilo Gateway для доступа к нескольким AI-моделям."
keywords: ["kilocode", "kilo gateway", "провайдеры openclaw", "унифицированный api", "маршрутизация моделей", "настройка api-ключа", "claude sonnet", "gpt-5.2"]
---

  Провайдеры

  
# Kilocode

Kilo Gateway предоставляет **унифицированный API**, который направляет запросы ко многим моделям через единую конечную точку и API-ключ. Он совместим с OpenAI, поэтому большинство SDK OpenAI работают после смены базового URL.

## Получение API-ключа

1.  Перейдите на [app.kilo.ai](https://app.kilo.ai)
2.  Войдите в систему или создайте учетную запись
3.  Перейдите в раздел API Keys и создайте новый ключ

## Настройка через CLI

```bash
openclaw onboard --kilocode-api-key <key>
```

Или установите переменную окружения:

```bash
export KILOCODE_API_KEY="<your-kilocode-api-key>" # pragma: allowlist secret
```

## Пример конфигурации

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

## Модель по умолчанию

Модель по умолчанию — `kilocode/kilo/auto`. Это интеллектуальная модель маршрутизации, которая автоматически выбирает лучшую базовую модель в зависимости от задачи:

-   Задачи планирования, отладки и оркестрации направляются в Claude Opus
-   Задачи написания и исследования кода направляются в Claude Sonnet

## Доступные модели

OpenClaw динамически обнаруживает доступные модели из Kilo Gateway при запуске. Используйте `/models kilocode`, чтобы увидеть полный список моделей, доступных для вашей учетной записи. Любая модель, доступная на шлюзе, может быть использована с префиксом `kilocode/`:

```
kilocode/kilo/auto              (по умолчанию - интеллектуальная маршрутизация)
kilocode/anthropic/claude-sonnet-4
kilocode/openai/gpt-5.2
kilocode/google/gemini-3-pro-preview
...и многие другие
```

## Примечания

-   Ссылки на модели имеют вид `kilocode/<model-id>` (например, `kilocode/anthropic/claude-sonnet-4`).
-   Модель по умолчанию: `kilocode/kilo/auto`
-   Базовый URL: `https://api.kilo.ai/api/gateway/`
-   Для получения дополнительных опций моделей/провайдеров см. [/concepts/model-providers](../concepts/model-providers.md).
-   Kilo Gateway использует Bearer-токен с вашим API-ключом под капотом.

[Hugging Face (Inference)](./huggingface.md)[Litellm](./litellm.md)