

  Окружение и отладка

  
# Тестирование

В OpenClaw есть три набора тестов Vitest (модульные/интеграционные, e2e, live) и небольшой набор Docker-раннеров. Этот документ — руководство «как мы тестируем»:

-   Что покрывает каждый набор (и что он *намеренно* не покрывает)
-   Какие команды запускать для распространённых рабочих процессов (локально, перед пушем, отладка)
-   Как live-тесты обнаруживают учётные данные и выбирают модели/провайдеры
-   Как добавлять регрессионные тесты для проблем с реальными моделями/провайдерами

## Быстрый старт

В большинстве случаев:

-   Полная проверка (ожидается перед пушем): `pnpm build && pnpm check && pnpm test`

Когда вы касаетесь тестов или хотите дополнительной уверенности:

-   Проверка покрытия: `pnpm test:coverage`
-   Набор E2E: `pnpm test:e2e`

При отладке реальных провайдеров/моделей (требуются реальные учётные данные):

-   Набор live (модели + проверки инструмента/образа шлюза): `pnpm test:live`

Совет: когда нужен только один падающий кейс, предпочтительнее сузить live-тесты с помощью переменных окружения allowlist, описанных ниже.

## Наборы тестов (что где запускается)

Думайте о наборах как о «возрастающей реалистичности» (и возрастающей хрупкости/стоимости):

### Модульные / интеграционные (по умолчанию)

-   Команда: `pnpm test`
-   Конфиг: `scripts/test-parallel.mjs` (запускает `vitest.unit.config.ts`, `vitest.extensions.config.ts`, `vitest.gateway.config.ts`)
-   Файлы: `src/**/*.test.ts`, `extensions/**/*.test.ts`
-   Область:
    -   Чистые модульные тесты
    -   In-process интеграционные тесты (аутентификация шлюза, маршрутизация, инструменты, парсинг, конфигурация)
    -   Детерминированные регрессии для известных багов
-   Ожидания:
    -   Запускается в CI
    -   Не требуются реальные ключи
    -   Должны быть быстрыми и стабильными
-   Примечание о пуле:
    -   OpenClaw использует Vitest `vmForks` на Node 22/23 для более быстрого разделения модульных тестов.
    -   На Node 24+ OpenClaw автоматически откатывается к обычным `forks`, чтобы избежать ошибок линковки Node VM (`ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`).
    -   Переопределите вручную с помощью `OPENCLAW_TEST_VM_FORKS=0` (принудительно `forks`) или `OPENCLAW_TEST_VM_FORKS=1` (принудительно `vmForks`).

### E2E (smoke-тест шлюза)

-   Команда: `pnpm test:e2e`
-   Конфиг: `vitest.e2e.config.ts`
-   Файлы: `src/**/*.e2e.test.ts`
-   Параметры выполнения по умолчанию:
    -   Использует Vitest `vmForks` для более быстрого запуска файлов.
    -   Использует адаптивных воркеров (CI: 2-4, локально: 4-8).
    -   Запускается в тихом режиме по умолчанию для снижения накладных расходов на ввод-вывод в консоли.
-   Полезные переопределения:
    -   `OPENCLAW_E2E_WORKERS=` для принудительного задания количества воркеров (ограничено 16).
    -   `OPENCLAW_E2E_VERBOSE=1` для повторного включения подробного вывода в консоль.
-   Область:
    -   Сквозное поведение шлюза с несколькими экземплярами
    -   Поверхности WebSocket/HTTP, сопряжение узлов и более тяжёлые сетевые взаимодействия
-   Ожидания:
    -   Запускается в CI (когда включено в пайплайне)
    -   Не требуются реальные ключи
    -   Больше подвижных частей, чем в модульных тестах (может быть медленнее)

### Live (реальные провайдеры + реальные модели)

