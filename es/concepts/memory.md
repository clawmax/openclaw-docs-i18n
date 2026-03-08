

  Sesiones y memoria

  
# Memoria

La memoria de OpenClaw es **Markdown plano en el espacio de trabajo del agente**. Los archivos son la fuente de verdad; el modelo solo "recuerda" lo que se escribe en el disco. Las herramientas de búsqueda de memoria las proporciona el complemento de memoria activo (predeterminado: `memory-core`). Deshabilita los complementos de memoria con `plugins.slots.memory = "none"`.

## Archivos de memoria (Markdown)

El diseño predeterminado del espacio de trabajo utiliza dos capas de memoria:

-   `memory/YYYY-MM-DD.md`
    -   Registro diario (solo anexar).
    -   Se lee hoy + ayer al inicio de la sesión.
-   `MEMORY.md` (opcional)
    -   Memoria a largo plazo curada.
    -   **Solo se carga en la sesión principal y privada** (nunca en contextos grupales).

Estos archivos residen bajo el espacio de trabajo (`agents.defaults.workspace`, predeterminado `~/.openclaw/workspace`). Consulta [Espacio de trabajo del agente](./agent-workspace.md) para ver el diseño completo.

## Herramientas de memoria

OpenClaw expone dos herramientas orientadas al agente para estos archivos Markdown:

-   `memory_search` — recuperación semántica sobre fragmentos indexados.
-   `memory_get` — lectura dirigida de un archivo/rango de líneas Markdown específico.

`memory_get` ahora **degradará suavemente cuando un archivo no exista** (por ejemplo, el registro diario de hoy antes de la primera escritura). Tanto el administrador integrado como el backend QMD devuelven `{ text: "", path }` en lugar de lanzar `ENOENT`, por lo que los agentes pueden manejar "nada registrado aún" y continuar su flujo de trabajo sin envolver la llamada a la herramienta en lógica try/catch.

## Cuándo escribir en memoria

-   Las decisiones, preferencias y hechos duraderos van a `MEMORY.md`.
-   Las notas del día a día y el contexto en ejecución van a `memory/YYYY-MM-DD.md`.
-   Si alguien dice "recuerda esto", escríbelo (no lo mantengas en RAM).
-   Esta área aún está evolucionando. Ayuda recordar al modelo que almacene recuerdos; él sabrá qué hacer.
-   Si quieres que algo permanezca, **pídele al bot que lo escriba** en la memoria.

## Vaciado automático de memoria (ping pre-compactación)

Cuando una sesión está **cerca de la auto-compactación**, OpenClaw activa un **turno silencioso y agéntico** que recuerda al modelo que escriba memoria duradera **antes** de que el contexto se compacte. Los mensajes predeterminados dicen explícitamente que el modelo *puede responder*, pero normalmente `NO_REPLY` es la respuesta correcta para que el usuario nunca vea este turno. Esto se controla mediante `agents.defaults.compaction.memoryFlush`:

