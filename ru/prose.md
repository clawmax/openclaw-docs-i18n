

  Расширения

  
# OpenProse

OpenProse — это портируемый формат для воркфлоу, ориентированный на markdown, для оркестрации AI-сессий. В OpenClaw он поставляется в виде плагина, который устанавливает набор навыков OpenProse и слэш-команду `/prose`. Программы живут в файлах `.prose` и могут порождать несколько под-агентов с явным управлением потоком. Официальный сайт: [https://www.prose.md](https://www.prose.md)

## Что он умеет

-   Исследования и синтез с несколькими агентами с явным параллелизмом.
-   Повторяемые воркфлоу с защитой через аппрувы (ревью кода, триаж инцидентов, контент-пайплайны).
-   Переиспользуемые программы `.prose`, которые можно запускать в поддерживаемых средах выполнения агентов.

## Установка и включение

Встроенные плагины по умолчанию отключены. Включите OpenProse:

```bash
openclaw plugins enable open-prose
```

Перезапустите Gateway после включения плагина. Для локальной разработки/установки: `openclaw plugins install ./extensions/open-prose` Связанная документация: [Плагины](./tools/plugin.md), [Манифест плагина](./plugins/manifest.md), [Навыки](./tools/skills.md).

## Слэш-команда

OpenProse регистрирует `/prose` как команду навыка, которую может вызывать пользователь. Она направляет инструкции в виртуальную машину OpenProse и использует инструменты OpenClaw под капотом. Распространённые команды:

```bash
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## Пример: простой файл .prose

```bash
# Исследование и синтез с двумя агентами, работающими параллельно.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## Расположение файлов

OpenProse хранит состояние в директории `.prose/` вашего рабочего пространства:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

Постоянные агенты на уровне пользователя находятся по пути:

```
~/.prose/agents/
```

## Режимы состояния

OpenProse поддерживает несколько бэкендов для состояния:

-   **filesystem** (по умолчанию): `.prose/runs/...`
-   **in-context**: транзитное, для небольших программ
-   **sqlite** (экспериментальный): требует наличия бинарника `sqlite3`
-   **postgres** (экспериментальный): требует `psql` и строку подключения

Примечания:

-   sqlite/postgres являются опциональными и экспериментальными.
-   Учётные данные postgres попадают в логи под-агентов; используйте выделенную БД с минимальными привилегиями.

## Удалённые программы

`/prose run <handle/slug>` разрешается в `https://p.prose.md//`. Прямые URL-адреса загружаются как есть. Для этого используется инструмент `web_fetch` (или `exec` для POST).

## Соответствие среде выполнения OpenClaw

Программы OpenProse соответствуют примитивам OpenClaw:

| Концепция OpenProse | Инструмент OpenClaw |
| --- | --- |
| Создание сессии / Task tool | `sessions_spawn` |
| Чтение/запись файлов | `read` / `write` |
| Веб-запрос | `web_fetch` |

Если ваш список разрешённых инструментов блокирует эти инструменты, программы OpenProse завершатся с ошибкой. См. [Конфигурация навыков](./tools/skills-config.md).

## Безопасность и аппрувы

Относитесь к файлам `.prose` как к коду. Проверяйте их перед запуском. Используйте списки разрешённых инструментов OpenClaw и шлюзы аппрувов для контроля побочных эффектов. Для детерминированных воркфлоу с аппрувами сравните с [Lobster](./tools/lobster.md).

[Инструменты агента плагина](./plugins/agent-tools.md)[Хуки](./automation/hooks.md)