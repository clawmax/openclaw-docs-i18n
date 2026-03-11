

  Обзор платформ

  
# Приложение для Linux

Шлюз полностью поддерживается на Linux. **Рекомендуемая среда выполнения — Node**. Bun не рекомендуется для Шлюза (ошибки в WhatsApp/Telegram). Нативные приложения-компаньоны для Linux запланированы. Вклад приветствуется, если вы хотите помочь в разработке.

## Быстрый путь для начинающих (VPS)

1.  Установите Node 22+
2.  `npm i -g openclaw@latest`
3.  `openclaw onboard --install-daemon`
4.  С вашего ноутбука: `ssh -N -L 18789:127.0.0.1:18789 @`
5.  Откройте `http://127.0.0.1:18789/` и вставьте ваш токен

Пошаговое руководство по VPS: [exe.dev](../install/exe-dev.md)

## Установка

-   [Начало работы](../start/getting-started.md)
-   [Установка и обновления](../install/updating.md)
-   Дополнительные способы: [Bun (экспериментальный)](../install/bun.md), [Nix](../install/nix.md), [Docker](../install/docker.md)

## Шлюз

-   [Ранбук Шлюза](../gateway.md)
-   [Конфигурация](../gateway/configuration.md)

## Установка службы Шлюза (CLI)

Используйте одну из этих команд:

```bash
openclaw onboard --install-daemon
```

Или:

```bash
openclaw gateway install
```

Или:

```bash
openclaw configure
```

Выберите **Gateway service** при запросе. Восстановить/перенести:

```bash
openclaw doctor
```

## Управление системой (юнит systemd для пользователя)

По умолчанию OpenClaw устанавливает службу systemd для **пользователя**. Используйте **системную** службу для общих или постоянно работающих серверов. Полный пример юнита и инструкции находятся в [Ранбуке Шлюза](../gateway.md). Минимальная настройка: создайте `~/.config/systemd/user/openclaw-gateway[-].service`:

```ini
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Включите её:

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

[Приложение для macOS](./macos.md)[Windows (WSL2)](./windows.md)