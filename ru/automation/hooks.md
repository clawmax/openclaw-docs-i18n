

  Автоматизация

  
# Хуки

Хуки предоставляют расширяемую событийно-ориентированную систему для автоматизации действий в ответ на команды агента и события. Хуки автоматически обнаруживаются из директорий и могут управляться через CLI-команды, аналогично тому, как работают навыки в OpenClaw.

## Общее представление

Хуки — это небольшие скрипты, которые запускаются при возникновении какого-либо события. Существует два вида:

-   **Хуки** (эта страница): запускаются внутри Gateway при срабатывании событий агента, таких как `/new`, `/reset`, `/stop` или события жизненного цикла.
-   **Вебхуки**: внешние HTTP-вебхуки, которые позволяют другим системам запускать работу в OpenClaw. См. [Вебхуки](./webhook.md) или используйте `openclaw webhooks` для вспомогательных команд Gmail.

Хуки также могут быть встроены в плагины; см. [Плагины](../tools/plugin.md#plugin-hooks). Типичные варианты использования:

-   Сохранение снимка памяти при сбросе сессии
-   Ведение журнала аудита команд для отладки или соответствия требованиям
-   Запуск последующей автоматизации при начале или завершении сессии
-   Запись файлов в рабочее пространство агента или вызов внешних API при срабатывании событий

Если вы можете написать небольшую функцию на TypeScript, вы можете написать хук. Хуки обнаруживаются автоматически, и вы включаете или отключаете их через CLI.

## Обзор

Система хуков позволяет вам:

-   Сохранять контекст сессии в память при выполнении команды `/new`
-   Логировать все команды для аудита
-   Запускать пользовательскую автоматизацию при событиях жизненного цикла агента
-   Расширять поведение OpenClaw без изменения основного кода

## Начало работы

### Встроенные хуки

OpenClaw поставляется с четырьмя встроенными хуками, которые обнаруживаются автоматически:

-   **💾 session-memory**: Сохраняет контекст сессии в ваше рабочее пространство агента (по умолчанию `~/.openclaw/workspace/memory/`) при выполнении команды `/new`
-   **📎 bootstrap-extra-files**: Внедряет дополнительные файлы начальной загрузки рабочего пространства из настроенных шаблонов glob/path во время `agent:bootstrap`
-   **📝 command-logger**: Логирует все события команд в `~/.openclaw/logs/commands.log`
-   **🚀 boot-md**: Запускает `BOOT.md` при старте шлюза (требует включения внутренних хуков)

Список доступных хуков:

```bash
openclaw hooks list
```

Включить хук:

```bash
openclaw hooks enable session-memory
```

Проверить статус хука:

```bash
openclaw hooks check
```

Получить подробную информацию:

```bash
openclaw hooks info session-memory
```

### Онбординг

Во время онбординга (`openclaw onboard`) вам будет предложено включить рекомендуемые хуки. Мастер автоматически обнаруживает подходящие хуки и предлагает их для выбора.

## Обнаружение хуков

Хуки автоматически обнаруживаются из трех директорий (в порядке приоритета):

1.  **Хуки рабочего пространства**: `/hooks/` (на агента, наивысший приоритет)
2.  **Управляемые хуки**: `~/.openclaw/hooks/` (установленные пользователем, общие для всех рабочих пространств)
3.  **Встроенные хуки**: `/dist/hooks/bundled/` (поставляются с OpenClaw)

Директории управляемых хуков могут быть либо **одиночным хуком**, либо **пакетом хуков** (директория пакета). Каждый хук — это директория, содержащая:

```
my-hook/
├── HOOK.md          # Метаданные + документация
└── handler.ts       # Реализация обработчика
```

## Пакеты хуков (npm/архивы)

Пакеты хуков — это стандартные npm-пакеты, которые экспортируют один или несколько хуков через `openclaw.hooks` в `package.json`. Установите их с помощью:

```bash
openclaw hooks install <path-or-spec>
```

Спецификации npm предназначены только для реестра (имя пакета + необязательная точная версия или dist-тег). Git/URL/файловые спецификации и диапазоны semver отклоняются. Простые спецификации и `@latest` остаются на стабильном треке. Если npm разрешает любую из них в пререлиз, OpenClaw останавливается и просит вас явно согласиться с помощью пререлизного тега, такого как `@beta`/`@rc`, или точной пререлизной версии. Пример `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Каждая запись указывает на директорию хука, содержащую `HOOK.md` и `handler.ts` (или `index.ts`). Пакеты хуков могут включать зависимости; они будут установлены в `~/.openclaw/hooks/`. Каждая запись `openclaw.hooks` должна оставаться внутри директории пакета после разрешения символьных ссылок; записи, выходящие за пределы, отклоняются. Примечание по безопасности: `openclaw hooks install` устанавливает зависимости с помощью `npm install --ignore-scripts` (без скриптов жизненного цикла). Держите деревья зависимостей пакетов хуков «чистыми JS/TS» и избегайте пакетов, которые полагаются на сборку `postinstall`.

## Структура хука

### Формат HOOK.md

Файл `HOOK.md` содержит метаданные в YAML frontmatter плюс документацию Markdown:

```
---
name: my-hook
description: "Краткое описание того, что делает этот хук"
homepage: https://docs.openclaw.ai/automation/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# Мой хук

Подробная документация находится здесь...

## Что он делает

- Слушает команды `/new`
- Выполняет некоторое действие
- Логирует результат

## Требования

- Должен быть установлен Node.js

## Конфигурация

Конфигурация не требуется.
```

### Поля метаданных

Объект `metadata.openclaw` поддерживает:

-   **`emoji`**: Emoji для отображения в CLI (например, `"💾"`)
-   **`events`**: Массив событий для прослушивания (например, `["command:new", "command:reset"]`)
-   **`export`**: Именованный экспорт для использования (по умолчанию `"default"`)
-   **`homepage`**: URL документации
-   **`requires`**: Необязательные требования
    -   **`bins`**: Требуемые бинарные файлы в PATH (например, `["git", "node"]`)
    -   **`anyBins`**: Должен присутствовать хотя бы один из этих бинарных файлов
    -   **`env`**: Требуемые переменные окружения
    -   **`config`**: Требуемые пути конфигурации (например, `["workspace.dir"]`)
    -   **`os`**: Требуемые платформы (например, `["darwin", "linux"]`)
-   **`always`**: Обойти проверки соответствия (логическое значение)
-   **`install`**: Методы установки (для встроенных хуков: `[{"id":"bundled","kind":"bundled"}]`)

### Реализация обработчика

Файл `handler.ts` экспортирует функцию `HookHandler`:

```typescript
const myHandler = async (event) => {
  // Срабатывает только на команду 'new'
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] Команда new запущена`);
  console.log(`  Сессия: ${event.sessionKey}`);
  console.log(`  Метка времени: ${event.timestamp.toISOString()}`);

  // Ваша пользовательская логика здесь

  // Опционально отправить сообщение пользователю
  event.messages.push("✨ Мой хук выполнен!");
};

export default myHandler;
```

#### Контекст события

Каждое событие включает:

```json
{
  type: 'command' | 'session' | 'agent' | 'gateway' | 'message',
  action: string,              // например, 'new', 'reset', 'stop', 'received', 'sent'
  sessionKey: string,          // Идентификатор сессии
  timestamp: Date,             // Когда произошло событие
  messages: string[],          // Добавляйте сюда сообщения для отправки пользователю
  context: {
    // События команд:
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // например, 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig,
    // События сообщений (см. раздел События сообщений для полных деталей):
    from?: string,             // message:received
    to?: string,               // message:sent
    content?: string,
    channelId?: string,
    success?: boolean,         // message:sent
  }
}
```

## Типы событий

### События команд

Срабатывают при выполнении команд агента:

-   **`command`**: Все события команд (общий слушатель)
-   **`command:new`**: При выполнении команды `/new`
-   **`command:reset`**: При выполнении команды `/reset`
-   **`command:stop`**: При выполнении команды `/stop`

### События сессии

-   **`session:compact:before`**: Непосредственно перед сжатием, которое суммирует историю
-   **`session:compact:after`**: После завершения сжатия с метаданными сводки

Внутренние полезные нагрузки хуков испускают их как `type: "session"` с `action: "compact:before"` / `action: "compact:after"`; слушатели подписываются с помощью объединенных ключей выше. Конкретная регистрация обработчика использует буквальный формат ключа `${type}:${action}`. Для этих событий регистрируйте `session:compact:before` и `session:compact:after`.

### События агента

-   **`agent:bootstrap`**: Перед внедрением файлов начальной загрузки рабочего пространства (хуки могут изменять `context.bootstrapFiles`)

### События шлюза

Срабатывают при запуске шлюза:

-   **`gateway:startup`**: После запуска каналов и загрузки хуков

### События сообщений

Срабатывают при получении или отправке сообщений:

-   **`message`**: Все события сообщений (общий слушатель)
-   **`message:received`**: Когда входящее сообщение получено из любого канала. Срабатывает рано в процессе обработки, до понимания медиа. Содержимое может содержать необработанные заполнители, такие как `<media:audio>` для медиавложений, которые еще не были обработаны.
-   **`message:transcribed`**: Когда сообщение было полностью обработано, включая транскрипцию аудио и понимание ссылок. На этом этапе `transcript` содержит полный текст транскрипции для аудиосообщений. Используйте этот хук, когда вам нужен доступ к транскрибированному аудиосодержимому.
-   **`message:preprocessed`**: Срабатывает для каждого сообщения после завершения всего понимания медиа + ссылок, давая хукам доступ к полностью обогащенному телу (транскриптам, описаниям изображений, сводкам ссылок) до того, как агент его увидит.
-   **`message:sent`**: Когда исходящее сообщение успешно отправлено

#### Контекст события сообщения

События сообщений включают богатый контекст о сообщении:

```
// контекст message:received
{
  from: string,           // Идентификатор отправителя (номер телефона, ID пользователя и т.д.)
  content: string,        // Содержимое сообщения
  timestamp?: number,     // Unix-метка времени получения
  channelId: string,      // Канал (например, "whatsapp", "telegram", "discord")
  accountId?: string,     // ID аккаунта провайдера для многопользовательских настроек
  conversationId?: string, // ID чата/беседы
  messageId?: string,     // ID сообщения от провайдера
  metadata?: {            // Дополнительные данные, специфичные для провайдера
    to?: string,
    provider?: string,
    surface?: string,
    threadId?: string,
    senderId?: string,
    senderName?: string,
    senderUsername?: string,
    senderE164?: string,
  }
}

// контекст message:sent
{
  to: string,             // Идентификатор получателя
  content: string,        // Содержимое отправленного сообщения
  success: boolean,       // Успешна ли отправка
  error?: string,         // Сообщение об ошибке, если отправка не удалась
  channelId: string,      // Канал (например, "whatsapp", "telegram", "discord")
  accountId?: string,     // ID аккаунта провайдера
  conversationId?: string, // ID чата/беседы
  messageId?: string,     // ID сообщения, возвращенный провайдером
  isGroup?: boolean,      // Принадлежит ли это исходящее сообщение групповому/канальному контексту
  groupId?: string,       // Идентификатор группы/канала для корреляции с message:received
}

// контекст message:transcribed
{
  body?: string,          // Необработанное входящее тело до обогащения
  bodyForAgent?: string,  // Обогащенное тело, видимое агенту
  transcript: string,     // Текст транскрипции аудио
  channelId: string,      // Канал (например, "telegram", "whatsapp")
  conversationId?: string,
  messageId?: string,
}

// контекст message:preprocessed
{
  body?: string,          // Необработанное входящее тело
  bodyForAgent?: string,  // Окончательное обогащенное тело после понимания медиа/ссылок
  transcript?: string,    // Транскрипция при наличии аудио
  channelId: string,      // Канал (например, "telegram", "whatsapp")
  conversationId?: string,
  messageId?: string,
  isGroup?: boolean,
  groupId?: string,
}
```

#### Пример: Хук-логгер сообщений

```typescript
const isMessageReceivedEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "received";
const isMessageSentEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "sent";

const handler = async (event) => {
  if (isMessageReceivedEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Получено от ${event.context.from}: ${event.context.content}`);
  } else if (isMessageSentEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Отправлено ${event.context.to}: ${event.context.content}`);
  }
};

export default handler;
```

### Хуки результатов инструментов (API плагина)

Эти хуки не являются слушателями потока событий; они позволяют плагинам синхронно корректировать результаты инструментов до того, как OpenClaw сохранит их.

-   **`tool_result_persist`**: преобразует результаты инструментов перед записью в транскрипт сессии. Должен быть синхронным; возвращает обновленную полезную нагрузку результата инструмента или `undefined`, чтобы оставить как есть. См. [Цикл агента](../concepts/agent-loop.md).

### События хуков плагина

Хуки жизненного цикла сжатия, предоставляемые через запускатель хуков плагина:

-   **`before_compaction`**: Запускается перед сжатием с метаданными количества/токенов
-   **`after_compaction`**: Запускается после сжатия с метаданными сводки сжатия

### Будущие события

Планируемые типы событий:

-   **`session:start`**: Когда начинается новая сессия
-   **`session:end`**: Когда сессия завершается
-   **`agent:error`**: Когда агент сталкивается с ошибкой

## Создание пользовательских хуков

### 1\. Выберите расположение

-   **Хуки рабочего пространства** (`/hooks/`): На агента, наивысший приоритет
-   **Управляемые хуки** (`~/.openclaw/hooks/`): Общие для всех рабочих пространств

### 2\. Создайте структуру директорий

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3\. Создайте HOOK.md

```
---
name: my-hook
description: "Делает что-то полезное"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# Мой пользовательский хук

Этот хук делает что-то полезное, когда вы выполняете `/new`.
```

### 4\. Создайте handler.ts

```typescript
const handler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Запуск!");
  // Ваша логика здесь
};

export default handler;
```

### 5\. Включите и протестируйте

```bash
# Убедитесь, что хук обнаружен
openclaw hooks list

# Включите его
openclaw hooks enable my-hook

# Перезапустите процесс шлюза (перезапуск приложения в строке меню на macOS или перезапуск вашего процесса разработки)

# Запустите событие
# Отправьте /new через ваш мессенджер-канал
```

## Конфигурация

### Новый формат конфигурации (рекомендуется)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### Конфигурация на хук

Хуки могут иметь пользовательскую конфигурацию:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### Дополнительные директории

Загружайте хуки из дополнительных директорий:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### Устаревший формат конфигурации (все еще поддерживается)

Старый формат конфигурации все еще работает для обратной совместимости:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

Примечание: `module` должен быть путем относительно рабочего пространства. Абсолютные пути и переходы за пределы рабочего пространства отклоняются. **Миграция**: Используйте новую систему на основе обнаружения для новых хуков. Устаревшие обработчики загружаются после хуков на основе директорий.

## CLI-команды

### Список хуков

```bash
# Список всех хуков
openclaw hooks list

# Показать только подходящие хуки
openclaw hooks list --eligible

# Подробный вывод (показать отсутствующие требования)
openclaw hooks list --verbose

# JSON-вывод
openclaw hooks list --json
```

### Информация о хуке

```bash
# Показать подробную информацию о хуке
openclaw hooks info session-memory

# JSON-вывод
openclaw hooks info session-memory --json
```

### Проверка соответствия

```bash
# Показать сводку соответствия
openclaw hooks check

# JSON-вывод
openclaw hooks check --json
```

### Включение/Отключение

```bash
# Включить хук
openclaw hooks enable session-memory

# Отключить хук
openclaw hooks disable command-logger
```

## Справочник по встроенным хукам

### session-memory

Сохраняет контекст сессии в память при выполнении команды `/new`. **События**: `command:new` **Требования**: `workspace.dir` должен быть настроен **Вывод**: `/memory/YYYY-MM-DD-slug.md` (по умолчанию `~/.openclaw/workspace`) **Что он делает**:

1.  Использует запись сессии до сброса для поиска правильного транскрипта
2.  Извлекает последние 15 строк разговора
3.  Использует LLM для генерации описательного slug-имени файла
4.  Сохраняет метаданные сессии в файл памяти с датой

**Пример вывода**:

```bash
# Сессия: 2026-01-16 14:30:00 UTC

- **Ключ сессии**: agent:main:main
- **ID сессии**: abc123def456
- **Источник**: telegram
```

**Примеры имен файлов**:

-   `2026-01-16-vendor-pitch.md`
-   `2026-01-16-api-design.md`
-   `2026-01-16-1430.md` (резервная метка времени, если генерация slug не удалась)

**Включить**:

```bash
openclaw hooks enable session-memory
```

### bootstrap-extra-files

Внедряет дополнительные файлы начальной загрузки (например, локальные для монорепозитория `AGENTS.md` / `TOOLS.md`) во время `agent:bootstrap`. **События**: `agent:bootstrap` **Требования**: `workspace.dir` должен быть настроен **Вывод**: Файлы не записываются; контекст начальной загрузки изменяется только в памяти. **Конфигурация**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "bootstrap-extra-files": {
          "enabled": true,
          "paths": ["packages/*/AGENTS.md", "packages/*/TOOLS.md"]
        }
      }
    }
  }
}
```

**Примечания**:

-   Пути разрешаются относительно рабочего пространства.
-   Файлы должны оставаться внутри рабочего пространства (проверяется realpath).
-   Загружаются только распознанные базовые имена файлов начальной загрузки.
-   Белый список сабагентов сохраняется (только `AGENTS.md` и `TOOLS.md`).

**Включить**:

```bash
openclaw hooks enable bootstrap-extra-files
```

### command-logger

Логирует все события команд в централизованный файл аудита. **События**: `command` **Требования**: Отсутствуют **Вывод**: `~/.openclaw/logs/commands.log` **Что он делает**:

1.  Захватывает детали события (действие команды, метка времени, ключ сессии, ID отправителя, источник)
2.  Добавляет в файл журнала в формате JSONL
3.  Работает тихо в фоновом режиме

**Примеры записей журнала**:

```json
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Просмотр журналов**:

