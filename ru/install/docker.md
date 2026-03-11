

  Другие способы установки

  
# Docker

Docker — это **опционально**. Используйте его только если вам нужен контейнеризованный шлюз или для проверки работоспособности Docker-потока.

## Подходит ли мне Docker?

-   **Да**: если вам нужна изолированная, одноразовая среда шлюза или запуск OpenClaw на хосте без локальных установок.
-   **Нет**: если вы запускаете на своём компьютере и вам нужен самый быстрый цикл разработки. Вместо этого используйте обычный процесс установки.
-   **Примечание о песочнице**: песочница для агентов тоже использует Docker, но она **не** требует, чтобы весь шлюз работал в Docker. См. [Песочница](../gateway/sandboxing.md).

Это руководство охватывает:

-   Контейнеризованный шлюз (весь OpenClaw в Docker)
-   Песочница агента для каждой сессии (шлюз на хосте + инструменты агента, изолированные в Docker)

Подробности о песочнице: [Песочница](../gateway/sandboxing.md)

## Требования

-   Docker Desktop (или Docker Engine) + Docker Compose v2
-   Не менее 2 ГБ ОЗУ для сборки образа (`pnpm install` может быть завершён из-за нехватки памяти на хостах с 1 ГБ с кодом выхода 137)
-   Достаточно места на диске для образов и логов
-   Если запуск происходит на VPS/публичном хосте, ознакомьтесь с [Усилением безопасности для сетевого доступа](../gateway/security.md#04-network-exposure-bind--port--firewall), особенно с политикой фаервола Docker `DOCKER-USER`.

## Контейнеризованный шлюз (Docker Compose)

### Быстрый старт (рекомендуется)

> **ℹ️** Значения по умолчанию для Docker здесь предполагают режимы привязки (`lan`/`loopback`), а не псевдонимы хостов. Используйте значения режима привязки в `gateway.bind` (например, `lan` или `loopback`), а не псевдонимы хостов, такие как `0.0.0.0` или `localhost`.

 Из корня репозитория:

```
./docker-setup.sh
```

Этот скрипт:

-   собирает образ шлюза локально (или загружает удалённый образ, если задана переменная `OPENCLAW_IMAGE`)
-   запускает мастер настройки
-   выводит подсказки по настройке провайдеров
-   запускает шлюз через Docker Compose
-   генерирует токен шлюза и записывает его в `.env`

Опциональные переменные окружения:

-   `OPENCLAW_IMAGE` — использовать удалённый образ вместо локальной сборки (например, `ghcr.io/openclaw/openclaw:latest`)
-   `OPENCLAW_DOCKER_APT_PACKAGES` — установить дополнительные apt-пакеты во время сборки
-   `OPENCLAW_EXTENSIONS` — предустановить зависимости расширений во время сборки (имена расширений через пробел, например, `diagnostics-otel matrix`)
-   `OPENCLAW_EXTRA_MOUNTS` — добавить дополнительные привязки хоста
-   `OPENCLAW_HOME_VOLUME` — сохранить `/home/node` в именованном томе
-   `OPENCLAW_SANDBOX` — включить начальную настройку песочницы Docker для шлюза. Включается только при явных истинных значениях: `1`, `true`, `yes`, `on`
-   `OPENCLAW_INSTALL_DOCKER_CLI` — передача аргумента сборки для локальных сборок образа (`1` устанавливает Docker CLI в образ). `docker-setup.sh` устанавливает это автоматически при `OPENCLAW_SANDBOX=1` для локальных сборок.
-   `OPENCLAW_DOCKER_SOCKET` — переопределить путь к сокету Docker (по умолчанию: путь из `DOCKER_HOST=unix://...`, иначе `/var/run/docker.sock`)
-   `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` — аварийный режим: разрешить доверенные цели `ws://` в частной сети для путей клиента CLI/настройки (по умолчанию только loopback)
-   `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` — отключить флаги усиления безопасности контейнерного браузера `--disable-3d-apis`, `--disable-software-rasterizer`, `--disable-gpu`, когда требуется совместимость с WebGL/3D.
-   `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` — оставить расширения включёнными, когда потоки браузера требуют их (по умолчанию расширения отключены в браузере песочницы).
-   `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=` — установить лимит процессов рендерера Chromium; установите `0`, чтобы пропустить флаг и использовать поведение Chromium по умолчанию.

После завершения:

-   Откройте `http://127.0.0.1:18789/` в браузере.
-   Вставьте токен в Панель управления (Настройки → токен).
-   Нужна ссылка снова? Выполните `docker compose run --rm openclaw-cli dashboard --no-open`.

### Включить песочницу агента для Docker-шлюза (опционально)

`docker-setup.sh` также может выполнить начальную настройку `agents.defaults.sandbox.*` для развёртываний Docker. Включите с помощью:

```bash
export OPENCLAW_SANDBOX=1
./docker-setup.sh
```

Пользовательский путь к сокету (например, для rootless Docker):

```bash
export OPENCLAW_SANDBOX=1
export OPENCLAW_DOCKER_SOCKET=/run/user/1000/docker.sock
./docker-setup.sh
```

Примечания:

-   Скрипт монтирует `docker.sock` только после успешной проверки предварительных условий песочницы.
-   Если настройку песочницы не удаётся завершить, скрипт сбрасывает `agents.defaults.sandbox.mode` в `off`, чтобы избежать устаревшей/сломанной конфигурации песочницы при повторных запусках.
-   Если `Dockerfile.sandbox` отсутствует, скрипт выводит предупреждение и продолжает работу; соберите `openclaw-sandbox:bookworm-slim` с помощью `scripts/sandbox-setup.sh`, если нужно.
-   Для нелокальных значений `OPENCLAW_IMAGE` образ уже должен содержать поддержку Docker CLI для выполнения в песочнице.

### Автоматизация/CI (неинтерактивно, без вывода TTY)

Для скриптов и CI отключите выделение псевдо-TTY в Compose с помощью `-T`:

```bash
docker compose run -T --rm openclaw-cli gateway probe
docker compose run -T --rm openclaw-cli devices list --json
```

Если ваша автоматизация не экспортирует переменные сессии Claude, оставьте их незаданными — теперь они по умолчанию разрешаются в пустые значения в `docker-compose.yml`, чтобы избежать повторных предупреждений «переменная не задана».

### Примечание о безопасности общей сети (CLI + шлюз)

`openclaw-cli` использует `network_mode: "service:openclaw-gateway"`, чтобы команды CLI могли надёжно достигать шлюза через `127.0.0.1` в Docker. Рассматривайте это как общую границу доверия: привязка к loopback не обеспечивает изоляцию между этими двумя контейнерами. Если требуется более строгое разделение, запускайте команды из отдельного контейнера/пути сети хоста вместо встроенной службы `openclaw-cli`. Чтобы снизить ущерб в случае компрометации процесса CLI, конфигурация compose удаляет `NET_RAW`/`NET_ADMIN` и включает `no-new-privileges` для `openclaw-cli`. Она записывает конфигурацию/рабочее пространство на хосте:

-   `~/.openclaw/`
-   `~/.openclaw/workspace`

Запускаете на VPS? См. [Hetzner (Docker VPS)](./hetzner.md).

### Использовать удалённый образ (пропустить локальную сборку)

Официальные предварительно собранные образы публикуются по адресу:

-   [Пакет GitHub Container Registry](https://github.com/openclaw/openclaw/pkgs/container/openclaw)

Используйте имя образа `ghcr.io/openclaw/openclaw` (не похожие образы на Docker Hub). Распространённые теги:

-   `main` — последняя сборка из ветки `main`
-   `` — сборки по тегам релизов (например, `2026.2.26`)
-   `latest` — последний стабильный релизный тег

### Метаданные базового образа

Основной образ Docker в настоящее время использует:

-   `node:22-bookworm`

Образ docker теперь публикует аннотации базового образа OCI (sha256 — пример):

-   `org.opencontainers.image.base.name=docker.io/library/node:22-bookworm`
-   `org.opencontainers.image.base.digest=sha256:6d735b4d33660225271fda0a412802746658c3a1b975507b2803ed299609760a`
-   `org.opencontainers.image.source=https://github.com/openclaw/openclaw`
-   `org.opencontainers.image.url=https://openclaw.ai`
-   `org.opencontainers.image.documentation=https://docs.openclaw.ai/install/docker`
-   `org.opencontainers.image.licenses=MIT`
-   `org.opencontainers.image.title=OpenClaw`
-   `org.opencontainers.image.description=Контейнерный образ среды выполнения шлюза и CLI OpenClaw`
-   `org.opencontainers.image.revision=<git-sha>`
-   `org.opencontainers.image.version=<tag-or-main>`
-   `org.opencontainers.image.created=`

Справка: [Аннотации образов OCI](https://github.com/opencontainers/image-spec/blob/main/annotations.md) Контекст релиза: история тегов этого репозитория уже использует Bookworm в `v2026.2.22` и более ранних тегах 2026 года (например, `v2026.2.21`, `v2026.2.9`). По умолчанию скрипт настройки собирает образ из исходного кода. Чтобы загрузить предварительно собранный образ, задайте `OPENCLAW_IMAGE` перед запуском скрипта:

```bash
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
./docker-setup.sh
```

Скрипт определяет, что `OPENCLAW_IMAGE` отличается от значения по умолчанию `openclaw:local`, и выполняет `docker pull` вместо `docker build`. Всё остальное (настройка, запуск шлюза, генерация токена) работает так же. `docker-setup.sh` всё равно запускается из корня репозитория, потому что использует локальные `docker-compose.yml` и вспомогательные файлы. `OPENCLAW_IMAGE` пропускает время локальной сборки образа; он не заменяет рабочий процесс compose/настройки.

### Вспомогательные скрипты для оболочки (опционально)

Для более удобного повседневного управления Docker установите `ClawDock`:

```bash
mkdir -p ~/.clawdock && curl -sL https://raw.githubusercontent.com/openclaw/openclaw/main/scripts/shell-helpers/clawdock-helpers.sh -o ~/.clawdock/clawdock-helpers.sh
```

**Добавьте в конфигурацию вашей оболочки (zsh):**

```bash
echo 'source ~/.clawdock/clawdock-helpers.sh' >> ~/.zshrc && source ~/.zshrc
```

Затем используйте `clawdock-start`, `clawdock-stop`, `clawdock-dashboard` и т.д. Выполните `clawdock-help` для просмотра всех команд. Подробности см. в [`ClawDock` Helper README](https://github.com/openclaw/openclaw/blob/main/scripts/shell-helpers/README.md).

### Ручной процесс (compose)

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

Примечание: выполняйте `docker compose ...` из корня репозитория. Если вы включили `OPENCLAW_EXTRA_MOUNTS` или `OPENCLAW_HOME_VOLUME`, скрипт настройки создаёт `docker-compose.extra.yml`; включайте его при запуске Compose в другом месте:

```bash
docker compose -f docker-compose.yml -f docker-compose.extra.yml <command>
```

### Токен Панели управления + сопряжение (Docker)

Если вы видите «unauthorized» или «disconnected (1008): pairing required», получите свежую ссылку на панель управления и подтвердите устройство в браузере:

```bash
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

Подробнее: [Панель управления](../web/dashboard.md), [Устройства](../cli/devices.md).

### Дополнительные монтирования (опционально)

Если вы хотите смонтировать дополнительные каталоги хоста в контейнеры, задайте `OPENCLAW_EXTRA_MOUNTS` перед запуском `docker-setup.sh`. Это принимает список привязок Docker, разделённый запятыми, и применяет их к обоим `openclaw-gateway` и `openclaw-cli`, генерируя `docker-compose.extra.yml`. Пример:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Примечания:

-   Пути должны быть доступны для Docker Desktop на macOS/Windows.
-   Каждая запись должна быть в формате `источник:цель[:опции]` без пробелов, табуляций или переводов строк.
-   Если вы измените `OPENCLAW_EXTRA_MOUNTS`, перезапустите `docker-setup.sh`, чтобы заново сгенерировать дополнительный файл compose.
-   `docker-compose.extra.yml` генерируется автоматически. Не редактируйте его вручную.

### Сохранить весь домашний каталог контейнера (опционально)

Если вы хотите, чтобы `/home/node` сохранялся между пересозданиями контейнера, задайте именованный том через `OPENCLAW_HOME_VOLUME`. Это создаёт том Docker и монтирует его в `/home/node`, сохраняя стандартные привязки конфигурации/рабочего пространства. Используйте здесь именованный том (не путь привязки); для привязок используйте `OPENCLAW_EXTRA_MOUNTS`. Пример:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

Можно комбинировать с дополнительными монтированиями:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Примечания:

-   Именованные тома должны соответствовать шаблону `^[A-Za-z0-9][A-Za-z0-9_.-]*$`.
-   Если вы измените `OPENCLAW_HOME_VOLUME`, перезапустите `docker-setup.sh`, чтобы заново сгенерировать дополнительный файл compose.
-   Именованный том сохраняется до удаления командой `docker volume rm `.

### Установить дополнительные apt-пакеты (опционально)

Если вам нужны системные пакеты внутри образа (например, инструменты сборки или медиабиблиотеки), задайте `OPENCLAW_DOCKER_APT_PACKAGES` перед запуском `docker-setup.sh`. Это устанавливает пакеты во время сборки образа, поэтому они сохраняются даже при удалении контейнера. Пример:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Примечания:

-   Принимает список имён apt-пакетов, разделённых пробелами.
-   Если вы измените `OPENCLAW_DOCKER_APT_PACKAGES`, перезапустите `docker-setup.sh`, чтобы пересобрать образ.

### Предустановить зависимости расширений (опционально)

Расширения со своим собственным `package.json` (например, `diagnostics-otel`, `matrix`, `msteams`) устанавливают свои зависимости npm при первой загрузке. Чтобы встроить эти зависимости в образ, задайте `OPENCLAW_EXTENSIONS` перед запуском `docker-setup.sh`:

```bash
export OPENCLAW_EXTENSIONS="diagnostics-otel matrix"
./docker-setup.sh
```

Или при прямой сборке:

```bash
docker build --build-arg OPENCLAW_EXTENSIONS="diagnostics-otel matrix" .
```

Примечания:

-   Принимает список имён каталогов расширений (в `extensions/`), разделённых пробелами.
-   Затрагиваются только расширения с `package.json`; лёгкие плагины без него игнорируются.
-   Если вы измените `OPENCLAW_EXTENSIONS`, перезапустите `docker-setup.sh`, чтобы пересобрать образ.

### Для опытных пользователей / полнофункциональный контейнер (опционально)

Образ Docker по умолчанию **ориентирован на безопасность** и работает от имени непривилегированного пользователя `node`. Это уменьшает поверхность атаки, но означает:

-   отсутствие установки системных пакетов во время выполнения
-   отсутствие Homebrew по умолчанию
-   отсутствие встроенных браузеров Chromium/Playwright

Если вам нужен более полнофункциональный контейнер, используйте эти опциональные настройки:

1.  **Сохранить `/home/node`**, чтобы загрузки браузера и кэши инструментов сохранялись:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

2.  **Встроить системные зависимости в образ** (повторяемо + постоянно):

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="git curl jq"
./docker-setup.sh
```

3.  **Установить браузеры Playwright без `npx`** (избегает конфликтов переопределения npm):

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

Если нужно, чтобы Playwright установил системные зависимости, пересоберите образ с `OPENCLAW_DOCKER_APT_PACKAGES` вместо использования `--with-deps` во время выполнения.

4.  **Сохранить загрузки браузера Playwright**:

-   Установите `PLAYWRIGHT_BROWSERS_PATH=/home/node/.cache/ms-playwright` в `docker-compose.yml`.
-   Убедитесь, что `/home/node` сохраняется через `OPENCLAW_HOME_VOLUME`, или смонтируйте `/home/node/.cache/ms-playwright` через `OPENCLAW_EXTRA_MOUNTS`.

### Права доступа + EACCES

Образ запускается от имени `node` (uid 1000). Если вы видите ошибки прав доступа на `/home/node/.openclaw`, убедитесь, что привязки хоста принадлежат uid 1000. Пример (хост Linux):

```bash
sudo chown -R 1000:1000 /path/to/openclaw-config /path/to/openclaw-workspace
```

Если вы решите запускать от имени root для удобства, вы принимаете компромисс в безопасности.

### Ускорение пересборки (рекомендуется)

Чтобы ускорить пересборку, упорядочите Dockerfile так, чтобы слои зависимостей кэшировались. Это позволяет избежать повторного запуска `pnpm install`, если только не изменятся lock-файлы:

```dockerfile
FROM node:22-bookworm

# Установить Bun (требуется для скриптов сборки)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Кэшировать зависимости, если не изменились метаданные пакета
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### Настройка каналов (опционально)

Используйте контейнер CLI для настройки каналов, затем перезапустите шлюз, если нужно. WhatsApp (QR):

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (токен бота):

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (токен бота):

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

Документация: [WhatsApp](../channels/whatsapp.md), [Telegram](../channels/telegram.md), [Discord](../channels/discord.md)

### OAuth OpenAI Codex (безголовый Docker)

Если вы выберете OAuth OpenAI Codex в мастере, он откроет URL в браузере и попытается перехватить обратный вызов на `http://127.0.0.1:1455/auth/callback`. В Docker или безголовых настройках этот обратный вызов может показывать ошибку браузера. Скопируйте полный URL перенаправления, на который вы попали, и вставьте его обратно в мастер, чтобы завершить аутентификацию.

### Проверки работоспособности

Конечные точки проверки контейнера (аутентификация не требуется):

```bash
curl -fsS http://127.0.0.1:18789/healthz
curl -fsS http://127.0.0.1:18789/readyz
```

Псевдонимы: `/health` и `/ready`. `/healthz` — это простая проверка жизнеспособности для «процесс шлюза запущен». `/readyz` остаётся готовым во время периода ожидания запуска, затем становится `503` только если требуемые управляемые каналы всё ещё отключены после периода ожидания или отключаются позже. Образ Docker включает встроенную `HEALTHCHECK`, которая пингует `/healthz` в фоне. Простыми словами: Docker продолжает проверять, отвечает ли OpenClaw. Если проверки продолжают завершаться неудачей, Docker помечает контейнер как `unhealthy`, и системы оркестрации (политика перезапуска Docker Compose, Swarm, Kubernetes и т.д.) могут автоматически перезапустить или заменить его. Аутентифицированный глубокий снимок состояния (шлюз + каналы):

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### Сквозной тест (Docker)

```
scripts/e2e/onboard-docker.sh
```

### Тест импорта QR (Docker)

```bash
pnpm test:docker:qr
```

### LAN vs loopback (Docker Compose)

`docker-setup.sh` по умолчанию устанавливает `OPENCLAW_GATEWAY_BIND=lan`, чтобы доступ хоста к `http://127.0.0.1:18789` работал с публикацией портов Docker.

-   `lan` (по умолчанию): браузер хоста + CLI хоста могут достичь опубликованного порта шлюза.
-   `loopback`: только процессы внутри пространства имён сети контейнера могут напрямую достичь шлюза; доступ к опубликованному порту хоста может не работать.

Скрипт настройки также фиксирует `gateway.mode=local` после настройки, чтобы команды Docker CLI по умолчанию использовали локальную привязку к loopback. Примечание о старой конфигурации: используйте значения режима привязки в `gateway.bind` (`lan` / `loopback` / `custom` / `tailnet` / `auto`), а не псевдонимы хостов (`0.0.0.0`, `127.0.0.1`, `localhost`, `::`, `::1`). Если вы видите `Gateway target: ws://172.x.x.x:18789` или повторяющиеся ошибки `pairing required` от команд Docker CLI, выполните:

```bash
docker compose run --rm openclaw-cli config set gateway.mode local
docker compose run --rm openclaw-cli config set gateway.bind lan
docker compose run --rm openclaw-cli devices list --url ws://127.0.0.1:18789
```

### Примечания

-   Привязка шлюза по умолчанию `lan` для использования в контейнере (`OPENCLAW_GATEWAY_BIND`).
-   Dockerfile CMD использует `--allow-unconfigured`; смонтированная конфигурация с `gateway.mode`, отличным от `local`, всё равно запустится. Переопределите CMD, чтобы принудительно включить защиту.
-   Контейнер шлюза является источником истины для сессий (`~/.openclaw/agents//sessions/`).

### Модель хранения

-   **Постоянные данные хоста:** Docker Compose привязывает `OPENCLAW_CONFIG_DIR` к `/home/node/.openclaw` и `OPENCLAW_WORKSPACE_DIR` к `/home/node/.openclaw/workspace`, поэтому эти пути сохраняются при замене контейнера.
-   **Временная tmpfs песочницы:** когда включён `agents.defaults.sandbox`, контейнеры песочницы используют `tmpfs` для `/tmp`, `/var/tmp` и `/run`. Эти монтирования отделены от основного стека Compose и исчезают вместе с контейнером песочницы.
-   **Точки роста диска:** следите за `media/`, `agents//sessions/sessions.json`, файлами JSONL транскриптов, `cron/runs/*.jsonl` и ротируемыми файлами логов в `/tmp/openclaw/` (или вашем настроенном `logging.file`). Если вы также запускаете приложение macOS вне Docker, его логи службы снова отдельны: `~/.openclaw/logs/gateway.log`, `~/.openclaw/logs/gateway.err.log` и `/tmp/openclaw/openclaw-gateway.log`.

## Песочница агента (шлюз на хосте + инструменты Docker)

Подробности: [Песочница](../gateway/sandboxing.md)

### Что это делает

Когда включён `agents.defaults.sandbox`, **не основные сессии** запускают инструменты внутри контейнера Docker. Шлюз остаётся на вашем хосте, но выполнение инструментов изолировано:

-   область: `"agent"` по умолчанию (один контейнер + рабочее пространство на агента)
-   область: `"session"` для изоляции каждой сессии
-   папка рабочего пространства для каждой области монтируется в `/workspace`
-   опциональный доступ к рабочему пространству агента (`agents.defaults.sandbox.workspaceAccess`)
-   политика разрешения/запрета инструментов (запрет имеет приоритет)
-   входящие медиа копируются в активное рабочее пространство песочницы (`media/inbound/*`), чтобы инструменты могли их читать (при `workspaceAccess: "rw"` они попадают в рабочее пространство агента)

Предупреждение: `scope: "shared"` отключает изоляцию между сессиями. Все сессии используют один контейнер и одно рабочее пространство.

### Профили песочницы для каждого агента (многопользовательский режим)

Если вы используете маршрутизацию нескольких агентов, каждый агент может переопределить настройки песочницы и инструментов: `agents.list[].sandbox` и `agents.list[].tools` (плюс `agents.list[].tools.sandbox.tools`). Это позволяет запускать смешанные уровни доступа в одном шлюзе:

-   Полный доступ (личный агент)
-   Инструменты только для чтения + рабочее пространство только для чтения (семейный/рабочий агент)
-   Без инструментов файловой системы/оболочки (публичный агент)

См. [Песочница и инструменты для нескольких агентов](../tools/multi-agent-sandbox-tools.md) для примеров, приоритетов и устранения неполадок.

### Поведение по умолчанию

-   Образ: `openclaw-sandbox:bookworm-slim`
-   Один контейнер на агента
-   Доступ к рабочему пространству агента: `workspaceAccess: "none"` (по умолчанию) использует `~/.openclaw/sandboxes`
    -   `"ro"` сохраняет рабочее пространство песочницы в `/workspace` и монтирует рабочее пространство агента только для чтения в `/agent` (отключает `write`/`edit`/`apply_patch`)
    -   `"rw"` монтирует рабочее пространство агента для чтения/записи в `/workspace`
-   Автоочистка: бездействие > 24ч ИЛИ возраст > 7д
-   Сеть: `none` по умолчанию (явно включайте, если нужен исходящий трафик)
    -   `host` заблокирован.
    -   `container:` заблокирован по умолчанию (риск присоединения к пространству имён).
-   Разрешено по умолчанию: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   Запрещено по умолчанию: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

### Включить песочницу

Если вы планируете устанавливать пакеты в `setupCommand`, обратите внимание:

-   По умолчанию `docker.network` — `"none"` (без исходящего трафика).
-   `docker.network: "host"` заблокирован.
-   `docker.network: "container:"` заблокирован по умолчанию.
-   Аварийное переопределение: `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.
-   `readOnlyRoot: true` блокирует установку пакетов.
-   `user` должен быть root для `apt-get` (опустите `user` или установите `user: "0:0"`). OpenClaw автоматически пересоздаёт контейнеры при изменении `setupCommand` (или конфигурации docker), если контейнер **не использовался недавно** (в течение ~5 минут). Активные контейнеры выводят предупреждение с точной командой `openclaw sandbox recreate ...`.

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "