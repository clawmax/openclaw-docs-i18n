

  Descripción general de plataformas

  
# Aplicación Android

## Resumen de compatibilidad

-   Rol: aplicación de nodo compañero (Android no aloja la Puerta de enlace).
-   Puerta de enlace requerida: sí (ejecútala en macOS, Linux o Windows vía WSL2).
-   Instalación: [Primeros pasos](../start/getting-started.md) + [Emparejamiento](../channels/pairing.md).
-   Puerta de enlace: [Manual de operaciones](../gateway.md) + [Configuración](../gateway/configuration.md).
    -   Protocolos: [Protocolo de puerta de enlace](../gateway/protocol.md) (nodos + plano de control).

## Control del sistema

El control del sistema (launchd/systemd) reside en el host de la Puerta de enlace. Consulta [Puerta de enlace](../gateway.md).

## Manual de conexión

Aplicación de nodo Android ⇄ (mDNS/NSD + WebSocket) ⇄ **Puerta de enlace** Android se conecta directamente al WebSocket de la Puerta de enlace (por defecto `ws://:18789`) y usa emparejamiento de dispositivos (`role: node`).

### Prerrequisitos

-   Puedes ejecutar la Puerta de enlace en la máquina "maestra".
-   El dispositivo/emulador Android puede alcanzar el WebSocket de la puerta de enlace:
    -   Misma LAN con mDNS/NSD, **o**
    -   Misma red tailnet de Tailscale usando Wide-Area Bonjour / DNS-SD unicast (ver abajo), **o**
    -   Host/puerto manual de la puerta de enlace (modo de respaldo)
-   Puedes ejecutar la CLI (`openclaw`) en la máquina de la puerta de enlace (o vía SSH).

### 1) Iniciar la Puerta de enlace

```bash
openclaw gateway --port 18789 --verbose
```

Confirma en los registros que ves algo como:

-   `listening on ws://0.0.0.0:18789`

Para configuraciones solo tailnet (recomendado para Viena ⇄ Londres), vincula la puerta de enlace a la IP de la tailnet:

-   Establece `gateway.bind: "tailnet"` en `~/.openclaw/openclaw.json` en el host de la puerta de enlace.
-   Reinicia la Puerta de enlace / la aplicación de la barra de menús de macOS.

### 2) Verificar el descubrimiento (opcional)

Desde la máquina de la puerta de enlace:

```bash
dns-sd -B _openclaw-gw._tcp local.
```

Más notas de depuración: [Bonjour](../gateway/bonjour.md).

#### Descubrimiento en Tailnet (Viena ⇄ Londres) vía DNS-SD unicast

El descubrimiento NSD/mDNS de Android no cruza redes. Si tu nodo Android y la puerta de enlace están en redes diferentes pero conectadas vía Tailscale, usa Wide-Area Bonjour / DNS-SD unicast en su lugar:

1.  Configura una zona DNS-SD (ejemplo `openclaw.internal.`) en el host de la puerta de enlace y publica registros `_openclaw-gw._tcp`.
2.  Configura DNS dividido de Tailscale para tu dominio elegido apuntando a ese servidor DNS.

Detalles y configuración de ejemplo para CoreDNS: [Bonjour](../gateway/bonjour.md).

### 3) Conectar desde Android

En la aplicación Android:

-   La aplicación mantiene su conexión a la puerta de enlace viva mediante un **servicio en primer plano** (notificación persistente).
-   Abre la pestaña **Conectar**.
-   Usa el **Código de configuración** o el modo **Manual**.
-   Si el descubrimiento está bloqueado, usa host/puerto manual (y TLS/token/contraseña cuando sea necesario) en los **Controles avanzados**.

Después del primer emparejamiento exitoso, Android se reconecta automáticamente al iniciar:

-   Punto final manual (si está habilitado), de lo contrario
-   La última puerta de enlace descubierta (mejor esfuerzo).

### 4) Aprobar el emparejamiento (CLI)

En la máquina de la puerta de enlace:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

Detalles del emparejamiento: [Emparejamiento](../channels/pairing.md).

### 5) Verificar que el nodo está conectado

-   Vía estado de nodos:
    
    Copiar
    
    ```bash
    openclaw nodes status
    ```
    
-   Vía Puerta de enlace:
    
    Copiar
    
    ```bash
    openclaw gateway call node.list --params "{}"
    ```
    

### 6) Chat + historial

La pestaña Chat de Android admite selección de sesión (por defecto `main`, más otras sesiones existentes):

-   Historial: `chat.history`
-   Enviar: `chat.send`
-   Actualizaciones push (mejor esfuerzo): `chat.subscribe` → `event:"chat"`

### 7) Lienzo + pantalla + cámara

#### Host de Lienzo de la Puerta de enlace (recomendado para contenido web)

Si quieres que el nodo muestre HTML/CSS/JS real que el agente pueda editar en disco, apunta el nodo al host de lienzo de la Puerta de enlace. Nota: los nodos cargan el lienzo desde el servidor HTTP de la Puerta de enlace (mismo puerto que `gateway.port`, por defecto `18789`).

1.  Crea `~/.openclaw/workspace/canvas/index.html` en el host de la puerta de enlace.
2.  Navega el nodo hacia él (LAN):

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18789/__openclaw__/canvas/"}'
```

Tailnet (opcional): si ambos dispositivos están en Tailscale, usa un nombre MagicDNS o la IP de la tailnet en lugar de `.local`, ej. `http://<gateway-magicdns>:18789/__openclaw__/canvas/`. Este servidor inyecta un cliente de recarga en vivo en el HTML y recarga cuando cambian los archivos. El host A2UI está en `http://<gateway-host>:18789/__openclaw__/a2ui/`. Comandos de lienzo (solo en primer plano):

-   `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (usa `{"url":""}` o `{"url":"/"}` para volver al andamio predeterminado). `canvas.snapshot` devuelve `{ format, base64 }` (por defecto `format="jpeg"`).
-   A2UI: `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL` alias heredado)

Comandos de cámara (solo en primer plano; sujeto a permisos):

-   `camera.snap` (jpg)
-   `camera.clip` (mp4)

Consulta [Nodo de cámara](../nodes/camera.md) para parámetros y ayudas de CLI. Comandos de pantalla:

-   `screen.record` (mp4; solo en primer plano)

### 8) Voz + superficie de comandos expandida de Android

-   Voz: Android usa un flujo único de micrófono encendido/apagado en la pestaña Voz con captura de transcripción y reproducción TTS (ElevenLabs cuando está configurado, TTS del sistema como respaldo).
-   Los interruptores de activación por voz/modo de conversación están actualmente eliminados de la UX/runtime de Android.
-   Familias de comandos adicionales de Android (disponibilidad depende del dispositivo + permisos):
    -   `device.status`, `device.info`, `device.permissions`, `device.health`
    -   `notifications.list`, `notifications.actions`
    -   `photos.latest`
    -   `contacts.search`, `contacts.add`
    -   `calendar.events`, `calendar.add`
    -   `motion.activity`, `motion.pedometer`
    -   `app.update`

[Windows (WSL2)](./windows.md)[Aplicación iOS](./ios.md)