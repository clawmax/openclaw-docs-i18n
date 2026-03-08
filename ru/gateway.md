

  Шлюз

  
# Runbook шлюза

Используйте эту страницу для запуска в первый день (day-1) и эксплуатации на второй день (day-2) службы Шлюза.

## 5-минутный локальный запуск

### Шаг 1: Запуск шлюза

```bash
openclaw gateway --port 18789
# debug/trace mirrored to stdio
openclaw gateway --port 18789 --verbose
# force-kill listener on selected port, then start
openclaw gateway --force
```

### Шаг 2: Проверка работоспособности службы

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

Здоровое базовое состояние: `Runtime: running` и `RPC probe: ok`.

### Шаг 3: Проверка готовности каналов

```bash
openclaw channels status --probe
```

 

> **ℹ️** Перезагрузка конфигурации шлюза отслеживает активный путь к файлу конфигурации (разрешенный из профиля/стандартных значений состояния, или `OPENCLAW_CONFIG_PATH`, если задан). Режим по умолчанию: `gateway.reload.mode="hybrid"`.

## Модель выполнения

-   Один постоянно работающий процесс для маршрутизации, плоскости управления и подключений каналов.
-   Один мультиплексированный порт для:
    -   WebSocket управления/RPC
    -   HTTP API (OpenAI-совместимые, Responses, вызов инструментов)
    -   UI управления и хуков
-   Режим привязки по умолчанию: `loopback`.
-   Аутентификация требуется по умолчанию (`gateway.auth.token` / `gateway.auth.password`, или `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Приоритет порта и привязки

| Настройка | Порядок разрешения |
| --- | --- |
| Порт шлюза | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Режим привязки | CLI/переопределение → `gateway.bind` → `loopback` |

### Режимы горячей перезагрузки

| `gateway.reload.mode` | Поведение |
| --- | --- |
| `off` | Без перезагрузки конфигурации |
| `hot` | Применять только безопасные для горячей замены изменения |
| `restart` | Перезапуск при изменениях, требующих перезагрузки |
| `hybrid` (по умолчанию) | Горячее применение, когда безопасно, перезапуск, когда требуется |

## Набор команд оператора

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw secrets reload
openclaw logs --follow
openclaw doctor
```

## Удаленный доступ

Предпочтительно: Tailscale/VPN. Запасной вариант: SSH-туннель.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Затем подключайте клиенты локально к `ws://127.0.0.1:18789`.

> **⚠️** Если настроена аутентификация шлюза, клиенты все равно должны отправлять данные аутентификации (`token`/`password`) даже через SSH-туннели.

 См.: [Удаленный шлюз](./gateway/remote.md), [Аутентификация](./gateway/authentication.md), [Tailscale](./gateway/tailscale.md).

## Надзор и жизненный цикл службы

Используйте управляемые запуски для надежности, приближенной к производственной.

].service\nopenclaw gateway status', lang: 'bash' }, { label: 'Linux (системная служба)', code: 'sudo systemctl daemon-reload\nsudo systemctl enable --now openclaw-gateway[-].service', lang: 'bash' }]} />

## Несколько шлюзов на одном хосте

В большинстве настроек должен работать **один** шлюз. Используйте несколько только для строгой изоляции/резервирования (например, для резервного профиля). Контрольный список для каждого экземпляра:

-   Уникальный `gateway.port`
-   Уникальный `OPENCLAW_CONFIG_PATH`
-   Уникальный `OPENCLAW_STATE_DIR`
-   Уникальный `agents.defaults.workspace`

Пример:

```
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

См.: [Несколько шлюзов](./gateway/multiple-gateways.md).

### Быстрый путь для dev-профиля

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Значения по умолчанию включают изолированное состояние/конфигурацию и базовый порт шлюза `19001`.

## Краткий справочник по протоколу (с точки зрения оператора)

-   Первый фрейм от клиента должен быть `connect`.
-   Шлюз возвращает снимок состояния `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, лимиты/политики).
-   Запросы: `req(method, params)` → `res(ok/payload|error)`.
-   Общие события: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Запуски агента выполняются в два этапа:

1.  Немедленное подтверждение принятия (`status:"accepted"`)
2.  Финальный ответ о завершении (`status:"ok"|"error"`), с потоковыми событиями `agent` между ними.

Полная документация по протоколу: [Протокол шлюза](./gateway/protocol.md).

## Операционные проверки

### Активность (Liveness)

-   Откройте WS и отправьте `connect`.
-   Ожидайте ответ `hello-ok` со снимком состояния.

### Готовность (Readiness)

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Восстановление после разрыва

События не воспроизводятся повторно. При разрывах последовательности обновите состояние (`health`, `system-presence`) перед продолжением.

## Типичные сигнатуры сбоев

| Сигнатура | Вероятная проблема |
| --- | --- |
| `refusing to bind gateway ... without auth` | Привязка не к loopback без токена/пароля |
| `another gateway instance is already listening` / `EADDRINUSE` | Конфликт портов |
| `Gateway start blocked: set gateway.mode=local` | В конфигурации установлен удаленный режим |
| `unauthorized` во время подключения | Несоответствие аутентификации между клиентом и шлюзом |

Для полных лестниц диагностики используйте [Устранение неполадок шлюза](./gateway/troubleshooting.md).

## Гарантии безопасности

-   Клиенты протокола шлюза быстро завершаются с ошибкой при недоступности шлюза (без неявного перехода на прямой канал).
-   Неверные/не-connect первые фреймы отклоняются и соединение закрывается.
-   Плавное завершение работы отправляет событие `shutdown` перед закрытием сокета.

* * *

Связанные темы:

-   [Устранение неполадок](./gateway/troubleshooting.md)
-   [Фоновый процесс](./gateway/background-process.md)
-   [Конфигурация](./gateway/configuration.md)
-   [Работоспособность](./gateway/health.md)
-   [Doctor](./gateway/doctor.md)
-   [Аутентификация](./gateway/authentication.md)

[Конфигурация](./gateway/configuration.md)