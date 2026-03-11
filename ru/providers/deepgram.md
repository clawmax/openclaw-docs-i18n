

  Провайдеры

  
# Deepgram

Deepgram — это API для преобразования речи в текст. В OpenClaw он используется для **транскрипции входящих аудио/голосовых заметок** через `tools.media.audio`. Когда функция включена, OpenClaw загружает аудиофайл в Deepgram и вставляет транскрипт в конвейер ответа (`{{Transcript}}` + блок `[Audio]`). Это **не потоковая обработка**; используется конечная точка для предварительно записанного аудио. Веб-сайт: [https://deepgram.com](https://deepgram.com)  
Документация: [https://developers.deepgram.com](https://developers.deepgram.com)

## Быстрый старт

1.  Установите ваш API-ключ:

```
DEEPGRAM_API_KEY=dg_...
```

2.  Включите провайдер:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Параметры

-   `model`: Идентификатор модели Deepgram (по умолчанию: `nova-3`)
-   `language`: Указание языка (опционально)
-   `tools.media.audio.providerOptions.deepgram.detect_language`: включить определение языка (опционально)
-   `tools.media.audio.providerOptions.deepgram.punctuate`: включить расстановку знаков препинания (опционально)
-   `tools.media.audio.providerOptions.deepgram.smart_format`: включить интеллектуальное форматирование (опционально)

Пример с указанием языка:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3", language: "en" }],
      },
    },
  },
}
```

Пример с параметрами Deepgram:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true,
          },
        },
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Примечания

-   Аутентификация следует стандартному порядку для провайдеров; `DEEPGRAM_API_KEY` — самый простой путь.
-   Переопределите конечные точки или заголовки с помощью `tools.media.audio.baseUrl` и `tools.media.audio.headers` при использовании прокси.
-   Вывод следует тем же правилам для аудио, что и у других провайдеров (ограничения размера, таймауты, вставка транскрипта).

[Прокси API Claude Max](./claude-max-api-proxy.md)[GitHub Copilot](./github-copilot.md)