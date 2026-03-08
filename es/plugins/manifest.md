

  Extensiones

  
# Manifiesto del Plugin

Cada plugin **debe** incluir un archivo `openclaw.plugin.json` en la **raíz del plugin**. OpenClaw usa este manifiesto para validar la configuración **sin ejecutar el código del plugin**. Los manifiestos faltantes o inválidos se tratan como errores del plugin y bloquean la validación de la configuración. Consulta la guía completa del sistema de plugins: [Plugins](../tools/plugin.md).

## Campos obligatorios

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

Claves obligatorias:

-   `id` (string): identificador canónico del plugin.
-   `configSchema` (object): Esquema JSON para la configuración del plugin (en línea).

Claves opcionales:

-   `kind` (string): tipo de plugin (ejemplos: `"memory"`, `"context-engine"`).
-   `channels` (array): identificadores de canales registrados por este plugin (ejemplo: `["matrix"]`).
-   `providers` (array): identificadores de proveedores registrados por este plugin.
-   `skills` (array): directorios de habilidades a cargar (relativos a la raíz del plugin).
-   `name` (string): nombre para mostrar del plugin.
-   `description` (string): resumen breve del plugin.
-   `uiHints` (object): etiquetas/marcadores de posición/indicadores sensibles para campos de configuración para renderizado en la UI.
-   `version` (string): versión del plugin (informativa).

## Requisitos del Esquema JSON

-   **Cada plugin debe incluir un Esquema JSON**, incluso si no acepta configuración.
-   Un esquema vacío es aceptable (por ejemplo, `{ "type": "object", "additionalProperties": false }`).
-   Los esquemas se validan al momento de lectura/escritura de la configuración, no en tiempo de ejecución.

## Comportamiento de validación

-   Las claves desconocidas en `channels.*` son **errores**, a menos que el identificador del canal sea declarado por un manifiesto de plugin.
-   `plugins.entries.`, `plugins.allow`, `plugins.deny`, y `plugins.slots.*` deben hacer referencia a identificadores de plugin **detectables**. Los identificadores desconocidos son **errores**.
-   Si un plugin está instalado pero tiene un manifiesto o esquema roto o faltante, la validación falla y Doctor reporta el error del plugin.
-   Si existe configuración del plugin pero el plugin está **deshabilitado**, la configuración se mantiene y se muestra una **advertencia** en Doctor + registros.

## Notas

-   El manifiesto es **obligatorio para todos los plugins**, incluidas las cargas desde el sistema de archivos local.
-   El tiempo de ejecución aún carga el módulo del plugin por separado; el manifiesto es solo para detección + validación.
-   Los tipos de plugin exclusivos se seleccionan a través de `plugins.slots.*`.
    -   `kind: "memory"` es seleccionado por `plugins.slots.memory`.
    -   `kind: "context-engine"` es seleccionado por `plugins.slots.contextEngine` (predeterminado: el integrado `legacy`).
-   Si tu plugin depende de módulos nativos, documenta los pasos de compilación y cualquier requisito de lista de permitidos del gestor de paquetes (por ejemplo, `allow-build-scripts` de pnpm
    -   `pnpm rebuild `).

[Plugin Personal de Zalo](./zalouser.md)[Herramientas del Agente de Plugin](./agent-tools.md)