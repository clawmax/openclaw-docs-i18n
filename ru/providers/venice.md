

  Провайдеры

  
# Venice AI

**Venice** — это наша выделенная настройка Venice для приватного вывода с возможностью анонимизированного доступа к проприетарным моделям. Venice AI предоставляет AI-вывод с фокусом на конфиденциальность, поддержку нецензурируемых моделей и доступ к основным проприетарным моделям через их анонимизированный прокси. Весь вывод по умолчанию является приватным — ваши данные не используются для обучения, логирование отсутствует.

## Почему Venice в OpenClaw

-   **Приватный вывод** для моделей с открытым исходным кодом (без логирования).
-   **Нецензурируемые модели**, когда они нужны.
-   **Анонимизированный доступ** к проприетарным моделям (Opus/GPT/Gemini), когда важна качество.
-   Совместимые с OpenAI конечные точки `/v1`.

## Режимы конфиденциальности

Venice предлагает два уровня конфиденциальности — понимание этого ключевое для выбора модели:

| Режим | Описание | Модели |
| --- | --- | --- |
| **Приватный** | Полностью приватный. Промпты и ответы **никогда не сохраняются и не логируются**. Эфемерный. | Llama, Qwen, DeepSeek, Kimi, MiniMax, Venice Uncensored и т.д. |
| **Анонимизированный** | Проксируется через Venice с удалением метаданных. Базовый провайдер (OpenAI, Anthropic, Google, xAI) видит анонимизированные запросы. | Claude, GPT, Gemini, Grok |

## Возможности

-   **Фокус на конфиденциальность**: Выбор между режимами «приватный» (полностью приватный) и «анонимизированный» (проксированный)
-   **Нецензурируемые модели**: Доступ к моделям без ограничений по контенту
-   **Доступ к основным моделям**: Использование Claude, GPT, Gemini и Grok через анонимизированный прокси Venice
-   **Совместимый с OpenAI API**: Стандартные конечные точки `/v1` для легкой интеграции
-   **Стриминг**: ✅ Поддерживается для всех моделей
-   **Вызов функций**: ✅ Поддерживается для выбранных моделей (проверьте возможности модели)
-   **Зрение (Vision)**: ✅ Поддерживается для моделей с возможностью vision
-   **Нет жестких лимитов запросов**: Может применяться регулирование при экстремальном использовании

## Настройка

### 1\. Получите API-ключ

