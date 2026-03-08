

  Herramientas integradas

  
# Herramientas Web

OpenClaw incluye dos herramientas web ligeras:

-   `web_search` — Busca en la web usando la API de Búsqueda de Perplexity, la API de Búsqueda de Brave, Gemini con fundamentación de Google Search, Grok o Kimi.
-   `web_fetch` — Obtención HTTP + extracción legible (HTML → markdown/texto).

Estas **no son** automatización de navegador. Para sitios con mucho JavaScript o inicios de sesión, usa la [Herramienta de Navegador](./browser.md).

## Cómo funciona

-   `web_search` llama a tu proveedor configurado y devuelve resultados.
-   Los resultados se almacenan en caché por consulta durante 15 minutos (configurable).
-   `web_fetch` realiza una petición HTTP GET simple y extrae contenido legible (HTML → markdown/texto). **No** ejecuta JavaScript.
-   `web_fetch` está habilitada por defecto (a menos que se deshabilite explícitamente).

Consulta [Configuración de Búsqueda de Perplexity](../perplexity.md) y [Configuración de Búsqueda de Brave](../brave-search.md) para detalles específicos del proveedor.

## Elegir un proveedor de búsqueda

| Proveedor | Ventajas | Desventajas | Clave de API |
| --- | --- | --- | --- |
| **API de Búsqueda de Perplexity** | Rápida, resultados estructurados; filtros de dominio, idioma, región y frescura; extracción de contenido | — | `PERPLEXITY_API_KEY` |
| **API de Búsqueda de Brave** | Rápida, resultados estructurados | Menos opciones de filtrado; aplican términos de uso para IA | `BRAVE_API_KEY` |
| **Gemini** | Fundamentación con Google Search, sintetizado por IA | Requiere clave de API de Gemini | `GEMINI_API_KEY` |
| **Grok** | Respuestas fundamentadas en la web de xAI | Requiere clave de API de xAI | `XAI_API_KEY` |
| **Kimi** | Capacidad de búsqueda web de Moonshot | Requiere clave de API de Moonshot | `KIMI_API_KEY` / `MOONSHOT_API_KEY` |

### Autodetección

Si no se establece explícitamente un `provider`, OpenClaw detecta automáticamente qué proveedor usar según las claves de API disponibles, verificando en este orden:

1.  **Brave** — Variable de entorno `BRAVE_API_KEY` o configuración `tools.web.search.apiKey`
2.  **Gemini** — Variable de entorno `GEMINI_API_KEY` o configuración `tools.web.search.gemini.apiKey`
3.  **Kimi** — Variable de entorno `KIMI_API_KEY` / `MOONSHOT_API_KEY` o configuración `tools.web.search.kimi.apiKey`
4.  **Perplexity** — Variable de entorno `PERPLEXITY_API_KEY` o configuración `tools.web.search.perplexity.apiKey`
5.  **Grok** — Variable de entorno `XAI_API_KEY` o configuración `tools.web.search.grok.apiKey`

Si no se encuentran claves, recurre a Brave (obtendrás un error de clave faltante que te pedirá configurar una).

## Configurar la búsqueda web

Usa `openclaw configure --section web` para configurar tu clave de API y elegir un proveedor.

### Búsqueda de Perplexity

1.  Crea una cuenta en Perplexity en [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api)
2.  Genera una clave de API en el panel de control
3.  Ejecuta `openclaw configure --section web` para almacenar la clave en la configuración, o establece `PERPLEXITY_API_KEY` en tu entorno.

