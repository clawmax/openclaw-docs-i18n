

  Fundamentos

  
# Arquitectura de Integración Pi

Este documento describe cómo OpenClaw se integra con [pi-coding-agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) y sus paquetes hermanos (`pi-ai`, `pi-agent-core`, `pi-tui`) para potenciar sus capacidades de agente de IA.

## Descripción General

OpenClaw utiliza el SDK de pi para incrustar un agente de codificación de IA en su arquitectura de puerta de enlace de mensajería. En lugar de generar pi como un subproceso o usar el modo RPC, OpenClaw importa e instancia directamente `AgentSession` de pi a través de `createAgentSession()`. Este enfoque embebido proporciona:

-   Control total sobre el ciclo de vida de la sesión y el manejo de eventos
-   Inyección de herramientas personalizadas (mensajería, sandbox, acciones específicas del canal)
-   Personalización del prompt del sistema por canal/contexto
-   Persistencia de sesión con soporte de ramificación/compactación
-   Rotación de perfiles de autenticación multi-cuenta con conmutación por error
-   Cambio de modelo independiente del proveedor

## Dependencias de Paquetes

```json
{
  "@mariozechner/pi-agent-core": "0.49.3",
  "@mariozechner/pi-ai": "0.49.3",
  "@mariozechner/pi-coding-agent": "0.49.3",
  "@mariozechner/pi-tui": "0.49.3"
}
```

| Paquete | Propósito |
| --- | --- |
| `pi-ai` | Abstracciones centrales de LLM: `Model`, `streamSimple`, tipos de mensajes, APIs de proveedores |
| `pi-agent-core` | Bucle del agente, ejecución de herramientas, tipos `AgentMessage` |
| `pi-coding-agent` | SDK de alto nivel: `createAgentSession`, `SessionManager`, `AuthStorage`, `ModelRegistry`, herramientas integradas |
| `pi-tui` | Componentes de interfaz de terminal (usados en el modo TUI local de OpenClaw) |

## Estructura de Archivos

