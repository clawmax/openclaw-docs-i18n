

  Основы

  
# Архитектура интеграции Pi

В этом документе описывается, как OpenClaw интегрируется с [pi-coding-agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) и связанными пакетами (`pi-ai`, `pi-agent-core`, `pi-tui`) для обеспечения возможностей ИИ-агента.

## Обзор

OpenClaw использует SDK pi для встраивания ИИ-агента кодирования в свою архитектуру шлюза обмена сообщениями. Вместо запуска pi как подпроцесса или использования режима RPC, OpenClaw напрямую импортирует и создает экземпляр `AgentSession` pi через `createAgentSession()`. Такой встроенный подход обеспечивает:

-   Полный контроль над жизненным циклом сессии и обработкой событий
-   Внедрение пользовательских инструментов (обмен сообщениями, песочница, действия для конкретных каналов)
-   Настройку системного промпта для каждого канала/контекста
-   Сохранение сессий с поддержкой ветвления/компактизации
-   Ротацию профилей аутентификации для нескольких аккаунтов с отказоустойчивостью
-   Независимое от провайдера переключение моделей

## Зависимости пакетов

```json
{
  "@mariozechner/pi-agent-core": "0.49.3",
  "@mariozechner/pi-ai": "0.49.3",
  "@mariozechner/pi-coding-agent": "0.49.3",
  "@mariozechner/pi-tui": "0.49.3"
}
```

| Пакет | Назначение |
| --- | --- |
| `pi-ai` | Основные абстракции LLM: `Model`, `streamSimple`, типы сообщений, API провайдеров |
| `pi-agent-core` | Цикл агента, выполнение инструментов, типы `AgentMessage` |
| `pi-coding-agent` | SDK высокого уровня: `createAgentSession`, `SessionManager`, `AuthStorage`, `ModelRegistry`, встроенные инструменты |
| `pi-tui` | Компоненты терминального интерфейса (используются в локальном режиме TUI OpenClaw) |

## Структура файлов

