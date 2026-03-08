

  Herramientas integradas

  
# Perplexity Sonar

OpenClaw puede usar Perplexity Sonar para la herramienta `web_search`. Puedes conectarte a través de la API directa de Perplexity o mediante OpenRouter.

## Opciones de API

### Perplexity (directo)

-   URL base: [https://api.perplexity.ai](https://api.perplexity.ai)
-   Variable de entorno: `PERPLEXITY_API_KEY`

### OpenRouter (alternativa)

-   URL base: [https://openrouter.ai/api/v1](https://openrouter.ai/api/v1)
-   Variable de entorno: `OPENROUTER_API_KEY`
-   Admite créditos prepago/cripto.

## Ejemplo de configuración

```json
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

## Cambiar desde Brave

```json
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
        },
      },
    },
  },
}
```

Si están configuradas tanto `PERPLEXITY_API_KEY` como `OPENROUTER_API_KEY`, establece `tools.web.search.perplexity.baseUrl` (o `tools.web.search.perplexity.apiKey`) para eliminar ambigüedades. Si no se establece ninguna URL base, OpenClaw elige una predeterminada según el origen de la clave API:

-   `PERPLEXITY_API_KEY` o `pplx-...` → Perplexity directo (`https://api.perplexity.ai`)
-   `OPENROUTER_API_KEY` o `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
-   Formatos de clave desconocidos → OpenRouter (respaldo seguro)

## Modelos

-   `perplexity/sonar` — preguntas y respuestas rápidas con búsqueda web
-   `perplexity/sonar-pro` (predeterminado) — razonamiento de múltiples pasos + búsqueda web
-   `perplexity/sonar-reasoning-pro` — investigación profunda

Consulta [Herramientas web](./tools/web.md) para la configuración completa de web\_search.

[Brave Search](./brave-search.md)[Diferencias](./tools/diffs.md)

---