```json
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

Detalles:

-   **Umbral suave**: el vaciado se activa cuando la estimación de tokens de la sesión cruza `contextWindow - reserveTokensFloor - softThresholdTokens`.
-   **Silencioso** por defecto: los mensajes incluyen `NO_REPLY` para que no se entregue nada.
-   **Dos mensajes**: un mensaje de usuario más un mensaje del sistema anexan el recordatorio.
-   **Un vaciado por ciclo de compactación** (rastreado en `sessions.json`).
-   **El espacio de trabajo debe ser escribible**: si la sesión se ejecuta en un entorno aislado con `workspaceAccess: "ro"` o `"none"`, se omite el vaciado.

Para el ciclo de vida completo de la compactación, consulta [Gestión de sesiones + compactación](../reference/session-management-compaction.md).

## Búsqueda de memoria vectorial

OpenClaw puede construir un pequeño índice vectorial sobre `MEMORY.md` y `memory/*.md` para que las consultas semánticas puedan encontrar notas relacionadas incluso cuando la redacción difiera. Valores predeterminados:

-   Habilitado por defecto.
-   Observa los archivos de memoria en busca de cambios (con rebote).
-   Configura la búsqueda de memoria bajo `agents.defaults.memorySearch` (no en el nivel superior `memorySearch`).
-   Usa incrustaciones remotas por defecto. Si `memorySearch.provider` no está configurado, OpenClaw selecciona automáticamente:
    1.  `local` si se configura un `memorySearch.local.modelPath` y el archivo existe.
    2.  `openai` si se puede resolver una clave de OpenAI.
    3.  `gemini` si se puede resolver una clave de Gemini.
    4.  `voyage` si se puede resolver una clave de Voyage.
    5.  `mistral` si se puede resolver una clave de Mistral.
    6.  De lo contrario, la búsqueda de memoria permanece deshabilitada hasta que se configure.
-   El modo local usa node-llama-cpp y puede requerir `pnpm approve-builds`.
-   Usa sqlite-vec (cuando está disponible) para acelerar la búsqueda vectorial dentro de SQLite.
-   `memorySearch.provider = "ollama"` también es compatible para incrustaciones locales/autoalojadas de Ollama (`/api/embeddings`), pero no se selecciona automáticamente.

Las incrustaciones remotas **requieren** una clave API para el proveedor de incrustaciones. OpenClaw resuelve las claves desde perfiles de autenticación, `models.providers.*.apiKey` o variables de entorno. El OAuth de Codex solo cubre chat/completions y **no** satisface las incrustaciones para la búsqueda de memoria. Para Gemini, usa `GEMINI_API_KEY` o `models.providers.google.apiKey`. Para Voyage, usa `VOYAGE_API_KEY` o `models.providers.voyage.apiKey`. Para Mistral, usa `MISTRAL_API_KEY` o `models.providers.mistral.apiKey`. Ollama normalmente no requiere una clave API real (un marcador de posición como `OLLAMA_API_KEY=ollama-local` es suficiente cuando lo requiere la política local). Al usar un endpoint personalizado compatible con OpenAI, configura `memorySearch.remote.apiKey` (y opcionalmente `memorySearch.remote.headers`).

### Backend QMD (experimental)

Configura `memory.backend = "qmd"` para intercambiar el indexador SQLite integrado por [QMD](https://github.com/tobi/qmd): un sidecar de búsqueda local-first que combina BM25 + vectores + reranking. Markdown sigue siendo la fuente de verdad; OpenClaw ejecuta QMD externamente para la recuperación. Puntos clave: **Requisitos previos**

-   Deshabilitado por defecto. Opta por configuración (`memory.backend = "qmd"`).
-   Instala la CLI de QMD por separado (`bun install -g https://github.com/tobi/qmd` o descarga una versión) y asegúrate de que el binario `qmd` esté en el `PATH` de la puerta de enlace.
-   QMD necesita una compilación de SQLite que permita extensiones (`brew install sqlite` en macOS).
-   QMD se ejecuta completamente de forma local a través de Bun + `node-llama-cpp` y descarga automáticamente modelos GGUF de HuggingFace en el primer uso (no se requiere un daemon de Ollama separado).
-   La puerta de enlace ejecuta QMD en un directorio XDG autocontenido bajo `~/.openclaw/agents//qmd/` configurando `XDG_CONFIG_HOME` y `XDG_CACHE_HOME`.
-   Soporte de SO: macOS y Linux funcionan de inmediato una vez instalados Bun + SQLite. Windows se admite mejor a través de WSL2.

**Cómo se ejecuta el sidecar**

-   La puerta de enlace escribe un directorio QMD autocontenido bajo `~/.openclaw/agents//qmd/` (config + caché + base de datos sqlite).
-   Las colecciones se crean mediante `qmd collection add` desde `memory.qmd.paths` (más los archivos de memoria predeterminados del espacio de trabajo), luego `qmd update` + `qmd embed` se ejecutan al inicio y en un intervalo configurable (`memory.qmd.update.interval`, predeterminado 5 m).
-   La puerta de enlace ahora inicializa el administrador QMD al inicio, por lo que los temporizadores de actualización periódica están armados incluso antes de la primera llamada a `memory_search`.
-   La actualización de inicio ahora se ejecuta en segundo plano por defecto para que el inicio del chat no se bloquee; configura `memory.qmd.update.waitForBootSync = true` para mantener el comportamiento de bloqueo anterior.
-   Las búsquedas se ejecutan mediante `memory.qmd.searchMode` (predeterminado `qmd search --json`; también admite `vsearch` y `query`). Si el modo seleccionado rechaza banderas en tu compilación de QMD, OpenClaw lo reintenta con `qmd query`. Si QMD falla o falta el binario, OpenClaw automáticamente vuelve al administrador SQLite integrado para que las herramientas de memoria sigan funcionando.
-   OpenClaw no expone la configuración del tamaño de lote de incrustación de QMD hoy; el comportamiento por lotes lo controla QMD mismo.
-   **La primera búsqueda puede ser lenta**: QMD puede descargar modelos GGUF locales (reranker/expansión de consulta) en la primera ejecución de `qmd query`.
    -   OpenClaw configura `XDG_CONFIG_HOME`/`XDG_CACHE_HOME` automáticamente cuando ejecuta QMD.
    -   Si quieres descargar modelos manualmente de antemano (y calentar el mismo índice que usa OpenClaw), ejecuta una consulta única con los directorios XDG del agente. El estado de QMD de OpenClaw reside bajo tu **directorio de estado** (predeterminado `~/.openclaw`). Puedes apuntar `qmd` al mismo índice exacto exportando las mismas variables XDG que usa OpenClaw:
        
        Copiar
        
        ```bash
        # Elige el mismo directorio de estado que usa OpenClaw
        STATE_DIR="${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
        
        export XDG_CONFIG_HOME="$STATE_DIR/agents/main/qmd/xdg-config"
        export XDG_CACHE_HOME="$STATE_DIR/agents/main/qmd/xdg-cache"
        
        # (Opcional) fuerza una actualización del índice + incrustaciones
        qmd update
        qmd embed
        
        # Calienta / activa las descargas de modelos por primera vez
        qmd query "test" -c memory-root --json >/dev/null 2>&1
        ```
        

**Superficie de configuración (`memory.qmd.*`)**

-   `command` (predeterminado `qmd`): anula la ruta del ejecutable.
-   `searchMode` (predeterminado `search`): elige qué comando de QMD respalda `memory_search` (`search`, `vsearch`, `query`).
-   `includeDefaultMemory` (predeterminado `true`): indexa automáticamente `MEMORY.md` + `memory/**/*.md`.
-   `paths[]`: agrega directorios/archivos adicionales (`path`, opcional `pattern`, opcional `name` estable).
-   `sessions`: opta por indexación JSONL de sesiones (`enabled`, `retentionDays`, `exportDir`).
-   `update`: controla la cadencia de actualización y la ejecución de mantenimiento: (`interval`, `debounceMs`, `onBoot`, `waitForBootSync`, `embedInterval`, `commandTimeoutMs`, `updateTimeoutMs`, `embedTimeoutMs`).
-   `limits`: limita la carga útil de recuperación (`maxResults`, `maxSnippetChars`, `maxInjectedChars`, `timeoutMs`).
-   `scope`: mismo esquema que [`session.sendPolicy`](../gateway/configuration.md#session). El valor predeterminado es solo MD (`deny` todos, `allow` chats directos); aflójalo para mostrar resultados de QMD en grupos/canales.
    -   `match.keyPrefix` coincide con la clave de sesión **normalizada** (en minúsculas, eliminando cualquier prefijo `agent::`). Ejemplo: `discord:channel:`.
    -   `match.rawKeyPrefix` coincide con la clave de sesión **cruda** (en minúsculas), incluyendo `agent::`. Ejemplo: `agent:main:discord:`.
    -   Legado: `match.keyPrefix: "agent:..."` todavía se trata como un prefijo de clave cruda, pero prefiere `rawKeyPrefix` para mayor claridad.
-   Cuando `scope` deniega una búsqueda, OpenClaw registra una advertencia con el `channel`/`chatType` derivado para que los resultados vacíos sean más fáciles de depurar.
-   Los fragmentos obtenidos fuera del espacio de trabajo aparecen como `qmd//<relative-path>` en los resultados de `memory_search`; `memory_get` entiende ese prefijo y lee desde la raíz de la colección QMD configurada.
-   Cuando `memory.qmd.sessions.enabled = true`, OpenClaw exporta transcripciones de sesión saneadas (turnos de Usuario/Asistente) a una colección QMD dedicada bajo `~/.openclaw/agents//qmd/sessions/`, para que `memory_search` pueda recordar conversaciones recientes sin tocar el índice SQLite integrado.
-   Los fragmentos de `memory_search` ahora incluyen un pie de página `Source: <path#line>` cuando `memory.citations` es `auto`/`on`; configura `memory.citations = "off"` para mantener los metadatos de ruta internos (el agente aún recibe la ruta para `memory_get`, pero el texto del fragmento omite el pie de página y el mensaje del sistema advierte al agente que no lo cite).

