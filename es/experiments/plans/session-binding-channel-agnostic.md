

  Experimentos

  
# Plan de Enlace de Sesión Independiente del Canal

## Descripción general

Este documento define el modelo de enlace de sesión independiente del canal a largo plazo y el alcance concreto para la próxima iteración de implementación. Objetivo:

-   convertir el enrutamiento de sesiones enlazadas a subagentes en una capacidad central
-   mantener el comportamiento específico del canal en los adaptadores
-   evitar regresiones en el comportamiento normal de Discord

## Por qué existe esto

El comportamiento actual mezcla:

-   política de contenido de finalización
-   política de enrutamiento de destino
-   detalles específicos de Discord

Esto causó casos extremos como:

-   entrega duplicada en el canal principal y en el hilo bajo ejecuciones concurrentes
-   uso de tokens obsoletos en gestores de enlace reutilizados
-   falta de contabilidad de actividad para envíos mediante webhook

## Alcance de la iteración 1

Esta iteración está intencionalmente limitada.

### 1. Agregar interfaces centrales independientes del canal

Agregar tipos centrales e interfaces de servicio para enlaces y enrutamiento. Tipos centrales propuestos:

```bash
export type BindingTargetKind = "subagent" | "session";
export type BindingStatus = "active" | "ending" | "ended";

export type ConversationRef = {
  channel: string;
  accountId: string;
  conversationId: string;
  parentConversationId?: string;
};

export type SessionBindingRecord = {
  bindingId: string;
  targetSessionKey: string;
  targetKind: BindingTargetKind;
  conversation: ConversationRef;
  status: BindingStatus;
  boundAt: number;
  expiresAt?: number;
  metadata?: Record<string, unknown>;
};
```

Contrato del servicio central:

```bash
export interface SessionBindingService {
  bind(input: {
    targetSessionKey: string;
    targetKind: BindingTargetKind;
    conversation: ConversationRef;
    metadata?: Record<string, unknown>;
    ttlMs?: number;
  }): Promise<SessionBindingRecord>;

  listBySession(targetSessionKey: string): SessionBindingRecord[];
  resolveByConversation(ref: ConversationRef): SessionBindingRecord | null;
  touch(bindingId: string, at?: number): void;
  unbind(input: {
    bindingId?: string;
    targetSessionKey?: string;
    reason: string;
  }): Promise<SessionBindingRecord[]>;
}
```

### 2. Agregar un enrutador de entrega central para finalizaciones de subagentes

Agregar una única ruta de resolución de destino para eventos de finalización. Contrato del enrutador:

```bash
export interface BoundDeliveryRouter {
  resolveDestination(input: {
    eventKind: "task_completion";
    targetSessionKey: string;
    requester?: ConversationRef;
    failClosed: boolean;
  }): {
    binding: SessionBindingRecord | null;
    mode: "bound" | "fallback";
    reason: string;
  };
}
```

Para esta iteración:

-   solo `task_completion` se enruta a través de esta nueva ruta
-   las rutas existentes para otros tipos de eventos permanecen como están

### 3. Mantener Discord como adaptador

Discord sigue siendo la primera implementación de adaptador. Responsabilidades del adaptador:

-   crear/reutilizar conversaciones de hilo
-   enviar mensajes enlazados mediante webhook o envío de canal
-   validar el estado del hilo (archivado/eliminado)
-   mapear metadatos del adaptador (identidad del webhook, ids de hilo)

### 4. Corregir problemas de corrección actualmente conocidos

Requerido en esta iteración:

-   actualizar el uso del token al reutilizar un gestor de enlace de hilo existente
-   registrar actividad saliente para envíos de Discord basados en webhook
-   detener la vuelta al canal principal implícita cuando se selecciona un destino de hilo enlazado para la finalización en modo sesión

### 5. Preservar los valores predeterminados actuales de seguridad en tiempo de ejecución

Sin cambios de comportamiento para usuarios con la generación de hilos enlazados deshabilitada. Los valores predeterminados permanecen:

-   `channels.discord.threadBindings.spawnSubagentSessions = false`

Resultado:

-   los usuarios normales de Discord mantienen el comportamiento actual
-   la nueva ruta central afecta solo al enrutamiento de finalización de sesión enlazada donde esté habilitado

## No incluido en la iteración 1

Explícitamente diferido:

-   destinos de enlace ACP (`targetKind: "acp"`)
-   nuevos adaptadores de canal más allá de Discord
-   reemplazo global de todas las rutas de entrega (`spawn_ack`, futuro `subagent_message`)
-   cambios a nivel de protocolo
-   rediseño de migración/versionado del almacén para toda la persistencia de enlaces

Notas sobre ACP:

-   el diseño de la interfaz deja espacio para ACP
-   la implementación de ACP no se inicia en esta iteración

## Invariantes de enrutamiento

Estas invariantes son obligatorias para la iteración 1.

-   la selección de destino y la generación de contenido son pasos separados
-   si la finalización en modo sesión resuelve a un destino enlazado activo, la entrega debe apuntar a ese destino
-   no hay redirección oculta desde el destino enlazado al canal principal
-   el comportamiento de vuelta atrás debe ser explícito y observable

## Compatibilidad y despliegue

Objetivo de compatibilidad:

-   sin regresión para usuarios con la generación de hilos enlazados desactivada
-   sin cambios para canales que no sean Discord en esta iteración

Despliegue:

1.  Implementar interfaces y enrutador detrás de las puertas de características actuales.
2.  Enrutar las entregas enlazadas en modo de finalización de Discord a través del enrutador.
3.  Mantener la ruta heredada para flujos no enlazados.
4.  Verificar con pruebas específicas y registros de tiempo de ejecución en canary.

## Pruebas requeridas en la iteración 1

Cobertura de unidad e integración requerida:

-   la rotación de tokens del gestor utiliza el token más reciente después de reutilizar el gestor
-   los envíos mediante webhook actualizan las marcas de tiempo de actividad del canal
-   dos sesiones enlazadas activas en el mismo canal del solicitante no se duplican en el canal principal
-   la finalización para una ejecución en modo sesión enlazada resuelve solo al destino del hilo
-   la bandera de generación deshabilitada mantiene el comportamiento heredado sin cambios

## Archivos de implementación propuestos

Central:

-   `src/infra/outbound/session-binding-service.ts` (nuevo)
-   `src/infra/outbound/bound-delivery-router.ts` (nuevo)
-   `src/agents/subagent-announce.ts` (integración de resolución de destino de finalización)

Adaptador de Discord y tiempo de ejecución:

-   `src/discord/monitor/thread-bindings.manager.ts`
-   `src/discord/monitor/reply-delivery.ts`
-   `src/discord/send.outbound.ts`

Pruebas:

-   `src/discord/monitor/provider*.test.ts`
-   `src/discord/monitor/reply-delivery.test.ts`
-   `src/agents/subagent-announce.format.test.ts`

## Criterios de finalización para la iteración 1

-   las interfaces centrales existen y están conectadas para el enrutamiento de finalizaciones
-   las correcciones de exactitud anteriores están fusionadas con pruebas
-   no hay entrega de finalización duplicada en el canal principal y el hilo en ejecuciones enlazadas en modo sesión
-   sin cambios de comportamiento para despliegues con generación enlazada deshabilitada
-   ACP permanece explícitamente diferido

[Plan de Supervisión de PTY y Procesos](./pty-process-supervision.md)[Investigación sobre Memoria del Espacio de Trabajo](../research/memory.md)