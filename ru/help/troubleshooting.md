title: "Руководство по устранению неполадок OpenClaw и команды быстрого старта"
description: "Быстрая диагностика и исправление проблем OpenClaw. Следуйте пошаговым лестницам команд для проверки статуса, шлюза, каналов, логов, cron, узлов и устранения неполадок браузера."
keywords: ["устранение неполадок openclaw", "статус openclaw", "проверка шлюза", "статус каналов", "логи openclaw", "openclaw doctor", "руководство по устранению неполадок", "диагностика ошибок"]
---

  Помощь

  
# Устранение неполадок

Если у вас есть всего 2 минуты, используйте эту страницу как входную точку для сортировки проблем.

## Первые 60 секунд

Запустите эту точную последовательность команд по порядку:

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw gateway status
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

Хороший вывод в одной строке:

-   `openclaw status` → показывает настроенные каналы и отсутствие очевидных ошибок аутентификации.
-   `openclaw status --all` → полный отчет присутствует и доступен для отправки.
-   `openclaw gateway probe` → целевой шлюз доступен.
-   `openclaw gateway status` → `Runtime: running` и `RPC probe: ok`.
-   `openclaw doctor` → отсутствуют блокирующие ошибки конфигурации/сервиса.
-   `openclaw channels status --probe` → каналы отображают статус `connected` или `ready`.
-   `openclaw logs --follow` → стабильная активность, без повторяющихся фатальных ошибок.

## Anthropic long context 429

