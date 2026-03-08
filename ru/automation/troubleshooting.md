

  Автоматизация

  
# Устранение неполадок автоматизации

Используйте эту страницу для решения проблем с планировщиком и доставкой (`cron` + `heartbeat`).

## Лестница команд

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Затем выполните проверки автоматизации:

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```

## Cron не запускается

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

Хороший вывод выглядит так:

-   `cron status` сообщает, что cron включен, и показывает будущее время `nextWakeAtMs`.
-   Задача включена и имеет корректное расписание/часовой пояс.
-   `cron runs` показывает статус `ok` или явную причину пропуска.

Распространённые признаки:

-   `cron: scheduler disabled; jobs will not run automatically` → cron отключен в конфигурации/переменных окружения.
-   `cron: timer tick failed` → сбой тика планировщика; проверьте контекст стека/логов вокруг.
-   `reason: not-due` в выводе запуска → ручной запуск вызван без `--force`, и задача ещё не должна выполняться.

## Cron запустился, но доставки нет

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw cron list
openclaw channels status --probe
openclaw logs --follow
```

Хороший вывод выглядит так:

-   Статус запуска `ok`.
-   Для изолированных задач установлены режим доставки и цель.
-   Проверка канала (`probe`) сообщает, что целевой канал подключен.

Распространённые признаки:

-   Запуск завершился успешно, но режим доставки `none` → внешнее сообщение не ожидается.
-   Отсутствует или неверна цель доставки (`channel`/`to`) → запуск может завершиться успешно внутренне, но пропустить исходящую отправку.
-   Ошибки авторизации канала (`unauthorized`, `missing_scope`, `Forbidden`) → доставка заблокирована из-за проблем с учётными данными/разрешениями канала.

## Heartbeat подавлен или пропущен

```bash
openclaw system heartbeat last
openclaw logs --follow
openclaw config get agents.defaults.heartbeat
openclaw channels status --probe
```

Хороший вывод выглядит так:

-   Heartbeat включен с ненулевым интервалом.
-   Результат последнего heartbeat — `ran` (или причина пропуска понятна).

Распространённые признаки:

-   `heartbeat skipped` с `reason=quiet-hours` → вне `activeHours`.
-   `requests-in-flight` → основной канал занят; heartbeat отложен.
-   `empty-heartbeat-file` → интервальный heartbeat пропущен, потому что `HEARTBEAT.md` не содержит полезного контента и нет запланированных событий cron с тегами.
-   `alerts-disabled` → настройки видимости подавляют исходящие сообщения heartbeat.

## Подводные камни часовых поясов и активных часов (activeHours)

```bash
openclaw config get agents.defaults.heartbeat.activeHours
openclaw config get agents.defaults.heartbeat.activeHours.timezone
openclaw config get agents.defaults.userTimezone || echo "agents.defaults.userTimezone not set"
openclaw cron list
openclaw logs --follow
```

Краткие правила:

-   `Config path not found: agents.defaults.userTimezone` означает, что ключ не установлен; heartbeat использует часовой пояс хоста (или `activeHours.timezone`, если он задан).
-   Cron без `--tz` использует часовой пояс хоста шлюза.
-   `activeHours` для heartbeat использует настроенное разрешение часового пояса (`user`, `local` или явный IANA tz).
-   Временные метки ISO без указания часового пояса для расписаний cron `at` обрабатываются как UTC.

Распространённые признаки:

-   Задачи выполняются в неправильное реальное время после изменения часового пояса хоста.
-   Heartbeat всегда пропускается в течение вашего дня, потому что `activeHours.timezone` указан неверно.

Связанные темы:

-   [/automation/cron-jobs](./cron-jobs.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)
-   [/automation/cron-vs-heartbeat](./cron-vs-heartbeat.md)
-   [/concepts/timezone](../concepts/timezone.md)

[Cron vs Heartbeat](./cron-vs-heartbeat.md)[Вебхуки](./webhook.md)