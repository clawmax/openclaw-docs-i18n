title: "API HTTP OpenResponses para Configuración y Uso de OpenClaw Gateway"
description: "Aprende cómo habilitar y usar el endpoint POST /v1/responses compatible con OpenResponses en OpenClaw Gateway. Configura autenticación, seguridad, agentes, herramientas y manejo de archivos."
keywords: ["api openresponses", "openclaw gateway", "api http", "integración de agentes", "autenticación api", "herramientas cliente", "carga de archivos", "streaming sse"]
---

  Protocolos y APIs

  
# API OpenResponses

El Gateway de OpenClaw puede servir un endpoint `POST /v1/responses` compatible con OpenResponses. Este endpoint está **deshabilitado por defecto**. Habilítalo primero en la configuración.

-   `POST /v1/responses`
-   Mismo puerto que el Gateway (WS + HTTP multiplexado): `http://<gateway-host>:/v1/responses`

Internamente, las solicitudes se ejecutan como una ejecución normal de agente del Gateway (misma ruta de código que `openclaw agent`), por lo que el enrutamiento/permisos/configuración coinciden con tu Gateway.

## Autenticación

Utiliza la configuración de autenticación del Gateway. Envía un token bearer:

-   `Authorization: Bearer `

Notas:

-   Cuando `gateway.auth.mode="token"`, usa `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`).
-   Cuando `gateway.auth.mode="password"`, usa `gateway.auth.password` (o `OPENCLAW_GATEWAY_PASSWORD`).
-   Si `gateway.auth.rateLimit` está configurado y ocurren demasiados fallos de autenticación, el endpoint devuelve `429` con `Retry-After`.

## Límite de seguridad (importante)

Trata este endpoint como una superficie de **acceso completo de operador** para la instancia del gateway.

-   La autenticación HTTP bearer aquí no es un modelo de alcance estrecho por usuario.
-   Un token/contraseña válido del Gateway para este endpoint debe tratarse como una credencial de propietario/operador.
-   Las solicitudes pasan por la misma ruta de agente del plano de control que las acciones de operador confiables.
-   No hay un límite de herramientas separado para no propietarios/por usuario en este endpoint; una vez que un llamante pasa la autenticación del Gateway aquí, OpenClaw trata a ese llamante como un operador confiable para este gateway.
-   Si la política del agente objetivo permite herramientas sensibles, este endpoint puede usarlas.
-   Mantén este endpoint solo en loopback/tailnet/ingreso privado; no lo expongas directamente a la internet pública.

Consulta [Seguridad](./security.md) y [Acceso remoto](./remote.md).

## Eligiendo un agente

No se requieren cabeceras personalizadas: codifica el id del agente en el campo `model` de OpenResponses:

-   `model: "openclaw:"` (ejemplo: `"openclaw:main"`, `"openclaw:beta"`)
-   `model: "agent:"` (alias)

O apunta a un agente específico de OpenClaw por cabecera:

-   `x-openclaw-agent-id: ` (por defecto: `main`)

Avanzado:

-   `x-openclaw-session-key: ` para controlar completamente el enrutamiento de sesión.

## Habilitando el endpoint

Establece `gateway.http.endpoints.responses.enabled` en `true`:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true },
      },
    },
  },
}
```

## Deshabilitando el endpoint

Establece `gateway.http.endpoints.responses.enabled` en `false`:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false },
      },
    },
  },
}
```

## Comportamiento de sesión

Por defecto el endpoint es **sin estado por solicitud** (se genera una nueva clave de sesión en cada llamada). Si la solicitud incluye una cadena `user` de OpenResponses, el Gateway deriva una clave de sesión estable a partir de ella, por lo que las llamadas repetidas pueden compartir una sesión de agente.

## Forma de la solicitud (soportada)

La solicitud sigue la API de OpenResponses con entrada basada en ítems. Soporte actual:

-   `input`: cadena o array de objetos ítem.
-   `instructions`: se fusiona en el prompt del sistema.
-   `tools`: definiciones de herramientas del cliente (herramientas de función).
-   `tool_choice`: filtrar o requerir herramientas del cliente.
-   `stream`: habilita streaming SSE.
-   `max_output_tokens`: límite de salida de mejor esfuerzo (dependiente del proveedor).
-   `user`: enrutamiento de sesión estable.

Aceptados pero **actualmente ignorados**:

-   `max_tool_calls`
-   `reasoning`
-   `metadata`
-   `store`
-   `previous_response_id`
-   `truncation`

## Ítems (input)

### message

Roles: `system`, `developer`, `user`, `assistant`.

-   `system` y `developer` se añaden al prompt del sistema.
-   El ítem `user` o `function_call_output` más reciente se convierte en el "mensaje actual".
-   Los mensajes user/assistant anteriores se incluyen como historial para contexto.

### function\_call\_output (herramientas por turnos)

