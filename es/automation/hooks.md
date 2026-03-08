

  Automatización

  
# Ganchos

Los ganchos proporcionan un sistema extensible dirigido por eventos para automatizar acciones en respuesta a comandos y eventos del agente. Los ganchos se descubren automáticamente desde directorios y se pueden gestionar mediante comandos CLI, de manera similar a cómo funcionan las habilidades en OpenClaw.

## Orientación

Los ganchos son pequeños scripts que se ejecutan cuando algo sucede. Hay dos tipos:

-   **Ganchos** (esta página): se ejecutan dentro del Gateway cuando se disparan eventos del agente, como `/new`, `/reset`, `/stop` o eventos del ciclo de vida.
-   **Webhooks**: webhooks HTTP externos que permiten a otros sistemas activar trabajo en OpenClaw. Consulta [Ganchos Webhook](./webhook.md) o usa `openclaw webhooks` para comandos auxiliares de Gmail.

Los ganchos también se pueden agrupar dentro de plugins; consulta [Plugins](../tools/plugin.md#plugin-hooks). Usos comunes:

-   Guardar una instantánea de memoria cuando reinicias una sesión
-   Mantener un rastro de auditoría de comandos para solución de problemas o cumplimiento
-   Activar automatización de seguimiento cuando una sesión comienza o termina
-   Escribir archivos en el espacio de trabajo del agente o llamar a APIs externas cuando se disparan eventos

Si puedes escribir una pequeña función TypeScript, puedes escribir un gancho. Los ganchos se descubren automáticamente, y los habilitas o deshabilitas a través de la CLI.

## Descripción General

El sistema de ganchos te permite:

-   Guardar el contexto de la sesión en la memoria cuando se emite `/new`
-   Registrar todos los comandos para auditoría
-   Activar automatizaciones personalizadas en eventos del ciclo de vida del agente
-   Extender el comportamiento de OpenClaw sin modificar el código central

## Comenzando

### Ganchos Incluidos

OpenClaw incluye cuatro ganchos que se descubren automáticamente:

-   **💾 session-memory**: Guarda el contexto de la sesión en tu espacio de trabajo del agente (por defecto `~/.openclaw/workspace/memory/`) cuando emites `/new`
-   **📎 bootstrap-extra-files**: Inyecta archivos de arranque adicionales del espacio de trabajo desde patrones de glob/ruta configurados durante `agent:bootstrap`
-   **📝 command-logger**: Registra todos los eventos de comando en `~/.openclaw/logs/commands.log`
-   **🚀 boot-md**: Ejecuta `BOOT.md` cuando el gateway inicia (requiere que los ganchos internos estén habilitados)

Lista los ganchos disponibles:

```bash
openclaw hooks list
```

Habilita un gancho:

```bash
openclaw hooks enable session-memory
```

Verifica el estado del gancho:

```bash
openclaw hooks check
```

Obtén información detallada:

```bash
openclaw hooks info session-memory
```

### Incorporación

Durante la incorporación (`openclaw onboard`), se te pedirá que habilites ganchos recomendados. El asistente descubre automáticamente los ganchos elegibles y los presenta para su selección.

## Descubrimiento de Ganchos

Los ganchos se descubren automáticamente desde tres directorios (en orden de precedencia):

1.  **Ganchos del espacio de trabajo**: `/hooks/` (por agente, mayor precedencia)
2.  **Ganchos gestionados**: `~/.openclaw/hooks/` (instalados por el usuario, compartidos entre espacios de trabajo)
3.  **Ganchos incluidos**: `/dist/hooks/bundled/` (incluidos con OpenClaw)

Los directorios de ganchos gestionados pueden ser un **único gancho** o un **paquete de ganchos** (directorio de paquete). Cada gancho es un directorio que contiene:

```
my-hook/
├── HOOK.md          # Metadatos + documentación
└── handler.ts       # Implementación del manejador
```

## Paquetes de Ganchos (npm/archivos)

Los paquetes de ganchos son paquetes npm estándar que exportan uno o más ganchos a través de `openclaw.hooks` en `package.json`. Instálalos con:

```bash
openclaw hooks install <path-or-spec>
```

Las especificaciones npm son solo de registro (nombre del paquete + versión exacta opcional o etiqueta de distribución). Se rechazan las especificaciones Git/URL/archivo y los rangos semver. Las especificaciones simples y `@latest` permanecen en la vía estable. Si npm resuelve cualquiera de esos a una versión preliminar, OpenClaw se detiene y te pide que aceptes explícitamente con una etiqueta preliminar como `@beta`/`@rc` o una versión preliminar exacta. Ejemplo de `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Cada entrada apunta a un directorio de gancho que contiene `HOOK.md` y `handler.ts` (o `index.ts`). Los paquetes de ganchos pueden incluir dependencias; se instalarán bajo `~/.openclaw/hooks/`. Cada entrada `openclaw.hooks` debe permanecer dentro del directorio del paquete después de la resolución de enlaces simbólicos; se rechazan las entradas que escapan. Nota de seguridad: `openclaw hooks install` instala dependencias con `npm install --ignore-scripts` (sin scripts del ciclo de vida). Mantén los árboles de dependencias del paquete de ganchos "puro JS/TS" y evita paquetes que dependan de compilaciones `postinstall`.

## Estructura del Gancho

### Formato de HOOK.md

El archivo `HOOK.md` contiene metadatos en frontmatter YAML más documentación en Markdown:

```
---
name: my-hook
description: "Descripción corta de lo que hace este gancho"
homepage: https://docs.openclaw.ai/automation/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# Mi Gancho

La documentación detallada va aquí...

## Qué Hace

- Escucha comandos `/new`
- Realiza alguna acción
- Registra el resultado

## Requisitos

- Node.js debe estar instalado

## Configuración

No se necesita configuración.
```

### Campos de Metadatos

El objeto `metadata.openclaw` admite:

-   **`emoji`**: Emoji para mostrar en CLI (ej., `"💾"`)
-   **`events`**: Array de eventos a los que escuchar (ej., `["command:new", "command:reset"]`)
-   **`export`**: Exportación nombrada a usar (por defecto `"default"`)
-   **`homepage`**: URL de documentación
-   **`requires`**: Requisitos opcionales
    -   **`bins`**: Binarios requeridos en PATH (ej., `["git", "node"]`)
    -   **`anyBins`**: Al menos uno de estos binarios debe estar presente
    -   **`env`**: Variables de entorno requeridas
    -   **`config`**: Rutas de configuración requeridas (ej., `["workspace.dir"]`)
    -   **`os`**: Plataformas requeridas (ej., `["darwin", "linux"]`)
-   **`always`**: Omite las comprobaciones de elegibilidad (booleano)
-   **`install`**: Métodos de instalación (para ganchos incluidos: `[{"id":"bundled","kind":"bundled"}]`)

### Implementación del Manejador

El archivo `handler.ts` exporta una función `HookHandler`:

```typescript
const myHandler = async (event) => {
  // Solo activar en comando 'new'
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] Comando new activado`);
  console.log(`  Sesión: ${event.sessionKey}`);
  console.log(`  Marca de tiempo: ${event.timestamp.toISOString()}`);

  // Tu lógica personalizada aquí

  // Opcionalmente enviar mensaje al usuario
  event.messages.push("✨ ¡Mi gancho se ejecutó!");
};

export default myHandler;
```

#### Contexto del Evento

Cada evento incluye:

```json
{
  type: 'command' | 'session' | 'agent' | 'gateway' | 'message',
  action: string,              // ej., 'new', 'reset', 'stop', 'received', 'sent'
  sessionKey: string,          // Identificador de sesión
  timestamp: Date,             // Cuándo ocurrió el evento
  messages: string[],          // Empuja mensajes aquí para enviar al usuario
  context: {
    // Eventos de comando:
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // ej., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig,
    // Eventos de mensaje (ver sección Eventos de Mensaje para detalles completos):
    from?: string,             // message:received
    to?: string,               // message:sent
    content?: string,
    channelId?: string,
    success?: boolean,         // message:sent
  }
}
```

## Tipos de Eventos

### Eventos de Comando

Se activan cuando se emiten comandos del agente:

-   **`command`**: Todos los eventos de comando (oyente general)
-   **`command:new`**: Cuando se emite el comando `/new`
-   **`command:reset`**: Cuando se emite el comando `/reset`
-   **`command:stop`**: Cuando se emite el comando `/stop`

### Eventos de Sesión

-   **`session:compact:before`**: Justo antes de que la compactación resuma el historial
-   **`session:compact:after`**: Después de que la compactación se completa con metadatos de resumen

Las cargas útiles de ganchos internos emiten estos como `type: "session"` con `action: "compact:before"` / `action: "compact:after"`; los oyentes se suscriben con las claves combinadas anteriores. El registro específico del manejador usa el formato literal de clave `${type}:${action}`. Para estos eventos, registra `session:compact:before` y `session:compact:after`.

### Eventos del Agente

-   **`agent:bootstrap`**: Antes de que se inyecten los archivos de arranque del espacio de trabajo (los ganchos pueden mutar `context.bootstrapFiles`)

### Eventos del Gateway

Se activan cuando el gateway inicia:

-   **`gateway:startup`**: Después de que los canales inician y los ganchos se cargan

### Eventos de Mensaje

Se activan cuando se reciben o envían mensajes:

-   **`message`**: Todos los eventos de mensaje (oyente general)
-   **`message:received`**: Cuando se recibe un mensaje entrante desde cualquier canal. Se dispara temprano en el procesamiento antes de la comprensión de medios. El contenido puede contener marcadores de posición crudos como `<media:audio>` para archivos adjuntos de medios que aún no se han procesado.
-   **`message:transcribed`**: Cuando un mensaje ha sido procesado completamente, incluyendo transcripción de audio y comprensión de enlaces. En este punto, `transcript` contiene el texto completo de la transcripción para mensajes de audio. Usa este gancho cuando necesites acceso al contenido de audio transcrito.
-   **`message:preprocessed`**: Se dispara para cada mensaje después de que se completa toda la comprensión de medios + enlaces, dando a los ganchos acceso al cuerpo completamente enriquecido (transcripciones, descripciones de imágenes, resúmenes de enlaces) antes de que el agente lo vea.
-   **`message:sent`**: Cuando un mensaje saliente se envía con éxito

#### Contexto del Evento de Mensaje

Los eventos de mensaje incluyen contexto rico sobre el mensaje:

```
// contexto de message:received
{
  from: string,           // Identificador del remitente (número de teléfono, ID de usuario, etc.)
  content: string,        // Contenido del mensaje
  timestamp?: number,     // Marca de tiempo Unix cuando se recibió
  channelId: string,      // Canal (ej., "whatsapp", "telegram", "discord")
  accountId?: string,     // ID de cuenta del proveedor para configuraciones multi-cuenta
  conversationId?: string, // ID de chat/conversación
  messageId?: string,     // ID del mensaje del proveedor
  metadata?: {            // Datos adicionales específicos del proveedor
    to?: string,
    provider?: string,
    surface?: string,
    threadId?: string,
    senderId?: string,
    senderName?: string,
    senderUsername?: string,
    senderE164?: string,
  }
}

// contexto de message:sent
{
  to: string,             // Identificador del destinatario
  content: string,        // Contenido del mensaje que se envió
  success: boolean,       // Si el envío tuvo éxito
  error?: string,         // Mensaje de error si falló el envío
  channelId: string,      // Canal (ej., "whatsapp", "telegram", "discord")
  accountId?: string,     // ID de cuenta del proveedor
  conversationId?: string, // ID de chat/conversación
  messageId?: string,     // ID del mensaje devuelto por el proveedor
  isGroup?: boolean,      // Si este mensaje saliente pertenece a un contexto de grupo/canal
  groupId?: string,       // Identificador de grupo/canal para correlación con message:received
}

// contexto de message:transcribed
{
  body?: string,          // Cuerpo entrante crudo antes del enriquecimiento
  bodyForAgent?: string,  // Cuerpo enriquecido visible para el agente
  transcript: string,     // Texto de la transcripción de audio
  channelId: string,      // Canal (ej., "telegram", "whatsapp")
  conversationId?: string,
  messageId?: string,
}

// contexto de message:preprocessed
{
  body?: string,          // Cuerpo entrante crudo
  bodyForAgent?: string,  // Cuerpo enriquecido final después de la comprensión de medios/enlaces
  transcript?: string,    // Transcripción cuando había audio presente
  channelId: string,      // Canal (ej., "telegram", "whatsapp")
  conversationId?: string,
  messageId?: string,
  isGroup?: boolean,
  groupId?: string,
}
```

#### Ejemplo: Gancho de Registro de Mensajes

```typescript
const isMessageReceivedEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "received";
const isMessageSentEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "sent";

const handler = async (event) => {
  if (isMessageReceivedEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Recibido de ${event.context.from}: ${event.context.content}`);
  } else if (isMessageSentEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Enviado a ${event.context.to}: ${event.context.content}`);
  }
};

export default handler;
```

### Ganchos de Resultado de Herramienta (API de Plugin)

Estos ganchos no son oyentes del flujo de eventos; permiten a los plugins ajustar sincrónicamente los resultados de las herramientas antes de que OpenClaw los persista.

-   **`tool_result_persist`**: transforma los resultados de las herramientas antes de que se escriban en la transcripción de la sesión. Debe ser síncrono; devuelve la carga útil del resultado de la herramienta actualizada o `undefined` para mantenerla como está. Consulta [Bucle del Agente](../concepts/agent-loop.md).

### Eventos de Gancho de Plugin

Ganchos del ciclo de vida de compactación expuestos a través del ejecutor de ganchos del plugin:

-   **`before_compaction`**: Se ejecuta antes de la compactación con metadatos de recuento/tokens
-   **`after_compaction`**: Se ejecuta después de la compactación con metadatos de resumen de compactación

### Eventos Futuros

Tipos de eventos planificados:

-   **`session:start`**: Cuando comienza una nueva sesión
-   **`session:end`**: Cuando termina una sesión
-   **`agent:error`**: Cuando un agente encuentra un error

## Creando Ganchos Personalizados

### 1\. Elige Ubicación

-   **Ganchos del espacio de trabajo** (`/hooks/`): Por agente, mayor precedencia
-   **Ganchos gestionados** (`~/.openclaw/hooks/`): Compartidos entre espacios de trabajo

### 2\. Crea la Estructura del Directorio

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3\. Crea HOOK.md

```
---
name: my-hook
description: "Hace algo útil"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# Mi Gancho Personalizado

Este gancho hace algo útil cuando emites `/new`.
```

### 4\. Crea handler.ts

```typescript
const handler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] ¡Ejecutándose!");
  // Tu lógica aquí
};

export default handler;
```

### 5\. Habilita y Prueba

```bash
# Verifica que el gancho se descubra
openclaw hooks list

# Habilítalo
openclaw hooks enable my-hook

# Reinicia tu proceso de gateway (reinicio de la aplicación de la barra de menú en macOS, o reinicia tu proceso de desarrollo)

# Activa el evento
# Envía /new a través de tu canal de mensajería
```

## Configuración

### Nuevo Formato de Configuración (Recomendado)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### Configuración por Gancho

Los ganchos pueden tener configuración personalizada:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### Directorios Extra

Carga ganchos desde directorios adicionales:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### Formato de Configuración Heredado (Aún Soportado)

El formato de configuración antiguo aún funciona para compatibilidad hacia atrás:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

Nota: `module` debe ser una ruta relativa al espacio de trabajo. Se rechazan las rutas absolutas y el recorrido fuera del espacio de trabajo. **Migración**: Usa el nuevo sistema basado en descubrimiento para nuevos ganchos. Los manejadores heredados se cargan después de los ganchos basados en directorio.

## Comandos CLI

### Listar Ganchos

```bash
# Lista todos los ganchos
openclaw hooks list

# Muestra solo ganchos elegibles
openclaw hooks list --eligible

# Salida detallada (muestra requisitos faltantes)
openclaw hooks list --verbose

# Salida JSON
openclaw hooks list --json
```

### Información del Gancho

```bash
# Muestra información detallada sobre un gancho
openclaw hooks info session-memory

# Salida JSON
openclaw hooks info session-memory --json
```

### Verificar Elegibilidad

```bash
# Muestra resumen de elegibilidad
openclaw hooks check

# Salida JSON
openclaw hooks check --json
```

### Habilitar/Deshabilitar

```bash
# Habilita un gancho
openclaw hooks enable session-memory

# Deshabilita un gancho
openclaw hooks disable command-logger
```

## Referencia de ganchos incluidos

### session-memory

Guarda el contexto de la sesión en la memoria cuando emites `/new`. **Eventos**: `command:new` **Requisitos**: `workspace.dir` debe estar configurado **Salida**: `/memory/YYYY-MM-DD-slug.md` (por defecto `~/.openclaw/workspace`) **Qué hace**:

1.  Usa la entrada de sesión pre-reset para localizar la transcripción correcta
2.  Extrae las últimas 15 líneas de conversación
3.  Usa LLM para generar un slug de nombre de archivo descriptivo
4.  Guarda los metadatos de la sesión en un archivo de memoria con fecha

**Ejemplo de salida**:

```bash
# Sesión: 2026-01-16 14:30:00 UTC

- **Clave de Sesión**: agent:main:main
- **ID de Sesión**: abc123def456
- **Fuente**: telegram
```

**Ejemplos de nombres de archivo**:

-   `2026-01-16-vendor-pitch.md`
-   `2026-01-16-api-design.md`
-   `2026-01-16-1430.md` (marca de tiempo de respaldo si falla la generación del slug)

**Habilitar**:

```bash
openclaw hooks enable session-memory
```

### bootstrap-extra-files

Inyecta archivos de arranque adicionales (por ejemplo, `AGENTS.md` / `TOOLS.md` locales del monorepositorio) durante `agent:bootstrap`. **Eventos**: `agent:bootstrap` **Requisitos**: `workspace.dir` debe estar configurado **Salida**: No se escriben archivos; el contexto de arranque se modifica solo en memoria. **Configuración**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "bootstrap-extra-files": {
          "enabled": true,
          "paths": ["packages/*/AGENTS.md", "packages/*/TOOLS.md"]
        }
      }
    }
  }
}
```

**Notas**:

-   Las rutas se resuelven relativas al espacio de trabajo.
-   Los archivos deben permanecer dentro del espacio de trabajo (verificado con realpath).
-   Solo se cargan los nombres base de arranque reconocidos.
-   Se preserva la lista de permitidos de subagentes (solo `AGENTS.md` y `TOOLS.md`).

**Habilitar**:

```bash
openclaw hooks enable bootstrap-extra-files
```

### command-logger

Registra todos los eventos de comando en un archivo de auditoría centralizado. **Eventos**: `command` **Requisitos**: Ninguno **Salida**: `~/.openclaw/logs/commands.log` **Qué hace**:

1.  Captura detalles del evento (acción del comando, marca de tiempo, clave de sesión, ID del remitente, fuente)
2.  Añade al archivo de registro en formato JSONL
3.  Se ejecuta silenciosamente en segundo plano

**Ejemplos de entradas de registro**:

```json
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Ver registros**:

