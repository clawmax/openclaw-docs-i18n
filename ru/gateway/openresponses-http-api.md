title: "HTTP API OpenResponses для настройки и использования OpenClaw Gateway"
description: "Узнайте, как включить и использовать совместимую с OpenResponses конечную точку POST /v1/responses в OpenClaw Gateway. Настройте аутентификацию, безопасность, агентов, инструменты и работу с файлами."
keywords: ["api openresponses", "openclaw gateway", "http api", "интеграция агентов", "аутентификация api", "клиентские инструменты", "загрузка файлов", "стриминг sse"]
---

  Протоколы и API

  
# API OpenResponses

Шлюз OpenClaw может обслуживать совместимую с OpenResponses конечную точку `POST /v1/responses`. Эта конечная точка **по умолчанию отключена**. Сначала включите её в конфигурации.

-   `POST /v1/responses`
-   Тот же порт, что и у шлюза (мультиплексирование WS + HTTP): `http://<gateway-host>:/v1/responses`

Под капотом запросы выполняются как обычный запуск агента через шлюз (тот же путь выполнения, что и `openclaw agent`), поэтому маршрутизация/разрешения/конфигурация соответствуют вашему шлюзу.

## Аутентификация

Использует конфигурацию аутентификации шлюза. Отправляйте токен-носитель:

-   `Authorization: Bearer `

Примечания:

-   Когда `gateway.auth.mode="token"`, используйте `gateway.auth.token` (или `OPENCLAW_GATEWAY_TOKEN`).
-   Когда `gateway.auth.mode="password"`, используйте `gateway.auth.password` (или `OPENCLAW_GATEWAY_PASSWORD`).
-   Если настроен `gateway.auth.rateLimit` и происходит слишком много неудачных попыток аутентификации, конечная точка возвращает `429` с заголовком `Retry-After`.

## Граница безопасности (важно)

Рассматривайте эту конечную точку как **полноценную поверхность доступа оператора** для экземпляра шлюза.

-   HTTP-аутентификация с токеном-носителем здесь — это не узкая модель области действия для конкретного пользователя.
-   Действительный токен/пароль шлюза для этой конечной точки следует рассматривать как учётные данные владельца/оператора.
-   Запросы проходят по тому же пути агента плоскости управления, что и доверенные действия оператора.
-   Для этой конечной точки нет отдельной границы инструментов для непривилегированных пользователей; как только вызывающая сторона проходит аутентификацию шлюза здесь, OpenClaw рассматривает эту сторону как доверенного оператора для данного шлюза.
-   Если политика целевого агента разрешает использование чувствительных инструментов, эта конечная точка может их использовать.
-   Держите эту конечную точку только на loopback/tailnet/частном входе; не выставляйте её напрямую в публичный интернет.

См. [Безопасность](./security.md) и [Удалённый доступ](./remote.md).

## Выбор агента

Пользовательские заголовки не требуются: укажите идентификатор агента в поле OpenResponses `model`:

-   `model: "openclaw:"` (пример: `"openclaw:main"`, `"openclaw:beta"`)
-   `model: "agent:"` (псевдоним)

Или укажите конкретного агента OpenClaw через заголовок:

-   `x-openclaw-agent-id: ` (по умолчанию: `main`)

Для продвинутых сценариев:

-   `x-openclaw-session-key: ` для полного контроля над маршрутизацией сессии.

## Включение конечной точки

Установите `gateway.http.endpoints.responses.enabled` в `true`:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true },
      },
    },
  },
}
```

## Отключение конечной точки

Установите `gateway.http.endpoints.responses.enabled` в `false`:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false },
      },
    },
  },
}
```

## Поведение сессии

По умолчанию конечная точка **не сохраняет состояние между запросами** (для каждого вызова генерируется новый ключ сессии). Если запрос включает строку OpenResponses `user`, шлюз выводит стабильный ключ сессии из неё, чтобы повторные вызовы могли использовать одну сессию агента.

## Формат запроса (поддерживается)

Запрос следует API OpenResponses с вводом на основе элементов. Текущая поддержка:

-   `input`: строка или массив объектов элементов.
-   `instructions`: добавляются в системный промпт.
-   `tools`: определения клиентских инструментов (инструменты-функции).
-   `tool_choice`: фильтрация или требование использования клиентских инструментов.
-   `stream`: включает потоковую передачу SSE.
-   `max_output_tokens`: ограничение на вывод по возможности (зависит от провайдера).
-   `user`: стабильная маршрутизация сессии.

Принимаются, но **в настоящее время игнорируются**:

-   `max_tool_calls`
-   `reasoning`
-   `metadata`
-   `store`
-   `previous_response_id`
-   `truncation`

## Элементы (input)

### message

Роли: `system`, `developer`, `user`, `assistant`.

-   `system` и `developer` добавляются в системный промпт.
-   Самый последний элемент `user` или `function_call_output` становится «текущим сообщением».
-   Более ранние сообщения пользователя/ассистента включаются в историю для контекста.

### function\_call\_output (пошаговые инструменты)

