

  Руководства

  
# Автоматизация CLI

Используйте `--non-interactive` для автоматизации `openclaw onboard`.

> **ℹ️** `--json` не подразумевает неинтерактивный режим. Используйте `--non-interactive` (и `--workspace`) для скриптов.

## Базовый пример неинтерактивного режима

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --secret-input-mode plaintext \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Добавьте `--json` для получения машиночитаемой сводки. Используйте `--secret-input-mode ref`, чтобы хранить ссылки на переменные окружения в профилях аутентификации вместо открытых значений. Интерактивный выбор между ссылками на переменные окружения и настроенными ссылками провайдера (`file` или `exec`) доступен в потоке мастера подключения. В неинтерактивном режиме `ref` переменные окружения провайдера должны быть установлены в окружении процесса. Передача встроенных ключей через флаги без соответствующей переменной окружения теперь приводит к быстрому завершению с ошибкой. Пример:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

## Примеры для конкретных провайдеров

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-provider-id "my-custom" \
  --custom-compatibility anthropic \
  --gateway-port 18789 \
  --gateway-bind loopback
```

`--custom-api-key` является необязательным. Если он опущен, мастер подключения проверяет `CUSTOM_API_KEY`.Вариант с режимом ref:

```bash
export CUSTOM_API_KEY="your-key"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --secret-input-mode ref \
  --custom-provider-id "my-custom" \
  --custom-compatibility anthropic \
  --gateway-port 18789 \
  --gateway-bind loopback
```

В этом режиме мастер подключения сохраняет `apiKey` как `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`.

## Добавление другого агента

Используйте `openclaw agents add <имя>`, чтобы создать отдельного агента с его собственным рабочим пространством, сессиями и профилями аутентификации. Запуск без `--workspace` запускает мастер.

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

Что он устанавливает:

-   `agents.list[].name`
-   `agents.list[].workspace`
-   `agents.list[].agentDir`

Примечания:

-   Рабочие пространства по умолчанию следуют шаблону `~/.openclaw/workspace-`.
-   Добавьте `bindings` для маршрутизации входящих сообщений (мастер может это сделать).
-   Флаги неинтерактивного режима: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## Связанная документация

-   Центр подключения: [Мастер подключения (CLI)](./wizard.md)
-   Полный справочник: [Справочник по подключению через CLI](./wizard-cli-reference.md)
-   Справочник команд: [`openclaw onboard`](../cli/onboard.md)

[Справочник CLI](./wizard-cli-reference.md)