```bash
# Ver comandos recientes
tail -n 20 ~/.openclaw/logs/commands.log

# Formatear con jq
cat ~/.openclaw/logs/commands.log | jq .

# Filtrar por acción
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Habilitar**:

```bash
openclaw hooks enable command-logger
```

### boot-md

Ejecuta `BOOT.md` cuando el gateway inicia (después de que los canales inician). Los ganchos internos deben estar habilitados para que esto se ejecute. **Eventos**: `gateway:startup` **Requisitos**: `workspace.dir` debe estar configurado **Qué hace**:

1.  Lee `BOOT.md` desde tu espacio de trabajo
2.  Ejecuta las instrucciones a través del ejecutor del agente
3.  Envía cualquier mensaje saliente solicitado a través de la herramienta de mensaje

**Habilitar**:

```bash
openclaw hooks enable boot-md
```

## Mejores Prácticas

### Mantén los Manejadores Rápidos

Los ganchos se ejecutan durante el procesamiento de comandos. Mantenlos ligeros:

```
// ✓ Bien - trabajo asíncrono, regresa inmediatamente
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Dispara y olvida
};

// ✗ Mal - bloquea el procesamiento de comandos
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### Maneja Errores con Elegancia

Siempre envuelve operaciones riesgosas:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Falló:", err instanceof Error ? err.message : String(err));
    // No lances - deja que otros manejadores se ejecuten
  }
};
```

### Filtra Eventos Temprano

Regresa temprano si el evento no es relevante:

```typescript
const handler: HookHandler = async (event) => {
  // Solo manejar comandos 'new'
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Tu lógica aquí
};
```

### Usa Claves de Evento Específicas

Especifica eventos exactos en los metadatos cuando sea posible:

```
metadata: { "openclaw": { "events": ["command:new"] } } # Específico
```

En lugar de:

```
metadata: { "openclaw": { "events": ["command"] } } # General - más sobrecarga
```

## Depuración

### Habilita el Registro de Ganchos

El gateway registra la carga de ganchos al inicio:

```bash
Registered hook: session-memory -> command:new
Registered hook: bootstrap-extra-files -> agent:bootstrap
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Verifica el Descubrimiento

