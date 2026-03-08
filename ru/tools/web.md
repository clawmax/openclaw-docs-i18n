

  Встроенные инструменты

  
# Веб-инструменты

OpenClaw включает два легковесных веб-инструмента:

-   `web_search` — Поиск в интернете с использованием Perplexity Search API, Brave Search API, Gemini с Google Search grounding, Grok или Kimi.
-   `web_fetch` — HTTP-запрос + извлечение читаемого контента (HTML → markdown/текст).

Это **не** автоматизация браузера. Для сайтов с интенсивным использованием JavaScript или логинов используйте [Инструмент Browser](./browser.md).

## Как это работает

-   `web_search` обращается к вашему настроенному провайдеру и возвращает результаты.
-   Результаты кэшируются по запросу на 15 минут (настраивается).
-   `web_fetch` выполняет простой HTTP GET и извлекает читаемый контент (HTML → markdown/текст). Он **не** выполняет JavaScript.
-   `web_fetch` включен по умолчанию (если явно не отключен).

Подробности по настройке конкретных провайдеров см. в [настройке Perplexity Search](../perplexity.md) и [настройке Brave Search](../brave-search.md).

## Выбор поискового провайдера

| Провайдер | Плюсы | Минусы | Ключ API |
| --- | --- | --- | --- |
| **Perplexity Search API** | Быстрые, структурированные результаты; фильтры по домену, языку, региону и свежести; извлечение контента | — | `PERPLEXITY_API_KEY` |
| **Brave Search API** | Быстрые, структурированные результаты | Меньше опций фильтрации; применяются условия использования для ИИ | `BRAVE_API_KEY` |
| **Gemini** | Google Search grounding, ответы, синтезированные ИИ | Требуется ключ API Gemini | `GEMINI_API_KEY` |
| **Grok** | Веб-заземленные ответы от xAI | Требуется ключ API xAI | `XAI_API_KEY` |
| **Kimi** | Возможность веб-поиска от Moonshot | Требуется ключ API Moonshot | `KIMI_API_KEY` / `MOONSHOT_API_KEY` |

### Автоопределение

Если `provider` не задан явно, OpenClaw автоматически определяет, какой провайдер использовать, на основе доступных ключей API, проверяя в следующем порядке:

1.  **Brave** — переменная окружения `BRAVE_API_KEY` или конфиг `tools.web.search.apiKey`
2.  **Gemini** — переменная окружения `GEMINI_API_KEY` или конфиг `tools.web.search.gemini.apiKey`
3.  **Kimi** — переменная окружения `KIMI_API_KEY` / `MOONSHOT_API_KEY` или конфиг `tools.web.search.kimi.apiKey`
4.  **Perplexity** — переменная окружения `PERPLEXITY_API_KEY` или конфиг `tools.web.search.perplexity.apiKey`
5.  **Grok** — переменная окружения `XAI_API_KEY` или конфиг `tools.web.search.grok.apiKey`

Если ключи не найдены, происходит откат к Brave (вы получите ошибку об отсутствии ключа с предложением настроить его).

## Настройка веб-поиска

Используйте `openclaw configure --section web`, чтобы настроить ваш ключ API и выбрать провайдера.

### Perplexity Search

1.  Создайте аккаунт Perplexity на [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api)
2.  Сгенерируйте ключ API в панели управления
3.  Запустите `openclaw configure --section web`, чтобы сохранить ключ в конфигурации, или установите переменную окружения `PERPLEXITY_API_KEY`.

Подробнее см. в [документации Perplexity Search API](https://docs.perplexity.ai/guides/search-quickstart).

### Brave Search

1.  Создайте аккаунт Brave Search API на [brave.com/search/api](https://brave.com/search/api/)
2.  В панели управления выберите план **Data for Search** (не "Data for AI") и сгенерируйте ключ API.
3.  Запустите `openclaw configure --section web`, чтобы сохранить ключ в конфигурации (рекомендуется), или установите переменную окружения `BRAVE_API_KEY`.

Brave предоставляет платные тарифы; проверьте портал Brave API для получения информации о текущих лимитах и ценах.

### Где хранить ключ

**Через конфигурацию (рекомендуется):** запустите `openclaw configure --section web`. Ключ сохранится в `tools.web.search.perplexity.apiKey` или `tools.web.search.apiKey`. **Через окружение:** установите переменную окружения `PERPLEXITY_API_KEY` или `BRAVE_API_KEY` в окружении процесса Gateway. Для установки шлюза поместите его в `~/.openclaw/.env` (или в окружение вашей службы). См. [Переменные окружения](../help/faq.md#how-does-openclaw-load-environment-variables).

### Примеры конфигурации

**Perplexity Search:**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...", // опционально, если задана PERPLEXITY_API_KEY
        },
      },
    },
  },
}
```

**Brave Search:**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "brave",
        apiKey: "YOUR_BRAVE_API_KEY", // опционально, если задана BRAVE_API_KEY // pragma: allowlist secret
      },
    },
  },
}
```

## Использование Gemini (Google Search grounding)

