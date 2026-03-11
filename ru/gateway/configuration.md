

  Конфигурация и операции

  
# Конфигурация

OpenClaw читает необязательную конфигурацию **JSON5** из файла `~/.openclaw/openclaw.json`. Если файл отсутствует, OpenClaw использует безопасные значения по умолчанию. Распространённые причины добавить конфигурацию:

-   Подключить каналы и контролировать, кто может писать боту
-   Задать модели, инструменты, изоляцию или автоматизацию (cron, хуки)
-   Настроить сессии, медиа, сеть или интерфейс

См. [полный справочник](./configuration-reference.md) для всех доступных полей.

> **💡** **Новичок в настройке?** Начните с `openclaw onboard` для интерактивной настройки или ознакомьтесь с руководством [Примеры конфигураций](./configuration-examples.md) для готовых конфигураций, которые можно скопировать и вставить.

## Минимальная конфигурация

```
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Редактирование конфигурации

```bash
openclaw onboard       # полный мастер настройки
openclaw configure     # мастер конфигурации
```

```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```

Откройте [http://127.0.0.1:18789](http://127.0.0.1:18789) и используйте вкладку **Config**. Панель управления отображает форму на основе схемы конфигурации, с редактором **Raw JSON** в качестве запасного варианта.

Редактируйте файл `~/.openclaw/openclaw.json` напрямую. Шлюз отслеживает файл и автоматически применяет изменения (см. [горячую перезагрузку](#config-hot-reload)).

## Строгая валидация

> **⚠️** OpenClaw принимает только конфигурации, полностью соответствующие схеме. Неизвестные ключи, неправильные типы или недопустимые значения приводят к **отказу шлюза запускаться**. Единственное исключение на корневом уровне — `$schema` (строка), чтобы редакторы могли прикреплять метаданные JSON Schema.

 При сбое валидации:

-   Шлюз не запускается
-   Работают только диагностические команды (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
-   Запустите `openclaw doctor`, чтобы увидеть точные проблемы
-   Запустите `openclaw doctor --fix` (или `--yes`), чтобы применить исправления

## Распространённые задачи

Каждый канал имеет свой собственный раздел конфигурации под `channels.<провайдер>`. См. страницу конкретного канала для шагов настройки:

-   [WhatsApp](../channels/whatsapp.md) — `channels.whatsapp`
-   [Telegram](../channels/telegram.md) — `channels.telegram`
-   [Discord](../channels/discord.md) — `channels.discord`
-   [Slack](../channels/slack.md) — `channels.slack`
-   [Signal](../channels/signal.md) — `channels.signal`
-   [iMessage](../channels/imessage.md) — `channels.imessage`
-   [Google Chat](../channels/googlechat.md) — `channels.googlechat`
-   [Mattermost](../channels/mattermost.md) — `channels.mattermost`
-   [MS Teams](../channels/msteams.md) — `channels.msteams`

Все каналы используют одинаковый шаблон политики личных сообщений:

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",   // pairing | allowlist | open | disabled
      allowFrom: ["tg:123"], // только для allowlist/open
    },
  },
}
```

Задайте основную модель и опциональные резервные:

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5.2"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "openai/gpt-5.2": { alias: "GPT" },
      },
    },
  },
}
```

-   `agents.defaults.models` определяет каталог моделей и служит списком разрешённых для `/model`.
-   Ссылки на модели используют формат `провайдер/модель` (например, `anthropic/claude-opus-4-6`).
-   `agents.defaults.imageMaxDimensionPx` управляет уменьшением изображений в транскриптах/инструментах (по умолчанию `1200`); меньшие значения обычно снижают использование токенов для зрения при работе с большим количеством скриншотов.
-   См. [CLI для моделей](../concepts/models.md) для переключения моделей в чате и [Резервирование моделей](../concepts/model-failover.md) для ротации аутентификации и поведения резервных моделей.
-   Для пользовательских/самостоятельно размещённых провайдеров см. [Пользовательские провайдеры](./configuration-reference.md#custom-providers-and-base-urls) в справочнике.

Доступ к личным сообщениям контролируется для каждого канала через `dmPolicy`:

-   `"pairing"` (по умолчанию): неизвестные отправители получают одноразовый код для подтверждения
-   `"allowlist"`: только отправители из `allowFrom` (или сохранённого списка разрешённых)
-   `"open"`: разрешить все входящие личные сообщения (требует `allowFrom: ["*"]`)
-   `"disabled"`: игнорировать все личные сообщения

Для групп используйте `groupPolicy` + `groupAllowFrom` или специфичные для канала списки разрешённых. См. [полный справочник](./configuration-reference.md#dm-and-group-access) для деталей по каждому каналу.

По умолчанию групповые сообщения **требуют упоминания**. Настройте шаблоны для каждого агента:

```json
{
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw"],
        },
      },
    ],
  },
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