Lista todos los ganchos descubiertos:

```bash
openclaw hooks list --verbose
```

### Verifica el Registro

En tu manejador, registra cuando se llama:

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Activado:", event.type, event.action);
  // Tu lógica
};
```

### Verifica la Elegibilidad

Verifica por qué un gancho no es elegible:

```bash
openclaw hooks info my-hook
```

Busca requisitos faltantes en la salida.

## Pruebas

### Registros del Gateway

Monitorea los registros del gateway para ver la ejecución del gancho:

```bash
# macOS
./scripts/clawlog.sh -f

# Otras plataformas
tail -f ~/.openclaw/gateway.log
```

### Prueba Ganchos Directamente

Prueba tus manejadores de forma aislada:

```typescript
import { test } from "vitest";
import myHandler from "./hooks/my-hook/handler.js";

test("mi manejador funciona", async () => {
  const event = {
    type: "command",
    action: "new",
    sessionKey: "test-session",
    timestamp: new Date(),
    messages: [],
    context: { foo: "bar" },
  };

  await myHandler(event);

  // Afirma efectos secundarios
});
```

## Arquitectura

### Componentes Principales

-   **`src/hooks/types.ts`**: Definiciones de tipos
-   **`src/hooks/workspace.ts`**: Escaneo y carga de directorios
-   **`src/hooks/frontmatter.ts`**: Análisis de metadatos de HOOK.md
-   **`src/hooks/config.ts`**: Comprobación de elegibilidad
-   **`src/hooks/hooks-status.ts`**: Informes de estado
-   **`src/hooks/loader.ts`**: Cargador de módulos dinámico
-   **`src/cli/hooks-cli.ts`**: Comandos CLI
-   **`src/gateway/server-startup.ts`**: Carga ganchos al inicio del gateway
-   **`src/auto-reply/reply/commands-core.ts`**: Activa eventos de comando

### Flujo de Descubrimiento

```
Inicio del Gateway
    ↓
