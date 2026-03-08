

  Experimentos

  
# Plan de Supervisión de Procesos y PTY

## 1. Problema y objetivo

Necesitamos un ciclo de vida confiable para la ejecución de comandos de larga duración en:

-   Ejecuciones en primer plano (`exec`)
-   Ejecuciones en segundo plano (`exec`)
-   Acciones de seguimiento de `process` (`poll`, `log`, `send-keys`, `paste`, `submit`, `kill`, `remove`)
-   Subprocesos del ejecutor del agente CLI

El objetivo no es solo soportar PTY. El objetivo es una propiedad, cancelación, tiempo de espera y limpieza predecibles, sin heurísticas inseguras de coincidencia de procesos.

## 2. Alcance y límites

-   Mantener la implementación interna en `src/process/supervisor`.
-   No crear un nuevo paquete para esto.
-   Mantener la compatibilidad con el comportamiento actual cuando sea práctico.
-   No ampliar el alcance a la reproducción de terminal o la persistencia de sesiones estilo tmux.

## 3. Implementado en esta rama

### Línea base del supervisor ya presente

-   El módulo Supervisor está implementado en `src/process/supervisor/*`.
-   El tiempo de ejecución de Exec y el ejecutor CLI ya se enrutan a través del spawn y wait del supervisor.
-   La finalización del registro es idempotente.

### Esta iteración completada

1.  Contrato de comando PTY explícito

-   `SpawnInput` es ahora una unión discriminada en `src/process/supervisor/types.ts`.
-   Las ejecuciones PTY requieren `ptyCommand` en lugar de reutilizar el `argv` genérico.
-   El supervisor ya no reconstruye cadenas de comandos PTY a partir de uniones de argv en `src/process/supervisor/supervisor.ts`.
-   El tiempo de ejecución de Exec ahora pasa `ptyCommand` directamente en `src/agents/bash-tools.exec-runtime.ts`.

2.  Desacoplamiento de tipos en la capa de proceso

-   Los tipos del Supervisor ya no importan `SessionStdin` de los agentes.
-   El contrato de stdin local del proceso reside en `src/process/supervisor/types.ts` (`ManagedRunStdin`).
-   Los adaptadores ahora dependen solo de tipos de nivel de proceso:
    -   `src/process/supervisor/adapters/child.ts`
    -   `src/process/supervisor/adapters/pty.ts`

3.  Mejora de la propiedad del ciclo de vida de la herramienta de proceso

-   `src/agents/bash-tools.process.ts` ahora solicita la cancelación primero a través del supervisor.
-   `process kill/remove` ahora usa la terminación de respaldo por árbol de procesos cuando falla la búsqueda en el supervisor.
-   `remove` mantiene un comportamiento de eliminación determinista al descartar las entradas de sesión en ejecución inmediatamente después de solicitar la terminación.

4.  Fuente única de valores predeterminados del watchdog

-   Se agregaron valores predeterminados compartidos en `src/agents/cli-watchdog-defaults.ts`.
-   `src/agents/cli-backends.ts` consume los valores predeterminados compartidos.
-   `src/agents/cli-runner/reliability.ts` consume los mismos valores predeterminados compartidos.

5.  Limpieza de ayudantes muertos

-   Se eliminó la ruta del ayudante `killSession` no utilizada de `src/agents/bash-tools.shared.ts`.

6.  Pruebas de ruta directa del supervisor agregadas

-   Se agregó `src/agents/bash-tools.process.supervisor.test.ts` para cubrir el enrutamiento de kill y remove a través de la cancelación del supervisor.

7.  Correcciones de brechas de confiabilidad completadas

-   `src/agents/bash-tools.process.ts` ahora recurre a la terminación real a nivel de sistema operativo cuando falla la búsqueda en el supervisor.
-   `src/process/supervisor/adapters/child.ts` ahora usa semánticas de terminación de árbol de procesos para las rutas de eliminación predeterminadas de cancelación/tiempo de espera.
-   Se agregó una utilidad compartida de árbol de procesos en `src/process/kill-tree.ts`.

8.  Cobertura de casos extremos del contrato PTY agregada

-   Se agregó `src/process/supervisor/supervisor.pty-command.test.ts` para el reenvío literal de comandos PTY y el rechazo de comandos vacíos.
-   Se agregó `src/process/supervisor/adapters/child.test.ts` para el comportamiento de eliminación de árbol de procesos en la cancelación del adaptador hijo.

## 4. Brechas y decisiones restantes

### Estado de confiabilidad

Las dos brechas de confiabilidad requeridas para esta iteración ahora están cerradas:

-   `process kill/remove` ahora tiene una terminación real del sistema operativo como respaldo cuando falla la búsqueda en el supervisor.
-   la cancelación/tiempo de espera del proceso hijo ahora usa semánticas de eliminación de árbol de procesos para la ruta de eliminación predeterminada.
-   Se agregaron pruebas de regresión para ambos comportamientos.

### Durabilidad y reconciliación de inicio

El comportamiento de reinicio ahora está definido explícitamente como solo ciclo de vida en memoria.

