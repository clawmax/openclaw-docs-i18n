title: "Настройка и конфигурация провайдера OpenClaw Vercel AI Gateway"
description: "Узнайте, как настроить OpenClaw с Vercel AI Gateway. Настройте ваш API-ключ, определите модели по умолчанию, такие как Claude Opus, и используйте сокращённые имена моделей для унифицированного доступа к ИИ."
keywords: ["vercel ai gateway", "настройка openclaw", "claude opus", "api ключ ai gateway", "сокращённые имена моделей", "anthropic messages api", "провайдеры openclaw", "интеграция моделей ии"]
---

  Провайдеры

  
# Vercel AI Gateway

[Vercel AI Gateway](https://vercel.com/ai-gateway) предоставляет унифицированный API для доступа к сотням моделей через единую конечную точку.

-   Провайдер: `vercel-ai-gateway`
-   Аутентификация: `AI_GATEWAY_API_KEY`
-   API: Совместим с Anthropic Messages

## Быстрый старт

1.  Установите API-ключ (рекомендуется: сохранить его для Gateway):

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2.  Установите модель по умолчанию:

```json
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.6" },
    },
  },
}
```

## Пример неинтерактивной настройки

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## Примечание по окружению

Если Gateway работает как демон (launchd/systemd), убедитесь, что `AI_GATEWAY_API_KEY` доступен этому процессу (например, в `~/.openclaw/.env` или через `env.shellEnv`).

## Сокращённые идентификаторы моделей

OpenClaw принимает сокращённые ссылки на модели Vercel Claude и нормализует их во время выполнения:

-   `vercel-ai-gateway/claude-opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   `vercel-ai-gateway/opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4-6`

[Together](./together.md)[Venice AI](./venice.md)