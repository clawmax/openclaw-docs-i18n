

  Referencia técnica

  
# Uso y Costos de la API

Este documento enumera las **características que pueden invocar claves API** y dónde aparecen sus costos. Se centra en las características de OpenClaw que pueden generar uso del proveedor o llamadas API de pago.

## Dónde aparecen los costos (chat + CLI)

**Instantánea de costo por sesión**

-   `/status` muestra el modelo de la sesión actual, el uso de contexto y los tokens de la última respuesta.
-   Si el modelo usa **autenticación por clave API**, `/status` también muestra el **costo estimado** de la última respuesta.

**Pie de página de costo por mensaje**

-   `/usage full` agrega un pie de página de uso a cada respuesta, incluyendo el **costo estimado** (solo para clave API).
-   `/usage tokens` muestra solo tokens; los flujos OAuth ocultan el costo en dólares.

**Ventanas de uso en CLI (cuotas del proveedor)**

-   `openclaw status --usage` y `openclaw channels list` muestran las **ventanas de uso** del proveedor (instantáneas de cuota, no costos por mensaje).

Consulta [Uso y costos de tokens](./token-use.md) para más detalles y ejemplos.

## Cómo se descubren las claves

OpenClaw puede obtener credenciales de:

-   **Perfiles de autenticación** (por agente, almacenados en `auth-profiles.json`).
-   **Variables de entorno** (ej. `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`).
-   **Configuración** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`, `memorySearch.*`, `talk.apiKey`).
-   **Habilidades** (`skills.entries..apiKey`) que pueden exportar claves al entorno del proceso de la habilidad.

## Características que pueden gastar claves

### 1) Respuestas del modelo principal (chat + herramientas)

Cada respuesta o llamada a herramienta usa el **proveedor del modelo actual** (OpenAI, Anthropic, etc.). Esta es la fuente principal de uso y costo. Consulta [Modelos](../providers/models.md) para la configuración de precios y [Uso y costos de tokens](./token-use.md) para la visualización.

### 2) Comprensión de medios (audio/imagen/video)

Los medios entrantes pueden ser resumidos/transcritos antes de que se ejecute la respuesta. Esto usa las APIs del modelo/proveedor.

-   Audio: OpenAI / Groq / Deepgram (ahora **habilitado automáticamente** cuando existen claves).
-   Imagen: OpenAI / Anthropic / Google.
-   Video: Google.

Consulta [Comprensión de medios](../nodes/media-understanding.md).

### 3) Incrustaciones de memoria + búsqueda semántica

La búsqueda semántica de memoria usa **APIs de incrustación** cuando se configura para proveedores remotos:

-   `memorySearch.provider = "openai"` → incrustaciones de OpenAI
-   `memorySearch.provider = "gemini"` → incrustaciones de Gemini
-   `memorySearch.provider = "voyage"` → incrustaciones de Voyage
-   `memorySearch.provider = "mistral"` → incrustaciones de Mistral
-   `memorySearch.provider = "ollama"` → incrustaciones de Ollama (local/autoalojado; típicamente sin facturación de API alojada)
-   Retroceso opcional a un proveedor remoto si fallan las incrustaciones locales

Puedes mantenerlo local con `memorySearch.provider = "local"` (sin uso de API). Consulta [Memoria](../concepts/memory.md).

### 4) Herramienta de búsqueda web

`web_search` usa claves API y puede incurrir en cargos de uso dependiendo de tu proveedor:

-   **API de Búsqueda de Perplexity**: `PERPLEXITY_API_KEY`
-   **API de Búsqueda de Brave**: `BRAVE_API_KEY` o `tools.web.search.apiKey`
-   **Gemini (Búsqueda de Google)**: `GEMINI_API_KEY`
-   **Grok (xAI)**: `XAI_API_KEY`
-   **Kimi (Moonshot)**: `KIMI_API_KEY` o `MOONSHOT_API_KEY`

Consulta [Herramientas web](../tools/web.md).

### 5) Herramienta de obtención web (Firecrawl)

`web_fetch` puede llamar a **Firecrawl** cuando hay una clave API presente:

-   `FIRECRAWL_API_KEY` o `tools.web.fetch.firecrawl.apiKey`

Si Firecrawl no está configurado, la herramienta recurre a una obtención directa + legibilidad (sin API de pago). Consulta [Herramientas web](../tools/web.md).

### 6) Instantáneas de uso del proveedor (estado/salud)

Algunos comandos de estado llaman a **endpoints de uso del proveedor** para mostrar ventanas de cuota o estado de autenticación. Estas son típicamente llamadas de bajo volumen pero aún acceden a las APIs del proveedor:

-   `openclaw status --usage`
-   `openclaw models status --json`

Consulta [CLI de Modelos](../cli/models.md).

### 7) Resumen de protección de compactación

La protección de compactación puede resumir el historial de la sesión usando el **modelo actual**, lo que invoca las APIs del proveedor cuando se ejecuta. Consulta [Gestión de sesión + compactación](./session-management-compaction.md).

### 8) Escaneo / sondeo de modelos

`openclaw models scan` puede sondear modelos de OpenRouter y usa `OPENROUTER_API_KEY` cuando el sondeo está habilitado. Consulta [CLI de Modelos](../cli/models.md).

### 9) Hablar (voz)

El modo de hablar puede invocar **ElevenLabs** cuando está configurado:

-   `ELEVENLABS_API_KEY` o `talk.apiKey`

Consulta [Modo de hablar](../nodes/talk.md).

### 10) Habilidades (APIs de terceros)

Las habilidades pueden almacenar `apiKey` en `skills.entries..apiKey`. Si una habilidad usa esa clave para APIs externas, puede incurrir en costos según el proveedor de la habilidad. Consulta [Habilidades](../tools/skills.md).

[Almacenamiento en caché de prompts](./prompt-caching.md)[Higiene de transcripciones](./transcript-hygiene.md)