**Ejemplo**

```
memory: {
  backend: "qmd",
  citations: "auto",
  qmd: {
    includeDefaultMemory: true,
    update: { interval: "5m", debounceMs: 15000 },
    limits: { maxResults: 6, timeoutMs: 4000 },
    scope: {
      default: "deny",
      rules: [
        { action: "allow", match: { chatType: "direct" } },
        // Prefijo de clave de sesión normalizada (elimina `agent:<id>:`).
        { action: "deny", match: { keyPrefix: "discord:channel:" } },
        // Prefijo de clave de sesión cruda (incluye `agent:<id>:`).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ]
    },
    paths: [
      { name: "docs", path: "~/notes", pattern: "**/*.md" }
    ]
  }
}
```

**Citas y respaldo**

-   `memory.citations` se aplica independientemente del backend (`auto`/`on`/`off`).
-   Cuando `qmd` se ejecuta, etiquetamos `status().backend = "qmd"` para que los diagnósticos muestren qué motor sirvió los resultados. Si el subproceso QMD termina o la salida JSON no se puede analizar, el administrador de búsqueda registra una advertencia y devuelve el proveedor integrado (incrustaciones Markdown existentes) hasta que QMD se recupere.

### Rutas de memoria adicionales

Si deseas indexar archivos Markdown fuera del diseño predeterminado del espacio de trabajo, agrega rutas explícitas:

