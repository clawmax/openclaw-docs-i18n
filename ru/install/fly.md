title: "Развертывание OpenClaw Gateway на Fly.io с постоянным хранилищем"
description: "Пошаговое руководство по размещению и развертыванию OpenClaw AI Gateway на Fly.io. Научитесь настраивать постоянное хранилище, устанавливать секреты, включать HTTPS и подключаться к Discord."
keywords: ["развертывание на fly.io", "шлюз openclaw", "постоянное хранилище", "хостинг discord бота", "настройка ai шлюза", "конфигурация fly.toml", "приватное развертывание", "бессерверный ai"]
---

  Хостинг и развертывание

  
# Fly.io

**Цель:** Работающий OpenClaw Gateway на машине [Fly.io](https://fly.io) с постоянным хранилищем, автоматическим HTTPS и доступом к Discord/каналам.

## Что вам понадобится

-   Установленный [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/)
-   Аккаунт Fly.io (подойдет бесплатный тариф)
-   Авторизация модели: API-ключ для выбранного провайдера моделей
-   Учетные данные каналов: токен Discord бота, токен Telegram и т.д.

## Быстрый путь для начинающих

1.  Клонировать репозиторий → настроить `fly.toml`
2.  Создать приложение + том → установить секреты
3.  Развернуть с помощью `fly deploy`
4.  Подключиться по SSH для создания конфигурации или использовать Control UI

## 1) Создание приложения Fly

```bash
# Клонировать репозиторий
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Создать новое приложение Fly (выберите свое имя)
fly apps create my-openclaw

# Создать постоянный том (обычно достаточно 1 ГБ)
fly volumes create openclaw_data --size 1 --region iad
```

**Совет:** Выберите регион, близкий к вам. Распространенные варианты: `lhr` (Лондон), `iad` (Вирджиния), `sjc` (Сан-Хосе).

## 2) Настройка fly.toml

Отредактируйте `fly.toml`, чтобы он соответствовал имени вашего приложения и требованиям. **Примечание по безопасности:** Конфигурация по умолчанию открывает публичный URL. Для защищенного развертывания без публичного IP см. [Приватное развертывание](#приватное-развертывание-защищенное) или используйте `fly.private.toml`.

```
app = "my-openclaw"  # Ваше имя приложения
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**Ключевые настройки:**

| Настройка | Зачем |
| --- | --- |
| `--bind lan` | Привязывается к `0.0.0.0`, чтобы прокси Fly мог достучаться до шлюза |
| `--allow-unconfigured` | Запускается без конфигурационного файла (вы создадите его после) |
| `internal_port = 3000` | Должен совпадать с `--port 3000` (или `OPENCLAW_GATEWAY_PORT`) для проверок здоровья Fly |
| `memory = "2048mb"` | 512 МБ слишком мало; рекомендуется 2 ГБ |
| `OPENCLAW_STATE_DIR = "/data"` | Сохраняет состояние на томе |

## 3) Установка секретов

```bash
# Обязательно: Токен шлюза (для привязки не к loopback)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# API-ключи провайдеров моделей
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Опционально: Другие провайдеры
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Токены каналов
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**Примечания:**

-   Привязка не к loopback (`--bind lan`) требует `OPENCLAW_GATEWAY_TOKEN` для безопасности.
-   Относитесь к этим токенам как к паролям.
-   **Предпочитайте переменные окружения конфигурационному файлу** для всех API-ключей и токенов. Это убережет секреты от случайного попадания в `openclaw.json`, где они могут быть раскрыты или записаны в логи.

## 4) Развертывание

```bash
fly deploy
```

Первое развертывание собирает Docker-образ (~2-3 минуты). Последующие развертывания быстрее. После развертывания проверьте:

```bash
fly status
fly logs
```

Вы должны увидеть:

```ini
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```

## 5) Создание конфигурационного файла

Подключитесь по SSH к машине, чтобы создать правильную конфигурацию:

```bash
fly ssh console
```

Создайте каталог конфигурации и файл:

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**Примечание:** При `OPENCLAW_STATE_DIR=/data` путь к конфигурации — `/data/openclaw.json`. **Примечание:** Токен Discord может поступать из:

-   Переменной окружения: `DISCORD_BOT_TOKEN` (рекомендуется для секретов)
-   Конфигурационного файла: `channels.discord.token`

Если используется переменная окружения, добавлять токен в конфигурацию не нужно. Шлюз автоматически читает `DISCORD_BOT_TOKEN`. Перезапустите для применения:

```
exit
fly machine restart <machine-id>
```

## 6) Доступ к шлюзу

### Панель управления (Control UI)

Откройте в браузере:

```bash
fly open
```

Или перейдите по адресу `https://my-openclaw.fly.dev/` Вставьте ваш токен шлюза (тот, что из `OPENCLAW_GATEWAY_TOKEN`) для аутентификации.

### Логи

```bash
fly logs              # Логи в реальном времени
fly logs --no-tail    # Последние логи
```

### SSH-консоль

```bash
fly ssh console
```

## Устранение неполадок

### ”App is not listening on expected address”

Шлюз привязывается к `127.0.0.1` вместо `0.0.0.0`. **Решение:** Добавьте `--bind lan` в команду процесса в `fly.toml`.

### Сбои проверок здоровья / connection refused

Fly не может достучаться до шлюза на настроенном порту. **Решение:** Убедитесь, что `internal_port` совпадает с портом шлюза (установите `--port 3000` или `OPENCLAW_GATEWAY_PORT=3000`).

### OOM / Проблемы с памятью

Контейнер постоянно перезапускается или завершается. Признаки: `SIGABRT`, `v8::internal::Runtime_AllocateInYoungGeneration` или тихие перезапуски. **Решение:** Увеличьте память в `fly.toml`:

```
[[vm]]
  memory = "2048mb"
```

Или обновите существующую машину:

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**Примечание:** 512 МБ слишком мало. 1 ГБ может работать, но может привести к OOM под нагрузкой или с подробным логированием. **Рекомендуется 2 ГБ.**

### Проблемы с блокировкой шлюза

Шлюз отказывается запускаться с ошибками "already running". Это происходит, когда контейнер перезапускается, но файл блокировки PID сохраняется на томе. **Решение:** Удалите файл блокировки:

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

Файл блокировки находится по пути `/data/gateway.*.lock` (не в подкаталоге).

### Конфигурация не читается

При использовании `--allow-unconfigured` шлюз создает минимальную конфигурацию. Ваша пользовательская конфигурация в `/data/openclaw.json` должна читаться при перезапуске. Проверьте существование конфигурации:

```bash
fly ssh console --command "cat /data/openclaw.json"
```

### Запись конфигурации через SSH

Команда `fly ssh console -C` не поддерживает перенаправление вывода оболочки. Чтобы записать конфигурационный файл:

```bash
# Используйте echo + tee (передача из локальной системы на удаленную)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# Или используйте sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**Примечание:** `fly sftp` может завершиться ошибкой, если файл уже существует. Удалите его сначала:

```bash
fly ssh console --command "rm /data/openclaw.json"
```

### Состояние не сохраняется

Если вы теряете учетные данные или сеансы после перезапуска, каталог состояния записывается в файловую систему контейнера. **Решение:** Убедитесь, что `OPENCLAW_STATE_DIR=/data` установлен в `fly.toml` и выполните повторное развертывание.

## Обновления

```bash
# Получить последние изменения
git pull

# Повторно развернуть
fly deploy

# Проверить состояние
fly status
fly logs
```

### Обновление команды машины

Если вам нужно изменить команду запуска без полного повторного развертывания:

```bash
# Получить ID машины
fly machines list

# Обновить команду
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# Или с увеличением памяти
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**Примечание:** После `fly deploy` команда машины может сброситься к той, что указана в `fly.toml`. Если вы вносили изменения вручную, примените их снова после развертывания.

## Приватное развертывание (Защищенное)

По умолчанию Fly выделяет публичные IP-адреса, делая ваш шлюз доступным по адресу `https://your-app.fly.dev`. Это удобно, но означает, что ваше развертывание может быть обнаружено интернет-сканерами (Shodan, Censys и т.д.). Для защищенного развертывания **без публичного доступа** используйте приватный шаблон.

### Когда использовать приватное развертывание

-   Вы совершаете только **исходящие** вызовы/сообщения (без входящих вебхуков)
-   Вы используете **туннели ngrok или Tailscale** для любых обратных вызовов вебхуков
-   Вы получаете доступ к шлюзу через **SSH, прокси или WireGuard**, а не через браузер
-   Вы хотите **скрыть** развертывание от интернет-сканеров

### Настройка

Используйте `fly.private.toml` вместо стандартной конфигурации:

```bash
# Развернуть с приватной конфигурацией
fly deploy -c fly.private.toml
```

Или преобразуйте существующее развертывание:

```bash
# Список текущих IP
fly ips list -a my-openclaw

# Освободить публичные IP
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# Переключиться на приватную конфигурацию, чтобы будущие развертывания не выделяли публичные IP снова
# (удалите [http_service] или разверните с приватным шаблоном)
fly deploy -c fly.private.toml

# Выделить только приватный IPv6
fly ips allocate-v6 --private -a my-openclaw
```

После этого `fly ips list` должен показывать только IP-адрес типа `private`:

```bash
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```

### Доступ к приватному развертыванию

Поскольку публичного URL нет, используйте один из этих методов: **Вариант 1: Локальный прокси (проще всего)**

```bash
# Перенаправить локальный порт 3000 на приложение
fly proxy 3000:3000 -a my-openclaw

# Затем откройте http://localhost:3000 в браузере
```

**Вариант 2: VPN WireGuard**

```bash
# Создать конфигурацию WireGuard (однократно)
fly wireguard create

# Импортировать в клиент WireGuard, затем получить доступ через внутренний IPv6
# Пример: http://[fdaa:x:x:x:x::x]:3000
```

**Вариант 3: Только SSH**

```bash
fly ssh console -a my-openclaw
```

### Вебхуки с приватным развертыванием

Если вам нужны обратные вызовы вебхуков (Twilio, Telnyx и т.д.) без публичного доступа:

1.  **Туннель ngrok** - Запустите ngrok внутри контейнера или как sidecar
2.  **Tailscale Funnel** - Откройте определенные пути через Tailscale
3.  **Только исходящие** - Некоторые провайдеры (Twilio) хорошо работают для исходящих вызовов без вебхуков

Пример конфигурации голосовых вызовов с ngrok:

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" },
          "webhookSecurity": {
            "allowedHosts": ["example.ngrok.app"]
          }
        }
      }
    }
  }
}
```

Туннель ngrok работает внутри контейнера и предоставляет публичный URL вебхука, не открывая само приложение Fly. Установите `webhookSecurity.allowedHosts` на публичное имя хоста туннеля, чтобы пересылаемые заголовки хоста принимались.

### Преимущества безопасности

| Аспект | Публичное | Приватное |
| --- | --- | --- |
| Интернет-сканеры | Обнаруживаемо | Скрыто |
| Прямые атаки | Возможны | Блокированы |
| Доступ к Control UI | Браузер | Прокси/VPN |
| Доставка вебхуков | Прямая | Через туннель |

## Примечания

-   Fly.io использует **архитектуру x86** (не ARM)
-   Dockerfile совместим с обеими архитектурами
-   Для онбординга WhatsApp/Telegram используйте `fly ssh console`
-   Постоянные данные хранятся на томе по пути `/data`
-   Signal требует Java + signal-cli; используйте пользовательский образ и поддерживайте память от 2 ГБ+.

## Стоимость

С рекомендуемой конфигурацией (`shared-cpu-2x`, 2 ГБ ОЗУ):

-   ~$10-15/месяц в зависимости от использования
-   Бесплатный тариф включает некоторый лимит

Подробности см. в [ценообразовании Fly.io](https://fly.io/docs/about/pricing/).

[Хостинг на VPS](../vps.md)[Hetzner](./hetzner.md)