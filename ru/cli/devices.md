

  Команды CLI

  
# devices

Управление запросами на сопряжение устройств и токенами с ограниченной областью действия устройства.

## Команды

### openclaw devices list

Вывести список ожидающих запросов на сопряжение и сопряженных устройств.

```bash
openclaw devices list
openclaw devices list --json
```

### openclaw devices remove &lt;deviceId&gt;

Удалить одну запись о сопряженном устройстве.

```bash
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### openclaw devices clear --yes \[--pending\]

Массовое удаление сопряженных устройств.

```bash
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### openclaw devices approve \[requestId\] \[--latest\]

Одобрить ожидающий запрос на сопряжение устройства. Если `requestId` не указан, OpenClaw автоматически одобряет самый последний ожидающий запрос.

```bash
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### openclaw devices reject &lt;requestId&gt;

Отклонить ожидающий запрос на сопряжение устройства.

```bash
openclaw devices reject <requestId>
```

### openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; \[--scope &lt;scope...&gt;\]

Обновить токен устройства для конкретной роли (с возможным обновлением областей действия).

```bash
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### openclaw devices revoke --device &lt;id&gt; --role &lt;role&gt;

Отозвать токен устройства для конкретной роли.

```bash
openclaw devices revoke --device <deviceId> --role node
```

## Общие параметры

-   `--url `: URL WebSocket шлюза (по умолчанию используется `gateway.remote.url`, если настроен).
-   `--token `: Токен шлюза (если требуется).
-   `--password `: Пароль шлюза (аутентификация по паролю).
-   `--timeout `: Таймаут RPC.
-   `--json`: Вывод в формате JSON (рекомендуется для скриптов).

Примечание: при установке `--url` CLI не возвращается к учетным данным из конфигурации или переменных окружения. Передавайте `--token` или `--password` явно. Отсутствие явных учетных данных является ошибкой.

## Примечания

-   Ротация токена возвращает новый токен (конфиденциальные данные). Обращайтесь с ним как с секретом.
-   Эти команды требуют область действия `operator.pairing` (или `operator.admin`).
-   Команда `devices clear` намеренно защищена флагом `--yes`.
-   Если область действия для сопряжения недоступна на локальном loopback-интерфейсе (и явный `--url` не передан), list/approve могут использовать локальный резервный механизм сопряжения.

[Панель управления](./dashboard.md)[Справочник](./directory.md)