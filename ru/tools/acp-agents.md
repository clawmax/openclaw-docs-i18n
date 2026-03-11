

  Координация агентов

  
# ACP Агенты

Сессии [Протокола Агент-Клиент (ACP)](https://agentclientprotocol.com/) позволяют OpenClaw запускать внешние среды выполнения кода (например, Pi, Claude Code, Codex, OpenCode и Gemini CLI) через плагин бэкенда ACP. Если вы попросите OpenClaw простым языком «запустить это в Codex» или «запустить Claude Code в ветке», OpenClaw должен перенаправить этот запрос в среду выполнения ACP (а не в нативную среду выполнения суб-агентов).

## Быстрый рабочий процесс для оператора

Используйте это, когда вам нужен практический `/acp` runbook:

1.  Создать сессию:
    -   `/acp spawn codex --mode persistent --thread auto`
2.  Работайте в связанной ветке (или явно указывайте ключ этой сессии).
3.  Проверьте состояние среды выполнения:
    -   `/acp status`
4.  При необходимости настройте параметры среды выполнения:
    -   `/acp model <provider/model>`
    -   `/acp permissions `
    -   `/acp timeout `
5.  Подтолкните активную сессию без замены контекста:
    -   `/acp steer tighten logging and continue`
6.  Остановите работу:
    -   `/acp cancel` (остановить текущий ход), или
    -   `/acp close` (закрыть сессию + удалить привязки)

## Быстрый старт для пользователей

Примеры естественных запросов:

-   «Запустите постоянную сессию Codex в ветке здесь и держите её в фокусе».
-   «Запустите это как одноразовую сессию Claude Code ACP и суммируйте результат».
-   «Используйте Gemini CLI для этой задачи в ветке, а затем продолжайте обсуждение в той же ветке».

Что должен сделать OpenClaw:

1.  Выбрать `runtime: "acp"`.
2.  Определить запрошенную цель среды выполнения (`agentId`, например `codex`).
3.  Если запрошена привязка к ветке и текущий канал поддерживает это, привязать сессию ACP к ветке.
4.  Перенаправлять последующие сообщения в этой ветке в ту же сессию ACP, пока фокус не будет снят/сессия не закрыта/не истекла.

## ACP против суб-агентов

Используйте ACP, когда вам нужна внешняя среда выполнения. Используйте суб-агентов, когда вам нужны нативные делегированные запуски OpenClaw.

| Область | Сессия ACP | Запуск суб-агента |
| --- | --- | --- |
| Среда выполнения | Плагин бэкенда ACP (например, acpx) | Нативная среда выполнения суб-агентов OpenClaw |
| Ключ сессии | `agent::acp:` | `agent::subagent:` |
| Основные команды | `/acp ...` | `/subagents ...` |
| Инструмент создания | `sessions_spawn` с `runtime:"acp"` | `sessions_spawn` (среда выполнения по умолчанию) |

См. также [Суб-агенты](./subagents.md).

## Сессии, привязанные к веткам (независимо от канала)

Когда привязка к веткам включена для адаптера канала, сессии ACP могут быть привязаны к веткам:

-   OpenClaw привязывает ветку к целевой сессии ACP.
-   Последующие сообщения в этой ветке перенаправляются в привязанную сессию ACP.
-   Вывод ACP доставляется обратно в ту же ветку.
-   Снятие фокуса/закрытие/архивирование/истечение времени простоя или максимального срока действия удаляет привязку.

Поддержка привязки к веткам зависит от адаптера. Если активный адаптер канала не поддерживает привязку к веткам, OpenClaw возвращает понятное сообщение о неподдержке/недоступности. Необходимые флаги функций для ACP с привязкой к веткам:

-   `acp.enabled=true`
-   `acp.dispatch.enabled` включен по умолчанию (установите `false` для приостановки диспетчеризации ACP)
-   Флаг создания ACP в ветках для адаптера канала включен (зависит от адаптера)
    -   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

### Каналы, поддерживающие ветки

-   Любой адаптер канала, предоставляющий возможность привязки сессии/ветки.
-   Текущая встроенная поддержка:
    -   Ветки/каналы Discord
    -   Темы Telegram (темы форума в группах/супергруппах и темы в ЛС)
-   Плагины каналов могут добавить поддержку через тот же интерфейс привязки.

## Настройки для конкретных каналов

Для неэфемерных рабочих процессов настройте постоянные привязки ACP в записях верхнего уровня `bindings[]`.

### Модель привязки

-   `bindings[].type="acp"` отмечает постоянную привязку беседы ACP.
-   `bindings[].match` идентифицирует целевую беседу:
    -   Канал или ветка Discord: `match.channel="discord"` + `match.peer.id=""`
    -   Тема форума Telegram: `match.channel="telegram"` + `match.peer.id=":topic:"`
-   `bindings[].agentId` — это идентификатор агента OpenClaw, которому принадлежит привязка.
-   Дополнительные переопределения ACP находятся в `bindings[].acp`:
    -   `mode` (`persistent` или `oneshot`)
    -   `label`
    -   `cwd`
    -   `backend`

### Значения по умолчанию среды выполнения для каждого агента

Используйте `agents.list[].runtime` для определения значений по умолчанию ACP один раз для каждого агента:

-   `agents.list[].runtime.type="acp"`
-   `agents.list[].runtime.acp.agent` (идентификатор среды выполнения, например `codex` или `claude`)
-   `agents.list[].runtime.acp.backend`
-   `agents.list[].runtime.acp.mode`
-   `agents.list[].runtime.acp.cwd`

Приоритет переопределений для привязанных сессий ACP:

1.  `bindings[].acp.*`
2.  `agents.list[].runtime.acp.*`
3.  глобальные значения по умолчанию ACP (например, `acp.backend`)

Пример:

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
      {
        id: "claude",
        runtime: {
          type: "acp",
          acp: { agent: "claude", backend: "acpx", mode: "persistent" },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
    {
      type: "acp",
      agentId: "claude",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
      acp: { cwd: "/workspace/repo-b" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "discord", accountId: "default" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "telegram", accountId: "default" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": { requireMention: false },
          },
        },
      },
    },
    telegram: {
      groups: {
        "-1001234567890": {
          topics: { "42": { requireMention: false } },
        },
      },
    },
  },
}
```

Поведение:

-   OpenClaw гарантирует, что настроенная сессия ACP существует перед использованием.
-   Сообщения в этом канале или теме перенаправляются в настроенную сессию ACP.
-   В привязанных беседах `/new` и `/reset` сбрасывают тот же ключ сессии ACP на месте.
-   Временные привязки среды выполнения (например, созданные потоками фокусировки на ветке) по-прежнему применяются там, где они присутствуют.

## Запуск сессий ACP (интерфейсы)

### Из sessions_spawn

Используйте `runtime: "acp"` для запуска сессии ACP из хода агента или вызова инструмента.

```json
{
  "task": "Open the repo and summarize failing tests",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
```

Примечания:

-   `runtime` по умолчанию `subagent`, поэтому для сессий ACP явно укажите `runtime: "acp"`.
-   Если `agentId` опущен, OpenClaw использует `acp.defaultAgent`, когда он настроен.
-   `mode: "session"` требует `thread: true` для поддержания постоянной привязанной беседы.

Детали интерфейса:

-   `task` (обязательно): начальный промпт, отправляемый в сессию ACP.
-   `runtime` (обязательно для ACP): должно быть `"acp"`.
-   `agentId` (опционально): идентификатор целевой среды выполнения ACP. Возвращается к `acp.defaultAgent`, если он установлен.
-   `thread` (опционально, по умолчанию `false`): запросить поток привязки к ветке, где поддерживается.
-   `mode` (опционально): `run` (одноразовый) или `session` (постоянный).
    -   по умолчанию `run`
    -   если `thread: true` и режим опущен, OpenClaw может по умолчанию использовать постоянное поведение в зависимости от пути выполнения
    -   `mode: "session"` требует `thread: true`
-   `cwd` (опционально): запрошенный рабочий каталог среды выполнения (проверяется политикой бэкенда/среды выполнения).
-   `label` (опционально): метка, обращенная к оператору, используется в тексте сессии/баннера.
-   `streamTo` (опционально): `"parent"` передает начальные сводки прогресса выполнения ACP обратно в сессию инициатора как системные события.
    -   Когда доступно, принятые ответы включают `streamLogPath`, указывающий на журнал JSONL в рамках сессии (`.acp-stream.jsonl`), который можно отслеживать для полной истории ретрансляции.

## Совместимость с песочницей

Сессии ACP в настоящее время выполняются в среде выполнения хоста, а не внутри песочницы OpenClaw. Текущие ограничения:

-   Если сессия инициатора находится в песочнице, создание ACP блокируется.
    -   Ошибка: `Sandboxed sessions cannot spawn ACP sessions because runtime="acp" runs on the host. Use runtime="subagent" from sandboxed sessions.`
-   `sessions_spawn` с `runtime: "acp"` не поддерживает `sandbox: "require"`.
    -   Ошибка: `sessions_spawn sandbox="require" is unsupported for runtime="acp" because ACP sessions run outside the sandbox. Use runtime="subagent" or sandbox="inherit".`

Используйте `runtime: "subagent"`, когда вам нужно выполнение, обеспечиваемое песочницей.

### Из команды /acp

Используйте `/acp spawn` для явного контроля оператора из чата, когда это необходимо.

```bash
/acp spawn codex --mode persistent --thread auto
/acp spawn codex --mode oneshot --thread off
/acp spawn codex --thread here
```

Ключевые флаги:

-   `--mode persistent|oneshot`
-   `--thread auto|here|off`
-   `--cwd <absolute-path>`
-   `--label `

См. [Слэш-команды](./slash-commands.md).

## Разрешение цели сессии

Большинство действий `/acp` принимают необязательную цель сессии (`session-key`, `session-id` или `session-label`). Порядок разрешения:

1.  Явный аргумент цели (или `--session` для `/acp steer`)
    -   пробует ключ
    -   затем идентификатор сессии в формате UUID
    -   затем метку
2.  Текущая привязка к ветке (если эта беседа/ветка привязана к сессии ACP)
3.  Резервный вариант текущей сессии инициатора

Если цель не разрешается, OpenClaw возвращает понятную ошибку (`Unable to resolve session target: ...`).

## Режимы создания веток

`/acp spawn` поддерживает `--thread auto|here|off`.

| Режим | Поведение |
| --- | --- |
| `auto` | В активной ветке: привязать эту ветку. Вне ветки: создать/привязать дочернюю ветку, когда поддерживается. |
| `here` | Требовать текущую активную ветку; завершиться ошибкой, если её нет. |
| `off` | Без привязки. Сессия начинается непривязанной. |

Примечания:

-   На поверхностях без привязки к веткам поведение по умолчанию фактически `off`.
-   Создание с привязкой к ветке требует поддержки политики канала:
    -   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

## Управление ACP

Доступное семейство команд:

-   `/acp spawn`
-   `/acp cancel`
-   `/acp steer`
-   `/acp close`
-   `/acp status`
-   `/acp set-mode`
-   `/acp set`
-   `/acp cwd`
-   `/acp permissions`
-   `/acp timeout`
-   `/acp model`
-   `/acp reset-options`
-   `/acp sessions`
-   `/acp doctor`
-   `/acp install`

`/acp status` показывает эффективные параметры среды выполнения и, когда доступно, как идентификаторы сессии уровня среды выполнения, так и уровня бэкенда. Некоторые элементы управления зависят от возможностей бэкенда. Если бэкенд не поддерживает элемент управления, OpenClaw возвращает понятную ошибку неподдерживаемого управления.

## Поваренная книга команд ACP

| Команда | Что делает | Пример |
| --- | --- | --- |
| `/acp spawn` | Создать сессию ACP; опционально привязать к ветке. | `/acp spawn codex --mode persistent --thread auto --cwd /repo` |
| `/acp cancel` | Отменить текущий ход для целевой сессии. | `/acp cancel agent:codex:acp:` |
| `/acp steer` | Отправить инструкцию управления в выполняющуюся сессию. | `/acp steer --session support inbox prioritize failing tests` |
| `/acp close` | Закрыть сессию и отвязать цели веток. | `/acp close` |
| `/acp status` | Показать бэкенд, режим, состояние, параметры среды выполнения, возможности. | `/acp status` |
| `/acp set-mode` | Установить режим среды выполнения для целевой сессии. | `/acp set-mode plan` |
| `/acp set` | Общая запись параметра конфигурации среды выполнения. | `/acp set model openai/gpt-5.2` |
| `/acp cwd` | Установить переопределение рабочего каталога среды выполнения. | `/acp cwd /Users/user/Projects/repo` |
| `/acp permissions` | Установить профиль политики одобрения. | `/acp permissions strict` |
| `/acp timeout` | Установить таймаут среды выполнения (секунды). | `/acp timeout 120` |
| `/acp model` | Установить переопределение модели среды выполнения. | `/acp model anthropic/claude-opus-4-5` |
| `/acp reset-options` | Удалить переопределения параметров среды выполнения сессии. | `/acp reset-options` |
| `/acp sessions` | Список недавних сессий ACP из хранилища. | `/acp sessions` |
| `/acp doctor` | Проверка здоровья бэкенда, возможности, исправления. | `/acp doctor` |
| `/acp install` | Вывод детерминированных шагов установки и включения. | `/acp install` |

## Сопоставление параметров среды выполнения

`/acp` имеет удобные команды и общий сеттер. Эквивалентные операции:

-   `/acp model ` сопоставляется с ключом конфигурации среды выполнения `model`.
-   `/acp permissions ` сопоставляется с ключом конфигурации среды выполнения `approval_policy`.
-   `/acp timeout ` сопоставляется с ключом конфигурации среды выполнения `timeout`.
-   `/acp cwd ` напрямую обновляет переопределение рабочего каталога среды выполнения.
-   `/acp set  ` — это общий путь.
    -   Особый случай: `key=cwd` использует путь переопределения рабочего каталога.
-   `/acp reset-options` очищает все переопределения среды выполнения для целевой сессии.

## Поддержка сред выполнения acpx (текущая)

Текущие встроенные псевдонимы сред выполнения acpx:

-   `pi`
-   `claude`
-   `codex`
-   `opencode`
-   `gemini`
-   `kimi`

Когда OpenClaw использует бэкенд acpx, предпочитайте эти значения для `agentId`, если только ваша конфигурация acpx не определяет пользовательские псевдонимы агентов. Прямое использование CLI acpx также может нацеливаться на произвольные адаптеры через `--agent `, но этот сырой обходной путь является функцией CLI acpx (а не обычным путем `agentId` OpenClaw).

## Необходимая конфигурация

Базовые настройки ACP:

```json
{
  acp: {
    enabled: true,
    // Опционально. По умолчанию true; установите false для приостановки диспетчеризации ACP при сохранении управления /acp.
    dispatch: { enabled: true },
    backend: "acpx",
    defaultAgent: "codex",
    allowedAgents: ["pi", "claude", "codex", "opencode", "gemini", "kimi"],
    maxConcurrentSessions: 8,
    stream: {
      coalesceIdleMs: 300,
      maxChunkChars: 1200,
    },
    runtime: {
      ttlMinutes: 120,
    },
  },
}
```

Конфигурация привязки к веткам зависит от адаптера канала. Пример для Discord:

```json
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        spawnAcpSessions: true,
      },
    },
  },
}
```

Если создание ACP с привязкой к ветке не работает, сначала проверьте флаг функции адаптера:

-   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`

