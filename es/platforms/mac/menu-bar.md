

  Aplicación complementaria para macOS

  
# Barra de Menú

## Qué se muestra

-   Mostramos el estado de trabajo actual del agente en el icono de la barra de menú y en la primera fila de estado del menú.
-   El estado de salud se oculta mientras el trabajo está activo; regresa cuando todas las sesiones están inactivas.
-   El bloque "Nodos" en el menú lista solo **dispositivos** (nodos emparejados vía `node.list`), no entradas de cliente/presencia.
-   Una sección "Uso" aparece bajo Contexto cuando hay instantáneas de uso del proveedor disponibles.

## Modelo de estado

-   Sesiones: los eventos llegan con `runId` (por ejecución) más `sessionKey` en el payload. La sesión "principal" es la clave `main`; si está ausente, recurrimos a la sesión actualizada más recientemente.
-   Prioridad: la principal siempre gana. Si la principal está activa, su estado se muestra inmediatamente. Si la principal está inactiva, se muestra la sesión no principal más recientemente activa. No cambiamos a mitad de una actividad; solo cambiamos cuando la sesión actual se vuelve inactiva o la principal se vuelve activa.
-   Tipos de actividad:
    -   `job`: ejecución de comandos de alto nivel (`state: started|streaming|done|error`).
    -   `tool`: `phase: start|result` con `toolName` y `meta/args`.

## Enum IconState (Swift)

-   `idle`
-   `workingMain(ActivityKind)`
-   `workingOther(ActivityKind)`
-   `overridden(ActivityKind)` (anulación de depuración)

### ActivityKind → glifo

-   `exec` → 💻
-   `read` → 📄
-   `write` → ✍️
-   `edit` → 📝
-   `attach` → 📎
-   predeterminado → 🛠️

### Mapeo visual

-   `idle`: criatura normal.
-   `workingMain`: insignia con glifo, tinte completo, animación de "trabajando" en la pata.
-   `workingOther`: insignia con glifo, tinte apagado, sin movimiento.
-   `overridden`: usa el glifo/tinte elegido independientemente de la actividad.

## Texto de la fila de estado (menú)

-   Mientras el trabajo está activo: ` · `
    -   Ejemplos: `Principal · exec: pnpm test`, `Otra · read: apps/macos/Sources/OpenClaw/AppState.swift`.
-   Cuando está inactivo: recurre al resumen de salud.

## Ingesta de eventos

-   Fuente: eventos `agent` del canal de control (`ControlChannel.handleAgentEvent`).
-   Campos analizados:
    -   `stream: "job"` con `data.state` para inicio/detención.
    -   `stream: "tool"` con `data.phase`, `name`, opcional `meta`/`args`.
-   Etiquetas:
    -   `exec`: primera línea de `args.command`.
    -   `read`/`write`: ruta acortada.
    -   `edit`: ruta más el tipo de cambio inferido de `meta`/recuentos de diferencias.
    -   alternativa: nombre de la herramienta.

## Anulación de depuración

-   Configuración ▸ Depuración ▸ Selector "Anulación de icono":
    -   `Sistema (automático)` (predeterminado)
    -   `Trabajando: principal` (por tipo de herramienta)
    -   `Trabajando: otra` (por tipo de herramienta)
    -   `Inactivo`
-   Almacenado vía `@AppStorage("iconOverride")`; mapeado a `IconState.overridden`.

## Lista de verificación para pruebas

-   Activar trabajo de sesión principal: verificar que el icono cambia inmediatamente y la fila de estado muestra la etiqueta principal.
-   Activar trabajo de sesión no principal mientras la principal está inactiva: icono/estado muestra no principal; permanece estable hasta que termina.
-   Iniciar principal mientras otra está activa: el icono cambia a principal al instante.
-   Ráfagas rápidas de herramientas: asegurar que la insignia no parpadee (período de gracia TTL en resultados de herramientas).
-   La fila de salud reaparece una vez que todas las sesiones están inactivas.

[Configuración de Desarrollo para macOS](./dev-setup.md)[Activación por Voz](./voicewake.md)

---