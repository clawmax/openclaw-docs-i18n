title: "Установка OpenClaw Gateway в rootless-контейнере Podman"
description: "Узнайте, как установить и запустить шлюз OpenClaw AI в rootless-контейнере Podman. Включает скрипт настройки, службу systemd Quadlet и устранение неполадок."
keywords: ["podman", "openclaw", "rootless контейнер", "ai шлюз", "systemd quadlet", "настройка контейнера", "установка openclaw", "руководство по podman"]
---

  Другие способы установки

  
# Podman

Запустите шлюз OpenClaw в **rootless**-контейнере Podman. Используется тот же образ, что и для Docker (собирается из [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) репозитория).

## Требования

-   Podman (rootless)
-   Sudo для одноразовой настройки (создание пользователя, сборка образа)

## Быстрый старт

**1\. Одноразовая настройка** (из корня репозитория; создаёт пользователя, собирает образ, устанавливает скрипт запуска):

```
./setup-podman.sh
```

Это также создаёт минимальный файл `~openclaw/.openclaw/openclaw.json` (устанавливает `gateway.mode="local"`), чтобы шлюз мог запуститься без запуска мастера настройки. По умолчанию контейнер **не** устанавливается как служба systemd, вы запускаете его вручную (см. ниже). Для настройки в стиле production с автозапуском и перезапусками установите его как службу systemd Quadlet для пользователя:

```bash
./setup-podman.sh --quadlet
```

(Или установите `OPENCLAW_PODMAN_QUADLET=1`; используйте `--container` для установки только контейнера и скрипта запуска.) Дополнительные переменные окружения времени сборки (установите перед запуском `setup-podman.sh`):

-   `OPENCLAW_DOCKER_APT_PACKAGES` — установка дополнительных пакетов apt во время сборки образа
-   `OPENCLAW_EXTENSIONS` — предустановка зависимостей расширений (имена расширений через пробел, например `diagnostics-otel matrix`)

**2\. Запуск шлюза** (вручную, для быстрой проверки):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3\. Мастер настройки** (например, для добавления каналов или провайдеров):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Затем откройте `http://127.0.0.1:18789/` и используйте токен из `~openclaw/.openclaw/.env` (или значение, выведенное мастером настройки).

## Systemd (Quadlet, опционально)

Если вы запустили `./setup-podman.sh --quadlet` (или установили `OPENCLAW_PODMAN_QUADLET=1`), устанавливается [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) юнит, чтобы шлюз работал как служба systemd для пользователя openclaw. Служба включается и запускается в конце настройки.

-   **Запуск:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
-   **Остановка:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
-   **Статус:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
-   **Логи:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Файл quadlet находится по пути `~openclaw/.config/containers/systemd/openclaw.container`. Чтобы изменить порты или переменные окружения, отредактируйте этот файл (или файл `.env`, который он использует), затем выполните `sudo systemctl --machine openclaw@ --user daemon-reload` и перезапустите службу. При загрузке система служба запускается автоматически, если для пользователя openclaw включено lingering (настройка делает это, если доступен loginctl). Чтобы добавить quadlet **после** первоначальной настройки, которая его не использовала, перезапустите: `./setup-podman.sh --quadlet`.

## Пользователь openclaw (без входа в систему)

`setup-podman.sh` создаёт выделенного системного пользователя `openclaw`:

-   **Оболочка:** `nologin` — без интерактивного входа; уменьшает поверхность атаки.
-   **Домашний каталог:** например, `/home/openclaw` — содержит `~/.openclaw` (конфигурация, рабочая область) и скрипт запуска `run-openclaw-podman.sh`.
-   **Rootless Podman:** Пользователь должен иметь диапазоны **subuid** и **subgid**. Многие дистрибутивы назначают их автоматически при создании пользователя. Если настройка выводит предупреждение, добавьте строки в `/etc/subuid` и `/etc/subgid`:
    
    Копировать
    
    ```
    openclaw:100000:65536
    ```
    
    Затем запустите шлюз от имени этого пользователя (например, из cron или systemd):
    
    Копировать
    
    ```bash
    sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
    sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
    ```
    
