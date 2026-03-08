

  Navegador

  
# Resolución de Problemas del Navegador

## Problema: “Error al iniciar Chrome CDP en el puerto 18800”

El servidor de control del navegador de OpenClaw falla al iniciar Chrome/Brave/Edge/Chromium con el error:

```json
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```

### Causa Raíz

En Ubuntu (y muchas distribuciones de Linux), la instalación predeterminada de Chromium es un **paquete snap**. La confinación de AppArmor de Snap interfiere con la forma en que OpenClaw genera y monitorea el proceso del navegador. El comando `apt install chromium` instala un paquete stub que redirige a snap:

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

Esto NO es un navegador real — es solo un contenedor.

### Solución 1: Instalar Google Chrome (Recomendado)

Instala el paquete oficial `.deb` de Google Chrome, que no está confinado por snap:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # si hay errores de dependencias
```

Luego actualiza tu configuración de OpenClaw (`~/.openclaw/openclaw.json`):

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### Solución 2: Usar Chromium Snap con Modo de Solo Conexión (Attach-Only)

Si debes usar Chromium snap, configura OpenClaw para conectarse a un navegador iniciado manualmente:

1.  Actualiza la configuración:

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2.  Inicia Chromium manualmente:

```
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3.  Opcionalmente, crea un servicio de usuario systemd para iniciar Chrome automáticamente:

```bash
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Habilita con: `systemctl --user enable --now openclaw-browser.service`

### Verificar que el Navegador Funciona

Verifica el estado:

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Prueba la navegación:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

### Referencia de Configuración

| Opción | Descripción | Predeterminado |
| --- | --- | --- |
| `browser.enabled` | Habilitar control del navegador | `true` |
| `browser.executablePath` | Ruta al binario de un navegador basado en Chromium (Chrome/Brave/Edge/Chromium) | detectado automáticamente (prefiere el navegador predeterminado cuando es basado en Chromium) |
| `browser.headless` | Ejecutar sin interfaz gráfica | `false` |
| `browser.noSandbox` | Agregar flag `--no-sandbox` (necesario para algunas configuraciones de Linux) | `false` |
| `browser.attachOnly` | No iniciar navegador, solo conectarse a uno existente | `false` |
| `browser.cdpPort` | Puerto del Protocolo de Herramientas de Desarrollo de Chrome (CDP) | `18800` |

### Problema: “El relé de la extensión de Chrome está ejecutándose, pero no hay ninguna pestaña conectada”

Estás usando el perfil `chrome` (relé de extensión). Este espera que la extensión del navegador OpenClaw esté conectada a una pestaña activa. Opciones de solución:

1.  **Usar el navegador gestionado:** `openclaw browser start --browser-profile openclaw` (o establecer `browser.defaultProfile: "openclaw"`).
2.  **Usar el relé de extensión:** instala la extensión, abre una pestaña y haz clic en el icono de la extensión OpenClaw para conectarla.

Notas:

-   El perfil `chrome` usa tu **navegador Chromium predeterminado del sistema** cuando es posible.
-   Los perfiles locales `openclaw` asignan automáticamente `cdpPort`/`cdpUrl`; solo configura esos para CDP remoto.

[Extensión de Chrome](./chrome-extension.md)[Enviar Agente](./agent-send.md)