

  Обзор платформ

  
# DigitalOcean

## Цель

Запустить постоянный OpenClaw Gateway на DigitalOcean за **$6/месяц** (или $4/мес. с резервированием). Если вам нужен вариант за $0/месяц и вас не смущает ARM и специфичная настройка провайдера, см. [руководство по Oracle Cloud](./oracle.md).

## Сравнение стоимости (2026)

| Провайдер | Тариф | Характеристики | Цена/мес. | Примечания |
| --- | --- | --- | --- | --- |
| Oracle Cloud | Always Free ARM | до 4 OCPU, 24 ГБ ОЗУ | $0 | ARM, ограниченная мощность / особенности регистрации |
| Hetzner | CX22 | 2 vCPU, 4 ГБ ОЗУ | €3.79 (~$4) | Самый дешёвый платный вариант |
| DigitalOcean | Basic | 1 vCPU, 1 ГБ ОЗУ | $6 | Простой интерфейс, хорошая документация |
| Vultr | Cloud Compute | 1 vCPU, 1 ГБ ОЗУ | $6 | Много локаций |
| Linode | Nanode | 1 vCPU, 1 ГБ ОЗУ | $5 | Теперь часть Akamai |

**Выбор провайдера:**

-   DigitalOcean: самый простой UX + предсказуемая настройка (это руководство)
-   Hetzner: хорошее соотношение цена/производительность (см. [руководство по Hetzner](../install/hetzner.md))
-   Oracle Cloud: может быть $0/месяц, но более капризный и только ARM (см. [руководство по Oracle](./oracle.md))

* * *

## Предварительные требования

-   Аккаунт DigitalOcean ([регистрация с $200 бесплатного кредита](https://m.do.co/c/signup))
-   Пара SSH-ключей (или готовность использовать аутентификацию по паролю)
-   ~20 минут

## 1) Создание Droplet

> **⚠️** Используйте чистый базовый образ (Ubuntu 24.04 LTS). Избегайте сторонних образов из Marketplace, если вы не проверили их скрипты запуска и настройки брандмауэра по умолчанию.

1.  Войдите в [DigitalOcean](https://cloud.digitalocean.com/)
2.  Нажмите **Create → Droplets**
3.  Выберите:
    -   **Регион:** Ближайший к вам (или вашим пользователям)
    -   **Образ:** Ubuntu 24.04 LTS
    -   **Размер:** Basic → Regular → **$6/мес.** (1 vCPU, 1 ГБ ОЗУ, 25 ГБ SSD)
    -   **Аутентификация:** SSH-ключ (рекомендуется) или пароль
4.  Нажмите **Create Droplet**
5.  Запишите IP-адрес

## 2) Подключение по SSH

```bash
ssh root@YOUR_DROPLET_IP
```

## 3) Установка OpenClaw

```bash
# Update system
apt update && apt upgrade -y

# Install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Install OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# Verify
openclaw --version
```

## 4) Запуск начальной настройки

```bash
openclaw onboard --install-daemon
```

Мастер проведёт вас через:

-   Аутентификацию моделей (API-ключи или OAuth)
-   Настройку каналов (Telegram, WhatsApp, Discord и т.д.)
-   Токен шлюза (генерируется автоматически)
-   Установку демона (systemd)

## 5) Проверка шлюза

```bash
# Check status
openclaw status

# Check service
systemctl --user status openclaw-gateway.service

# View logs
journalctl --user -u openclaw-gateway.service -f
```

## 6) Доступ к панели управления

По умолчанию шлюз привязывается к loopback-интерфейсу. Для доступа к Control UI: **Вариант A: SSH-туннель (рекомендуется)**

```bash
# From your local machine
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# Then open: http://localhost:18789
```

**Вариант B: Tailscale Serve (HTTPS, только loopback)**

```bash
# On the droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configure Gateway to use Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

Откройте: `https:///` Примечания:

-   Serve оставляет шлюз доступным только на loopback и аутентифицирует трафик Control UI/WebSocket через заголовки идентификации Tailscale (бестокенная аутентификация предполагает доверенный хост шлюза; HTTP API по-прежнему требуют токен/пароль).
-   Чтобы требовать токен/пароль, установите `gateway.auth.allowTailscale: false` или используйте `gateway.auth.mode: "password"`.

**Вариант C: Привязка к Tailnet (без Serve)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Откройте: `http://<tailscale-ip>:18789` (требуется токен).

## 7) Подключение ваших каналов

### Telegram

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

### WhatsApp

```bash
openclaw channels login whatsapp
# Scan QR code
```

См. [Каналы](../channels.md) для других провайдеров.

* * *

## Оптимизация для 1 ГБ ОЗУ

У тарифа за $6 только 1 ГБ ОЗУ. Чтобы всё работало гладко:

### Добавьте swap (рекомендуется)

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### Используйте более лёгкую модель

Если возникают ошибки нехватки памяти (OOM), рассмотрите:

-   Использование моделей на основе API (Claude, GPT) вместо локальных моделей
-   Установку `agents.defaults.model.primary` на меньшую модель

### Мониторинг памяти

```
free -h
htop
```

* * *

## Сохранение состояния

Все данные хранятся в:

-   `~/.openclaw/` — конфигурация, учётные данные, данные сессий
-   `~/.openclaw/workspace/` — рабочее пространство (SOUL.md, память и т.д.)

Они сохраняются после перезагрузок. Регулярно создавайте их резервные копии:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## Бесплатная альтернатива Oracle Cloud

Oracle Cloud предлагает экземпляры **Always Free** ARM, которые значительно мощнее любого платного варианта здесь — за $0/месяц.

| Что вы получаете | Характеристики |
| --- | --- |
| **4 OCPU** | ARM Ampere A1 |
| **24 ГБ ОЗУ** | Более чем достаточно |
| **200 ГБ хранилища** | Блочный том |
| **Бесплатно навсегда** | Без списаний с карты |

**Ограничения:**

-   Регистрация может быть капризной (повторите попытку при неудаче)
-   Архитектура ARM — большинство вещей работает, но некоторым бинарникам нужны ARM-сборки

Полное руководство по настройке см. в [Oracle Cloud](./oracle.md). Советы по регистрации и устранению неполадок в процессе активации см. в этом [руководстве сообщества](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd).

* * *

## Устранение неполадок

### Шлюз не запускается

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

### Порт уже используется

```bash
lsof -i :18789
kill <PID>
```

### Нехватка памяти

```bash
# Check memory
free -h

# Add more swap
# Or upgrade to $12/mo droplet (2GB RAM)
```

* * *

## Смотрите также

-   [Руководство по Hetzner](../install/hetzner.md) — дешевле, мощнее
-   [Установка Docker](../install/docker.md) — контейнеризированная настройка
-   [Tailscale](../gateway/tailscale.md) — безопасный удалённый доступ
-   [Конфигурация](../gateway/configuration.md) — полный справочник по настройке

[iOS App](./ios.md)[Oracle Cloud](./oracle.md)

---