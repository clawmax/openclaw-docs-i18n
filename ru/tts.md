title: "Конфигурация OpenClaw Text-to-Speech (TTS) для ElevenLabs, OpenAI и Edge"
description: "Узнайте, как настроить OpenClaw TTS с ElevenLabs, OpenAI или Edge TTS. Настройте аудиоответы, резервные провайдеры, переопределения моделей и пользовательские предпочтения."
keywords: ["openclaw tts", "конфигурация text-to-speech", "elevenlabs api", "openai tts", "edge tts", "аудиоответы", "провайдер tts", "синтез речи"]
---

  Медиа и устройства

  
# Преобразование текста в речь

OpenClaw может преобразовывать исходящие ответы в аудио с помощью ElevenLabs, OpenAI или Edge TTS. Это работает везде, где OpenClaw может отправлять аудио; в Telegram ответы приходят в круглом пузырьке голосового сообщения.

## Поддерживаемые сервисы

-   **ElevenLabs** (основной или резервный провайдер)
-   **OpenAI** (основной или резервный провайдер; также используется для суммаризации)
-   **Edge TTS** (основной или резервный провайдер; использует `node-edge-tts`, используется по умолчанию при отсутствии API-ключей)

### Примечания по Edge TTS

Edge TTS использует онлайн-нейронный сервис синтеза речи Microsoft Edge через библиотеку `node-edge-tts`. Это хостируемый сервис (не локальный), использующий конечные точки Microsoft, и не требующий API-ключа. `node-edge-tts` предоставляет опции конфигурации речи и форматы вывода, но не все опции поддерживаются сервисом Edge. citeturn2search0 Поскольку Edge TTS — это публичный веб-сервис без опубликованного SLA или квот, рассматривайте его как сервис по принципу "как получится". Если вам нужны гарантированные лимиты и поддержка, используйте OpenAI или ElevenLabs. В документации к REST API Microsoft Speech указано ограничение в 10 минут аудио на запрос; Edge TTS не публикует лимиты, поэтому предполагайте аналогичные или более низкие ограничения. citeturn0search3

## Необязательные ключи

Если вы хотите использовать OpenAI или ElevenLabs:

-   `ELEVENLABS_API_KEY` (или `XI_API_KEY`)
-   `OPENAI_API_KEY`

Edge TTS **не** требует API-ключа. Если API-ключи не найдены, OpenClaw по умолчанию использует Edge TTS (если не отключено через `messages.tts.edge.enabled=false`). Если настроено несколько провайдеров, выбранный провайдер используется первым, а остальные служат резервными вариантами. Авто-суммаризация использует настроенную `summaryModel` (или `agents.defaults.model.primary`), поэтому этот провайдер также должен быть аутентифицирован, если вы включаете суммаризацию.

## Ссылки на сервисы

