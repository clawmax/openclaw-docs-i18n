

  Обзор платформ

  
# Raspberry Pi

## Цель

Запустить постоянный, всегда работающий шлюз OpenClaw на Raspberry Pi за **~$35-80** единовременно (без ежемесячных платежей). Идеально для:

*   Круглосуточного персонального AI-ассистента
*   Центра управления умным домом
*   Маломощного, всегда доступного бота для Telegram/WhatsApp

## Требования к оборудованию

| Модель Pi | ОЗУ | Работает? | Примечания |
| --- | --- | --- | --- |
| **Pi 5** | 4GB/8GB | ✅ Лучше всего | Самый быстрый, рекомендуется |
| **Pi 4** | 4GB | ✅ Хорошо | Оптимальный выбор для большинства |
| **Pi 4** | 2GB | ✅ Нормально | Работает, добавьте swap |
| **Pi 4** | 1GB | ⚠️ На пределе | Возможно с swap, минимальная конфигурация |
| **Pi 3B+** | 1GB | ⚠️ Медленно | Работает, но тормозит |
| **Pi Zero 2 W** | 512MB | ❌ | Не рекомендуется |

**Минимальные характеристики:** 1 ГБ ОЗУ, 1 ядро, 500 МБ диска  
**Рекомендуемые:** 2+ ГБ ОЗУ, 64-битная ОС, SD-карта 16+ ГБ (или USB SSD)

## Что вам понадобится

*   Raspberry Pi 4 или 5 (рекомендуется 2 ГБ+ ОЗУ)
*   Карта MicroSD (16 ГБ+) или USB SSD (лучшая производительность)
*   Блок питания (рекомендуется официальный от Pi)
*   Сетевое подключение (Ethernet или WiFi)
*   ~30 минут

## 1) Запись ОС

Используйте **Raspberry Pi OS Lite (64-bit)** — для headless-сервера графический интерфейс не нужен.

