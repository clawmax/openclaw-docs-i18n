

  Команды CLI

  
# tui

Открывает терминальный интерфейс, подключенный к Шлюзу. Связанные материалы:

-   Руководство по TUI: [TUI](../web/tui.md)

Примечания:

-   `tui` разрешает настроенные SecretRefs для аутентификации шлюза (токен/пароль), когда это возможно (провайдеры `env`/`file`/`exec`).

## Примеры

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```

[system](./system.md)[uninstall](./uninstall.md)

---