См. [Справочник по конфигурации](../gateway/configuration-reference.md).

## Настройка плагина для бэкенда acpx

Установите и включите плагин:

```bash
openclaw plugins install acpx
openclaw config set plugins.entries.acpx.enabled true
```

Локальная установка в рабочем пространстве во время разработки:

```bash
openclaw plugins install ./extensions/acpx
```

Затем проверьте здоровье бэкенда:

```bash
/acp doctor
```

### Конфигурация команды и версии acpx

По умолчанию плагин acpx (опубликованный как `@openclaw/acpx`) использует закрепленный локальный бинарный файл плагина:

1.  Команда по умолчанию: `extensions/acpx/node_modules/.bin/acpx`.
2.  Ожидаемая версия по умолчанию соответствует закреплению расширения.
3.  При запуске бэкенд ACP немедленно регистрируется как неготовый.
4.  Фоновая задача ensure проверяет `acpx --version`.
5.  Если локальный бинарный файл плагина отсутствует или не соответствует версии, выполняется: `npm install --omit=dev --no-save acpx@` и повторная проверка.

Вы можете переопределить команду/версию в конфигурации плагина:

```json
{
  "plugins": {
    "entries": {
      "acpx": {
        "enabled": true,
        "config": {
          "command": "../acpx/dist/cli.js",
          "expectedVersion": "any"
        }
      }
    }
  }
}
```

