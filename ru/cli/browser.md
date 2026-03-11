

  CLI команды

  
# browser

Управляйте сервером управления браузером OpenClaw и выполняйте действия в браузере (вкладки, снимки, скриншоты, навигация, клики, ввод текста). Связанное:

-   Инструмент Browser + API: [Инструмент Browser](../tools/browser.md)
-   Chrome extension relay: [Chrome расширение](../tools/chrome-extension.md)

## Общие флаги

-   `--url `: WebSocket URL шлюза (по умолчанию из конфигурации).
-   `--token `: Токен шлюза (если требуется).
-   `--timeout `: таймаут запроса (мс).
-   `--browser-profile `: выбрать профиль браузера (по умолчанию из конфигурации).
-   `--json`: машинно-читаемый вывод (где поддерживается).

## Быстрый старт (локально)

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## Профили

Профили — это именованные конфигурации маршрутизации браузера. На практике:

-   `openclaw`: запускает/подключается к выделенному экземпляру Chrome, управляемому OpenClaw (изолированная директория данных пользователя).
-   `chrome`: управляет вашими существующими вкладками Chrome через Chrome extension relay.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Использование конкретного профиля:

```bash
openclaw browser --browser-profile work tabs
```

## Вкладки

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## Снимок / скриншот / действия

Снимок:

```bash
openclaw browser snapshot
```

Скриншот:

```bash
openclaw browser screenshot
```

Навигация/клик/ввод текста (UI автоматизация на основе ссылок):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## Chrome extension relay (подключение через кнопку на панели инструментов)

Этот режим позволяет агенту управлять существующей вкладкой Chrome, которую вы подключаете вручную (автоматическое подключение не выполняется). Установите распакованное расширение в стабильный путь:

```bash
openclaw browser extension install
openclaw browser extension path
```

Затем в Chrome → `chrome://extensions` → включите «Режим разработчика» → «Загрузить распакованное расширение» → выберите указанную папку. Полное руководство: [Chrome расширение](../tools/chrome-extension.md)

## Удаленное управление браузером (прокси узла хоста)

Если Шлюз работает на другой машине, чем браузер, запустите **узел хоста** на машине, где установлен Chrome/Brave/Edge/Chromium. Шлюз будет проксировать действия браузера на этот узел (отдельный сервер управления браузером не требуется). Используйте `gateway.nodes.browser.mode` для управления авто-маршрутизацией и `gateway.nodes.browser.node` для привязки к конкретному узлу, если подключено несколько. Безопасность и настройка удаленного доступа: [Инструмент Browser](../tools/browser.md), [Удаленный доступ](../gateway/remote.md), [Tailscale](../gateway/tailscale.md), [Безопасность](../gateway/security.md)

[approvals](./approvals.md)[channels](./channels.md)