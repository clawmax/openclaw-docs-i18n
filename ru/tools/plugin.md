title: "Руководство по плагинам навыков OpenClaw: установка, конфигурация и SDK"
description: "Узнайте, как устанавливать, настраивать и разрабатывать плагины для OpenClaw. Расширяйте функциональность с помощью официальных плагинов, HTTP-маршрутов и вспомогательных средств времени выполнения."
keywords: ["плагины openclaw", "плагины навыков", "установка плагинов", "sdk плагинов", "gateway rpc", "инструменты агента", "манифест плагина", "вспомогательные средства времени выполнения"]
---

  Навыки

  
# Плагины

## Быстрый старт (новичок в плагинах?)

Плагин — это просто **небольшой модуль кода**, который расширяет OpenClaw дополнительными функциями (командами, инструментами и Gateway RPC). В большинстве случаев вы будете использовать плагины, когда вам нужна функция, которая ещё не встроена в ядро OpenClaw (или вы хотите держать опциональные функции отдельно от основной установки). Быстрый путь:

1.  Посмотрите, что уже загружено:

```bash
openclaw plugins list
```

2.  Установите официальный плагин (пример: Voice Call):

```bash
openclaw plugins install @openclaw/voice-call
```

Спецификации Npm являются **только для реестра** (имя пакета + опционально **точная версия** или **дистрибутивный тег**). Спецификации Git/URL/файлов и диапазоны semver отклоняются. "Голые" спецификации и `@latest` остаются на стабильном треке. Если npm разрешает любой из них в предрелизную версию, OpenClaw останавливается и просит вас явно согласиться с помощью предрелизного тега, такого как `@beta`/`@rc`, или точной предрелизной версии.

3.  Перезапустите Gateway, затем настройте в `plugins.entries..config`.

См. [Voice Call](../plugins/voice-call.md) для конкретного примера плагина. Ищете сторонние списки? См. [Сообщество плагинов](../plugins/community.md).

## Доступные плагины (официальные)

-   Microsoft Teams доступен только как плагин с версии 2026.1.15; установите `@openclaw/msteams`, если используете Teams.
-   Память (Core) — встроенный плагин поиска в памяти (включён по умолчанию через `plugins.slots.memory`)
-   Память (LanceDB) — встроенный плагин долговременной памяти (авто-вспоминание/захват; установите `plugins.slots.memory = "memory-lancedb"`)
-   [Voice Call](../plugins/voice-call.md) — `@openclaw/voice-call`
-   [Zalo Personal](../plugins/zalouser.md) — `@openclaw/zalouser`
-   [Matrix](../channels/matrix.md) — `@openclaw/matrix`
-   [Nostr](../channels/nostr.md) — `@openclaw/nostr`
-   [Zalo](../channels/zalo.md) — `@openclaw/zalo`
-   [Microsoft Teams](../channels/msteams.md) — `@openclaw/msteams`
-   Google Antigravity OAuth (аутентификация провайдера) — встроен как `google-antigravity-auth` (отключён по умолчанию)
-   Gemini CLI OAuth (аутентификация провайдера) — встроен как `google-gemini-cli-auth` (отключён по умолчанию)
-   Qwen OAuth (аутентификация провайдера) — встроен как `qwen-portal-auth` (отключён по умолчанию)
-   Copilot Proxy (аутентификация провайдера) — локальный мост VS Code Copilot Proxy; отличается от встроенного входа через устройство `github-copilot` (встроен, отключён по умолчанию)

Плагины OpenClaw — это **модули TypeScript**, загружаемые во время выполнения через jiti. **Валидация конфигурации не выполняет код плагина**; вместо этого используется манифест плагина и JSON Schema. См. [Манифест плагина](../plugins/manifest.md). Плагины могут регистрировать:

-   Методы Gateway RPC
-   HTTP-маршруты Gateway
-   Инструменты агента
-   CLI-команды
-   Фоновые службы
-   Движки контекста
-   Опциональную валидацию конфигурации
-   **Навыки** (перечисляя каталоги `skills` в манифесте плагина)
-   **Команды автоответа** (выполняются без вызова AI-агента)

Плагины выполняются **внутри процесса** Gateway, поэтому относитесь к ним как к доверенному коду. Руководство по созданию инструментов: [Инструменты агента плагина](../plugins/agent-tools.md).

