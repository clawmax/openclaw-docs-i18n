

  Medios y dispositivos

  
# Voice Wake

OpenClaw trata **las palabras de activación como una lista global única** propiedad de la **Puerta de Enlace (Gateway)**.

-   **No hay palabras de activación personalizadas por nodo**.
-   **Cualquier nodo/interfaz de usuario de la aplicación puede editar** la lista; los cambios son persistidos por la Puerta de Enlace y transmitidos a todos.
-   macOS e iOS mantienen interruptores locales de **Voice Wake activado/desactivado** (la experiencia de usuario local y los permisos difieren).
-   Android mantiene actualmente Voice Wake desactivado y utiliza un flujo manual del micrófono en la pestaña Voz.

## Almacenamiento (host de la Puerta de Enlace)

Las palabras de activación se almacenan en la máquina de la puerta de enlace en:

-   `~/.openclaw/settings/voicewake.json`

Estructura:

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```

## Protocolo

### Métodos

-   `voicewake.get` → `{ triggers: string[] }`
-   `voicewake.set` con parámetros `{ triggers: string[] }` → `{ triggers: string[] }`

Notas:

-   Los activadores se normalizan (se recortan espacios, se eliminan vacíos). Las listas vacías vuelven a los valores predeterminados.
-   Se aplican límites por seguridad (límites de cantidad/longitud).

### Eventos

-   `voicewake.changed` con carga útil `{ triggers: string[] }`

Quién lo recibe:

-   Todos los clientes WebSocket (aplicación macOS, WebChat, etc.)
-   Todos los nodos conectados (iOS/Android), y también al conectar un nodo como un envío inicial del "estado actual".

## Comportamiento del cliente

### Aplicación macOS

-   Utiliza la lista global para controlar los activadores de `VoiceWakeRuntime`.
-   Editar "Palabras de activación" en los ajustes de Voice Wake llama a `voicewake.set` y luego depende de la transmisión para mantener sincronizados a otros clientes.

### Nodo iOS

-   Utiliza la lista global para la detección de activadores de `VoiceWakeManager`.
-   Editar Palabras de Activación en Ajustes llama a `voicewake.set` (a través del WS de la Puerta de Enlace) y también mantiene receptiva la detección local de palabras de activación.

### Nodo Android

-   Voice Wake está actualmente desactivado en el tiempo de ejecución/Ajustes de Android.
-   La voz en Android utiliza la captura manual del micrófono en la pestaña Voz en lugar de activadores por palabra de activación.

[Modo Conversación](./talk.md)[Comando de Ubicación](./location-command.md)

---