-   [Руководство по OpenAI Text-to-Speech](https://platform.openai.com/docs/guides/text-to-speech)
-   [Справочник по OpenAI Audio API](https://platform.openai.com/docs/api-reference/audio)
-   [ElevenLabs Text to Speech](https://elevenlabs.io/docs/api-reference/text-to-speech)
-   [ElevenLabs Authentication](https://elevenlabs.io/docs/api-reference/authentication)
-   [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
-   [Форматы вывода Microsoft Speech](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

## Включено ли по умолчанию?

Нет. Авто‑TTS по умолчанию **выключен**. Включите его в конфиге с помощью `messages.tts.auto` или для сессии командой `/tts always` (алиас: `/tts on`). Edge TTS по умолчанию **включен**, как только TTS активирован, и используется автоматически, когда нет доступных API-ключей OpenAI или ElevenLabs.

## Конфигурация

Конфигурация TTS находится в `openclaw.json` в разделе `messages.tts`. Полная схема в [Конфигурации шлюза](./gateway/configuration.md).

### Минимальная конфигурация (включение + провайдер)

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs",
    },
  },
}
```

### OpenAI основной с ElevenLabs резервным

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      openai: {
        apiKey: "openai_api_key",
        baseUrl: "https://api.openai.com/v1",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
    },
  },
}
```

### Edge TTS основной (без API-ключа)

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%",
      },
    },
  },
}
```

### Отключить Edge TTS

```json
{
  messages: {
    tts: {
      edge: {
        enabled: false,
      },
    },
  },
}
```

### Пользовательские лимиты + путь к настройкам

```json
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
    },
  },
}
```

### Отправлять аудио только после входящего голосового сообщения

```json
{
  messages: {
    tts: {
      auto: "inbound",
    },
  },
}
```

### Отключить авто-суммаризацию для длинных ответов

```json
{
  messages: {
    tts: {
      auto: "always",
    },
  },
}
```

Затем выполните:

```bash
/tts summary off
```

### Примечания к полям

-   `auto`: режим авто‑TTS (`off`, `always`, `inbound`, `tagged`).
    -   `inbound` отправляет аудио только после входящего голосового сообщения.
    -   `tagged` отправляет аудио только когда ответ содержит теги `[[tts]]`.
-   `enabled`: устаревший переключатель (доктор мигрирует это в `auto`).
-   `mode`: `"final"` (по умолчанию) или `"all"` (включает ответы инструментов/блоков).
-   `provider`: `"elevenlabs"`, `"openai"` или `"edge"` (резервный провайдер выбирается автоматически).
-   Если `provider` **не задан**, OpenClaw предпочитает `openai` (если есть ключ), затем `elevenlabs` (если есть ключ), иначе `edge`.
-   `summaryModel`: необязательная дешёвая модель для авто-суммаризации; по умолчанию `agents.defaults.model.primary`.
    -   Принимает `provider/model` или настроенный алиас модели.
-   `modelOverrides`: разрешить модели отправлять TTS-директивы (включено по умолчанию).
    -   `allowProvider` по умолчанию `false` (переключение провайдера требует явного разрешения).
-   `maxTextLength`: жёсткое ограничение на входной текст для TTS (символы). `/tts audio` завершится ошибкой при превышении.
-   `timeoutMs`: таймаут запроса (мс).
-   `prefsPath`: переопределить путь к локальному JSON с настройками (провайдер/лимит/суммаризация).
-   Значения `apiKey` используют переменные окружения как запасной вариант (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
-   `elevenlabs.baseUrl`: переопределить базовый URL API ElevenLabs.
-   `openai.baseUrl`: переопределить конечную точку OpenAI TTS.
    -   Порядок определения: `messages.tts.openai.baseUrl` -> `OPENAI_TTS_BASE_URL` -> `https://api.openai.com/v1`
    -   Нестандартные значения рассматриваются как совместимые с OpenAI TTS конечные точки, поэтому принимаются пользовательские имена моделей и голосов.
-   `elevenlabs.voiceSettings`:
    -   `stability`, `similarityBoost`, `style`: `0..1`
    -   `useSpeakerBoost`: `true|false`
    -   `speed`: `0.5..2.0` (1.0 = нормальная скорость)
-   `elevenlabs.applyTextNormalization`: `auto|on|off`
-   `elevenlabs.languageCode`: 2-буквенный код ISO 639-1 (например, `en`, `de`)
-   `elevenlabs.seed`: целое число `0..4294967295` (детерминированность по возможности)
-   `edge.enabled`: разрешить использование Edge TTS (по умолчанию `true`; без API-ключа).
-   `edge.voice`: имя нейронного голоса Edge (например, `en-US-MichelleNeural`).
-   `edge.lang`: код языка (например, `en-US`).
-   `edge.outputFormat`: формат вывода Edge (например, `audio-24khz-48kbitrate-mono-mp3`).
    -   См. форматы вывода Microsoft Speech для допустимых значений; не все форматы поддерживаются Edge.
-   `edge.rate` / `edge.pitch` / `edge.volume`: строки процентов (например, `+10%`, `-5%`).
-   `edge.saveSubtitles`: записывать JSON-субтитры рядом с аудиофайлом.
-   `edge.proxy`: URL прокси для запросов Edge TTS.
-   `edge.timeoutMs`: переопределение таймаута запроса (мс).

## Переопределения на уровне модели (включено по умолчанию)

По умолчанию модель **может** отправлять TTS-директивы для одного ответа. Когда `messages.tts.auto` установлен в `tagged`, эти директивы необходимы для запуска аудио. При включении модель может отправлять директивы `[[tts:...]]` для переопределения голоса для одного ответа, плюс необязательный блок `[[tts:text]]...[[/tts:text]]` для предоставления выразительных тегов (смех, реплики для пения и т.д.), которые должны появляться только в аудио. Директивы `provider=...` игнорируются, если не установлено `modelOverrides.allowProvider: true`. Пример полезной нагрузки ответа:

```
Вот, пожалуйста.

[[tts:voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](смеётся) Прочти песню ещё раз.[[/tts:text]]
```

Доступные ключи директив (при включении):

