title: "Исправление ошибки Chrome CDP порт 18800 в Linux для OpenClaw"
description: "Решите ошибку 'Failed to start Chrome CDP on port 18800' в OpenClaw. Узнайте, как установить Google Chrome или настроить snap Chromium в Ubuntu и Linux."
keywords: ["ошибка браузера openclaw", "chrome cdp порт 18800", "устранение неполадок браузера linux", "snap chromium openclaw", "установить google chrome linux", "режим attachonly браузера", "конфигурация openclaw", "chromium snap apparmor"]
---

  Браузер

  
# Устранение неполадок браузера

## Проблема: “Failed to start Chrome CDP on port 18800”

Сервер управления браузером OpenClaw не может запустить Chrome/Brave/Edge/Chromium с ошибкой:

```json
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```

### Причина

В Ubuntu (и многих дистрибутивах Linux) установка Chromium по умолчанию — это **snap-пакет**. Ограничения AppArmor в snap мешают OpenClaw запускать и отслеживать процесс браузера. Команда `apt install chromium` устанавливает заглушку, которая перенаправляет на snap:

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

Это НЕ настоящий браузер — это просто обёртка.

### Решение 1: Установить Google Chrome (Рекомендуется)

Установите официальный пакет Google Chrome `.deb`, который не изолирован snap:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # если есть ошибки зависимостей
```

Затем обновите конфигурацию OpenClaw (`~/.openclaw/openclaw.json`):

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### Решение 2: Использовать Snap Chromium в режиме только подключения

Если необходимо использовать snap Chromium, настройте OpenClaw на подключение к браузеру, запущенному вручную:

1.  Обновите конфигурацию:

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2.  Запустите Chromium вручную:

```
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3.  Опционально создайте пользовательский сервис systemd для автоматического запуска Chrome:

```bash
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Включите командой: `systemctl --user enable --now openclaw-browser.service`

### Проверка работы браузера

Проверьте статус:

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Протестируйте навигацию:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

### Справочник по конфигурации

| Параметр | Описание | По умолчанию |
| --- | --- | --- |
| `browser.enabled` | Включить управление браузером | `true` |
| `browser.executablePath` | Путь к исполняемому файлу браузера на базе Chromium (Chrome/Brave/Edge/Chromium) | определяется автоматически (предпочитает браузер по умолчанию, если он на базе Chromium) |
| `browser.headless` | Запускать без графического интерфейса | `false` |
| `browser.noSandbox` | Добавлять флаг `--no-sandbox` (необходим для некоторых настроек Linux) | `false` |
| `browser.attachOnly` | Не запускать браузер, только подключаться к существующему | `false` |
| `browser.cdpPort` | Порт Chrome DevTools Protocol | `18800` |

### Проблема: “Chrome extension relay is running, but no tab is connected”

Вы используете профиль `chrome` (ретранслятор расширения). Он ожидает, что расширение OpenClaw будет подключено к активной вкладке. Варианты решения:

1.  **Используйте управляемый браузер:** `openclaw browser start --browser-profile openclaw` (или установите `browser.defaultProfile: "openclaw"`).
2.  **Используйте ретранслятор расширения:** установите расширение, откройте вкладку и нажмите на иконку расширения OpenClaw, чтобы подключить его.

Примечания:

-   Профиль `chrome` использует ваш **системный браузер Chromium по умолчанию**, когда это возможно.
-   Локальные профили `openclaw` автоматически назначают `cdpPort`/`cdpUrl`; задавайте эти параметры только для удалённого CDP.

[Расширение Chrome](./chrome-extension.md)[Agent Send](./agent-send.md)