Consulta la [Documentación de la API de Búsqueda de Perplexity](https://docs.perplexity.ai/guides/search-quickstart) para más detalles.

### Búsqueda de Brave

1.  Crea una cuenta para la API de Búsqueda de Brave en [brave.com/search/api](https://brave.com/search/api/)
2.  En el panel de control, elige el plan **Data for Search** (no "Data for AI") y genera una clave de API.
3.  Ejecuta `openclaw configure --section web` para almacenar la clave en la configuración (recomendado), o establece `BRAVE_API_KEY` en tu entorno.

Brave ofrece planes de pago; consulta el portal de la API de Brave para conocer los límites y precios actuales.

### Dónde almacenar la clave

**Vía configuración (recomendado):** ejecuta `openclaw configure --section web`. Almacena la clave bajo `tools.web.search.perplexity.apiKey` o `tools.web.search.apiKey`. **Vía entorno:** establece `PERPLEXITY_API_KEY` o `BRAVE_API_KEY` en el entorno del proceso del Gateway. Para una instalación de gateway, colócala en `~/.openclaw/.env` (o en el entorno de tu servicio). Consulta [Variables de entorno](../help/faq.md#how-does-openclaw-load-environment-variables).

### Ejemplos de configuración

**Búsqueda de Perplexity:**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...", // opcional si PERPLEXITY_API_KEY está establecida
        },
      },
    },
  },
}
```

**Búsqueda de Brave:**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "brave",
        apiKey: "YOUR_BRAVE_API_KEY", // opcional si BRAVE_API_KEY está establecida // pragma: allowlist secret
      },
    },
  },
}
```

## Usar Gemini (fundamentación con Google Search)

