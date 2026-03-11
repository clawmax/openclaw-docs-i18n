

  Experimentos

  
# Agentes Vinculados a Hilos ACP

## Descripción general

Este plan define cómo OpenClaw debe soportar agentes de codificación ACP en canales con capacidad de hilos (Discord primero) con ciclo de vida y recuperación de nivel de producción. Documento relacionado:

-   [Plan de Refactorización de Streaming de Tiempo de Ejecución Unificado](./acp-unified-streaming-refactor.md)

Experiencia de usuario objetivo:

-   un usuario genera o enfoca una sesión ACP en un hilo
-   los mensajes del usuario en ese hilo se enrutan a la sesión ACP vinculada
-   la salida del agente se transmite de vuelta a la misma persona del hilo
-   la sesión puede ser persistente o de un solo uso con controles de limpieza explícitos

## Resumen de decisiones

La recomendación a largo plazo es una arquitectura híbrida:

-   El núcleo de OpenClaw posee las preocupaciones del plano de control de ACP
    -   identidad y metadatos de la sesión
    -   vinculación de hilos y decisiones de enrutamiento
    -   invariantes de entrega y supresión de duplicados
    -   semántica de limpieza del ciclo de vida y recuperación
-   El backend del tiempo de ejecución de ACP es conectable
    -   el primer backend es un servicio de plugin respaldado por acpx
    -   el tiempo de ejecución maneja el transporte ACP, encolamiento, cancelación, reconexión

OpenClaw no debe reimplementar los internos del transporte ACP en el núcleo. OpenClaw no debe depender de una ruta de intercepción puramente de plugin para el enrutamiento.

## Arquitectura objetivo (santo grial)

Tratar ACP como un plano de control de primera clase en OpenClaw, con adaptadores de tiempo de ejecución conectables. Invariantes no negociables:

-   cada vinculación de hilo ACP referencia un registro de sesión ACP válido
-   cada sesión ACP tiene un estado de ciclo de vida explícito (`creating`, `idle`, `running`, `cancelling`, `closed`, `error`)
-   cada ejecución ACP tiene un estado de ejecución explícito (`queued`, `running`, `completed`, `failed`, `cancelled`)
-   generar, vincular y encolar inicialmente son atómicos
-   los reintentos de comandos son idempotentes (sin ejecuciones duplicadas ni salidas de Discord duplicadas)
-   la salida del canal de hilo vinculado es una proyección de eventos de ejecución ACP, nunca efectos secundarios ad-hoc

Modelo de propiedad a largo plazo:

-   `AcpSessionManager` es el único escritor y orquestador de ACP
-   el gestor vive en el proceso de puerta de enlace primero; puede moverse a un sidecar dedicado más tarde detrás de la misma interfaz
-   por clave de sesión ACP, el gestor posee un actor en memoria (ejecución de comandos serializada)
-   los adaptadores (`acpx`, futuros backends) son solo implementaciones de transporte/tiempo de ejecución

Modelo de persistencia a largo plazo:

-   mover el estado del plano de control de ACP a un almacén SQLite dedicado (modo WAL) bajo el directorio de estado de OpenClaw
-   mantener `SessionEntry.acp` como proyección de compatibilidad durante la migración, no como fuente de verdad
-   almacenar eventos ACP solo de adición para soportar reproducción, recuperación de fallos y entrega determinista

### Estrategia de entrega (puente al santo grial)

-   puente a corto plazo
    -   mantener la mecánica actual de vinculación de hilos y la superficie de configuración ACP existente
    -   corregir errores de brecha de metadatos y enrutar turnos ACP a través de una única rama ACP central
    -   añadir claves de idempotencia y comprobaciones de enrutamiento de fallo cerrado inmediatamente
-   migración a largo plazo
    -   mover la fuente de verdad de ACP a la base de datos del plano de control + actores
    -   hacer que la entrega de hilos vinculados sea puramente basada en proyección de eventos
    -   eliminar el comportamiento de respaldo heredado que depende de metadatos oportunistas de entrada de sesión

## Por qué no solo plugin puro

Los ganchos de plugin actuales no son suficientes para el enrutamiento de sesiones ACP de extremo a extremo sin cambios en el núcleo.