-   **Упоминания через метаданные**: нативные @-упоминания (тап-упоминание в WhatsApp, @bot в Telegram и т.д.)
-   **Текстовые шаблоны**: regex-шаблоны в `mentionPatterns`
-   См. [полный справочник](./configuration-reference.md#group-chat-mention-gating) для переопределений на уровне канала и режима self-chat.

Сессии управляют непрерывностью и изоляцией разговоров:

```json
{
  session: {
    dmScope: "per-channel-peer",  // рекомендуется для многопользовательского режима
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
  },
}
```

-   `dmScope`: `main` (общая) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
-   `threadBindings`: глобальные значения по умолчанию для маршрутизации сессий, привязанных к тредам (Discord поддерживает `/focus`, `/unfocus`, `/agents`, `/session idle` и `/session max-age`).
-   См. [Управление сессиями](../concepts/session.md) для областей видимости, связей идентификаторов и политики отправки.
-   См. [полный справочник](./configuration-reference.md#session) для всех полей.

Запускайте сессии агентов в изолированных контейнерах Docker:

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",  // off | non-main | all
        scope: "agent",    // session | agent | shared
      },
    },
  },
}
```

Сначала соберите образ: `scripts/sandbox-setup.sh`См. [Изоляция](./sandboxing.md) для полного руководства и [полный справочник](./configuration-reference.md#sandbox) для всех опций.

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
  },
}
```

-   `every`: строка длительности (`30m`, `2h`). Установите `0m` для отключения.
-   `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
-   `directPolicy`: `allow` (по умолчанию) или `block` для целей heartbeat в стиле личных сообщений
-   См. [Heartbeat](./heartbeat.md) для полного руководства.

```json
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h",
    runLog: {
      maxBytes: "2mb",
      keepLines: 2000,
    },
  },
}
```

-   `sessionRetention`: удаляет завершённые изолированные сессии запусков из `sessions.json` (по умолчанию `24h`; установите `false` для отключения).
-   `runLog`: удаляет `cron/runs/.jsonl` по размеру и количеству сохраняемых строк.
-   См. [Cron-задачи](../automation/cron-jobs.md) для обзора функций и примеров CLI.

Включите HTTP-эндпоинты вебхуков на шлюзе:

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "main",
        deliver: true,
      },
    ],
  },
}
```

Примечание по безопасности:

-   Считайте всё содержимое полезной нагрузки хука/вебхука ненадёжными входными данными.
-   Держите флаги обхода небезопасного контента отключёнными (`hooks.gmail.allowUnsafeExternalContent`, `hooks.mappings[].allowUnsafeExternalContent`), если не выполняете строго ограниченную отладку.
-   Для агентов, управляемых хуками, предпочитайте сильные современные модели и строгую политику инструментов (например, только обмен сообщениями плюс изоляция, где это возможно).

См. [полный справочник](./configuration-reference.md#hooks) для всех опций маппинга и интеграции с Gmail.

Запускайте несколько изолированных агентов с отдельными рабочими областями и сессиями:

```json
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

См. [Мульти-агент](../concepts/multi-agent.md) и [полный справочник](./configuration-reference.md#multi-agent-routing) для правил привязки и профилей доступа для каждого агента.

Используйте `$include` для организации больших конфигураций:

```
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/a.json5", "./clients/b.json5"],
  },
}
```

-   **Один файл**: заменяет содержащий его объект
-   **Массив файлов**: глубоко объединяются по порядку (поздний побеждает)
-   **Соседние ключи**: объединяются после включений (переопределяют включённые значения)
-   **Вложенные включения**: поддерживаются до 10 уровней в глубину
-   **Относительные пути**: разрешаются относительно включающего файла
-   **Обработка ошибок**: понятные ошибки для отсутствующих файлов, ошибок парсинга и циклических включений

## Горячая перезагрузка конфигурации

Шлюз отслеживает файл `~/.openclaw/openclaw.json` и автоматически применяет изменения — для большинства настроек ручной перезапуск не требуется.

### Режимы перезагрузки

| Режим | Поведение |
| --- | --- |
| **`hybrid`** (по умолчанию) | Мгновенно применяет безопасные изменения. Автоматически перезапускается для критических. |
| **`hot`** | Применяет только безопасные изменения. Логирует предупреждение, когда требуется перезапуск — вы обрабатываете его вручную. |
| **`restart`** | Перезапускает шлюз при любом изменении конфигурации, безопасном или нет. |
| **`off`** | Отключает отслеживание файлов. Изменения вступают в силу при следующем ручном перезапуске. |

```json
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Что применяется "на горячую", а что требует перезапуска

Большинство полей применяются "на горячую" без простоя. В режиме `hybrid` изменения, требующие перезапуска, обрабатываются автоматически.

| Категория | Поля | Требуется перезапуск? |
| --- | --- | --- |
| Каналы | `channels.*`, `web` (WhatsApp) — все встроенные и расширенные каналы | Нет |
| Агенты и модели | `agent`, `agents`, `models`, `routing` | Нет |
| Автоматизация | `hooks`, `cron`, `agent.heartbeat` | Нет |
| Сессии и сообщения | `session`, `messages` | Нет |
| Инструменты и медиа | `tools`, `browser`, `skills`, `audio`, `talk` | Нет |
| UI и прочее | `ui`, `logging`, `identity`, `bindings` | Нет |
| Сервер шлюза | `gateway.*` (порт, привязка, аутентификация, tailscale, TLS, HTTP) | **Да** |
| Инфраструктура | `discovery`, `canvasHost`, `plugins` | **Да** |