```
src/agents/
├── pi-embedded-runner.ts          # Re-exporta desde pi-embedded-runner/
├── pi-embedded-runner/
│   ├── run.ts                     # Punto de entrada principal: runEmbeddedPiAgent()
│   ├── run/
│   │   ├── attempt.ts             # Lógica de intento único con configuración de sesión
│   │   ├── params.ts              # Tipo RunEmbeddedPiAgentParams
│   │   ├── payloads.ts            # Construye cargas útiles de respuesta desde resultados de ejecución
│   │   ├── images.ts              # Inyección de imágenes del modelo de visión
│   │   └── types.ts               # EmbeddedRunAttemptResult
│   ├── abort.ts                   # Detección de error de aborto
│   ├── cache-ttl.ts               # Seguimiento de TTL de caché para poda de contexto
│   ├── compact.ts                 # Lógica de compactación manual/automática
│   ├── extensions.ts              # Carga extensiones de pi para ejecuciones embebidas
│   ├── extra-params.ts            # Parámetros de flujo específicos del proveedor
│   ├── google.ts                  # Correcciones de orden de turnos de Google/Gemini
│   ├── history.ts                 # Limitación de historial (DM vs grupo)
│   ├── lanes.ts                   # Carriles de comandos de sesión/globales
│   ├── logger.ts                  # Registrador del subsistema
│   ├── model.ts                   # Resolución de modelo via ModelRegistry
│   ├── runs.ts                    # Seguimiento de ejecuciones activas, aborto, cola
│   ├── sandbox-info.ts            # Información del sandbox para el prompt del sistema
│   ├── session-manager-cache.ts   # Almacenamiento en caché de instancias de SessionManager
│   ├── session-manager-init.ts    # Inicialización de archivos de sesión
│   ├── system-prompt.ts           # Constructor del prompt del sistema
│   ├── tool-split.ts              # Divide herramientas en builtIn vs custom
│   ├── types.ts                   # EmbeddedPiAgentMeta, EmbeddedPiRunResult
│   └── utils.ts                   # Mapeo de ThinkLevel, descripción de error
├── pi-embedded-subscribe.ts       # Suscripción/despacho de eventos de sesión
├── pi-embedded-subscribe.types.ts # SubscribeEmbeddedPiSessionParams
├── pi-embedded-subscribe.handlers.ts # Fábrica de manejadores de eventos
├── pi-embedded-subscribe.handlers.lifecycle.ts
├── pi-embedded-subscribe.handlers.types.ts
├── pi-embedded-block-chunker.ts   # Fragmentación de respuestas en bloques por flujo
├── pi-embedded-messaging.ts       # Seguimiento de herramientas de mensajería enviadas
├── pi-embedded-helpers.ts         # Clasificación de errores, validación de turnos
├── pi-embedded-helpers/           # Módulos auxiliares
├── pi-embedded-utils.ts           # Utilidades de formato
├── pi-tools.ts                    # createOpenClawCodingTools()
├── pi-tools.abort.ts              # Envoltura de AbortSignal para herramientas
├── pi-tools.policy.ts             # Política de lista de permitidos/denegados de herramientas
├── pi-tools.read.ts               # Personalizaciones de la herramienta read
├── pi-tools.schema.ts             # Normalización de esquemas de herramientas
├── pi-tools.types.ts              # Alias de tipo AnyAgentTool
├── pi-tool-definition-adapter.ts  # Adaptador de AgentTool -> ToolDefinition
├── pi-settings.ts                 # Anulaciones de configuración
├── pi-extensions/                 # Extensiones personalizadas de pi
│   ├── compaction-safeguard.ts    # Extensión de salvaguarda de compactación
│   ├── compaction-safeguard-runtime.ts
│   ├── context-pruning.ts         # Extensión de poda de contexto basada en Cache-TTL
│   └── context-pruning/
├── model-auth.ts                  # Resolución de perfil de autenticación
├── auth-profiles.ts               # Almacén de perfiles, enfriamiento, conmutación por error
├── model-selection.ts             # Resolución de modelo por defecto
├── models-config.ts               # Generación de models.json
├── model-catalog.ts               # Caché del catálogo de modelos
├── context-window-guard.ts        # Validación de ventana de contexto
├── failover-error.ts              # Clase FailoverError
├── defaults.ts                    # DEFAULT_PROVIDER, DEFAULT_MODEL
├── system-prompt.ts               # buildAgentSystemPrompt()
├── system-prompt-params.ts        # Resolución de parámetros del prompt del sistema
├── system-prompt-report.ts        # Generación de informe de depuración
├── tool-summaries.ts              # Resúmenes de descripción de herramientas
├── tool-policy.ts                 # Resolución de política de herramientas
├── transcript-policy.ts           # Política de validación de transcripción
├── skills.ts                      # Instantánea/construcción de prompt de habilidades
├── skills/                        # Subsistema de habilidades
├── sandbox.ts                     # Resolución de contexto del sandbox
├── sandbox/                       # Subsistema de sandbox
├── channel-tools.ts               # Inyección de herramientas específicas del canal
├── openclaw-tools.ts              # Herramientas específicas de OpenClaw
├── bash-tools.ts                  # Herramientas exec/process
├── apply-patch.ts                 # Herramienta apply_patch (OpenAI)
├── tools/                         # Implementaciones de herramientas individuales
│   ├── browser-tool.ts
│   ├── canvas-tool.ts
│   ├── cron-tool.ts
│   ├── discord-actions*.ts
│   ├── gateway-tool.ts
│   ├── image-tool.ts
│   ├── message-tool.ts
│   ├── nodes-tool.ts
│   ├── session*.ts
│   ├── slack-actions.ts
│   ├── telegram-actions.ts
│   ├── web-*.ts
│   └── whatsapp-actions.ts
└── ...
```

