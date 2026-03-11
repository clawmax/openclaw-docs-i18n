

  Приложение-компаньон для macOS

  
# Шлюз на macOS

Приложение OpenClaw.app больше не включает в себя Node/Bun или среду выполнения шлюза. Приложение для macOS ожидает **внешнюю** установку CLI `openclaw`, не запускает шлюз как дочерний процесс и управляет пользовательской службой launchd для поддержания работы шлюза (или подключается к существующему локальному шлюзу, если он уже запущен).

## Установите CLI (требуется для локального режима)

Вам нужен Node 22+ на Mac, затем установите `openclaw` глобально:

```bash
npm install -g openclaw@<version>
```

Кнопка **Install CLI** в приложении macOS запускает тот же процесс через npm/pnpm (bun не рекомендуется для среды выполнения шлюза).

## Launchd (шлюз как LaunchAgent)

Метка:

-   `ai.openclaw.gateway` (или `ai.openclaw.`; устаревшие `com.openclaw.*` могут оставаться)

Расположение файла Plist (для каждого пользователя):

-   `~/Library/LaunchAgents/ai.openclaw.gateway.plist` (или `~/Library/LaunchAgents/ai.openclaw..plist`)

Управление:

-   Приложение macOS отвечает за установку/обновление LaunchAgent в локальном режиме.
-   CLI также может его установить: `openclaw gateway install`.

Поведение:

-   «OpenClaw Active» включает/отключает LaunchAgent.
-   Закрытие приложения **не** останавливает шлюз (launchd поддерживает его работу).
-   Если шлюз уже работает на настроенном порту, приложение подключается к нему вместо запуска нового.

Логирование:

-   stdout/err от launchd: `/tmp/openclaw/openclaw-gateway.log`

## Совместимость версий

Приложение macOS проверяет версию шлюза на соответствие своей версии. Если они несовместимы, обновите глобальный CLI до версии приложения.

## Быстрая проверка

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

Затем:

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```

[Релиз для macOS](./release.md)[IPC в macOS](./xpc.md)