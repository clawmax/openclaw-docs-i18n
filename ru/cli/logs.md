title: "Использование команды logs в OpenClaw CLI и примеры"
description: "Узнайте, как использовать команду openclaw logs в CLI для отслеживания файловых логов шлюза через RPC. Примеры для слежения, вывода в JSON и локального часового пояса."
keywords: ["openclaw logs", "cli логи", "логи шлюза", "tail логи", "rpc логирование", "логи командной строки", "мониторинг логов"]
---

  Команды CLI

  
# logs

Отслеживание файловых логов Шлюза через RPC (работает в удаленном режиме). Связанное:

-   Обзор логирования: [Логирование](../logging.md)

## Примеры

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

Используйте `--local-time` для отображения временных меток в вашем локальном часовом поясе.

[hooks](./hooks.md)[memory](./memory.md)

---