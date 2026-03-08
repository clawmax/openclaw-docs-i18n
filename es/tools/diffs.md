

  Herramientas integradas

  
# Diffs

`diffs` es una herramienta de plugin opcional con una guía del sistema integrada breve y una habilidad complementaria que convierte contenido de cambios en un artefacto de diferencias de solo lectura para agentes. Acepta:

-   texto `before` y `after`
-   un `patch` unificado

Puede devolver:

-   una URL de visor del gateway para presentación en el lienzo
-   una ruta de archivo renderizado (PNG o PDF) para entrega en mensajes
-   ambas salidas en una sola llamada

Cuando está habilitado, el plugin antepone una guía de uso concisa al espacio del system-prompt y también expone una habilidad detallada para los casos en que el agente necesite instrucciones más completas.

## Inicio rápido

1.  Habilita el plugin.
2.  Llama a `diffs` con `mode: "view"` para flujos centrados en el lienzo.
3.  Llama a `diffs` con `mode: "file"` para flujos de entrega de archivos en el chat.
4.  Llama a `diffs` con `mode: "both"` cuando necesites ambos artefactos.

## Habilitar el plugin

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
      },
    },
  },
}
```

## Deshabilitar la guía del sistema integrada

Si deseas mantener la herramienta `diffs` habilitada pero deshabilitar su guía integrada del system-prompt, establece `plugins.entries.diffs.hooks.allowPromptInjection` en `false`:

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        hooks: {
          allowPromptInjection: false,
        },
      },
    },
  },
}
```

Esto bloquea el hook `before_prompt_build` del plugin diffs mientras mantiene disponible el plugin, la herramienta y la habilidad complementaria. Si deseas deshabilitar tanto la guía como la herramienta, deshabilita el plugin en su lugar.

## Flujo de trabajo típico del agente

1.  El agente llama a `diffs`.
2.  El agente lee los campos `details`.
3.  El agente, o bien:
    -   abre `details.viewerUrl` con `canvas present`
    -   envía `details.filePath` con `message` usando `path` o `filePath`
    -   hace ambas cosas

## Ejemplos de entrada

Before y after:

```json
{
  "before": "# Hello\n\nOne",
  "after": "# Hello\n\nTwo",
  "path": "docs/example.md",
  "mode": "view"
}
```

Patch:

```json
{
  "patch": "diff --git a/src/example.ts b/src/example.ts\n--- a/src/example.ts\n+++ b/src/example.ts\n@@ -1 +1 @@\n-const x = 1;\n+const x = 2;\n",
  "mode": "both"
}
```

## Referencia de entrada de la herramienta

Todos los campos son opcionales a menos que se indique lo contrario:

-   `before` (`string`): texto original. Requerido con `after` cuando se omite `patch`.
-   `after` (`string`): texto actualizado. Requerido con `before` cuando se omite `patch`.
-   `patch` (`string`): texto de diff unificado. Excluyente mutuamente con `before` y `after`.
-   `path` (`string`): nombre de archivo para mostrar en el modo before y after.
-   `lang` (`string`): sugerencia de anulación de idioma para el modo before y after.
-   `title` (`string`): anulación del título del visor.
-   `mode` (`"view" | "file" | "both"`): modo de salida. Por defecto, el valor predeterminado del plugin `defaults.mode`.
-   `theme` (`"light" | "dark"`): tema del visor. Por defecto, el valor predeterminado del plugin `defaults.theme`.
-   `layout` (`"unified" | "split"`): diseño de diferencias. Por defecto, el valor predeterminado del plugin `defaults.layout`.
-   `expandUnchanged` (`boolean`): expandir secciones sin cambios cuando el contexto completo está disponible. Opción solo por llamada (no es una clave predeterminada del plugin).
-   `fileFormat` (`"png" | "pdf"`): formato de archivo renderizado. Por defecto, el valor predeterminado del plugin `defaults.fileFormat`.
-   `fileQuality` (`"standard" | "hq" | "print"`): preajuste de calidad para renderizado PNG o PDF.
-   `fileScale` (`number`): anulación de escala del dispositivo (`1`\-`4`).
-   `fileMaxWidth` (`number`): ancho máximo de renderizado en píxeles CSS (`640`\-`2400`).
-   `ttlSeconds` (`number`): TTL del artefacto del visor en segundos. Por defecto 1800, máximo 21600.
-   `baseUrl` (`string`): anulación del origen de la URL del visor. Debe ser `http` o `https`, sin query/hash.

Validación y límites:

-   `before` y `after` máximo 512 KiB cada uno.
-   `patch` máximo 2 MiB.
-   `path` máximo 2048 bytes.
-   `lang` máximo 128 bytes.
-   `title` máximo 1024 bytes.
-   Límite de complejidad del patch: máximo 128 archivos y 120000 líneas en total.
-   `patch` junto con `before` o `after` es rechazado.
-   Límites de seguridad del archivo renderizado (se aplican a PNG y PDF):
    -   `fileQuality: "standard"`: máximo 8 MP (8,000,000 píxeles renderizados).
    -   `fileQuality: "hq"`: máximo 14 MP (14,000,000 píxeles renderizados).
    -   `fileQuality: "print"`: máximo 24 MP (24,000,000 píxeles renderizados).
    -   PDF también tiene un máximo de 50 páginas.

## Contrato de detalles de salida

La herramienta devuelve metadatos estructurados bajo `details`. Campos compartidos para modos que crean un visor:

-   `artifactId`
-   `viewerUrl`
-   `viewerPath`
-   `title`
-   `expiresAt`
-   `inputKind`
-   `fileCount`
-   `mode`

Campos de archivo cuando se renderiza PNG o PDF:

-   `filePath`
-   `path` (mismo valor que `filePath`, para compatibilidad con la herramienta message)
-   `fileBytes`
-   `fileFormat`
-   `fileQuality`
-   `fileScale`
-   `fileMaxWidth`

Resumen del comportamiento del modo:

-   `mode: "view"`: solo campos del visor.
-   `mode: "file"`: solo campos de archivo, sin artefacto de visor.
-   `mode: "both"`: campos del visor más campos de archivo. Si falla el renderizado del archivo, el visor aún se devuelve con `fileError`.

## Secciones sin cambios colapsadas

-   El visor puede mostrar filas como `N líneas sin modificar`.
-   Los controles de expansión en esas filas son condicionales y no están garantizados para cada tipo de entrada.
-   Los controles de expansión aparecen cuando el diff renderizado tiene datos de contexto expandibles, lo cual es típico para la entrada before y after.
-   Para muchas entradas de patch unificado, los cuerpos de contexto omitidos no están disponibles en los fragmentos del patch analizado, por lo que la fila puede aparecer sin controles de expansión. Este es el comportamiento esperado.
-   `expandUnchanged` se aplica solo cuando existe contexto expandible.

## Valores predeterminados del plugin

Establece valores predeterminados para todo el plugin en `~/.openclaw/openclaw.json`:

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          defaults: {
            fontFamily: "Fira Code",
            fontSize: 15,
            lineSpacing: 1.6,
            layout: "unified",
            showLineNumbers: true,
            diffIndicators: "bars",
            wordWrap: true,
            background: true,
            theme: "dark",
            fileFormat: "png",
            fileQuality: "standard",
            fileScale: 2,
            fileMaxWidth: 960,
            mode: "both",
          },
        },
      },
    },
  },
}
```

Valores predeterminados admitidos:

-   `fontFamily`
-   `fontSize`
-   `lineSpacing`
-   `layout`
-   `showLineNumbers`
-   `diffIndicators`
-   `wordWrap`
-   `background`
-   `theme`
-   `fileFormat`
-   `fileQuality`
-   `fileScale`
-   `fileMaxWidth`
-   `mode`

Los parámetros explícitos de la herramienta anulan estos valores predeterminados.

## Configuración de seguridad

-   `security.allowRemoteViewer` (`boolean`, predeterminado `false`)
    -   `false`: se deniegan las solicitudes no loopback a las rutas del visor.
    -   `true`: se permiten visores remotos si la ruta tokenizada es válida.

Ejemplo:

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          security: {
            allowRemoteViewer: false,
          },
        },
      },
    },
  },
}
```

## Ciclo de vida y almacenamiento de artefactos

-   Los artefactos se almacenan en la subcarpeta temporal: `$TMPDIR/openclaw-diffs`.
-   Los metadatos del artefacto del visor contienen:
    -   ID de artefacto aleatorio (20 caracteres hex)
    -   token aleatorio (48 caracteres hex)
    -   `createdAt` y `expiresAt`
    -   ruta almacenada `viewer.html`
-   El TTL predeterminado del visor es de 30 minutos cuando no se especifica.
-   El TTL máximo aceptado del visor es de 6 horas.
-   La limpieza se ejecuta de forma oportunista después de la creación del artefacto.
-   Los artefactos expirados se eliminan.
-   La limpieza de respaldo elimina carpetas obsoletas de más de 24 horas cuando faltan metadatos.

