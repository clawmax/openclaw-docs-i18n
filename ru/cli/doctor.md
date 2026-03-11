

  Команды CLI

  
# doctor

Проверка состояния и быстрое исправление для шлюза и каналов. Связанные темы:

-   Устранение неполадок: [Устранение неполадок](../gateway/troubleshooting.md)
-   Проверка безопасности: [Безопасность](../gateway/security.md)

## Примеры

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Примечания:

-   Интерактивные запросы (например, исправления связки ключей/OAuth) запускаются только тогда, когда stdin является TTY и параметр `--non-interactive` **не** установлен. Автоматические запуски (cron, Telegram, без терминала) пропускают запросы.
-   `--fix` (псевдоним для `--repair`) создает резервную копию в `~/.openclaw/openclaw.json.bak` и удаляет неизвестные ключи конфигурации, выводя список каждого удаления.
-   Проверки целостности состояния теперь обнаруживают "осиротевшие" файлы транскриптов в директории сессий и могут архивировать их как `.deleted.` для безопасного освобождения места.
-   Doctor включает проверку готовности поиска в памяти и может рекомендовать `openclaw configure --section model`, когда отсутствуют учетные данные для эмбеддингов.
-   Если режим песочницы включен, но Docker недоступен, doctor сообщает о важном предупреждении с рекомендацией по исправлению (`install Docker` или `openclaw config set agents.defaults.sandbox.mode off`).

## macOS: переопределения env в launchctl

Если вы ранее запускали `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (или `...PASSWORD`), это значение переопределяет ваш конфигурационный файл и может вызывать постоянные ошибки "unauthorized".

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```

[документация](./docs.md)[шлюз](./gateway.md)