```
src/agents/
├── pi-embedded-runner.ts          # Реэкспорт из pi-embedded-runner/
├── pi-embedded-runner/
│   ├── run.ts                     # Основная точка входа: runEmbeddedPiAgent()
│   ├── run/
│   │   ├── attempt.ts             # Логика единичной попытки с настройкой сессии
│   │   ├── params.ts              # Тип RunEmbeddedPiAgentParams
│   │   ├── payloads.ts            # Создание полезных данных ответа из результатов запуска
│   │   ├── images.ts              # Внедрение изображений для моделей с поддержкой зрения
│   │   └── types.ts               # EmbeddedRunAttemptResult
│   ├── abort.ts                   # Обнаружение ошибки прерывания
│   ├── cache-ttl.ts               # Отслеживание TTL кэша для обрезки контекста
│   ├── compact.ts                 # Логика ручной/автоматической компактизации
│   ├── extensions.ts              # Загрузка расширений pi для встроенных запусков
│   ├── extra-params.ts            # Параметры потока, специфичные для провайдера
│   ├── google.ts                  # Исправления порядка реплик для Google/Gemini
│   ├── history.ts                 # Ограничение истории (личные сообщения vs группа)
│   ├── lanes.ts                   # Сессионные/глобальные командные линии
│   ├── logger.ts                  # Логгер подсистемы
│   ├── model.ts                   # Разрешение модели через ModelRegistry
│   ├── runs.ts                    # Отслеживание активных запусков, прерывание, очередь
│   ├── sandbox-info.ts            # Информация о песочнице для системного промпта
│   ├── session-manager-cache.ts   # Кэширование экземпляров SessionManager
│   ├── session-manager-init.ts    # Инициализация файла сессии
│   ├── system-prompt.ts           # Конструктор системного промпта
│   ├── tool-split.ts              # Разделение инструментов на builtIn и custom
│   ├── types.ts                   # EmbeddedPiAgentMeta, EmbeddedPiRunResult
│   └── utils.ts                   # Сопоставление ThinkLevel, описание ошибки
├── pi-embedded-subscribe.ts       # Подписка на события сессии и их диспетчеризация
├── pi-embedded-subscribe.types.ts # SubscribeEmbeddedPiSessionParams
├── pi-embedded-subscribe.handlers.ts # Фабрика обработчиков событий
├── pi-embedded-subscribe.handlers.lifecycle.ts
├── pi-embedded-subscribe.handlers.types.ts
├── pi-embedded-block-chunker.ts   # Разбиение потокового ответа на блоки
├── pi-embedded-messaging.ts       # Отслеживание отправленных сообщений инструментами
├── pi-embedded-helpers.ts         # Классификация ошибок, валидация реплик
├── pi-embedded-helpers/           # Вспомогательные модули
├── pi-embedded-utils.ts           # Утилиты форматирования
├── pi-tools.ts                    # createOpenClawCodingTools()
├── pi-tools.abort.ts              # Обертка AbortSignal для инструментов
├── pi-tools.policy.ts             # Политика разрешенных/запрещенных инструментов
├── pi-tools.read.ts               # Кастомизация инструмента read
├── pi-tools.schema.ts             # Нормализация схем инструментов
├── pi-tools.types.ts              # Псевдоним типа AnyAgentTool
├── pi-tool-definition-adapter.ts  # Адаптер AgentTool -> ToolDefinition
├── pi-settings.ts                 # Переопределения настроек
├── pi-extensions/                 # Пользовательские расширения pi
│   ├── compaction-safeguard.ts    # Расширение защиты компактизации
│   ├── compaction-safeguard-runtime.ts
│   ├── context-pruning.ts         # Расширение обрезки контекста на основе Cache-TTL
│   └── context-pruning/
├── model-auth.ts                  # Разрешение профиля аутентификации
├── auth-profiles.ts               # Хранилище профилей, время ожидания, отказоустойчивость
├── model-selection.ts             # Разрешение модели по умолчанию
├── models-config.ts               # Генерация models.json
├── model-catalog.ts               # Кэш каталога моделей
├── context-window-guard.ts        # Проверка окна контекста
├── failover-error.ts              # Класс FailoverError
├── defaults.ts                    # DEFAULT_PROVIDER, DEFAULT_MODEL
├── system-prompt.ts               # buildAgentSystemPrompt()
├── system-prompt-params.ts        # Разрешение параметров системного промпта
├── system-prompt-report.ts        # Генерация отладочного отчета
├── tool-summaries.ts              # Сводки описаний инструментов
├── tool-policy.ts                 # Разрешение политики инструментов
├── transcript-policy.ts           # Политика валидации транскрипта
├── skills.ts                      # Снимок навыков/построение промпта
├── skills/                        # Подсистема навыков
├── sandbox.ts                     # Разрешение контекста песочницы
├── sandbox/                       # Подсистема песочницы
├── channel-tools.ts               # Внедрение инструментов для конкретного канала
├── openclaw-tools.ts              # Инструменты, специфичные для OpenClaw
├── bash-tools.ts                  # Инструменты exec/process
├── apply-patch.ts                 # Инструмент apply_patch (OpenAI)
├── tools/                         # Реализации отдельных инструментов
│   ├── browser-tool.ts
│   ├── canvas-tool.ts
│   ├── cron-tool.ts
│   ├── discord-actions*.ts
│   ├── gateway-tool.ts
│   ├── image-tool.ts
│   ├── message-tool.ts
│   ├── nodes-tool.ts
│   ├── session*.ts
│   ├── slack-actions.ts
│   ├── telegram-actions.ts
│   ├── web-*.ts
│   └── whatsapp-actions.ts
└── ...
```

## Основной поток интеграции

### 1\. Запуск встроенного агента

Основная точка входа — `runEmbeddedPiAgent()` в `pi-embedded-runner/run.ts`:

```typescript
import { runEmbeddedPiAgent } from "./agents/pi-embedded-runner.js";

const result = await runEmbeddedPiAgent({
  sessionId: "user-123",
  sessionKey: "main:whatsapp:+1234567890",
  sessionFile: "/path/to/session.jsonl",
  workspaceDir: "/path/to/workspace",
  config: openclawConfig,
  prompt: "Hello, how are you?",
  provider: "anthropic",
  model: "claude-sonnet-4-20250514",
  timeoutMs: 120_000,
  runId: "run-abc",
  onBlockReply: async (payload) => {
    await sendToChannel(payload.text, payload.mediaUrls);
  },
});
```

### 2\. Создание сессии

Внутри `runEmbeddedAttempt()` (вызывается `runEmbeddedPiAgent()`) используется SDK pi:

```typescript
import {
  createAgentSession,
  DefaultResourceLoader,
  SessionManager,
  SettingsManager,
} from "@mariozechner/pi-coding-agent";

const resourceLoader = new DefaultResourceLoader({
  cwd: resolvedWorkspace,
  agentDir,
  settingsManager,
  additionalExtensionPaths,
});
await resourceLoader.reload();

const { session } = await createAgentSession({
  cwd: resolvedWorkspace,
  agentDir,
  authStorage: params.authStorage,
  modelRegistry: params.modelRegistry,
  model: params.model,
  thinkingLevel: mapThinkingLevel(params.thinkLevel),
  tools: builtInTools,
  customTools: allCustomTools,
  sessionManager,
  settingsManager,
  resourceLoader,
});

applySystemPromptOverrideToSession(session, systemPromptOverride);
```