Если вы видите: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`, перейдите по ссылке [/gateway/troubleshooting#anthropic-429-extra-usage-required-for-long-context](../gateway/troubleshooting.md#anthropic-429-extra-usage-required-for-long-context).

## Установка плагина не удается из-за отсутствия расширений openclaw

Если установка завершается ошибкой `package.json missing openclaw.extensions`, пакет плагина использует устаревшую структуру, которую OpenClaw больше не принимает. Исправьте в пакете плагина:

1.  Добавьте `openclaw.extensions` в `package.json`.
2.  Укажите записи на скомпилированные файлы времени выполнения (обычно `./dist/index.js`).
3.  Переопубликуйте плагин и снова запустите `openclaw plugins install <npm-spec>`.

Пример:

```json
{
  "name": "@openclaw/my-plugin",
  "version": "1.2.3",
  "openclaw": {
    "extensions": ["./dist/index.js"]
  }
}
```

Ссылка: [/tools/plugin#distribution-npm](../tools/plugin.md#distribution-npm)

## Дерево решений

```bash
openclaw status
openclaw gateway status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
```

Хороший вывод выглядит так:

-   `Runtime: running`
-   `RPC probe: ok`
-   Ваш канал показывает статус connected/ready в `channels status --probe`
-   Отправитель отображается как одобренный (или политика ЛС открыта/в белом списке)

Распространенные сигнатуры в логах:

-   `drop guild message (mention required` → упоминание заблокировало сообщение в Discord.
-   `pairing request` → отправитель не одобрен и ожидает подтверждения в ЛС.
-   `blocked` / `allowlist` в логах канала → отправитель, комната или группа отфильтрованы.

Подробные страницы:

-   [/gateway/troubleshooting#no-replies](../gateway/troubleshooting.md#no-replies)
-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Хороший вывод выглядит так:

-   `Dashboard: http://...` отображается в `openclaw gateway status`
-   `RPC probe: ok`
-   В логах нет цикла аутентификации

Распространенные сигнатуры в логах:

-   `device identity required` → HTTP/небезопасный контекст не может завершить аутентификацию устройства.
-   `unauthorized` / цикл переподключения → неверный токен/пароль или несоответствие режима аутентификации.
-   `gateway connect failed:` → UI указывает на неверный URL/порт или недоступный шлюз.

Подробные страницы:

-   [/gateway/troubleshooting#dashboard-control-ui-connectivity](../gateway/troubleshooting.md#dashboard-control-ui-connectivity)
-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](../gateway/authentication.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Хороший вывод выглядит так:

-   `Service: ... (loaded)`
-   `Runtime: running`
-   `RPC probe: ok`

Распространенные сигнатуры в логах:

-   `Gateway start blocked: set gateway.mode=local` → режим шлюза не установлен/удаленный.
-   `refusing to bind gateway ... without auth` → привязка не к локальному интерфейсу без токена/пароля.
-   `another gateway instance is already listening` или `EADDRINUSE` → порт уже занят.

Подробные страницы:

-   [/gateway/troubleshooting#gateway-service-not-running](../gateway/troubleshooting.md#gateway-service-not-running)
-   [/gateway/background-process](../gateway/background-process.md)
-   [/gateway/configuration](../gateway/configuration.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Хороший вывод выглядит так:

-   Транспорт канала подключен.
-   Проверки сопряжения/белого списка пройдены.
-   Упоминания обнаруживаются там, где это требуется.

Распространенные сигнатуры в логах:

-   `mention required` → блокировка обработки из-за требования упоминания в группе.
-   `pairing` / `pending` → отправитель в ЛС еще не одобрен.
-   `not_in_channel`, `missing_scope`, `Forbidden`, `401/403` → проблема с токеном разрешений канала.

Подробные страницы:

-   [/gateway/troubleshooting#channel-connected-messages-not-flowing](../gateway/troubleshooting.md#channel-connected-messages-not-flowing)
-   [/channels/troubleshooting](../channels/troubleshooting.md)

```bash
openclaw status
openclaw gateway status
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

Хороший вывод выглядит так:

-   `cron.status` показывает включенный статус с ожиданием следующего запуска.
-   `cron runs` показывает недавние записи `ok`.
-   Heartbeat включен и не находится за пределами активных часов.

Распространенные сигнатуры в логах:

-   `cron: scheduler disabled; jobs will not run automatically` → cron отключен.
-   `heartbeat skipped` с `reason=quiet-hours` → вне настроенных активных часов.
-   `requests-in-flight` → основной канал занят; запуск heartbeat был отложен.
-   `unknown accountId` → целевой аккаунт для доставки heartbeat не существует.

Подробные страницы:

-   [/gateway/troubleshooting#cron-and-heartbeat-delivery](../gateway/troubleshooting.md#cron-and-heartbeat-delivery)
-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)

```bash
openclaw status
openclaw gateway status
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw logs --follow
```

Хороший вывод выглядит так:

-   Узел указан как подключенный и сопряженный для роли `node`.
-   Существует возможность для вызываемой команды.
-   Состояние разрешения предоставлено для инструмента.

Распространенные сигнатуры в логах:

-   `NODE_BACKGROUND_UNAVAILABLE` → переведите приложение узла на передний план.
-   `*_PERMISSION_REQUIRED` → разрешение ОС было отклонено/отсутствует.
-   `SYSTEM_RUN_DENIED: approval required` → ожидание подтверждения выполнения.
-   `SYSTEM_RUN_DENIED: allowlist miss` → команда отсутствует в белом списке выполнения.

Подробные страницы:

-   [/gateway/troubleshooting#node-paired-tool-fails](../gateway/troubleshooting.md#node-paired-tool-fails)
-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

```bash
openclaw status
openclaw gateway status
openclaw browser status
openclaw logs --follow
openclaw doctor
```

Хороший вывод выглядит так:

-   Статус браузера показывает `running: true` и выбранный браузер/профиль.
-   Профиль `openclaw` запускается или реле `chrome` имеет подключенную вкладку.

Распространенные сигнатуры в логах:

-   `Failed to start Chrome CDP on port` → не удалось запустить локальный браузер.
-   `browser.executablePath not found` → настроенный путь к бинарному файлу неверен.
-   `Chrome extension relay is running, but no tab is connected` → расширение не подключено.
-   `Browser attachOnly is enabled ... not reachable` → профиль только для подключения не имеет активной цели CDP.

Подробные страницы:

-   [/gateway/troubleshooting#browser-tool-fails](../gateway/troubleshooting.md#browser-tool-fails)
-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)

[Помощь](../help.md)[ЧаВо](./faq.md)