-   `provider` (`openai` | `elevenlabs` | `edge`, требует `allowProvider: true`)
-   `voice` (голос OpenAI) или `voiceId` (ElevenLabs)
-   `model` (модель TTS OpenAI или идентификатор модели ElevenLabs)
-   `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
-   `applyTextNormalization` (`auto|on|off`)
-   `languageCode` (ISO 639-1)
-   `seed`

Отключить все переопределения модели:

```json
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false,
      },
    },
  },
}
```

Необязательный белый список (разрешить переключение провайдера, оставляя другие настройки конфигурируемыми):

```json
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: true,
        allowSeed: false,
      },
    },
  },
}
```

## Пользовательские предпочтения

Слэш-команды записывают локальные переопределения в `prefsPath` (по умолчанию: `~/.openclaw/settings/tts.json`, можно переопределить через `OPENCLAW_TTS_PREFS` или `messages.tts.prefsPath`). Сохраняемые поля:

-   `enabled`
-   `provider`
-   `maxLength` (порог для суммаризации; по умолчанию 1500 символов)
-   `summarize` (по умолчанию `true`)

Они переопределяют `messages.tts.*` для данного хоста.

## Форматы вывода (фиксированные)

-   **Telegram**: голосовое сообщение Opus (`opus_48000_64` от ElevenLabs, `opus` от OpenAI).
    -   48 кГц / 64 кбит/с — хороший компромисс для голосовых сообщений и требуется для круглого пузырька.
-   **Другие каналы**: MP3 (`mp3_44100_128` от ElevenLabs, `mp3` от OpenAI).
    -   44.1 кГц / 128 кбит/с — баланс по умолчанию для чёткости речи.
-   **Edge TTS**: использует `edge.outputFormat` (по умолчанию `audio-24khz-48kbitrate-mono-mp3`).
    -   `node-edge-tts` принимает `outputFormat`, но не все форматы доступны в сервисе Edge. citeturn2search0
    -   Значения форматов вывода следуют форматам вывода Microsoft Speech (включая Ogg/WebM Opus). citeturn1search0
    -   Telegram `sendVoice` принимает OGG/MP3/M4A; используйте OpenAI/ElevenLabs, если вам нужны гарантированные голосовые сообщения Opus. citeturn1search1
    -   Если настроенный формат вывода Edge не срабатывает, OpenClaw повторяет попытку с MP3.

Форматы OpenAI/ElevenLabs фиксированы; Telegram ожидает Opus для UX голосовых сообщений.

## Поведение авто-TTS

При включении OpenClaw:

-   пропускает TTS, если ответ уже содержит медиа или директиву `MEDIA:`.
-   пропускает очень короткие ответы (< 10 символов).
-   суммаризирует длинные ответы, когда включено, используя `agents.defaults.model.primary` (или `summaryModel`).
-   прикрепляет сгенерированное аудио к ответу.

Если ответ превышает `maxLength` и суммаризация выключена (или нет API-ключа для модели суммаризации), аудио пропускается и отправляется обычный текстовый ответ.

## Диаграмма потока

```
Ответ -> TTS включен?
  нет  -> отправить текст
  да -> есть медиа / MEDIA: / короткий?
          да -> отправить текст
          нет  -> длина > лимит?
                   нет  -> TTS -> прикрепить аудио
                   да -> суммаризация включена?
                            нет  -> отправить текст
                            да -> суммаризировать (summaryModel или agents.defaults.model.primary)
                                      -> TTS -> прикрепить аудио
```

## Использование слэш-команд

Есть одна команда: `/tts`. Подробности о включении см. в [Слэш-командах](./tools/slash-commands.md). Примечание для Discord: `/tts` — это встроенная команда Discord, поэтому OpenClaw регистрирует `/voice` как нативную команду там. Текст `/tts ...` всё равно работает.

```bash
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

Примечания:

-   Команды требуют авторизованного отправителя (правила белого списка/владельца всё ещё применяются).
-   `commands.text` или регистрация нативных команд должны быть включены.
-   `off|always|inbound|tagged` — это переключатели для сессии (`/tts on` — это алиас для `/tts always`).
-   `limit` и `summary` сохраняются в локальных настройках, а не в основном конфиге.
-   `/tts audio` генерирует разовый аудиоответ (не включает TTS).

## Инструмент агента

Инструмент `tts` преобразует текст в речь и возвращает путь `MEDIA:`. Когда результат совместим с Telegram, инструмент включает `[[audio_as_voice]]`, чтобы Telegram отправил голосовой пузырёк.

## RPC шлюза

Методы шлюза:

-   `tts.status`
-   `tts.enable`
-   `tts.disable`
-   `tts.convert`
-   `tts.setProvider`
-   `tts.providers`

[Команда Location](./nodes/location-command.md)

---