1.  Скачайте [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2.  Выберите ОС: **Raspberry Pi OS Lite (64-bit)**
3.  Нажмите на значок шестерёнки (⚙️) для предварительной настройки:
    *   Установите имя хоста: `gateway-host`
    *   Включите SSH
    *   Задайте имя пользователя/пароль
    *   Настройте WiFi (если не используете Ethernet)
4.  Запишите образ на вашу SD-карту / USB-накопитель
5.  Вставьте и загрузите Pi

## 2) Подключение по SSH

```bash
ssh user@gateway-host
# или используйте IP-адрес
ssh user@192.168.x.x
```

## 3) Настройка системы

```bash
# Обновление системы
sudo apt update && sudo apt upgrade -y

# Установка основных пакетов
sudo apt install -y git curl build-essential

# Установка часового пояса (важно для cron/напоминаний)
sudo timedatectl set-timezone America/Chicago  # Измените на свой часовой пояс
```

## 4) Установка Node.js 22 (ARM64)

```bash
# Установка Node.js через NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Проверка
node --version  # Должна отображаться версия v22.x.x
npm --version
```

## 5) Добавление Swap (Важно для 2 ГБ ОЗУ и меньше)

Swap предотвращает сбои из-за нехватки памяти:

```bash
# Создание файла подкачки размером 2 ГБ
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Сделать постоянным
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Оптимизация для малого объёма ОЗУ (уменьшение swappiness)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## 6) Установка OpenClaw

### Вариант A: Стандартная установка (Рекомендуется)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Вариант B: Установка для экспериментов (Для модификаций)

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

Установка для экспериментов даёт прямой доступ к логам и коду — полезно для отладки проблем, специфичных для ARM.

## 7) Запуск первоначальной настройки

```bash
openclaw onboard --install-daemon
```

Следуйте указаниям мастера:

1.  **Режим шлюза:** Local
2.  **Аутентификация:** Рекомендуются API-ключи (OAuth может работать нестабильно на headless Pi)
3.  **Каналы:** Telegram — самый простой для начала
4.  **Демон:** Да (systemd)

## 8) Проверка установки

```bash
# Проверить статус
openclaw status

# Проверить службу
sudo systemctl status openclaw

# Просмотреть логи
journalctl -u openclaw -f
```

## 9) Доступ к панели управления

Поскольку Pi работает без монитора, используйте SSH-туннель:

```bash
# С вашего ноутбука/ПК
ssh -L 18789:localhost:18789 user@gateway-host

# Затем откройте в браузере
open http://localhost:18789
```

Или используйте Tailscale для постоянного доступа:

```bash
# На Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Обновить конфигурацию
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

* * *

## Оптимизация производительности

### Используйте USB SSD (Значительное улучшение)

SD-карты медленные и изнашиваются. USB SSD значительно улучшает производительность:

```bash
# Проверить, загружаемся ли с USB
lsblk
```

Смотрите [руководство по загрузке Pi с USB](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) для настройки.

### Ускорение запуска CLI (кэш компиляции модулей)

На маломощных Pi включите кэш компиляции модулей Node, чтобы повторные запуски CLI были быстрее:

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF' # pragma: allowlist secret
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

Примечания:

*   `NODE_COMPILE_CACHE` ускоряет последующие запуски (`status`, `health`, `--help`).
*   `/var/tmp` сохраняется после перезагрузок лучше, чем `/tmp`.
*   `OPENCLAW_NO_RESPAWN=1` избегает дополнительных затрат на самоперезапуск CLI.
*   Первый запуск прогревает кэш; последующие запуски получают наибольшую выгоду.

### Настройка запуска systemd (опционально)

Если этот Pi в основном работает под OpenClaw, добавьте drop-in для службы, чтобы уменьшить "дёргание" при перезапуске и сохранить стабильность окружения при старте:

```bash
sudo systemctl edit openclaw
```

```ini
[Service]
Environment=OPENCLAW_NO_RESPAWN=1
Environment=NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
Restart=always
RestartSec=2
TimeoutStartSec=90
```

Затем примените:

```bash
sudo systemctl daemon-reload
sudo systemctl restart openclaw
```

По возможности храните состояние/кэш OpenClaw на хранилище с SSD, чтобы избежать узких мест при случайных операциях ввода-вывода на SD-карте во время холодных стартов. Как политики `Restart=` помогают автоматическому восстановлению: [systemd может автоматизировать восстановление служб](https://www.redhat.com/en/blog/systemd-automate-recovery).

### Снижение потребления памяти

```bash
# Отключить выделение памяти для GPU (headless)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Отключить Bluetooth, если не нужен
sudo systemctl disable bluetooth
```

### Мониторинг ресурсов

```bash
# Проверить память
free -h

# Проверить температуру CPU
vcgencmd measure_temp

# Мониторинг в реальном времени
htop
```

* * *

## Особенности архитектуры ARM

### Совместимость бинарных файлов

Большинство функций OpenClaw работают на ARM64, но некоторым внешним бинарным файлам могут потребоваться сборки для ARM:

| Инструмент | Статус ARM64 | Примечания |
| --- | --- | --- |
| Node.js | ✅ | Работает отлично |
| WhatsApp (Baileys) | ✅ | Чистый JS, проблем нет |
| Telegram | ✅ | Чистый JS, проблем нет |
| gog (Gmail CLI) | ⚠️ | Проверьте наличие сборки для ARM |
| Chromium (браузер) | ✅ | `sudo apt install chromium-browser` |

Если навык не работает, проверьте, есть ли у его бинарного файла сборка для ARM. Многие инструменты на Go/Rust имеют; некоторые — нет.

### 32-битная vs 64-битная

**Всегда используйте 64-битную ОС.** Node.js и многие современные инструменты требуют её. Проверьте:

```bash
uname -m
# Должно отображаться: aarch64 (64-bit), а не armv7l (32-bit)
```

* * *

## Рекомендуемая настройка моделей

Поскольку Pi — это только шлюз (модели работают в облаке), используйте модели на основе API:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**Не пытайтесь запускать локальные LLM на Pi** — даже маленькие модели слишком медленные. Пусть Claude/GPT выполняют тяжёлую работу.

* * *

## Автозапуск при загрузке

Мастер первоначальной настройки делает это, но для проверки:

```bash
# Проверить, включена ли служба
sudo systemctl is-enabled openclaw

# Включить, если нет
sudo systemctl enable openclaw

# Запустить при загрузке
sudo systemctl start openclaw
```

* * *

## Устранение неполадок

### Нехватка памяти (OOM)

```bash
# Проверить память
free -h

# Добавить больше swap (см. Шаг 5)
# Или сократить количество служб, работающих на Pi
```

### Низкая производительность

*   Используйте USB SSD вместо SD-карты
*   Отключите неиспользуемые службы: `sudo systemctl disable cups bluetooth avahi-daemon`
*   Проверьте троттлинг CPU: `vcgencmd get_throttled` (должно вернуть `0x0`)

### Служба не запускается

```bash
# Проверить логи
journalctl -u openclaw --no-pager -n 100

# Распространённое решение: пересобрать
cd ~/openclaw  # если используется установка для экспериментов
npm run build
sudo systemctl restart openclaw
```

### Проблемы с ARM-бинарниками

Если навык завершается с ошибкой "exec format error":

1.  Проверьте, есть ли у бинарного файла сборка для ARM64
2.  Попробуйте собрать из исходного кода
3.  Или используйте Docker-контейнер с поддержкой ARM

### Обрывы WiFi

Для headless Pi на WiFi:

```bash
# Отключить управление питанием WiFi
sudo iwconfig wlan0 power off

# Сделать постоянным
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

* * *

## Сравнение стоимости

| Настройка | Единовременная стоимость | Ежемесячная стоимость | Примечания |
| --- | --- | --- | --- |
| **Pi 4 (2GB)** | ~$45 | $0 | \+ электричество (~$5/год) |
| **Pi 4 (4GB)** | ~$55 | $0 | Рекомендуется |
| **Pi 5 (4GB)** | ~$60 | $0 | Лучшая производительность |
| **Pi 5 (8GB)** | ~$80 | $0 | Избыточно, но с запасом на будущее |
| DigitalOcean | $0 | $6/мес | $72/год |
| Hetzner | $0 | €3.79/мес | ~$50/год |

**Окупаемость:** Pi окупает себя за ~6-12 месяцев по сравнению с облачным VPS.

* * *

## Смотрите также

*   [Руководство по Linux](./linux.md) — общая настройка Linux
*   [Руководство по DigitalOcean](./digitalocean.md) — облачная альтернатива
*   [Руководство по Hetzner](../install/hetzner.md) — настройка Docker
*   [Tailscale](../gateway/tailscale.md) — удалённый доступ
*   [Узлы](../nodes.md) — подключите ваш ноутбук/телефон к шлюзу на Pi

[Oracle Cloud](./oracle.md)[Настройка для разработки на macOS](./mac/dev-setup.md)