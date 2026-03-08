

  Medios y dispositivos

  
# Audio y Notas de Voz

## Qué funciona

-   **Comprensión de medios (audio)**: Si la comprensión de audio está habilitada (o se detecta automáticamente), OpenClaw:
    1.  Localiza el primer archivo adjunto de audio (ruta local o URL) y lo descarga si es necesario.
    2.  Aplica `maxBytes` antes de enviar a cada entrada del modelo.
    3.  Ejecuta la primera entrada de modelo elegible en orden (proveedor o CLI).
    4.  Si falla o se omite (tamaño/tiempo de espera), intenta con la siguiente entrada.
    5.  En caso de éxito, reemplaza `Body` con un bloque `[Audio]` y establece `{{Transcript}}`.
-   **Análisis de comandos**: Cuando la transcripción tiene éxito, `CommandBody`/`RawBody` se establecen en la transcripción para que los comandos de barra inclinada sigan funcionando.
-   **Registro detallado**: En `--verbose`, registramos cuándo se ejecuta la transcripción y cuándo reemplaza el cuerpo.

## Detección automática (predeterminado)

Si **no configuras modelos** y `tools.media.audio.enabled` **no** está establecido en `false`, OpenClaw detecta automáticamente en este orden y se detiene en la primera opción que funcione:

1.  **CLIs locales** (si están instalados)
    -   `sherpa-onnx-offline` (requiere `SHERPA_ONNX_MODEL_DIR` con encoder/decoder/joiner/tokens)
    -   `whisper-cli` (de `whisper-cpp`; usa `WHISPER_CPP_MODEL` o el modelo tiny incluido)
    -   `whisper` (CLI de Python; descarga modelos automáticamente)
2.  **CLI de Gemini** (`gemini`) usando `read_many_files`
3.  **Claves de proveedor** (OpenAI → Groq → Deepgram → Google)

Para deshabilitar la detección automática, establece `tools.media.audio.enabled: false`. Para personalizar, establece `tools.media.audio.models`. Nota: La detección de binarios es de mejor esfuerzo en macOS/Linux/Windows; asegúrate de que el CLI esté en `PATH` (expandimos `~`), o establece un modelo CLI explícito con una ruta de comando completa.

## Ejemplos de configuración

### Proveedor + respaldo CLI (OpenAI + Whisper CLI)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### Solo proveedor con control de alcance

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### Solo proveedor (Deepgram)

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

### Solo proveedor (Mistral Voxtral)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

### Eco de transcripción al chat (opt-in)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        echoTranscript: true, // el valor predeterminado es false
        echoFormat: '📝 "{transcript}"', // opcional, admite {transcript}
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

## Notas y límites

-   La autenticación del proveedor sigue el orden estándar de autenticación de modelos (perfiles de auth, variables de entorno, `models.providers.*.apiKey`).
-   Deepgram recoge `DEEPGRAM_API_KEY` cuando se usa `provider: "deepgram"`.
-   Detalles de configuración de Deepgram: [Deepgram (transcripción de audio)](../providers/deepgram.md).
-   Detalles de configuración de Mistral: [Mistral](../providers/mistral.md).
-   Los proveedores de audio pueden anular `baseUrl`, `headers` y `providerOptions` a través de `tools.media.audio`.
-   El límite de tamaño predeterminado es 20MB (`tools.media.audio.maxBytes`). El audio que exceda el tamaño se omite para ese modelo y se intenta con la siguiente entrada.
-   Los archivos de audio pequeños/vacíos por debajo de 1024 bytes se omiten antes de la transcripción del proveedor/CLI.
-   El `maxChars` predeterminado para audio es **no establecido** (transcripción completa). Establece `tools.media.audio.maxChars` o `maxChars` por entrada para recortar la salida.
-   El valor predeterminado automático de OpenAI es `gpt-4o-mini-transcribe`; establece `model: "gpt-4o-transcribe"` para mayor precisión.
-   Usa `tools.media.audio.attachments` para procesar múltiples notas de voz (`mode: "all"` + `maxAttachments`).
-   La transcripción está disponible para las plantillas como `{{Transcript}}`.
-   `tools.media.audio.echoTranscript` está desactivado por defecto; habilítalo para enviar una confirmación de transcripción al chat de origen antes del procesamiento del agente.
-   `tools.media.audio.echoFormat` personaliza el texto del eco (marcador de posición: `{transcript}`).
-   La salida estándar del CLI tiene un límite (5MB); mantén la salida del CLI concisa.

### Soporte de entorno proxy

La transcripción de audio basada en proveedor respeta las variables de entorno de proxy de salida estándar:

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

Si no se establecen variables de entorno de proxy, se usa salida directa. Si la configuración del proxy está mal formada, OpenClaw registra una advertencia y recurre a la obtención directa.

## Detección de Menciones en Grupos

Cuando `requireMention: true` está establecido para un chat grupal, OpenClaw ahora transcribe el audio **antes** de verificar las menciones. Esto permite que las notas de voz se procesen incluso cuando contienen menciones. **Cómo funciona:**

1.  Si un mensaje de voz no tiene cuerpo de texto y el grupo requiere menciones, OpenClaw realiza una transcripción de "pre-vuelo".
2.  La transcripción se verifica en busca de patrones de mención (ej., `@BotName`, desencadenadores de emoji).
3.  Si se encuentra una mención, el mensaje procede a través de la tubería de respuesta completa.
4.  La transcripción se usa para la detección de menciones para que las notas de voz puedan pasar la compuerta de mención.

**Comportamiento de respaldo:**

-   Si la transcripción falla durante el pre-vuelo (tiempo de espera, error de API, etc.), el mensaje se procesa basándose en la detección de menciones solo de texto.
-   Esto asegura que los mensajes mixtos (texto + audio) nunca se descarten incorrectamente.

**Exclusión por grupo/tema de Telegram:**

-   Establece `channels.telegram.groups..disableAudioPreflight: true` para omitir las comprobaciones de menciones de transcripción de pre-vuelo para ese grupo.
-   Establece `channels.telegram.groups..topics..disableAudioPreflight` para anular por tema (`true` para omitir, `false` para forzar habilitación).
-   El valor predeterminado es `false` (pre-vuelo habilitado cuando coinciden las condiciones de compuerta de mención).

**Ejemplo:** Un usuario envía una nota de voz diciendo "Oye @Claude, ¿cómo está el clima?" en un grupo de Telegram con `requireMention: true`. La nota de voz se transcribe, se detecta la mención y el agente responde.

## Advertencias

-   Las reglas de alcance usan "primera coincidencia gana". `chatType` se normaliza a `direct`, `group` o `room`.
-   Asegúrate de que tu CLI salga con código 0 e imprima texto plano; JSON necesita ser procesado vía `jq -r .text`.
-   Para `parakeet-mlx`, si pasas `--output-dir`, OpenClaw lee `<output-dir>/<media-basename>.txt` cuando `--output-format` es `txt` (u omitido); los formatos de salida que no son `txt` recurren al análisis de la salida estándar.
-   Mantén los tiempos de espera razonables (`timeoutSeconds`, predeterminado 60s) para evitar bloquear la cola de respuestas.
-   La transcripción de pre-vuelo solo procesa el **primer** archivo adjunto de audio para la detección de menciones. El audio adicional se procesa durante la fase principal de comprensión de medios.

[Soporte de Imágenes y Medios](./images.md)[Captura de Cámara](./camera.md)