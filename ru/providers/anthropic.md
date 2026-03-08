title: "Настройка провайдера OpenClaw Anthropic: API-ключ и токен Claude"
description: "Узнайте, как аутентифицировать OpenClaw в Anthropic с помощью API-ключа или setup-token от Claude. Настройте кэширование промптов, 1M контекстное окно и устраните распространённые проблемы."
keywords: ["anthropic", "claude api", "настройка openclaw", "кэширование промптов", "setup-token", "1m контекстное окно", "claude 4.6", "bedrock claude"]
---

  Провайдеры

  
# Anthropic

Anthropic создаёт семейство моделей **Claude** и предоставляет доступ через API. В OpenClaw вы можете аутентифицироваться с помощью API-ключа или **setup-token**.

## Вариант A: API-ключ Anthropic

**Лучше для:** стандартного доступа к API и биллинга по использованию. Создайте свой API-ключ в Anthropic Console.

### Настройка через CLI

```bash
openclaw onboard
# выбрать: Anthropic API key

# или неинтерактивно
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### Фрагмент конфигурации

```json
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Настройки мышления по умолчанию (Claude 4.6)

-   Модели Anthropic Claude 4.6 по умолчанию используют `adaptive` мышление в OpenClaw, когда явный уровень мышления не задан.
-   Вы можете переопределить это для каждого сообщения (`/think:`) или в параметрах модели: `agents.defaults.models["anthropic/"].params.thinking`.
-   Связанная документация Anthropic:
    -   [Адаптивное мышление](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
    -   [Расширенное мышление](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

## Кэширование промптов (API Anthropic)

OpenClaw поддерживает функцию кэширования промптов от Anthropic. Это работает **только для API**; аутентификация по подписке не учитывает настройки кэша.

### Конфигурация

Используйте параметр `cacheRetention` в конфигурации модели:

| Значение | Длительность кэша | Описание |
| --- | --- | --- |
| `none` | Без кэширования | Отключить кэширование промптов |
| `short` | 5 минут | По умолчанию для аутентификации по API-ключу |
| `long` | 1 час | Расширенный кэш (требуется бета-флаг) |

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

### Значения по умолчанию

При использовании аутентификации по API-ключу Anthropic, OpenClaw автоматически применяет `cacheRetention: "short"` (5-минутный кэш) для всех моделей Anthropic. Вы можете переопределить это, явно задав `cacheRetention` в своей конфигурации.

### Переопределение cacheRetention для конкретного агента

Используйте параметры на уровне модели как базовые, затем переопределите для конкретных агентов через `agents.list[].params`.

```json
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" }, // базовое значение для большинства агентов
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } }, // переопределение только для этого агента
    ],
  },
}
```

Порядок слияния конфигурации для параметров кэширования:

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params` (совпадающий `id`, переопределяет по ключу)

Это позволяет одному агенту использовать долгоживущий кэш, в то время как другой агент на той же модели отключает кэширование, чтобы избежать затрат на запись при всплесках трафика или низком повторном использовании.

### Примечания для Bedrock Claude

-   Модели Anthropic Claude на Bedrock (`amazon-bedrock/*anthropic.claude*`) принимают сквозную передачу параметра `cacheRetention` при настройке.
-   Для моделей Bedrock, не от Anthropic, в runtime принудительно устанавливается `cacheRetention: "none"`.
-   Умные настройки по умолчанию для API-ключа Anthropic также устанавливают `cacheRetention: "short"` для ссылок на модели Claude-on-Bedrock, когда явное значение не задано.

### Устаревший параметр

Старый параметр `cacheControlTtl` всё ещё поддерживается для обратной совместимости:

-   `"5m"` соответствует `short`
-   `"1h"` соответствует `long`

Мы рекомендуем перейти на новый параметр `cacheRetention`. OpenClaw включает бета-флаг `extended-cache-ttl-2025-04-11` для запросов к Anthropic API; оставьте его, если вы переопределяете заголовки провайдера (см. [/gateway/configuration](../gateway/configuration.md)).

## Контекстное окно 1M (бета Anthropic)

Контекстное окно 1M от Anthropic находится в бета-доступе. В OpenClaw включите его для каждой модели с помощью `params.context1m: true` для поддерживаемых моделей Opus/Sonnet.

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { context1m: true },
        },
      },
    },
  },
}
```

OpenClaw преобразует это в `anthropic-beta: context-1m-2025-08-07` для запросов к Anthropic. Это активируется только когда `params.context1m` явно установлен в `true` для этой модели. Требование: Anthropic должен разрешать использование длинного контекста для этих учётных данных (обычно биллинг по API-ключу или аккаунт подписки с включённым Extra Usage). В противном случае Anthropic вернёт: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`. Примечание: Anthropic в настоящее время отклоняет бета-запросы `context-1m-*` при использовании OAuth/токенов подписки (`sk-ant-oat-*`). OpenClaw автоматически пропускает бета-заголовок context1m для OAuth-аутентификации и сохраняет необходимые OAuth-бета-флаги.

## Вариант B: setup-token Claude

**Лучше для:** использования вашей подписки Claude.

### Где взять setup-token

Setup-token создаётся **Claude Code CLI**, а не в Anthropic Console. Вы можете запустить это на **любой машине**:

```bash
claude setup-token
```

Вставьте токен в OpenClaw (мастер: **Anthropic token (paste setup-token)**) или запустите на хосте шлюза:

```bash
openclaw models auth setup-token --provider anthropic
```

Если вы сгенерировали токен на другой машине, вставьте его:

```bash
openclaw models auth paste-token --provider anthropic
```

### Настройка через CLI (setup-token)

```bash
# Вставьте setup-token во время онбординга
openclaw onboard --auth-choice setup-token
```

### Фрагмент конфигурации (setup-token)

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Примечания

-   Сгенерируйте setup-token с помощью `claude setup-token` и вставьте его, или запустите `openclaw models auth setup-token` на хосте шлюза.
-   Если вы видите “OAuth token refresh failed …” для подписки Claude, выполните повторную аутентификацию с помощью setup-token. См. [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](../gateway/troubleshooting.md#oauth-token-refresh-failed-anthropic-claude-subscription).
-   Подробности аутентификации и правила повторного использования описаны в [/concepts/oauth](../concepts/oauth.md).

## Устранение неполадок

**Ошибки 401 / токен внезапно стал недействительным**

-   Аутентификация по подписке Claude может истечь или быть отозвана. Повторно запустите `claude setup-token` и вставьте токен на **хосте шлюза**.
-   Если вход в Claude CLI выполнен на другой машине, используйте `openclaw models auth paste-token --provider anthropic` на хосте шлюза.

**No API key found for provider “anthropic”**

-   Аутентификация **привязана к агенту**. Новые агенты не наследуют ключи основного агента.
-   Повторно запустите онбординг для этого агента или вставьте setup-token / API-ключ на хосте шлюза, затем проверьте с помощью `openclaw models status`.

**No credentials found for profile `anthropic:default`**

-   Запустите `openclaw models status`, чтобы увидеть, какой профиль аутентификации активен.
-   Повторно запустите онбординг или вставьте setup-token / API-ключ для этого профиля.

**No available auth profile (all in cooldown/unavailable)**

-   Проверьте `openclaw models status --json` на наличие `auth.unusableProfiles`.
-   Добавьте другой профиль Anthropic или дождитесь окончания времени охлаждения.

Подробнее: [/gateway/troubleshooting](../gateway/troubleshooting.md) и [/help/faq](../help/faq.md).

[Резервирование моделей](../concepts/model-failover.md)[Amazon Bedrock](./bedrock.md)