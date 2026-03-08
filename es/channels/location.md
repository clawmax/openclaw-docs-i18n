

  Configuración

  
# Análisis de Ubicación del Canal

OpenClaw normaliza las ubicaciones compartidas desde canales de chat en:

-   texto legible añadido al cuerpo del mensaje entrante, y
-   campos estructurados en la carga útil de contexto de la respuesta automática.

Actualmente compatible:

-   **Telegram** (pines de ubicación + lugares + ubicaciones en vivo)
-   **WhatsApp** (locationMessage + liveLocationMessage)
-   **Matrix** (`m.location` con `geo_uri`)

## Formato de texto

Las ubicaciones se muestran como líneas amigables sin corchetes:

-   Pin:
    -   `📍 48.858844, 2.294351 ±12m`
-   Lugar con nombre:
    -   `📍 Torre Eiffel — Campo de Marte, París (48.858844, 2.294351 ±12m)`
-   Compartir en vivo:
    -   `🛰 Ubicación en vivo: 48.858844, 2.294351 ±12m`

Si el canal incluye un pie de foto/comentario, se añade en la siguiente línea:

```
📍 48.858844, 2.294351 ±12m
Encuentro aquí
```

## Campos de contexto

Cuando hay una ubicación presente, estos campos se añaden a `ctx`:

-   `LocationLat` (número)
-   `LocationLon` (número)
-   `LocationAccuracy` (número, metros; opcional)
-   `LocationName` (cadena; opcional)
-   `LocationAddress` (cadena; opcional)
-   `LocationSource` (`pin | place | live`)
-   `LocationIsLive` (booleano)

## Notas del canal

-   **Telegram**: los lugares se asignan a `LocationName/LocationAddress`; las ubicaciones en vivo usan `live_period`.
-   **WhatsApp**: `locationMessage.comment` y `liveLocationMessage.caption` se añaden como la línea del pie de foto.
-   **Matrix**: `geo_uri` se analiza como una ubicación de pin; se ignora la altitud y `LocationIsLive` siempre es falso.

[Enrutamiento de Canales](./channel-routing.md)[Solución de Problemas del Canal](./troubleshooting.md)