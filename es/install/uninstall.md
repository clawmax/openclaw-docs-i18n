

  Mantenimiento

  
# Desinstalar

Dos caminos:

-   **Camino fácil** si `openclaw` aún está instalado.
-   **Eliminación manual del servicio** si la CLI ya no está pero el servicio sigue ejecutándose.

## Camino fácil (CLI aún instalada)

Recomendado: usar el desinstalador incorporado:

```bash
openclaw uninstall
```

No interactivo (automatización / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

Pasos manuales (mismo resultado):

1.  Detener el servicio del gateway:

```bash
openclaw gateway stop
```

2.  Desinstalar el servicio del gateway (launchd/systemd/schtasks):

```bash
openclaw gateway uninstall
```

3.  Eliminar estado + configuración:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

Si configuraste `OPENCLAW_CONFIG_PATH` en una ubicación personalizada fuera del directorio de estado, elimina ese archivo también.

4.  Eliminar tu espacio de trabajo (opcional, elimina archivos del agente):

```bash
rm -rf ~/.openclaw/workspace
```

5.  Remover la instalación de la CLI (elige el que usaste):

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6.  Si instalaste la aplicación de macOS:

```bash
rm -rf /Applications/OpenClaw.app
```

Notas:

-   Si usaste perfiles (`--profile` / `OPENCLAW_PROFILE`), repite el paso 3 para cada directorio de estado (los valores por defecto son `~/.openclaw-`).
-   En modo remoto, el directorio de estado reside en el **host del gateway**, así que ejecuta los pasos 1-4 allí también.

## Eliminación manual del servicio (CLI no instalada)

Usa esto si el servicio del gateway sigue ejecutándose pero `openclaw` no está.

### macOS (launchd)

La etiqueta por defecto es `ai.openclaw.gateway` (o `ai.openclaw.`; puede que aún existan las heredadas `com.openclaw.*`):

```bash
launchctl bootout gui/$UID/ai.openclaw.gateway
rm -f ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

Si usaste un perfil, reemplaza la etiqueta y el nombre del plist con `ai.openclaw.`. Elimina cualquier plist heredado `com.openclaw.*` si está presente.

### Linux (unidad de usuario systemd)

El nombre de la unidad por defecto es `openclaw-gateway.service` (o `openclaw-gateway-.service`):

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

### Windows (Tarea Programada)

El nombre de la tarea por defecto es `OpenClaw Gateway` (o `OpenClaw Gateway ()`). El script de la tarea reside bajo tu directorio de estado.

```bash
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

Si usaste un perfil, elimina el nombre de tarea correspondiente y `~\.openclaw-\gateway.cmd`.

## Instalación normal vs clonación del código fuente

### Instalación normal (install.sh / npm / pnpm / bun)

Si usaste `https://openclaw.ai/install.sh` o `install.ps1`, la CLI se instaló con `npm install -g openclaw@latest`. Remuévela con `npm rm -g openclaw` (o `pnpm remove -g` / `bun remove -g` si instalaste de esa manera).

### Clonación del código fuente (git clone)

Si ejecutas desde un clon del repositorio (`git clone` + `openclaw ...` / `bun run openclaw ...`):

1.  Desinstala el servicio del gateway **antes** de eliminar el repositorio (usa el camino fácil anterior o la eliminación manual del servicio).
2.  Elimina el directorio del repositorio.
3.  Elimina el estado + espacio de trabajo como se mostró arriba.

[Guía de Migración](./migrating.md)[Alojamiento VPS](../vps.md)

---