-   Команда: `pnpm test:live`
-   Конфиг: `vitest.live.config.ts`
-   Файлы: `src/**/*.live.test.ts`
-   По умолчанию: **включено** командой `pnpm test:live` (устанавливает `OPENCLAW_LIVE_TEST=1`)
-   Область:
    -   «Работает ли этот провайдер/модель *сегодня* с реальными учётными данными?»
    -   Отлавливает изменения формата провайдера, особенности tool-calling, проблемы аутентификации и поведение rate limit
-   Ожидания:
    -   По замыслу нестабильны в CI (реальные сети, реальные политики провайдеров, квоты, сбои)
    -   Стоят денег / используют лимиты запросов
    -   Предпочтительнее запускать суженные подмножества, а не «всё»
    -   Live-запуски будут загружать `~/.profile`, чтобы подхватить отсутствующие API-ключи
-   Ротация API-ключей (специфичная для провайдера): установите `*_API_KEYS` в формате с запятой/точкой с запятой или `*_API_KEY_1`, `*_API_KEY_2` (например, `OPENAI_API_KEYS`, `ANTHROPIC_API_KEYS`, `GEMINI_API_KEYS`) или переопределение для конкретного live-теста через `OPENCLAW_LIVE_*_KEY`; тесты повторяются при ответах с rate limit.

## Какой набор мне запускать?

Используйте эту таблицу решений:

-   Редактирование логики/тестов: запустите `pnpm test` (и `pnpm test:coverage`, если много изменили)
-   Касаетесь сетевого взаимодействия шлюза / протокола WS / сопряжения: добавьте `pnpm test:e2e`
-   Отладка «мой бот не работает» / специфичных для провайдера сбоев / tool calling: запустите суженный `pnpm test:live`

## Live: проверка возможностей Android-узла

-   Тест: `src/gateway/android-node.capabilities.live.test.ts`
-   Скрипт: `pnpm android:test:integration`
-   Цель: вызвать **каждую команду, рекламируемую в данный момент** подключённым Android-узлом и проверить поведение контракта команды.
-   Область:
    -   Предварительно настроенная/ручная подготовка (набор не устанавливает/не запускает/не сопрягает приложение).
    -   Построчная валидация `node.invoke` шлюза для выбранного Android-узла.
-   Необходимая предварительная настройка:
    -   Android-приложение уже подключено + сопряжено со шлюзом.
    -   Приложение остаётся на переднем плане.
    -   Разрешения/согласие на захват предоставлены для возможностей, которые должны проходить.
-   Опциональные переопределения цели:
    -   `OPENCLAW_ANDROID_NODE_ID` или `OPENCLAW_ANDROID_NODE_NAME`.
    -   `OPENCLAW_ANDROID_GATEWAY_URL` / `OPENCLAW_ANDROID_GATEWAY_TOKEN` / `OPENCLAW_ANDROID_GATEWAY_PASSWORD`.
-   Подробности полной настройки Android: [Android App](../platforms/android.md)

## Live: smoke-тест моделей (ключи профиля)

Live-тесты разделены на два слоя, чтобы изолировать сбои:

-   «Прямая модель» говорит нам, что провайдер/модель вообще могут ответить с данным ключом.
-   «Smoke-тест шлюза» говорит нам, что полный пайплайн шлюза+агента работает для этой модели (сессии, история, инструменты, политика песочницы и т.д.).

### Слой 1: Прямое завершение модели (без шлюза)

-   Тест: `src/agents/models.profiles.live.test.ts`
-   Цель:
    -   Перечислить обнаруженные модели
    -   Использовать `getApiKeyForModel` для выбора моделей, для которых есть учётные данные
    -   Запустить небольшое завершение для каждой модели (и целевые регрессии, где нужно)
-   Как включить:
    -   `pnpm test:live` (или `OPENCLAW_LIVE_TEST=1` при прямом вызове Vitest)
