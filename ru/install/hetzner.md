title: "Развертывание OpenClaw Gateway на Hetzner VPS с использованием Docker"
description: "Пошаговое руководство по запуску постоянного OpenClaw Gateway на Hetzner VPS с использованием Docker. Узнайте, как настроить состояние, включить бинарные файлы и получить доступ через SSH-туннель."
keywords: ["hetzner vps", "шлюз openclaw", "развертывание docker", "постоянное состояние", "ssh туннель", "docker compose", "сохранение бинарных файлов", "самостоятельный хостинг openclaw"]
---

  Хостинг и развертывание

  
# Hetzner

## Цель

Запустить постоянный OpenClaw Gateway на Hetzner VPS с использованием Docker, с устойчивым состоянием, встроенными бинарными файлами и безопасным поведением при перезапуске. Если вам нужно «OpenClaw 24/7 за ~$5», это самая простая надежная настройка. Цены Hetzner меняются; выберите самый маленький VPS на Debian/Ubuntu и увеличьте масштаб, если столкнетесь с нехваткой памяти. Напоминание о модели безопасности:

-   Агенты, общие для компании, подходят, когда все находятся в одном доверительном периметре, а среда выполнения предназначена только для бизнеса.
-   Соблюдайте строгое разделение: выделенный VPS/среда выполнения + выделенные учетные записи; никаких личных профилей Apple/Google/браузера/менеджера паролей на этом хосте.
-   Если пользователи являются противниками друг для друга, разделите их по шлюзу/хосту/пользователю ОС.

См. [Безопасность](../gateway/security.md) и [Хостинг VPS](../vps.md).

## Что мы делаем (простыми словами)?

-   Арендуем небольшой Linux-сервер (Hetzner VPS)
-   Устанавливаем Docker (изолированная среда выполнения приложений)
-   Запускаем OpenClaw Gateway в Docker
-   Сохраняем `~/.openclaw` + `~/.openclaw/workspace` на хосте (переживает перезапуски/пересборки)
-   Получаем доступ к UI управления с вашего ноутбука через SSH-туннель

Доступ к шлюзу можно получить через:

-   Проброс портов SSH с вашего ноутбука
-   Прямое открытие порта, если вы самостоятельно управляете фаерволом и токенами

В этом руководстве предполагается использование Ubuntu или Debian на Hetzner.  
Если вы используете другой Linux VPS, сопоставьте пакеты соответствующим образом. Общий процесс работы с Docker см. в [Docker](./docker.md).

* * *

## Быстрый путь (для опытных операторов)

1.  Подготовьте Hetzner VPS
2.  Установите Docker
3.  Клонируйте репозиторий OpenClaw
4.  Создайте постоянные каталоги на хосте
5.  Настройте `.env` и `docker-compose.yml`
6.  Включите необходимые бинарные файлы в образ
7.  `docker compose up -d`
8.  Проверьте сохранение состояния и доступ к шлюзу

* * *

## Что вам понадобится

-   Hetzner VPS с доступом root
-   Доступ по SSH с вашего ноутбука
-   Базовое умение работать с SSH + копировать/вставлять команды
-   ~20 минут
-   Docker и Docker Compose
-   Учетные данные для аутентификации модели
-   Опционально учетные данные провайдеров
    -   WhatsApp QR
    -   Токен бота Telegram
    -   Gmail OAuth

* * *

## 1) Подготовка VPS

Создайте VPS на Ubuntu или Debian в Hetzner. Подключитесь как root:

```bash
ssh root@YOUR_VPS_IP
```

В этом руководстве предполагается, что VPS является постоянным. Не рассматривайте его как одноразовую инфраструктуру.

* * *

## 2) Установка Docker (на VPS)

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Проверьте:

```bash
docker --version
docker compose version
```

* * *

## 3) Клонирование репозитория OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

В этом руководстве предполагается, что вы будете собирать пользовательский образ, чтобы гарантировать сохранность бинарных файлов.

* * *

## 4) Создание постоянных каталогов на хосте

Контейнеры Docker являются эфемерными. Все долгоживущие состояния должны храниться на хосте.

```bash
mkdir -p /root/.openclaw/workspace

# Установите владельца для пользователя контейнера (uid 1000):
chown -R 1000:1000 /root/.openclaw
```

* * *

## 5) Настройка переменных окружения

Создайте файл `.env` в корне репозитория.

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Сгенерируйте надежные секреты:

```bash
openssl rand -hex 32
```

**Не коммитьте этот файл.**

* * *

