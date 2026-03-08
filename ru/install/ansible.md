title: "Руководство по развертыванию OpenClaw с помощью автоматического установщика Ansible"
description: "Узнайте, как установить OpenClaw безопасно с помощью автоматического установщика openclaw-ansible. Получите готовую к работе в production среду с фаерволом, VPN и изоляцией Docker за считанные минуты."
keywords: ["openclaw ansible", "ansible установщик", "развертывание openclaw", "tailscale vpn", "docker sandbox", "настройка production сервера", "установка с приоритетом безопасности", "автоматическое развертывание"]
---

  Другие способы установки

  
# Ansible

Рекомендуемый способ развертывания OpenClaw на production-серверах — с помощью **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — автоматического установщика с архитектурой, ориентированной на безопасность.

## Быстрый старт

Установка одной командой:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Полное руководство: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** Репозиторий openclaw-ansible является основным источником информации по развертыванию с помощью Ansible. Эта страница — краткий обзор.

## Что вы получаете

-   🔒 **Безопасность на основе фаервола**: UFW + изоляция Docker (доступны только SSH + Tailscale)
-   🔐 **Tailscale VPN**: Безопасный удаленный доступ без публичного открытия сервисов
-   🐳 **Docker**: Изолированные контейнеры-песочницы, привязка только к localhost
-   🛡️ **Защита в глубину**: 4-уровневая архитектура безопасности
-   🚀 **Настройка одной командой**: Полное развертывание за минуты
-   🔧 **Интеграция с Systemd**: Автозапуск при загрузке с усилением безопасности

## Требования

-   **ОС**: Debian 11+ или Ubuntu 20.04+
-   **Доступ**: Привилегии root или sudo
-   **Сеть**: Подключение к интернету для установки пакетов
-   **Ansible**: 2.14+ (устанавливается автоматически скриптом быстрого старта)

## Что устанавливается

Плейбук Ansible устанавливает и настраивает:

1.  **Tailscale** (mesh VPN для безопасного удаленного доступа)
2.  **Фаервол UFW** (только порты SSH + Tailscale)
3.  **Docker CE + Compose V2** (для песочниц агентов)
4.  **Node.js 22.x + pnpm** (зависимости среды выполнения)
5.  **OpenClaw** (работает на хосте, не в контейнере)
6.  **Сервис Systemd** (автозапуск с усилением безопасности)

Примечание: Шлюз работает **напрямую на хосте** (не в Docker), но песочницы агентов используют Docker для изоляции. Подробности см. в разделе [Песочницы](../gateway/sandboxing.md).

## Настройка после установки

После завершения установки переключитесь на пользователя openclaw:

```bash
sudo -i -u openclaw
```

Скрипт пост-установки проведет вас через:

1.  **Мастер настройки**: Конфигурация параметров OpenClaw
2.  **Вход в провайдер**: Подключение WhatsApp/Telegram/Discord/Signal
3.  **Тестирование шлюза**: Проверка установки
4.  **Настройка Tailscale**: Подключение к вашей VPN-сети

### Быстрые команды

```bash
# Проверить статус сервиса
sudo systemctl status openclaw

# Просмотр логов в реальном времени
sudo journalctl -u openclaw -f

# Перезапустить шлюз
sudo systemctl restart openclaw

# Вход в провайдер (запускать от пользователя openclaw)
sudo -i -u openclaw
openclaw channels login
```

## Архитектура безопасности

### 4-уровневая защита

1.  **Фаервол (UFW)**: Публично открыты только SSH (22) + Tailscale (41641/udp)
2.  **VPN (Tailscale)**: Шлюз доступен только через VPN-сеть
3.  **Изоляция Docker**: Цепочка iptables DOCKER-USER предотвращает открытие портов наружу
4.  **Усиление Systemd**: NoNewPrivileges, PrivateTmp, непривилегированный пользователь

### Проверка

Протестируйте внешнюю поверхность атаки:

```bash
nmap -p- YOUR_SERVER_IP
```

Должен показывать **только порт 22** (SSH) открытым. Все остальные сервисы (шлюз, Docker) заблокированы.

### Доступность Docker

Docker установлен для **песочниц агентов** (изолированное выполнение инструментов), а не для запуска самого шлюза. Шлюз привязывается только к localhost и доступен через VPN Tailscale. См. [Песочница для мульти-агентов и инструменты](../tools/multi-agent-sandbox-tools.md) для конфигурации песочницы.

## Ручная установка

Если вы предпочитаете ручной контроль над автоматизацией:

```bash
# 1. Установите необходимые компоненты
sudo apt update && sudo apt install -y ansible git

# 2. Клонируйте репозиторий
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Установите коллекции Ansible
ansible-galaxy collection install -r requirements.yml

# 4. Запустите плейбук
./run-playbook.sh

# Или запустите напрямую (затем вручную выполните /tmp/openclaw-setup.sh)
# ansible-playbook playbook.yml --ask-become-pass
```

## Обновление OpenClaw

Установщик Ansible настраивает OpenClaw для ручного обновления. См. [Обновление](./updating.md) для стандартного процесса обновления. Чтобы повторно запустить плейбук Ansible (например, для изменения конфигурации):

```bash
cd openclaw-ansible
./run-playbook.sh
```

Примечание: Это идемпотентная операция, её можно безопасно запускать многократно.

## Устранение неполадок

### Фаервол блокирует подключение

Если вы заблокированы:

-   Убедитесь, что сначала у вас есть доступ через VPN Tailscale
-   Доступ по SSH (порт 22) всегда разрешен
-   Шлюз **только** доступен через Tailscale по замыслу

### Сервис не запускается

```bash
# Проверить логи
sudo journalctl -u openclaw -n 100

# Проверить права доступа
sudo ls -la /opt/openclaw

# Протестировать ручной запуск
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Проблемы с песочницей Docker

```bash
# Проверить, что Docker запущен
sudo systemctl status docker

# Проверить образ песочницы
sudo docker images | grep openclaw-sandbox

# Собрать образ песочницы, если отсутствует
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### Не удается войти в провайдер

Убедитесь, что вы работаете от пользователя `openclaw`:

```bash
sudo -i -u openclaw
openclaw channels login
```

## Расширенная конфигурация

Подробная архитектура безопасности и устранение неполадок:

-   [Архитектура безопасности](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
-   [Технические детали](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
-   [Руководство по устранению неполадок](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## Связанное

-   [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — полное руководство по развертыванию
-   [Docker](./docker.md) — настройка шлюза в контейнере
-   [Песочницы](../gateway/sandboxing.md) — конфигурация песочниц агентов
-   [Песочница для мульти-агентов и инструменты](../tools/multi-agent-sandbox-tools.md) — изоляция для каждого агента

[Nix](./nix.md)[Bun (Экспериментальный)](./bun.md)