-   Установите `OPENCLAW_LIVE_MODELS=modern` (или `all`, алиас для modern), чтобы фактически запустить этот набор; иначе он пропускается, чтобы `pnpm test:live` оставался сфокусированным на smoke-тесте шлюза
-   Как выбрать модели:
    -   `OPENCLAW_LIVE_MODELS=modern` для запуска современного allowlist (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.5, Grok 4)
    -   `OPENCLAW_LIVE_MODELS=all` — это алиас для современного allowlist
    -   или `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,..."` (allowlist через запятую)
-   Как выбрать провайдеров:
    -   `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (allowlist через запятую)
-   Откуда берутся ключи:
    -   По умолчанию: хранилище профилей и резервные env-переменные
    -   Установите `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1`, чтобы принудительно использовать **только хранилище профилей**
-   Зачем это нужно:
    -   Отделяет «API провайдера сломан / ключ невалиден» от «пайплайн агента шлюза сломан»
    -   Содержит небольшие изолированные регрессии (пример: воспроизведение рассуждений OpenAI Responses/Codex Responses + потоки tool-call)

### Слой 2: Smoke-тест шлюза + dev-агента (что фактически делает «@openclaw»)

-   Тест: `src/gateway/gateway-models.profiles.live.test.ts`
-   Цель:
    -   Запустить in-process шлюз
    -   Создать/обновить сессию `agent:dev:*` (переопределение модели на запуск)
    -   Перебрать модели с ключами и проверить:
        -   «Осмысленный» ответ (без инструментов)
        -   Работает реальный вызов инструмента (read probe)
        -   Опциональные дополнительные проверки инструментов (exec+read probe)
        -   Регрессионные пути OpenAI (tool-call-only → follow-up) продолжают работать
-   Детали проверок (чтобы можно было быстро объяснить сбои):
    -   `read` probe: тест записывает nonce-файл в рабочее пространство и просит агента `прочитать` его и вернуть nonce.
    -   `exec+read` probe: тест просит агента `выполнить` запись nonce во временный файл, затем `прочитать` его обратно.
    -   image probe: тест прикрепляет сгенерированный PNG (кот + рандомизированный код) и ожидает, что модель вернёт `cat <КОД>`.
    -   Ссылка на реализацию: `src/gateway/gateway-models.profiles.live.test.ts` и `src/gateway/live-image-probe.ts`.
-   Как включить:
    -   `pnpm test:live` (или `OPENCLAW_LIVE_TEST=1` при прямом вызове Vitest)
-   Как выбрать модели:
    -   По умолчанию: современный allowlist (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.5, Grok 4)
    -   `OPENCLAW_LIVE_GATEWAY_MODELS=all` — это алиас для современного allowlist
    -   Или установите `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (или список через запятую) для сужения
-   Как выбрать провайдеров (избежать «OpenRouter всё»):
    -   `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (allowlist через запятую)
-   Проверки инструментов и изображений всегда включены в этом live-тесте:
    -   `read` probe + `exec+read` probe (нагрузка на инструменты)
    -   image probe запускается, когда модель заявляет о поддержке ввода изображений
    -   Поток (высокоуровнево):
        -   Тест генерирует крошечный PNG с «CAT» + случайным кодом (`src/gateway/live-image-probe.ts`)
        -   Отправляет его через `agent` `attachments: [{ mimeType: "image/png", content: "" }]`
        -   Шлюз парсит вложения в `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
        -   Встроенный агент пересылает мультимодальное пользовательское сообщение модели
        -   Проверка: ответ содержит `cat` + код (допуск OCR: допускаются мелкие ошибки)

Совет: чтобы увидеть, что можно протестировать на вашей машине (и точные `provider/model` id), выполните:

```bash
openclaw models list
openclaw models list --json
```

## Live: smoke-тест setup-token Anthropic

-   Тест: `src/agents/anthropic.setup-token.live.test.ts`
-   Цель: проверить, что setup-token Claude Code CLI (или вставленный setup-token профиля) может завершить промпт Anthropic.
-   Включить:
    -   `pnpm test:live` (или `OPENCLAW_LIVE_TEST=1` при прямом вызове Vitest)
    -   `OPENCLAW_LIVE_SETUP_TOKEN=1`