```bash
# Просмотр последних команд
tail -n 20 ~/.openclaw/logs/commands.log

# Красивое форматирование с jq
cat ~/.openclaw/logs/commands.log | jq .

# Фильтрация по действию
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Включить**:

```bash
openclaw hooks enable command-logger
```

### boot-md

Запускает `BOOT.md` при старте шлюза (после запуска каналов). Внутренние хуки должны быть включены для его работы. **События**: `gateway:startup` **Требования**: `workspace.dir` должен быть настроен **Что он делает**:

1.  Читает `BOOT.md` из вашего рабочего пространства
2.  Запускает инструкции через раннер агента
3.  Отправляет любые запрошенные исходящие сообщения через инструмент сообщений

**Включить**:

```bash
openclaw hooks enable boot-md
```

## Рекомендации

### Делайте обработчики быстрыми

Хуки выполняются во время обработки команд. Делайте их легковесными:

```
// ✓ Хорошо - асинхронная работа, возвращается немедленно
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Запустить и забыть
};

// ✗ Плохо - блокирует обработку команд
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### Обрабатывайте ошибки корректно

Всегда оборачивайте рискованные операции:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Ошибка:", err instanceof Error ? err.message : String(err));
    // Не выбрасывайте - дайте другим обработчикам выполниться
  }
};
```

### Фильтруйте события рано

Возвращайтесь раньше, если событие не релевантно:

```typescript
const handler: HookHandler = async (event) => {
  // Обрабатывать только команды 'new'
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Ваша логика здесь
};
```

### Используйте конкретные ключи событий

По возможности указывайте точные события в метаданных:

```
metadata: { "openclaw": { "events": ["command:new"] } } # Конкретно
```

Вместо:

```
metadata: { "openclaw": { "events": ["command"] } } # Общее - больше накладных расходов
```

## Отладка

### Включение логирования хуков

Шлюз логирует загрузку хуков при запуске:

```bash
Registered hook: session-memory -> command:new
Registered hook: bootstrap-extra-files -> agent:bootstrap
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Проверка обнаружения

