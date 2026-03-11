

  Обслуживание

  
# Удаление

Два пути:

-   **Простой путь**, если `openclaw` всё ещё установлен.
-   **Ручное удаление службы**, если CLI удалён, но служба всё ещё работает.

## Простой путь (CLI всё ещё установлен)

Рекомендуется: использовать встроенный деинсталлятор:

```bash
openclaw uninstall
```

Неинтерактивный режим (для автоматизации / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

Ручные шаги (тот же результат):

1.  Остановите службу gateway:

```bash
openclaw gateway stop
```

2.  Удалите службу gateway (launchd/systemd/schtasks):

```bash
openclaw gateway uninstall
```

3.  Удалите состояние и конфигурацию:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

Если вы задали `OPENCLAW_CONFIG_PATH` в пользовательском расположении вне каталога состояния, удалите и этот файл.

4.  Удалите вашу рабочую область (опционально, удаляет файлы агентов):

```bash
rm -rf ~/.openclaw/workspace
```

5.  Удалите установленный CLI (выберите тот, который вы использовали):

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6.  Если вы устанавливали приложение для macOS:

```bash
rm -rf /Applications/OpenClaw.app
```

Примечания:

-   Если вы использовали профили (`--profile` / `OPENCLAW_PROFILE`), повторите шаг 3 для каждого каталога состояния (по умолчанию это `~/.openclaw-`).
-   В удалённом режиме каталог состояния находится на **хосте gateway**, поэтому выполните шаги 1-4 и там.

## Ручное удаление службы (CLI не установлен)

Используйте этот способ, если служба gateway продолжает работать, а `openclaw` отсутствует.

### macOS (launchd)

Метка по умолчанию — `ai.openclaw.gateway` (или `ai.openclaw.`; устаревшие `com.openclaw.*` могут всё ещё существовать):

```bash
launchctl bootout gui/$UID/ai.openclaw.gateway
rm -f ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

Если вы использовали профиль, замените метку и имя plist-файла на `ai.openclaw.`. Удалите все оставшиеся устаревшие plist-файлы `com.openclaw.*`, если они есть.

### Linux (systemd user unit)

Имя юнита по умолчанию — `openclaw-gateway.service` (или `openclaw-gateway-.service`):

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

### Windows (Scheduled Task)

Имя задачи по умолчанию — `OpenClaw Gateway` (или `OpenClaw Gateway ()`). Скрипт задачи находится в вашем каталоге состояния.

```bash
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

Если вы использовали профиль, удалите соответствующее имя задачи и файл `~\.openclaw-\gateway.cmd`.

## Обычная установка vs установка из исходников

### Обычная установка (install.sh / npm / pnpm / bun)

Если вы использовали `https://openclaw.ai/install.sh` или `install.ps1`, CLI был установлен с помощью `npm install -g openclaw@latest`. Удалите его командой `npm rm -g openclaw` (или `pnpm remove -g` / `bun remove -g`, если вы устанавливали таким способом).

### Установка из исходников (git clone)

Если вы запускали из клонированного репозитория (`git clone` + `openclaw ...` / `bun run openclaw ...`):

1.  Удалите службу gateway **до** удаления репозитория (используйте простой путь выше или ручное удаление службы).
2.  Удалите каталог репозитория.
3.  Удалите состояние и рабочую область, как показано выше.

[Руководство по миграции](./migrating.md)[Хостинг на VPS](../vps.md)

---