-   el enrutamiento entrante desde la vinculación de hilos resuelve a una clave de sesión en el despacho central primero
-   los ganchos de mensaje son de tipo "dispara y olvida" y no pueden interrumpir la ruta principal de respuesta
-   los comandos de plugin son buenos para operaciones de control pero no para reemplazar el flujo de despacho por turno central

Resultado:

-   el tiempo de ejecución de ACP puede ser pluginizado
-   la rama de enrutamiento ACP debe existir en el núcleo

## Fundación existente para reutilizar

Ya implementado y debe permanecer canónico:

-   el objetivo de vinculación de hilos soporta `subagent` y `acp`
-   la anulación de enrutamiento de hilos entrante resuelve por vinculación antes del despacho normal
-   identidad de hilo saliente a través de webhook en la entrega de respuesta
-   flujo `/focus` y `/unfocus` con compatibilidad de objetivo ACP
-   almacén de vinculación persistente con restauración al inicio
-   ciclo de vida de desvinculación en archivar, eliminar, desenfocar, reiniciar y borrar

Este plan extiende esa fundación en lugar de reemplazarla.

## Arquitectura

### Modelo de límites

Núcleo (debe estar en el núcleo de OpenClaw):

-   rama de despacho de modo de sesión ACP en la tubería de respuesta
-   arbitraje de entrega para evitar duplicación de padre más hilo
-   persistencia del plano de control de ACP (con proyección de compatibilidad `SessionEntry.acp` durante la migración)
-   semántica de desvinculación del ciclo de vida y desacoplamiento del tiempo de ejecución vinculada a reinicio/borrado de sesión

Backend de plugin (implementación acpx):

-   supervisión de trabajadores del tiempo de ejecución ACP
-   invocación de proceso acpx y análisis de eventos
-   manejadores de comandos ACP (`/acp ...`) y UX del operador
-   valores predeterminados de configuración y diagnósticos específicos del backend

### Modelo de propiedad del tiempo de ejecución

-   un proceso de puerta de enlace posee el estado de orquestación ACP
-   la ejecución de ACP se ejecuta en procesos hijos supervisados a través del backend acpx
-   la estrategia de proceso es de larga duración por clave de sesión ACP activa, no por mensaje

Esto evita el costo de inicio en cada prompt y mantiene la semántica de cancelación y reconexión confiable.

### Contrato de tiempo de ejecución central

Añadir un contrato de tiempo de ejecución ACP central para que el código de enrutamiento no dependa de detalles de CLI y pueda cambiar backends sin cambiar la lógica de despacho:

```bash
export type AcpRuntimePromptMode = "prompt" | "steer";

export type AcpRuntimeHandle = {
  sessionKey: string;
  backend: string;
  runtimeSessionName: string;
};

export type AcpRuntimeEvent =
  | { type: "text_delta"; stream: "output" | "thought"; text: string }
  | { type: "tool_call"; name: string; argumentsText: string }
  | { type: "done"; usage?: Record<string, number> }
  | { type: "error"; code: string; message: string; retryable?: boolean };

export interface AcpRuntime {
  ensureSession(input: {
    sessionKey: string;
    agent: string;
    mode: "persistent" | "oneshot";
    cwd?: string;
    env?: Record<string, string>;
    idempotencyKey: string;
  }): Promise<AcpRuntimeHandle>;

  submit(input: {
    handle: AcpRuntimeHandle;
    text: string;
    mode: AcpRuntimePromptMode;
    idempotencyKey: string;
  }): Promise<{ runtimeRunId: string }>;

  stream(input: {
    handle: AcpRuntimeHandle;
    runtimeRunId: string;
    onEvent: (event: AcpRuntimeEvent) => Promise<void> | void;
    signal?: AbortSignal;
  }): Promise<void>;

  cancel(input: {
    handle: AcpRuntimeHandle;
    runtimeRunId?: string;
    reason?: string;
    idempotencyKey: string;
  }): Promise<void>;

  close(input: { handle: AcpRuntimeHandle; reason: string; idempotencyKey: string }): Promise<void>;

  health?(): Promise<{ ok: boolean; details?: string }>;
}
```

Detalle de implementación:

-   primer backend: `AcpxRuntime` enviado como un servicio de plugin
-   el núcleo resuelve el tiempo de ejecución a través del registro y falla con un error explícito del operador cuando no hay backend de tiempo de ejecución ACP disponible

### Modelo de datos del plano de control y persistencia

La fuente de verdad a largo plazo es una base de datos SQLite ACP dedicada (modo WAL), para actualizaciones transaccionales y recuperación segura ante fallos:

-   `acp_sessions`
    -   `session_key` (pk), `backend`, `agent`, `mode`, `cwd`, `state`, `created_at`, `updated_at`, `last_error`
-   `acp_runs`
    -   `run_id` (pk), `session_key` (fk), `state`, `requester_message_id`, `idempotency_key`, `started_at`, `ended_at`, `error_code`, `error_message`
-   `acp_bindings`
    -   `binding_key` (pk), `thread_id`, `channel_id`, `account_id`, `session_key` (fk), `expires_at`, `bound_at`
-   `acp_events`
    -   `event_id` (pk), `run_id` (fk), `seq`, `kind`, `payload_json`, `created_at`
-   `acp_delivery_checkpoint`
    -   `run_id` (pk/fk), `last_event_seq`, `last_discord_message_id`, `updated_at`
-   `acp_idempotency`
    -   `scope`, `idempotency_key`, `result_json`, `created_at`, único `(scope, idempotency_key)`

```bash
export type AcpSessionMeta = {
  backend: string;
  agent: string;
  runtimeSessionName: string;
  mode: "persistent" | "oneshot";
  cwd?: string;
  state: "idle" | "running" | "error";
  lastActivityAt: number;
  lastError?: string;
};
```

Reglas de almacenamiento:

-   mantener `SessionEntry.acp` como una proyección de compatibilidad durante la migración
-   los ids de proceso y sockets permanecen solo en memoria
-   el ciclo de vida duradero y el estado de ejecución viven en la base de datos ACP, no en JSON de sesión genérico
-   si el propietario del tiempo de ejecución muere, la puerta de enlace rehidrata desde la base de datos ACP y reanuda desde los puntos de control

### Enrutamiento y entrega

Entrante:

-   mantener la búsqueda actual de vinculación de hilos como primer paso de enrutamiento
-   si el objetivo vinculado es una sesión ACP, enrutar a la rama de tiempo de ejecución ACP en lugar de `getReplyFromConfig`
-   el comando explícito `/acp steer` usa `mode: "steer"`

Saliente:

-   el flujo de eventos ACP se normaliza a fragmentos de respuesta de OpenClaw
-   el objetivo de entrega se resuelve a través de la ruta de destino vinculada existente
-   cuando un hilo vinculado está activo para ese turno de sesión, se suprime la finalización del canal padre

Política de transmisión:

-   transmitir salida parcial con ventana de coalescencia
-   intervalo mínimo configurable y bytes máximos por fragmento para mantenerse bajo los límites de tasa de Discord
-   el mensaje final siempre se emite al completarse o fallar

### Máquinas de estado y límites de transacción

Máquina de estado de sesión:

-   `creating -> idle -> running -> idle`
-   `running -> cancelling -> idle | error`
-   `idle -> closed`
-   `error -> idle | closed`

Máquina de estado de ejecución:

-   `queued -> running -> completed`
-   `running -> failed | cancelled`
-   `queued -> cancelled`

Límites de transacción requeridos:

-   transacción de generación
    -   crear fila de sesión ACP
    -   crear/actualizar fila de vinculación de hilo ACP
    -   encolar fila de ejecución inicial
-   transacción de cierre
    -   marcar sesión como cerrada
    -   eliminar/caducar filas de vinculación
    -   escribir evento de cierre final
-   transacción de cancelación
    -   marcar ejecución objetivo como cancelando/cancelada con clave de idempotencia

No se permite éxito parcial a través de estos límites.

### Modelo de actor por sesión

`AcpSessionManager` ejecuta un actor por clave de sesión ACP:

-   el buzón del actor serializa los efectos secundarios `submit`, `cancel`, `close` y `stream`
-   el actor posee la hidratación del manejador de tiempo de ejecución y el ciclo de vida del proceso del adaptador de tiempo de ejecución para esa sesión
-   el actor escribe eventos de ejecución en orden (`seq`) antes de cualquier entrega a Discord
-   el actor actualiza los puntos de control de entrega después del envío saliente exitoso