Список всех обнаруженных хуков:

```bash
openclaw hooks list --verbose
```

### Проверка регистрации

В вашем обработчике логируйте, когда он вызывается:

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Сработало:", event.type, event.action);
  // Ваша логика
};
```

### Проверка соответствия

Проверьте, почему хук не соответствует требованиям:

```bash
openclaw hooks info my-hook
```

Ищите отсутствующие требования в выводе.

## Тестирование

### Журналы шлюза

Мониторьте журналы шлюза, чтобы видеть выполнение хуков:

```bash
# macOS
./scripts/clawlog.sh -f

# Другие платформы
tail -f ~/.openclaw/gateway.log
```

### Прямое тестирование хуков

Тестируйте ваши обработчики изолированно:

```typescript
import { test } from "vitest";
import myHandler from "./hooks/my-hook/handler.js";

test("мой обработчик работает", async () => {
  const event = {
    type: "command",
    action: "new",
    sessionKey: "test-session",
    timestamp: new Date(),
    messages: [],
    context: { foo: "bar" },
  };

  await myHandler(event);

  // Проверьте побочные эффекты
});
```

## Архитектура

### Основные компоненты

-   **`src/hooks/types.ts`**: Определения типов
-   **`src/hooks/workspace.ts`**: Сканирование и загрузка директорий
-   **`src/hooks/frontmatter.ts`**: Парсинг метаданных HOOK.md
-   **`src/hooks/config.ts`**: Проверка соответствия
-   **`src/hooks/hooks-status.ts`**: Отчеты о статусе
-   **`src/hooks/loader.ts`**: Динамический загрузчик модулей
-   **`src/cli/hooks-cli.ts`**: CLI-команды
-   **`src/gateway/server-startup.ts`**: Загружает хуки при запуске шлюза
-   **`src/auto-reply/reply/commands-core.ts`**: Запускает события команд

### Поток обнаружения

```
Запуск шлюза
    ↓
