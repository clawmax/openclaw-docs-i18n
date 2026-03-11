

  Провайдеры

  
# NVIDIA

NVIDIA предоставляет OpenAI-совместимый API по адресу `https://integrate.api.nvidia.com/v1` для моделей Nemotron и NeMo. Для аутентификации используйте API-ключ с [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## Настройка через CLI

Экспортируйте ключ один раз, затем выполните онбординг и установите модель NVIDIA:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Если вы всё ещё передаёте `--token`, помните, что он попадает в историю оболочки и вывод `ps`; по возможности предпочтительнее использовать переменную окружения.

## Фрагмент конфигурации

```json
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## Идентификаторы моделей

-   `nvidia/llama-3.1-nemotron-70b-instruct` (по умолчанию)
-   `meta/llama-3.3-70b-instruct`
-   `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Примечания

-   OpenAI-совместимая конечная точка `/v1`; используйте API-ключ от NVIDIA NGC.
-   Провайдер автоматически активируется при установке `NVIDIA_API_KEY`; использует статические значения по умолчанию (контекстное окно 131 072 токена, максимум 4 096 токенов).

[Mistral](./mistral.md)[Ollama](./ollama.md)