Примечания:

-   `command` принимает абсолютный путь, относительный путь или имя команды (`acpx`).
-   Относительные пути разрешаются от рабочего каталога OpenClaw.
-   `expectedVersion: "any"` отключает строгое соответствие версий.
-   Когда `command` указывает на пользовательский бинарный файл/путь, автоматическая установка в рамках плагина отключается.
-   Запуск OpenClaw остается неблокирующим, пока выполняется проверка здоровья бэкенда.

См. [Плагины](./plugin.md).

## Конфигурация разрешений

Сессии ACP выполняются неинтерактивно — нет TTY для одобрения или отклонения запросов разрешений на запись файлов и выполнение команд. Плагин acpx предоставляет два ключа конфигурации, которые контролируют обработку разрешений:

### permissionMode

Контролирует, какие операции агент среды выполнения может выполнять без запроса.

| Значение | Поведение |
| --- | --- |
| `approve-all` | Автоматически одобрять все записи в файлы и команды оболочки. |
| `approve-reads` | Автоматически одобрять только чтение; запись и выполнение требуют запросов. |
| `deny-all` | Запрещать все запросы разрешений. |

### nonInteractivePermissions

Контролирует, что происходит, когда должен быть показан запрос разрешения, но интерактивный TTY недоступен (что всегда имеет место для сессий ACP).