## URL del visor y comportamiento de red

Ruta del visor:

-   `/plugins/diffs/view/{artifactId}/{token}`

Recursos del visor:

-   `/plugins/diffs/assets/viewer.js`
-   `/plugins/diffs/assets/viewer-runtime.js`

Comportamiento de construcción de URL:

-   Si se proporciona `baseUrl`, se utiliza después de una validación estricta.
-   Sin `baseUrl`, la URL del visor predeterminada es loopback `127.0.0.1`.
-   Si el modo de enlace del gateway es `custom` y `gateway.customBindHost` está establecido, se utiliza ese host.

Reglas de `baseUrl`:

-   Debe ser `http://` o `https://`.
-   Se rechazan query y hash.
-   Se permite el origen más una ruta base opcional.

## Modelo de seguridad

Refuerzo del visor:

-   Solo loopback por defecto.
-   Rutas de visor tokenizadas con validación estricta de ID y token.
-   CSP de respuesta del visor:
    -   `default-src 'none'`
    -   scripts y recursos solo desde sí mismo
    -   sin `connect-src` saliente
-   Limitación de fallos remotos cuando el acceso remoto está habilitado:
    -   40 fallos por 60 segundos
    -   bloqueo de 60 segundos (`429 Too Many Requests`)

Refuerzo del renderizado de archivos:

-   El enrutamiento de solicitudes del navegador de captura de pantalla es denegar por defecto.
-   Solo se permiten recursos del visor local desde `http://127.0.0.1/plugins/diffs/assets/*`.
-   Las solicitudes de red externas están bloqueadas.

## Requisitos del navegador para el modo archivo

`mode: "file"` y `mode: "both"` necesitan un navegador compatible con Chromium. Orden de resolución:

1.  `browser.executablePath` en la configuración de OpenClaw.
2.  Variables de entorno:
    -   `OPENCLAW_BROWSER_EXECUTABLE_PATH`
    -   `BROWSER_EXECUTABLE_PATH`
    -   `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH`
3.  Respaldo de descubrimiento de comando/ruta de la plataforma.

Texto de error común:

-   `Diff PNG/PDF rendering requires a Chromium-compatible browser...`

Solución instalando Chrome, Chromium, Edge o Brave, o estableciendo una de las opciones de ruta ejecutable anteriores.

## Resolución de problemas

Errores de validación de entrada:

-   `Provide patch or both before and after text.`
    -   Incluye tanto `before` como `after`, o proporciona `patch`.
-   `Provide either patch or before/after input, not both.`
    -   No mezcles modos de entrada.
-   `Invalid baseUrl: ...`
    -   Usa origen `http(s)` con ruta opcional, sin query/hash.
-   `{field} exceeds maximum size (...)`
    -   Reduce el tamaño de la carga útil.
-   Rechazo de patch grande
    -   Reduce el número de archivos del patch o el total de líneas.

Problemas de accesibilidad del visor:

-   La URL del visor se resuelve a `127.0.0.1` por defecto.
-   Para escenarios de acceso remoto, o bien:
    -   pasa `baseUrl` por llamada de herramienta, o
    -   usa `gateway.bind=custom` y `gateway.customBindHost`
-   Habilita `security.allowRemoteViewer` solo cuando tengas la intención de acceso externo al visor.

La fila de líneas sin modificar no tiene botón de expandir:

-   Esto puede ocurrir para la entrada de patch cuando el patch no lleva contexto expandible.
-   Esto es esperado y no indica un fallo del visor.

Artefacto no encontrado:

-   El artefacto expiró debido al TTL.
-   El token o la ruta cambiaron.
-   La limpieza eliminó datos obsoletos.

## Guía operativa

-   Prefiere `mode: "view"` para revisiones interactivas locales en el lienzo.
-   Prefiere `mode: "file"` para canales de chat salientes que necesiten un adjunto.
-   Mantén `allowRemoteViewer` deshabilitado a menos que tu despliegue requiera URLs de visor remotas.
-   Establece un `ttlSeconds` corto y explícito para diffs sensibles.
-   Evita enviar secretos en la entrada de diff cuando no sea necesario.
-   Si tu canal comprime imágenes agresivamente (por ejemplo Telegram o WhatsApp), prefiere la salida PDF (`fileFormat: "pdf"`).

Motor de renderizado de diferencias:

## Documentación relacionada

-   [Resumen de herramientas](../tools.md)
-   [Plugins](./plugin.md)
-   [Navegador](./browser.md)

[Perplexity Sonar](../perplexity.md)[Herramienta PDF](./pdf.md)