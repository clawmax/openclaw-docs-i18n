

  Experimentos

  
# Plan de Refactorización de Streaming de Tiempo de Ejecución Unificado

## Objetivo

Entregar una canalización de streaming compartida para `main`, `subagent` y `acp` para que todos los tiempos de ejecución obtengan un comportamiento idéntico de coalescencia, fragmentación, orden de entrega y recuperación ante fallos.

## Por qué existe esto

-   El comportamiento actual está dividido en múltiples rutas de conformación específicas de cada tiempo de ejecución.
-   Los errores de formato/coalescencia pueden corregirse en una ruta pero permanecer en otras.
-   La consistencia de entrega, la supresión de duplicados y la semántica de recuperación son más difíciles de razonar.

## Arquitectura objetivo

Canalización única, adaptadores específicos por tiempo de ejecución:

1.  Los adaptadores de tiempo de ejecución emiten solo eventos canónicos.
2.  El ensamblador de flujo compartido coalesce y finaliza eventos de texto/herramienta/estado.
3.  El proyector de canal compartido aplica la fragmentación/formato específico del canal una sola vez.
4.  El libro mayor de entrega compartido aplica semántica de envío/reproducción idempotente.
5.  El adaptador de canal saliente ejecuta los envíos y registra los puntos de control de entrega.

Contrato de evento canónico:

-   `turn_started`
-   `text_delta`
-   `block_final`
-   `tool_started`
-   `tool_finished`
-   `status`
-   `turn_completed`
-   `turn_failed`
-   `turn_cancelled`

## Flujos de trabajo

### 1) Contrato de streaming canónico

-   Definir un esquema de evento estricto + validación en el núcleo.
-   Añadir pruebas de contrato de adaptador para garantizar que cada tiempo de ejecución emita eventos compatibles.
-   Rechazar eventos de tiempo de ejecución malformados temprano y mostrar diagnósticos estructurados.

### 2) Procesador de flujo compartido

-   Reemplazar la lógica de coalescencia/proyección específica del tiempo de ejecución con un solo procesador.
-   El procesador posee el almacenamiento en búfer de deltas de texto, el vaciado por inactividad, la división de fragmentos máximos y el vaciado por finalización.
-   Mover la resolución de configuración de ACP/main/subagent a un solo ayudante para prevenir desviaciones.

### 3) Proyección de canal compartido

-   Mantener los adaptadores de canal simples: aceptar bloques finalizados y enviar.
-   Mover las peculiaridades de fragmentación específicas de Discord solo al proyector de canal.
-   Mantener la canalización independiente del canal antes de la proyección.

### 4) Libro mayor de entrega + reproducción

-   Añadir IDs de entrega por turno/por fragmento.
-   Registrar puntos de control antes y después del envío físico.
-   Al reiniciar, reproducir fragmentos pendientes de forma idempotente y evitar duplicados.

### 5) Migración y transición

-   Fase 1: modo sombra (la nueva canalización calcula la salida pero la ruta antigua envía; comparar).
-   Fase 2: transición tiempo de ejecución por tiempo de ejecución (`acp`, luego `subagent`, luego `main` o al revés según el riesgo).
-   Fase 3: eliminar el código de streaming heredado específico del tiempo de ejecución.

## No objetivos

-   No se realizarán cambios en el modelo de políticas/permisos de ACP en esta refactorización.
-   No se expandirán funciones específicas del canal fuera de las correcciones de compatibilidad de proyección.
-   No se rediseñará el transporte/backend (el contrato del plugin acpx permanece como está a menos que sea necesario para la paridad de eventos).

## Riesgos y mitigaciones

-   Riesgo: regresiones de comportamiento en las rutas existentes de main/subagent. Mitigación: comparación de diferencias en modo sombra + pruebas de contrato de adaptador + pruebas e2e de canal.
-   Riesgo: envíos duplicados durante la recuperación ante fallos. Mitigación: IDs de entrega duraderos + reproducción idempotente en el adaptador de entrega.
-   Riesgo: los adaptadores de tiempo de ejecución divergen nuevamente. Mitigación: suite de pruebas de contrato compartido obligatoria para todos los adaptadores.

## Criterios de aceptación

-   Todos los tiempos de ejecución pasan las pruebas de contrato de streaming compartido.
-   Discord ACP/main/subagent producen un comportamiento equivalente de espaciado/fragmentación para deltas pequeños.
-   La reproducción tras fallo/reinicio no envía fragmentos duplicados para el mismo ID de entrega.
-   Se elimina la ruta heredada de proyector/coalescedor de ACP.
-   La resolución de configuración de streaming es compartida e independiente del tiempo de ejecución.

[Agentes Vinculados a Hilos de ACP](./acp-thread-bound-agents.md)[Refactorización de CDP de Evaluación del Navegador](./browser-evaluate-cdp-refactor.md)