### 3\. Подписка на события

`subscribeEmbeddedPiSession()` подписывается на события `AgentSession` pi:

```typescript
const subscription = subscribeEmbeddedPiSession({
  session: activeSession,
  runId: params.runId,
  verboseLevel: params.verboseLevel,
  reasoningMode: params.reasoningLevel,
  toolResultFormat: params.toolResultFormat,
  onToolResult: params.onToolResult,
  onReasoningStream: params.onReasoningStream,
  onBlockReply: params.onBlockReply,
  onPartialReply: params.onPartialReply,
  onAgentEvent: params.onAgentEvent,
});
```

Обрабатываемые события включают:

-   `message_start` / `message_end` / `message_update` (потоковый текст/рассуждения)
-   `tool_execution_start` / `tool_execution_update` / `tool_execution_end`
-   `turn_start` / `turn_end`
-   `agent_start` / `agent_end`
-   `auto_compaction_start` / `auto_compaction_end`

### 4\. Отправка промпта

После настройки сессии отправляется промпт:

```typescript
await session.prompt(effectivePrompt, { images: imageResult.images });
```

SDK обрабатывает полный цикл агента: отправка в LLM, выполнение вызовов инструментов, потоковые ответы. Внедрение изображений локально для промпта: OpenClaw загружает ссылки на изображения из текущего промпта и передает их через `images` только для этого хода. Он не сканирует повторно старые ходы истории для повторного внедрения изображений.

## Архитектура инструментов

### Конвейер инструментов

1.  **Базовые инструменты**: `codingTools` pi (read, bash, edit, write)
2.  **Пользовательские замены**: OpenClaw заменяет bash на `exec`/`process`, кастомизирует read/edit/write для песочницы
3.  **Инструменты OpenClaw**: обмен сообщениями, браузер, canvas, сессии, cron, шлюз и т.д.
4.  **Инструменты каналов**: инструменты действий, специфичные для Discord/Telegram/Slack/WhatsApp
5.  **Фильтрация по политике**: Инструменты фильтруются по политикам профиля, провайдера, агента, группы, песочницы
6.  **Нормализация схем**: Схемы очищаются для особенностей Gemini/OpenAI
7.  **Обертка AbortSignal**: Инструменты обернуты для уважения сигналов прерывания

### Адаптер определения инструментов

`AgentTool` из pi-agent-core имеет другую сигнатуру `execute`, чем `ToolDefinition` из pi-coding-agent. Адаптер в `pi-tool-definition-adapter.ts` устраняет этот разрыв:

```bash
export function toToolDefinitions(tools: AnyAgentTool[]): ToolDefinition[] {
  return tools.map((tool) => ({
    name: tool.name,
    label: tool.label ?? name,
    description: tool.description ?? "",
    parameters: tool.parameters,
    execute: async (toolCallId, params, onUpdate, _ctx, signal) => {
      // Сигнатура pi-coding-agent отличается от pi-agent-core
      return await tool.execute(toolCallId, params, signal, onUpdate);
    },
  }));
}
```

### Стратегия разделения инструментов

`splitSdkTools()` передает все инструменты через `customTools`:

```bash
export function splitSdkTools(options: { tools: AnyAgentTool[]; sandboxEnabled: boolean }) {
  return {
    builtInTools: [], // Пусто. Мы переопределяем всё
    customTools: toToolDefinitions(options.tools),
  };
}
```

Это гарантирует, что фильтрация по политике OpenClaw, интеграция с песочницей и расширенный набор инструментов остаются согласованными для всех провайдеров.

## Построение системного промпта

Системный промпт строится в `buildAgentSystemPrompt()` (`system-prompt.ts`). Он собирает полный промпт с разделами, включая Инструменты, Стиль вызова инструментов, Защитные ограничения, Справку по CLI OpenClaw, Навыки, Документацию, Рабочее пространство, Песочницу, Обмен сообщениями, Теги ответов, Голос, Тихие ответы, Пульс, Метаданные времени выполнения, плюс Память и Реакции, когда они включены, и опциональные файлы контекста и дополнительный системный промпт. Разделы обрезаются для минимального режима промпта, используемого субагентами. Промпт применяется после создания сессии через `applySystemPromptOverrideToSession()`:

```typescript
const systemPromptOverride = createSystemPromptOverride(appendPrompt);
applySystemPromptOverrideToSession(session, systemPromptOverride);
```

## Управление сессиями

### Файлы сессий

Сессии — это файлы JSONL с древовидной структурой (связывание id/parentId). `SessionManager` pi обрабатывает сохранение:

```typescript
const sessionManager = SessionManager.open(params.sessionFile);
```

OpenClaw оборачивает это `guardSessionManager()` для безопасности результатов инструментов.

### Кэширование сессий

`session-manager-cache.ts` кэширует экземпляры SessionManager, чтобы избежать повторного разбора файлов:

```typescript
await prewarmSessionFile(params.sessionFile);
sessionManager = SessionManager.open(params.sessionFile);
trackSessionManagerAccess(params.sessionFile);
```

### Ограничение истории

`limitHistoryTurns()` обрезает историю разговора в зависимости от типа канала (личные сообщения vs группа).

### Компактизация

Автоматическая компактизация срабатывает при переполнении контекста. `compactEmbeddedPiSessionDirect()` обрабатывает ручную компактизацию:

```typescript
const compactResult = await compactEmbeddedPiSessionDirect({
  sessionId, sessionFile, provider, model, ...
});
```

## Аутентификация и разрешение модели

### Профили аутентификации

OpenClaw поддерживает хранилище профилей аутентификации с несколькими API-ключами на провайдера:

```typescript
const authStore = ensureAuthProfileStore(agentDir, { allowKeychainPrompt: false });
const profileOrder = resolveAuthProfileOrder({ cfg, store: authStore, provider, preferredProfile });
```

Профили ротируются при сбоях с отслеживанием времени ожидания:

```typescript
await markAuthProfileFailure({ store, profileId, reason, cfg, agentDir });
const rotated = await advanceAuthProfile();
```

### Разрешение модели

```typescript
import { resolveModel } from "./pi-embedded-runner/model.js";

const { model, error, authStorage, modelRegistry } = resolveModel(
  provider,
  modelId,
  agentDir,
  config,
);

// Использует ModelRegistry и AuthStorage pi
authStorage.setRuntimeApiKey(model.provider, apiKeyInfo.apiKey);
```

### Отказоустойчивость

`FailoverError` запускает переход на резервную модель при настройке:

```
if (fallbackConfigured && isFailoverErrorMessage(errorText)) {
  throw new FailoverError(errorText, {
    reason: promptFailoverReason ?? "unknown",
    provider,
    model: modelId,
    profileId,
    status: resolveFailoverStatus(promptFailoverReason),
  });
}
```

## Расширения Pi

OpenClaw загружает пользовательские расширения pi для специализированного поведения:

### Защита компактизации

`src/agents/pi-extensions/compaction-safeguard.ts` добавляет защитные ограничения для компактизации, включая адаптивное бюджетирование токенов, а также сводки сбоев инструментов и файловых операций:

```
if (resolveCompactionMode(params.cfg) === "safeguard") {
  setCompactionSafeguardRuntime(params.sessionManager, { maxHistoryShare });
  paths.push(resolvePiExtensionPath("compaction-safeguard"));
}
```

### Обрезка контекста

`src/agents/pi-extensions/context-pruning.ts` реализует обрезку контекста на основе cache-TTL:

```
if (cfg?.agents?.defaults?.contextPruning?.mode === "cache-ttl") {
  setContextPruningRuntime(params.sessionManager, {
    settings,
    contextWindowTokens,
    isToolPrunable,
    lastCacheTouchAt,
  });
  paths.push(resolvePiExtensionPath("context-pruning"));
}
```

## Потоковая передача и ответы блоками

### Разбиение на блоки

`EmbeddedBlockChunker` управляет потоковым текстом, разбивая его на отдельные блоки ответов:

```typescript
const blockChunker = blockChunking ? new EmbeddedBlockChunker(blockChunking) : null;
```

### Удаление тегов thinking/final

Потоковый вывод обрабатывается для удаления блоков ``/`` и извлечения содержимого ``:

```typescript
const stripBlockTags = (text: string, state: { thinking: boolean; final: boolean }) => {
  // Удалить содержимое <think>...</think>
  // Если enforceFinalTag, возвращать только содержимое <final>...</final>
};
```

### Директивы ответа

Директивы ответа, такие как `[[media:url]]`, `[[voice]]`, `[[reply:id]]`, анализируются и извлекаются:

```typescript
const { text: cleanedText, mediaUrls, audioAsVoice, replyToId } = consumeReplyDirectives(chunk);
```

## Обработка ошибок

### Классификация ошибок

`pi-embedded-helpers.ts` классифицирует ошибки для соответствующей обработки:

```
isContextOverflowError(errorText)     // Контекст слишком большой
isCompactionFailureError(errorText)   // Компактизация не удалась
isAuthAssistantError(lastAssistant)   // Ошибка аутентификации
isRateLimitAssistantError(...)        // Превышен лимит запросов
isFailoverAssistantError(...)         // Должен переключиться на резерв
classifyFailoverReason(errorText)     // "auth" | "rate_limit" | "quota" | "timeout" | ...
```

### Откат уровня рассуждений

Если уровень рассуждений не поддерживается, происходит откат:

```typescript
const fallbackThinking = pickFallbackThinkingLevel({
  message: errorText,
  attempted: attemptedThinking,
});
if (fallbackThinking) {
  thinkLevel = fallbackThinking;
  continue;
}
```

## Интеграция с песочницей

Когда включен режим песочницы, инструменты и пути ограничиваются:

```typescript
const sandbox = await resolveSandboxContext({
  config: params.config,
  sessionKey: sandboxSessionKey,
  workspaceDir: resolvedWorkspace,
});

if (sandboxRoot) {
  // Использовать инструменты read/edit/write в песочнице
  // Exec запускается в контейнере
  // Браузер использует URL моста
}
```

## Обработка, специфичная для провайдера

### Anthropic

-   Очистка магических строк отказа
-   Проверка реплик на последовательность ролей
-   Совместимость с параметром Claude Code

### Google/Gemini

-   Исправления порядка реплик (`applyGoogleTurnOrderingFix`)
-   Санитизация схем инструментов (`sanitizeToolsForGoogle`)
-   Санитизация истории сессии (`sanitizeSessionHistory`)

### OpenAI

-   Инструмент `apply_patch` для моделей Codex
-   Обработка понижения уровня рассуждений

## Интеграция TUI

OpenClaw также имеет локальный режим TUI, который напрямую использует компоненты pi-tui:

```bash
// src/tui/tui.ts
import { ... } from "@mariozechner/pi-tui";
```

Это обеспечивает интерактивный терминальный опыт, аналогичный нативному режиму pi.

## Ключевые отличия от Pi CLI

| Аспект | Pi CLI | OpenClaw Embedded |
| --- | --- | --- |
| Вызов | Команда `pi` / RPC | SDK через `createAgentSession()` |
| Инструменты | Инструменты кодирования по умолчанию | Пользовательский набор инструментов OpenClaw |
| Системный промпт | AGENTS.md + промпты | Динамический для каждого канала/контекста |
| Хранение сессий | `~/.pi/agent/sessions/` | `~/.openclaw/agents//sessions/` (или `$OPENCLAW_STATE_DIR/agents//sessions/`) |
| Аутентификация | Один набор учетных данных | Мультипрофиль с ротацией |
| Расширения | Загружаются с диска | Программные + пути на диске |
| Обработка событий | Отрисовка TUI | На основе обратных вызовов (onBlockReply и т.д.) |

## Будущие соображения

Области для потенциальной доработки:

1.  **Согласование сигнатур инструментов**: В настоящее время адаптация между сигнатурами pi-agent-core и pi-coding-agent
2.  **Оборачивание менеджера сессий**: `guardSessionManager` добавляет безопасность, но увеличивает сложность
3.  **Загрузка расширений**: Можно использовать `ResourceLoader` pi более напрямую
4.  **Сложность обработчиков потоковой передачи**: `subscribeEmbeddedPiSession` стал слишком большим
5.  **Особенности провайдеров**: Много кода, специфичного для провайдеров, который pi потенциально мог бы обрабатывать

## Тесты

Покрытие интеграции Pi включает следующие наборы:

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-auth-json.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-embedded-helpers*.test.ts`
-   `src/agents/pi-embedded-runner*.test.ts`
-   `src/agents/pi-embedded-runner/**/*.test.ts`
-   `src/agents/pi-embedded-subscribe*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-tool-definition-adapter*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-extensions/**/*.test.ts`

Живые/опциональные:

-   `src/agents/pi-embedded-runner-extraparams.live.test.ts` (включить `OPENCLAW_LIVE_TEST=1`)

Для текущих команд запуска см. [Рабочий процесс разработки Pi](./pi-dev.md).

[Архитектура шлюза](./concepts/architecture.md)