Escanea directorios (espacio de trabajo → gestionados → incluidos)
    ↓
Analiza archivos HOOK.md
    ↓
Verifica elegibilidad (bins, env, config, os)
    ↓
Carga manejadores desde ganchos elegibles
    ↓
Registra manejadores para eventos
```

### Flujo de Eventos

```
Usuario envía /new
    ↓
Validación de comando
    ↓
Crea evento de gancho
    ↓
Activa gancho (todos los manejadores registrados)
    ↓
El procesamiento del comando continúa
    ↓
Reinicio de sesión
```

## Solución de Problemas

### Gancho No Descubierto

1.  Verifica la estructura del directorio:
    
    Copy
    
    ```bash
    ls -la ~/.openclaw/hooks/my-hook/
    # Debería mostrar: HOOK.md, handler.ts
    ```
    
2.  Verifica el formato de HOOK.md:
    
    Copy
    
    ```bash
    cat ~/.openclaw/hooks/my-hook/HOOK.md
    # Debería tener frontmatter YAML con nombre y metadatos
    ```
    
3.  Lista todos los ganchos descubiertos:
    
    Copy
    
    ```bash
    openclaw hooks list
    ```
    

### Gancho No Elegible

Verifica requisitos:

```bash
openclaw hooks info my-hook
```

Busca faltantes:

-   Binarios (verifica PATH)
-   Variables de entorno
-   Valores de configuración
-   Compatibilidad del SO

### Gancho No Se Ejecuta

1.  Verifica que el gancho esté habilitado:
    
    Copy
    
    ```bash
    openclaw hooks list
    # Debería mostrar ✓ al lado de los ganchos habilitados
    ```
    
2.  Reinicia tu proceso de gateway para que los ganchos se recarguen.
3.  Verifica los registros del gateway en busca de errores:
    
    Copy
    
    ```bash
    ./scripts/clawlog.sh | grep hook
    ```
    

### Errores del Manejador

Verifica errores de TypeScript/importación:

```bash
# Prueba la importación directamente
node -e "import('./path/to/handler.ts').then(console.log)"
```

## Guía de Migración

### Desde Configuración Heredada a Descubrimiento

**Antes**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**Después**:

1.  Crea directorio del gancho:
    
    Copy
    
    ```bash
    mkdir -p ~/.openclaw/hooks/my-hook
    mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
    ```
    
2.  Crea HOOK.md:
    
    Copy
    
    ```
    ---
    name: my-hook
    description: "Mi gancho personalizado"
    metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
    ---
    
    # Mi Gancho
    
    Hace algo útil.
    ```
    
3.  Actualiza la configuración:
    
    Copy
    
    ```json
    {
      "hooks": {
        "internal": {
          "enabled": true,