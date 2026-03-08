

  Otros métodos de instalación

  
# Bun (Experimental)

Objetivo: ejecutar este repositorio con **Bun** (opcional, no recomendado para WhatsApp/Telegram) sin desviarse de los flujos de trabajo de pnpm. ⚠️ **No recomendado para el runtime de Gateway** (errores en WhatsApp/Telegram). Usa Node para producción.

## Estado

-   Bun es un runtime local opcional para ejecutar TypeScript directamente (`bun run …`, `bun --watch …`).
-   `pnpm` es el predeterminado para las compilaciones y sigue siendo totalmente compatible (y utilizado por algunas herramientas de documentación).
-   Bun no puede usar `pnpm-lock.yaml` y lo ignorará.

## Instalar

Por defecto:

```bash
bun install
```

Nota: `bun.lock`/`bun.lockb` están en .gitignore, por lo que no hay cambios en el repositorio de ninguna manera. Si no quieres *ninguna escritura de archivo de bloqueo*:

```bash
bun install --no-save
```

## Compilar / Probar (Bun)

```bash
bun run build
bun run vitest run
```

## Scripts de ciclo de vida de Bun (bloqueados por defecto)

Bun puede bloquear los scripts de ciclo de vida de las dependencias a menos que se confíe explícitamente en ellos (`bun pm untrusted` / `bun pm trust`). Para este repositorio, los scripts comúnmente bloqueados no son necesarios:

-   `@whiskeysockets/baileys` `preinstall`: verifica que Node sea mayor o igual a 20 (ejecutamos Node 22+).
-   `protobufjs` `postinstall`: emite advertencias sobre esquemas de versiones incompatibles (sin artefactos de compilación).

Si encuentras un problema real de ejecución que requiera estos scripts, confía en ellos explícitamente:

```bash
bun pm trust @whiskeysockets/baileys protobufjs
```

## Advertencias

-   Algunos scripts aún hacen referencia directa a pnpm (ej. `docs:build`, `ui:*`, `protocol:check`). Ejecuta esos con pnpm por ahora.

[Ansible](./ansible.md)[Actualizando](./updating.md)

---