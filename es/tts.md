

  Medios y dispositivos

  
# Texto a Voz

OpenClaw puede convertir respuestas salientes en audio usando ElevenLabs, OpenAI o Edge TTS. Funciona donde sea que OpenClaw pueda enviar audio; Telegram recibe una burbuja de nota de voz redonda.

## Servicios soportados

-   **ElevenLabs** (proveedor principal o de respaldo)
-   **OpenAI** (proveedor principal o de respaldo; también usado para resúmenes)
-   **Edge TTS** (proveedor principal o de respaldo; usa `node-edge-tts`, predeterminado cuando no hay claves API)

### Notas sobre Edge TTS

Edge TTS utiliza el servicio de TTS neuronal en línea de Microsoft Edge a través de la biblioteca `node-edge-tts`. Es un servicio alojado (no local), utiliza los endpoints de Microsoft y no requiere una clave API. `node-edge-tts` expone opciones de configuración de voz y formatos de salida, pero no todas las opciones son compatibles con el servicio Edge. citeturn2search0 Dado que Edge TTS es un servicio web público sin un SLA publicado ni cuota, trátalo como de mejor esfuerzo. Si necesitas límites y soporte garantizados, usa OpenAI o ElevenLabs. La API REST de Speech de Microsoft documenta un límite de 10 minutos de audio por solicitud; Edge TTS no publica límites, así que asume límites similares o menores. citeturn0search3

## Claves opcionales

Si quieres OpenAI o ElevenLabs:

-   `ELEVENLABS_API_KEY` (o `XI_API_KEY`)
-   `OPENAI_API_KEY`

Edge TTS **no** requiere una clave API. Si no se encuentran claves API, OpenClaw usa Edge TTS por defecto (a menos que se deshabilite mediante `messages.tts.edge.enabled=false`). Si se configuran múltiples proveedores, el proveedor seleccionado se usa primero y los otros son opciones de respaldo. El auto-resumen usa el `summaryModel` configurado (o `agents.defaults.model.primary`), por lo que ese proveedor también debe estar autenticado si habilitas los resúmenes.

## Enlaces de servicio