-   Источники токена (выберите один):
    -   Профиль: `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
    -   Сырой токен: `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
-   Переопределение модели (опционально):
    -   `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-6`

Пример настройки:

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

## Live: smoke-тест CLI-бэкенда (Claude Code CLI или другие локальные CLI)

-   Тест: `src/gateway/gateway-cli-backend.live.test.ts`
-   Цель: проверить пайплайн Шлюз + агент, используя локальный CLI-бэкенд, не затрагивая вашу конфигурацию по умолчанию.
-   Включить:
    -   `pnpm test:live` (или `OPENCLAW_LIVE_TEST=1` при прямом вызове Vitest)
    -   `OPENCLAW_LIVE_CLI_BACKEND=1`
-   Значения по умолчанию:
    -   Модель: `claude-cli/claude-sonnet-4-6`
    -   Команда: `claude`
    -   Аргументы: `["-p","--output-format","json","--permission-mode","bypassPermissions"]`
-   Переопределения (опционально):
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-6"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.4"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` для отправки реального вложения изображения (пути внедряются в промпт).
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` для передачи путей к файлам изображений как аргументов CLI вместо внедрения в промпт.
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (или `"list"`) для управления передачей аргументов изображения, когда установлен `IMAGE_ARG`.
    -   `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` для отправки второго хода и проверки потока возобновления.
-   `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` чтобы оставить конфигурацию MCP Claude Code CLI включённой (по умолчанию отключает конфигурацию MCP с помощью временного пустого файла).

Пример:

```
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-6" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

### Рекомендуемые live-рецепты

Узкие, явные allowlist'ы самые быстрые и наименее хрупкие:

-   Одна модель, прямая (без шлюза):
    -   `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`
-   Одна модель, smoke-тест шлюза:
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   Tool calling для нескольких провайдеров:
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   Фокус на Google (Gemini API key + Antigravity):
    -   Gemini (API key): `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
    -   Antigravity (OAuth): `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

Примечания:

-   `google/...` использует Gemini API (API key).
-   `google-antigravity/...` использует мост Antigravity OAuth (агентская конечная точка в стиле Cloud Code Assist).
-   `google-gemini-cli/...` использует локальный Gemini CLI на вашей машине (отдельная аутентификация + особенности инструментов).
-   Gemini API vs Gemini CLI:
    -   API: OpenClaw вызывает размещённый Gemini API Google по HTTP (API key / аутентификация профиля); это то, что большинство пользователей подразумевает под «Gemini».
    -   CLI: OpenClaw запускает локальный бинарник `gemini`; у него своя аутентификация и может вести себя иначе (поддержка стриминга/инструментов/расхождение версий).

## Live: матрица моделей (что мы покрываем)

Нет фиксированного «списка моделей для CI» (live — опциональный), но это **рекомендуемые** модели для регулярного покрытия на машине разработчика с ключами.

### Современный smoke-набор (tool calling + изображения)

Это «распространённые модели», которые мы ожидаем держать в рабочем состоянии:

-   OpenAI (не-Codex): `openai/gpt-5.2` (опционально: `openai/gpt-5.1`)
-   OpenAI Codex: `openai-codex/gpt-5.4`
-   Anthropic: `anthropic/claude-opus-4-6` (или `anthropic/claude-sonnet-4-5`)
-   Google (Gemini API): `google/gemini-3-pro-preview` и `google/gemini-3-flash-preview` (избегайте старых моделей Gemini 2.x)
-   Google (Antigravity): `google-antigravity/claude-opus-4-6-thinking` и `google-antigravity/gemini-3-flash`
-   Z.AI (GLM): `zai/glm-4.7`
-   MiniMax: `minimax/minimax-m2.5`

Запустите smoke-тест шлюза с инструментами + изображениями: `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.4,anthropic/claude-opus-4-6,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