Сканирование директорий (рабочее пространство → управляемые → встроенные)
    ↓
Парсинг файлов HOOK.md
    ↓
Проверка соответствия (bins, env, config, os)
    ↓
Загрузка обработчиков из подходящих хуков
    ↓
Регистрация обработчиков для событий
```

### Поток событий

```
Пользователь отправляет /new
    ↓
Проверка команды
    ↓
Создание события хука
    ↓
Запуск хука (все зарегистрированные обработчики)
    ↓
Обработка команды продолжается
    ↓
Сброс сессии
```

## Устранение неполадок

### Хук не обнаружен

1.  Проверьте структуру директорий:
    
    ```bash
    ls -la ~/.openclaw/hooks/my-hook/
    # Должно показывать: HOOK.md, handler.ts
    ```
    
2.  Проверьте формат HOOK.md:
    
    ```bash
    cat ~/.openclaw/hooks/my-hook/HOOK.md
    # Должен иметь YAML frontmatter с name и metadata
    ```
    
3.  Список всех обнаруженных хуков:
    
    ```bash
    openclaw hooks list
    ```
    

### Хук не соответствует требованиям

Проверьте требования:

```bash
openclaw hooks info my-hook
```

Ищите отсутствующие:

-   Бинарные файлы (проверьте PATH)
-   Переменные окружения
-   Значения конфигурации
-   Совместимость с ОС

### Хук не выполняется

1.  Убедитесь, что хук включен:
    
    ```bash
    openclaw hooks list
    # Должен показывать ✓ рядом с включенными хуками
    ```
    
2.  Перезапустите процесс шлюза, чтобы хуки перезагрузились.
3.  Проверьте журналы шлюза на наличие ошибок:
    
    ```bash
    ./scripts/clawlog.sh | grep hook
    ```
    

### Ошибки обработчика

Проверьте на ошибки TypeScript/импорта:

```bash
# Протестируйте импорт напрямую
node -e "import('./path/to/handler.ts').then(