

  Acceso remoto

  
# Configuración de Puerta de Enlace Remota

OpenClaw.app usa túneles SSH para conectarse a una puerta de enlace remota. Esta guía te muestra cómo configurarlo.

## Descripción General

## Configuración Rápida

### Paso 1: Añadir Configuración SSH

Edita `~/.ssh/config` y añade:

```
Host remote-gateway
    HostName <REMOTE_IP>          # e.g., 172.27.187.184
    User <REMOTE_USER>            # e.g., jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

Reemplaza `<REMOTE_IP>` y `<REMOTE_USER>` con tus valores.

### Paso 2: Copiar Clave SSH

Copia tu clave pública a la máquina remota (ingresa la contraseña una vez):

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```

### Paso 3: Establecer Token de la Puerta de Enlace

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```

### Paso 4: Iniciar Túnel SSH

```bash
ssh -N remote-gateway &
```

### Paso 5: Reiniciar OpenClaw.app

```bash
# Cierra OpenClaw.app (⌘Q), luego ábrelo de nuevo:
open /path/to/OpenClaw.app
```

La aplicación ahora se conectará a la puerta de enlace remota a través del túnel SSH.

* * *

## Inicio Automático del Túnel al Iniciar Sesión

Para que el túnel SSH se inicie automáticamente cuando inicias sesión, crea un Agente de Inicio (Launch Agent).

### Crear el archivo PLIST

Guarda esto como `~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>ai.openclaw.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

### Cargar el Agente de Inicio

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist
```

El túnel ahora:

-   Se iniciará automáticamente cuando inicies sesión
-   Se reiniciará si falla
-   Seguirá ejecutándose en segundo plano

Nota sobre versiones anteriores: elimina cualquier Agente de Inicio `com.openclaw.ssh-tunnel` que quede si está presente.

* * *

## Solución de Problemas

**Verificar si el túnel está en ejecución:**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**Reiniciar el túnel:**

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.ssh-tunnel
```

**Detener el túnel:**

```bash
launchctl bootout gui/$UID/ai.openclaw.ssh-tunnel
```

* * *

## Cómo Funciona

| Componente | Qué Hace |
| --- | --- |
| `LocalForward 18789 127.0.0.1:18789` | Reenvía el puerto local 18789 al puerto remoto 18789 |
| `ssh -N` | SSH sin ejecutar comandos remotos (solo reenvío de puertos) |
| `KeepAlive` | Reinicia automáticamente el túnel si falla |
| `RunAtLoad` | Inicia el túnel cuando se carga el agente |

OpenClaw.app se conecta a `ws://127.0.0.1:18789` en tu máquina cliente. El túnel SSH reenvía esa conexión al puerto 18789 en la máquina remota donde se ejecuta la Puerta de Enlace.

[Acceso Remoto](./remote.md)[Tailscale](./tailscale.md)

---