```
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

Notas:

-   Las rutas pueden ser absolutas o relativas al espacio de trabajo.
-   Los directorios se escanean recursivamente en busca de archivos `.md`.
-   Solo se indexan archivos Markdown.
-   Se ignoran los enlaces simbólicos (archivos o directorios).

### Incrustaciones Gemini (nativas)

Configura el proveedor a `gemini` para usar la API de incrustaciones de Gemini directamente:

```
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

Notas:

-   `remote.baseUrl` es opcional (predeterminado a la URL base de la API de Gemini).
-   `remote.headers` te permite agregar encabezados adicionales si es necesario.
-   Modelo predeterminado: `gemini-embedding-001`.

Si deseas usar un **endpoint personalizado compatible con OpenAI** (OpenRouter, vLLM o un proxy), puedes usar la configuración `remote` con el proveedor OpenAI:

```
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

Si no deseas configurar una clave API, usa `memorySearch.provider = "local"` o configura `memorySearch.fallback = "none"`. Respaldos:

-   `memorySearch.fallback` puede ser `openai`, `gemini`, `voyage`, `mistral`, `ollama`, `local` o `none`.
-   El proveedor de respaldo solo se usa cuando falla el proveedor de incrustaciones principal.

Indexación por lotes (OpenAI + Gemini + Voyage):

-   Deshabilitado por defecto. Configura `agents.defaults.memorySearch.remote.batch.enabled = true` para habilitar para indexación de corpus grandes (OpenAI, Gemini y Voyage).
-   El comportamiento predeterminado espera la finalización del lote; ajusta `remote.batch.wait`, `remote.batch.pollIntervalMs` y `remote.batch.timeoutMinutes` si es necesario.
-   Configura `remote.batch.concurrency` para controlar cuántos trabajos por lotes enviamos en paralelo (predeterminado: 2).
-   El modo por lotes se aplica cuando `memorySearch.provider = "openai"` o `"gemini"` y usa la clave API correspondiente.
-   Los trabajos por lotes de Gemini usan el endpoint de lotes de incrustaciones asíncronas y requieren disponibilidad de la API de Lotes de Gemini.

Por qué el lote de OpenAI es rápido + económico:

-   Para rellenos grandes, OpenAI es típicamente la opción más rápida que admitimos porque podemos enviar muchas solicitudes de incrustación en un solo trabajo por lotes y dejar que OpenAI las procese de forma asíncrona.
-   OpenAI ofrece precios con descuento para cargas de trabajo de la API por Lotes, por lo que las ejecuciones de indexación grandes suelen ser más económicas que enviar las mismas solicitudes de forma síncrona.
-   Consulta la documentación y precios de la API por Lotes de OpenAI para más detalles:
    -   [https://platform.openai.com/docs/api-reference/batch](https://platform.openai.com/docs/api-reference/batch)
    -   [https://platform.openai.com/pricing](https://platform.openai.com/pricing)

Ejemplo de configuración:

```
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