Отправляйте результаты работы инструментов обратно модели:

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```

### reasoning и item\_reference

Принимаются для совместимости со схемой, но игнорируются при построении промпта.

## Инструменты (клиентские инструменты-функции)

Предоставляйте инструменты с помощью `tools: [{ type: "function", function: { name, description?, parameters? } }]`. Если агент решит вызвать инструмент, в ответе будет возвращён элемент вывода `function_call`. Затем вы отправляете последующий запрос с `function_call_output`, чтобы продолжить ход.

## Изображения (input\_image)

Поддерживает источники base64 или URL:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Разрешённые MIME-типы (текущие): `image/jpeg`, `image/png`, `image/gif`, `image/webp`, `image/heic`, `image/heif`. Максимальный размер (текущий): 10 МБ.

## Файлы (input\_file)

Поддерживает источники base64 или URL:

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

Разрешённые MIME-типы (текущие): `text/plain`, `text/markdown`, `text/html`, `text/csv`, `application/json`, `application/pdf`. Максимальный размер (текущий): 5 МБ. Текущее поведение:

-   Содержимое файла декодируется и добавляется в **системный промпт**, а не в сообщение пользователя, поэтому остаётся эфемерным (не сохраняется в истории сессии).
-   PDF-файлы парсятся для извлечения текста. Если текста найдено мало, первые страницы растеризуются в изображения и передаются модели.

Для парсинга PDF используется Node-дружественная legacy-сборка `pdfjs-dist` (без воркера). Современная сборка PDF.js ожидает браузерных воркеров/DOM-глобальных объектов, поэтому не используется в шлюзе. Настройки загрузки по URL по умолчанию:

-   `files.allowUrl`: `true`
-   `images.allowUrl`: `true`
-   `maxUrlParts`: `8` (общее количество частей `input_file` + `input_image` на основе URL за запрос)
-   Запросы защищены (разрешение DNS, блокировка приватных IP, ограничение редиректов, таймауты).
-   Поддерживаются опциональные списки разрешённых хостов для каждого типа ввода (`files.urlAllowlist`, `images.urlAllowlist`).
    -   Точное имя хоста: `"cdn.example.com"`
    -   Wildcard для поддоменов: `"*.assets.example.com"` (не соответствует apex-домену)

## Ограничения для файлов и изображений (конфигурация)

Значения по умолчанию можно настроить в разделе `gateway.http.endpoints.responses`:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          maxUrlParts: 8,
          files: {
            allowUrl: true,
            urlAllowlist: ["cdn.example.com", "*.assets.example.com"],
            allowedMimes: [
              "text/plain",
              "text/markdown",
              "text/html",
              "text/csv",
              "application/json",
              "application/pdf",
            ],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200,
            },
          },
          images: {
            allowUrl: true,
            urlAllowlist: ["images.example.com"],
            allowedMimes: [
              "image/jpeg",
              "image/png",
              "image/gif",
              "image/webp",
              "image/heic",
              "image/heif",
            ],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000,
          },
        },
      },
    },
  },
}
```

Значения по умолчанию, если опущены:

-   `maxBodyBytes`: 20 МБ
-   `maxUrlParts`: 8
-   `files.maxBytes`: 5 МБ
-   `files.maxChars`: 200 тыс.
-   `files.maxRedirects`: 3
-   `files.timeoutMs`: 10 с
-   `files.pdf.maxPages`: 4
-   `files.pdf.maxPixels`: 4 000 000
-   `files.pdf.minTextChars`: 200
-   `images.maxBytes`: 10 МБ
-   `images.maxRedirects`: 3
-   `images.timeoutMs`: 10 с
-   Источники `input_image` в форматах HEIC/HEIF принимаются и нормализуются в JPEG перед отправкой провайдеру.

Примечание по безопасности:

-   Списки разрешённых URL применяются до загрузки и на каждом шаге редиректа.
-   Добавление имени хоста в белый список не отменяет блокировку приватных/внутренних IP.
-   Для шлюзов, доступных из интернета, применяйте сетевые средства контроля исходящего трафика в дополнение к защите на уровне приложения. См. [Безопасность](./security.md).

## Потоковая передача (SSE)

Установите `stream: true` для получения событий, отправляемых сервером (SSE):

-   `Content-Type: text/event-stream`
-   Каждая строка события имеет вид `event: ` и `data: `
-   Поток завершается строкой `data: [DONE]`

Типы событий, отправляемые в настоящее время:

-   `response.created`
-   `response.in_progress`
-   `response.output_item.added`
-   `response.content_part.added`
-   `response.output_text.delta`
-   `response.output_text.done`
-   `response.content_part.done`
-   `response.output_item.done`
-   `response.completed`
-   `response.failed` (при ошибке)

## Использование (usage)

`usage` заполняется, когда базовый провайдер сообщает количество токенов.

## Ошибки

Ошибки используют объект JSON вида:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

Распространённые случаи:

-   `401` отсутствует/недействительна аутентификация
-   `400` неверное тело запроса
-   `405` неверный метод

## Примеры

Без потоковой передачи:

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

С потоковой передачей:

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```

[OpenAI Chat Completions](./openai-http-api.md)[Tools Invoke API](./tools-invoke-http-api.md)