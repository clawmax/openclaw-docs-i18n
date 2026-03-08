

  Aplicación complementaria para macOS

  
# Voice Wake

## Modos

-   **Modo palabra de activación** (predeterminado): el reconocedor de voz siempre activo espera los tokens de activación (`swabbleTriggerWords`). Al coincidir, inicia la captura, muestra la superposición con texto parcial y lo envía automáticamente tras el silencio.
-   **Pulsar para hablar (mantener Opción derecha)**: mantén presionada la tecla Opción derecha para capturar inmediatamente—sin necesidad de activación. La superposición aparece mientras se mantiene presionada; soltarla finaliza y reenvía tras un breve retardo para que puedas ajustar el texto.

## Comportamiento en tiempo de ejecución (palabra de activación)

-   El reconocedor de voz reside en `VoiceWakeRuntime`.
-   El disparador solo se activa cuando hay una **pausa significativa** entre la palabra de activación y la siguiente palabra (~0.55s de separación). La superposición/tono puede comenzar en la pausa incluso antes de que empiece el comando.
-   Ventanas de silencio: 2.0s cuando hay flujo de voz, 5.0s si solo se escuchó el disparador.
-   Parada forzada: 120s para evitar sesiones descontroladas.
-   Rebote entre sesiones: 350ms.
-   La superposición es manejada por `VoiceWakeOverlayController` con coloreado comprometido/volátil.
-   Después del envío, el reconocedor se reinicia limpiamente para escuchar el siguiente disparador.

## Invariantes del ciclo de vida

-   Si Voice Wake está habilitado y se han concedido los permisos, el reconocedor de palabra de activación debería estar escuchando (excepto durante una captura explícita de pulsar para hablar).
-   La visibilidad de la superposición (incluyendo el descarte manual mediante el botón X) nunca debe impedir que el reconocedor se reanude.

## Modo de fallo de superposición persistente (anterior)

Anteriormente, si la superposición se quedaba visible atascada y la cerrabas manualmente, Voice Wake podía parecer "muerto" porque el intento de reinicio del runtime podía bloquearse por la visibilidad de la superposición y no se programaba ningún reinicio posterior. Refuerzo:

-   El reinicio del runtime de activación ya no se bloquea por la visibilidad de la superposición.
-   La finalización del descarte de la superposición activa un `VoiceWakeRuntime.refresh(...)` a través de `VoiceSessionCoordinator`, por lo que el descarte manual con la X siempre reanuda la escucha.

## Detalles de pulsar para hablar

-   La detección de tecla rápida usa un monitor global `.flagsChanged` para la **Opción derecha** (`keyCode 61` + `.option`). Solo observamos eventos (sin consumirlos).
-   La canalización de captura reside en `VoicePushToTalk`: inicia Speech inmediatamente, transmite parciales a la superposición y llama a `VoiceWakeForwarder` al soltar.
-   Cuando comienza pulsar para hablar, pausamos el runtime de palabra de activación para evitar conflictos de captura de audio; se reinicia automáticamente después de soltar.
-   Permisos: requiere Micrófono + Voz; para ver eventos necesita aprobación de Accesibilidad/Monitoreo de Entrada.
-   Teclados externos: algunos pueden no exponer la Opción derecha como se espera—ofrece un atajo alternativo si los usuarios reportan fallos.

## Ajustes visibles para el usuario

-   **Alternar Voice Wake**: habilita el runtime de palabra de activación.
-   **Mantener Cmd+Fn para hablar**: habilita el monitor de pulsar para hablar. Deshabilitado en macOS < 26.
-   Selectores de idioma y micrófono, medidor de nivel en vivo, tabla de palabras de activación, probador (solo local; no reenvía).
-   El selector de micrófono preserva la última selección si un dispositivo se desconecta, muestra una indicación de desconectado y temporalmente recurre al predeterminado del sistema hasta que regrese.
-   **Sonidos**: tonos al detectar el disparador y al enviar; predeterminado al sonido del sistema "Glass" de macOS. Puedes elegir cualquier archivo cargable por `NSSound` (ej. MP3/WAV/AIFF) para cada evento o seleccionar **Sin Sonido**.

## Comportamiento de reenvío

-   Cuando Voice Wake está habilitado, las transcripciones se reenvían a la puerta de enlace/agente activa (el mismo modo local vs remoto usado por el resto de la aplicación de mac).
-   Las respuestas se entregan al **proveedor principal usado por última vez** (WhatsApp/Telegram/Discord/WebChat). Si falla la entrega, el error se registra y la ejecución sigue siendo visible a través de WebChat/registros de sesión.

## Carga útil de reenvío

-   `VoiceWakeForwarder.prefixedTranscript(_:)` antepone la pista de máquina antes de enviar. Compartido entre las rutas de palabra de activación y pulsar para hablar.

## Verificación rápida

-   Activa pulsar para hablar, mantén Cmd+Fn, habla, suelta: la superposición debería mostrar parciales y luego enviar.
-   Mientras se mantiene presionado, las orejas de la barra de menú deberían permanecer agrandadas (usa `triggerVoiceEars(ttl:nil)`); se reducen después de soltar.

[Barra de Menú](./menu-bar.md)[Superposición de Voz](./voice-overlay.md)

---