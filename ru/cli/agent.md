title: "Руководство по команде agent в OpenClaw CLI и примеры использования"
description: "Узнайте, как выполнить ход агента через OpenClaw CLI. Используйте примеры для отправки сообщений, выбора конкретных агентов и управления параметрами доставки."
keywords: ["openclaw cli", "команда agent", "cli агент", "запуск агента", "примеры агента", "отправка агенту", "агент шлюза", "cli инструменты"]
---

  Команды CLI

  
# agent

Выполните ход агента через Шлюз (используйте `--local` для встроенного режима). Используйте `--agent ` для прямого обращения к настроенному агенту. Связанные команды:

-   Инструмент отправки агенту: [Agent send](../tools/agent-send.md)

## Примеры

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## Примечания

-   Когда эта команда вызывает регенерацию `models.json`, учетные данные провайдеров, управляемые через SecretRef, сохраняются как несекретные маркеры (например, имена переменных окружения или `secretref-managed`), а не в виде расшифрованного текста секрета.

[acp](./acp.md)[agents](./agents.md)

---