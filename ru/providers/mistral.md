title: "Настройка провайдера Mistral AI для моделей и аудио в OpenClaw"
description: "Узнайте, как настроить Mistral AI и Voxtral в OpenClaw для генерации текста, транскрипции аудио и эмбеддингов памяти. Включает примеры CLI и конфигурации."
keywords: ["mistral ai", "openclaw", "voxtral", "транскрипция аудио", "провайдер llm", "эмбеддинги памяти", "настройка api", "mistral api key"]
---

  Провайдеры

  
# Mistral

OpenClaw поддерживает Mistral как для маршрутизации текстовых/изобразительных моделей (`mistral/...`), так и для транскрипции аудио через Voxtral в рамках понимания медиа. Mistral также можно использовать для эмбеддингов памяти (`memorySearch.provider = "mistral"`).

## Настройка через CLI

```bash
openclaw onboard --auth-choice mistral-api-key
# или неинтерактивно
openclaw onboard --mistral-api-key "$MISTRAL_API_KEY"
```

## Фрагмент конфигурации (провайдер LLM)

```json
{
  env: { MISTRAL_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "mistral/mistral-large-latest" } } },
}
```

## Фрагмент конфигурации (транскрипция аудио с Voxtral)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## Примечания

-   Аутентификация Mistral использует `MISTRAL_API_KEY`.
-   Базовый URL провайдера по умолчанию: `https://api.mistral.ai/v1`.
-   Модель по умолчанию при онбординге — `mistral/mistral-large-latest`.
-   Модель для аудио по умолчанию для Mistral в понимании медиа — `voxtral-mini-latest`.
-   Путь для транскрипции медиа использует `/v1/audio/transcriptions`.
-   Путь для эмбеддингов памяти использует `/v1/embeddings` (модель по умолчанию: `mistral-embed`).

[Moonshot AI](./moonshot.md)[NVIDIA](./nvidia.md)