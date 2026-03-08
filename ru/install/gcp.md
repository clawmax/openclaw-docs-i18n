title: "Развертывание OpenClaw Gateway на GCP Compute Engine с использованием Docker"
description: "Пошаговое руководство по запуску постоянного OpenClaw Gateway на виртуальной машине GCP. Узнайте, как установить Docker, настроить сохранение состояния, встроить бинарные файлы и получить доступ к UI управления."
keywords: ["openclaw gcp", "gcp compute engine", "развертывание docker", "постоянный шлюз", "установка openclaw", "настройка gcp vm", "docker compose", "шлюз openclaw"]
---

  Хостинг и развертывание

  
# GCP

## Цель

Запустить постоянный OpenClaw Gateway на виртуальной машине GCP Compute Engine с использованием Docker, с устойчивым состоянием, встроенными бинарными файлами и безопасным поведением при перезапуске. Если вам нужно «OpenClaw 24/7 за ~$5-12/мес», это надежная настройка в Google Cloud. Цены зависят от типа машины и региона; выберите наименьшую ВМ, подходящую для вашей нагрузки, и масштабируйтесь при нехватке памяти.

## Что мы делаем (простыми словами)?

-   Создаем проект GCP и включаем биллинг
-   Создаем виртуальную машину Compute Engine
-   Устанавливаем Docker (изолированная среда выполнения приложений)
-   Запускаем OpenClaw Gateway в Docker
-   Сохраняем `~/.openclaw` + `~/.openclaw/workspace` на хосте (переживает перезапуски/пересборки)
-   Получаем доступ к UI управления с вашего ноутбука через SSH-туннель

Доступ к шлюзу возможен через:

-   SSH port forwarding с вашего ноутбука
-   Прямое открытие портов, если вы самостоятельно управляете фаерволом и токенами

В этом руководстве используется Debian на GCP Compute Engine. Ubuntu также подходит; сопоставьте пакеты соответствующим образом. Общий процесс работы с Docker см. в [Docker](./docker.md).

* * *

## Быстрый путь (для опытных операторов)

1.  Создайте проект GCP + включите Compute Engine API
2.  Создайте виртуальную машину Compute Engine (e2-small, Debian 12, 20GB)
3.  Подключитесь к ВМ по SSH
4.  Установите Docker
5.  Клонируйте репозиторий OpenClaw
6.  Создайте постоянные директории на хосте
7.  Настройте `.env` и `docker-compose.yml`
8.  Встройте необходимые бинарные файлы, соберите и запустите

* * *

## Что вам понадобится

-   Аккаунт GCP (подходит для бесплатного тарифа e2-micro)
-   Установленный gcloud CLI (или используйте Cloud Console)
-   SSH-доступ с вашего ноутбука
-   Базовое умение работать с SSH + копировать/вставлять
-   ~20-30 минут
-   Docker и Docker Compose
-   Учетные данные для моделей
-   Опциональные учетные данные провайдеров
    -   WhatsApp QR
    -   Токен бота Telegram
    -   Gmail OAuth

* * *

## 1) Установите gcloud CLI (или используйте Console)

**Вариант А: gcloud CLI** (рекомендуется для автоматизации) Установите с [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install) Инициализируйте и аутентифицируйтесь:

```bash
gcloud init
gcloud auth login
```