Herramientas:

-   `memory_search` — devuelve fragmentos con archivo + rangos de línea.
-   `memory_get` — lee el contenido del archivo de memoria por ruta.

Modo local:

-   Configura `agents.defaults.memorySearch.provider = "local"`.
-   Proporciona `agents.defaults.memorySearch.local.modelPath` (GGUF o URI `hf:`).
-   Opcional: configura `agents.defaults.memorySearch.fallback = "none"` para evitar el respaldo remoto.

### Cómo funcionan las herramientas de memoria

-   `memory_search` busca semánticamente fragmentos de Markdown (~400 tokens objetivo, superposición de 80 tokens) de `MEMORY.md` + `memory/**/*.md`. Devuelve texto del fragmento (limitado a ~700 caracteres), ruta del archivo, rango de línea, puntuación, proveedor/modelo, y si recurrimos de incrustaciones locales → remotas. No se devuelve la carga completa del archivo.
-   `memory_get` lee un archivo Markdown de memoria específico (relativo al espacio de trabajo), opcionalmente desde una línea inicial y durante N líneas. Las rutas fuera de `MEMORY.md` / `memory/` son rechazadas.
-   Ambas herramientas se habilitan solo cuando `memorySearch.enabled` se resuelve como verdadero para el agente.

### Qué se indexa (y cuándo)

-   Tipo de archivo: solo Markdown (`MEMORY.md`, `memory/**/*.md`).
-   Almacenamiento del índice: SQLite por agente en `~/.openclaw/memory/.sqlite` (configurable mediante `agents.defaults.memorySearch.store.path`, admite token `{agentId}`).
-   Actualidad: el observador en `MEMORY.md` + `memory/` marca el índice como sucio (rebote de 1.5s). La sincronización se programa al inicio de la sesión, en búsqueda o en un intervalo y se ejecuta de forma asíncrona. Las transcripciones de sesión usan umbrales delta para activar la sincronización en segundo plano.
-   Disparadores de reindexación: el índice almacena la **huella digital del proveedor/modelo de incrustación + endpoint + parámetros de fragmentación**. Si alguno de esos cambia, OpenClaw automáticamente restablece y reindexa todo el almacén.

### Búsqueda híbrida (BM25 + vector)

Cuando está habilitada, OpenClaw combina:

-   **Similitud vectorial** (coincidencia semántica, la redacción puede diferir)
-   **Relevancia de palabras clave BM25** (tokens exactos como IDs, variables de entorno, símbolos de código)

Si la búsqueda de texto completo no está disponible en tu plataforma, OpenClaw recurre a la búsqueda solo vectorial.

#### ¿Por qué híbrida?

La búsqueda vectorial es excelente para "esto significa lo mismo":

-   "host de puerta de enlace Mac Studio" vs "la máquina que ejecuta la puerta de enlace"
-   "debounce de actualizaciones de archivos" vs "evitar indexar en cada escritura"

Pero puede ser débil en tokens exactos y de alta señal:

-   IDs (`a828e60`, `b3b9895a…`)
-   símbolos de código (`memorySearch.query.hybrid`)
-   cadenas de error ("sqlite-vec no disponible")

