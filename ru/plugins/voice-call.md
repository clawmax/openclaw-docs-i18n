

  Расширения

  
# Плагин Voice Call

Голосовые вызовы для OpenClaw через плагин. Поддерживает исходящие уведомления и многошаговые диалоги с политиками для входящих вызовов. Текущие провайдеры:

-   `twilio` (Programmable Voice + Media Streams)
-   `telnyx` (Call Control v2)
-   `plivo` (Voice API + XML transfer + GetInput speech)
-   `mock` (разработка/без сети)

Краткая модель:

-   Установите плагин
-   Перезапустите Gateway
-   Настройте в разделе `plugins.entries.voice-call.config`
-   Используйте `openclaw voicecall ...` или инструмент `voice_call`

## Где работает (локально vs удалённо)

Плагин Voice Call работает **внутри процесса Gateway**. Если вы используете удалённый Gateway, установите/настройте плагин на **машине, где работает Gateway**, затем перезапустите Gateway для его загрузки.

## Установка

### Вариант A: установка из npm (рекомендуется)

```bash
openclaw plugins install @openclaw/voice-call
```

После этого перезапустите Gateway.

### Вариант B: установка из локальной папки (разработка, без копирования)

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

После этого перезапустите Gateway.

## Конфигурация

Задайте конфигурацию в разделе `plugins.entries.voice-call.config`:

```json
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // или "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          telnyx: {
            apiKey: "...",
            connectionId: "...",
            // Публичный ключ вебхука Telnyx из Telnyx Mission Control Portal
            // (Строка Base64; также можно задать через TELNYX_PUBLIC_KEY).
            publicKey: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // Вебхук-сервер
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // Безопасность вебхуков (рекомендуется для туннелей/прокси)
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // Публичная доступность (выберите один вариант)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify", // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream",
            preStartTimeoutMs: 5000,
            maxPendingConnections: 32,
            maxPendingConnectionsPerIp: 4,
            maxConnections: 128,
          },
        },
      },
    },
  },
}
```

Примечания:

-   Twilio/Telnyx требуют **публично доступный** URL вебхука.
-   Plivo требует **публично доступный** URL вебхука.
-   `mock` — это локальный провайдер для разработки (без сетевых вызовов).
-   Telnyx требует `telnyx.publicKey` (или `TELNYX_PUBLIC_KEY`), если только `skipSignatureVerification` не равен true.
-   `skipSignatureVerification` предназначен только для локального тестирования.
-   Если вы используете бесплатный тариф ngrok, задайте `publicUrl` как точный URL ngrok; проверка подписи всегда применяется.
-   `tunnel.allowNgrokFreeTierLoopbackBypass: true` разрешает вебхуки Twilio с недействительными подписями **только** когда `tunnel.provider="ngrok"` и `serve.bind` является loopback (локальный агент ngrok). Используйте только для локальной разработки.
-   URL бесплатного тарифа ngrok могут меняться или добавлять промежуточное поведение; если `publicUrl` изменится, проверка подписей Twilio не пройдёт. Для продакшена предпочтительнее стабильный домен или Tailscale funnel.
-   Настройки безопасности потоковой передачи по умолчанию:
    -   `streaming.preStartTimeoutMs` закрывает сокеты, которые никогда не отправляют корректный фрейм `start`.
    -   `streaming.maxPendingConnections` ограничивает общее количество неподтверждённых сокетов до старта.
    -   `streaming.maxPendingConnectionsPerIp` ограничивает неподтверждённые сокеты до старта на IP-адрес источника.
    -   `streaming.maxConnections` ограничивает общее количество открытых сокетов медиапотока (ожидающие + активные).

## Очистка зависших вызовов

Используйте `staleCallReaperSeconds` для завершения вызовов, которые никогда не получают терминальный вебхук (например, вызовы в режиме notify, которые никогда не завершаются). Значение по умолчанию — `0` (отключено). Рекомендуемые диапазоны:

-   **Продакшен:** `120`–`300` секунд для потоков в стиле notify.
-   Держите это значение **выше, чем `maxDurationSeconds`**, чтобы обычные вызовы могли завершиться. Хорошая отправная точка — `maxDurationSeconds + 30–60` секунд.

Пример:

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          maxDurationSeconds: 300,
          staleCallReaperSeconds: 360,
        },
      },
    },
  },
}
```

## Безопасность вебхуков

Когда перед Gateway находится прокси или туннель, плагин восстанавливает публичный URL для проверки подписи. Эти параметры контролируют, каким переадресованным заголовкам можно доверять. `webhookSecurity.allowedHosts` разрешает хосты из переадресованных заголовков. `webhookSecurity.trustForwardingHeaders` доверяет переадресованным заголовкам без списка разрешённых хостов. `webhookSecurity.trustedProxyIPs` доверяет переадресованным заголовкам только тогда, когда IP-адрес запроса совпадает со списком. Защита от повторного воспроизведения вебхуков включена для Twilio и Plivo. Повторённые корректные запросы вебхуков подтверждаются, но пропускаются для побочных эффектов. Шаги диалога Twilio включают токен на каждый шаг в колбэках ``, поэтому устаревшие/повторённые колбэки речи не могут удовлетворить более новый ожидающий шаг расшифровки. Пример со стабильным публичным хостом:

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          publicUrl: "https://voice.example.com/voice/webhook",
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
          },
        },
      },
    },
  },
}
```

## TTS для вызовов

Voice Call использует основную конфигурацию `messages.tts` (OpenAI или ElevenLabs) для потоковой речи в вызовах. Вы можете переопределить её в конфигурации плагина с **такой же структурой** — она глубоко объединяется с `messages.tts`.

```json
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2",
    },
  },
}
```

Примечания:

-   **Edge TTS игнорируется для голосовых вызовов** (аудио телефонии требует PCM; вывод Edge ненадёжен).
-   Основной TTS используется, когда включена потоковая передача медиа Twilio; в противном случае вызовы возвращаются к нативным голосам провайдера.

### Другие примеры

Использовать только основной TTS (без переопределения):

```json
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" },
    },
  },
}
```

Переопределить на ElevenLabs только для вызовов (сохранить основной TTS по умолчанию в других местах):

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2",
            },
          },
        },
      },
    },
  },
}
```

Переопределить только модель OpenAI для вызовов (пример глубокого объединения):

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin",
            },
          },
        },
      },
    },
  },
}
```

## Входящие вызовы

Политика для входящих вызовов по умолчанию — `disabled`. Чтобы включить входящие вызовы, задайте:

```json
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Привет! Чем могу помочь?",
}
```

Автоответы используют систему агента. Настройте с помощью:

-   `responseModel`
-   `responseSystemPrompt`
-   `responseTimeoutMs`

## CLI

```bash
openclaw voicecall call --to "+15555550123" --message "Привет от OpenClaw"
openclaw voicecall continue --call-id <id> --message "Есть вопросы?"
openclaw voicecall speak --call-id <id> --message "Один момент"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```

## Инструмент агента

Название инструмента: `voice_call` Действия:

-   `initiate_call` (message, to?, mode?)
-   `continue_call` (callId, message)
-   `speak_to_user` (callId, message)
-   `end_call` (callId)
-   `get_status` (callId)

Этот репозиторий содержит соответствующую документацию навыка в `skills/voice-call/SKILL.md`.

## Gateway RPC

-   `voicecall.initiate` (`to?`, `message`, `mode?`)
-   `voicecall.continue` (`callId`, `message`)
-   `voicecall.speak` (`callId`, `message`)
-   `voicecall.end` (`callId`)
-   `voicecall.status` (`callId`)

[Сообщество плагинов](./community.md)[Плагин Zalo Personal](./zalouser.md)