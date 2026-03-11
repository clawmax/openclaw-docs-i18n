

  Medios y dispositivos

  
# Comprensión de Medios

OpenClaw puede **resumir medios entrantes** (imagen/audio/video) antes de que se ejecute el pipeline de respuesta. Detecta automáticamente cuando hay herramientas locales o claves de proveedor disponibles, y puede desactivarse o personalizarse. Si la comprensión está desactivada, los modelos aún reciben los archivos/URLs originales como de costumbre.

## Objetivos

-   Opcional: pre-digerir medios entrantes en texto corto para un enrutamiento más rápido y un mejor análisis de comandos.
-   Preservar la entrega del medio original al modelo (siempre).
-   Soporte para **APIs de proveedores** y **respaldos CLI**.
-   Permitir múltiples modelos con respaldo ordenado (error/tamaño/tiempo de espera).

## Comportamiento de alto nivel

1.  Recopilar adjuntos entrantes (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2.  Para cada capacidad habilitada (imagen/audio/video), seleccionar adjuntos según la política (por defecto: **el primero**).
3.  Elegir la primera entrada de modelo elegible (tamaño + capacidad + autenticación).
4.  Si un modelo falla o el medio es demasiado grande, **recurrir a la siguiente entrada**.
5.  En caso de éxito:
    -   `Body` se convierte en un bloque `[Image]`, `[Audio]`, o `[Video]`.
    -   El audio establece `{{Transcript}}`; el análisis de comandos usa el texto del pie de foto cuando está presente, de lo contrario la transcripción.
    -   Los pies de foto se conservan como `User text:` dentro del bloque.

Si la comprensión falla o está desactivada, **el flujo de respuesta continúa** con el cuerpo original + adjuntos.

## Resumen de configuración

`tools.media` soporta **modelos compartidos** más anulaciones por capacidad:

-   `tools.media.models`: lista de modelos compartidos (usar `capabilities` para restringir).
-   `tools.media.image` / `tools.media.audio` / `tools.media.video`:
    -   valores por defecto (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
    -   anulaciones de proveedor (`baseUrl`, `headers`, `providerOptions`)
    -   opciones de audio Deepgram vía `tools.media.audio.providerOptions.deepgram`
    -   controles de eco de transcripción de audio (`echoTranscript`, por defecto `false`; `echoFormat`)
    -   lista **opcional de `models` por capacidad** (preferida antes de los modelos compartidos)
    -   política de `attachments` (`mode`, `maxAttachments`, `prefer`)
    -   `scope` (restricción opcional por canal/tipoChat/clave de sesión)
-   `tools.media.concurrency`: máximo de ejecuciones concurrentes por capacidad (por defecto **2**).

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

### Entradas de modelo

Cada entrada `models[]` puede ser de **proveedor** o **CLI**:

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

Las plantillas CLI también pueden usar:

-   `{{MediaDir}}` (directorio que contiene el archivo multimedia)
-   `{{OutputDir}}` (directorio temporal creado para esta ejecución)
-   `{{OutputBase}}` (ruta base del archivo temporal, sin extensión)

## Valores por defecto y límites

Valores por defecto recomendados:

-   `maxChars`: **500** para imagen/video (corto, apto para comandos)
-   `maxChars`: **sin establecer** para audio (transcripción completa a menos que establezcas un límite)
-   `maxBytes`:
    -   imagen: **10MB**
    -   audio: **20MB**
    -   video: **50MB**

Reglas:

-   Si el medio excede `maxBytes`, ese modelo se omite y se **prueba el siguiente modelo**.
-   Los archivos de audio más pequeños de **1024 bytes** se tratan como vacíos/corruptos y se omiten antes de la transcripción por proveedor/CLI.
-   Si el modelo devuelve más de `maxChars`, la salida se recorta.
-   `prompt` por defecto es una simple "Describe el ." más la guía de `maxChars` (solo imagen/video).
-   Si `.enabled: true` pero no hay modelos configurados, OpenClaw intenta usar el **modelo de respuesta activo** cuando su proveedor soporta la capacidad.

### Detección automática de comprensión de medios (por defecto)

Si `tools.media..enabled` **no** está establecido en `false` y no has configurado modelos, OpenClaw detecta automáticamente en este orden y **se detiene en la primera opción que funcione**:

1.  **CLIs locales** (solo audio; si están instalados)
    -   `sherpa-onnx-offline` (requiere `SHERPA_ONNX_MODEL_DIR` con encoder/decoder/joiner/tokens)
    -   `whisper-cli` (`whisper-cpp`; usa `WHISPER_CPP_MODEL` o el modelo tiny incluido)
    -   `whisper` (CLI de Python; descarga modelos automáticamente)
2.  **CLI Gemini** (`gemini`) usando `read_many_files`
3.  **Claves de proveedor**
    -   Audio: OpenAI → Groq → Deepgram → Google
    -   Imagen: OpenAI → Anthropic → Google → MiniMax
    -   Video: Google

Para desactivar la detección automática, establece:

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

Nota: La detección de binarios es de mejor esfuerzo en macOS/Linux/Windows; asegúrate de que el CLI esté en `PATH` (expandimos `~`), o establece un modelo CLI explícito con una ruta de comando completa.

### Soporte de entorno proxy (modelos de proveedor)

Cuando la comprensión de medios **audio** y **video** basada en proveedor está habilitada, OpenClaw respeta las variables de entorno estándar de proxy saliente para las llamadas HTTP del proveedor:

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

Si no se establecen variables de entorno de proxy, la comprensión de medios usa salida directa. Si el valor del proxy está mal formado, OpenClaw registra una advertencia y recurre a la obtención directa.

## Capacidades (opcional)

Si estableces `capabilities`, la entrada solo se ejecuta para esos tipos de medios. Para listas compartidas, OpenClaw puede inferir valores por defecto:

-   `openai`, `anthropic`, `minimax`: **imagen**
-   `google` (API Gemini): **imagen + audio + video**
-   `groq`: **audio**
-   `deepgram`: **audio**

Para entradas CLI, **establece `capabilities` explícitamente** para evitar coincidencias sorpresa. Si omites `capabilities`, la entrada es elegible para la lista en la que aparece.

## Matriz de soporte de proveedores (integraciones de OpenClaw)

| Capacidad | Integración de proveedor | Notas |
| --- | --- | --- |
| Imagen | OpenAI / Anthropic / Google / otros vía `pi-ai` | Cualquier modelo con capacidad de imagen en el registro funciona. |
| Audio | OpenAI, Groq, Deepgram, Google, Mistral | Transcripción por proveedor (Whisper/Deepgram/Gemini/Voxtral). |
| Video | Google (API Gemini) | Comprensión de video por proveedor. |

## Guía de selección de modelos

-   Prefiere el modelo más potente de última generación disponible para cada capacidad de medios cuando la calidad y la seguridad importen.
-   Para agentes con herramientas que manejan entradas no confiables, evita modelos de medios más antiguos/débiles.
-   Mantén al menos un respaldo por capacidad para disponibilidad (modelo de calidad + modelo más rápido/económico).
-   Los respaldos CLI (`whisper-cli`, `whisper`, `gemini`) son útiles cuando las APIs de proveedor no están disponibles.
-   Nota sobre `parakeet-mlx`: con `--output-dir`, OpenClaw lee `<output-dir>/<media-basename>.txt` cuando el formato de salida es `txt` (o no especificado); los formatos no-`txt` recurren a stdout.

## Política de adjuntos

`attachments` por capacidad controla qué adjuntos se procesan:

-   `mode`: `first` (por defecto) o `all`
-   `maxAttachments`: limitar el número procesado (por defecto **1**)
-   `prefer`: `first`, `last`, `path`, `url`

Cuando `mode: "all"`, las salidas se etiquetan como `[Image 1/2]`, `[Audio 2/2]`, etc.

## Ejemplos de configuración

### 1) Lista de modelos compartidos + anulaciones

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

### 2) Solo Audio + Video (imagen desactivada)

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

### 3) Comprensión de imagen opcional

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

### 4) Entrada multimodal única (capacidades explícitas)

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

## Salida de estado

Cuando se ejecuta la comprensión de medios, `/status` incluye una línea de resumen corta:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Esto muestra los resultados por capacidad y el proveedor/modelo elegido cuando es aplicable.

## Notas

-   La comprensión es de **mejor esfuerzo**. Los errores no bloquean las respuestas.
-   Los adjuntos aún se pasan a los modelos incluso cuando la comprensión está desactivada.
-   Usa `scope` para limitar dónde se ejecuta la comprensión (ej. solo mensajes directos).

## Documentación relacionada

-   [Configuración](../gateway/configuration.md)
-   [Soporte de Imágenes y Medios](./images.md)

[Solución de Problemas de Nodos](./troubleshooting.md)[Soporte de Imágenes y Medios](./images.md)

---