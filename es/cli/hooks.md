

  Comandos CLI

  
# hooks

Gestiona hooks del agente (automatizaciones dirigidas por eventos para comandos como `/new`, `/reset` y el inicio del gateway). Relacionado:

-   Hooks: [Hooks](../automation/hooks.md)
-   Hooks de plugins: [Plugins](../tools/plugin.md#plugin-hooks)

## Listar Todos los Hooks

```bash
openclaw hooks list
```

Lista todos los hooks descubiertos desde los directorios del espacio de trabajo, gestionados y empaquetados. **Opciones:**

-   `--eligible`: Muestra solo los hooks elegibles (requisitos cumplidos)
-   `--json`: Salida en formato JSON
-   `-v, --verbose`: Muestra información detallada incluyendo requisitos faltantes

**Ejemplo de salida:**

```
Hooks (4/4 listos)

Listos:
  🚀 boot-md ✓ - Ejecuta BOOT.md al iniciar el gateway
  📎 bootstrap-extra-files ✓ - Inyecta archivos de arranque adicionales del espacio de trabajo durante el arranque del agente
  📝 command-logger ✓ - Registra todos los eventos de comandos en un archivo de auditoría centralizado
  💾 session-memory ✓ - Guarda el contexto de la sesión en memoria cuando se emite el comando /new
```

**Ejemplo (verbose):**

```bash
openclaw hooks list --verbose
```

Muestra los requisitos faltantes para hooks no elegibles. **Ejemplo (JSON):**

```bash
openclaw hooks list --json
```

Devuelve JSON estructurado para uso programático.

## Obtener Información de un Hook

```bash
openclaw hooks info <name>
```

Muestra información detallada sobre un hook específico. **Argumentos:**

-   ``: Nombre del hook (ej., `session-memory`)

**Opciones:**

-   `--json`: Salida en formato JSON

**Ejemplo:**

```bash
openclaw hooks info session-memory
```

**Salida:**

```
💾 session-memory ✓ Listo

Guarda el contexto de la sesión en memoria cuando se emite el comando /new

Detalles:
  Fuente: openclaw-bundled
  Ruta: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Manejador: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Página de inicio: https://docs.openclaw.ai/automation/hooks#session-memory
  Eventos: command:new

Requisitos:
  Config: ✓ workspace.dir
```

## Verificar Elegibilidad de Hooks

```bash
openclaw hooks check
```

Muestra un resumen del estado de elegibilidad de los hooks (cuántos están listos vs. no listos). **Opciones:**

-   `--json`: Salida en formato JSON

**Ejemplo de salida:**

```
Estado de los Hooks

Total de hooks: 4
Listos: 4
No listos: 0
```

## Habilitar un Hook

```bash
openclaw hooks enable <name>
```

Habilita un hook específico agregándolo a tu configuración (`~/.openclaw/config.json`). **Nota:** Los hooks gestionados por plugins muestran `plugin:` en `openclaw hooks list` y no se pueden habilitar/deshabilitar aquí. En su lugar, habilita/deshabilita el plugin. **Argumentos:**

-   ``: Nombre del hook (ej., `session-memory`)

**Ejemplo:**

```bash
openclaw hooks enable session-memory
```

**Salida:**

```
✓ Hook habilitado: 💾 session-memory
```

**Lo que hace:**

-   Verifica si el hook existe y es elegible
-   Actualiza `hooks.internal.entries..enabled = true` en tu configuración
-   Guarda la configuración en disco

**Después de habilitar:**

-   Reinicia el gateway para que los hooks se recarguen (reinicio de la aplicación de la barra de menús en macOS, o reinicia tu proceso del gateway en desarrollo).

## Deshabilitar un Hook

```bash
openclaw hooks disable <name>
```

Deshabilita un hook específico actualizando tu configuración. **Argumentos:**

-   ``: Nombre del hook (ej., `command-logger`)

**Ejemplo:**

```bash
openclaw hooks disable command-logger
```

**Salida:**

```
⏸ Hook deshabilitado: 📝 command-logger
```

**Después de deshabilitar:**

-   Reinicia el gateway para que los hooks se recarguen

## Instalar Hooks

```bash
openclaw hooks install <path-or-spec>
openclaw hooks install <npm-spec> --pin
```

Instala un paquete de hooks desde una carpeta/archivo local o npm. Las especificaciones de npm son **solo del registro** (nombre del paquete + **versión exacta** opcional o **dist-tag**). Se rechazan las especificaciones Git/URL/archivo y los rangos semver. Las instalaciones de dependencias se ejecutan con `--ignore-scripts` por seguridad. Las especificaciones simples y `@latest` permanecen en la vía estable. Si npm resuelve cualquiera de esos a una versión preliminar, OpenClaw se detiene y te pide que aceptes explícitamente con una etiqueta preliminar como `@beta`/`@rc` o una versión preliminar exacta. **Lo que hace:**

-   Copia el paquete de hooks en `~/.openclaw/hooks/`
-   Habilita los hooks instalados en `hooks.internal.entries.*`
-   Registra la instalación en `hooks.internal.installs`

**Opciones:**

-   `-l, --link`: Enlaza un directorio local en lugar de copiarlo (lo agrega a `hooks.internal.load.extraDirs`)
-   `--pin`: Registra las instalaciones de npm como `name@version` exacta resuelta en `hooks.internal.installs`

**Archivos soportados:** `.zip`, `.tgz`, `.tar.gz`, `.tar` **Ejemplos:**

```bash
# Directorio local
openclaw hooks install ./my-hook-pack

# Archivo local
openclaw hooks install ./my-hook-pack.zip

# Paquete NPM
openclaw hooks install @openclaw/my-hook-pack

# Enlazar un directorio local sin copiar
openclaw hooks install -l ./my-hook-pack
```

## Actualizar Hooks

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Actualiza paquetes de hooks instalados (solo instalaciones de npm). **Opciones:**

-   `--all`: Actualiza todos los paquetes de hooks rastreados
-   `--dry-run`: Muestra lo que cambiaría sin escribir

Cuando existe un hash de integridad almacenado y el hash del artefacto obtenido cambia, OpenClaw imprime una advertencia y pide confirmación antes de proceder. Usa el global `--yes` para omitir las preguntas en ejecuciones CI/no interactivas.

## Hooks Empaquetados

### session-memory

Guarda el contexto de la sesión en memoria cuando emites `/new`. **Habilitar:**

```bash
openclaw hooks enable session-memory
```

**Salida:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md` **Ver:** [documentación de session-memory](../automation/hooks.md#session-memory)

### bootstrap-extra-files

Inyecta archivos de arranque adicionales (por ejemplo, `AGENTS.md` / `TOOLS.md` locales del monorepositorio) durante `agent:bootstrap`. **Habilitar:**

```bash
openclaw hooks enable bootstrap-extra-files
```

**Ver:** [documentación de bootstrap-extra-files](../automation/hooks.md#bootstrap-extra-files)

### command-logger

Registra todos los eventos de comandos en un archivo de auditoría centralizado. **Habilitar:**

```bash
openclaw hooks enable command-logger
```

**Salida:** `~/.openclaw/logs/commands.log` **Ver registros:**

```bash
# Comandos recientes
tail -n 20 ~/.openclaw/logs/commands.log

# Formato legible
cat ~/.openclaw/logs/commands.log | jq .

# Filtrar por acción
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Ver:** [documentación de command-logger](../automation/hooks.md#command-logger)

### boot-md

Ejecuta `BOOT.md` cuando el gateway inicia (después de que los canales comiencen). **Eventos**: `gateway:startup` **Habilitar**:

```bash
openclaw hooks enable boot-md
```

**Ver:** [documentación de boot-md](../automation/hooks.md#boot-md)

[health](./health.md)[logs](./logs.md)