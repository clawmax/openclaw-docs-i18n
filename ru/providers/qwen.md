title: "Настройка провайдера OpenClaw Qwen: аутентификация и идентификаторы моделей"
description: "Узнайте, как включить, аутентифицировать и использовать провайдер Qwen в OpenClaw для бесплатного доступа к моделям Qwen Coder и Vision."
keywords: ["qwen", "openclaw", "провайдер моделей", "аутентификация", "oauth", "qwen coder", "qwen vision", "интеграция api"]
---

  Провайдеры

  
# Qwen

Qwen предоставляет бесплатный OAuth-флоу для моделей Qwen Coder и Qwen Vision (2000 запросов в день, с учётом ограничений скорости Qwen).

## Включение плагина

```bash
openclaw plugins enable qwen-portal-auth
```

После включения перезапустите Gateway.

## Аутентификация

```bash
openclaw models auth login --provider qwen-portal --set-default
```

Эта команда запускает OAuth-флоу с кодом устройства от Qwen и записывает запись о провайдере в ваш `models.json` (плюс создаёт псевдоним `qwen` для быстрого переключения).

## Идентификаторы моделей

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

Переключение моделей:

```bash
openclaw models set qwen-portal/coder-model
```

## Повторное использование входа через Qwen Code CLI

Если вы уже вошли через Qwen Code CLI, OpenClaw синхронизирует учётные данные из `~/.qwen/oauth_creds.json` при загрузке хранилища аутентификации. Вам всё равно понадобится запись `models.providers.qwen-portal` (используйте команду входа выше, чтобы создать её).

## Примечания

-   Токены обновляются автоматически; повторно запустите команду входа, если обновление не удалось или доступ был отозван.
-   Базовый URL по умолчанию: `https://portal.qwen.ai/v1` (можно переопределить с помощью `models.providers.qwen-portal.baseUrl`, если Qwen предоставляет другой эндпоинт).
-   См. [Провайдеры моделей](../concepts/model-providers.md) для общих правил работы с провайдерами.

[Qianfan](./qianfan.md)[Synthetic](./synthetic.md)

---