title: "Быстрый старт: установка OpenClaw с помощью Nix на macOS"
description: "Узнайте, как установить и запустить OpenClaw с помощью Nix и nix-openclaw. Получите полную настройку с сервисами launchd, плагинами и детерминированной конфигурацией."
keywords: ["openclaw nix", "nix-openclaw", "установка на macos", "модуль home manager", "детерминированная конфигурация", "сервис launchd", "настройка telegram бота", "режим nix"]
---

  Другие способы установки

  
# Nix

Рекомендуемый способ запуска OpenClaw с Nix — через **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — готовый модуль для Home Manager, включающий всё необходимое.

## Быстрый старт

Вставьте это в ваш ИИ-агент (Claude, Cursor и т.д.):

```
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, model provider API key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 Полное руководство: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)** Репозиторий nix-openclaw является основным источником информации по установке через Nix. Эта страница — лишь краткий обзор.

## Что вы получаете

-   Шлюз + приложение для macOS + инструменты (whisper, spotify, камеры) — всё зафиксировано
-   Сервис launchd, работающий после перезагрузок
-   Система плагинов с декларативной конфигурацией
-   Мгновенный откат: `home-manager switch --rollback`

* * *

## Поведение в режиме Nix

Когда установлена переменная `OPENCLAW_NIX_MODE=1` (автоматически с nix-openclaw): OpenClaw поддерживает **режим Nix**, который делает конфигурацию детерминированной и отключает автоматические установки. Включите его, экспортировав:

```
OPENCLAW_NIX_MODE=1
```

В macOS GUI-приложение не наследует переменные окружения из оболочки автоматически. Вы также можете включить режим Nix через defaults:

```bash
defaults write ai.openclaw.mac openclaw.nixMode -bool true
```

### Пути конфигурации и состояния

OpenClaw читает конфигурацию JSON5 из `OPENCLAW_CONFIG_PATH` и хранит изменяемые данные в `OPENCLAW_STATE_DIR`. При необходимости вы также можете установить `OPENCLAW_HOME`, чтобы контролировать базовый домашний каталог, используемый для разрешения внутренних путей.

-   `OPENCLAW_HOME` (приоритет по умолчанию: `HOME` / `USERPROFILE` / `os.homedir()`)
-   `OPENCLAW_STATE_DIR` (по умолчанию: `~/.openclaw`)
-   `OPENCLAW_CONFIG_PATH` (по умолчанию: `$OPENCLAW_STATE_DIR/openclaw.json`)

При работе под Nix явно задавайте эти переменные для расположений, управляемых Nix, чтобы состояние среды выполнения и конфигурация оставались вне неизменяемого хранилища.

### Поведение среды выполнения в режиме Nix

-   Автоматическая установка и процессы самоизменения отключены
-   Отсутствующие зависимости выводят сообщения об исправлении, специфичные для Nix
-   Интерфейс отображает баннер режима Nix только для чтения, когда он активен

## Примечание по упаковке (macOS)

Процесс упаковки для macOS ожидает стабильный шаблон Info.plist по пути:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) копирует этот шаблон в бандл приложения и заменяет динамические поля (идентификатор бандла, версия/сборка, Git SHA, ключи Sparkle). Это сохраняет plist детерминированным для упаковки SwiftPM и сборок Nix (которые не зависят от полного набора инструментов Xcode).

## Связанное

-   [nix-openclaw](https://github.com/openclaw/nix-openclaw) — полное руководство по настройке
-   [Мастер настройки](../start/wizard.md) — настройка через CLI (без Nix)
-   [Docker](./docker.md) — установка в контейнере

[Podman](./podman.md)[Ansible](./ansible.md)

---