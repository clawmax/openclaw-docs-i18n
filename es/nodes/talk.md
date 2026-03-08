

  Medios y dispositivos

  
# Modo Habla

El Modo Habla es un bucle de conversación de voz continua:

1.  Escuchar el habla
2.  Enviar la transcripción al modelo (sesión principal, chat.send)
3.  Esperar la respuesta
4.  Reproducirla mediante ElevenLabs (reproducción en streaming)

## Comportamiento (macOS)

-   **Superposición siempre activa** mientras el Modo Habla está habilitado.
-   Transiciones de fase: **Escuchando → Pensando → Hablando**.
-   Tras una **pausa corta** (ventana de silencio), se envía la transcripción actual.
-   Las respuestas se **escriben en WebChat** (igual que al escribir).
-   **Interrumpir al hablar** (activado por defecto): si el usuario comienza a hablar mientras el asistente está hablando, detenemos la reproducción y registramos la marca de tiempo de interrupción para el siguiente mensaje.

## Directivas de voz en las respuestas

El asistente puede prefijar su respuesta con una **única línea JSON** para controlar la voz:

```json
{ "voice": "<voice-id>", "once": true }
```

Reglas:

-   Solo la primera línea no vacía.
-   Se ignoran las claves desconocidas.
-   `once: true` se aplica solo a la respuesta actual.
-   Sin `once`, la voz se convierte en la nueva predeterminada para el Modo Habla.
-   La línea JSON se elimina antes de la reproducción TTS.

Claves admitidas:

-   `voice` / `voice_id` / `voiceId`
-   `model` / `model_id` / `modelId`
-   `speed`, `rate` (PPM), `stability`, `similarity`, `style`, `speakerBoost`
-   `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
-   `once`

## Configuración (~/.openclaw/openclaw.json)

```json
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

Valores predeterminados:

-   `interruptOnSpeech`: true
-   `voiceId`: recurre a `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID` (o a la primera voz de ElevenLabs cuando la clave API está disponible)
-   `modelId`: por defecto `eleven_v3` cuando no está configurado
-   `apiKey`: recurre a `ELEVENLABS_API_KEY` (o al perfil de shell del gateway si está disponible)
-   `outputFormat`: por defecto `pcm_44100` en macOS/iOS y `pcm_24000` en Android (configura `mp3_*` para forzar streaming MP3)

## Interfaz de usuario en macOS

-   Alternar en la barra de menú: **Hablar**
-   Pestaña de configuración: grupo **Modo Habla** (id de voz + alternar interrupción)
-   Superposición:
    -   **Escuchando**: nube pulsa con el nivel del micrófono
    -   **Pensando**: animación de hundimiento
    -   **Hablando**: anillos radiantes
    -   Clic en la nube: dejar de hablar
    -   Clic en X: salir del Modo Habla

## Notas

-   Requiere permisos de Voz + Micrófono.
-   Utiliza `chat.send` con la clave de sesión `main`.
-   TTS utiliza la API de streaming de ElevenLabs con `ELEVENLABS_API_KEY` y reproducción incremental en macOS/iOS/Android para menor latencia.
-   `stability` para `eleven_v3` se valida a `0.0`, `0.5`, o `1.0`; otros modelos aceptan `0..1`.
-   `latency_tier` se valida a `0..4` cuando se configura.
-   Android admite los formatos de salida `pcm_16000`, `pcm_22050`, `pcm_24000`, y `pcm_44100` para streaming de baja latencia con AudioTrack.

[Captura de Cámara](./camera.md)[Activación por Voz](./voicewake.md)