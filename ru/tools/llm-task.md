

  Встроенные инструменты

  
# LLM Task

`llm-task` — это **опциональный инструмент-плагин**, который выполняет LLM-задачу только с JSON-выводом и возвращает структурированный результат (опционально проверенный на соответствие JSON Schema). Это идеально подходит для движков рабочих процессов, таких как Lobster: вы можете добавить один шаг LLM без написания пользовательского кода OpenClaw для каждого рабочего процесса.

## Включение плагина

1.  Включите плагин:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2.  Добавьте инструмент в разрешённый список (он регистрируется с `optional: true`):

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

## Конфигурация (опционально)

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.4",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.4"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` — это разрешённый список строк вида `провайдер/модель`. Если он задан, любой запрос вне этого списка отклоняется.

## Параметры инструмента

-   `prompt` (строка, обязательный)
-   `input` (любой, опционально)
-   `schema` (объект, опционально JSON Schema)
-   `provider` (строка, опционально)
-   `model` (строка, опционально)
-   `authProfileId` (строка, опционально)
-   `temperature` (число, опционально)
-   `maxTokens` (число, опционально)
-   `timeoutMs` (число, опционально)

## Вывод

Возвращает `details.json`, содержащий распарсенный JSON (и проверяет его на соответствие `schema`, если она предоставлена).

## Пример: шаг рабочего процесса Lobster

```
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

## Примечания по безопасности

-   Инструмент работает **только с JSON** и инструктирует модель выводить только JSON (без блоков кода, без комментариев).
-   Для этого запуска модели не предоставляется доступ к каким-либо инструментам.
-   Считайте вывод ненадёжным, если вы не проверили его с помощью `schema`.
-   Размещайте этапы подтверждения перед любым шагом с побочными эффектами (отправка, публикация, выполнение).

[Firecrawl](./firecrawl.md)[Lobster](./lobster.md)