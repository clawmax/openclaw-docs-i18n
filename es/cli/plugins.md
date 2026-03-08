

  Comandos CLI

  
# plugins

Gestiona plugins/extensiones del Gateway (cargados en proceso). Relacionado:

-   Sistema de plugins: [Plugins](../tools/plugin.md)
-   Manifiesto y esquema de plugin: [Manifiesto de plugin](../plugins/manifest.md)
-   Fortalecimiento de seguridad: [Seguridad](../gateway/security.md)

## Comandos

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Los plugins incluidos vienen con OpenClaw pero comienzan deshabilitados. Usa `plugins enable` para activarlos. Todos los plugins deben incluir un archivo `openclaw.plugin.json` con un esquema JSON en línea (`configSchema`, incluso si está vacío). Los manifiestos o esquemas faltantes/inválidos impiden que el plugin se cargue y fallan la validación de configuración.

### Instalar

```bash
openclaw plugins install <path-or-spec>
openclaw plugins install <npm-spec> --pin
```

Nota de seguridad: trata las instalaciones de plugins como ejecutar código. Prefiere versiones fijadas. Las especificaciones de npm son **solo del registro** (nombre del paquete + **versión exacta** opcional o **dist-tag**). Se rechazan las especificaciones de Git/URL/archivo y los rangos semver. Las instalaciones de dependencias se ejecutan con `--ignore-scripts` por seguridad. Las especificaciones simples y `@latest` permanecen en la rama estable. Si npm resuelve cualquiera de esos a una versión preliminar, OpenClaw se detiene y te pide que aceptes explícitamente con una etiqueta preliminar como `@beta`/`@rc` o una versión preliminar exacta como `@1.2.3-beta.4`. Si una especificación de instalación simple coincide con un id de plugin incluido (por ejemplo `diffs`), OpenClaw instala el plugin incluido directamente. Para instalar un paquete npm con el mismo nombre, usa una especificación con ámbito explícita (por ejemplo `@scope/diffs`). Archivos soportados: `.zip`, `.tgz`, `.tar.gz`, `.tar`. Usa `--link` para evitar copiar un directorio local (agrega a `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

Usa `--pin` en instalaciones de npm para guardar la especificación exacta resuelta (`name@version`) en `plugins.installs` mientras mantienes el comportamiento predeterminado sin fijar.

### Desinstalar

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` elimina los registros de plugin de `plugins.entries`, `plugins.installs`, la lista de permitidos de plugins y las entradas vinculadas en `plugins.load.paths` cuando corresponda. Para los plugins de memoria activos, la ranura de memoria se restablece a `memory-core`. Por defecto, desinstalar también elimina el directorio de instalación del plugin bajo la raíz de extensiones del directorio de estado activo (`$OPENCLAW_STATE_DIR/extensions/`). Usa `--keep-files` para mantener los archivos en el disco. `--keep-config` es compatible como un alias obsoleto para `--keep-files`.

### Actualizar

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Las actualizaciones solo se aplican a plugins instalados desde npm (rastreados en `plugins.installs`). Cuando existe un hash de integridad almacenado y el hash del artefacto obtenido cambia, OpenClaw imprime una advertencia y solicita confirmación antes de proceder. Usa el global `--yes` para omitir las solicitudes en ejecuciones CI/no interactivas.

[pairing](./pairing.md)[qr](./qr.md)

---