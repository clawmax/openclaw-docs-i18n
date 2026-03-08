title: "План внедрения OpenResponses Gateway для экспериментов OpenClaw"
description: "Узнайте о плане реализации конечной точки API OpenResponses в OpenClaw Gateway, включая архитектуру, валидацию и путь к устареванию для Chat Completions."
keywords: ["openresponses", "gateway", "api endpoint", "совместимость с openai", "стриминг", "эксперименты", "агентные рабочие процессы", "стандарт инференса"]
---

  Эксперименты

  
# План внедрения OpenResponses Gateway

## Контекст

OpenClaw Gateway в настоящее время предоставляет минимально совместимую с OpenAI конечную точку Chat Completions по адресу `/v1/chat/completions` (см. [OpenAI Chat Completions](../../gateway/openai-http-api.md)). Open Responses — это открытый стандарт инференса, основанный на OpenAI Responses API. Он разработан для агентных рабочих процессов и использует ввод на основе элементов (item-based inputs) и семантические стриминговые события. Спецификация OpenResponses определяет `/v1/responses`, а не `/v1/chat/completions`.

## Цели

-   Добавить конечную точку `/v1/responses`, соответствующую семантике OpenResponses.
-   Сохранить Chat Completions в качестве слоя совместимости, который легко отключить и в конечном итоге удалить.
-   Стандартизировать валидацию и парсинг с помощью изолированных, переиспользуемых схем.

## Нецели

-   Полная функциональная эквивалентность OpenResponses в первой реализации (изображения, файлы, размещённые инструменты).
-   Замена внутренней логики выполнения агентов или оркестрации инструментов.
-   Изменение поведения существующей конечной точки `/v1/chat/completions` на первом этапе.

## Краткое описание исследования

Источники: OpenResponses OpenAPI, сайт спецификации OpenResponses и публикация в блоге Hugging Face. Ключевые моменты:

-   `POST /v1/responses` принимает поля `CreateResponseBody`, такие как `model`, `input` (строка или `ItemParam[]`), `instructions`, `tools`, `tool_choice`, `stream`, `max_output_tokens` и `max_tool_calls`.
-   `ItemParam` — это дискриминированное объединение (discriminated union):
    -   элементы `message` с ролями `system`, `developer`, `user`, `assistant`
    -   `function_call` и `function_call_output`
    -   `reasoning`
    -   `item_reference`
-   Успешные ответы возвращают `ResponseResource` с полями `object: "response"`, `status` и элементами `output`.
-   Стриминг использует семантические события, такие как:
    -   `response.created`, `response.in_progress`, `response.completed`, `response.failed`
    -   `response.output_item.added`, `response.output_item.done`
    -   `response.content_part.added`, `response.content_part.done`
    -   `response.output_text.delta`, `response.output_text.done`
-   Спецификация требует:
    -   `Content-Type: text/event-stream`
    -   `event:` должен соответствовать полю JSON `type`
    -   терминальным событием должна быть строка `[DONE]`
-   Элементы reasoning (рассуждения) могут содержать `content`, `encrypted_content` и `summary`.
-   Примеры от HF включают `OpenResponses-Version: latest` в запросах (необязательный заголовок).

## Предлагаемая архитектура

-   Добавить `src/gateway/open-responses.schema.ts`, содержащий только схемы Zod (без импортов из gateway).
-   Добавить `src/gateway/openresponses-http.ts` (или `open-responses-http.ts`) для `/v1/responses`.
-   Оставить `src/gateway/openai-http.ts` без изменений в качестве адаптера для обратной совместимости.
-   Добавить конфигурацию `gateway.http.endpoints.responses.enabled` (по умолчанию `false`).
-   Оставить `gateway.http.endpoints.chatCompletions.enabled` независимой; разрешить включать/выключать обе конечные точки отдельно.
-   Выдавать предупреждение при запуске, когда Chat Completions включён, чтобы обозначить его устаревший статус.

## Путь к устареванию Chat Completions

-   Сохранять строгие границы модулей: никаких общих типов схем между responses и chat completions.
-   Сделать Chat Completions опциональным через конфигурацию, чтобы его можно было отключить без изменений кода.
-   Обновить документацию, пометив Chat Completions как устаревший, когда `/v1/responses` станет стабильным.
-   Опциональный будущий шаг: преобразовывать запросы Chat Completions в обработчик Responses для упрощения пути к удалению.

## Поддерживаемое подмножество в Фазе 1

-   Принимать `input` как строку или `ItemParam[]` с ролями сообщений и `function_call_output`.
-   Извлекать системные и developer-сообщения в `extraSystemPrompt`.
-   Использовать последнее сообщение `user` или `function_call_output` в качестве текущего сообщения для запуска агента.
-   Отклонять неподдерживаемые части контента (изображение/файл) с ошибкой `invalid_request_error`.
-   Возвращать одно сообщение assistant с контентом `output_text`.
-   Возвращать `usage` с нулевыми значениями, пока не будет подключён учёт токенов.

## Стратегия валидации (без SDK)

-   Реализовать схемы Zod для поддерживаемого подмножества:
    -   `CreateResponseBody`
    -   `ItemParam` + объединения частей контента сообщений
    -   `ResponseResource`
    -   Формы стриминговых событий, используемые шлюзом
-   Хранить схемы в одном изолированном модуле, чтобы избежать расхождения и позволить будущую генерацию кода.

## Реализация стриминга (Фаза 1)

-   Строки SSE с `event:` и `data:`.
-   Обязательная последовательность (минимально жизнеспособная):
    -   `response.created`
    -   `response.output_item.added`
    -   `response.content_part.added`
    -   `response.output_text.delta` (повторять по необходимости)
    -   `response.output_text.done`
    -   `response.content_part.done`
    -   `response.completed`
    -   `[DONE]`

## План тестирования и проверки

-   Добавить сквозное (e2e) тестирование для `/v1/responses`:
    -   Требуется аутентификация
    -   Форма нестримингового ответа
    -   Порядок событий стриминга и `[DONE]`
    -   Маршрутизация сессий с заголовками и `user`
-   Оставить `src/gateway/openai-http.test.ts` без изменений.
-   Ручная проверка: curl к `/v1/responses` с `stream: true` и проверка порядка событий и терминального `[DONE]`.

## Обновления документации (в последующем)

-   Добавить новую страницу документации по использованию `/v1/responses` с примерами.
-   Обновить `/gateway/openai-http-api` с пометкой об устаревании и ссылкой на `/v1/responses`.

[Рефакторинг Browser Evaluate CDP](./browser-evaluate-cdp-refactor.md)[План PTY и Process Supervision](./pty-process-supervision.md)