1.  Зарегистрируйтесь на [venice.ai](https://venice.ai)
2.  Перейдите в **Settings → API Keys → Create new key**
3.  Скопируйте ваш API-ключ (формат: `vapi_xxxxxxxxxxxx`)

### 2\. Настройте OpenClaw

**Вариант A: Переменная окружения**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**Вариант B: Интерактивная настройка (Рекомендуется)**

```bash
openclaw onboard --auth-choice venice-api-key
```

Это выполнит:

1.  Запрос вашего API-ключа (или использование существующего `VENICE_API_KEY`)
2.  Показ всех доступных моделей Venice
3.  Выбор вашей модели по умолчанию
4.  Автоматическую настройку провайдера

**Вариант C: Неинтерактивная настройка**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```

### 3\. Проверьте настройку

```bash
openclaw agent --model venice/kimi-k2-5 --message "Hello, are you working?"
```

## Выбор модели

После настройки OpenClaw показывает все доступные модели Venice. Выбирайте исходя из ваших потребностей:

-   **Модель по умолчанию**: `venice/kimi-k2-5` для сильного приватного рассуждения плюс vision.
-   **Опция высокой мощности**: `venice/claude-opus-4-6` для самого мощного анонимизированного пути через Venice.
-   **Конфиденциальность**: Выбирайте «приватные» модели для полностью приватного вывода.
-   **Возможности**: Выбирайте «анонимизированные» модели для доступа к Claude, GPT, Gemini через прокси Venice.

Измените модель по умолчанию в любое время:

```bash
openclaw models set venice/kimi-k2-5
openclaw models set venice/claude-opus-4-6
```

Вывести список всех доступных моделей:

```bash
openclaw models list | grep venice
```

## Настройка через openclaw configure

1.  Запустите `openclaw configure`
2.  Выберите **Model/auth**
3.  Выберите **Venice AI**

## Какую модель мне использовать?

| Сценарий использования | Рекомендуемая модель | Почему |
| --- | --- | --- |
| **Общий чат (по умолчанию)** | `kimi-k2-5` | Сильное приватное рассуждение плюс vision |
| **Лучшее общее качество** | `claude-opus-4-6` | Самый мощный анонимизированный вариант через Venice |
| **Конфиденциальность + программирование** | `qwen3-coder-480b-a35b-instruct` | Приватная модель для программирования с большим контекстом |
| **Приватное зрение (Vision)** | `kimi-k2-5` | Поддержка vision без выхода из приватного режима |
| **Быстрая + дешевая** | `qwen3-4b` | Легковесная модель для рассуждений |
| **Сложные приватные задачи** | `deepseek-v3.2` | Сильное рассуждение, но без поддержки инструментов Venice |
| **Нецензурируемая** | `venice-uncensored` | Без ограничений по контенту |

## Доступные модели (Всего 41)

### Приватные модели (26) — Полностью приватные, без логирования

| ID модели | Название | Контекст | Возможности |
| --- | --- | --- | --- |
| `kimi-k2-5` | Kimi K2.5 | 256k | По умолчанию, рассуждение, vision |
| `kimi-k2-thinking` | Kimi K2 Thinking | 256k | Рассуждение |
| `llama-3.3-70b` | Llama 3.3 70B | 128k | Общая |
| `llama-3.2-3b` | Llama 3.2 3B | 128k | Общая |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 128k | Общая, инструменты отключены |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 128k | Рассуждение |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 128k | Общая |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 256k | Программирование |
| `qwen3-coder-480b-a35b-instruct-turbo` | Qwen3 Coder 480B Turbo | 256k | Программирование |
| `qwen3-5-35b-a3b` | Qwen3.5 35B A3B | 256k | Рассуждение, vision |
| `qwen3-next-80b` | Qwen3 Next 80B | 256k | Общая |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B (Vision) | 256k | Vision |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | Быстрая, рассуждение |
| `deepseek-v3.2` | DeepSeek V3.2 | 160k | Рассуждение, инструменты отключены |
| `venice-uncensored` | Venice Uncensored (Dolphin-Mistral) | 32k | Нецензурируемая, инструменты отключены |
| `mistral-31-24b` | Venice Medium (Mistral) | 128k | Vision |
| `google-gemma-3-27b-it` | Google Gemma 3 27B Instruct | 198k | Vision |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 128k | Общая |
| `nvidia-nemotron-3-nano-30b-a3b` | NVIDIA Nemotron 3 Nano 30B | 128k | Общая |
| `olafangensan-glm-4.7-flash-heretic` | GLM 4.7 Flash Heretic | 128k | Рассуждение |
| `zai-org-glm-4.6` | GLM 4.6 | 198k | Общая |
| `zai-org-glm-4.7` | GLM 4.7 | 198k | Рассуждение |
| `zai-org-glm-4.7-flash` | GLM 4.7 Flash | 128k | Рассуждение |
| `zai-org-glm-5` | GLM 5 | 198k | Рассуждение |
| `minimax-m21` | MiniMax M2.1 | 198k | Рассуждение |
| `minimax-m25` | MiniMax M2.5 | 198k | Рассуждение |

### Анонимизированные модели (15) — Через прокси Venice

| ID модели | Название | Контекст | Возможности |
| --- | --- | --- | --- |
| `claude-opus-4-6` | Claude Opus 4.6 (через Venice) | 1M | Рассуждение, vision |
| `claude-opus-4-5` | Claude Opus 4.5 (через Venice) | 198k | Рассуждение, vision |
| `claude-sonnet-4-6` | Claude Sonnet 4.6 (через Venice) | 1M | Рассуждение, vision |
| `claude-sonnet-4-5` | Claude Sonnet 4.5 (через Venice) | 198k | Рассуждение, vision |
| `openai-gpt-54` | GPT-5.4 (через Venice) | 1M | Рассуждение, vision |
| `openai-gpt-53-codex` | GPT-5.3 Codex (через Venice) | 400k | Рассуждение, vision, программирование |
| `openai-gpt-52` | GPT-5.2 (через Venice) | 256k | Рассуждение |
| `openai-gpt-52-codex` | GPT-5.2 Codex (через Venice) | 256k | Рассуждение, vision, программирование |
| `openai-gpt-4o-2024-11-20` | GPT-4o (через Venice) | 128k | Vision |
| `openai-gpt-4o-mini-2024-07-18` | GPT-4o Mini (через Venice) | 128k | Vision |
| `gemini-3-1-pro-preview` | Gemini 3.1 Pro (через Venice) | 1M | Рассуждение, vision |
| `gemini-3-pro-preview` | Gemini 3 Pro (через Venice) | 198k | Рассуждение, vision |
| `gemini-3-flash-preview` | Gemini 3 Flash (через Venice) | 256k | Рассуждение, vision |
| `grok-41-fast` | Grok 4.1 Fast (через Venice) | 1M | Рассуждение, vision |
| `grok-code-fast-1` | Grok Code Fast 1 (через Venice) | 256k | Рассуждение, программирование |

## Обнаружение моделей

OpenClaw автоматически обнаруживает модели из Venice API, когда установлена переменная `VENICE_API_KEY`. Если API недоступен, используется статический каталог. Конечная точка `/models` является публичной (для получения списка аутентификация не требуется), но для вывода нужен действительный API-ключ.

## Поддержка стриминга и инструментов

| Возможность | Поддержка |
| --- | --- |
| **Стриминг** | ✅ Все модели |
| **Вызов функций** | ✅ Большинство моделей (проверьте `supportsFunctionCalling` в API) |
| **Зрение/Изображения (Vision)** | ✅ Модели, отмеченные возможностью “Vision” |
| **Режим JSON** | ✅ Поддерживается через `response_format` |

## Ценообразование

Venice использует кредитную систему. Актуальные тарифы смотрите на [venice.ai/pricing](https://venice.ai/pricing):

-   **Приватные модели**: Как правило, ниже стоимость
-   **Анонимизированные модели**: Аналогично ценам прямого API + небольшая комиссия Venice

## Сравнение: Venice vs Прямой API

| Аспект | Venice (Анонимизированный) | Прямой API |
| --- | --- | --- |
| **Конфиденциальность** | Метаданные удалены, анонимизирован | Ваш аккаунт привязан |
| **Задержка** | +10-50мс (прокси) | Прямая |
| **Возможности** | Большинство функций поддерживается | Полный набор функций |
| **Биллинг** | Кредиты Venice | Биллинг провайдера |

## Примеры использования

```bash
# Использовать приватную модель по умолчанию
openclaw agent --model venice/kimi-k2-5 --message "Quick health check"

# Использовать Claude Opus через Venice (анонимизированный)
openclaw agent --model venice/claude-opus-4-6 --message "Summarize this task"

# Использовать нецензурируемую модель
openclaw agent --model venice/venice-uncensored --message "Draft options"

# Использовать модель с vision с изображением
openclaw agent --model venice/qwen3-vl-235b-a22b --message "Review attached image"

# Использовать модель для программирования
openclaw agent --model venice/qwen3-coder-480b-a35b-instruct --message "Refactor this function"
```

## Устранение неполадок

### API-ключ не распознается

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

Убедитесь, что ключ начинается с `vapi_`.

### Модель недоступна

Каталог моделей Venice обновляется динамически. Запустите `openclaw models list`, чтобы увидеть текущие доступные модели. Некоторые модели могут быть временно недоступны.

### Проблемы с подключением

API Venice находится по адресу `https://api.venice.ai/api/v1`. Убедитесь, что ваша сеть разрешает HTTPS-соединения.

## Пример конфигурационного файла

```json
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/kimi-k2-5" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2-5",
            name: "Kimi K2.5",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

## Ссылки

-   [Venice AI](https://venice.ai)
-   [Документация API](https://docs.venice.ai)
-   [Ценообразование](https://venice.ai/pricing)
-   [Статус](https://status.venice.ai)

[Vercel AI Gateway](./vercel-ai-gateway.md)[vLLM](./vllm.md)