BM25 (texto completo) es lo opuesto: fuerte en tokens exactos, más débil en paráfrasis. La búsqueda híbrida es el término medio pragmático: **usa ambas señales de recuperación** para obtener buenos resultados tanto para consultas de "lenguaje natural" como para consultas de "aguja en un pajar".

#### Cómo fusionamos resultados (el diseño actual)

Esquema de implementación:

1.  Recupera un grupo de candidatos de ambos lados:

-   **Vectorial**: los mejores `maxResults * candidateMultiplier` por similitud coseno.
-   **BM25**: los mejores `maxResults * candidateMultiplier` por rango BM25 de FTS5 (menor es mejor).

2.  Convierte el rango BM25 en una puntuación de 0..1 aproximada:

-   `textScore = 1 / (1 + max(0, bm25Rank))`

3.  Une candidatos por id de fragmento y calcula una puntuación ponderada:

-   `finalScore = vectorWeight * vectorScore + textWeight * textScore`

Notas:

-   `vectorWeight` + `textWeight` se normaliza a 1.0 en la resolución de configuración, por lo que los pesos se comportan como porcentajes.
-   Si las incrustaciones no están disponibles (o el proveedor devuelve un vector cero), aún ejecutamos BM25 y devolvemos coincidencias de palabras clave.
-   Si no se puede crear FTS5, mantenemos la búsqueda solo vectorial (sin fallo grave).

Esto no es "perfecto según la teoría de RI", pero es simple, rápido y tiende a mejorar la recuperación/precisión en notas reales. Si queremos ser más sofisticados más adelante, los siguientes pasos comunes son Fusión de Rango Recíproco (RRF) o normalización de puntuación (min/max o puntuación z) antes de mezclar.

#### Pipeline de post-procesamiento

Después de fusionar las puntuaciones vectoriales y de palabras clave, dos etapas opcionales de post-procesamiento refinan la lista de resultados antes de que llegue al agente:

```bash
Vector + Palabra clave → Fusión Ponderada → Decaimiento Temporal → Ordenar → MMR → Resultados Top-K
```

Ambas etapas están **deshabilitadas por defecto** y se pueden habilitar independientemente.

#### Reordenamiento MMR (diversidad)

Cuando la búsqueda híbrida devuelve resultados, múltiples fragmentos pueden contener contenido similar o superpuesto. Por ejemplo, buscar "configuración de red doméstica" podría devolver cinco fragmentos casi idénticos de diferentes notas diarias que mencionan la misma configuración del router. **MMR (Relevancia Marginal Máxima)** reordena los resultados para equilibrar la relevancia con la diversidad, asegurando que los mejores resultados cubran diferentes aspectos de la consulta en lugar de repetir la misma información. Cómo funciona:

1.  Los resultados se puntúan por su relevancia original (puntuación ponderada vector + BM25).
2.  MMR selecciona iterativamente resultados que maximizan: `λ × relevancia − (1−λ) × max_similitud_con_seleccionados`.
3.  La similitud entre resultados se mide usando similitud de texto Jaccard en contenido tokenizado.

El parámetro `lambda` controla la compensación:

-   `lambda = 1.0` → relevancia pura (sin penalización de diversidad)
-   `lambda = 0.0` → diversidad máxima (ignora la relevancia)
-   Predeterminado: `0.7` (equilibrado, ligero sesgo de relevancia)

**Ejemplo — consulta: "configuración de red doméstica"** Dados estos archivos de memoria:

```bash
memory/2026-02-10.md  → "Configuré router Omada, establecí VLAN 10 para dispositivos IoT"
memory/2026-02-08.md  → "Configuré router Omada, moví IoT a VLAN 10"
memory/2026-02-05.md  → "Configuré AdGuard DNS en 192.168.10.2"
memory/network.md     → "Router: Omada ER605, AdGuard: 192.168.10.2, VLAN 10: IoT"
```

Sin MMR — 3 mejores resultados:

```bash
1. memory/2026-02-10.md  (puntuación: 0.92)  ← router + VLAN
2. memory/2026-02-08.md  (puntuación: 0.89)  ← router + VLAN (¡casi duplicado!)
3. memory/network.md     (puntuación: 0.85)  ← documento de referencia
```

Con MMR (λ=0.7) — 3 mejores resultados:

```bash
1. memory/2026-02-10.md  (puntuación: 0.92)  ← router + VLAN
2. memory/network.md     (puntuación: 0.85)  ← documento de referencia (¡diverso!)
3. memory/2026-02-05.md  (puntuación: 0.78)  ← AdGuard DNS (¡diverso!)
```

El casi duplicado del 8 de febrero desaparece, y el agente obtiene tres piezas distintas de información. **Cuándo habilitar:** Si notas que `memory_search` devuelve fragmentos redundantes o casi duplicados, especialmente con notas diarias que a menudo repiten información similar a lo largo de los días.

#### Decaimiento temporal (impulso de actualidad)

Los agentes con notas diarias acumulan cientos de archivos fechados con el tiempo. Sin decaimiento, una nota bien redactada de hace seis meses puede superar en rango a la actualización de ayer sobre el mismo tema. **El decaimiento temporal** aplica un multiplicador exponencial a las puntuaciones basado en la antigüedad de cada resultado, por lo que los recuerdos recientes naturalmente tienen un rango más alto mientras que los antiguos se desvanecen:

```bash
decayedScore = score × e^(-λ × ageInDays)
```

donde `λ = ln(2) / halfLifeDays`. Con la vida media predeterminada de 30 días:

-   Notas de hoy: **100%** de la puntuación original
-   Hace 7 días: **~84%**
-   Hace 30 días: **50%**
-   Hace 90 días: **12.5%**
-   Hace 180 días: **~1.6%**

**Los archivos perennes nunca se desvanecen:**

-   `MEMORY.md` (archivo de memoria raíz)
-   Archivos no fechados en `memory/` (p.ej., `memory/projects.md`, `memory/network.md`)
-   Estos contienen información de referencia duradera que siempre debe tener un rango normal.

**Los archivos diarios fechados** (`memory/YYYY-MM-DD.md`) usan la fecha extraída del nombre del archivo. Otras fuentes (p.ej., transcripciones de sesión) recurren al tiempo de modificación del archivo (`mtime`). **Ejemplo — consulta: "¿cuál es el horario de trabajo de Rod?"** Dados estos archivos de memoria (hoy es 10 de febrero):

```bash
memory/2025-09-15.md  → "Rod trabaja Lun-Vie, standup a las 10am, pairing a las 2pm"  (148 días de antigüedad)
memory/2026-02-10.md  → "Rod tiene standup a las 14:15, 1:1 con Zeb a las 14:45"    (hoy)
memory/2026-02-03.md  → "Rod comenzó nuevo equipo, standup movido a las 14:15"        (7 días de antigüedad)
```

Sin decaimiento:

```bash
1. memory/2025-09-15.md  (puntuación: 0.91)  ← mejor coincidencia semántica, ¡pero obsoleta!
2. memory/2026-02-10.md  (puntuación: 0.82)
3. memory/2026-02-03.md  (puntuación: 0.80)
```

Con decaimiento (halfLife=30):

```bash
1. memory/2026-02-10.md  (puntuación: 0.82 × 1.00 = 0.82)  ← hoy, sin decaimiento
2. memory/2026-02-03.md  (puntuación: 0.80 × 0.85 = 0.68)  ← 7 días, decaimiento leve
3. memory/2025-09-15.md  (puntuación: 0.91 × 0.03 = 0.03)  ← 148 días, casi desaparecida
```

La nota obsoleta de septiembre cae al fondo a pesar de tener la mejor coincidencia semántica cruda. **Cuándo habilitar:** Si tu agente tiene meses de notas diarias y encuentras que información antigua y obsoleta supera en rango al contexto reciente. Una vida media de 30 días funciona bien para flujos de trabajo con muchas notas diarias; auméntala (p.ej., 90 días) si consultas notas más antiguas con frecuencia.

#### Configuración

Ambas características se configuran bajo `memorySearch.query.hybrid`:

```
agents: {
  defaults: {
    memorySearch: {
      query: {
        hybrid: {
          enabled: true,
          vectorWeight: 0.7,
          textWeight: 0.3,
          candidateMultiplier: 4,
          // Diversidad: reduce resultados redundantes
          mmr: {
            enabled: true,