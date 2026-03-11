

  Команды CLI

  
# daemon

Устаревший алиас для команд управления службой Gateway. `openclaw daemon ...` соответствует тому же интерфейсу управления службой, что и команды службы `openclaw gateway ...`.

## Использование

```bash
openclaw daemon status
openclaw daemon install
openclaw daemon start
openclaw daemon stop
openclaw daemon restart
openclaw daemon uninstall
```

## Подкоманды

-   `status`: показать состояние установки службы и проверить работоспособность Gateway
-   `install`: установить службу (`launchd`/`systemd`/`schtasks`)
-   `uninstall`: удалить службу
-   `start`: запустить службу
-   `stop`: остановить службу
-   `restart`: перезапустить службу

## Общие параметры

-   `status`: `--url`, `--token`, `--password`, `--timeout`, `--no-probe`, `--deep`, `--json`
-   `install`: `--port`, `--runtime <node|bun>`, `--token`, `--force`, `--json`
-   жизненный цикл (`uninstall|start|stop|restart`): `--json`

Примечания:

-   `status` при возможности разрешает настроенные SecretRefs для аутентификации при проверке работоспособности.
-   Когда для аутентификации по токену требуется токен и `gateway.auth.token` управляется через SecretRef, команда `install` проверяет, что SecretRef может быть разрешен, но не сохраняет разрешенный токен в метаданных среды службы.
-   Если для аутентификации по токену требуется токен, а настроенный SecretRef токена не разрешен, установка завершается с ошибкой.
-   Если настроены и `gateway.auth.token`, и `gateway.auth.password`, а `gateway.auth.mode` не задан, установка блокируется до явного указания режима.

## Рекомендуется

Используйте [`openclaw gateway`](./gateway.md) для актуальной документации и примеров.

[cron](./cron.md)[dashboard](./dashboard.md)

---