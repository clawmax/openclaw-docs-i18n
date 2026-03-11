

  Обзор платформ

  
# Платформы

Ядро OpenClaw написано на TypeScript. **Node — рекомендуемая среда выполнения**. Bun не рекомендуется для Gateway (ошибки в WhatsApp/Telegram). Существуют приложения-компаньоны для macOS (приложение в строке меню) и мобильных узлов (iOS/Android). Приложения-компаньоны для Windows и Linux запланированы, но Gateway полностью поддерживается уже сегодня. Нативные приложения-компаньоны для Windows также запланированы; для Gateway рекомендуется использовать WSL2.

## Выберите вашу ОС

-   macOS: [macOS](./platforms/macos.md)
-   iOS: [iOS](./platforms/ios.md)
-   Android: [Android](./platforms/android.md)
-   Windows: [Windows](./platforms/windows.md)
-   Linux: [Linux](./platforms/linux.md)

## VPS и хостинг

-   VPS хаб: [VPS хостинг](./vps.md)
-   Fly.io: [Fly.io](./install/fly.md)
-   Hetzner (Docker): [Hetzner](./install/hetzner.md)
-   GCP (Compute Engine): [GCP](./install/gcp.md)
-   exe.dev (VM + HTTPS прокси): [exe.dev](./install/exe-dev.md)

## Общие ссылки

-   Руководство по установке: [Начало работы](./start/getting-started.md)
-   Руководство по Gateway: [Gateway](./gateway.md)
-   Конфигурация Gateway: [Конфигурация](./gateway/configuration.md)
-   Статус службы: `openclaw gateway status`

## Установка службы Gateway (CLI)

Используйте один из этих способов (все поддерживаются):

-   Мастер (рекомендуется): `openclaw onboard --install-daemon`
-   Прямая установка: `openclaw gateway install`
-   Через конфигурацию: `openclaw configure` → выберите **Gateway service**
-   Восстановление/миграция: `openclaw doctor` (предлагает установить или исправить службу)

Целевая служба зависит от ОС:

-   macOS: LaunchAgent (`ai.openclaw.gateway` или `ai.openclaw.`; устаревшие `com.openclaw.*`)
-   Linux/WSL2: systemd user service (`openclaw-gateway[-].service`)

[Приложение для macOS](./platforms/macos.md)

---