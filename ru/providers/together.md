

  Провайдеры

  
# Together

[Together AI](https://together.ai) предоставляет доступ к ведущим open-source моделям, включая Llama, DeepSeek, Kimi и другие, через единый API.

-   Провайдер: `together`
-   Аутентификация: `TOGETHER_API_KEY`
-   API: Совместимый с OpenAI

## Быстрый старт

1.  Установите API-ключ (рекомендуется: сохранить его для Шлюза):

```bash
openclaw onboard --auth-choice together-api-key
```

2.  Установите модель по умолчанию:

```json
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Пример неинтерактивной настройки

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Это установит `together/moonshotai/Kimi-K2.5` в качестве модели по умолчанию.

## Примечание по окружению

Если Шлюз работает как демон (launchd/systemd), убедитесь, что `TOGETHER_API_KEY` доступен этому процессу (например, в `~/.openclaw/.env` или через `env.shellEnv`).

## Доступные модели

Together AI предоставляет доступ ко многим популярным open-source моделям:

-   **GLM 4.7 Fp8** - Модель по умолчанию с контекстным окном 200K
-   **Llama 3.3 70B Instruct Turbo** - Быстрая и эффективная модель для следования инструкциям
-   **Llama 4 Scout** - Модель с поддержкой зрения (понимание изображений)
-   **Llama 4 Maverick** - Продвинутая модель для зрения и логических рассуждений
-   **DeepSeek V3.1** - Мощная модель для программирования и логических рассуждений
-   **DeepSeek R1** - Продвинутая модель для логических рассуждений
-   **Kimi K2 Instruct** - Высокопроизводительная модель с контекстным окном 262K

Все модели поддерживают стандартные чат-завершения и совместимы с OpenAI API.

[Synthetic](./synthetic.md)[Vercel AI Gateway](./vercel-ai-gateway.md)