### Базовый уровень: tool calling (Read + опционально Exec)

Выберите хотя бы одну на семейство провайдеров:

-   OpenAI: `openai/gpt-5.2` (или `openai/gpt-5-mini`)
-   Anthropic: `anthropic/claude-opus-4-6` (или `anthropic/claude-sonnet-4-5`)
-   Google: `google/gemini-3-flash-preview` (или `google/gemini-3-pro-preview`)
-   Z.AI (GLM): `zai/glm-4.7`
-   MiniMax: `minimax/minimax-m2.5`

Опциональное дополнительное покрытие (желательно иметь):

-   xAI: `xai/grok-4` (или последняя доступная)
-   Mistral: `mistral/`… (выберите одну модель с поддержкой «tools», которая у вас включена)
-   Cerebras: `cerebras/`… (если есть доступ)
-   LM Studio: `lmstudio/`… (локальная; tool calling зависит от режима API)

### Vision: отправка изображений (вложение → мультимодальное сообщение)

Включите хотя бы одну модель с поддержкой изображений в `OPENCLAW_LIVE_GATEWAY_MODELS` (Claude/Gemini/OpenAI vision-capable варианты и т.д.), чтобы проверить image probe.

### Агрегаторы / альтернативные шлюзы

Если у вас включены ключи, мы также поддерживаем тестирование через:

-   OpenRouter: `openrouter/...` (сотни моделей; используйте `openclaw models scan`, чтобы найти кандидатов с поддержкой инструментов+изображений)
-   OpenCode Zen: `opencode/...` (аутентификация через `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

Больше провайдеров, которых можно включить в live-матрицу (если есть учётные данные/конфигурация):

-   Встроенные: `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
-   Через `models.providers` (пользовательские конечные точки): `minimax` (облако/API), плюс любой OpenAI/Anthropic-совместимый прокси (LM Studio, vLLM, LiteLLM и т.д.)

Совет: не пытайтесь жёстко прописать «все модели» в документации. Авторитетный список — это то, что возвращает `discoverModels(...)` на вашей машине + какие ключи доступны.

## Учётные данные (никогда не коммитить)

Live-тесты обнаруживают учётные данные так же, как это делает CLI. Практические следствия:

-   Если CLI работает, live-тесты должны найти те же ключи.
-   Если live-тест говорит «нет учётных данных», отлаживайте так же, как отлаживали бы `openclaw models list` / выбор модели.
-   Хранилище профилей: `~/.openclaw/credentials/` (предпочтительно; что означает «ключи профиля» в тестах)
-   Конфиг: `~/.openclaw/openclaw.json` (или `OPENCLAW_CONFIG_PATH`)

Если вы хотите полагаться на env-ключи (например, экспортированные в вашем `~/.profile`), запускайте локальные тесты после `source ~/.profile` или используйте Docker-раннеры ниже (они могут монтировать `~/.profile` в контейнер).

## Deepgram live (аудио транскрипция)

-   Тест: `src/media-understanding/providers/deepgram/audio.live.test.ts`
-   Включить: `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

## BytePlus coding plan live

-   Тест: `src/agents/byteplus.live.test.ts`
-   Включить: `BYTEPLUS_API_KEY=... BYTEPLUS_LIVE_TEST=1 pnpm test:live src/agents/byteplus.live.test.ts`
-   Опциональное переопределение модели: `BYTEPLUS_CODING_MODEL=ark-code-latest`

## Docker-раннеры (опциональные проверки «работает в Linux»)

Они запускают `pnpm test:live` внутри Docker-образа репозитория, монтируя вашу локальную директорию конфигурации и рабочее пространство (и загружая `~/.profile`, если смонтирован):

-   Прямые модели: `pnpm test:docker:live-models` (скрипт: `scripts/test-live-models-docker.sh`)
-   Шлюз + dev-агент: `pnpm test:docker:live-gateway` (скрипт: `scripts/test-live-gateway-models-docker.sh`)
-   Мастер онбординга (TTY, полное создание каркаса): `pnpm test:docker:onboard` (скрипт: `scripts/e2e/onboard-docker.sh`)
-   Сетевое взаимодействие шлюза (два контейнера, WS auth + health): `pnpm test:docker:gateway-network` (скрипт: `scripts/e2e/gateway-network-docker.sh`)
-   Плагины (загрузка пользовательских расширений + smoke-тест реестра): `pnpm test:docker:plugins` (скрипт: `scripts/e2e/plugins-docker.sh`)

Ручной smoke-тест простого языка ACP (не CI):

-   `bun scripts/dev/discord-acp-plain-language-smoke.ts --channel <discord-channel-id> ...`
-   Сохраните этот скрипт для рабочих процессов регрессии/отладки. Он может снова понадобиться для валидации маршрутизации тредов ACP, поэтому не удаляйте его.

Полезные переменные окружения:

-   `OPENCLAW_CONFIG_DIR=...` (по умолчанию: `~/.openclaw`) монтируется в `/home/node/.openclaw`
-   `OPENCLAW_WORKSPACE_DIR=...` (по умолчанию: `~/.openclaw/workspace`) монтируется в `/home/node/.openclaw/workspace`
-   `OPENCLAW_PROFILE_FILE=...` (по умолчанию: `~/.profile`) монтируется в `/home/node/.profile` и загружается перед запуском тестов
-   `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` для сужения запуска
-   `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` чтобы гарантировать, что учётные данные берутся из хранилища профилей (не из env)

## Проверка документации

Запускайте проверки документации после правок: `pnpm docs:list`.

## Офлайн-регрессии (безопасные для CI)

Это «реальные пайплайновые» регрессии без реальных провайдеров:

-   Tool calling шлюза (mock OpenAI, реальный шлюз + цикл агента): `src/gateway/gateway.test.ts` (кейс: «запускает mock OpenAI tool call сквозным образом через цикл агента шлюза»)
-   Мастер шлюза (WS `wizard.start`/`wizard.next`, запись конфига + принудительная аутентификация): `src/gateway/gateway.test.ts` (кейс: «запускает мастер по ws и записывает конфиг токена аутентификации»)

## Оценки надёжности агента (навыки)

У нас уже есть несколько безопасных для CI тестов, которые ведут себя как «оценки надёжности агента»:

-   Mock tool-calling через реальный шлюз + цикл агента (`src/gateway/gateway.test.ts`).
-   Сквозные потоки мастера, которые проверяют подключение сессий и эффекты конфигурации (`src/gateway/gateway.test.ts`).

Что всё ещё отсутствует для навыков (см. [Skills](../tools/skills.md)):

-   **Принятие решений:** когда навыки перечислены в промпте, выбирает ли агент правильный навык (или избегает нерелевантных)?
-   **Соответствие:** читает ли агент `SKILL.md` перед использованием и следует ли требуемым шагам/аргументам?
-   **Контракты рабочих процессов:** многопользовательские сценарии, которые проверяют порядок инструментов, перенос истории сессий и границы песочницы.

Будущие оценки должны оставаться в первую очередь детерминированными:

-   Запускатель сценариев, использующий mock-провайдеры для проверки вызовов инструментов + порядка, чтения файлов навыков и подключения сессий.
-   Небольшой набор сценариев, сфокусированных на навыках (использование vs избегание, контроль, внедрение промпта).
-   Опциональные live-оценки (опциональные, управляемые env) только после того, как безопасный для CI набор будет на месте.

## Добавление регрессий (руководство)

Когда вы исправляете проблему провайдера/модели, обнаруженную в live:

-   Добавьте безопасную для CI регрессию, если возможно (mock/stub провайдера, или захватите точное преобразование формы запроса)
-   Если она по своей природе только live (rate limits, политики аутентификации), оставьте live-тест узким и опциональным через env-переменные
-   Предпочитайте нацеливаться на наименьший слой