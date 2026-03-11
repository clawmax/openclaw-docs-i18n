

  Команды CLI

  
# onboard

Интерактивный мастер адаптации (локальная или удаленная настройка Шлюза).

## Связанные руководства

-   Центр адаптации CLI: [Мастер адаптации (CLI)](../start/wizard.md)
-   Обзор адаптации: [Обзор адаптации](../start/onboarding-overview.md)
-   Справочник по адаптации CLI: [Справочник по адаптации CLI](../start/wizard-cli-reference.md)
-   Автоматизация CLI: [Автоматизация CLI](../start/wizard-cli-automation.md)
-   Адаптация для macOS: [Адаптация (Приложение для macOS)](../start/onboarding.md)

## Примеры

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url wss://gateway-host:18789
```

Для целей с открытым текстом в частной сети `ws://` (только доверенные сети) установите `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` в окружении процесса адаптации. Неинтерактивный пользовательский провайдер:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --secret-input-mode plaintext \
  --custom-compatibility openai
```

`--custom-api-key` является необязательным в неинтерактивном режиме. Если опущено, адаптация проверяет `CUSTOM_API_KEY`. Храните ключи провайдеров как ссылки вместо открытого текста:

```bash
openclaw onboard --non-interactive \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

При использовании `--secret-input-mode ref` адаптация записывает ссылки, основанные на переменных окружения, вместо значений ключей в открытом тексте. Для провайдеров на основе профиля аутентификации это записывает записи `keyRef`; для пользовательских провайдеров это записывает `models.providers..apiKey` как ссылку на переменную окружения (например, `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`). Контракт неинтерактивного режима `ref`:

-   Установите переменную окружения провайдера в окружении процесса адаптации (например, `OPENAI_API_KEY`).
-   Не передавайте встроенные флаги ключей (например, `--openai-api-key`), если эта переменная окружения также не установлена.
-   Если встроенный флаг ключа передан без требуемой переменной окружения, адаптация немедленно завершается сбоем с инструкциями.

Варианты токена Шлюза в неинтерактивном режиме:

-   `--gateway-auth token --gateway-token ` сохраняет токен в открытом тексте.
-   `--gateway-auth token --gateway-token-ref-env ` сохраняет `gateway.auth.token` как SecretRef на основе переменной окружения.
-   `--gateway-token` и `--gateway-token-ref-env` являются взаимоисключающими.
-   `--gateway-token-ref-env` требует непустую переменную окружения в окружении процесса адаптации.
-   При использовании `--install-daemon`, когда аутентификация по токену требует токена, токены Шлюза, управляемые через SecretRef, проверяются, но не сохраняются в виде раскрытого открытого текста в метаданных окружения службы супервизора.
-   При использовании `--install-daemon`, если режим токена требует токена, а настроенная ссылка SecretRef не разрешена, адаптация завершается сбоем с инструкциями по исправлению.
-   При использовании `--install-daemon`, если настроены и `gateway.auth.token`, и `gateway.auth.password`, а `gateway.auth.mode` не установлен, адаптация блокирует установку до явного указания режима.

Пример:

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN \
  --accept-risk
```

Поведение интерактивной адаптации с режимом ссылок:

-   Выберите **Использовать ссылку на секрет** при запросе.
-   Затем выберите либо:
    -   Переменную окружения
    -   Настроенный провайдер секретов (`file` или `exec`)
-   Адаптация выполняет быструю предварительную проверку перед сохранением ссылки.
    -   Если проверка не удалась, адаптация показывает ошибку и позволяет повторить попытку.

Выбор конечной точки Z.AI в неинтерактивном режиме: Примечание: `--auth-choice zai-api-key` теперь автоматически определяет лучшую конечную точку Z.AI для вашего ключа (предпочитает общий API с `zai/glm-5`). Если вам нужны конечные точки GLM Coding Plan, выберите `zai-coding-global` или `zai-coding-cn`.

```bash
# Выбор конечной точки без запроса
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Другие варианты конечных точек Z.AI:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Неинтерактивный пример для Mistral:

```bash
openclaw onboard --non-interactive \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY"
```

Примечания по потокам:

-   `quickstart`: минимальные запросы, автоматически генерирует токен шлюза.
-   `manual`: полные запросы для порта/привязки/аутентификации (псевдоним `advanced`).
-   Поведение области DM при локальной адаптации: [Справочник по адаптации CLI](../start/wizard-cli-reference.md#outputs-and-internals).
-   Самый быстрый первый чат: `openclaw dashboard` (UI управления, без настройки каналов).
-   Пользовательский провайдер: подключите любую конечную точку, совместимую с OpenAI или Anthropic, включая размещенные провайдеры, не указанные в списке. Используйте Unknown для автоматического определения.

## Часто используемые последующие команды

```bash
openclaw configure
openclaw agents add <name>
```

> **ℹ️** `--json` не подразумевает неинтерактивный режим. Используйте `--non-interactive` для скриптов.

[nodes](./nodes.md)[pairing](./pairing.md)

---