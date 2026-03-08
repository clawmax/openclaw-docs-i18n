

  Resumen de instalación

  
# Instalar

¿Ya seguiste [Primeros pasos](./start/getting-started.md)? Ya estás listo — esta página es para métodos de instalación alternativos, instrucciones específicas de plataforma y mantenimiento.

## Requisitos del sistema

-   **[Node 22+](./install/node.md)** (el [script de instalación](#install-methods) lo instalará si falta)
-   macOS, Linux o Windows
-   `pnpm` solo si compilas desde el código fuente

> **ℹ️** En Windows, recomendamos encarecidamente ejecutar OpenClaw bajo [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install).

## Métodos de instalación

> **💡** El **script de instalación** es la forma recomendada de instalar OpenClaw. Maneja la detección de Node, la instalación y la configuración inicial en un solo paso.

 

> **⚠️** Para hosts VPS/en la nube, evita las imágenes de marketplace de terceros "con un clic" cuando sea posible. Prefiere una imagen base limpia del sistema operativo (por ejemplo, Ubuntu LTS), luego instala OpenClaw tú mismo con el script de instalación.

 

Descarga la CLI, la instala globalmente vía npm y lanza el asistente de configuración inicial.

Eso es todo — el script maneja la detección de Node, la instalación y la configuración inicial.Para omitir la configuración inicial y solo instalar el binario:

Para todas las banderas, variables de entorno y opciones de CI/automatización, consulta [Internals del instalador](./install/installer.md).

Si ya tienes Node 22+ y prefieres gestionar la instalación tú mismo:

Para colaboradores o cualquiera que quiera ejecutar desde una copia local.

1

[

](#)

Clonar y compilar

Clona el [repositorio de OpenClaw](https://github.com/openclaw/openclaw) y compila:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build
pnpm build
```

2

[

](#)

Enlazar la CLI

Haz que el comando `openclaw` esté disponible globalmente:

```bash
pnpm link --global
```

Alternativamente, omite el enlace y ejecuta comandos vía `pnpm openclaw ...` desde dentro del repositorio.

3

[

](#)

Ejecutar configuración inicial

```bash
openclaw onboard --install-daemon
```

Para flujos de trabajo de desarrollo más profundos, consulta [Configuración](./start/setup.md).

## Otros métodos de instalación

## Después de la instalación

Verifica que todo funcione:

```bash
openclaw doctor         # comprobar problemas de configuración
openclaw status         # estado del gateway
openclaw dashboard      # abrir la interfaz de usuario en el navegador
```

Si necesitas rutas de ejecución personalizadas, usa:

-   `OPENCLAW_HOME` para rutas internas basadas en el directorio de inicio
-   `OPENCLAW_STATE_DIR` para la ubicación del estado mutable
-   `OPENCLAW_CONFIG_PATH` para la ubicación del archivo de configuración

Consulta [Variables de entorno](./help/environment.md) para el orden de precedencia y detalles completos.

## Solución de problemas: openclaw no encontrado

Diagnóstico rápido:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Si `$(npm prefix -g)/bin` (macOS/Linux) o `$(npm prefix -g)` (Windows) **no** está en tu `$PATH`, tu shell no puede encontrar los binarios globales de npm (incluyendo `openclaw`).Solución — agrégalo a tu archivo de inicio del shell (`~/.zshrc` o `~/.bashrc`):

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

En Windows, agrega la salida de `npm prefix -g` a tu PATH.Luego abre una nueva terminal (o `rehash` en zsh / `hash -r` en bash).

## Actualizar / Desinstalar

[Internals del instalador](./install/installer.md)