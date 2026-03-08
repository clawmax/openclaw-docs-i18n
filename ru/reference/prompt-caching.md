

  Технический справочник

  
# Кэширование промптов

Кэширование промптов означает, что провайдер модели может повторно использовать неизмененные префиксы промптов (обычно системные/разработческие инструкции и другой стабильный контекст) между запросами, вместо того чтобы обрабатывать их заново каждый раз. Первый соответствующий запрос записывает токены кэша (`cacheWrite`), а последующие соответствующие запросы могут их читать (`cacheRead`). Почему это важно: снижение стоимости токенов, более быстрые ответы и более предсказуемая производительность для длительных сессий. Без кэширования повторяющиеся промпты оплачивают полную стоимость промпта при каждом запросе, даже если большая часть ввода не изменилась. На этой странице рассматриваются все связанные с кэшем настройки, влияющие на повторное использование промптов и стоимость токенов. Подробности о ценах Anthropic см.: [https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

## Основные настройки

### cacheRetention (модель и на уровне агента)

Установите удержание кэша в параметрах модели:

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "short" # none | short | long
```

Переопределение на уровне агента:

```yaml
agents:
  list:
    - id: "alerts"
      params:
        cacheRetention: "none"
```

Порядок слияния конфигурации:

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params` (соответствующий id агента; переопределяет по ключу)

### Устаревший cacheControlTtl

Устаревшие значения по-прежнему принимаются и преобразуются:

-   `5m` -> `short`
-   `1h` -> `long`

Для новой конфигурации предпочтительнее использовать `cacheRetention`.

### contextPruning.mode: "cache-ttl"

Очищает старый контекст результатов инструментов после окон TTL кэша, чтобы запросы после простоя не кэшировали заново разросшуюся историю.

```yaml
agents:
  defaults:
    contextPruning:
      mode: "cache-ttl"
      ttl: "1h"
```

Полное описание поведения см. в разделе [Очистка сессии](../concepts/session-pruning.md).

### Подогрев через Heartbeat

Heartbeat может поддерживать окна кэша в теплом состоянии и уменьшать повторные записи в кэш после периодов простоя.

```yaml
agents:
  defaults:
    heartbeat:
      every: "55m"
```

Heartbeat на уровне агента поддерживается в `agents.list[].heartbeat`.

## Поведение провайдеров

### Anthropic (прямой API)

-   `cacheRetention` поддерживается.
-   При использовании профилей аутентификации с ключом API Anthropic, OpenClaw автоматически устанавливает `cacheRetention: "short"` для ссылок на модели Anthropic, если значение не задано.

### Amazon Bedrock

-   Ссылки на модели Anthropic Claude (`amazon-bedrock/*anthropic.claude*`) поддерживают явную передачу `cacheRetention`.
-   Для моделей Bedrock, не относящихся к Anthropic, во время выполнения принудительно устанавливается `cacheRetention: "none"`.

### Модели Anthropic через OpenRouter

Для ссылок на модели `openrouter/anthropic/*` OpenClaw внедряет `cache_control` Anthropic в блоки системных/разработческих промптов для улучшения повторного использования кэша промптов.

### Другие провайдеры

Если провайдер не поддерживает этот режим кэширования, `cacheRetention` не оказывает эффекта.

## Шаблоны настройки

### Смешанный трафик (рекомендуемый по умолчанию)

Сохраняйте долгоживущую базовую настройку для вашего основного агента, отключите кэширование для агентов с всплесками уведомлений:

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m"
    - id: "alerts"
      params:
        cacheRetention: "none"
```

### Базовый подход с упором на стоимость

-   Установите базовое значение `cacheRetention: "short"`.
-   Включите `contextPruning.mode: "cache-ttl"`.
-   Поддерживайте heartbeat ниже вашего TTL только для агентов, которым выгоден теплый кэш.

## Диагностика кэша

OpenClaw предоставляет специальную диагностику трассировки кэша для встроенных запусков агентов.

### Конфигурация diagnostics.cacheTrace

```yaml
diagnostics:
  cacheTrace:
    enabled: true
    filePath: "~/.openclaw/logs/cache-trace.jsonl" # опционально
    includeMessages: false # по умолчанию true
    includePrompt: false # по умолчанию true
    includeSystem: false # по умолчанию true
```

Значения по умолчанию:

-   `filePath`: `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`
-   `includeMessages`: `true`
-   `includePrompt`: `true`
-   `includeSystem`: `true`

### Переключатели окружения (для разовой отладки)

-   `OPENCLAW_CACHE_TRACE=1` включает трассировку кэша.
-   `OPENCLAW_CACHE_TRACE_FILE=/path/to/cache-trace.jsonl` переопределяет путь вывода.
-   `OPENCLAW_CACHE_TRACE_MESSAGES=0|1` переключает захват полной полезной нагрузки сообщений.
-   `OPENCLAW_CACHE_TRACE_PROMPT=0|1` переключает захват текста промпта.
-   `OPENCLAW_CACHE_TRACE_SYSTEM=0|1` переключает захват системного промпта.

### Что проверять

-   События трассировки кэша представлены в формате JSONL и включают снимки состояний, такие как `session:loaded`, `prompt:before`, `stream:context` и `session:after`.
-   Влияние токенов кэша на каждый запрос видно в обычных интерфейсах использования через `cacheRead` и `cacheWrite` (например, `/usage full` и сводки использования сессии).

## Быстрое устранение неполадок

-   Высокий `cacheWrite` при большинстве запросов: проверьте наличие изменчивых входных данных системного промпта и убедитесь, что модель/провайдер поддерживает ваши настройки кэша.
-   Отсутствие эффекта от `cacheRetention`: убедитесь, что ключ модели соответствует `agents.defaults.models["provider/model"]`.
-   Запросы к Bedrock Nova/Mistral с настройками кэша: ожидаемое принудительное установление `none` во время выполнения.

Связанная документация:

-   [Anthropic](../providers/anthropic.md)
-   [Использование токенов и стоимость](./token-use.md)
-   [Очистка сессии](../concepts/session-pruning.md)
-   [Справочник по конфигурации шлюза](../gateway/configuration-reference.md)

[Поверхность учетных данных SecretRef](./secretref-credential-surface.md)[Использование API и стоимость](./api-usage-costs.md)