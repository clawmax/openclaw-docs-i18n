

  Herramientas integradas

  
# Brave Search

OpenClaw admite Brave Search como proveedor de búsqueda web para `web_search`.

## Obtener una clave API

1.  Crea una cuenta en la API de Brave Search en [https://brave.com/search/api/](https://brave.com/search/api/)
2.  En el panel de control, elige el plan **Data for Search** y genera una clave API.
3.  Guarda la clave en la configuración (recomendado) o establece `BRAVE_API_KEY` en el entorno del Gateway.

## Ejemplo de configuración

```json
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

## Parámetros de la herramienta

| Parámetro | Descripción |
| --- | --- |
| `query` | Consulta de búsqueda (requerida) |
| `count` | Número de resultados a devolver (1-10, predeterminado: 5) |
| `country` | Código de país ISO de 2 letras (ej., “US”, “DE”) |
| `language` | Código de idioma ISO 639-1 para los resultados de búsqueda (ej., “en”, “de”, “fr”) |
| `ui_lang` | Código de idioma ISO para elementos de la interfaz de usuario |
| `freshness` | Filtro de tiempo: `day` (24h), `week`, `month`, o `year` |
| `date_after` | Solo resultados publicados después de esta fecha (YYYY-MM-DD) |
| `date_before` | Solo resultados publicados antes de esta fecha (YYYY-MM-DD) |

**Ejemplos:**

```javascript
// Búsqueda específica por país e idioma
await web_search({
  query: "energía renovable",
  country: "DE",
  language: "de",
});

// Resultados recientes (última semana)
await web_search({
  query: "noticias de IA",
  freshness: "week",
});

// Búsqueda por rango de fechas
await web_search({
  query: "desarrollos en IA",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});
```

## Notas

-   El plan Data for AI **no** es compatible con `web_search`.
-   Brave ofrece planes de pago; consulta el portal de la API de Brave para conocer los límites actuales.
-   Los Términos de Brave incluyen restricciones sobre algunos usos relacionados con IA de los Resultados de Búsqueda. Revisa los Términos de Servicio de Brave y confirma que tu uso previsto sea conforme. Para preguntas legales, consulta a tu asesor legal.
-   Los resultados se almacenan en caché durante 15 minutos por defecto (configurable mediante `cacheTtlMinutes`).

Consulta [Herramientas web](./tools/web.md) para la configuración completa de web\_search.

[Herramienta apply\_patch](./tools/apply-patch.md)[Perplexity Sonar](./perplexity.md)