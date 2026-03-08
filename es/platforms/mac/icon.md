

  Aplicación complementaria de macOS

  
# Icono de la Barra de Menú

Autor: steipete · Actualizado: 2025-12-06 · Alcance: aplicación macOS (`apps/macos`)

-   **Inactivo:** Animación normal del icono (parpadeo, ocasional meneo).
-   **En pausa:** El elemento de estado usa `appearsDisabled`; sin movimiento.
-   **Activación por voz (orejas grandes):** El detector de activación por voz llama a `AppState.triggerVoiceEars(ttl: nil)` cuando se escucha la palabra de activación, manteniendo `earBoostActive=true` mientras se captura la expresión. Las orejas se escalan (1.9x), obtienen agujeros circulares para legibilidad, luego bajan mediante `stopVoiceEars()` después de 1s de silencio. Solo se activa desde la canalización de voz dentro de la aplicación.
-   **Trabajando (agente en ejecución):** `AppState.isWorking=true` impulsa un micro-movimiento de "carrera de cola/patas": meneo de patas más rápido y ligero desplazamiento mientras el trabajo está en curso. Actualmente se activa/desactiva alrededor de las ejecuciones del agente WebChat; añade el mismo interruptor alrededor de otras tareas largas cuando las conectes.

Puntos de conexión

-   Activación por voz: el tiempo de ejecución/probador llama a `AppState.triggerVoiceEars(ttl: nil)` al activarse y a `stopVoiceEars()` después de 1s de silencio para coincidir con la ventana de captura.
-   Actividad del agente: establece `AppStateStore.shared.setWorking(true/false)` alrededor de los intervalos de trabajo (ya hecho en la llamada del agente WebChat). Mantén los intervalos cortos y restablece en bloques `defer` para evitar animaciones atascadas.

Formas y tamaños

-   Icono base dibujado en `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)`.
-   La escala de orejas por defecto es `1.0`; el impulso de voz establece `earScale=1.9` y activa `earHoles=true` sin cambiar el marco general (imagen plantilla de 18×18 pt renderizada en un almacén de respaldo Retina de 36×36 px).
-   La carrera usa meneo de patas hasta ~1.0 con un pequeño movimiento horizontal; es aditivo a cualquier meneo inactivo existente.

Notas de comportamiento

-   No hay interruptor CLI/broker externo para orejas/trabajando; mantenlo interno a las señales propias de la aplicación para evitar activaciones accidentales.
-   Mantén los TTL cortos (<10s) para que el icono vuelva a la línea base rápidamente si un trabajo se cuelga.

[Comprobaciones de Estado](./health.md)[Registro en macOS](./logging.md)

---