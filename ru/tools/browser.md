

  Браузер

  
# Браузер (управляемый OpenClaw)

OpenClaw может запускать **выделенный профиль Chrome/Brave/Edge/Chromium**, которым управляет агент. Он изолирован от вашего личного браузера и управляется через небольшую локальную службу управления внутри Шлюза (только loopback). Упрощенный взгляд:

-   Представьте его как **отдельный браузер только для агента**.
-   Профиль `openclaw` **не затрагивает** ваш личный профиль браузера.
-   Агент может **открывать вкладки, читать страницы, кликать и печатать** в безопасной среде.
-   Профиль по умолчанию `chrome` использует **системный браузер Chromium по умолчанию** через ретранслятор расширения; переключитесь на `openclaw` для изолированного управляемого браузера.

## Что вы получаете

-   Отдельный профиль браузера с именем **openclaw** (по умолчанию с оранжевым акцентом).
-   Детерминированное управление вкладками (список/открытие/фокус/закрытие).
-   Действия агента (клик/печать/перетаскивание/выбор), снапшоты, скриншоты, PDF.
-   Поддержка нескольких профилей по желанию (`openclaw`, `work`, `remote`, …).

Этот браузер **не** для повседневного использования. Это безопасная, изолированная поверхность для автоматизации и проверки агента.

## Быстрый старт

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Если вы видите «Browser disabled», включите его в конфигурации (см. ниже) и перезапустите Шлюз.

## Профили: openclaw vs chrome

-   `openclaw`: управляемый, изолированный браузер (расширение не требуется).
-   `chrome`: ретранслятор расширения к вашему **системному браузеру** (требует, чтобы расширение OpenClaw было подключено к вкладке).

Установите `browser.defaultProfile: "openclaw"`, если хотите использовать управляемый режим по умолчанию.

## Конфигурация

Настройки браузера находятся в `~/.openclaw/openclaw.json`.

```json
{
  browser: {
    enabled: true, // по умолчанию: true
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: true, // режим доверенной сети по умолчанию
      // allowPrivateNetwork: true, // устаревший алиас для совместимости
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    // cdpUrl: "http://127.0.0.1:18792", // устаревшее переопределение для одного профиля
    remoteCdpTimeoutMs: 1500, // таймаут HTTP для удаленного CDP (мс)
    remoteCdpHandshakeTimeoutMs: 3000, // таймаут рукопожатия WebSocket для удаленного CDP (мс)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

Примечания:

-   Служба управления браузером привязывается к loopback на порту, производном от `gateway.port` (по умолчанию: `18791`, что равно gateway + 2). Ретранслятор использует следующий порт (`18792`).
-   Если вы переопределите порт Шлюза (`gateway.port` или `OPENCLAW_GATEWAY_PORT`), производные порты браузера сдвинутся, чтобы остаться в той же «семействе».
-   `cdpUrl` по умолчанию равен порту ретранслятора, если не задан.
-   `remoteCdpTimeoutMs` применяется к проверкам доступности удаленного (не loopback) CDP.
-   `remoteCdpHandshakeTimeoutMs` применяется к проверкам доступности WebSocket удаленного CDP.
-   Навигация браузера/открытие вкладки защищены от SSRF перед навигацией и повторно проверяются по возможности на финальном `http(s)` URL после навигации.
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork` по умолчанию `true` (модель доверенной сети). Установите `false` для строгого просмотра только публичных сайтов.
-   `browser.ssrfPolicy.allowPrivateNetwork` остается поддерживаемым как устаревший алиас для совместимости.
-   `attachOnly: true` означает «никогда не запускать локальный браузер; подключаться только если он уже запущен».
-   `color` + `color` для каждого профиля окрашивают UI браузера, чтобы вы могли видеть, какой профиль активен.
-   Профиль по умолчанию — `openclaw` (автономный браузер, управляемый OpenClaw). Используйте `defaultProfile: "chrome"`, чтобы выбрать ретранслятор расширения Chrome.
-   Порядок автоопределения: системный браузер по умолчанию, если на основе Chromium; иначе Chrome → Brave → Edge → Chromium → Chrome Canary.
-   Локальным профилям `openclaw` автоматически назначаются `cdpPort`/`cdpUrl` — задавайте их только для удаленного CDP.

