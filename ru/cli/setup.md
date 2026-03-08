title: "Руководство и примеры команды setup в OpenClaw CLI"
description: "Узнайте, как инициализировать конфигурацию OpenClaw CLI и рабочее пространство агента с помощью команды setup, включая примеры и параметры мастера настройки."
keywords: ["openclaw setup", "конфигурация cli", "рабочее пространство агента", "openclaw.json", "настройка командной строки", "мастер начальной настройки", "инициализация cli", "настройка рабочего пространства"]
---

  Команды CLI

  
# setup

Инициализирует файл `~/.openclaw/openclaw.json` и рабочее пространство агента. Связанные материалы:

-   Начало работы: [Начало работы](../start/getting-started.md)
-   Мастер настройки: [Первоначальная настройка](../start/onboarding.md)

## Примеры

```bash
openclaw setup
openclaw setup --workspace ~/.openclaw/workspace
```

Для запуска мастера настройки через setup:

```bash
openclaw setup --wizard
```

[sessions](./sessions.md)[skills](./skills.md)