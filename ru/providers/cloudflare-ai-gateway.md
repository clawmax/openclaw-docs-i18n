

  Провайдеры

  
# Cloudflare AI Gateway

Cloudflare AI Gateway располагается перед API провайдеров и позволяет добавлять аналитику, кэширование и элементы управления. Для Anthropic OpenClaw использует Anthropic Messages API через вашу конечную точку Gateway.

-   Провайдер: `cloudflare-ai-gateway`
-   Базовый URL: `https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic`
-   Модель по умолчанию: `cloudflare-ai-gateway/claude-sonnet-4-5`
-   API-ключ: `CLOUDFLARE_AI_GATEWAY_API_KEY` (ваш API-ключ провайдера для запросов через Gateway)

Для моделей Anthropic используйте ваш API-ключ Anthropic.

## Быстрый старт

1.  Установите API-ключ провайдера и данные Gateway:

```bash
openclaw onboard --auth-choice cloudflare-ai-gateway-api-key
```

2.  Установите модель по умолчанию:

```json
{
  agents: {
    defaults: {
      model: { primary: "cloudflare-ai-gateway/claude-sonnet-4-5" },
    },
  },
}
```

## Неинтерактивный пример

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY"
```

## Аутентифицированные шлюзы

Если вы включили аутентификацию Gateway в Cloudflare, добавьте заголовок `cf-aig-authorization` (это дополнение к вашему API-ключу провайдера).

```json
{
  models: {
    providers: {
      "cloudflare-ai-gateway": {
        headers: {
          "cf-aig-authorization": "Bearer <cloudflare-ai-gateway-token>",
        },
      },
    },
  },
}
```

## Примечание по окружению

Если Gateway работает как демон (launchd/systemd), убедитесь, что `CLOUDFLARE_AI_GATEWAY_API_KEY` доступен этому процессу (например, в `~/.openclaw/.env` или через `env.shellEnv`).

[Amazon Bedrock](./bedrock.md)[Claude Max API Proxy](./claude-max-api-proxy.md)