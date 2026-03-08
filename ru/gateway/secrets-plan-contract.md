

  Конфигурация и операции

  
# Контракт плана применения секретов

На этой странице определяется строгий контракт, применяемый командой `openclaw secrets apply`. Если цель не соответствует этим правилам, применение завершается ошибкой до изменения конфигурации.

## Структура файла плана

`openclaw secrets apply --from <plan.json>` ожидает массив `targets` с целевыми объектами плана:

```json
{
  version: 1,
  protocolVersion: 1,
  targets: [
    {
      type: "models.providers.apiKey",
      path: "models.providers.openai.apiKey",
      pathSegments: ["models", "providers", "openai", "apiKey"],
      providerId: "openai",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
    {
      type: "auth-profiles.api_key.key",
      path: "profiles.openai:default.key",
      pathSegments: ["profiles", "openai:default", "key"],
      agentId: "main",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
  ],
}
```

## Поддерживаемая область действия целей

Цели плана принимаются для поддерживаемых путей учетных данных в:

-   [Поверхность учетных данных SecretRef](../reference/secretref-credential-surface.md)

## Поведение типа цели

Общее правило:

-   `target.type` должен быть распознан и должен соответствовать нормализованной форме `target.path`.

Псевдонимы для совместимости остаются принятыми для существующих планов:

-   `models.providers.apiKey`
-   `skills.entries.apiKey`
-   `channels.googlechat.serviceAccount`

## Правила валидации пути

Каждая цель проверяется по всем следующим критериям:

-   `type` должен быть распознанным типом цели.
-   `path` должен быть непустым путем в точечной нотации.
-   `pathSegments` может быть опущен. Если предоставлен, он должен нормализоваться точно в тот же путь, что и `path`.
-   Запрещенные сегменты отклоняются: `__proto__`, `prototype`, `constructor`.
-   Нормализованный путь должен соответствовать зарегистрированной форме пути для данного типа цели.
-   Если задан `providerId` или `accountId`, он должен совпадать с идентификатором, закодированным в пути.
-   Для целей `auth-profiles.json` требуется `agentId`.
-   При создании нового сопоставления в `auth-profiles.json` укажите `authProfileProvider`.

## Поведение при ошибке

Если цель не проходит валидацию, применение завершается с ошибкой, например:

```bash
Invalid plan target path for models.providers.apiKey: models.providers.openai.baseUrl
```

Для невалидного плана никакие записи не фиксируются.

## Примечания по области действия выполнения и аудита

-   Записи `auth-profiles.json`, содержащие только ссылки (`keyRef`/`tokenRef`), включаются в разрешение во время выполнения и охват аудита.
-   `secrets apply` записывает поддерживаемые цели `openclaw.json`, поддерживаемые цели `auth-profiles.json` и опциональные цели очистки.

## Проверки оператора

```bash
# Проверить план без записи
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run

# Затем применить по-настоящему
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
```

Если применение завершается ошибкой с сообщением о недопустимом пути цели, пересоздайте план с помощью `openclaw secrets configure` или исправьте путь цели на поддерживаемую форму, указанную выше.

## Связанная документация

-   [Управление секретами](./secrets.md)
-   [CLI `secrets`](../cli/secrets.md)
-   [Поверхность учетных данных SecretRef](../reference/secretref-credential-surface.md)
-   [Справочник по конфигурации](./configuration-reference.md)

[Управление секретами](./secrets.md)[Аутентификация через доверенный прокси](./trusted-proxy-auth.md)