## Использование Brave (или другого браузера на основе Chromium)

Если ваш **системный браузер по умолчанию** на основе Chromium (Chrome/Brave/Edge и т.д.), OpenClaw использует его автоматически. Установите `browser.executablePath`, чтобы переопределить автоопределение: пример CLI:

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

## Локальное vs удаленное управление

-   **Локальное управление (по умолчанию):** Шлюз запускает службу управления loopback и может запускать локальный браузер.
-   **Удаленное управление (хост узла):** запустите хост узла на машине, где есть браузер; Шлюз проксирует действия браузера к нему.
-   **Удаленный CDP:** установите `browser.profiles..cdpUrl` (или `browser.cdpUrl`), чтобы подключиться к удаленному браузеру на основе Chromium. В этом случае OpenClaw не будет запускать локальный браузер.

URL удаленного CDP могут включать аутентификацию:

-   Токены в запросе (например, `https://provider.example?token=`)
-   HTTP Basic auth (например, `https://user:pass@provider.example`)

OpenClaw сохраняет аутентификацию при вызовах конечных точек `/json/*` и при подключении к WebSocket CDP. Предпочитайте переменные окружения или менеджеры секретов для токенов вместо их записи в конфигурационные файлы.

## Прокси браузера узла (нулевая конфигурация по умолчанию)

Если вы запускаете **хост узла** на машине, где находится ваш браузер, OpenClaw может автоматически направлять вызовы инструментов браузера на этот узел без дополнительной конфигурации браузера. Это путь по умолчанию для удаленных шлюзов. Примечания:

-   Хост узла предоставляет свой локальный сервер управления браузером через **команду прокси**.
-   Профили берутся из собственной конфигурации `browser.profiles` узла (такой же, как локальная).
-   Отключите, если не хотите:
    -   На узле: `nodeHost.browserProxy.enabled=false`
    -   На шлюзе: `gateway.nodes.browser.mode="off"`

## Browserless (размещенный удаленный CDP)