Модели Gemini поддерживают встроенный [Google Search grounding](https://ai.google.dev/gemini-api/docs/grounding), который возвращает ответы, синтезированные ИИ, основанные на результатах живого поиска Google с цитированием источников.

### Получение ключа API Gemini

1.  Перейдите в [Google AI Studio](https://aistudio.google.com/apikey)
2.  Создайте ключ API
3.  Установите переменную окружения `GEMINI_API_KEY` в окружении Gateway или настройте `tools.web.search.gemini.apiKey`

### Настройка поиска Gemini

```json
{
  tools: {
    web: {
      search: {
        provider: "gemini",
        gemini: {
          // Ключ API (опционально, если задана GEMINI_API_KEY)
          apiKey: "AIza...",
          // Модель (по умолчанию "gemini-2.5-flash")
          model: "gemini-2.5-flash",
        },
      },
    },
  },
}
```

**Альтернатива через окружение:** установите переменную окружения `GEMINI_API_KEY` в окружении Gateway. Для установки шлюза поместите его в `~/.openclaw/.env`.

### Примечания

-   URL-адреса цитирования из Gemini grounding автоматически преобразуются из редиректов Google в прямые URL.
-   Преобразование редиректов использует путь защиты от SSRF (HEAD + проверка редиректов + валидация http/https) перед возвратом окончательного URL для цитирования.
-   Преобразование редиректов использует строгие настройки SSRF по умолчанию, поэтому редиректы на частные/внутренние цели блокируются.
-   Модель по умолчанию (`gemini-2.5-flash`) быстрая и экономичная. Можно использовать любую модель Gemini, поддерживающую grounding.

## web\_search

Поиск в интернете с использованием вашего настроенного провайдера.

### Требования

-   `tools.web.search.enabled` не должно быть `false` (по умолчанию: включено)
-   Ключ API для выбранного провайдера:
    -   **Brave**: `BRAVE_API_KEY` или `tools.web.search.apiKey`
    -   **Perplexity**: `PERPLEXITY_API_KEY` или `tools.web.search.perplexity.apiKey`
    -   **Gemini**: `GEMINI_API_KEY` или `tools.web.search.gemini.apiKey`
    -   **Grok**: `XAI_API_KEY` или `tools.web.search.grok.apiKey`
    -   **Kimi**: `KIMI_API_KEY`, `MOONSHOT_API_KEY` или `tools.web.search.kimi.apiKey`

### Конфигурация

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // опционально, если задана BRAVE_API_KEY
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### Параметры инструмента

Все параметры работают как для Brave, так и для Perplexity, если не указано иное.

| Параметр | Описание |
| --- | --- |
| `query` | Поисковый запрос (обязательный) |
| `count` | Количество возвращаемых результатов (1-10, по умолчанию: 5) |
| `country` | 2-буквенный код страны ISO (например, "US", "DE") |
| `language` | Код языка ISO 639-1 (например, "en", "de") |
| `freshness` | Фильтр по времени: `day`, `week`, `month` или `year` |
| `date_after` | Результаты после этой даты (ГГГГ-ММ-ДД) |
| `date_before` | Результаты до этой даты (ГГГГ-ММ-ДД) |
| `ui_lang` | Код языка интерфейса (только Brave) |
| `domain_filter` | Массив разрешенных/запрещенных доменов (только Perplexity) |
| `max_tokens` | Общий бюджет на контент, по умолчанию 25000 (только Perplexity) |
| `max_tokens_per_page` | Лимит токенов на страницу, по умолчанию 2048 (только Perplexity) |

**Примеры:**

```
// Поиск с учетом немецкого региона
await web_search({
  query: "TV online schauen",
  country: "DE",
  language: "de",
});

// Недавние результаты (за последнюю неделю)
await web_search({
  query: "TMBG interview",
  freshness: "week",
});

// Поиск по диапазону дат
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});

// Фильтрация по доменам (только Perplexity)
await web_search({
  query: "climate research",
  domain_filter: ["nature.com", "science.org", ".edu"],
});

// Исключение доменов (только Perplexity)
await web_search({
  query: "product reviews",
  domain_filter: ["-reddit.com", "-pinterest.com"],
});

// Больше извлечения контента (только Perplexity)
await web_search({
  query: "detailed AI research",
  max_tokens: 50000,
  max_tokens_per_page: 4096,
});
```

## web\_fetch

Получить URL и извлечь читаемый контент.

### Требования для web\_fetch

-   `tools.web.fetch.enabled` не должно быть `false` (по умолчанию: включено)
-   Опциональный резервный вариант Firecrawl: установите `tools.web.fetch.firecrawl.apiKey` или `FIRECRAWL_API_KEY`.

### Конфигурация web\_fetch

```json
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // опционально, если задана FIRECRAWL_API_KEY
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // мс (1 день)
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### Параметры инструмента web\_fetch

-   `url` (обязательный, только http/https)
-   `extractMode` (`markdown` | `text`)
-   `maxChars` (обрезать длинные страницы)

Примечания:

-   `web_fetch` сначала использует Readability (извлечение основного контента), затем Firecrawl (если настроен). Если оба метода не сработают, инструмент вернет ошибку.
-   Запросы Firecrawl используют режим обхода ботов и по умолчанию кэшируют результаты.
-   `web_fetch` по умолчанию отправляет User-Agent, похожий на Chrome, и `Accept-Language`; переопределите `userAgent`, если нужно.
-   `web_fetch` блокирует частные/внутренние имена хостов и повторно проверяет редиректы (ограничение `maxRedirects`).
-   `maxChars` ограничивается значением `tools.web.fetch.maxCharsCap`.
-   `web_fetch` ограничивает размер загружаемого тела ответа до `tools.web.fetch.maxResponseBytes` перед парсингом; слишком большие ответы обрезаются и содержат предупреждение.
-   `web_fetch` — это извлечение по принципу "best-effort"; для некоторых сайтов потребуется инструмент браузера.
-   См. [Firecrawl](./firecrawl.md) для получения информации о настройке ключа и деталях сервиса.
-   Ответы кэшируются (по умолчанию 15 минут), чтобы уменьшить количество повторных запросов.
-   Если вы используете профили/списки разрешенных инструментов, добавьте `web_search`/`web_fetch` или `group:web`.
-   Если ключ API отсутствует, `web_search` возвращает краткую подсказку по настройке со ссылкой на документацию.

[Уровни мышления](./thinking.md)[Браузер (управляемый OpenClaw)](./browser.md)