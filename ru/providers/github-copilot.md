title: "Как использовать GitHub Copilot в качестве провайдера моделей в OpenClaw"
description: "Узнайте два способа интеграции GitHub Copilot с OpenClaw: встроенный провайдер и плагин Copilot Proxy. Включает команды настройки и конфигурации."
keywords: ["github copilot", "openclaw", "ai помощник для программирования", "провайдер моделей", "copilot proxy", "настройка cli", "авторизация", "gpt-4o"]
---

  Провайдеры

  
# GitHub Copilot

## Что такое GitHub Copilot?

GitHub Copilot — это AI-помощник для программирования от GitHub. Он предоставляет доступ к моделям Copilot в соответствии с вашей учётной записью и тарифным планом GitHub. OpenClaw может использовать Copilot в качестве провайдера моделей двумя разными способами.

## Два способа использования Copilot в OpenClaw

### 1) Встроенный провайдер GitHub Copilot (github-copilot)

Используйте нативный процесс входа через устройство для получения токена GitHub, который затем обменивается на токены API Copilot при запуске OpenClaw. Это **стандартный** и самый простой путь, так как он не требует VS Code.

### 2) Плагин Copilot Proxy (copilot-proxy)

Используйте расширение **Copilot Proxy** для VS Code в качестве локального моста. OpenClaw обращается к конечной точке `/v1` прокси и использует список моделей, который вы там настроите. Выберите этот способ, если вы уже используете Copilot Proxy в VS Code или вам нужно маршрутизировать запросы через него. Вы должны включить плагин и держать расширение VS Code запущенным. Используйте GitHub Copilot как провайдера моделей (`github-copilot`). Команда входа запускает процесс авторизации GitHub через устройство, сохраняет профиль аутентификации и обновляет вашу конфигурацию для использования этого профиля.

## Настройка через CLI

```bash
openclaw models auth login-github-copilot
```

Вам будет предложено перейти по URL и ввести одноразовый код. Не закрывайте терминал, пока процесс не завершится.

### Необязательные флаги

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```

## Установка модели по умолчанию

```bash
openclaw models set github-copilot/gpt-4o
```

### Пример конфигурации

```json
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```

## Примечания

-   Требуется интерактивный TTY; запускайте команду напрямую в терминале.
-   Доступность моделей Copilot зависит от вашего тарифного плана; если модель отклонена, попробуйте другой идентификатор (например, `github-copilot/gpt-4.1`).
-   Процесс входа сохраняет токен GitHub в хранилище профилей аутентификации и обменивает его на токен API Copilot при запуске OpenClaw.

[Deepgram](./deepgram.md)[Hugging Face (Inference)](./huggingface.md)