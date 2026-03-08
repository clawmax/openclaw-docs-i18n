title: "Настройка провайдера Qianfan в OpenClaw и конфигурация API-ключа"
description: "Узнайте, как настроить и использовать провайдер Qianfan в OpenClaw. Получите свой Baidu Qianfan API-ключ и настройте CLI для доступа к AI-моделям."
keywords: ["qianfan", "baidu", "openclaw", "api ключ", "maas", "openai совместимый", "провайдер моделей", "настройка cli"]
---

  Провайдеры

  
# Qianfan

Qianfan — это MaaS-платформа от Baidu, предоставляющая **единый API**, который направляет запросы ко многим моделям через одну конечную точку и один API-ключ. Она совместима с OpenAI, поэтому большинство SDK OpenAI будут работать после смены базового URL.

## Предварительные требования

1.  Учетная запись Baidu Cloud с доступом к API Qianfan
2.  API-ключ из консоли Qianfan
3.  Установленный на вашей системе OpenClaw

## Получение вашего API-ключа

1.  Перейдите в [Консоль Qianfan](https://console.bce.baidu.com/qianfan/ais/console/apiKey)
2.  Создайте новое приложение или выберите существующее
3.  Сгенерируйте API-ключ (формат: `bce-v3/ALTAK-...`)
4.  Скопируйте API-ключ для использования с OpenClaw

## Настройка CLI

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## Связанная документация

-   [Конфигурация OpenClaw](../gateway/configuration.md)
-   [Провайдеры моделей](../concepts/model-providers.md)
-   [Настройка агента](../concepts/agent.md)
-   [Документация по API Qianfan](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)

[OpenRouter](./openrouter.md)[Qwen](./qwen.md)

---