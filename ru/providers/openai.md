

  Провайдеры

  
# OpenAI

OpenAI предоставляет разработчикам API для моделей GPT. Codex поддерживает **вход через ChatGPT** для доступа по подписке или **вход по API-ключу** для доступа с оплатой по использованию. Облачный Codex требует входа через ChatGPT. OpenAI явно поддерживает использование OAuth по подписке во внешних инструментах/воркфлоу, таких как OpenClaw.

## Вариант A: API-ключ OpenAI (OpenAI Platform)

**Лучше для:** прямого доступа к API и биллинга по использованию. Получите свой API-ключ в панели управления OpenAI.

### Настройка через CLI

```bash
openclaw onboard --auth-choice openai-api-key
# или неинтерактивно
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### Фрагмент конфигурации

```json
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.4" } } },
}
```

Текущая документация OpenAI по API моделей перечисляет `gpt-5.4` и `gpt-5.4-pro` для прямого использования API OpenAI. OpenClaw перенаправляет оба через путь `openai/*` Responses.

## Вариант B: Подписка OpenAI Code (Codex)

**Лучше для:** использования доступа по подписке ChatGPT/Codex вместо API-ключа. Облачный Codex требует входа через ChatGPT, в то время как CLI Codex поддерживает вход через ChatGPT или API-ключ.

### Настройка через CLI (OAuth Codex)

```bash
# Запустите OAuth Codex в мастере
openclaw onboard --auth-choice openai-codex

# Или запустите OAuth напрямую
openclaw models auth login --provider openai-codex
```

### Фрагмент конфигурации (подписка Codex)

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.4" } } },
}
```

Текущая документация OpenAI по Codex перечисляет `gpt-5.4` как текущую модель Codex. OpenClaw сопоставляет её с `openai-codex/gpt-5.4` для использования OAuth ChatGPT/Codex.

### Транспорт по умолчанию

OpenClaw использует `pi-ai` для потоковой передачи моделей. Для обоих путей `openai/*` и `openai-codex/*` транспорт по умолчанию — `"auto"` (сначала WebSocket, затем откат на SSE). Вы можете установить `agents.defaults.models.<provider/model>.params.transport`:

-   `"sse"`: принудительно использовать SSE
-   `"websocket"`: принудительно использовать WebSocket
-   `"auto"`: попробовать WebSocket, затем откатиться на SSE

Для `openai/*` (Responses API) OpenClaw также по умолчанию включает прогрев WebSocket (`openaiWsWarmup: true`), когда используется транспорт WebSocket. Связанная документация OpenAI:

-   [Realtime API с WebSocket](https://platform.openai.com/docs/guides/realtime-websocket)
-   [Потоковые ответы API (SSE)](https://platform.openai.com/docs/guides/streaming-responses)

```json
{
  agents: {
    defaults: {
      model: { primary: "openai-codex/gpt-5.4" },
      models: {
        "openai-codex/gpt-5.4": {
          params: {
            transport: "auto",
          },
        },
      },
    },
  },
}
```

### Прогрев WebSocket OpenAI

Документация OpenAI описывает прогрев как опциональный. OpenClaw включает его по умолчанию для `openai/*`, чтобы уменьшить задержку первого хода при использовании транспорта WebSocket.

### Отключить прогрев

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: false,
          },
        },
      },
    },
  },
}
```

### Включить прогрев явно

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: true,
          },
        },
      },
    },
  },
}
```

### Приоритетная обработка OpenAI

API OpenAI предоставляет приоритетную обработку через `service_tier=priority`. В OpenClaw установите `agents.defaults.models["openai/"].params.serviceTier`, чтобы передать это поле в прямых запросах к `openai/*` Responses.

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            serviceTier: "priority",
          },
        },
      },
    },
  },
}
```

Поддерживаемые значения: `auto`, `default`, `flex` и `priority`.

### Серверная компрессия OpenAI Responses

Для прямых моделей OpenAI Responses (`openai/*`, использующих `api: "openai-responses"` с `baseUrl` на `api.openai.com`), OpenClaw теперь автоматически включает подсказки полезной нагрузки для серверной компрессии OpenAI:

-   Принудительно устанавливает `store: true` (если только совместимость модели не устанавливает `supportsStore: false`)
-   Внедряет `context_management: [{ type: "compaction", compact_threshold: ... }]`

По умолчанию `compact_threshold` составляет `70%` от `contextWindow` модели (или `80000`, если недоступно).

### Включить серверную компрессию явно

Используйте это, когда хотите принудительно внедрить `context_management` в совместимых моделях Responses (например, Azure OpenAI Responses):

```json
{
  agents: {
    defaults: {
      models: {
        "azure-openai-responses/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
          },
        },
      },
    },
  },
}
```

### Включить с пользовательским порогом

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
            responsesCompactThreshold: 120000,
          },
        },
      },
    },
  },
}
```

### Отключить серверную компрессию

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: false,
          },
        },
      },
    },
  },
}
```

`responsesServerCompaction` управляет только внедрением `context_management`. Прямые модели OpenAI Responses всё равно принудительно устанавливают `store: true`, если только совместимость не устанавливает `supportsStore: false`.

## Примечания

-   Ссылки на модели всегда используют формат `провайдер/модель` (см. [/concepts/models](../concepts/models.md)).
-   Подробности аутентификации и правила повторного использования находятся в [/concepts/oauth](../concepts/oauth.md).

[Ollama](./ollama.md)[OpenCode Zen](./opencode.md)