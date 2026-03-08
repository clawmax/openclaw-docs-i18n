

  Resumen de instalación

  
# Internos del Instalador

OpenClaw incluye tres scripts de instalación, servidos desde `openclaw.ai`.

| Script | Plataforma | Qué hace |
| --- | --- | --- |
| [`install.sh`](#installsh) | macOS / Linux / WSL | Instala Node si es necesario, instala OpenClaw vía npm (por defecto) o git, y puede ejecutar la configuración inicial. |
| [`install-cli.sh`](#install-clish) | macOS / Linux / WSL | Instala Node + OpenClaw en un prefijo local (`~/.openclaw`). No requiere root. |
| [`install.ps1`](#installps1) | Windows (PowerShell) | Instala Node si es necesario, instala OpenClaw vía npm (por defecto) o git, y puede ejecutar la configuración inicial. |

## Comandos rápidos

> **ℹ️** Si la instalación tiene éxito pero `openclaw` no se encuentra en una nueva terminal, consulta [Solución de problemas de Node.js](./node.md#troubleshooting).

* * *

## install.sh

> **💡** Recomendado para la mayoría de instalaciones interactivas en macOS/Linux/WSL.

### Flujo (install.sh)

### Paso 1: Detectar SO

Soporta macOS y Linux (incluyendo WSL). Si se detecta macOS, instala Homebrew si falta.

### Paso 2: Asegurar Node.js 22+

Verifica la versión de Node e instala Node 22 si es necesario (Homebrew en macOS, scripts de configuración de NodeSource en apt/dnf/yum de Linux).

### Paso 3: Asegurar Git

Instala Git si falta.

### Paso 4: Instalar OpenClaw

-   Método `npm` (por defecto): instalación global con npm
-   Método `git`: clona/actualiza el repositorio, instala dependencias con pnpm, construye, luego instala el wrapper en `~/.local/bin/openclaw`

### Paso 5: Tareas posteriores a la instalación

-   Ejecuta `openclaw doctor --non-interactive` en actualizaciones e instalaciones git (mejor esfuerzo)
-   Intenta la configuración inicial cuando es apropiado (TTY disponible, configuración inicial no deshabilitada, y las comprobaciones de bootstrap/config pasan)
-   Establece por defecto `SHARP_IGNORE_GLOBAL_LIBVIPS=1`

### Detección de checkout de código fuente

Si se ejecuta dentro de un checkout de OpenClaw (`package.json` + `pnpm-workspace.yaml`), el script ofrece:

-   usar el checkout (`git`), o
-   usar la instalación global (`npm`)

Si no hay TTY disponible y no se establece un método de instalación, por defecto usa `npm` y advierte. El script sale con código `2` para selección de método inválida o valores inválidos de `--install-method`.

### Ejemplos (install.sh)

| Bandera | Descripción |
| --- | --- |
| `--install-method npm\|git` | Elegir método de instalación (por defecto: `npm`). Alias: `--method` |
| `--npm` | Atajo para el método npm |
| `--git` | Atajo para el método git. Alias: `--github` |
| `--version <version\|dist-tag>` | Versión de npm o dist-tag (por defecto: `latest`) |
| `--beta` | Usar dist-tag beta si está disponible, si no, recurrir a `latest` |
| `--git-dir ` | Directorio del checkout (por defecto: `~/openclaw`). Alias: `--dir` |
| `--no-git-update` | Omitir `git pull` para un checkout existente |
| `--no-prompt` | Deshabilitar prompts |
| `--no-onboard` | Saltar configuración inicial |
| `--onboard` | Habilitar configuración inicial |
| `--dry-run` | Imprimir acciones sin aplicar cambios |
| `--verbose` | Habilitar salida de depuración (`set -x`, logs de nivel notice de npm) |
| `--help` | Mostrar uso (`-h`) |

| Variable | Descripción |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | Método de instalación |
| `OPENCLAW_VERSION=latest\|next\|` | Versión de npm o dist-tag |
| `OPENCLAW_BETA=0\|1` | Usar beta si está disponible |
| `OPENCLAW_GIT_DIR=` | Directorio del checkout |
| `OPENCLAW_GIT_UPDATE=0\|1` | Activar/desactivar actualizaciones git |
| `OPENCLAW_NO_PROMPT=1` | Deshabilitar prompts |
| `OPENCLAW_NO_ONBOARD=1` | Saltar configuración inicial |
| `OPENCLAW_DRY_RUN=1` | Modo de ejecución en seco |
| `OPENCLAW_VERBOSE=1` | Modo de depuración |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | Nivel de log de npm |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | Controlar comportamiento de sharp/libvips (por defecto: `1`) |

* * *

## install-cli.sh

> **ℹ️** Diseñado para entornos donde se quiere todo bajo un prefijo local (por defecto `~/.openclaw`) y sin dependencia de Node del sistema.

### Flujo (install-cli.sh)

### Paso 1: Instalar entorno de ejecución Node local

Descarga el tarball de Node (por defecto `22.22.0`) a `/tools/node-v` y verifica SHA-256.

### Paso 2: Asegurar Git

Si falta Git, intenta instalarlo vía apt/dnf/yum en Linux o Homebrew en macOS.

### Paso 3: Instalar OpenClaw bajo el prefijo

Instala con npm usando `--prefix `, luego escribe el wrapper en `/bin/openclaw`.

### Ejemplos (install-cli.sh)

| Bandera | Descripción |
| --- | --- |
| `--prefix ` | Prefijo de instalación (por defecto: `~/.openclaw`) |
| `--version ` | Versión o dist-tag de OpenClaw (por defecto: `latest`) |
| `--node-version ` | Versión de Node (por defecto: `22.22.0`) |
| `--json` | Emitir eventos NDJSON |
| `--onboard` | Ejecutar `openclaw onboard` después de la instalación |
| `--no-onboard` | Saltar configuración inicial (por defecto) |
| `--set-npm-prefix` | En Linux, forzar el prefijo de npm a `~/.npm-global` si el prefijo actual no es escribible |
| `--help` | Mostrar uso (`-h`) |

| Variable | Descripción |
| --- | --- |
| `OPENCLAW_PREFIX=` | Prefijo de instalación |
| `OPENCLAW_VERSION=` | Versión o dist-tag de OpenClaw |
| `OPENCLAW_NODE_VERSION=` | Versión de Node |
| `OPENCLAW_NO_ONBOARD=1` | Saltar configuración inicial |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | Nivel de log de npm |
| `OPENCLAW_GIT_DIR=` | Ruta de búsqueda para limpieza heredada (usada al eliminar el antiguo checkout de submodule `Peekaboo`) |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | Controlar comportamiento de sharp/libvips (por defecto: `1`) |

* * *

## install.ps1

### Flujo (install.ps1)

### Paso 1: Asegurar PowerShell + entorno Windows

Requiere PowerShell 5+.

### Paso 2: Asegurar Node.js 22+

Si falta, intenta instalarlo vía winget, luego Chocolatey, luego Scoop.

### Paso 3: Instalar OpenClaw

-   Método `npm` (por defecto): instalación global con npm usando el `-Tag` seleccionado
-   Método `git`: clona/actualiza el repositorio, instala/construye con pnpm, e instala el wrapper en `%USERPROFILE%\.local\bin\openclaw.cmd`

### Paso 4: Tareas posteriores a la instalación

Añade el directorio bin necesario al PATH del usuario cuando es posible, luego ejecuta `openclaw doctor --non-interactive` en actualizaciones e instalaciones git (mejor esfuerzo).

### Ejemplos (install.ps1)

| Bandera | Descripción |
| --- | --- |
| `-InstallMethod npm\|git` | Método de instalación (por defecto: `npm`) |
| `-Tag ` | Dist-tag de npm (por defecto: `latest`) |
| `-GitDir ` | Directorio del checkout (por defecto: `%USERPROFILE%\openclaw`) |
| `-NoOnboard` | Saltar configuración inicial |
| `-NoGitUpdate` | Omitir `git pull` |
| `-DryRun` | Solo imprimir acciones |

| Variable | Descripción |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | Método de instalación |
| `OPENCLAW_GIT_DIR=` | Directorio del checkout |
| `OPENCLAW_NO_ONBOARD=1` | Saltar configuración inicial |
| `OPENCLAW_GIT_UPDATE=0` | Deshabilitar git pull |
| `OPENCLAW_DRY_RUN=1` | Modo de ejecución en seco |

> **ℹ️** Si se usa `-InstallMethod git` y falta Git, el script sale e imprime el enlace de Git for Windows.

* * *

## CI y automatización

Usa banderas/variables de entorno no interactivas para ejecuciones predecibles.

* * *

## Solución de problemas

Git es requerido para el método de instalación `git`. Para instalaciones `npm`, Git aún se verifica/instala para evitar fallos `spawn git ENOENT` cuando las dependencias usan URLs de git.

Algunas configuraciones de Linux apuntan el prefijo global de npm a rutas propiedad de root. `install.sh` puede cambiar el prefijo a `~/.npm-global` y añadir exportaciones PATH a los archivos rc del shell (cuando esos archivos existen).

Los scripts establecen por defecto `SHARP_IGNORE_GLOBAL_LIBVIPS=1` para evitar que sharp se construya contra libvips del sistema. Para anular:

```
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

Instala Git for Windows, reabre PowerShell, vuelve a ejecutar el instalador.

Ejecuta `npm config get prefix` y añade ese directorio a tu PATH de usuario (no se necesita el sufijo `\bin` en Windows), luego reabre PowerShell.

`install.ps1` actualmente no expone un interruptor `-Verbose`. Usa el rastreo de PowerShell para diagnósticos a nivel de script:

```powershell
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

Generalmente es un problema de PATH. Consulta [Solución de problemas de Node.js](./node.md#troubleshooting).

[Instalar](../install.md)[Docker](./docker.md)