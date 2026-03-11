

  Хостинг и развертывание

  
# VPS Хостинг

Этот раздел содержит ссылки на поддерживаемые руководства по VPS/хостингу и объясняет, как работают облачные развертывания на высоком уровне.

## Выберите провайдера

-   **Railway** (одно‑кликовая настройка + браузер): [Railway](./install/railway.md)
-   **Northflank** (одно‑кликовая настройка + браузер): [Northflank](./install/northflank.md)
-   **Oracle Cloud (Always Free)**: [Oracle](./platforms/oracle.md) — $0/месяц (Always Free, ARM; могут быть нюансы с доступностью и регистрацией)
-   **Fly.io**: [Fly.io](./install/fly.md)
-   **Hetzner (Docker)**: [Hetzner](./install/hetzner.md)
-   **GCP (Compute Engine)**: [GCP](./install/gcp.md)
-   **exe.dev** (VM + HTTPS прокси): [exe.dev](./install/exe-dev.md)
-   **AWS (EC2/Lightsail/free tier)**: также хорошо работает. Видео-руководство: [https://x.com/techfrenAJ/status/2014934471095812547](https://x.com/techfrenAJ/status/2014934471095812547)

## Как работают облачные настройки

-   **Шлюз работает на VPS** и хранит состояние и рабочее пространство.
-   Вы подключаетесь со своего ноутбука/телефона через **Панель управления** или **Tailscale/SSH**.
-   Рассматривайте VPS как источник истины и **регулярно создавайте резервные копии** состояния и рабочего пространства.
-   Безопасная настройка по умолчанию: держите Шлюз на loopback-интерфейсе и получайте доступ к нему через SSH-туннель или Tailscale Serve. Если вы привязываетесь к `lan`/`tailnet`, требуйте `gateway.auth.token` или `gateway.auth.password`.

Удаленный доступ: [Удаленный шлюз](./gateway/remote.md)  
Раздел платформ: [Платформы](./platforms.md)

## Общий корпоративный агент на VPS

Это допустимая настройка, когда пользователи находятся в одном доверенном периметре (например, одна команда компании), и агент используется только для бизнес-задач.

-   Держите его на выделенной среде выполнения (VPS/VM/контейнер + выделенный пользователь/учетные записи ОС).
-   Не входите в этой среде выполнения в личные учетные записи Apple/Google или личные профили браузера/менеджера паролей.
-   Если пользователи не доверяют друг другу, разделите их по шлюзам/хостам/пользователям ОС.

Подробности модели безопасности: [Безопасность](./gateway/security.md)

## Использование узлов с VPS

Вы можете держать Шлюз в облаке и подключить к нему **узлы** на ваших локальных устройствах (Mac/iOS/Android/headless). Узлы предоставляют локальный экран/камеру/холст и возможности `system.run`, в то время как Шлюз остается в облаке. Документация: [Узлы](./nodes.md), [CLI узлов](./cli/nodes.md)

## Оптимизация запуска для небольших VM и ARM-хостов

Если команды CLI выполняются медленно на маломощных VM (или ARM-хостах), включите кэш компиляции модулей Node:

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF'
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

-   `NODE_COMPILE_CACHE` улучшает время запуска повторяющихся команд.
-   `OPENCLAW_NO_RESPAWN=1` позволяет избежать дополнительных накладных расходов при запуске из-за пути самоперезапуска.
-   Первый запуск команды прогревает кэш; последующие запуски выполняются быстрее.
-   Для специфики Raspberry Pi см. [Raspberry Pi](./platforms/raspberry-pi.md).

### Контрольный список настройки systemd (опционально)

Для VM-хостов, использующих `systemd`, рассмотрите:

-   Добавьте переменные окружения службы для стабильного пути запуска:
    -   `OPENCLAW_NO_RESPAWN=1`
    -   `NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache`
-   Явно задайте поведение перезапуска:
    -   `Restart=always`
    -   `RestartSec=2`
    -   `TimeoutStartSec=90`
-   Предпочитайте диски на SSD для путей состояния/кэша, чтобы уменьшить задержки холодного старта из-за случайных операций ввода-вывода.

Пример:

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

Как политики `Restart=` помогают автоматическому восстановлению: [systemd может автоматизировать восстановление служб](https://www.redhat.com/en/blog/systemd-automate-recovery).

[Удаление](./install/uninstall.md)[Fly.io](./install/fly.md)