> **ℹ️** `gateway.reload` и `gateway.remote` являются исключениями — их изменение **не** вызывает перезапуск.

## Конфигурация через RPC (программные обновления)

> **ℹ️** RPC-запросы на запись в плоскость управления (`config.apply`, `config.patch`, `update.run`) ограничены частотой **3 запроса в 60 секунд** на `deviceId+clientIp`. При ограничении RPC возвращает `UNAVAILABLE` с `retryAfterMs`.

 

Валидирует + записывает полную конфигурацию и перезапускает шлюз за один шаг.

`config.apply` заменяет **всю конфигурацию**. Используйте `config.patch` для частичных обновлений или `openclaw config set` для отдельных ключей.

Параметры:

-   `raw` (строка) — полезная нагрузка JSON5 для всей конфигурации
-   `baseHash` (опционально) — хэш конфигурации из `config.get` (обязателен, если конфигурация существует)
-   `sessionKey` (опционально) — ключ сессии для пинга пробуждения после перезапуска
-   `note` (опционально) — заметка для сентинеля перезапуска
-   `restartDelayMs` (опционально) — задержка перед перезапуском (по умолчанию 2000)

Запросы на перезапуск объединяются, пока один уже находится в ожидании/выполнении, и применяется 30-секундный период охлаждения между циклами перезапуска.

```bash
openclaw gateway call config.get --params '{}'  # получить payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
  "baseHash": "<hash>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123"
}'
```

Объединяет частичное обновление с существующей конфигурацией (семантика JSON merge patch):

-   Объекты объединяются рекурсивно
-   `null` удаляет ключ
-   Массивы заменяются

Параметры:

-   `raw` (строка) — JSON5 только с ключами для изменения
-   `baseHash` (обязательно) — хэш конфигурации из `config.get`
-   `sessionKey`, `note`, `restartDelayMs` — как в `config.apply`

Поведение при перезапуске совпадает с `config.apply`: объединённые ожидающие перезапуски плюс 30-секундный период охлаждения между циклами перезапуска.

```bash
openclaw gateway call config.patch --params '{
  "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
  "baseHash": "<hash>"
}'
```

## Переменные окружения

OpenClaw читает переменные окружения из родительского процесса плюс:

-   `.env` из текущей рабочей директории (если присутствует)
-   `~/.openclaw/.env` (глобальный запасной вариант)

Ни один из файлов не переопределяет существующие переменные окружения. Вы также можете задать встроенные переменные окружения в конфигурации:

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

Если включено и ожидаемые ключи не установлены, OpenClaw запускает вашу логин-оболочку и импортирует только отсутствующие ключи:

```json
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Эквивалент переменной окружения: `OPENCLAW_LOAD_SHELL_ENV=1`

 

Ссылайтесь на переменные окружения в любом строковом значении конфигурации с помощью `${VAR_NAME}`:

```json
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Правила:

-   Сопоставляются только имена в верхнем регистре: `[A-Z_][A-Z0-9_]*`
-   Отсутствующие/пустые переменные вызывают ошибку при загрузке
-   Экранируйте с помощью `$${VAR}` для буквального вывода
-   Работает внутри файлов `$include`
-   Встроенная подстановка: `"${BASE}/v1"` → `"https://api.example.com/v1"`

 

Для полей, поддерживающих объекты SecretRef, вы можете использовать:

```json
{
  models: {
    providers: {
      openai: { apiKey: { source: "env", provider: "default", id: "OPENAI_API_KEY" } },
    },
  },
  skills: {
    entries: {
      "nano-banana-pro": {
        apiKey: {
          source: "file",
          provider: "filemain",
          id: "/skills/entries/nano-banana-pro/apiKey",
        },
      },
    },
  },
  channels: {
    googlechat: {
      serviceAccountRef: {
        source: "exec",
        provider: "vault",
        id: "channels/googlechat/serviceAccount",
      },
    },
  },
}
```

Детали SecretRef (включая `secrets.providers` для `env`/`file`/`exec`) находятся в [Управление секретами](./secrets.md). Поддерживаемые пути для учётных данных перечислены в [Поверхность учётных данных SecretRef](../reference/secretref-credential-surface.md).

 См. [Окружение](../help/environment.md) для полного порядка приоритета и источников.

## Полный справочник

Для полного пошагового справочника см. **[Справочник по конфигурации](./configuration-reference.md)**.

* * *

*Связанные темы: [Примеры конфигураций](./configuration-examples.md) · [Справочник по конфигурации](./configuration-reference.md) · [Doctor](./doctor.md)*

[Ранбук шлюза](../gateway.md)[Справочник по конфигурации](./configuration-reference.md)