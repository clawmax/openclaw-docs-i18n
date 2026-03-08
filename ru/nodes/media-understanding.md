

  Медиа и устройства

  
# Понимание медиа

OpenClaw может **суммаризировать входящие медиафайлы** (изображения/аудио/видео) до запуска конвейера ответа. Он автоматически определяет доступность локальных инструментов или ключей провайдеров, и его можно отключить или настроить. Если функция понимания отключена, модели по-прежнему получают исходные файлы/URL-адреса как обычно.

## Цели

-   Опционально: предварительно обрабатывать входящие медиа в короткий текст для более быстрой маршрутизации и лучшего разбора команд.
-   Сохранять доставку исходных медиа модели (всегда).
-   Поддерживать **API провайдеров** и **резервные CLI-инструменты**.
-   Разрешать использование нескольких моделей с упорядоченным резервированием (ошибка/размер/таймаут).

## Общее поведение

1.  Собрать входящие вложения (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2.  Для каждой включенной возможности (изображение/аудио/видео) выбрать вложения согласно политике (по умолчанию: **первое**).
3.  Выбрать первую подходящую запись модели (размер + возможность + аутентификация).
4.  Если модель не сработала или медиафайл слишком большой, **перейти к следующей записи**.
5.  При успехе:
    -   `Body` становится блоком `[Image]`, `[Audio]` или `[Video]`.
    -   Для аудио устанавливается `{{Transcript}}`; разбор команд использует текст подписи, если он есть, в противном случае — транскрипт.
    -   Подписи сохраняются как `User text:` внутри блока.

Если понимание завершилось ошибкой или отключено, **поток ответа продолжается** с исходным телом и вложениями.

## Обзор конфигурации

`tools.media` поддерживает **общие модели** плюс переопределения для каждой возможности:

-   `tools.media.models`: общий список моделей (используйте `capabilities` для ограничения).
-   `tools.media.image` / `tools.media.audio` / `tools.media.video`:
    -   значения по умолчанию (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
    -   переопределения провайдера (`baseUrl`, `headers`, `providerOptions`)
    -   опции Deepgram для аудио через `tools.media.audio.providerOptions.deepgram`
    -   управление выводом транскрипта аудио (`echoTranscript`, по умолчанию `false`; `echoFormat`)
    -   опциональный **список `models` для конкретной возможности** (имеет приоритет перед общими моделями)
    -   политика `attachments` (`mode`, `maxAttachments`, `prefer`)
    -   `scope` (опциональное ограничение по каналу/типу чата/ключу сессии)
-   `tools.media.concurrency`: максимальное количество одновременных запусков обработки возможностей (по умолчанию **2**).

```json
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
        echoTranscript: true,
        echoFormat: '📝 "{transcript}"',
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### Записи моделей

Каждая запись `models[]` может быть **провайдерской** или **CLI**:

```json
{
  type: "provider", // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, used for multi‑modal entries
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

В CLI-шаблонах также можно использовать:

-   `{{MediaDir}}` (директория, содержащая медиафайл)
-   `{{OutputDir}}` (временная директория, созданная для этого запуска)
-   `{{OutputBase}}` (базовый путь временного файла, без расширения)

## Значения по умолчанию и ограничения

Рекомендуемые значения по умолчанию:

-   `maxChars`: **500** для изображения/видео (коротко, удобно для команд)
-   `maxChars`: **не задано** для аудио (полный транскрипт, если не задано ограничение)
-   `maxBytes`:
    -   изображение: **10MB**
    -   аудио: **20MB**
    -   видео: **50MB**

Правила:

-   Если медиафайл превышает `maxBytes`, эта модель пропускается и **пробуется следующая модель**.
-   Аудиофайлы меньше **1024 байт** считаются пустыми/поврежденными и пропускаются до транскрипции провайдером/CLI.
-   Если модель возвращает больше `maxChars`, вывод обрезается.
-   `prompt` по умолчанию — простое «Опиши .» плюс указание `maxChars` (только для изображения/видео).
-   Если `.enabled: true`, но модели не настроены, OpenClaw пробует **активную модель ответа**, когда её провайдер поддерживает данную возможность.

### Автоопределение понимания медиа (по умолчанию)

Если `tools.media..enabled` **не** установлено в `false` и вы не настроили модели, OpenClaw автоматически определяет в следующем порядке и **останавливается на первом рабочем варианте**:

1.  **Локальные CLI** (только аудио; если установлены)
    -   `sherpa-onnx-offline` (требует `SHERPA_ONNX_MODEL_DIR` с encoder/decoder/joiner/tokens)
    -   `whisper-cli` (`whisper-cpp`; использует `WHISPER_CPP_MODEL` или встроенную tiny-модель)
    -   `whisper` (Python CLI; автоматически загружает модели)
2.  **Gemini CLI** (`gemini`) с использованием `read_many_files`
3.  **Ключи провайдеров**
    -   Аудио: OpenAI → Groq → Deepgram → Google
    -   Изображение: OpenAI → Anthropic → Google → MiniMax
    -   Видео: Google

Чтобы отключить автоопределение, установите:

```json
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

Примечание: Определение наличия бинарных файлов выполняется по возможности на macOS/Linux/Windows; убедитесь, что CLI находится в `PATH` (мы раскрываем `~`), или задайте явную CLI-модель с полным путём команды.

### Поддержка прокси-окружения (модели провайдеров)

Когда включено понимание медиа на основе провайдера для **аудио** и **видео**, OpenClaw учитывает стандартные переменные окружения для исходящего прокси при HTTP-вызовах к провайдерам:

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

Если переменные окружения прокси не заданы, понимание медиа использует прямое соединение. Если значение прокси имеет неверный формат, OpenClaw логирует предупреждение и переходит на прямое соединение.

## Возможности (опционально)

Если вы задали `capabilities`, запись выполняется только для этих типов медиа. Для общих списков OpenClaw может определить значения по умолчанию:

-   `openai`, `anthropic`, `minimax`: **изображение**
-   `google` (Gemini API): **изображение + аудио + видео**
-   `groq`: **аудио**
-   `deepgram`: **аудио**

Для CLI-записей **явно задавайте `capabilities`**, чтобы избежать неожиданных совпадений. Если вы опустите `capabilities`, запись будет доступна для списка, в котором она находится.

## Матрица поддержки провайдеров (интеграции OpenClaw)

| Возможность | Интеграция провайдера | Примечания |
| --- | --- | --- |
| Изображение | OpenAI / Anthropic / Google / другие через `pi-ai` | Работает любая модель в реестре, поддерживающая изображения. |
| Аудио | OpenAI, Groq, Deepgram, Google, Mistral | Транскрипция провайдера (Whisper/Deepgram/Gemini/Voxtral). |
| Видео | Google (Gemini API) | Понимание видео провайдером. |

## Рекомендации по выбору модели

-   Предпочитайте самую мощную модель последнего поколения, доступную для каждой медиа-возможности, когда важны качество и безопасность.
-   Для агентов с инструментами, обрабатывающих непроверенные входные данные, избегайте старых/слабых медиа-моделей.
-   Держите хотя бы один резервный вариант на каждую возможность для обеспечения доступности (качественная модель + более быстрая/дешёвая модель).
-   CLI-резервы (`whisper-cli`, `whisper`, `gemini`) полезны, когда API провайдеров недоступны.
-   Примечание для `parakeet-mlx`: с `--output-dir` OpenClaw читает `<output-dir>/<media-basename>.txt`, когда выходной формат `txt` (или не указан); для форматов, отличных от `txt`, используется вывод в stdout.

## Политика обработки вложений

`attachments` для каждой возможности управляет тем, какие вложения обрабатываются:

-   `mode`: `first` (по умолчанию) или `all`
-   `maxAttachments`: ограничить количество обрабатываемых (по умолчанию **1**)
-   `prefer`: `first`, `last`, `path`, `url`

При `mode: "all"` выводы помечаются как `[Image 1/2]`, `[Audio 2/2]` и т.д.

## Примеры конфигурации

### 1) Общий список моделей + переопределения

```json
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2) Только аудио + видео (изображение отключено)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3) Опциональное понимание изображений

```json
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4) Мультимодальная одиночная запись (явные возможности)

```json
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## Вывод статуса

Когда выполняется понимание медиа, `/status` включает краткую строку сводки:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Это показывает результаты по каждой возможности и выбранного провайдера/модель, где применимо.

## Примечания

-   Понимание работает по принципу **best‑effort**. Ошибки не блокируют ответы.
-   Вложения всё равно передаются моделям, даже когда понимание отключено.
-   Используйте `scope`, чтобы ограничить, где выполняется понимание (например, только в личных сообщениях).

## Связанные документы

-   [Конфигурация](../gateway/configuration.md)
-   [Поддержка изображений и медиа](./images.md)

[Устранение неполадок узлов](./troubleshooting.md)[Поддержка изображений и медиа](./images.md)