Los modelos de Gemini admiten la [fundamentación con Google Search](https://ai.google.dev/gemini-api/docs/grounding) integrada, que devuelve respuestas sintetizadas por IA respaldadas por resultados en vivo de Google Search con citas.

### Obtener una clave de API de Gemini

1.  Ve a [Google AI Studio](https://aistudio.google.com/apikey)
2.  Crea una clave de API
3.  Establece `GEMINI_API_KEY` en el entorno del Gateway, o configura `tools.web.search.gemini.apiKey`

### Configurar la búsqueda de Gemini

```json
{
  tools: {
    web: {
      search: {
        provider: "gemini",
        gemini: {
          // Clave de API (opcional si GEMINI_API_KEY está establecida)
          apiKey: "AIza...",
          // Modelo (por defecto "gemini-2.5-flash")
          model: "gemini-2.5-flash",
        },
      },
    },
  },
}
```

**Alternativa por entorno:** establece `GEMINI_API_KEY` en el entorno del Gateway. Para una instalación de gateway, colócala en `~/.openclaw/.env`.

### Notas

-   Las URLs de citas de la fundamentación de Gemini se resuelven automáticamente desde las URLs de redirección de Google a URLs directas.
-   La resolución de redirecciones utiliza la ruta de protección SSRF (HEAD + comprobaciones de redirección + validación http/https) antes de devolver la URL de cita final.
-   La resolución de redirecciones utiliza valores predeterminados estrictos de SSRF, por lo que se bloquean las redirecciones a objetivos privados/internos.
-   El modelo predeterminado (`gemini-2.5-flash`) es rápido y rentable. Se puede usar cualquier modelo Gemini que admita fundamentación.

## web\_search

Busca en la web usando tu proveedor configurado.

### Requisitos

-   `tools.web.search.enabled` no debe ser `false` (predeterminado: habilitado)
-   Clave de API para tu proveedor elegido:
    -   **Brave**: `BRAVE_API_KEY` o `tools.web.search.apiKey`
    -   **Perplexity**: `PERPLEXITY_API_KEY` o `tools.web.search.perplexity.apiKey`
    -   **Gemini**: `GEMINI_API_KEY` o `tools.web.search.gemini.apiKey`
    -   **Grok**: `XAI_API_KEY` o `tools.web.search.grok.apiKey`
    -   **Kimi**: `KIMI_API_KEY`, `MOONSHOT_API_KEY`, o `tools.web.search.kimi.apiKey`

### Configuración

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // opcional si BRAVE_API_KEY está establecida
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### Parámetros de la herramienta

Todos los parámetros funcionan tanto para Brave como para Perplexity a menos que se indique lo contrario.

| Parámetro | Descripción |
| --- | --- |
| `query` | Consulta de búsqueda (requerida) |
| `count` | Resultados a devolver (1-10, predeterminado: 5) |
| `country` | Código de país ISO de 2 letras (ej., “US”, “DE”) |
| `language` | Código de idioma ISO 639-1 (ej., “en”, “de”) |
| `freshness` | Filtro de tiempo: `day`, `week`, `month`, o `year` |
| `date_after` | Resultados después de esta fecha (YYYY-MM-DD) |
| `date_before` | Resultados antes de esta fecha (YYYY-MM-DD) |
| `ui_lang` | Código de idioma de la interfaz (solo Brave) |
| `domain_filter` | Lista de dominios permitidos/denegados (array) (solo Perplexity) |
| `max_tokens` | Presupuesto total de contenido, predeterminado 25000 (solo Perplexity) |
| `max_tokens_per_page` | Límite de tokens por página, predeterminado 2048 (solo Perplexity) |

**Ejemplos:**

```
// Búsqueda específica para alemán
await web_search({
  query: "TV online schauen",
  country: "DE",
  language: "de",
});

// Resultados recientes (última semana)
await web_search({
  query: "TMBG interview",
  freshness: "week",
});

// Búsqueda por rango de fechas
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});

// Filtrado por dominio (solo Perplexity)
await web_search({
  query: "climate research",
  domain_filter: ["nature.com", "science.org", ".edu"],
});

// Excluir dominios (solo Perplexity)
await web_search({
  query: "product reviews",
  domain_filter: ["-reddit.com", "-pinterest.com"],
});

// Más extracción de contenido (solo Perplexity)
await web_search({
  query: "detailed AI research",
  max_tokens: 50000,
  max_tokens_per_page: 4096,
});
```

## web\_fetch

Obtiene una URL y extrae contenido legible.

### Requisitos de web\_fetch

-   `tools.web.fetch.enabled` no debe ser `false` (predeterminado: habilitado)
-   Respaldo opcional de Firecrawl: establece `tools.web.fetch.firecrawl.apiKey` o `FIRECRAWL_API_KEY`.

### Configuración de web\_fetch

```json
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // opcional si FIRECRAWL_API_KEY está establecida
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // ms (1 día)
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### Parámetros de la herramienta web\_fetch

-   `url` (requerida, solo http/https)
-   `extractMode` (`markdown` | `text`)
-   `maxChars` (truncar páginas largas)

Notas:

-   `web_fetch` usa Readability (extracción de contenido principal) primero, luego Firecrawl (si está configurado). Si ambos fallan, la herramienta devuelve un error.
-   Las peticiones de Firecrawl usan el modo de evasión de bots y almacenan resultados en caché por defecto.
-   `web_fetch` envía un User-Agent similar a Chrome y `Accept-Language` por defecto; anula `userAgent` si es necesario.
-   `web_fetch` bloquea nombres de host privados/internos y re-comprueba redirecciones (límite con `maxRedirects`).
-   `maxChars` se limita a `tools.web.fetch.maxCharsCap`.
-   `web_fetch` limita el tamaño del cuerpo de respuesta descargado a `tools.web.fetch.maxResponseBytes` antes de analizar; las respuestas demasiado grandes se truncan e incluyen una advertencia.
-   `web_fetch` es una extracción de mejor esfuerzo; algunos sitios necesitarán la herramienta de navegador.
-   Consulta [Firecrawl](./firecrawl.md) para detalles de configuración de clave y servicio.
-   Las respuestas se almacenan en caché (predeterminado 15 minutos) para reducir peticiones repetidas.
-   Si usas perfiles/listas de permitidos de herramientas, añade `web_search`/`web_fetch` o `group:web`.
-   Si falta la clave de API, `web_search` devuelve una breve pista de configuración con un enlace a la documentación.

[Niveles de Pensamiento](./thinking.md)[Navegador (gestionado por OpenClaw)](./browser.md)