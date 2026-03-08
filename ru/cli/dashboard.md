title: "Использование команды dashboard в OpenClaw CLI и доступные опции"
description: "Узнайте, как открыть панель управления OpenClaw Control UI через CLI, управлять безопасностью токенов и использовать флаг --no-open. Включает заметки по работе с SecretRef."
keywords: ["openclaw dashboard", "cli команда dashboard", "control ui", "gateway auth token", "secretref", "безопасность cli", "openclaw cli"]
---

  Команды CLI

  
# dashboard

Откройте Control UI, используя текущую аутентификацию.

```bash
openclaw dashboard
openclaw dashboard --no-open
```

Примечания:

-   Команда `dashboard` разрешает настроенные ссылки `gateway.auth.token` SecretRef, когда это возможно.
-   Для токенов, управляемых через SecretRef (разрешённых или нет), `dashboard` выводит/копирует/открывает URL без токенизации, чтобы избежать раскрытия внешних секретов в выводе терминала, истории буфера обмена или аргументах запуска браузера.
-   Если `gateway.auth.token` управляется через SecretRef, но не разрешён в данном пути выполнения команды, команда выводит URL без токенизации и явные инструкции по исправлению, вместо того чтобы вставлять недопустимый заполнитель токена.

[daemon](./daemon.md)[devices](./devices.md)