## Flujo de Integración Central

### 1\. Ejecutar un Agente Embebido

El punto de entrada principal es `runEmbeddedPiAgent()` en `pi-embedded-runner/run.ts`:

```typescript
import { runEmbeddedPiAgent } from "./agents/pi-embedded-runner.js";

const result = await runEmbeddedPiAgent({
  sessionId: "user-123",
  sessionKey: "main:whatsapp:+1234567890",
  sessionFile: "/path/to/session.jsonl",
  workspaceDir: "/path/to/workspace",
  config: openclawConfig,
  prompt: "Hello, how are you?",
  provider: "anthropic",
  model: "claude-sonnet-4-20250514",
  timeoutMs: 120_000,
  runId: "run-abc",
  onBlockReply: async (payload) => {
    await sendToChannel(payload.text, payload.mediaUrls);
  },
});
```

### 2\. Creación de Sesión

Dentro de `runEmbeddedAttempt()` (llamado por `runEmbeddedPiAgent()`), se usa el SDK de pi:

```typescript
import {
  createAgentSession,
  DefaultResourceLoader,
  SessionManager,
  SettingsManager,
} from "@mariozechner/pi-coding-agent";

const resourceLoader = new DefaultResourceLoader({
  cwd: resolvedWorkspace,
  agentDir,
  settingsManager,
  additionalExtensionPaths,
});
await resourceLoader.reload();

const { session } = await createAgentSession({
  cwd: resolvedWorkspace,
  agentDir,
  authStorage: params.authStorage,
  modelRegistry: params.modelRegistry,
  model: params.model,
  thinkingLevel: mapThinkingLevel(params.thinkLevel),
  tools: builtInTools,
  customTools: allCustomTools,
  sessionManager,
  settingsManager,
  resourceLoader,
});

applySystemPromptOverrideToSession(session, systemPromptOverride);
```

### 3\. Suscripción a Eventos

`subscribeEmbeddedPiSession()` se suscribe a los eventos `AgentSession` de pi:

```typescript
const subscription = subscribeEmbeddedPiSession({
  session: activeSession,
  runId: params.runId,
  verboseLevel: params.verboseLevel,
  reasoningMode: params.reasoningLevel,
  toolResultFormat: params.toolResultFormat,
  onToolResult: params.onToolResult,
  onReasoningStream: params.onReasoningStream,
  onBlockReply: params.onBlockReply,
  onPartialReply: params.onPartialReply,
  onAgentEvent: params.onAgentEvent,
});
```

Los eventos manejados incluyen:

-   `message_start` / `message_end` / `message_update` (texto/pensamiento en flujo)
-   `tool_execution_start` / `tool_execution_update` / `tool_execution_end`
-   `turn_start` / `turn_end`
-   `agent_start` / `agent_end`
-   `auto_compaction_start` / `auto_compaction_end`

### 4\. Solicitud (Prompting)

Después de la configuración, se le envía un prompt a la sesión:

```typescript
await session.prompt(effectivePrompt, { images: imageResult.images });
```

El SDK maneja el bucle completo del agente: enviar al LLM, ejecutar llamadas a herramientas, transmitir respuestas. La inyección de imágenes es local al prompt: OpenClaw carga referencias de imágenes del prompt actual y las pasa via `images` solo para ese turno. No re-escanea turnos de historial más antiguos para re-inyectar cargas útiles de imágenes.

## Arquitectura de Herramientas

### Canalización de Herramientas

1.  **Herramientas Base**: `codingTools` de pi (read, bash, edit, write)
2.  **Reemplazos Personalizados**: OpenClaw reemplaza bash con `exec`/`process`, personaliza read/edit/write para sandbox
3.  **Herramientas OpenClaw**: mensajería, navegador, canvas, sesiones, cron, puerta de enlace, etc.
4.  **Herramientas de Canal**: Herramientas de acción específicas de Discord/Telegram/Slack/WhatsApp
5.  **Filtrado por Política**: Herramientas filtradas por perfil, proveedor, agente, grupo, políticas de sandbox
6.  **Normalización de Esquema**: Esquemas limpiados por peculiaridades de Gemini/OpenAI
7.  **Envoltura de AbortSignal**: Herramientas envueltas para respetar señales de aborto

