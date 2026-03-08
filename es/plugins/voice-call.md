

  Extensiones

  
# Plugin de Llamadas de Voz

Llamadas de voz para OpenClaw a través de un plugin. Soporta notificaciones salientes y conversaciones de múltiples turnos con políticas entrantes. Proveedores actuales:

-   `twilio` (Programmable Voice + Media Streams)
-   `telnyx` (Call Control v2)
-   `plivo` (Voice API + transferencia XML + GetInput speech)
-   `mock` (dev/sin red)

Modelo mental rápido:

-   Instalar plugin
-   Reiniciar Gateway
-   Configurar bajo `plugins.entries.voice-call.config`
-   Usar `openclaw voicecall ...` o la herramienta `voice_call`

## Dónde se ejecuta (local vs remoto)

El plugin de Llamadas de Voz se ejecuta **dentro del proceso del Gateway**. Si usas un Gateway remoto, instala/configura el plugin en la **máquina que ejecuta el Gateway**, luego reinicia el Gateway para cargarlo.

## Instalar

### Opción A: instalar desde npm (recomendado)

```bash
openclaw plugins install @openclaw/voice-call
```

Reinicia el Gateway después.

### Opción B: instalar desde una carpeta local (dev, sin copiar)

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

Reinicia el Gateway después.

## Configuración

Establece la configuración bajo `plugins.entries.voice-call.config`:

```json
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // o "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          telnyx: {
            apiKey: "...",
            connectionId: "...",
            // Clave pública del webhook de Telnyx desde el Portal Mission Control de Telnyx
            // (Cadena Base64; también se puede configurar mediante TELNYX_PUBLIC_KEY).
            publicKey: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // Servidor webhook
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // Seguridad del webhook (recomendado para túneles/proxies)
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // Exposición pública (elige una)
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

Notas:

-   Twilio/Telnyx requieren una URL de webhook **públicamente accesible**.
-   Plivo requiere una URL de webhook **públicamente accesible**.
-   `mock` es un proveedor de desarrollo local (sin llamadas de red).
-   Telnyx requiere `telnyx.publicKey` (o `TELNYX_PUBLIC_KEY`) a menos que `skipSignatureVerification` sea verdadero.
-   `skipSignatureVerification` es solo para pruebas locales.
-   Si usas el nivel gratuito de ngrok, establece `publicUrl` a la URL exacta de ngrok; la verificación de firma siempre se aplica.
-   `tunnel.allowNgrokFreeTierLoopbackBypass: true` permite webhooks de Twilio con firmas inválidas **solo** cuando `tunnel.provider="ngrok"` y `serve.bind` es loopback (agente local de ngrok). Usar solo para desarrollo local.
-   Las URLs del nivel gratuito de ngrok pueden cambiar o agregar comportamiento intersticial; si `publicUrl` se desvía, las firmas de Twilio fallarán. Para producción, prefiere un dominio estable o Tailscale funnel.
-   Seguridad de streaming por defecto:
    -   `streaming.preStartTimeoutMs` cierra sockets que nunca envían un frame `start` válido.
    -   `streaming.maxPendingConnections` limita el total de sockets pre-inicio no autenticados.
    -   `streaming.maxPendingConnectionsPerIp` limita los sockets pre-inicio no autenticados por IP de origen.
    -   `streaming.maxConnections` limita el total de sockets de flujo de medios abiertos (pendientes + activos).

## Limpiador de llamadas obsoletas

Usa `staleCallReaperSeconds` para finalizar llamadas que nunca reciben un webhook terminal (por ejemplo, llamadas en modo notify que nunca se completan). El valor por defecto es `0` (deshabilitado). Rangos recomendados:

-   **Producción:** `120`–`300` segundos para flujos tipo notify.
-   Mantén este valor **mayor que `maxDurationSeconds`** para que las llamadas normales puedan terminar. Un buen punto de partida es `maxDurationSeconds + 30–60` segundos.

Ejemplo:

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

## Seguridad del Webhook

Cuando un proxy o túnel se sitúa frente al Gateway, el plugin reconstruye la URL pública para la verificación de firma. Estas opciones controlan qué encabezados reenviados son confiables. `webhookSecurity.allowedHosts` permite listar hosts de encabezados de reenvío. `webhookSecurity.trustForwardingHeaders` confía en encabezados reenviados sin una lista de permitidos. `webhookSecurity.trustedProxyIPs` solo confía en encabezados reenviados cuando la IP remota de la solicitud coincide con la lista. La protección contra repetición de webhooks está habilitada para Twilio y Plivo. Las solicitudes de webhook válidas repetidas se reconocen pero se omiten para efectos secundarios. Los turnos de conversación de Twilio incluyen un token por turno en las devoluciones de llamada ``, por lo que las devoluciones de llamada de voz obsoletas/repetidas no pueden satisfacer un turno de transcripción pendiente más nuevo. Ejemplo con un host público estable:

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

## TTS para llamadas

Voice Call usa la configuración central `messages.tts` (OpenAI o ElevenLabs) para el habla en streaming en llamadas. Puedes anularla bajo la configuración del plugin con la **misma estructura** — se fusiona en profundidad con `messages.tts`.

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

Notas:

-   **Edge TTS se ignora para llamadas de voz** (el audio de telefonía necesita PCM; la salida de Edge no es confiable).
-   El TTS central se usa cuando el streaming de medios de Twilio está habilitado; de lo contrario, las llamadas recurren a voces nativas del proveedor.

### Más ejemplos

Usar solo TTS central (sin anulación):

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

Anular a ElevenLabs solo para llamadas (mantener el valor por defecto central en otros lugares):

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

Anular solo el modelo de OpenAI para llamadas (ejemplo de fusión en profundidad):

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

## Llamadas entrantes

La política entrante por defecto es `disabled`. Para habilitar llamadas entrantes, configura:

```json
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "¡Hola! ¿En qué puedo ayudarte?",
}
```

Las respuestas automáticas usan el sistema del agente. Ajusta con:

-   `responseModel`
-   `responseSystemPrompt`
-   `responseTimeoutMs`

## CLI

```bash
openclaw voicecall call --to "+15555550123" --message "Hola desde OpenClaw"
openclaw voicecall continue --call-id <id> --message "¿Alguna pregunta?"
openclaw voicecall speak --call-id <id> --message "Un momento"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```

## Herramienta del agente

Nombre de la herramienta: `voice_call` Acciones:

-   `initiate_call` (message, to?, mode?)
-   `continue_call` (callId, message)
-   `speak_to_user` (callId, message)
-   `end_call` (callId)
-   `get_status` (callId)

Este repositorio incluye un documento de habilidad correspondiente en `skills/voice-call/SKILL.md`.

## Gateway RPC

-   `voicecall.initiate` (`to?`, `message`, `mode?`)
-   `voicecall.continue` (`callId`, `message`)
-   `voicecall.speak` (`callId`, `message`)
-   `voicecall.end` (`callId`)
-   `voicecall.status` (`callId`)

[Plugins de la comunidad](./community.md)[Plugin Zalo Personal](./zalouser.md)