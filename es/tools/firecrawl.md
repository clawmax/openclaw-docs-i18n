

  Herramientas integradas

  
# Firecrawl

OpenClaw puede usar **Firecrawl** como un extractor de respaldo para `web_fetch`. Es un servicio de extracción de contenido alojado que soporta evasión de bots y caché, lo que ayuda con sitios con mucho JavaScript o páginas que bloquean peticiones HTTP simples.

## Obtener una clave API

1.  Crea una cuenta en Firecrawl y genera una clave API.
2.  Guárdala en la configuración o establece `FIRECRAWL_API_KEY` en el entorno de la pasarela.

## Configurar Firecrawl

```json
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

Notas:

-   `firecrawl.enabled` es verdadero por defecto cuando hay una clave API presente.
-   `maxAgeMs` controla qué tan antiguos pueden ser los resultados en caché (ms). El valor por defecto es 2 días.

## Sigilo / evasión de bots

Firecrawl expone un parámetro de **modo proxy** para evasión de bots (`basic`, `stealth`, o `auto`). OpenClaw siempre usa `proxy: "auto"` más `storeInCache: true` para las peticiones a Firecrawl. Si se omite proxy, Firecrawl usa `auto` por defecto. `auto` reintenta con proxies sigilosos si un intento básico falla, lo que puede usar más créditos que el raspado solo básico.

## Cómo web\_fetch usa Firecrawl

Orden de extracción de `web_fetch`:

1.  Readability (local)
2.  Firecrawl (si está configurado)
3.  Limpieza básica de HTML (último recurso)

Consulta [Herramientas web](./web.md) para la configuración completa de las herramientas web.

[Aprobaciones de ejecución](./exec-approvals.md)[Tarea LLM](./llm-task.md)

---