Esto elimina las carreras entre turnos y evita salidas de hilo duplicadas o fuera de orden.

### Idempotencia y proyección de entrega

Todas las acciones ACP externas deben llevar claves de idempotencia:

-   clave de idempotencia de generación
-   clave de idempotencia de prompt/steer
-   clave de idempotencia de cancelación
-   clave de idempotencia de cierre

Reglas de entrega:

-   los mensajes de Discord se derivan de `acp_events` más `acp_delivery_checkpoint`
-   los reintentos se reanudan desde el punto de control sin reenviar fragmentos ya entregados
-   la emisión de respuesta final es exactamente una vez por ejecución desde la lógica de proyección

### Recuperación y autocuración

Al inicio de la puerta de enlace:

-   cargar sesiones ACP no terminales (`creating`, `idle`, `running`, `cancelling`, `error`)
-   recrear actores de forma perezosa en el primer evento entrante o de forma ansiosa bajo un límite configurado
-   reconciliar cualquier ejecución `running` que falte latidos y marcarla como `failed` o recuperar a través del adaptador

En mensaje de hilo de Discord entrante:

-   si existe una vinculación pero falta la sesión ACP, fallar cerrado con un mensaje explícito de vinculación obsoleta
-   opcionalmente desvincular automáticamente la vinculación obsoleta después de una validación segura para el operador
-   nunca enrutar silenciosamente vinculaciones ACP obsoletas a la ruta LLM normal

### Ciclo de vida y seguridad

Operaciones soportadas:

-   cancelar ejecución actual: `/acp cancel`
-   desvincular hilo: `/unfocus`
-   cerrar sesión ACP: `/acp close`
-   cierre automático de sesiones inactivas por TTL efectivo

Política de TTL:

-   TTL efectivo es el mínimo de
    -   TTL global/de sesión
    -   TTL de vinculación de hilo de Discord
    -   TTL del propietario del tiempo de ejecución ACP

Controles de seguridad:

-   lista de permitidos de agentes ACP por nombre
-   restringir raíces de espacio de trabajo para sesiones ACP
-   lista de permitidos de variables de entorno para pasar
-   máximo de sesiones ACP concurrentes por cuenta y globalmente
-   retroceso de reinicio limitado para fallos del tiempo de ejecución

## Superficie de configuración

Claves centrales:

-   `acp.enabled`
-   `acp.dispatch.enabled` (interruptor de eliminación de enrutamiento ACP independiente)
-   `acp.backend` (predeterminado `acpx`)
-   `acp.defaultAgent`
-   `acp.allowedAgents[]`
-   `acp.maxConcurrentSessions`
-   `acp.stream.coalesceIdleMs`
-   `acp.stream.maxChunkChars`
-   `acp.runtime.ttlMinutes`
-   `acp.controlPlane.store` (`sqlite` predeterminado)
-   `acp.controlPlane.storePath`
-   `acp.controlPlane.recovery.eagerActors`
-   `acp.controlPlane.recovery.reconcileRunningAfterMs`
-   `acp.controlPlane.checkpoint.flushEveryEvents`
-   `acp.controlPlane.checkpoint.flushEveryMs`
-   `acp.idempotency.ttlHours`
-   `channels.discord.threadBindings.spawnAcpSessions`

Claves de plugin/backend (sección de plugin acpx):

-   anulaciones de comando/ruta del backend
-   lista de permitidos de variables de entorno del backend
-   preajustes por agente del backend
-   tiempos de espera de inicio/detención del backend
-   máximo de ejecuciones en curso por sesión del backend

## Especificación de implementación

### Módulos del plano de control (nuevos)

Añadir módulos del plano de control ACP dedicados en el núcleo:

-   `src/acp/control-plane/manager.ts`
    -   posee actores ACP, transiciones de ciclo de vida, serialización de comandos
-   `src/acp/control-plane/store.ts`
    -   gestión de esquema SQLite, transacciones, ayudantes de consulta
-   `src/acp/control-plane/events.ts`
    -   definiciones de eventos ACP tipadas y serialización