-   `reconcileOrphans()` permanece como una no-operación en `src/process/supervisor/supervisor.ts` por diseño.
-   Las ejecuciones activas no se recuperan después del reinicio del proceso.
-   Este límite es intencional para esta iteración de implementación para evitar riesgos de persistencia parcial.

### Seguimientos de mantenibilidad

1.  `runExecProcess` en `src/agents/bash-tools.exec-runtime.ts` aún maneja múltiples responsabilidades y puede dividirse en ayudantes enfocados en un seguimiento.

## 5. Plan de implementación

La iteración de implementación para los elementos de confiabilidad y contrato requeridos está completa. Completado:

-   Terminación real de respaldo para `process kill/remove`
-   Cancelación por árbol de procesos para la ruta de eliminación predeterminada del adaptador hijo
-   Pruebas de regresión para la eliminación de respaldo y la ruta de eliminación del adaptador hijo
-   Pruebas de casos extremos de comandos PTY bajo `ptyCommand` explícito
-   Límite de reinicio en memoria explícito con `reconcileOrphans()` como no-operación por diseño

Seguimiento opcional:

-   dividir `runExecProcess` en ayudantes enfocados sin desviación de comportamiento

## 6. Mapa de archivos

### Supervisor de procesos

-   `src/process/supervisor/types.ts` actualizado con entrada de spawn discriminada y contrato de stdin local del proceso.
-   `src/process/supervisor/supervisor.ts` actualizado para usar `ptyCommand` explícito.
-   `src/process/supervisor/adapters/child.ts` y `src/process/supervisor/adapters/pty.ts` desacoplados de los tipos de agente.
-   `src/process/supervisor/registry.ts` finalización idempotente sin cambios y retenida.

### Integración de Exec y proceso

-   `src/agents/bash-tools.exec-runtime.ts` actualizado para pasar el comando PTY explícitamente y mantener la ruta de respaldo.
-   `src/agents/bash-tools.process.ts` actualizado para cancelar a través del supervisor con terminación de respaldo real por árbol de procesos.
-   `src/agents/bash-tools.shared.ts` eliminó la ruta del ayudante de eliminación directa.

### Confiabilidad CLI

-   `src/agents/cli-watchdog-defaults.ts` agregado como línea base compartida.
-   `src/agents/cli-backends.ts` y `src/agents/cli-runner/reliability.ts` ahora consumen los mismos valores predeterminados.

## 7. Ejecución de validación en esta iteración

Pruebas unitarias:

-   `pnpm vitest src/process/supervisor/registry.test.ts`
-   `pnpm vitest src/process/supervisor/supervisor.test.ts`
-   `pnpm vitest src/process/supervisor/supervisor.pty-command.test.ts`
-   `pnpm vitest src/process/supervisor/adapters/child.test.ts`
-   `pnpm vitest src/agents/cli-backends.test.ts`
-   `pnpm vitest src/agents/bash-tools.exec.pty-cleanup.test.ts`
-   `pnpm vitest src/agents/bash-tools.process.poll-timeout.test.ts`
-   `pnpm vitest src/agents/bash-tools.process.supervisor.test.ts`
-   `pnpm vitest src/process/exec.test.ts`

Objetivos E2E:

-   `pnpm vitest src/agents/cli-runner.test.ts`
-   `pnpm vitest run src/agents/bash-tools.exec.pty-fallback.test.ts src/agents/bash-tools.exec.background-abort.test.ts src/agents/bash-tools.process.send-keys.test.ts`

Nota de verificación de tipos:

-   Usa `pnpm build` (y `pnpm check` para la puerta completa de lint/docs) en este repositorio. Las notas antiguas que mencionan `pnpm tsgo` están obsoletas.

## 8. Garantías operativas preservadas

-   El comportamiento de fortalecimiento del entorno de Exec no ha cambiado.
-   El flujo de aprobación y lista de permitidos no ha cambiado.
-   La sanitización de salida y los límites de salida no han cambiado.
-   El adaptador PTY aún garantiza el asentamiento de wait en la eliminación forzada y la disposición de listeners.

## 9. Definición de terminado

1.  El Supervisor es el propietario del ciclo de vida para las ejecuciones gestionadas.
2.  El spawn PTY usa un contrato de comando explícito sin reconstrucción de argv.
3.  La capa de proceso no tiene dependencia de tipos en la capa de agente para los contratos de stdin del supervisor.
4.  Los valores predeterminados del watchdog son de una sola fuente.
5.  Las pruebas unitarias y e2e específicas permanecen en verde.
6.  El límite de durabilidad de reinicio está explícitamente documentado o completamente implementado.

## 10. Resumen

La rama ahora tiene una forma de supervisión coherente y más segura:

-   contrato PTY explícito
-   capas de proceso más limpias
-   ruta de cancelación impulsada por el supervisor para operaciones de proceso
-   terminación de respaldo real cuando falla la búsqueda en el supervisor
-   cancelación por árbol de procesos para las rutas de eliminación predeterminadas de ejecuciones hijo
-   valores predeterminados unificados del watchdog
-   límite de reinicio en memoria explícito (sin reconciliación de huérfanos a través del reinicio en esta iteración)

[Plan de Puerta de Enlace OpenResponses](./openresponses-gateway.md)[Plan de Enlace de Sesión Independiente del Canal](./session-binding-channel-agnostic.md)