**Вариант Б: Cloud Console** Все шаги можно выполнить через веб-интерфейс по адресу [https://console.cloud.google.com](https://console.cloud.google.com)

* * *

## 2) Создайте проект GCP

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

Включите биллинг на [https://console.cloud.google.com/billing](https://console.cloud.google.com/billing) (требуется для Compute Engine). Включите Compute Engine API:

```bash
gcloud services enable compute.googleapis.com
```

**Console:**

1.  Перейдите в IAM & Admin > Create Project
2.  Назовите проект и создайте его
3.  Включите биллинг для проекта
4.  Перейдите в APIs & Services > Enable APIs > найдите “Compute Engine API” > Enable

* * *

## 3) Создайте виртуальную машину

**Типы машин:**

| Тип | Характеристики | Стоимость | Примечания |
| --- | --- | --- | --- |
| e2-medium | 2 vCPU, 4GB RAM | ~$25/мес | Наиболее надежно для локальных сборок Docker |
| e2-small | 2 vCPU, 2GB RAM | ~$12/мес | Минимальная рекомендация для сборки Docker |
| e2-micro | 2 vCPU (shared), 1GB RAM | Подходит для бесплатного тарифа | Часто падает с ошибкой OOM при сборке Docker (exit 137) |

**CLI:**

```
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Console:**

1.  Перейдите в Compute Engine > VM instances > Create instance
2.  Имя: `openclaw-gateway`
3.  Регион: `us-central1`, Зона: `us-central1-a`
4.  Тип машины: `e2-small`
5.  Загрузочный диск: Debian 12, 20GB
6.  Создать

* * *

## 4) Подключитесь к ВМ по SSH

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Console:** Нажмите кнопку “SSH” рядом с вашей ВМ в панели управления Compute Engine. Примечание: Распространение SSH-ключа может занять 1-2 минуты после создания ВМ. Если соединение отклонено, подождите и повторите попытку.

* * *

## 5) Установите Docker (на ВМ)

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

Выйдите и войдите снова, чтобы изменение группы вступило в силу:

```
exit
```

Затем снова подключитесь по SSH:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

Проверьте:

```bash
docker --version
docker compose version
```

* * *

## 6) Клонируйте репозиторий OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

В этом руководстве предполагается, что вы соберете пользовательский образ, чтобы гарантировать сохранность бинарных файлов.

* * *

## 7) Создайте постоянные директории на хосте

Контейнеры Docker эфемерны. Все долгоживущие данные должны храниться на хосте.

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

* * *

## 8) Настройте переменные окружения

Создайте `.env` в корне репозитория.

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Сгенерируйте надежные секреты:

```bash
openssl rand -hex 32
```

**Не коммитьте этот файл.**

* * *

## 9) Конфигурация Docker Compose

Создайте или обновите `docker-compose.yml`.

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
      # Рекомендуется: оставить шлюз доступным только на loopback-интерфейсе ВМ; доступ через SSH-туннель.
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
      ]
```

* * *

## 10) Встройте необходимые бинарные файлы в образ (критически важно)

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

## 11) Сборка и запуск

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Если сборка завершается с ошибкой `Killed` / `exit code 137` во время `pnpm install --frozen-lockfile`, на ВМ не хватает памяти. Используйте минимум `e2-small` или `e2-medium` для более надежной первой сборки. При привязке к локальной сети (`OPENCLAW_GATEWAY_BIND=lan`) настройте доверенный источник браузера перед продолжением:

```bash
docker compose run --rm openclaw-cli config set gateway.controlUi.allowedOrigins '["http://127.0.0.1:18789"]' --strict-json
```

Если вы изменили порт шлюза, замените `18789` на ваш настроенный порт. Проверьте бинарные файлы:

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

## 12) Проверка шлюза

```bash
docker compose logs -f openclaw-gateway
```

Успех:

```ini
[gateway] listening on ws://0.0.0.0:18789
```

* * *

## 13) Доступ с вашего ноутбука

Создайте SSH-туннель для проброса порта шлюза:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

Откройте в браузере: `http://127.0.0.1:18789/` Получите свежую ссылку на дашборд с токеном:

```bash
docker compose run --rm openclaw-cli dashboard --no-open
```

Вставьте токен из этого URL. Если UI управления показывает `unauthorized` или `disconnected (1008): pairing required`, подтвердите устройство браузера:

```bash
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

* * *

## Что где сохраняется (источник истины)

OpenClaw работает в Docker, но Docker не является источником истины. Все долгоживущие данные должны переживать перезапуски, пересборки и перезагрузки.

| Компонент | Расположение | Механизм сохранения | Примечания |
| --- | --- | --- | --- |
| Конфигурация шлюза | `/home/node/.openclaw/` | Том хоста (volume mount) | Включает `openclaw.json`, токены |
| Профили аутентификации моделей | `/home/node/.openclaw/` | Том хоста (volume mount) | OAuth-токены, API-ключи |
| Конфигурации навыков | `/home/node/.openclaw/skills/` | Том хоста (volume mount) | Состояние на уровне навыка |
| Рабочее пространство агента | `/home/node/.openclaw/workspace/` | Том хоста (volume mount) | Код и артефакты агента |
| Сессия WhatsApp | `/home/node/.openclaw/` | Том хоста (volume mount) | Сохраняет QR-логин |
| Ключевое хранилище Gmail | `/home/node/.openclaw/` | Том хоста + пароль | Требует `GOG_KEYRING_PASSWORD` |
| Внешние бинарные файлы | `/usr/local/bin/` | Образ Docker | Должны быть встроены при сборке |
| Среда выполнения Node | Файловая система контейнера | Образ Docker | Пересобирается при каждой сборке образа |
| Пакеты ОС | Файловая система контейнера | Образ Docker | Не устанавливайте во время выполнения |
| Контейнер Docker | Эфемерный | Перезапускаемый | Безопасно удалять |

* * *

## Обновления

Чтобы обновить OpenClaw на ВМ:

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

* * *

## Устранение неполадок

**SSH-соединение отклонено** Распространение SSH-ключа может занять 1-2 минуты после создания ВМ. Подождите и повторите попытку. **Проблемы с OS Login** Проверьте ваш профиль OS Login:

```bash
gcloud compute os-login describe-profile
```

Убедитесь, что ваш аккаунт имеет необходимые разрешения IAM (Compute OS Login или Compute OS Admin Login). **Нехватка памяти (OOM)** Если сборка Docker завершается с ошибкой `Killed` и `exit code 137`, ВМ была завершена из-за нехватки памяти. Перейдите на e2-small (минимум) или e2-medium (рекомендуется для надежных локальных сборок):

```bash
# Сначала остановите ВМ
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# Измените тип машины
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# Запустите ВМ
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

* * *

## Сервисные аккаунты (лучшие практики безопасности)

Для личного использования ваш аккаунт пользователя по умолчанию подходит. Для автоматизации или CI/CD пайплайнов создайте выделенный сервисный аккаунт с минимальными правами:

1.  Создайте сервисный аккаунт:
    
    Copy
    
    ```
    gcloud iam service-accounts create openclaw-deploy \
      --display-name="OpenClaw Deployment"
    ```
    
2.  Предоставьте роль Compute Instance Admin (или более узкую пользовательскую роль):
    
    Copy
    
    ```
    gcloud projects add-iam-policy-binding my-openclaw-project \
      --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
      --role="roles/compute.instanceAdmin.v1"
    ```
    

Избегайте использования роли Owner для автоматизации. Используйте принцип наименьших привилегий. Подробности о ролях IAM см. на [https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles).

* * *

## Следующие шаги

-   Настройте каналы связи: [Каналы](../channels.md)
-   Свяжите локальные устройства как узлы: [Узлы](../nodes.md)
-   Настройте шлюз: [Конфигурация шлюза](../gateway/configuration.md)

[Hetzner](./hetzner.md)[ВМ macOS](./macos-vm.md)