-   `src/acp/control-plane/checkpoint.ts`
    -   puntos de control de entrega duraderos y cursores de reproducción
-   `src/acp/control-plane/idempotency.ts`
    -   reserva de claves de idempotencia y reproducción de respuesta
-   `src/acp/control-plane/recovery.ts`
    -   reconciliación al inicio y plan de rehidratación de actores

Módulos puente de compatibilidad:

-   `src/acp/runtime/session-meta.ts`
    -   permanece temporalmente para proyección en `SessionEntry.acp`
    -   debe dejar de ser fuente de verdad después de la migración

### Invariantes requeridas (debe hacer cumplir en código)

-   la creación de sesión ACP y la vinculación de hilo son atómicas (transacción única)
-   hay como máximo una ejecución activa por actor de sesión ACP a la vez
-   el `seq` del evento es estrictamente creciente por ejecución
-   el punto de control de entrega nunca avanza más allá del último evento confirmado
-   la reproducción de idempotencia devuelve la carga útil de éxito anterior para claves de comando duplicadas
-   los metadatos ACP obsoletos/faltantes no pueden enrutarse a la ruta de respuesta no ACP normal

### Puntos de contacto centrales

Archivos centrales a cambiar:

-   `src/auto-reply/reply/dispatch-from-config.ts`
    -   la rama ACP llama a `AcpSessionManager.submit` y entrega por proyección de eventos
    -   eliminar la ruta de respaldo ACP directa que omite las invariantes del plano de control
-   `src/auto-reply/reply/inbound-context.ts` (o el límite de contexto normalizado más cercano)
    -   exponer claves de enrutamiento normalizadas y semillas de idempotencia para el plano de control ACP
-   `src/config/sessions/types.ts`
    -   mantener `SessionEntry.acp` como campo de compatibilidad solo de proyección
-   `src/gateway/server-methods/sessions.ts`
    -   reiniciar/eliminar/archivar debe llamar a la ruta de transacción de cierre/desvinculación del gestor ACP
-   `src/infra/outbound/bound-delivery-router.ts`
    -   hacer cumplir el comportamiento de destino de fallo cerrado para turnos de sesión ACP vinculados
-   `src/discord/monitor/thread-bindings.ts`
    -   añadir ayudantes de validación de vinculación obsoleta ACP conectados a búsquedas del plano de control
-   `src/auto-reply/reply/commands-acp.ts`
    -   enrutar generar/cancelar/cerrar/steer a través de las APIs del gestor ACP
-   `src/agents/acp-spawn.ts`
    -   dejar de escribir metadatos ad-hoc; llamar a la transacción de generación del gestor ACP
-   `src/plugin-sdk/**` y puente de tiempo de ejecución de plugin
    -   exponer el registro de backend ACP y la semántica de salud de forma limpia

Archivos centrales explícitamente no reemplazados:

-   `src/discord/monitor/message-handler.preflight.ts`
    -   mantener el comportamiento de anulación de vinculación de hilos como el resolvedor de clave de sesión canónico

### API de registro de tiempo de ejecución ACP

Añadir un módulo de registro central:

-   `src/acp/runtime/registry.ts`

API requerida:

```bash
export type AcpRuntimeBackend = {
  id: string;
  runtime: AcpRuntime;
  healthy?: () => boolean;
};

export function registerAcpRuntimeBackend(backend: AcpRuntimeBackend): void;
export function unregisterAcpRuntimeBackend(id: string): void;
export function getAcpRuntimeBackend(id?: string): AcpRuntimeBackend | null;
export function requireAcpRuntimeBackend(id?: string): AcpRuntimeBackend;
```

Comportamiento:

-   `requireAcpRuntimeBackend` lanza un error tipado de backend ACP faltante cuando no está disponible
-   el servicio de plugin registra el backend en `start` y lo anula en `stop`
-   las búsquedas de tiempo de ejecución son de solo lectura y locales al proceso

### Contrato de plugin de tiempo de ejecución acpx (detalle de implementación)

Para el primer backend de producción (`extensions/acpx`), OpenClaw y acpx están conectados con un contrato de comando estricto:

-   id de backend: `acpx`
-   id de servicio de plugin: `acpx-runtime`
-   codificación del manejador de tiempo de ejecución: `runtimeSessionName = acpx:v1:<base64url(json)>`
-   campos de carga útil codificados:
    -   `name` (sesión nombrada de acpx; usa `sessionKey` de OpenClaw)
    -   `agent` (comando de agente de acpx)
    -   `cwd` (raíz del espacio de trabajo de la sesión)
    -   `mode` (`persistent | oneshot`)

Mapeo de comandos:

-   asegurar sesión:
    -   `acpx --format json --json-strict --cwd   sessions ensure --name `
-   turno de prompt:
    -   `acpx --format json --json-strict --cwd   prompt --session  --file -`
-   cancelar:
    -   `acpx --format json --json-strict --cwd   cancel --session `
-   cerrar:
    -   `acpx --format json --json-strict --cwd   sessions close `

Transmisión:

-   OpenClaw consume eventos ndjson de `acpx --format json --json-strict`
-   `text` => `text_delta/output`
-   `thought` => `text_delta/thought`
-   `tool_call` => `tool_call`
-   `done` => `done`
-   `error` => `error`

### Parche de esquema de sesión

Parchear `SessionEntry` en `src/config/sessions/types.ts`:

```typescript
type SessionAcpMeta = {
  backend: string;
  agent: string;
  runtimeSessionName: string;
  mode: "persistent" | "oneshot";
  cwd?: string;
  state: "idle" | "running" | "error";
  lastActivityAt: number;
  lastError?: string;
};
```

Campo persistido:

-   `SessionEntry.acp?: SessionAcpMeta`

Reglas de migración:

-   fase A: escritura dual (proyección `acp` + fuente de verdad SQLite ACP)
-   fase B: lectura principal desde SQLite ACP, lectura de respaldo desde `SessionEntry.acp` heredado
-   fase C: comando de migración rellena filas ACP faltantes desde entradas heredadas válidas
-   fase D: eliminar lectura de respaldo y mantener la proyección opcional solo para UX
-   los campos heredados (`cliSessionIds`, `claudeCliSessionId`) permanecen sin tocar

### Contrato de error

Añadir códigos de error ACP estables y mensajes orientados al usuario:

-   `ACP_BACKEND_MISSING`
    -   mensaje: `El backend de tiempo de ejecución ACP no está configurado. Instala y habilita el plugin de tiempo de ejecución acpx.`
-   `ACP_BACKEND_UNAVAILABLE`
    -   mensaje: `El backend de tiempo de ejecución ACP no está disponible actualmente. Inténtalo de nuevo en un momento.`
-   `ACP_SESSION_INIT_FAILED`
    -   mensaje: `No se pudo inicializar el tiempo de ejecución de la sesión ACP.`
-   `ACP_TURN_FAILED`
    -   mensaje: `El turno ACP falló antes de completarse.`

Reglas:

-   devolver un mensaje seguro y accionable para el usuario en el hilo
-   registrar el error detallado del backend/sistema solo en los registros de tiempo de ejecución
-   nunca volver silenciosamente a la ruta LLM normal cuando se seleccionó explícitamente el enrutamiento ACP

### Arbitraje de entrega duplicada

Regla de enrutamiento única para turnos ACP vinculados:

-   si existe una vinculación de hilo activa para la sesión ACP objetivo y el contexto del solicitante, entregar solo a ese hilo vinculado
-   no enviar también al canal padre para el mismo turno
-   si la selección del destino vinculado es ambigua, fallar cerrado con error explícito (sin respaldo implícito al padre)
-   si no existe ninguna vinculación activa, usar el comportamiento de destino de sesión normal

### Observabilidad y preparación operativa

Métricas requeridas:

-   recuento de éxito/fallo de generación ACP por backend y código de error
-   percentiles de latencia de ejecución ACP (espera en cola, tiempo de turno de ejecución, tiempo de proyección de entrega)
-   recuento de reinicios de actor ACP y razón de reinicio
-   recuento de detección de vinculaciones obsoletas
-   tasa de aciertos de reproducción de idempotencia
-   contadores de reintentos de entrega de Discord y límites de tasa

Registros requeridos:

-   registros estructurados con clave `sessionKey`, `runId`, `backend`, `threadId`, `idempotencyKey`
-   registros explícitos de transición de estado para las máquinas de estado de sesión y ejecución
-   registros de comandos del adaptador con argumentos seguros para redacción y resumen de salida

Diagnósticos requeridos:

-   `/acp sessions` incluye estado, ejecución activa, último error y estado de vinculación
-   `/acp doctor` (o equivalente) valida el registro del backend, la salud del almacén y las vinculaciones obsoletas

### Precedencia de configuración y valores efectivos

Precedencia de habilitación de ACP:

-   anulación de cuenta: `channels.discord.accounts..threadBindings.spawnAcpSessions`
-   anulación de canal: `channels.discord.threadBindings.spawnAcpSessions`
-   puerta ACP global: `acp.enabled`
-   puerta de despacho: `acp.dispatch.enabled`
-   disponibilidad del backend: backend registrado para `acp.backend`

Comportamiento de auto-habilitación:

-   cuando ACP está configurado (`acp.enabled=true`, `acp.dispatch.enabled=true`, o `acp.backend=acpx`), la auto-habilitación del plugin marca `plugins.entries.acpx.enabled=true` a menos que esté en lista de denegación o explícitamente deshabilitado

Valor efectivo de TTL:

-   `min(ttl de sesión, ttl de vinculación de hilo de discord, ttl de tiempo de ejecución de acp)`

### Mapa de pruebas

Pruebas unitarias:

-   `src/acp/runtime/registry.test.ts` (nuevo)
-   `src/auto-reply/reply/dispatch-from-config.acp.test.ts` (nuevo)
-   `src/infra/outbound/bound-delivery-router.test.ts` (extender casos de fallo cerrado ACP)
-   `src/config/sessions/types.test.ts` o pruebas de almacén de sesiones más cercanas (persistencia de metadatos ACP)

Pruebas de integración:

-   `src/discord/monitor/reply-delivery.test.ts` (comportamiento de destino de entrega ACP vinculado)
-   `src/discord/monitor/message-handler.preflight*.test.ts` (continuidad de enrutamiento de clave de sesión ACP vinculada)
-   pruebas de tiempo de ejecución del plugin acpx en el paquete backend (registro/inicio/detención del servicio + normalización de eventos)

Pruebas e2e de puerta de enlace:

-   `src/gateway/server.sessions.gateway-server-sessions-a.e2e.test.ts` (extender cobertura de ciclo de vida de reinicio/eliminación ACP)
-   e2e de turno de hilo ACP para generación, mensaje, transmisión, cancelación, desenfoque, recuperación de reinicio

### Guardia de implementación

Añadir interruptor de eliminación de despacho ACP independiente:

-   `acp.dispatch.enabled` predeterminado `false` para la primera versión
-   cuando está deshabilitado:
    -   los comandos de control de generación/enfoque ACP aún pueden vincular sesiones
    -   la ruta de despacho ACP no se activa
    -   el usuario recibe un mensaje explícito de que el despacho ACP está deshabilitado por política
-   después de la validación canaria, el valor predeterminado puede cambiarse a `true` en una versión posterior

## Plan de comandos y UX

### Comandos nuevos

-   `/acp spawn <agent-id> [--mode persistent|oneshot] [--thread auto|here|off]`
-   `/acp cancel [session]`
-   `/acp steer `
-   `/acp close [session]`
-   `/acp sessions`

### Compatibilidad de comandos existentes

-   `/focus ` continúa soportando objetivos ACP
-   `/unfocus` mantiene la semántica actual
-   `/session idle` y `/session max-age` reemplazan la antigua anulación de TTL

## Implementación por fases

### Fase 0 ADR y congelación de esquema

-   enviar ADR para propiedad del plano de control ACP y límites de adaptador
-   congelar esquema de base de datos (`acp_sessions`, `acp_runs`, `acp_bindings`, `acp_events`, `acp_delivery_checkpoint`, `acp_idempotency`)
-   definir códigos de error ACP estables, contrato de eventos y guardias de transición de estado

### Fase 1 Fundación del plano de control en el núcleo

-   implementar `AcpSessionManager` y tiempo de ejecución de actor por sesión
-   implementar almacén SQLite ACP y ayudantes de transacción
-   implementar almacén de idempotencia y ayudantes de reproducción