## 6) Конфигурация Docker Compose

Создайте или обновите файл `docker-compose.yml`.

```
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Рекомендуется: оставить шлюз доступным только на loopback-интерфейсе VPS; доступ через SSH-туннель.
      # Чтобы открыть его публично, удалите префикс `127.0.0.1:` и настройте фаервол соответствующим образом.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
        "--allow-unconfigured",
      ]
```

`--allow-unconfigured` предназначен только для удобства начальной настройки и не заменяет правильную конфигурацию шлюза. Все равно установите аутентификацию (`gateway.auth.token` или пароль) и используйте безопасные настройки привязки для вашего развертывания.

* * *

## 7) Включение необходимых бинарных файлов в образ (критически важно)

Установка бинарных файлов внутри запущенного контейнера — это ловушка. Все, что установлено во время выполнения, будет потеряно при перезапуске. Все внешние бинарные файлы, требуемые навыками, должны быть установлены во время сборки образа. В примерах ниже показаны только три распространенных бинарных файла:

-   `gog` для доступа к Gmail
-   `goplaces` для Google Places
-   `wacli` для WhatsApp

Это примеры, а не полный список. Вы можете установить столько бинарных файлов, сколько нужно, используя тот же шаблон. Если позже вы добавите новые навыки, зависящие от дополнительных бинарных файлов, вы должны:

1.  Обновить Dockerfile
2.  Пересобрать образ
3.  Перезапустить контейнеры

**Пример Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Пример бинарного файла 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Пример бинарного файла 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Пример бинарного файла 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Добавьте больше бинарных файлов ниже, используя тот же шаблон

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

* * *

## 8) Сборка и запуск

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Проверьте бинарные файлы:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Ожидаемый вывод:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

* * *

## 9) Проверка шлюза

```bash
docker compose logs -f openclaw-gateway
```

Успех:

```ini
[gateway] listening on ws://0.0.0.0:18789
```

С вашего ноутбука:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Откройте: `http://127.0.0.1:18789/` Вставьте токен вашего шлюза.

* * *

## Что где сохраняется (источник истины)

OpenClaw работает в Docker, но Docker не является источником истины. Все долгоживущие состояния должны переживать перезапуски, пересборки и перезагрузки.

| Компонент | Расположение | Механизм сохранения | Примечания |
| --- | --- | --- | --- |
| Конфигурация шлюза | `/home/node/.openclaw/` | Том хоста (volume mount) | Включает `openclaw.json`, токены |
| Профили аутентификации моделей | `/home/node/.openclaw/` | Том хоста (volume mount) | Токены OAuth, API-ключи |
| Конфигурации навыков | `/home/node/.openclaw/skills/` | Том хоста (volume mount) | Состояние на уровне навыка |
| Рабочее пространство агента | `/home/node/.openclaw/workspace/` | Том хоста (volume mount) | Код и артефакты агента |
| Сессия WhatsApp | `/home/node/.openclaw/` | Том хоста (volume mount) | Сохраняет QR-логин |
| Ключевое хранилище Gmail | `/home/node/.openclaw/` | Том хоста + пароль | Требует `GOG_KEYRING_PASSWORD` |
| Внешние бинарные файлы | `/usr/local/bin/` | Образ Docker | Должны быть включены при сборке |
| Среда выполнения Node | Файловая система контейнера | Образ Docker | Пересобирается при каждой сборке образа |
| Пакеты ОС | Файловая система контейнера | Образ Docker | Не устанавливайте во время выполнения |
| Контейнер Docker | Эфемерный | Перезапускаемый | Безопасно уничтожать |

* * *

## Инфраструктура как код (Terraform)

Для команд, предпочитающих рабочие процессы инфраструктуры как код, общедоступная настройка Terraform предоставляет:

-   Модульная конфигурация Terraform с управлением удаленным состоянием
-   Автоматизированная подготовка через cloud-init
-   Скрипты развертывания (начальная загрузка, деплой, резервное копирование/восстановление)
-   Усиление безопасности (фаервол, UFW, доступ только по SSH)
-   Конфигурация SSH-туннеля для доступа к шлюзу

**Репозитории:**

-   Инфраструктура: [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)
-   Конфигурация Docker: [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)

Этот подход дополняет настройку Docker выше воспроизводимыми развертываниями, контролируемой версиями инфраструктурой и автоматизированным восстановлением после сбоев.

> **Примечание:** Поддерживается сообществом. По вопросам или предложениям см. ссылки на репозитории выше.

[Fly.io](./fly.md)[GCP](./gcp.md)