## Вспомогательные средства времени выполнения

Плагины могут получать доступ к выбранным основным вспомогательным средствам через `api.runtime`. Для телефонии TTS:

```typescript
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Примечания:

-   Использует основную конфигурацию `messages.tts` (OpenAI или ElevenLabs).
-   Возвращает PCM-буфер аудио + частоту дискретизации. Плагины должны ресемплировать/кодировать для провайдеров.
-   Edge TTS не поддерживается для телефонии.

Для STT/транскрипции плагины могут вызывать:

```typescript
const { text } = await api.runtime.stt.transcribeAudioFile({
  filePath: "/tmp/inbound-audio.ogg",
  cfg: api.config,
  // Опционально, когда MIME-тип не может быть надёжно определён:
  mime: "audio/ogg",
});
```

Примечания:

-   Использует основную конфигурацию аудио для понимания медиа (`tools.media.audio`) и порядок резервных провайдеров.
-   Возвращает `{ text: undefined }`, когда вывод транскрипции не произведён (например, пропущенный/неподдерживаемый ввод).

## HTTP-маршруты Gateway

Плагины могут предоставлять HTTP-эндпоинты с помощью `api.registerHttpRoute(...)`.

```
api.registerHttpRoute({
  path: "/acme/webhook",
  auth: "plugin",
  match: "exact",
  handler: async (_req, res) => {
    res.statusCode = 200;
    res.end("ok");
    return true;
  },
});
```

Поля маршрута:

-   `path`: путь маршрута под HTTP-сервером gateway.
-   `auth`: обязательное. Используйте `"gateway"` для требования обычной аутентификации gateway или `"plugin"` для аутентификации/верификации вебхука, управляемой плагином.
-   `match`: опционально. `"exact"` (по умолчанию) или `"prefix"`.
-   `replaceExisting`: опционально. Позволяет тому же плагину заменить свою существующую регистрацию маршрута.
-   `handler`: возвращает `true`, когда маршрут обработал запрос.

Примечания:

-   `api.registerHttpHandler(...)` устарела. Используйте `api.registerHttpRoute(...)`.
-   Маршруты плагина должны явно объявлять `auth`.
-   Конфликты точного `path + match` отклоняются, если не указано `replaceExisting: true`, и один плагин не может заменить маршрут другого плагина.
-   Перекрывающиеся маршруты с разными уровнями `auth` отклоняются. Держите цепочки перехода `exact`/`prefix` только на одном уровне аутентификации.

## Пути импорта SDK плагина

Используйте подпути SDK вместо монолитного импорта `openclaw/plugin-sdk` при создании плагинов:

-   `openclaw/plugin-sdk/core` для общих API плагинов, типов аутентификации провайдера и общих вспомогательных средств.
-   `openclaw/plugin-sdk/compat` для встроенного/внутреннего кода плагина, которому требуются более широкие общие вспомогательные средства времени выполнения, чем `core`.
-   `openclaw/plugin-sdk/telegram` для плагинов канала Telegram.
-   `openclaw/plugin-sdk/discord` для плагинов канала Discord.
-   `openclaw/plugin-sdk/slack` для плагинов канала Slack.
-   `openclaw/plugin-sdk/signal` для плагинов канала Signal.
-   `openclaw/plugin-sdk/imessage` для плагинов канала iMessage.
-   `openclaw/plugin-sdk/whatsapp` для плагинов канала WhatsApp.
-   `openclaw/plugin-sdk/line` для плагинов канала LINE.
-   `openclaw/plugin-sdk/msteams` для поверхности встроенного плагина Microsoft Teams.
-   Также доступны специфичные для расширений встроенные подпути: `openclaw/plugin-sdk/acpx`, `openclaw/plugin-sdk/bluebubbles`, `openclaw/plugin-sdk/copilot-proxy`, `openclaw/plugin-sdk/device-pair`, `openclaw/plugin-sdk/diagnostics-otel`, `openclaw/plugin-sdk/diffs`, `openclaw/plugin-sdk/feishu`, `openclaw/plugin-sdk/google-gemini-cli-auth`, `openclaw/plugin-sdk/googlechat`, `openclaw/plugin-sdk/irc`, `openclaw/plugin-sdk/llm-task`, `openclaw/plugin-sdk/lobster`, `openclaw/plugin-sdk/matrix`, `openclaw/plugin-sdk/mattermost`, `openclaw/plugin-sdk/memory-core`, `openclaw/plugin-sdk/memory-lancedb`, `openclaw/plugin-sdk/minimax-portal-auth`, `openclaw/plugin-sdk/nextcloud-talk`, `openclaw/plugin-sdk/nostr`, `openclaw/plugin-sdk/open-prose`, `openclaw/plugin-sdk/phone-control`, `openclaw/plugin-sdk/qwen-portal-auth`, `openclaw/plugin-sdk/synology-chat`, `openclaw/plugin-sdk/talk-voice`, `openclaw/plugin-sdk/test-utils`, `openclaw/plugin-sdk/thread-ownership`, `openclaw/plugin-sdk/tlon`, `openclaw/plugin-sdk/twitch`, `openclaw/plugin-sdk/voice-call`, `openclaw/plugin-sdk/zalo` и `openclaw/plugin-sdk/zalouser`.

Примечание о совместимости:

-   `openclaw/plugin-sdk` остаётся поддерживаемым для существующих внешних плагинов.
-   Новые и перенесённые встроенные плагины должны использовать специфичные для канала или расширения подпути; используйте `core` для общих поверхностей и `compat` только когда требуются более широкие общие вспомогательные средства.

## Только для чтения: инспекция канала

Если ваш плагин регистрирует канал, предпочтительно реализовать `plugin.config.inspectAccount(cfg, accountId)` вместе с `resolveAccount(...)`. Почему:

-   `resolveAccount(...)` — это путь времени выполнения. Ему разрешено предполагать, что учётные данные полностью материализованы, и он может быстро завершиться ошибкой, когда требуемые секреты отсутствуют.
-   Пути команд только для чтения, такие как `openclaw status`, `openclaw status --all`, `openclaw channels status`, `openclaw channels resolve` и потоки исправления доктора/конфигурации, не должны материализовывать учётные данные времени выполнения только для описания конфигурации.

Рекомендуемое поведение `inspectAccount(...)`:

-   Возвращайте только описательное состояние учётной записи.
-   Сохраняйте `enabled` и `configured`.
-   Включайте поля источника/статуса учётных данных, когда это уместно, например:
    -   `tokenSource`, `tokenStatus`
    -   `botTokenSource`, `botTokenStatus`
    -   `appTokenSource`, `appTokenStatus`
    -   `signingSecretSource`, `signingSecretStatus`
-   Вам не нужно возвращать сырые значения токенов только для отчёта о доступности только для чтения. Возврата `tokenStatus: "available"` (и соответствующего поля источника) достаточно для команд в стиле статуса.
-   Используйте `configured_unavailable`, когда учётные данные настроены через SecretRef, но недоступны в текущем пути команды.

Это позволяет командам только для чтения сообщать "настроено, но недоступно в этом пути команды" вместо сбоя или некорректного сообщения о том, что учётная запись не настроена. Примечание о производительности:

-   Обнаружение плагинов и метаданные манифеста используют короткие внутрипроцессные кеши для уменьшения всплесков работы при запуске/перезагрузке.
-   Установите `OPENCLAW_DISABLE_PLUGIN_DISCOVERY_CACHE=1` или `OPENCLAW_DISABLE_PLUGIN_MANIFEST_CACHE=1`, чтобы отключить эти кеши.
-   Настройте окна кеширования с помощью `OPENCLAW_PLUGIN_DISCOVERY_CACHE_MS` и `OPENCLAW_PLUGIN_MANIFEST_CACHE_MS`.

## Обнаружение и приоритет

OpenClaw сканирует в порядке:

1.  Пути конфигурации

-   `plugins.load.paths` (файл или каталог)

2.  Расширения рабочей области

-   `/.openclaw/extensions/*.ts`
-   `/.openclaw/extensions/*/index.ts`

3.  Глобальные расширения

-   `~/.openclaw/extensions/*.ts`
-   `~/.openclaw/extensions/*/index.ts`

4.  Встроенные расширения (поставляются с OpenClaw, в основном отключены по умолчанию)

-   `/extensions/*`

Большинство встроенных плагинов должны быть явно включены через `plugins.entries..enabled` или `openclaw plugins enable `. Исключения для встроенных плагинов, включённых по умолчанию:

-   `device-pair`
-   `phone-control`
-   `talk-voice`
-   активный плагин слота памяти (слот по умолчанию: `memory-core`)

Установленные плагины включены по умолчанию, но могут быть отключены таким же образом. Примечания по усилению безопасности:

-   Если `plugins.allow` пуст и обнаруживаются невстроенные плагины, OpenClaw выводит предупреждение при запуске с идентификаторами плагинов и источниками.
-   Кандидатные пути проверяются на безопасность перед допуском к обнаружению. OpenClaw блокирует кандидатов, когда:
    -   точка входа расширения разрешается вне корня плагина (включая обходы символьных ссылок/путей),
    -   корневой/исходный путь плагина доступен для записи всем,
    -   владение путём подозрительно для невстроенных плагинов (владелец POSIX не является ни текущим uid, ни root).
-   Загруженные невстроенные плагины без происхождения от установки/пути загрузки выводят предупреждение, чтобы вы могли зафиксировать доверие (`plugins.allow`) или отслеживание установки (`plugins.installs`).

Каждый плагин должен включать файл `openclaw.plugin.json` в своём корне. Если путь указывает на файл, корнем плагина является каталог файла и должен содержать манифест. Если несколько плагинов разрешаются в один и тот же идентификатор, первое совпадение в порядке выше побеждает, а копии с более низким приоритетом игнорируются.

### Пакеты расширений

Каталог плагина может включать `package.json` с `openclaw.extensions`:

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

Каждая запись становится плагином. Если пакет перечисляет несколько расширений, идентификатор плагина становится `name/`. Если ваш плагин импортирует npm-зависимости, установите их в этом каталоге, чтобы `node_modules` был доступен (`npm install` / `pnpm install`). Защитный механизм безопасности: каждая запись `openclaw.extensions` должна оставаться внутри каталога плагина после разрешения символьных ссылок. Записи, которые выходят за пределы каталога пакета, отклоняются. Примечание по безопасности: `openclaw plugins install` устанавливает зависимости плагина с помощью `npm install --ignore-scripts` (без скриптов жизненного цикла). Держите деревья зависимостей плагина "чистыми JS/TS" и избегайте пакетов, требующих сборки `postinstall`.

### Метаданные каталога каналов

Плагины каналов могут рекламировать метаданные онбординга через `openclaw.channel` и подсказки по установке через `openclaw.install`. Это сохраняет данные каталога ядра свободными от данных. Пример:

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (самостоятельно размещённый)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Самостоятельно размещённый чат через вебхук-ботов Nextcloud Talk.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw также может объединять **внешние каталоги каналов** (например, экспорт реестра MPM). Поместите JSON-файл в одно из:

-   `~/.openclaw/mpm/plugins.json`
-   `~/.openclaw/mpm/catalog.json`
-   `~/.openclaw/plugins/catalog.json`

Или укажите `OPENCLAW_PLUGIN_CATALOG_PATHS` (или `OPENCLAW_MPM_CATALOG_PATHS`) на один или несколько JSON-файлов (разделённых запятой/точкой с запятой/`PATH`). Каждый файл должен содержать `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`.

## Идентификаторы плагинов

Идентификаторы плагинов по умолчанию:

-   Пакеты расширений: `package.json` `name`
-   Отдельный файл: базовое имя файла (`~/.../voice-call.ts` → `voice-call`)

Если плагин экспортирует `id`, OpenClaw использует его, но предупреждает, когда он не совпадает с настроенным идентификатором.

## Конфигурация

```json
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

Поля:

-   `enabled`: главный переключатель (по умолчанию: true)
-   `allow`: белый список (опционально)
-   `deny`: чёрный список (опционально; чёрный список имеет приоритет)
-   `load.paths`: дополнительные файлы/каталоги плагинов
-   `slots`: эксклюзивные селекторы слотов, такие как `memory` и `contextEngine`
-   `entries.`: переключатели + конфигурация для каждого плагина

Изменения конфигурации **требуют перезапуска gateway**. Правила валидации (строгие):

-   Неизвестные идентификаторы плагинов в `entries`, `allow`, `deny` или `slots` — это **ошибки**.
-   Неизвестные ключи `channels.` — это **ошибки**, если только манифест плагина не объявляет идентификатор канала.
-   Конфигурация плагина проверяется с использованием JSON Schema, встроенной в `openclaw.plugin.json` (`configSchema`).
-   Если плагин отключён, его конфигурация сохраняется и выводится **предупреждение**.

## Слоты плагинов (эксклюзивные категории)

Некоторые категории плагинов являются **эксклюзивными** (только один активен одновременно). Используйте `plugins.slots`, чтобы выбрать, какой плагин владеет слотом:

```json
{
  plugins: {
    slots: {
      memory: "memory-core", // или "none" для отключения плагинов памяти
      contextEngine: "legacy", // или идентификатор плагина, например "lossless-claw"
    },
  },
}
```

Поддерживаемые эксклюзивные слоты:

-   `memory`: активный плагин памяти (`"none"` отключает плагины памяти)
-   `contextEngine`: активный плагин движка контекста (`"legacy"` — встроенный вариант по умолчанию)

Если несколько плагинов объявляют `kind: "memory"` или `kind: "context-engine"`, только выбранный плагин загружается для этого слота. Остальные отключаются с диагностикой.

### Плагины движка контекста

Плагины движка контекста управляют оркестрацией контекста сессии для приёма, сборки и уплотнения. Зарегистрируйте их из вашего плагина с помощью `api.registerContextEngine(id, factory)`, затем выберите активный движок с помощью `plugins.slots.contextEngine`. Используйте это, когда ваш плагин должен заменить или расширить конвейер контекста по умолчанию, а не просто добавить поиск в памяти или хуки.

## UI управления (схема + метки)

UI управления использует `config.schema` (JSON Schema + `uiHints`) для рендеринга лучших форм. OpenClaw дополняет `uiHints` во время выполнения на основе обнаруженных плагинов:

-   Добавляет метки для каждого плагина для `plugins.entries.` / `.enabled` / `.config`
-   Объединяет опциональные подсказки полей конфигурации, предоставляемые плагином, под: `plugins.entries..config.`

Если вы хотите, чтобы поля конфигурации вашего плагина показывали хорошие метки/заполнители (и отмечали секреты как чувствительные), предоставьте `uiHints` вместе с вашей JSON Schema в манифесте плагина. Пример:

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "API-ключ", "sensitive": true },
    "region": { "label": "Регион", "placeholder": "us-east-1" }
  }
}
```

## CLI

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # копирует локальный файл/каталог в ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # относительный путь ок
openclaw plugins install ./plugin.tgz           # установить из локального tarball
openclaw plugins install ./plugin.zip           # установить из локального zip
openclaw plugins install -l ./extensions/voice-call # ссылка (без копирования) для разработки
openclaw plugins install @openclaw/voice-call # установить из npm
openclaw plugins install @openclaw/voice-call --pin # сохранить точное разрешённое имя@версия
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` работает только для установок из npm, отслеживаемых под `plugins.installs`. Если сохранённые метаданные целостности изменяются между обновлениями, OpenClaw предупреждает и запрашивает подтверждение (используйте глобальный `--yes`, чтобы обойти запросы). Плагины также могут регистрировать свои собственные команды верхнего уровня (пример: `openclaw voicecall`).

## API плагина (обзор)

Плагины экспортируют либо:

-   Функцию: `(api) => { ... }`
-   Объект: `{ id, name, configSchema, register(api) { ... } }`

Плагины движка контекста также могут регистрировать менеджер контекста, управляемый временем выполнения:

```bash
export default function (api) {
  api.registerContextEngine("lossless-claw", () => ({
    info: { id: "lossless-claw", name: "Lossless Claw", ownsCompaction: true },
    async ingest() {
      return { ingested: true };
    },
    async assemble({ messages }) {
      return { messages, estimatedTokens: 0 };
    },
    async compact() {
      return { ok: true, compacted: false };
    },
  }));
}
```

Затем включите его в конфигурации:

```json
{
  plugins: {
    slots: {
      contextEngine: "lossless-claw",
    },
  },
}
```

## Хуки плагинов

Плагины могут регистрировать хуки во время выполнения. Это позволяет плагину включать событийную автоматизацию без отдельной установки пакета хуков.

### Пример

```bash
export default function register(api) {
  api.registerHook(
    "command:new",
    async () => {
      // Логика хука здесь.
    },
    {
      name: "my-plugin.command-new",
      description: "Выполняется при вызове /new",
    },
  );
}
```

Примечания:

-   Регистрируйте хуки явно через `api.registerHook(...)`.
-   Правила соответствия хуков всё ещё применяются (требования ОС/бинарников/окружения/конфигурации).
-   Хуки, управляемые плагином, отображаются в `openclaw hooks list` с `plugin:`.
-   Вы не можете включать/отключать хуки, управляемые плагином, через `openclaw hooks`; включайте/отключайте сам плагин.

### Хуки жизненного цикла агента (api.on)

Для типизированных хуков жизненного цикла времени выполнения используйте `api.on(...)`:

```bash
export default function register(api) {
  api.on(
    "before_prompt_build",
    (event, ctx) => {
      return {
        prependSystemContext: "Следуйте руководству по стилю компании.",
      };
    },
    { priority: 10 },
  );
}
```

Важные хуки для построения промпта:

-   `before_model_resolve`: выполняется перед загрузкой сессии (`messages` недоступны). Используйте это для детерминированного переопределения `modelOverride` или `providerOverride`.
-   `before_prompt_build`: выполняется после загрузки сессии (`messages` доступны). Используйте это для формирования ввода промпта.
-   `before_agent_start`: хук для обратной совместимости. Предпочитайте два явных хука выше.

Политика хуков, применяемая ядром:

-   Операторы могут отключать хуки модификации промпта для каждого плагина через `plugins.entries..hooks.allowPromptInjection: false`.
-   Когда отключено, OpenClaw блокирует `before_prompt_build` и игнорирует поля, изменяющие промпт, возвращённые из устаревшего `before_agent_start`, сохраняя при этом устаревшие `modelOverride` и `providerOverride`.

Поля результата `before_prompt_build`:

-   `prependContext`: добавляет текст к пользовательскому промпту для этого запуска. Лучше всего для контента, специфичного для хода или динамического.
-   `systemPrompt`: полное переопределение системного промпта.
-   `prependSystemContext`: добавляет текст к текущему системному промпту.
-   `appendSystemContext`: добавляет текст к текущему системному промпту.

Порядок построения промпта во встроенной среде выполнения:

1.  Применить `prependContext` к пользовательскому промпту.
2.  Применить переопределение `systemPrompt`, если предоставлено.
3.  Применить `prependSystemContext + текущий системный промпт + appendSystemContext`.

Примечания по слиянию и приоритету:

-   Обработчики хуков выполняются по приоритету (высший первый).
-   Для объединённых полей контекста значения конкатенируются в порядке выполнения.
-   Значения `before_prompt_build` применяются перед значениями отката устаревшего `before_agent_start`.

Рекомендации по миграции:

-   Переместите статические инструкции из `prependContext` в `prependSystemContext` (или `appendSystemContext`), чтобы провайдеры могли кэшировать стабильный контент префикса системы.
-   Сохраняйте `prependContext` для динамического контекста на каждый ход, который должен оставаться привязанным к пользовательскому сообщению.

## Плагины провайдеров (аутентификация модели)

Плагины могут регистрировать **потоки аутентификации провайдера модели**, чтобы пользователи могли выполнять OAuth или настройку API-ключа внутри OpenClaw (без внешних скриптов). Зарегистрируйте провайдера через `api.registerProvider(...)`. Каждый провайдер предоставляет один или несколько методов аутентификации (OAuth, API-ключ, код устройства и т.д.). Эти методы обеспечивают:

-   `openclaw models auth login --provider  [--method ]`

Пример:

```
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // Запустите поток OAuth и верните профили аутентификации.
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

Примечания:

-   `run` получает `ProviderAuthContext` с помощниками `prompter`, `runtime`, `openUrl` и `oauth.createVpsAwareHandlers`.
-   Возвращайте `configPatch`, когда вам нужно добавить модели по умолчанию или конфигурацию провайдера.
-   Возвращайте `defaultModel`, чтобы `--set-default` мог обновить настройки агента по умолчанию.

### Регистрация канала обмена сообщениями

Плагины могут регистрировать **плагины каналов**, которые ведут себя как встроенные каналы (WhatsApp, Telegram и т.д.). Конфигурация канала находится под `channels.` и проверяется кодом вашего плагина канала.

```typescript
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "демонстрационный плагин канала.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["