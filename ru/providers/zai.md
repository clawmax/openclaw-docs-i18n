title: "Руководство по настройке и конфигурации провайдера Z.AI в OpenClaw"
description: "Узнайте, как настроить и сконфигурировать провайдер Z.AI в OpenClaw для использования моделей GLM через API. Включает подключение через CLI и примеры конфигурации."
keywords: ["z.ai", "openclaw", "glм модели", "api провайдер", "api ключ", "стриминг инструментов", "настройка cli", "конфигурация"]
---

  Провайдеры

  
# Z.AI

Z.AI — это API-платформа для моделей **GLM**. Она предоставляет REST API для GLM и использует API-ключи для аутентификации. Создайте свой API-ключ в консоли Z.AI. OpenClaw использует провайдер `zai` с API-ключом Z.AI.

## Настройка через CLI

```bash
openclaw onboard --auth-choice zai-api-key
# или неинтерактивно
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## Фрагмент конфигурации

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Примечания

-   Модели GLM доступны как `zai/` (например: `zai/glm-5`).
-   `tool_stream` включён по умолчанию для стриминга вызовов инструментов Z.AI. Установите `agents.defaults.models["zai/"].params.tool_stream` в `false`, чтобы отключить его.
-   Смотрите [/providers/glm](./glm.md) для обзора семейства моделей.
-   Z.AI использует Bearer-аутентификацию с вашим API-ключом.

[Xiaomi MiMo](./xiaomi.md)

---