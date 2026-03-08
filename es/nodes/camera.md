

  Medios y dispositivos

  
# Captura de Cámara

OpenClaw soporta **captura de cámara** para flujos de trabajo de agentes:

-   **Nodo iOS** (emparejado vía Gateway): captura una **foto** (`jpg`) o **clip de video corto** (`mp4`, con audio opcional) vía `node.invoke`.
-   **Nodo Android** (emparejado vía Gateway): captura una **foto** (`jpg`) o **clip de video corto** (`mp4`, con audio opcional) vía `node.invoke`.
-   **Aplicación macOS** (nodo vía Gateway): captura una **foto** (`jpg`) o **clip de video corto** (`mp4`, con audio opcional) vía `node.invoke`.

Todo acceso a la cámara está controlado por **ajustes gestionados por el usuario**.

## Nodo iOS

### Ajuste de usuario (activado por defecto)

-   Pestaña Ajustes de iOS → **Cámara** → **Permitir Cámara** (`camera.enabled`)
    -   Por defecto: **activado** (la ausencia de la clave se trata como activado).
    -   Cuando está desactivado: los comandos `camera.*` devuelven `CAMERA_DISABLED`.

### Comandos (vía Gateway node.invoke)

-   `camera.list`
    -   Respuesta de carga útil:
        -   `devices`: array de `{ id, name, position, deviceType }`
-   `camera.snap`
    -   Parámetros:
        -   `facing`: `front|back` (por defecto: `front`)
        -   `maxWidth`: número (opcional; por defecto `1600` en el nodo iOS)
        -   `quality`: `0..1` (opcional; por defecto `0.9`)
        -   `format`: actualmente `jpg`
        -   `delayMs`: número (opcional; por defecto `0`)
        -   `deviceId`: string (opcional; de `camera.list`)
    -   Respuesta de carga útil:
        -   `format: "jpg"`
        -   `base64: "<...>"`
        -   `width`, `height`
    -   Protección de carga útil: las fotos se recomprimen para mantener la carga útil base64 por debajo de 5 MB.
-   `camera.clip`
    -   Parámetros:
        -   `facing`: `front|back` (por defecto: `front`)
        -   `durationMs`: número (por defecto `3000`, limitado a un máximo de `60000`)
        -   `includeAudio`: booleano (por defecto `true`)
        -   `format`: actualmente `mp4`
        -   `deviceId`: string (opcional; de `camera.list`)
    -   Respuesta de carga útil:
        -   `format: "mp4"`
        -   `base64: "<...>"`
        -   `durationMs`
        -   `hasAudio`

### Requisito de primer plano

Al igual que `canvas.*`, el nodo iOS solo permite comandos `camera.*` en **primer plano**. Las invocaciones en segundo plano devuelven `NODE_BACKGROUND_UNAVAILABLE`.

### Ayudante CLI (archivos temporales + MEDIA)

La forma más fácil de obtener adjuntos es mediante el ayudante CLI, que escribe los medios decodificados en un archivo temporal e imprime `MEDIA:`. Ejemplos:

```bash
openclaw nodes camera snap --node <id>               # por defecto: frontal + trasera (2 líneas MEDIA)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Notas:

-   `nodes camera snap` por defecto captura **ambas** orientaciones para dar al agente ambas vistas.
-   Los archivos de salida son temporales (en el directorio temporal del SO) a menos que construyas tu propio envoltorio.

## Nodo Android

### Ajuste de usuario en Android (activado por defecto)

-   Hoja de Ajustes de Android → **Cámara** → **Permitir Cámara** (`camera.enabled`)
    -   Por defecto: **activado** (la ausencia de la clave se trata como activado).
    -   Cuando está desactivado: los comandos `camera.*` devuelven `CAMERA_DISABLED`.

### Permisos

-   Android requiere permisos en tiempo de ejecución:
    -   `CAMERA` para `camera.snap` y `camera.clip`.
    -   `RECORD_AUDIO` para `camera.clip` cuando `includeAudio=true`.

Si faltan permisos, la aplicación solicitará cuando sea posible; si se deniegan, las solicitudes `camera.*` fallan con un error `*_PERMISSION_REQUIRED`.

### Requisito de primer plano en Android

Al igual que `canvas.*`, el nodo Android solo permite comandos `camera.*` en **primer plano**. Las invocaciones en segundo plano devuelven `NODE_BACKGROUND_UNAVAILABLE`.

### Comandos Android (vía Gateway node.invoke)

-   `camera.list`
    -   Respuesta de carga útil:
        -   `devices`: array de `{ id, name, position, deviceType }`

### Protección de carga útil

Las fotos se recomprimen para mantener la carga útil base64 por debajo de 5 MB.

## Aplicación macOS

### Ajuste de usuario (desactivado por defecto)

La aplicación complementaria de macOS expone una casilla de verificación:

-   **Ajustes → General → Permitir Cámara** (`openclaw.cameraEnabled`)
    -   Por defecto: **desactivado**
    -   Cuando está desactivado: las solicitudes de cámara devuelven "Cámara deshabilitada por el usuario".

### Ayudante CLI (node invoke)

Usa el CLI principal `openclaw` para invocar comandos de cámara en el nodo macOS. Ejemplos:

```bash
openclaw nodes camera list --node <id>            # listar ids de cámara
openclaw nodes camera snap --node <id>            # imprime MEDIA:<ruta>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # imprime MEDIA:<ruta>
openclaw nodes camera clip --node <id> --duration-ms 3000      # imprime MEDIA:<ruta> (bandera antigua)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Notas:

-   `openclaw nodes camera snap` por defecto usa `maxWidth=1600` a menos que se sobrescriba.
-   En macOS, `camera.snap` espera `delayMs` (por defecto 2000ms) después del calentamiento/estabilización de la exposición antes de capturar.
-   Las cargas útiles de fotos se recomprimen para mantener base64 por debajo de 5 MB.

## Seguridad + límites prácticos

-   El acceso a la cámara y al micrófono activa las solicitudes de permiso habituales del SO (y requiere cadenas de uso en Info.plist).
-   Los clips de video tienen un límite (actualmente `<= 60s`) para evitar cargas útiles de nodo demasiado grandes (sobrecarga base64 + límites de mensaje).

## Video de pantalla macOS (a nivel de SO)

Para video de *pantalla* (no de cámara), usa el complemento de macOS:

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # imprime MEDIA:<ruta>
```

Notas:

-   Requiere permiso de **Grabación de Pantalla** de macOS (TCC).

[Audio y Notas de Voz](./audio.md)[Modo Hablar](./talk.md)