-   [Guía de Texto a Voz de OpenAI](https://platform.openai.com/docs/guides/text-to-speech)
-   [Referencia de la API de Audio de OpenAI](https://platform.openai.com/docs/api-reference/audio)
-   [Texto a Voz de ElevenLabs](https://elevenlabs.io/docs/api-reference/text-to-speech)
-   [Autenticación de ElevenLabs](https://elevenlabs.io/docs/api-reference/authentication)
-   [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
-   [Formatos de salida de Microsoft Speech](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

## ¿Está habilitado por defecto?

No. El TTS automático está **desactivado** por defecto. Actívalo en la configuración con `messages.tts.auto` o por sesión con `/tts always` (alias: `/tts on`). Edge TTS **sí** está habilitado por defecto una vez que el TTS está activado, y se usa automáticamente cuando no hay claves API de OpenAI o ElevenLabs disponibles.

## Configuración

La configuración de TTS se encuentra bajo `messages.tts` en `openclaw.json`. El esquema completo está en [Configuración de Gateway](./gateway/configuration.md).

### Configuración mínima (habilitar + proveedor)

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

### OpenAI principal con ElevenLabs de respaldo

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

### Edge TTS principal (sin clave API)

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

### Deshabilitar Edge TTS

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

### Límites personalizados + ruta de preferencias

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

### Solo responder con audio después de una nota de voz entrante

```json
{
  messages: {
    tts: {
      auto: "inbound",
    },
  },
}
```

### Deshabilitar auto-resumen para respuestas largas

```json
{
  messages: {
    tts: {
      auto: "always",
    },
  },
}
```

Luego ejecuta:

```bash
/tts summary off
```

### Notas sobre los campos

-   `auto`: modo de TTS automático (`off`, `always`, `inbound`, `tagged`).
    -   `inbound` solo envía audio después de una nota de voz entrante.
    -   `tagged` solo envía audio cuando la respuesta incluye etiquetas `[[tts]]`.
-   `enabled`: interruptor heredado (el doctor migra esto a `auto`).
-   `mode`: `"final"` (predeterminado) o `"all"` (incluye respuestas de herramienta/bloque).
-   `provider`: `"elevenlabs"`, `"openai"`, o `"edge"` (el respaldo es automático).
-   Si `provider` **no está configurado**, OpenClaw prefiere `openai` (si hay clave), luego `elevenlabs` (si hay clave), de lo contrario `edge`.
-   `summaryModel`: modelo económico opcional para auto-resumen; por defecto es `agents.defaults.model.primary`.
    -   Acepta `provider/model` o un alias de modelo configurado.
-   `modelOverrides`: permite que el modelo emita directivas de TTS (activado por defecto).
    -   `allowProvider` por defecto es `false` (el cambio de proveedor es opcional).
-   `maxTextLength`: límite máximo para la entrada de TTS (caracteres). `/tts audio` falla si se excede.
-   `timeoutMs`: tiempo de espera de la solicitud (ms).
-   `prefsPath`: anula la ruta JSON de preferencias locales (proveedor/límite/resumen).
-   Los valores de `apiKey` recurren a variables de entorno (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
-   `elevenlabs.baseUrl`: anula la URL base de la API de ElevenLabs.
-   `openai.baseUrl`: anula el endpoint de TTS de OpenAI.
    -   Orden de resolución: `messages.tts.openai.baseUrl` -> `OPENAI_TTS_BASE_URL` -> `https://api.openai.com/v1`
    -   Los valores no predeterminados se tratan como endpoints de TTS compatibles con OpenAI, por lo que se aceptan nombres de modelo y voz personalizados.
-   `elevenlabs.voiceSettings`:
    -   `stability`, `similarityBoost`, `style`: `0..1`
    -   `useSpeakerBoost`: `true|false`
    -   `speed`: `0.5..2.0` (1.0 = normal)
-   `elevenlabs.applyTextNormalization`: `auto|on|off`
-   `elevenlabs.languageCode`: ISO 639-1 de 2 letras (ej. `en`, `de`)
-   `elevenlabs.seed`: entero `0..4294967295` (determinismo de mejor esfuerzo)
-   `edge.enabled`: permite el uso de Edge TTS (predeterminado `true`; sin clave API).
-   `edge.voice`: nombre de voz neuronal de Edge (ej. `en-US-MichelleNeural`).
-   `edge.lang`: código de idioma (ej. `en-US`).
-   `edge.outputFormat`: formato de salida de Edge (ej. `audio-24khz-48kbitrate-mono-mp3`).
    -   Consulta los formatos de salida de Microsoft Speech para valores válidos; no todos los formatos son compatibles con Edge.
-   `edge.rate` / `edge.pitch` / `edge.volume`: cadenas de porcentaje (ej. `+10%`, `-5%`).
-   `edge.saveSubtitles`: escribe subtítulos JSON junto al archivo de audio.
-   `edge.proxy`: URL de proxy para solicitudes de Edge TTS.
-   `edge.timeoutMs`: anulación del tiempo de espera de la solicitud (ms).

## Anulaciones impulsadas por el modelo (activadas por defecto)

Por defecto, el modelo **puede** emitir directivas de TTS para una sola respuesta. Cuando `messages.tts.auto` es `tagged`, estas directivas son necesarias para activar el audio. Cuando está habilitado, el modelo puede emitir directivas `[[tts:...]]` para anular la voz para una sola respuesta, más un bloque opcional `[[tts:text]]...[[/tts:text]]` para proporcionar etiquetas expresivas (risas, indicaciones de canto, etc.) que solo deben aparecer en el audio. Las directivas `provider=...` se ignoran a menos que `modelOverrides.allowProvider: true`. Ejemplo de carga útil de respuesta:

```
Aquí tienes.

[[tts:voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](se ríe) Lee la canción una vez más.[[/tts:text]]
```

Claves de directiva disponibles (cuando están habilitadas):

-   `provider` (`openai` | `elevenlabs` | `edge`, requiere `allowProvider: true`)
-   `voice` (voz de OpenAI) o `voiceId` (ElevenLabs)
-   `model` (modelo de TTS de OpenAI o id de modelo de ElevenLabs)
-   `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
-   `applyTextNormalization` (`auto|on|off`)
-   `languageCode` (ISO 639-1)
-   `seed`

Deshabilita todas las anulaciones del modelo:

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

Lista blanca opcional (habilita el cambio de proveedor mientras mantiene otras opciones configurables):

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

## Preferencias por usuario

Los comandos de barra escriben anulaciones locales en `prefsPath` (predeterminado: `~/.openclaw/settings/tts.json`, anular con `OPENCLAW_TTS_PREFS` o `messages.tts.prefsPath`). Campos almacenados:

-   `enabled`
-   `provider`
-   `maxLength` (umbral de resumen; predeterminado 1500 caracteres)
-   `summarize` (predeterminado `true`)

Estos anulan `messages.tts.*` para ese host.

## Formatos de salida (fijos)

-   **Telegram**: Nota de voz Opus (`opus_48000_64` de ElevenLabs, `opus` de OpenAI).
    -   48kHz / 64kbps es un buen equilibrio para notas de voz y es necesario para la burbuja redonda.
-   **Otros canales**: MP3 (`mp3_44100_128` de ElevenLabs, `mp3` de OpenAI).
    -   44.1kHz / 128kbps es el equilibrio predeterminado para la claridad del habla.
-   **Edge TTS**: usa `edge.outputFormat` (predeterminado `audio-24khz-48kbitrate-mono-mp3`).
    -   `node-edge-tts` acepta un `outputFormat`, pero no todos los formatos están disponibles en el servicio Edge. citeturn2search0
    -   Los valores de formato de salida siguen los formatos de salida de Microsoft Speech (incluyendo Ogg/WebM Opus). citeturn1search0
    -   `sendVoice` de Telegram acepta OGG/MP3/M4A; usa OpenAI/ElevenLabs si necesitas notas de voz Opus garantizadas. citeturn1search1
    -   Si falla el formato de salida de Edge configurado, OpenClaw reintenta con MP3.

Los formatos de OpenAI/ElevenLabs son fijos; Telegram espera Opus para la UX de nota de voz.

## Comportamiento del TTS automático

Cuando está habilitado, OpenClaw:

-   omite TTS si la respuesta ya contiene medios o una directiva `MEDIA:`.
-   omite respuestas muy cortas (< 10 caracteres).
-   resume respuestas largas cuando está habilitado usando `agents.defaults.model.primary` (o `summaryModel`).
-   adjunta el audio generado a la respuesta.

Si la respuesta excede `maxLength` y el resumen está desactivado (o no hay clave API para el modelo de resumen), se omite el audio y se envía la respuesta de texto normal.

## Diagrama de flujo

```
Respuesta -> ¿TTS habilitado?
  no  -> enviar texto
  sí  -> ¿tiene medios / MEDIA: / corta?
          sí -> enviar texto
          no  -> ¿longitud > límite?
                   no  -> TTS -> adjuntar audio
                   sí  -> ¿resumen habilitado?
                            no  -> enviar texto
                            sí  -> resumir (summaryModel o agents.defaults.model.primary)
                                      -> TTS -> adjuntar audio
```

## Uso de comandos de barra

Hay un solo comando: `/tts`. Consulta [Comandos de barra](./tools/slash-commands.md) para detalles de habilitación. Nota de Discord: `/tts` es un comando integrado de Discord, por lo que OpenClaw registra `/voice` como el comando nativo allí. El texto `/tts ...` aún funciona.

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

Notas:

-   Los comandos requieren un remitente autorizado (las reglas de lista de permitidos/propietario aún aplican).
-   `commands.text` o el registro de comandos nativos debe estar habilitado.
-   `off|always|inbound|tagged` son interruptores por sesión (`/tts on` es un alias para `/tts always`).
-   `limit` y `summary` se almacenan en preferencias locales, no en la configuración principal.
-   `/tts audio` genera una respuesta de audio única (no activa el TTS).

## Herramienta del agente

La herramienta `tts` convierte texto a voz y devuelve una ruta `MEDIA:`. Cuando el resultado es compatible con Telegram, la herramienta incluye `[[audio_as_voice]]` para que Telegram envíe una burbuja de voz.

## RPC de Gateway

Métodos de Gateway:

-   `tts.status`
-   `tts.enable`
-   `tts.disable`
-   `tts.convert`
-   `tts.setProvider`
-   `tts.providers`

[Comando de Ubicación](./nodes/location-command.md)

---