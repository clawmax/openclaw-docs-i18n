title: "Настройка удаленного шлюза для OpenClaw с использованием SSH-туннелирования"
description: "Узнайте, как настроить удаленный шлюз для OpenClaw.app с помощью SSH-туннелирования. Пошаговое руководство по настройке, автозапуску и устранению неполадок."
keywords: ["удаленный шлюз", "ssh туннелирование", "настройка openclaw", "проброс портов", "агент запуска", "устранение неполадок ssh", "удаленный доступ"]
---

  Удаленный доступ

  
# Настройка удаленного шлюза

OpenClaw.app использует SSH-туннелирование для подключения к удаленному шлюзу. Это руководство покажет вам, как его настроить.

## Обзор

## Быстрая настройка

### Шаг 1: Добавление конфигурации SSH

Отредактируйте файл `~/.ssh/config` и добавьте:

```
Host remote-gateway
    HostName <REMOTE_IP>          # e.g., 172.27.187.184
    User <REMOTE_USER>            # e.g., jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

Замените `<REMOTE_IP>` и `<REMOTE_USER>` на свои значения.

### Шаг 2: Копирование SSH-ключа

Скопируйте ваш открытый ключ на удаленную машину (введите пароль один раз):

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```

### Шаг 3: Установка токена шлюза

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```

### Шаг 4: Запуск SSH-туннеля

```bash
ssh -N remote-gateway &
```

### Шаг 5: Перезапуск OpenClaw.app

```bash
# Завершите работу OpenClaw.app (⌘Q), затем откройте снова:
open /path/to/OpenClaw.app
```

Теперь приложение будет подключаться к удаленному шлюзу через SSH-туннель.

* * *

## Автозапуск туннеля при входе в систему

Чтобы SSH-туннель запускался автоматически при входе в систему, создайте Launch Agent.

### Создание PLIST-файла

Сохраните это как `~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>ai.openclaw.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

### Загрузка Launch Agent

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist
```

Теперь туннель будет:

-   Автоматически запускаться при входе в систему
-   Перезапускаться при сбое
-   Работать в фоновом режиме

Примечание для устаревших версий: удалите оставшийся LaunchAgent `com.openclaw.ssh-tunnel`, если он присутствует.

* * *

## Устранение неполадок

**Проверка, запущен ли туннель:**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**Перезапуск туннеля:**

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.ssh-tunnel
```

**Остановка туннеля:**

```bash
launchctl bootout gui/$UID/ai.openclaw.ssh-tunnel
```

* * *

## Как это работает

| Компонент | Что он делает |
| --- | --- |
| `LocalForward 18789 127.0.0.1:18789` | Пробрасывает локальный порт 18789 на удаленный порт 18789 |
| `ssh -N` | SSH без выполнения удаленных команд (только проброс портов) |
| `KeepAlive` | Автоматически перезапускает туннель при сбое |
| `RunAtLoad` | Запускает туннель при загрузке агента |

OpenClaw.app подключается к `ws://127.0.0.1:18789` на вашем клиентском компьютере. SSH-туннель перенаправляет это соединение на порт 18789 удаленной машины, где работает Шлюз.

[Удаленный доступ](./remote.md)[Tailscale](./tailscale.md)