[Browserless](https://browserless.io) — это размещенный сервис Chromium, который предоставляет конечные точки CDP через HTTPS. Вы можете направить профиль браузера OpenClaw на конечную точку региона Browserless и аутентифицироваться с помощью вашего API-ключа. Пример:

```json
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00",
      },
    },
  },
}
```

Примечания:

-   Замените `<BROWSERLESS_API_KEY>` на ваш реальный токен Browserless.
-   Выберите конечную точку региона, соответствующую вашей учетной записи Browserless (см. их документацию).

## Безопасность

Ключевые идеи:

-   Управление браузером только через loopback; доступ проходит через аутентификацию Шлюза или сопряжение узлов.
-   Если управление браузером включено и аутентификация не настроена, OpenClaw автоматически генерирует `gateway.auth.token` при запуске и сохраняет его в конфигурации.
-   Держите Шлюз и любые хосты узлов в частной сети (Tailscale); избегайте публичного доступа.
-   Относитесь к URL/токенам удаленного CDP как к секретам; предпочитайте переменные окружения или менеджер секретов.

Советы по удаленному CDP:

-   Предпочитайте HTTPS конечные точки и краткосрочные токены, где это возможно.
-   Избегайте встраивания долгоживущих токенов прямо в конфигурационные файлы.

## Профили (мультибраузер)

OpenClaw поддерживает несколько именованных профилей (конфигураций маршрутизации). Профили могут быть:

-   **Управляемые openclaw**: выделенный экземпляр браузера на основе Chromium со своей собственной директорией пользовательских данных + портом CDP
-   **Удаленные**: явный URL CDP (браузер на основе Chromium, работающий в другом месте)
-   **Ретранслятор расширения**: ваши существующие вкладки Chrome через локальный ретранслятор + расширение Chrome

По умолчанию:

-   Профиль `openclaw` создается автоматически, если отсутствует.
-   Профиль `chrome` встроен для ретранслятора расширения Chrome (по умолчанию указывает на `http://127.0.0.1:18792`).
-   Локальные порты CDP выделяются из диапазона **18800–18899** по умолчанию.
-   Удаление профиля перемещает его локальную директорию данных в Корзину.

Все конечные точки управления принимают `?profile=`; CLI использует `--browser-profile`.

## Ретранслятор расширения Chrome (используйте ваш существующий Chrome)

OpenClaw также может управлять **вашими существующими вкладками Chrome** (без отдельного экземпляра Chrome «openclaw») через локальный ретранслятор CDP + расширение Chrome. Полное руководство: [Расширение Chrome](./chrome-extension.md) Поток:

-   Шлюз работает локально (на той же машине) или хост узла работает на машине с браузером.
-   Локальный **сервер ретранслятора** слушает на loopback `cdpUrl` (по умолчанию: `http://127.0.0.1:18792`).
-   Вы нажимаете значок расширения **OpenClaw Browser Relay** на вкладке, чтобы подключить ее (оно не подключается автоматически).
-   Агент управляет этой вкладкой через обычный инструмент `browser`, выбирая нужный профиль.

Если Шлюз работает в другом месте, запустите хост узла на машине с браузером, чтобы Шлюз мог проксировать действия браузера.

### Изолированные сессии

Если сессия агента изолирована, инструмент `browser` может по умолчанию использовать `target="sandbox"` (браузер песочницы). Перехват управления через ретранслятор расширения Chrome требует контроля над хост-браузером, поэтому либо:

-   запустите сессию без изоляции, либо
-   установите `agents.defaults.sandbox.browser.allowHostControl: true` и используйте `target="host"` при вызове инструмента.

### Настройка

1.  Загрузите расширение (dev/unpacked):

```bash
openclaw browser extension install
```

-   Chrome → `chrome://extensions` → включите «Режим разработчика»
-   «Загрузить распакованное» → выберите директорию, выведенную командой `openclaw browser extension path`
-   Закрепите расширение, затем нажмите на него на вкладке, которой хотите управлять (значок показывает `ON`).

2.  Используйте его:

-   CLI: `openclaw browser --browser-profile chrome tabs`
-   Инструмент агента: `browser` с `profile="chrome"`

Опционально: если хотите другое имя или порт ретранслятора, создайте свой собственный профиль:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

Примечания:

-   Этот режим полагается на Playwright-on-CDP для большинства операций (скриншоты/снапшоты/действия).
-   Отключитесь, снова нажав на значок расширения.

## Гарантии изоляции

-   **Выделенная директория пользовательских данных**: никогда не затрагивает ваш личный профиль браузера.
-   **Выделенные порты**: избегает `9222`, чтобы предотвратить конфликты с рабочими процессами разработки.
-   **Детерминированное управление вкладками**: целевые вкладки по `targetId`, а не «последняя вкладка».

## Выбор браузера

При локальном запуске OpenClaw выбирает первый доступный:

1.  Chrome
2.  Brave
3.  Edge
4.  Chromium
5.  Chrome Canary

Вы можете переопределить с помощью `browser.executablePath`. Платформы:

-   macOS: проверяет `/Applications` и `~/Applications`.
-   Linux: ищет `google-chrome`, `brave`, `microsoft-edge`, `chromium` и т.д.
-   Windows: проверяет обычные места установки.

## API управления (опционально)

Только для локальных интеграций, Шлюз предоставляет небольшой HTTP API loopback:

-   Статус/старт/стоп: `GET /`, `POST /start`, `POST /stop`
-   Вкладки: `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
-   Снапшот/скриншот: `GET /snapshot`, `POST /screenshot`
-   Действия: `POST /navigate`, `POST /act`
-   Хуки: `POST /hooks/file-chooser`, `POST /hooks/dialog`
-   Загрузки: `POST /download`, `POST /wait/download`
-   Отладка: `GET /console`, `POST /pdf`
-   Отладка: `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
-   Сеть: `POST /response/body`
-   Состояние: `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
-   Состояние: `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
-   Настройки: `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

Все конечные точки принимают `?profile=`. Если настроена аутентификация шлюза, HTTP-маршруты браузера также требуют аутентификации:

-   `Authorization: Bearer `
-   `x-openclaw-password: ` или HTTP Basic auth с этим паролем

### Требование Playwright

Некоторые функции (навигация/действие/AI снапшот/ролевой снапшот, скриншоты элементов, PDF) требуют Playwright. Если Playwright не установлен, эти конечные точки возвращают четкую ошибку 501. ARIA снапшоты и базовые скриншоты все еще работают для управляемого Chrome openclaw. Для драйвера ретранслятора расширения Chrome ARIA снапшоты и скриншоты требуют Playwright. Если вы видите `Playwright is not available in this gateway build`, установите полный пакет Playwright (не `playwright-core`) и перезапустите шлюз, или переустановите OpenClaw с поддержкой браузера.

#### Установка Playwright в Docker

Если ваш Шлюз работает в Docker, избегайте `npx playwright` (конфликты переопределения npm). Используйте встроенный CLI:

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

Чтобы сохранить загруженные браузеры, установите `PLAYWRIGHT_BROWSERS_PATH` (например, `/home/node/.cache/ms-playwright`) и убедитесь, что `/home/node` сохраняется через `OPENCLAW_HOME_VOLUME` или bind mount. См. [Docker](../install/docker.md).

## Как это работает (внутренне)

Высокоуровневый поток:

-   Небольшой **сервер управления** принимает HTTP-запросы.
-   Он подключается к браузерам на основе Chromium (Chrome/Brave/Edge/Chromium) через **CDP**.
-   Для расширенных действий (клик/печать/снапшот/PDF) он использует **Playwright** поверх CDP.
-   Когда Playwright отсутствует, доступны только операции без Playwright.

Эта конструкция сохраняет агента на стабильном, детерминированном интерфейсе, позволяя вам менять локальные/удаленные браузеры и профили.

## Быстрая справка по CLI

Все команды принимают `--browser-profile ` для указания конкретного профиля. Все команды также принимают `--json` для машиночитаемого вывода (стабильные полезные данные). Основы:

-   `openclaw browser status`
-   `openclaw browser start`
-   `openclaw browser stop`
-   `openclaw browser tabs`
-   `openclaw browser tab`
-   `openclaw browser tab new`
-   `openclaw browser tab select 2`
-   `openclaw browser tab close 2`
-   `openclaw browser open https://example.com`
-   `openclaw browser focus abcd1234`
-   `openclaw browser close abcd1234`

Инспекция:

-   `openclaw browser screenshot`
-   `openclaw browser screenshot --full-page`
-   `openclaw browser screenshot --ref 12`
-   `openclaw browser screenshot --ref e12`
-   `openclaw browser snapshot`
-   `openclaw browser snapshot --format aria --limit 200`
-   `openclaw browser snapshot --interactive --compact --depth 6`
-   `openclaw browser snapshot --efficient`
-   `openclaw browser snapshot --labels`
-   `openclaw browser snapshot --selector "#main" --interactive`
-   `openclaw browser snapshot --frame "iframe#main" --interactive`
-   `openclaw browser console --level error`
-   `openclaw browser errors --clear`
-   `openclaw browser requests --filter api --clear`
-   `openclaw browser pdf`
-   `openclaw browser responsebody "**/api" --max-chars 5000`

Действия:

-   `openclaw browser navigate https://example.com`
-   `openclaw browser resize 1280 720`
-   `openclaw browser click 12 --double`
-   `openclaw browser click e12 --double`
-   `openclaw browser type 23 "hello" --submit`
-   `openclaw browser press Enter`
-   `openclaw browser hover 44`
-   `openclaw browser scrollintoview e12`
-   `openclaw browser drag 10 11`
-   `openclaw browser select 9 OptionA OptionB`
-   `openclaw browser download e12 report.pdf`
-   `openclaw browser waitfordownload report.pdf`
-   `openclaw browser upload /tmp/openclaw/uploads/file.pdf`
-   `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
-   `openclaw browser dialog --accept`
-   `openclaw browser wait --text "Done"`
-   `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
-   `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
-   `openclaw browser highlight e12`
-   `openclaw browser trace start`
-   `openclaw browser trace stop`

Состояние:

-   `openclaw browser cookies`
-   `openclaw browser cookies set session abc123 --url "https://example.com"`
-   `openclaw browser cookies clear`
-   `openclaw browser storage local get`
-   `openclaw browser storage local set theme dark`
-   `openclaw browser storage session clear`
-   `openclaw browser set offline on`
-   `openclaw browser set headers --headers-json '{"X-Debug":"1"}'`
-   `openclaw browser set credentials user pass`
-   `openclaw browser set credentials --clear`
-   `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
-   `openclaw browser set geo --clear`
-   `openclaw browser set media dark`
-   `openclaw browser set timezone America/New_York`
-   `openclaw browser set locale en-US`
-   `openclaw browser set device "iPhone 14"`

Примечания:

-   `upload` и `dialog` — это **взводящие** вызовы; выполняйте их перед кликом/нажатием, которое вызывает выбор файла/диалог.
-   Пути вывода загрузок и трассировок ограничены корневыми временными директориями OpenClaw:
    -   трассировки: `/tmp/openclaw` (резерв: `${os.tmpdir()}/openclaw`)
    -   загрузки: `/tmp/openclaw/downloads` (резерв: `${os.tmpdir()}/openclaw/downloads`)
-   Пути загрузки ограничены корневой временной директорией загрузок OpenClaw:
    -   загрузки: `/tmp/openclaw/uploads` (резерв: `${os.tmpdir()}/openclaw/uploads`)
-   `upload` также может напрямую устанавливать файловые поля через `--input-ref` или `--element`.
-   `snapshot`:
    -   `--format ai` (по умолчанию при установленном Playwright): возвращает AI снапшот с числовыми ссылками (`aria-ref=""`).
    -   `--format aria`: возвращает дерево доступности (без ссылок; только для инспекции).
    -   `--efficient` (или `--mode efficient`): предустановка компактного ролевого снапшота (interactive + compact + depth + lower maxChars).
    -   Конфигурация по умолчанию (только для инструмента/CLI): установите `browser.snapshotDefaults.mode: "efficient"`, чтобы использовать эффективные снапшоты, когда вызывающая сторона не передает режим (см. [Конфигурация Шлюза](../gateway/configuration.md#browser-openclaw-managed-browser)).
    -   Опции ролевого снапшота (`--interactive`, `--compact`, `--depth`, `--selector`) принудительно создают ролевой снапшот со ссылками вида `ref=e12`.
    -   `--frame ""` ограничивает ролевые снапшоты iframe'ом (используется с ролевыми ссылками вида `e12`).
    -   `--interactive` выводит плоский, удобный для выбора список интерактивных элементов (лучше всего для выполнения действий).
    -   `--labels` добавляет скриншот только области просмотра с наложенными метками ссылок (выводит `MEDIA:`).
-   `click`/`type`/и т.д. требуют `ref` из `snapshot` (либо числовой `12`, либо ролевая ссылка `e12`). CSS селекторы намеренно не поддерживаются для действий.

## Снапшоты и ссылки

OpenClaw поддерживает два стиля «снапшота»:

-   **AI снапшот (числовые ссылки)**: `openclaw browser snapshot` (по умолчанию; `--format ai`)
    -   Вывод: текстовый снапшот, включающий числовые ссылки.
    -   Действия: `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
    -   Внутренне ссылка разрешается через `aria-ref` Playwright.
-   **Ролевой снапшот (ролевые ссылки вида `e12`)**: `openclaw browser snapshot --interactive` (или `--compact`, `--depth`, `--selector`, `--frame`)
    -   Вывод: ролевой список/дерево с `[ref=e12]` (и опционально `[nth=1]`).
    -   Действия: `openclaw browser click e12`, `openclaw browser highlight e12`.
    -   Внутренне ссылка разрешается через `getByRole(...)` (плюс `nth()` для дубликатов).
    -   Добавьте `--labels`, чтобы включить скриншот области просмотра с наложенными метками `e12`.

Поведение ссылок:

-   Ссылки **не стабильны между навигациями**; если что-то не работает, перезапустите `snapshot` и используйте свежую ссылку.
-   Если ролевой снапшот был сделан с `--frame`, ролевые ссылки ограничены этим iframe'ом до следующего ролевого снапшота.

## Усиления ожидания

Вы можете ожидать большего, чем просто время/текст:

-   Ожидание URL (поддерживаются шаблоны Playwright):
    -   `openclaw browser wait --url "**/dash"`
-   Ожидание состояния загрузки:
    -   `openclaw browser wait --load networkidle`
-   Ожидание JS-предиката:
    -   `openclaw browser wait --fn "window.ready===true"`
-   Ожидание, пока селектор станет видимым:
    -   `openclaw browser wait "#main"`

Их можно комбинировать:

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

## Рабочие процессы отладки

Когда действие не выполняется (например, «not visible», «strict mode violation», «covered»):

1.  `openclaw browser snapshot --interactive`
2.  Используйте `click ` / `type ` (предпочитайте ролевые ссылки в интерактивном режиме)
3.  Если все еще не работает: `openclaw browser highlight `, чтобы увидеть, на что нацеливается Playwright
4.  Если страница ведет себя странно:
    -   `openclaw browser errors --clear`
    -   `openclaw browser requests --filter api --clear`
5.  Для глубокой отладки: запишите трассировку:
    -   `openclaw browser trace start`
    -   воспроизведите проблему
    -   `openclaw browser trace stop` (выводит `TRACE:`)

## JSON вывод

`--json` предназначен для скриптов и структурированных инструментов. Примеры:

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

Ролевые снапшоты в JSON включают `refs` плюс небольшой блок `stats` (строки/символы/ссылки/интерактивные), чтобы инструменты могли оценивать размер и плотность полезной нагрузки.

## Состояние и регуляторы окружения

Они полезны для рабочих процессов «заставьте сайт вести себя как X»:

-   Куки: `cookies`, `cookies set`, `cookies clear`
-   Хранилище: `storage local|session get|set|clear`
-   Офлайн: `set offline on|off`
-   Заголовки: `set headers --headers-json '{"X-Debug":"1"}'` (устаревший `set headers --json '{"X-Debug":"1"}'` остается поддерживаемым)
-   HTTP basic auth: `set credentials user pass` (или `--clear`)
-   Геолокация: `set geo   --origin "https://example.com"` (или `--clear`)
-   Медиа: `set media dark|light|no-preference|none`
-   Часовой пояс / локаль: `set timezone ...`, `set locale ...`
-   Устройство / область просмотра:
    -   `set device "iPhone 14"` (предустановки устройств Playwright)
    -   `set viewport 1280 720`

## Безопасность и конфиденциальность

-   Профиль браузера openclaw может содержать сессии входа; относитесь к нему как к конфиденциальному.
-   `browser act kind=evaluate` / `openclaw browser evaluate` и `wait --fn` выполняют произвольный JavaScript в контексте страницы. Инъекция в промпт может направлять это. Отключите это с помощью `browser.evaluateEnabled=false`, если не нужно.
-   Для заметок о входах и защите от ботов (X/Twitter и т.д.) см. [Вход в браузер + публикация в X/Twitter](./browser-login.md).
-   Держите Шлюз/хост узла приватными (только loopback или tailnet).
-   Конечные точки удаленного CDP мощные; туннелируйте и защищайте их.

Пример строгого режима (блокировать частные/внутренние назначения по умолчанию):

```json
{
  browser: {
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: false,
      hostnameAllowlist: ["*.example.com", "example.com"],
      allowedHostnames: ["localhost"], // опциональное точное разрешение
    },
  },
}
```

## Устранение неполадок

Для проблем, специфичных для Linux (особенно snap Chromium), см. [Устранение неполадок браузера](./browser-linux-troubleshooting.md).

## Инструменты агента + как работает управление

Агент получает **оди