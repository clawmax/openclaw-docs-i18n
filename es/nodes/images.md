

  Medios y dispositivos

  
# Soporte de Imágenes y Medios

El canal de WhatsApp funciona a través de **Baileys Web**. Este documento captura las reglas actuales de manejo de medios para envíos, puerta de enlace y respuestas del agente.

## Objetivos

-   Enviar medios con subtítulos opcionales mediante `openclaw message send --media`.
-   Permitir que las respuestas automáticas desde la bandeja de entrada web incluyan medios junto con texto.
-   Mantener límites por tipo sensatos y predecibles.

## Superficie de CLI

-   `openclaw message send --media <path-or-url> [--message ]`
    -   `--media` opcional; el subtítulo puede estar vacío para envíos solo de medios.
    -   `--dry-run` imprime la carga útil resuelta; `--json` emite `{ channel, to, messageId, mediaUrl, caption }`.

## Comportamiento del canal WhatsApp Web

-   Entrada: ruta de archivo local **o** URL HTTP(S).
-   Flujo: cargar en un Buffer, detectar el tipo de medio y construir la carga útil correcta:
    -   **Imágenes:** redimensionar y recomprimir a JPEG (lado máximo 2048px) apuntando a `agents.defaults.mediaMaxMb` (por defecto 5 MB), con un límite de 6 MB.
    -   **Audio/Voz/Video:** pasar directamente hasta 16 MB; el audio se envía como nota de voz (`ptt: true`).
    -   **Documentos:** cualquier otra cosa, hasta 100 MB, conservando el nombre del archivo cuando esté disponible.
-   Reproducción estilo GIF de WhatsApp: enviar un MP4 con `gifPlayback: true` (CLI: `--gif-playback`) para que los clientes móviles lo reproduzcan en bucle en línea.
-   La detección de MIME prefiere bytes mágicos, luego encabezados, luego la extensión del archivo.
-   El subtítulo proviene de `--message` o `reply.text`; se permite un subtítulo vacío.
-   Registro: el modo no-verbose muestra `↩️`/`✅`; el verbose incluye tamaño y ruta/URL de origen.

## Canalización de Respuesta Automática

-   `getReplyFromConfig` devuelve `{ text?, mediaUrl?, mediaUrls? }`.
-   Cuando hay medios presentes, el remitente web resuelve rutas locales o URL utilizando la misma canalización que `openclaw message send`.
-   Múltiples entradas de medios se envían secuencialmente si se proporcionan.

## Medios Entrantes a Comandos (Pi)

-   Cuando los mensajes web entrantes incluyen medios, OpenClaw los descarga a un archivo temporal y expone variables de plantilla:
    -   `{{MediaUrl}}` pseudo-URL para el medio entrante.
    -   `{{MediaPath}}` ruta local temporal escrita antes de ejecutar el comando.
-   Cuando se habilita un sandbox de Docker por sesión, los medios entrantes se copian al espacio de trabajo del sandbox y `MediaPath`/`MediaUrl` se reescriben a una ruta relativa como `media/inbound/`.
-   La comprensión de medios (si se configura mediante `tools.media.*` o `tools.media.models` compartidos) se ejecuta antes de la plantilla y puede insertar bloques `[Image]`, `[Audio]` y `[Video]` en `Body`.
    -   El audio establece `{{Transcript}}` y usa la transcripción para el análisis de comandos, por lo que los comandos de barra diagonal aún funcionan.
    -   Las descripciones de video e imagen preservan cualquier texto de subtítulo para el análisis de comandos.
-   Por defecto solo se procesa el primer archivo adjunto de imagen/audio/video coincidente; establece `tools.media..attachments` para procesar múltiples archivos adjuntos.

## Límites y Errores

**Límites de envío saliente (envío web de WhatsApp)**

-   Imágenes: límite de ~6 MB después de la recompresión.
-   Audio/voz/video: límite de 16 MB; documentos: límite de 100 MB.
-   Medios demasiado grandes o ilegibles → error claro en los registros y la respuesta se omite.

**Límites de comprensión de medios (transcripción/descripción)**

-   Imagen por defecto: 10 MB (`tools.media.image.maxBytes`).
-   Audio por defecto: 20 MB (`tools.media.audio.maxBytes`).
-   Video por defecto: 50 MB (`tools.media.video.maxBytes`).
-   Los medios demasiado grandes omiten la comprensión, pero las respuestas aún se procesan con el cuerpo original.

## Notas para Pruebas

-   Cubrir flujos de envío + respuesta para casos de imagen/audio/documento.
-   Validar la recompresión para imágenes (límite de tamaño) y el indicador de nota de voz para audio.
-   Asegurar que las respuestas con múltiples medios se envíen secuencialmente.

[Comprensión de Medios](./media-understanding.md)[Audio y Notas de Voz](./audio.md)