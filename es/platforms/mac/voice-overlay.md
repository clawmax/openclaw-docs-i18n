

  Aplicación complementaria de macOS

  
# Superposición de Voz

Audiencia: colaboradores de la aplicación de macOS. Objetivo: mantener la superposición de voz predecible cuando se solapan la palabra de activación y el pulsar para hablar.

## Intención actual

-   Si la superposición ya es visible por la palabra de activación y el usuario presiona la tecla de acceso rápido, la sesión de la tecla *adopta* el texto existente en lugar de restablecerlo. La superposición permanece visible mientras se mantiene presionada la tecla. Cuando el usuario la suelta: enviar si hay texto recortado, de lo contrario descartar.
-   La palabra de activación por sí sola aún envía automáticamente en silencio; el pulsar para hablar envía inmediatamente al soltar.

## Implementado (9 de diciembre de 2025)

-   Las sesiones de superposición ahora llevan un token por captura (palabra de activación o pulsar para hablar). Las actualizaciones parciales/finales/envío/descarte/nivel se descartan cuando el token no coincide, evitando devoluciones de llamada obsoletas.
-   El pulsar para hablar adopta cualquier texto visible de la superposición como prefijo (así que presionar la tecla de acceso rápido mientras la superposición de activación está visible mantiene el texto y añade el nuevo habla). Espera hasta 1.5s para una transcripción final antes de recurrir al texto actual.
-   El registro de tono/superposición se emite en `info` en las categorías `voicewake.overlay`, `voicewake.ptt` y `voicewake.chime` (inicio de sesión, parcial, final, envío, descarte, motivo del tono).

## Próximos pasos

1.  **VoiceSessionCoordinator (actor)**
    -   Posee exactamente una `VoiceSession` a la vez.
    -   API (basada en token): `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`.
    -   Descarta devoluciones de llamada que llevan tokens obsoletos (evita que reconocedores antiguos vuelvan a abrir la superposición).
2.  **VoiceSession (modelo)**
    -   Campos: `token`, `source` (wakeWord|pushToTalk), texto comprometido/volátil, banderas de tono, temporizadores (envío automático, inactivo), `overlayMode` (visualización|edición|envío), fecha límite de enfriamiento.
3.  **Vinculación de superposición**
    -   `VoiceSessionPublisher` (`ObservableObject`) refleja la sesión activa en SwiftUI.
    -   `VoiceWakeOverlayView` se renderiza solo a través del publicador; nunca modifica singletons globales directamente.
    -   Las acciones del usuario en la superposición (`sendNow`, `dismiss`, `edit`) llaman de vuelta al coordinador con el token de sesión.
4.  **Ruta de envío unificada**
    -   En `endCapture`: si el texto recortado está vacío → descartar; de lo contrario `performSend(session:)` (reproduce el tono de envío una vez, reenvía, descarta).
    -   Pulsar para hablar: sin retraso; palabra de activación: retraso opcional para envío automático.
    -   Aplicar un breve período de enfriamiento al tiempo de ejecución de activación después de que termine el pulsar para hablar, para que la palabra de activación no se vuelva a activar inmediatamente.
5.  **Registro**
    -   El coordinador emite registros `.info` en el subsistema `ai.openclaw`, categorías `voicewake.overlay` y `voicewake.chime`.
    -   Eventos clave: `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`.

## Lista de verificación para depuración

-   Transmitir registros mientras se reproduce una superposición persistente:
    
    Copiar
    
    ```bash
    sudo log stream --predicate 'subsystem == "ai.openclaw" AND category CONTAINS "voicewake"' --level info --style compact
    ```
    
-   Verificar que solo hay un token de sesión activo; las devoluciones de llamada obsoletas deben ser descartadas por el coordinador.
-   Asegurarse de que la liberación del pulsar para hablar siempre llame a `endCapture` con el token activo; si el texto está vacío, esperar `dismiss` sin tono ni envío.

## Pasos de migración (sugeridos)

1.  Agregar `VoiceSessionCoordinator`, `VoiceSession` y `VoiceSessionPublisher`.
2.  Refactorizar `VoiceWakeRuntime` para crear/actualizar/terminar sesiones en lugar de tocar `VoiceWakeOverlayController` directamente.
3.  Refactorizar `VoicePushToTalk` para adoptar sesiones existentes y llamar a `endCapture` al soltar; aplicar enfriamiento del tiempo de ejecución.
4.  Conectar `VoiceWakeOverlayController` al publicador; eliminar llamadas directas desde runtime/PTT.
5.  Agregar pruebas de integración para adopción de sesiones, enfriamiento y descarte de texto vacío.

[Voice Wake](./voicewake.md)[WebChat](./webchat.md)