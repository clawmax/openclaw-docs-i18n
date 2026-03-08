

  Herramientas integradas

  
# Herramienta PDF

`pdf` analiza uno o más documentos PDF y devuelve texto. Comportamiento rápido:

-   Modo de proveedor nativo para los proveedores de modelos Anthropic y Google.
-   Modo de respaldo por extracción para otros proveedores (extrae texto primero, luego imágenes de página cuando es necesario).
-   Soporta entrada única (`pdf`) o múltiple (`pdfs`), máximo 10 PDFs por llamada.

## Disponibilidad

La herramienta solo se registra cuando OpenClaw puede resolver una configuración de modelo compatible con PDF para el agente:

1.  `agents.defaults.pdfModel`
2.  respaldo a `agents.defaults.imageModel`
3.  respaldo a valores predeterminados del proveedor de mejor esfuerzo según la autenticación disponible

Si no se puede resolver un modelo utilizable, la herramienta `pdf` no se expone.

## Referencia de entrada

-   `pdf` (`string`): una ruta o URL de PDF
-   `pdfs` (`string[]`): múltiples rutas o URLs de PDF, hasta 10 en total
-   `prompt` (`string`): indicación de análisis, predeterminada `Analyze this PDF document.`
-   `pages` (`string`): filtro de páginas como `1-5` o `1,3,7-9`
-   `model` (`string`): anulación opcional del modelo (`provider/model`)
-   `maxBytesMb` (`number`): límite de tamaño por PDF en MB

Notas de entrada:

-   `pdf` y `pdfs` se fusionan y se eliminan duplicados antes de cargar.
-   Si no se proporciona entrada de PDF, la herramienta genera un error.
-   `pages` se analiza como números de página basados en 1, se eliminan duplicados, se ordenan y se limitan al máximo de páginas configurado.
-   `maxBytesMb` tiene como valor predeterminado `agents.defaults.pdfMaxBytesMb` o `10`.

## Referencias PDF admitidas

-   ruta de archivo local (incluyendo expansión `~`)
-   URL `file://`
-   URL `http://` y `https://`

Notas de referencia:

-   Otros esquemas URI (por ejemplo `ftp://`) son rechazados con `unsupported_pdf_reference`.
-   En modo sandbox, las URLs remotas `http(s)` son rechazadas.
-   Con la política de solo archivos de espacio de trabajo habilitada, las rutas de archivo local fuera de las raíces permitidas son rechazadas.

## Modos de ejecución

### Modo de proveedor nativo

El modo nativo se usa para los proveedores `anthropic` y `google`. La herramienta envía los bytes brutos del PDF directamente a las APIs del proveedor. Límites del modo nativo:

-   `pages` no es compatible. Si se establece, la herramienta devuelve un error.

### Modo de respaldo por extracción

El modo de respaldo se usa para proveedores no nativos. Flujo:

1.  Extrae texto de las páginas seleccionadas (hasta `agents.defaults.pdfMaxPages`, predeterminado `20`).
2.  Si la longitud del texto extraído es inferior a `200` caracteres, renderiza las páginas seleccionadas a imágenes PNG y las incluye.
3.  Envía el contenido extraído más la indicación al modelo seleccionado.

Detalles del respaldo:

-   La extracción de imágenes de página usa un presupuesto de píxeles de `4,000,000`.
-   Si el modelo objetivo no admite entrada de imagen y no hay texto extraíble, la herramienta genera un error.
-   El respaldo por extracción requiere `pdfjs-dist` (y `@napi-rs/canvas` para el renderizado de imágenes).

## Configuración

```json
{
  agents: {
    defaults: {
      pdfModel: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["openai/gpt-5-mini"],
      },
      pdfMaxBytesMb: 10,
      pdfMaxPages: 20,
    },
  },
}
```

Consulta la [Referencia de Configuración](../gateway/configuration-reference.md) para detalles completos de los campos.

## Detalles de salida

La herramienta devuelve texto en `content[0].text` y metadatos estructurados en `details`. Campos comunes de `details`:

-   `model`: referencia del modelo resuelto (`provider/model`)
-   `native`: `true` para modo de proveedor nativo, `false` para respaldo
-   `attempts`: intentos de respaldo que fallaron antes del éxito

Campos de ruta:

-   entrada de PDF único: `details.pdf`
-   entradas de PDF múltiples: `details.pdfs[]` con entradas `pdf`
-   metadatos de reescritura de ruta sandbox (cuando aplica): `rewrittenFrom`

## Comportamiento de error

-   Falta entrada de PDF: lanza `pdf required: provide a path or URL to a PDF document`
-   Demasiados PDFs: devuelve error estructurado en `details.error = "too_many_pdfs"`
-   Esquema de referencia no admitido: devuelve `details.error = "unsupported_pdf_reference"`
-   Modo nativo con `pages`: lanza error claro `pages is not supported with native PDF providers`

## Ejemplos

PDF único:

```json
{
  "pdf": "/tmp/report.pdf",
  "prompt": "Summarize this report in 5 bullets"
}
```

PDFs múltiples:

```json
{
  "pdfs": ["/tmp/q1.pdf", "/tmp/q2.pdf"],
  "prompt": "Compare risks and timeline changes across both documents"
}
```

Modelo de respaldo con filtro de páginas:

```json
{
  "pdf": "https://example.com/report.pdf",
  "pages": "1-3,7",
  "model": "openai/gpt-5-mini",
  "prompt": "Extract only customer-impacting incidents"
}
```

[Diferencias](./diffs.md)[Modo Elevado](./elevated.md)