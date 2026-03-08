

  Runtime de Node

  
# Node.js

OpenClaw requiere **Node 22 o más reciente**. El [script de instalación](../install.md#install-methods) detectará e instalará Node automáticamente — esta página es para cuando quieres configurar Node tú mismo y asegurarte de que todo esté conectado correctamente (versiones, PATH, instalaciones globales).

## Verifica tu versión

```bash
node -v
```

Si esto imprime `v22.x.x` o superior, estás listo. Si Node no está instalado o la versión es muy antigua, elige un método de instalación a continuación.

## Instalar Node

 

Los gestores de versiones te permiten cambiar entre versiones de Node fácilmente. Opciones populares:

-   [**fnm**](https://github.com/Schniz/fnm) — rápido, multiplataforma
-   [**nvm**](https://github.com/nvm-sh/nvm) — ampliamente usado en macOS/Linux
-   [**mise**](https://mise.jdx.dev/) — políglota (Node, Python, Ruby, etc.)

Ejemplo con fnm:

```bash
fnm install 22
fnm use 22
```

Asegúrate de que tu gestor de versiones esté inicializado en tu archivo de inicio del shell (`~/.zshrc` o `~/.bashrc`). Si no lo está, es posible que `openclaw` no se encuentre en nuevas sesiones de terminal porque el PATH no incluirá el directorio bin de Node.

## Solución de problemas

### openclaw: comando no encontrado

Esto casi siempre significa que el directorio bin global de npm no está en tu PATH. 

### Paso 1: Encuentra tu prefijo global de npm

```bash
npm prefix -g
```

### Paso 2: Verifica si está en tu PATH

```bash
echo "$PATH"
```

Busca `<npm-prefix>/bin` (macOS/Linux) o `<npm-prefix>` (Windows) en la salida.

### Paso 3: Agrégalo a tu archivo de inicio del shell

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

Agrega la salida de `npm prefix -g` a tu PATH del sistema a través de Configuración → Sistema → Variables de entorno.

### Errores de permisos en npm install -g (Linux)

Si ves errores `EACCES`, cambia el prefijo global de npm a un directorio escribible por el usuario:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Agrega la línea `export PATH=...` a tu `~/.bashrc` o `~/.zshrc` para hacerlo permanente.

[Diagnostics Flags](../diagnostics/flags.md)[Session Management Deep Dive](../reference/session-management-compaction.md)