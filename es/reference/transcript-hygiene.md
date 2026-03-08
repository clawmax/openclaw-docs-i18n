

  Referencia técnica

  
# Higiene de Transcripciones

Este documento describe las **correcciones específicas del proveedor** aplicadas a las transcripciones antes de una ejecución (construcción del contexto del modelo). Estos son ajustes **en memoria** utilizados para satisfacer los requisitos estrictos de los proveedores. Estos pasos de higiene **no** reescriben la transcripción JSONL almacenada en disco; sin embargo, un paso de reparación de archivos de sesión separado puede reescribir archivos JSONL malformados eliminando líneas inválidas antes de que se cargue la sesión. Cuando ocurre una reparación, el archivo original se respalda junto al archivo de sesión. El alcance incluye:

-   Sanitización de ID de llamadas a herramientas
-   Validación de entrada de llamadas a herramientas
-   Reparación de emparejamiento de resultados de herramientas
-   Validación / ordenación de turnos
-   Limpieza de firmas de pensamiento
-   Sanitización de cargas de imágenes
-   Etiquetado de procedencia de entrada del usuario (para prompts enrutados entre sesiones)

Si necesitas detalles sobre el almacenamiento de transcripciones, consulta:

-   [/reference/session-management-compaction](./session-management-compaction.md)

* * *

## Dónde se ejecuta

Toda la higiene de transcripciones está centralizada en el ejecutor integrado:

-   Selección de política: `src/agents/transcript-policy.ts`
-   Aplicación de sanitización/reparación: `sanitizeSessionHistory` en `src/agents/pi-embedded-runner/google.ts`

La política utiliza `provider`, `modelApi` y `modelId` para decidir qué aplicar. Separado de la higiene de transcripciones, los archivos de sesión se reparan (si es necesario) antes de cargar:

-   `repairSessionFileIfNeeded` en `src/agents/session-file-repair.ts`
-   Llamado desde `run/attempt.ts` y `compact.ts` (ejecutor integrado)

* * *

## Regla global: sanitización de imágenes

Las cargas de imágenes siempre se sanitizan para evitar el rechazo por parte del proveedor debido a límites de tamaño (reducir escala/recomprimir imágenes base64 demasiado grandes). Esto también ayuda a controlar la presión de tokens impulsada por imágenes para modelos con capacidad de visión. Dimensiones máximas más bajas generalmente reducen el uso de tokens; dimensiones más altas preservan el detalle. Implementación:

-   `sanitizeSessionMessagesImages` en `src/agents/pi-embedded-helpers/images.ts`
-   `sanitizeContentBlocksImages` en `src/agents/tool-images.ts`
-   El lado máximo de la imagen es configurable mediante `agents.defaults.imageMaxDimensionPx` (predeterminado: `1200`).

* * *

## Regla global: llamadas a herramientas malformadas

Los bloques de llamadas a herramientas del asistente que carecen tanto de `input` como de `arguments` se eliminan antes de construir el contexto del modelo. Esto evita rechazos del proveedor por llamadas a herramientas parcialmente persistidas (por ejemplo, después de un fallo por límite de tasa). Implementación:

-   `sanitizeToolCallInputs` en `src/agents/session-transcript-repair.ts`
-   Aplicado en `sanitizeSessionHistory` en `src/agents/pi-embedded-runner/google.ts`

* * *

## Regla global: procedencia de entrada entre sesiones

Cuando un agente envía un prompt a otra sesión a través de `sessions_send` (incluyendo pasos de respuesta/anuncio de agente a agente), OpenClaw persiste el turno de usuario creado con:

-   `message.provenance.kind = "inter_session"`

Estos metadatos se escriben en el momento de la anexión de la transcripción y no cambian el rol (`role: "user"` permanece para compatibilidad con el proveedor). Los lectores de transcripciones pueden usar esto para evitar tratar los prompts internos enrutados como instrucciones escritas por el usuario final. Durante la reconstrucción del contexto, OpenClaw también antepone un breve marcador `[Mensaje entre sesiones]` a esos turnos de usuario en memoria para que el modelo pueda distinguirlos de las instrucciones externas del usuario final.

* * *

## Matriz de proveedores (comportamiento actual)

**OpenAI / OpenAI Codex**

-   Solo sanitización de imágenes.
-   Eliminar firmas de razonamiento huérfanas (ítems de razonamiento independientes sin un bloque de contenido siguiente) para transcripciones de OpenAI Responses/Codex.
-   Sin sanitización de ID de llamadas a herramientas.
-   Sin reparación de emparejamiento de resultados de herramientas.
-   Sin validación de turnos o reordenación.
-   Sin resultados de herramientas sintéticos.
-   Sin eliminación de firmas de pensamiento.

**Google (Generative AI / Gemini CLI / Antigravity)**

-   Sanitización de ID de llamadas a herramientas: alfanumérica estricta.
-   Reparación de emparejamiento de resultados de herramientas y resultados de herramientas sintéticos.
-   Validación de turnos (alternancia de turnos estilo Gemini).
-   Corrección de ordenación de turnos de Google (anteponer un pequeño bootstrap de usuario si el historial comienza con asistente).
-   Antigravity Claude: normalizar firmas de pensamiento; eliminar bloques de pensamiento sin firmar.

**Anthropic / Minimax (compatible con Anthropic)**

-   Reparación de emparejamiento de resultados de herramientas y resultados de herramientas sintéticos.
-   Validación de turnos (fusionar turnos de usuario consecutivos para satisfacer la alternancia estricta).

**Mistral (incluyendo detección basada en model-id)**

-   Sanitización de ID de llamadas a herramientas: strict9 (longitud alfanumérica 9).

**OpenRouter Gemini**

-   Limpieza de firmas de pensamiento: eliminar valores `thought_signature` que no sean base64 (conservar base64).

**Todo lo demás**

-   Solo sanitización de imágenes.

* * *

## Comportamiento histórico (pre-2026.1.22)

Antes del lanzamiento 2026.1.22, OpenClaw aplicaba múltiples capas de higiene de transcripciones:

-   Una **extensión transcript-sanitize** se ejecutaba en cada construcción de contexto y podía:
    -   Reparar el emparejamiento de uso/resultado de herramientas.
    -   Sanitizar IDs de llamadas a herramientas (incluyendo un modo no estricto que preservaba `_`/`-`).
-   El ejecutor también realizaba sanitización específica del proveedor, lo que duplicaba el trabajo.
-   Ocurrieron mutaciones adicionales fuera de la política del proveedor, incluyendo:
    -   Eliminar etiquetas `` del texto del asistente antes de la persistencia.
    -   Eliminar turnos de error del asistente vacíos.
    -   Recortar contenido del asistente después de llamadas a herramientas.

Esta complejidad causó regresiones entre proveedores (notablemente el emparejamiento `call_id|fc_id` de `openai-responses`). La limpieza de 2026.1.22 eliminó la extensión, centralizó la lógica en el ejecutor e hizo que OpenAI fuera **no-touch** más allá de la sanitización de imágenes.

[Uso y Costos de la API](./api-usage-costs.md)[Fecha y Hora](../date-time.md)