### Adaptador de Definición de Herramienta

`AgentTool` de pi-agent-core tiene una firma `execute` diferente a `ToolDefinition` de pi-coding-agent. El adaptador en `pi-tool-definition-adapter.ts` los conecta:

```bash
export function toToolDefinitions(tools: AnyAgentTool[]): ToolDefinition[] {
  return tools.map((tool) => ({
    name: tool.name,
    label: tool.label ?? name,
    description: tool.description ?? "",
    parameters: tool.parameters,
    execute: async (toolCallId, params, onUpdate, _ctx, signal) => {
      // La firma de pi-coding-agent difiere de pi-agent-core
      return await tool.execute(toolCallId, params, signal, onUpdate);
    },
  }));
}
```

### Estrategia de División de Herramientas

`splitSdkTools()` pasa todas las herramientas via `customTools`:

```bash
export function splitSdkTools(options: { tools: AnyAgentTool[]; sandboxEnabled: boolean }) {
  return {
    builtInTools: [], // Vacío. Sobreescribimos todo
    customTools: toToolDefinitions(options.tools),
  };
}
```

Esto asegura que el filtrado por política de OpenClaw, la integración del sandbox y el conjunto de herramientas extendido permanezcan consistentes entre proveedores.

## Construcción del Prompt del Sistema

El prompt del sistema se construye en `buildAgentSystemPrompt()` (`system-prompt.ts`). Ensambla un prompt completo con secciones que incluyen Herramientas, Estilo de Llamada a Herramientas, Barreras de seguridad, Referencia de CLI de OpenClaw, Habilidades, Documentación, Espacio de Trabajo, Sandbox, Mensajería, Etiquetas de Respuesta, Voz, Respuestas Silenciosas, Latidos, Metadatos de tiempo de ejecución, más Memoria y Reacciones cuando están habilitadas, y archivos de contexto y contenido extra del prompt del sistema opcionales. Las secciones se recortan para el modo de prompt mínimo usado por subagentes. El prompt se aplica después de la creación de la sesión via `applySystemPromptOverrideToSession()`:

```typescript
const systemPromptOverride = createSystemPromptOverride(appendPrompt);
applySystemPromptOverrideToSession(session, systemPromptOverride);
```

## Gestión de Sesiones

### Archivos de Sesión

Las sesiones son archivos JSONL con estructura de árbol (vinculación id/parentId). El `SessionManager` de pi maneja la persistencia:

```typescript
const sessionManager = SessionManager.open(params.sessionFile);
```

OpenClaw envuelve esto con `guardSessionManager()` para seguridad en los resultados de las herramientas.

### Almacenamiento en Caché de Sesiones

`session-manager-cache.ts` almacena en caché instancias de SessionManager para evitar análisis repetidos de archivos:

```typescript
await prewarmSessionFile(params.sessionFile);
sessionManager = SessionManager.open(params.sessionFile);
trackSessionManagerAccess(params.sessionFile);
```

### Limitación de Historial

`limitHistoryTurns()` recorta el historial de conversación basado en el tipo de canal (DM vs grupo).

### Compactación

La auto-compactación se activa en desbordamiento de contexto. `compactEmbeddedPiSessionDirect()` maneja la compactación manual:

```typescript
const compactResult = await compactEmbeddedPiSessionDirect({
  sessionId, sessionFile, provider, model, ...
});
```

## Autenticación y Resolución de Modelos

### Perfiles de Autenticación

OpenClaw mantiene un almacén de perfiles de autenticación con múltiples claves API por proveedor:

```typescript
const authStore = ensureAuthProfileStore(agentDir, { allowKeychainPrompt: false });
const profileOrder = resolveAuthProfileOrder({ cfg, store: authStore, provider, preferredProfile });
```

Los perfiles rotan en fallos con seguimiento de enfriamiento:

```typescript
await markAuthProfileFailure({ store, profileId, reason, cfg, agentDir });
const rotated = await advanceAuthProfile();
```

### Resolución de Modelo

```typescript
import { resolveModel } from "./pi-embedded-runner/model.js";

const { model, error, authStorage, modelRegistry } = resolveModel(
  provider,
  modelId,
  agentDir,
  config,
);

// Usa ModelRegistry y AuthStorage de pi
authStorage.setRuntimeApiKey(model.provider, apiKeyInfo.apiKey);
```

### Conmutación por Error (Failover)

`FailoverError` activa la reserva de modelo cuando está configurada:

```
if (fallbackConfigured && isFailoverErrorMessage(errorText)) {
  throw new FailoverError(errorText, {
    reason: promptFailoverReason ?? "unknown",
    provider,
    model: modelId,
    profileId,
    status: resolveFailoverStatus(promptFailoverReason),
  });
}
```

## Extensiones de Pi

OpenClaw carga extensiones personalizadas de pi para comportamientos especializados:

### Salvaguarda de Compactación

`src/agents/pi-extensions/compaction-safeguard.ts` agrega barreras de protección a la compactación, incluyendo presupuesto de tokens adaptativo más resúmenes de fallos de herramientas y operaciones de archivos:

```
if (resolveCompactionMode(params.cfg) === "safeguard") {
  setCompactionSafeguardRuntime(params.sessionManager, { maxHistoryShare });
  paths.push(resolvePiExtensionPath("compaction-safeguard"));
}
```

### Poda de Contexto

`src/agents/pi-extensions/context-pruning.ts` implementa poda de contexto basada en cache-TTL:

```
if (cfg?.agents?.defaults?.contextPruning?.mode === "cache-ttl") {
  setContextPruningRuntime(params.sessionManager, {
    settings,
    contextWindowTokens,
    isToolPrunable,
    lastCacheTouchAt,
  });
  paths.push(resolvePiExtensionPath("context-pruning"));
}
```

## Transmisión en Flujo y Respuestas en Bloques

### Fragmentación en Bloques

`EmbeddedBlockChunker` gestiona la transmisión de texto en bloques de respuesta discretos:

```typescript
const blockChunker = blockChunking ? new EmbeddedBlockChunker(blockChunking) : null;
```

### Eliminación de Etiquetas Thinking/Final

La salida en flujo se procesa para eliminar bloques ``/`` y extraer contenido ``:

```typescript
const stripBlockTags = (text: string, state: { thinking: boolean; final: boolean }) => {
  // Elimina contenido <think>...</think>
  // Si enforceFinalTag, solo devuelve contenido <final>...</final>
};
```

### Directivas de Respuesta

Las directivas de respuesta como `[[media:url]]`, `[[voice]]`, `[[reply:id]]` se analizan y extraen:

```typescript
const { text: cleanedText, mediaUrls, audioAsVoice, replyToId } = consumeReplyDirectives(chunk);
```

## Manejo de Errores

### Clasificación de Errores

`pi-embedded-helpers.ts` clasifica errores para un manejo apropiado:

```
isContextOverflowError(errorText)     // Contexto demasiado grande
isCompactionFailureError(errorText)   // Fallo de compactación
isAuthAssistantError(lastAssistant)   // Fallo de autenticación
isRateLimitAssistantError(...)        // Límite de tasa alcanzado
isFailoverAssistantError(...)         // Debería conmutar por error
classifyFailoverReason(errorText)     // "auth" | "rate_limit" | "quota" | "timeout" | ...
```

### Reserva de Nivel de Pensamiento (Thinking Level)

