

  Experimentos

  
# Protocolo de Incorporación y Configuración

Propósito: superficies compartidas de incorporación + configuración en CLI, aplicación macOS e Interfaz Web.

## Componentes

-   Motor de asistente (sesión compartida + prompts + estado de incorporación).
-   La incorporación por CLI utiliza el mismo flujo de asistente que los clientes de UI.
-   Gateway RPC expone endpoints de asistente + esquema de configuración.
-   La incorporación en macOS utiliza el modelo de pasos del asistente.
-   La Interfaz Web renderiza formularios de configuración a partir de JSON Schema + sugerencias de UI.

## Gateway RPC

-   `wizard.start` parámetros: `{ mode?: "local"|"remote", workspace?: string }`
-   `wizard.next` parámetros: `{ sessionId, answer?: { stepId, value? } }`
-   `wizard.cancel` parámetros: `{ sessionId }`
-   `wizard.status` parámetros: `{ sessionId }`
-   `config.schema` parámetros: `{}`
-   `config.schema.lookup` parámetros: `{ path }`
    -   `path` acepta segmentos de configuración estándar más IDs de plugin delimitados por barra, por ejemplo `plugins.entries.pack/one.config`.

Respuestas (forma)

-   Asistente: `{ sessionId, done, step?, status?, error? }`
-   Esquema de configuración: `{ schema, uiHints, version, generatedAt }`
-   Búsqueda de esquema de configuración: `{ path, schema, hint?, hintPath?, children[] }`

## Sugerencias de UI

-   `uiHints` indexado por ruta; metadatos opcionales (etiqueta/ayuda/grupo/orden/avanzado/sensible/marcador de posición).
-   Los campos sensibles se renderizan como entradas de contraseña; sin capa de redacción.
-   Los nodos de esquema no soportados recurren al editor JSON crudo.

## Notas

-   Este documento es el único lugar para rastrear refactorizaciones del protocolo para incorporación/configuración.

[Integración de gateway Kilo](../design/kilo-gateway-integration.md)[Agentes Vinculados a Hilo ACP](./plans/acp-thread-bound-agents.md)

---