Envía los resultados de la herramienta de vuelta al modelo:

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```

### reasoning y item\_reference

Aceptados para compatibilidad del esquema pero ignorados al construir el prompt.

## Herramientas (herramientas de función del lado del cliente)

Proporciona herramientas con `tools: [{ type: "function", function: { name, description?, parameters? } }]`. Si el agente decide llamar a una herramienta, la respuesta devuelve un ítem de salida `function_call`. Luego envías una solicitud de seguimiento con `function_call_output` para continuar el turno.

## Imágenes (input\_image)

Soporta fuentes base64 o URL:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Tipos MIME permitidos (actual): `image/jpeg`, `image/png`, `image/gif`, `image/webp`, `image/heic`, `image/heif`. Tamaño máximo (actual): 10MB.

## Archivos (input\_file)

Soporta fuentes base64 o URL:

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

Tipos MIME permitidos (actual): `text/plain`, `text/markdown`, `text/html`, `text/csv`, `application/json`, `application/pdf`. Tamaño máximo (actual): 5MB. Comportamiento actual:

-   El contenido del archivo se decodifica y se añade al **prompt del sistema**, no al mensaje del usuario, por lo que permanece efímero (no persiste en el historial de sesión).
-   Los PDFs se analizan para extraer texto. Si se encuentra poco texto, las primeras páginas se rasterizan en imágenes y se pasan al modelo.

El análisis de PDF usa la compilación legacy `pdfjs-dist` compatible con Node (sin worker). La compilación moderna de PDF.js espera workers/DOM globales del navegador, por lo que no se usa en el Gateway. Valores por defecto de fetch de URL:

-   `files.allowUrl`: `true`
-   `images.allowUrl`: `true`
-   `maxUrlParts`: `8` (total de partes `input_file` + `input_image` basadas en URL por solicitud)
-   Las solicitudes están protegidas (resolución DNS, bloqueo de IPs privadas, límite de redirecciones, timeouts).
-   Se admiten listas blancas de nombres de host opcionales por tipo de entrada (`files.urlAllowlist`, `images.urlAllowlist`).
    -   Host exacto: `"cdn.example.com"`
    -   Subdominios comodín: `"*.assets.example.com"` (no coincide con el apex)

## Límites de archivos + imágenes (config)

Los valores por defecto se pueden ajustar bajo `gateway.http.endpoints.responses`:

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          maxUrlParts: 8,
          files: {
            allowUrl: true,
            urlAllowlist: ["cdn.example.com", "*.assets.example.com"],
            allowedMimes: [
              "text/plain",
              "text/markdown",
              "text/html",
              "text/csv",
              "application/json",
              "application/pdf",
            ],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200,
            },
          },
          images: {
            allowUrl: true,
            urlAllowlist: ["images.example.com"],
            allowedMimes: [
              "image/jpeg",
              "image/png",
              "image/gif",
              "image/webp",
              "image/heic",
              "image/heif",
            ],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000,
          },
        },
      },
    },
  },
}
```

Valores por defecto cuando se omiten:

-   `maxBodyBytes`: 20MB
-   `maxUrlParts`: 8
-   `files.maxBytes`: 5MB
-   `files.maxChars`: 200k
-   `files.maxRedirects`: 3
-   `files.timeoutMs`: 10s
-   `files.pdf.maxPages`: 4
-   `files.pdf.maxPixels`: 4,000,000
-   `files.pdf.minTextChars`: 200
-   `images.maxBytes`: 10MB
-   `images.maxRedirects`: 3
-   `images.timeoutMs`: 10s
-   Las fuentes `input_image` HEIC/HEIF se aceptan y normalizan a JPEG antes de la entrega al proveedor.

Nota de seguridad:

-   Las listas blancas de URL se aplican antes del fetch y en los saltos de redirección.
-   Permitir un nombre de host no evita el bloqueo de IPs privadas/internas.
-   Para gateways expuestos a internet, aplica controles de salida de red además de las protecciones a nivel de aplicación. Consulta [Seguridad](./security.md).

## Streaming (SSE)

Establece `stream: true` para recibir Eventos Enviados por el Servidor (SSE):

-   `Content-Type: text/event-stream`
-   Cada línea de evento es `event: ` y `data: `
-   El stream termina con `data: [DONE]`

Tipos de evento emitidos actualmente:

-   `response.created`
-   `response.in_progress`
-   `response.output_item.added`
-   `response.content_part.added`
-   `response.output_text.delta`
-   `response.output_text.done`
-   `response.content_part.done`
-   `response.output_item.done`
-   `response.completed`
-   `response.failed` (en error)

## Uso

`usage` se completa cuando el proveedor subyacente reporta conteos de tokens.

## Errores

Los errores usan un objeto JSON como:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

Casos comunes:

-   `401` autenticación faltante/inválida
-   `400` cuerpo de solicitud inválido
-   `405` método incorrecto

## Ejemplos

Sin streaming:

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

Con streaming:

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```

[Chat Completions de OpenAI](./openai-http-api.md)[API Tools Invoke](./tools-invoke-http-api.md)