Si un nivel de pensamiento no es compatible, se recurre a una reserva:

```typescript
const fallbackThinking = pickFallbackThinkingLevel({
  message: errorText,
  attempted: attemptedThinking,
});
if (fallbackThinking) {
  thinkLevel = fallbackThinking;
  continue;
}
```

## Integración del Sandbox

Cuando el modo sandbox está habilitado, las herramientas y rutas están restringidas:

```typescript
const sandbox = await resolveSandboxContext({
  config: params.config,
  sessionKey: sandboxSessionKey,
  workspaceDir: resolvedWorkspace,
});

if (sandboxRoot) {
  // Usa herramientas read/edit/write en sandbox
  // Exec se ejecuta en contenedor
  // Navegador usa URL de puente
}
```

## Manejo Específico del Proveedor

### Anthropic

-   Eliminación de cadenas mágicas de rechazo
-   Validación de turnos para roles consecutivos
-   Compatibilidad con parámetros Claude Code

### Google/Gemini

-   Correcciones de orden de turnos (`applyGoogleTurnOrderingFix`)
-   Saneamiento de esquemas de herramientas (`sanitizeToolsForGoogle`)
-   Saneamiento del historial de sesión (`sanitizeSessionHistory`)

### OpenAI

-   Herramienta `apply_patch` para modelos Codex
-   Manejo de degradación del nivel de pensamiento

## Integración TUI

OpenClaw también tiene un modo TUI local que usa componentes pi-tui directamente:

```bash
// src/tui/tui.ts
import { ... } from "@mariozechner/pi-tui";
```

Esto proporciona la experiencia de terminal interactiva similar al modo nativo de pi.

## Diferencias Clave con la CLI de Pi

| Aspecto | CLI de Pi | OpenClaw Embebido |
| --- | --- | --- |
| Invocación | Comando `pi` / RPC | SDK via `createAgentSession()` |
| Herramientas | Herramientas de codificación por defecto | Conjunto de herramientas personalizado de OpenClaw |
| Prompt del sistema | AGENTS.md + prompts | Dinámico por canal/contexto |
| Almacenamiento de sesión | `~/.pi/agent/sessions/` | `~/.openclaw/agents//sessions/` (o `$OPENCLAW_STATE_DIR/agents//sessions/`) |
| Autenticación | Credencial única | Multi-perfil con rotación |
| Extensiones | Cargadas desde disco | Programáticas + rutas de disco |
| Manejo de eventos | Renderizado TUI | Basado en callbacks (onBlockReply, etc.) |

## Consideraciones Futuras

Áreas para posible re-trabajo:

1.  **Alineación de firma de herramientas**: Actualmente adaptando entre firmas de pi-agent-core y pi-coding-agent
2.  **Envoltura del gestor de sesiones**: `guardSessionManager` agrega seguridad pero aumenta la complejidad
3.  **Carga de extensiones**: Podría usar `ResourceLoader` de pi más directamente
4.  **Complejidad del manejador de flujo**: `subscribeEmbeddedPiSession` ha crecido mucho
5.  **Peculiaridades del proveedor**: Muchas rutas de código específicas del proveedor que pi podría manejar potencialmente

## Pruebas

La cobertura de integración de Pi abarca estas suites:

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-auth-json.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-embedded-helpers*.test.ts`
-   `src/agents/pi-embedded-runner*.test.ts`
-   `src/agents/pi-embedded-runner/**/*.test.ts`
-   `src/agents/pi-embedded-subscribe*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-tool-definition-adapter*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-extensions/**/*.test.ts`

En vivo/opt-in:

-   `src/agents/pi-embedded-runner-extraparams.live.test.ts` (habilitar `OPENCLAW_LIVE_TEST=1`)

Para los comandos de ejecución actuales, consulta [Flujo de Trabajo de Desarrollo Pi](./pi-dev.md).

[Arquitectura de la Puerta de Enlace](./concepts/architecture.md)