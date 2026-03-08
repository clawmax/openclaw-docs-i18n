title: "Руководство по установке и настройке OpenClaw на Windows через WSL2"
description: "Узнайте, как установить и запустить OpenClaw на Windows с помощью WSL2. Пошаговое руководство по настройке, службе шлюза, автозапуску и доступу по локальной сети."
keywords: ["openclaw windows", "настройка wsl2", "установка openclaw", "шлюз windows", "проброс портов wsl2", "openclaw wsl2", "автоматизация windows", "systemd wsl2"]
---

  Обзор платформ

  
# Windows (WSL2)

Запуск OpenClaw на Windows рекомендуется **через WSL2** (рекомендуется Ubuntu). CLI и Шлюз работают внутри Linux, что обеспечивает согласованность среды выполнения и значительно повышает совместимость инструментов (Node/Bun/pnpm, бинарные файлы Linux, навыки). Нативная установка на Windows может быть сложнее. WSL2 даёт полный опыт работы с Linux — одна команда для установки: `wsl --install`. Нативные приложения-компаньоны для Windows запланированы.

## Установка (WSL2)

-   [Начало работы](../start/getting-started.md) (используйте внутри WSL)
-   [Установка и обновления](../install/updating.md)
-   Официальное руководство по WSL2 (Microsoft): [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)

## Шлюз

-   [Ранбук шлюза](../gateway.md)
-   [Конфигурация](../gateway/configuration.md)

## Установка службы шлюза (CLI)

Внутри WSL2:

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

Выберите **Службу шлюза** при запросе. Восстановление/миграция:

```bash
openclaw doctor
```

## Автозапуск шлюза до входа в Windows

Для безголовых (headless) конфигураций необходимо обеспечить запуск всей цепочки загрузки, даже когда никто не входит в Windows.

### 1) Обеспечьте работу пользовательских служб без входа в систему

Внутри WSL:

```bash
sudo loginctl enable-linger "$(whoami)"
```

### 2) Установите пользовательскую службу шлюза OpenClaw

Внутри WSL:

```bash
openclaw gateway install
```

### 3) Запускайте WSL автоматически при загрузке Windows

В PowerShell от имени Администратора:

```bash
schtasks /create /tn "WSL Boot" /tr "wsl.exe -d Ubuntu --exec /bin/true" /sc onstart /ru SYSTEM
```

Замените `Ubuntu` на имя вашего дистрибутива из:

```bash
wsl --list --verbose
```

### Проверка цепочки запуска

После перезагрузки (до входа в Windows) проверьте из WSL:

```bash
systemctl --user is-enabled openclaw-gateway
systemctl --user status openclaw-gateway --no-pager
```

## Продвинутая настройка: предоставление доступа к службам WSL по локальной сети (portproxy)

WSL имеет свою собственную виртуальную сеть. Если другой машине нужен доступ к службе, работающей **внутри WSL** (SSH, локальный TTS-сервер или Шлюз), необходимо пробросить порт Windows на текущий IP-адрес WSL. IP-адрес WSL меняется после перезапусков, поэтому правило проброса портов может потребовать обновления. Пример (PowerShell **от имени Администратора**):

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Разрешите порт в брандмауэре Windows (однократно):

```
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

Обновите правило portproxy после перезапуска WSL:

```
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

Примечания:

-   SSH с другой машины должен указывать на **IP-адрес хоста Windows** (пример: `ssh user@windows-host -p 2222`).
-   Удалённые узлы должны указывать на **доступный** URL шлюза (не `127.0.0.1`); используйте `openclaw status --all` для проверки.
-   Используйте `listenaddress=0.0.0.0` для доступа по локальной сети; `127.0.0.1` оставляет доступ только локальным.
-   Для автоматизации зарегистрируйте Планировщик заданий (Scheduled Task) для выполнения шага обновления при входе в систему.

## Пошаговая установка WSL2

### 1) Установите WSL2 + Ubuntu

Откройте PowerShell (Администратор):

```bash
wsl --install
# Или выберите дистрибутив явно:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Перезагрузитесь, если запросит Windows.

### 2) Включите systemd (требуется для установки шлюза)

В терминале WSL:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Затем из PowerShell:

```bash
wsl --shutdown
```

Снова откройте Ubuntu и проверьте:

```bash
systemctl --user status
```

### 3) Установите OpenClaw (внутри WSL)

Следуйте инструкциям "Начало работы" для Linux внутри WSL:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # автоматически установит зависимости UI при первом запуске
pnpm build
openclaw onboard
```

Полное руководство: [Начало работы](../start/getting-started.md)

## Приложение-компаньон для Windows

У нас пока нет приложения-компаньона для Windows. Вклад в разработку приветствуется, если вы хотите помочь в его создании.

[Приложение для Linux](./linux.md)[Приложение для Android](./android.md)