| Значение | Поведение |
| --- | --- |
| `fail` | Прервать сессию с `AcpRuntimeError`. **(по умолчанию)** |
| `deny` | Тихо запретить разрешение и продолжить (градуированная деградация). |

### Конфигурация

Установите через конфигурацию плагина:

```bash
openclaw config set plugins.entries.acpx.config.permissionMode approve-all
openclaw config set plugins.entries.acpx.config.nonInteractivePermissions fail
```

Перезапустите шлюз после изменения этих значений.

> **Важно:** OpenClaw в настоящее время по умолчанию использует `permissionMode=approve-reads` и `nonInteractivePermissions=fail`. В неинтерактивных сессиях ACP любая запись или выполнение, вызывающее запрос разрешения, может завершиться ошибкой `AcpRuntimeError: Permission prompt unavailable in non-interactive mode`. Если вам нужно ограничить разрешения, установите `nonInteractivePermissions` в `deny`, чтобы сессии деградировали плавно, а не аварийно завершались.

## Устранение неполадок

| Симптом | Вероятная причина | Исправление |
| --- | --- | --- |
| `ACP runtime backend is not configured` | Плагин бэкенда отсутствует или отключен. | Установите и включите плагин бэкенда, затем запустите `/acp doctor`. |
| `ACP is disabled by policy (acp.enabled=false)` | ACP глобально отключен. | Установите `acp.enabled=true`. |
| `ACP dispatch is disabled by policy (acp.dispatch.enabled=false)` | Диспетчеризация из обычных сообщений ветки отключена. | Установите `acp.dispatch.enabled=true`. |
| `ACP agent "" is not allowed by policy` | Агент не в списке разрешенных. | Используйте разрешенный `agentId` или обновите `acp.allowedAgents`. |
| `Unable to resolve session target: ...` | Неверный ключ/идентификатор/метка. | Запустите `/acp sessions`, скопируйте точный ключ/метку, повторите попытку. |
| `--thread here requires running /acp spawn inside an active ... thread` | `--thread here` использован вне контекста ветки. | Перейдите в целевую ветку или используйте `--thread auto`/`off`. |
| `Only <user-id> can rebind this thread.` | Другой пользователь владеет привязкой ветки. | Перепривяжите как владелец или используйте другую ветку. |
| `Thread bindings are unavailable for .` | Адаптер не имеет возможности привязки к веткам. | Используйте `--thread off` или перейдите на поддерживаемый адаптер/канал. |
| `Sandboxed sessions cannot spawn ACP sessions ...` | Среда выполнения ACP находится на стороне хоста; сессия инициатора в песочнице. | Используйте `runtime="subagent"` из сессий в песочнице или запустите создание ACP из сессии вне песочницы. |
| `sessions_spawn sandbox="require" is unsupported for runtime="acp" ...` | Запрошено `sandbox="require"` для среды выполнения ACP. | Используйте `runtime="subagent"` для обязательной песочницы или используйте ACP с `sandbox="inherit"` из сессии вне песочницы. |
| Отсутствуют метаданные ACP для привязанной сессии | Устаревшие/удаленные метаданные сессии ACP. | Пересоздайте с помощью `/acp spawn`, затем перепривяжите/сфокусируйте ветку. |
| `AcpRuntimeError: Permission prompt unavailable in non-interactive mode` | `permissionMode` блокирует запись/выполнение в неинтерактивной сессии ACP. | Установите `plugins.entries.acpx.config.permissionMode` в `approve-all` и перезапустите шлюз. См. [Конфигурация разрешений](#permission-configuration). |
| Сессия ACP завершается рано с малым выводом | Запросы разрешений заблокированы `permissionMode`/`nonInteractivePermissions`. | Проверьте логи шлюза на наличие `AcpRuntimeError`. Для полных разрешений установите `permissionMode=approve-all`; для градуированной деградации установите `nonInteractivePermissions=deny`. |
| Сессия ACP бесконечно простаивает после завершения работы | Процесс среды выполнения завершился, но сессия ACP не сообщила о завершении. | Мониторьте с помощью `ps aux \| grep acpx`; завершите зависшие процессы вручную. |

[Суб-агенты](./subagents.md)[Мульти-агентная песочница и инструменты](./multi-agent-sandbox-tools.md)