-   **Конфигурация:** Только `openclaw` и root имеют доступ к `/home/openclaw/.openclaw`. Для редактирования конфигурации: используйте UI управления после запуска шлюза или `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Окружение и конфигурация

-   **Токен:** Хранится в `~openclaw/.openclaw/.env` как `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` и `run-openclaw-podman.sh` генерируют его, если он отсутствует (используют `openssl`, `python3` или `od`).
-   **Опционально:** В этом файле `.env` вы можете установить ключи провайдеров (например, `GROQ_API_KEY`, `OLLAMA_API_KEY`) и другие переменные окружения OpenClaw.
-   **Порты хоста:** По умолчанию скрипт пробрасывает порты `18789` (шлюз) и `18790` (мост). Чтобы изменить проброс портов на **хосте**, используйте переменные `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` и `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` при запуске.
-   **Привязка шлюза:** По умолчанию `run-openclaw-podman.sh` запускает шлюз с `--bind loopback` для безопасного локального доступа. Чтобы открыть доступ в локальной сети, установите `OPENCLAW_GATEWAY_BIND=lan` и настройте `gateway.controlUi.allowedOrigins` (или явно включите резервный вариант с заголовком хоста) в `openclaw.json`.
-   **Пути:** Конфигурация и рабочая область на хосте по умолчанию находятся в `~openclaw/.openclaw` и `~openclaw/.openclaw/workspace`. Чтобы переопределить пути на хосте, используемые скриптом запуска, установите переменные `OPENCLAW_CONFIG_DIR` и `OPENCLAW_WORKSPACE_DIR`.

## Модель хранения данных

-   **Постоянные данные на хосте:** `OPENCLAW_CONFIG_DIR` и `OPENCLAW_WORKSPACE_DIR` подключаются в контейнер через bind mount и сохраняют состояние на хосте.
-   **Временная песочница tmpfs:** если вы включите `agents.defaults.sandbox`, контейнеры песочницы инструментов монтируют `tmpfs` по путям `/tmp`, `/var/tmp` и `/run`. Эти пути находятся в памяти и исчезают вместе с контейнером песочницы; настройка контейнера Podman верхнего уровня не добавляет собственных монтирований tmpfs.
-   **Точки роста диска:** основные пути для наблюдения — это `media/`, `agents//sessions/sessions.json`, файлы транскриптов JSONL, `cron/runs/*.jsonl` и ротируемые файлы логов в `/tmp/openclaw/` (или в вашем настроенном `logging.file`).

`setup-podman.sh` теперь размещает tar-архив образа в приватном временном каталоге и выводит выбранный базовый каталог во время настройки. Для запусков не от root он принимает `TMPDIR` только тогда, когда эта база безопасна для использования; в противном случае происходит откат к `/var/tmp`, затем к `/tmp`. Сохранённый tar-архив остаётся доступным только владельцу и передаётся в `podman load` целевого пользователя, поэтому приватные временные каталоги вызывающей стороны не блокируют настройку.

## Полезные команды

-   **Логи:** С quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Со скриптом: `sudo -u openclaw podman logs -f openclaw`
-   **Остановка:** С quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Со скриптом: `sudo -u openclaw podman stop openclaw`
-   **Запуск снова:** С quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Со скриптом: перезапустите скрипт запуска или выполните `podman start openclaw`
-   **Удаление контейнера:** `sudo -u openclaw podman rm -f openclaw` — конфигурация и рабочая область на хосте сохраняются

## Устранение неполадок

-   **Доступ запрещён (EACCES) для конфигурации или auth-profiles:** Контейнер по умолчанию использует `--userns=keep-id` и запускается с теми же uid/gid, что и пользователь хоста, запускающий скрипт. Убедитесь, что ваши каталоги `OPENCLAW_CONFIG_DIR` и `OPENCLAW_WORKSPACE_DIR` на хосте принадлежат этому пользователю.
-   **Запуск шлюза заблокирован (отсутствует `gateway.mode=local`):** Убедитесь, что файл `~openclaw/.openclaw/openclaw.json` существует и содержит `gateway.mode="local"`. `setup-podman.sh` создаёт этот файл, если он отсутствует.
-   **Rootless Podman не работает для пользователя openclaw:** Проверьте, что `/etc/subuid` и `/etc/subgid` содержат строку для `openclaw` (например, `openclaw:100000:65536`). Добавьте её, если отсутствует, и перезапустите.
-   **Имя контейнера уже используется:** Скрипт запуска использует `podman run --replace`, поэтому существующий контейнер заменяется при повторном запуске. Для ручной очистки: `podman rm -f openclaw`.
-   **Скрипт не найден при запуске от имени openclaw:** Убедитесь, что был запущен `setup-podman.sh`, чтобы `run-openclaw-podman.sh` был скопирован в домашний каталог openclaw (например, `/home/openclaw/run-openclaw-podman.sh`).
-   **Служба Quadlet не найдена или не запускается:** Выполните `sudo systemctl --machine openclaw@ --user daemon-reload` после редактирования файла `.container`. Quadlet требует cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` должен показывать `2`.

## Опционально: запуск от имени вашего пользователя

Чтобы запустить шлюз от имени вашего обычного пользователя (без выделенного пользователя openclaw): соберите образ, создайте `~/.openclaw/.env` с `OPENCLAW_GATEWAY_TOKEN` и запустите контейнер с `--userns=keep-id` и монтированием в ваш `~/.openclaw`. Скрипт запуска разработан для работы с пользователем openclaw; для однопользовательской настройки вы можете вместо этого запустить команду `podman run` из скрипта вручную, указав конфигурацию и рабочую область в вашем домашнем каталоге. Для большинства пользователей рекомендуется: используйте `setup-podman.sh` и запускайте от имени пользователя openclaw, чтобы конфигурация и процесс были изолированы.

[Docker](./docker.md)[Nix](./nix.md)