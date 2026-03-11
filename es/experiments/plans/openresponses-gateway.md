

  Experimentos

  
# Plan de Gateway OpenResponses

## Contexto

OpenClaw Gateway actualmente expone un endpoint mínimo compatible con OpenAI Chat Completions en `/v1/chat/completions` (ver [OpenAI Chat Completions](../../gateway/openai-http-api.md)). Open Responses es un estándar de inferencia abierto basado en la API de Respuestas de OpenAI. Está diseñado para flujos de trabajo agenticos y utiliza entradas basadas en ítems más eventos de streaming semántico. La especificación OpenResponses define `/v1/responses`, no `/v1/chat/completions`.

## Objetivos

-   Añadir un endpoint `/v1/responses` que se adhiera a la semántica de OpenResponses.
-   Mantener Chat Completions como una capa de compatibilidad que sea fácil de deshabilitar y eventualmente eliminar.
-   Estandarizar la validación y el análisis con esquemas reutilizables y aislados.

## No objetivos

-   Paridad completa de características de OpenResponses en la primera iteración (imágenes, archivos, herramientas alojadas).
-   Reemplazar la lógica interna de ejecución de agentes o la orquestación de herramientas.
-   Cambiar el comportamiento existente de `/v1/chat/completions` durante la primera fase.

## Resumen de Investigación

Fuentes: OpenAPI de OpenResponses, sitio de especificación de OpenResponses y la publicación del blog de Hugging Face. Puntos clave extraídos:

-   `POST /v1/responses` acepta campos `CreateResponseBody` como `model`, `input` (string o `ItemParam[]`), `instructions`, `tools`, `tool_choice`, `stream`, `max_output_tokens`, y `max_tool_calls`.
-   `ItemParam` es una unión discriminada de:
    -   ítems `message` con roles `system`, `developer`, `user`, `assistant`
    -   `function_call` y `function_call_output`
    -   `reasoning`
    -   `item_reference`
-   Las respuestas exitosas devuelven un `ResponseResource` con `object: "response"`, `status`, e ítems `output`.
-   El streaming utiliza eventos semánticos como:
    -   `response.created`, `response.in_progress`, `response.completed`, `response.failed`
    -   `response.output_item.added`, `response.output_item.done`
    -   `response.content_part.added`, `response.content_part.done`
    -   `response.output_text.delta`, `response.output_text.done`
-   La especificación requiere:
    -   `Content-Type: text/event-stream`
    -   `event:` debe coincidir con el campo JSON `type`
    -   el evento terminal debe ser el literal `[DONE]`
-   Los ítems de razonamiento pueden exponer `content`, `encrypted_content`, y `summary`.
-   Los ejemplos de HF incluyen `OpenResponses-Version: latest` en las solicitudes (cabecera opcional).

## Arquitectura Propuesta

-   Añadir `src/gateway/open-responses.schema.ts` que contenga solo esquemas Zod (sin importaciones de gateway).
-   Añadir `src/gateway/openresponses-http.ts` (o `open-responses-http.ts`) para `/v1/responses`.
-   Mantener `src/gateway/openai-http.ts` intacto como un adaptador de compatibilidad heredado.
-   Añadir la configuración `gateway.http.endpoints.responses.enabled` (por defecto `false`).
-   Mantener `gateway.http.endpoints.chatCompletions.enabled` independiente; permitir que ambos endpoints se activen/desactiven por separado.
-   Emitir una advertencia al inicio cuando Chat Completions esté habilitado para señalar su estado heredado.

## Ruta de Desuso para Chat Completions

-   Mantener límites estrictos de módulos: sin tipos de esquema compartidos entre responses y chat completions.
-   Hacer que Chat Completions sea opcional por configuración para que pueda deshabilitarse sin cambios de código.
-   Actualizar la documentación para etiquetar Chat Completions como heredado una vez que `/v1/responses` sea estable.
-   Paso futuro opcional: mapear solicitudes de Chat Completions al manejador de Responses para una ruta de eliminación más simple.

## Subconjunto Soportado en Fase 1

-   Aceptar `input` como string o `ItemParam[]` con roles de mensaje y `function_call_output`.
-   Extraer mensajes de sistema y desarrollador en `extraSystemPrompt`.
-   Usar el `user` o `function_call_output` más reciente como el mensaje actual para las ejecuciones del agente.
-   Rechazar partes de contenido no soportadas (imagen/archivo) con `invalid_request_error`.
-   Devolver un único mensaje de asistente con contenido `output_text`.
-   Devolver `usage` con valores en cero hasta que se conecte la contabilidad de tokens.

## Estrategia de Validación (Sin SDK)

-   Implementar esquemas Zod para el subconjunto soportado de:
    -   `CreateResponseBody`
    -   `ItemParam` + uniones de partes de contenido de mensaje
    -   `ResponseResource`
    -   Formas de eventos de streaming utilizadas por el gateway
-   Mantener los esquemas en un único módulo aislado para evitar desviaciones y permitir futura generación de código.

## Implementación de Streaming (Fase 1)

-   Líneas SSE con `event:` y `data:`.
-   Secuencia requerida (mínimo viable):
    -   `response.created`
    -   `response.output_item.added`
    -   `response.content_part.added`
    -   `response.output_text.delta` (repetir según sea necesario)
    -   `response.output_text.done`
    -   `response.content_part.done`
    -   `response.completed`
    -   `[DONE]`

## Plan de Pruebas y Verificación

-   Añadir cobertura e2e para `/v1/responses`:
    -   Autenticación requerida
    -   Forma de respuesta sin streaming
    -   Orden de eventos de streaming y `[DONE]`
    -   Enrutamiento de sesión con cabeceras y `user`
-   Mantener `src/gateway/openai-http.test.ts` sin cambios.
-   Manual: curl a `/v1/responses` con `stream: true` y verificar el orden de eventos y el terminal `[DONE]`.

## Actualizaciones de Documentación (Seguimiento)

-   Añadir una nueva página de documentación para el uso y ejemplos de `/v1/responses`.
-   Actualizar `/gateway/openai-http-api` con una nota de legado y un enlace a `/v1/responses`.

[Refactorización de Browser Evaluate CDP](./browser-evaluate-cdp-refactor.md)[Plan de PTY y Supervisión de Procesos](./pty-process-supervision.md)