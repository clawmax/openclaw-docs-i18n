

  Notas de la versión

  
# Lista de Verificación de Lanzamiento

Usa `pnpm` (Node 22+) desde la raíz del repositorio. Mantén el árbol de trabajo limpio antes de etiquetar/publicar.

## Disparador del operador

Cuando el operador diga "release", haz inmediatamente esta comprobación previa (no hagas preguntas adicionales a menos que estés bloqueado):

-   Lee este documento y `docs/platforms/mac/release.md`.
-   Carga las variables de entorno desde `~/.profile` y confirma que `SPARKLE_PRIVATE_KEY_FILE` + las variables de App Store Connect estén configuradas (SPARKLE\_PRIVATE\_KEY\_FILE debe estar en `~/.profile`).
-   Usa las claves de Sparkle desde `~/Library/CloudStorage/Dropbox/Backup/Sparkle` si es necesario.

1.  **Versión y metadatos**

-   [ ]  Incrementa la versión en `package.json` (ej., `2026.1.29`).
-   [ ]  Ejecuta `pnpm plugins:sync` para alinear las versiones de los paquetes de extensiones + los registros de cambios.
-   [ ]  Actualiza las cadenas de CLI/versión en [`src/version.ts`](https://github.com/openclaw/openclaw/blob/main/src/version.ts) y el agente de usuario de Baileys en [`src/web/session.ts`](https://github.com/openclaw/openclaw/blob/main/src/web/session.ts).
-   [ ]  Confirma los metadatos del paquete (nombre, descripción, repositorio, palabras clave, licencia) y que el mapa `bin` apunte a [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) para `openclaw`.
-   [ ]  Si las dependencias cambiaron, ejecuta `pnpm install` para que `pnpm-lock.yaml` esté actualizado.

2.  **Compilación y artefactos**

-   [ ]  Si los inputs de A2UI cambiaron, ejecuta `pnpm canvas:a2ui:bundle` y haz commit de cualquier actualización en [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js).
-   [ ]  `pnpm run build` (regenera `dist/`).
-   [ ]  Verifica que el paquete npm `files` incluya todas las carpetas `dist/*` requeridas (notablemente `dist/node-host/**` y `dist/acp/**` para node headless + CLI ACP).
-   [ ]  Confirma que `dist/build-info.json` existe e incluye el hash `commit` esperado (el banner de la CLI usa esto para las instalaciones de npm).
-   [ ]  Opcional: `npm pack --pack-destination /tmp` después de la compilación; inspecciona el contenido del tarball y tenlo a mano para el lanzamiento en GitHub (no lo hagas commit).

3.  **Registro de cambios y documentación**

-   [ ]  Actualiza `CHANGELOG.md` con los aspectos más destacados para el usuario (crea el archivo si falta); mantén las entradas estrictamente descendentes por versión.
-   [ ]  Asegúrate de que los ejemplos/banderas del README coincidan con el comportamiento actual de la CLI (notablemente nuevos comandos u opciones).

4.  **Validación**

-   [ ]  `pnpm build`
-   [ ]  `pnpm check`
-   [ ]  `pnpm test` (o `pnpm test:coverage` si necesitas salida de cobertura)
-   [ ]  `pnpm release:check` (verifica el contenido del paquete npm)
-   [ ]  `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (prueba de humo de instalación con Docker, ruta rápida; requerida antes del lanzamiento)
    -   Si se sabe que el lanzamiento inmediatamente anterior en npm está roto, configura `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` o `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` para el paso de preinstalación.
-   [ ]  (Opcional) Prueba de humo completa del instalador (añade cobertura sin root + CLI): `pnpm test:install:smoke`
-   [ ]  (Opcional) E2E del instalador (Docker, ejecuta `curl -fsSL https://openclaw.ai/install.sh | bash`, realiza la incorporación, luego ejecuta llamadas reales a la herramienta):
    -   `pnpm test:install:e2e:openai` (requiere `OPENAI_API_KEY`)
    -   `pnpm test:install:e2e:anthropic` (requiere `ANTHROPIC_API_KEY`)
    -   `pnpm test:install:e2e` (requiere ambas claves; ejecuta ambos proveedores)
-   [ ]  (Opcional) Verificación puntual de la pasarela web si tus cambios afectan las rutas de envío/recepción.

5.  **Aplicación macOS (Sparkle)**

-   [ ]  Compila + firma la aplicación macOS, luego comprímela para distribución.
-   [ ]  Genera el appcast de Sparkle (notas HTML vía [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) y actualiza `appcast.xml`.
-   [ ]  Mantén el zip de la aplicación (y el zip opcional de dSYM) listo para adjuntar al lanzamiento de GitHub.
-   [ ]  Sigue [Lanzamiento de macOS](../platforms/mac/release.md) para los comandos exactos y las variables de entorno requeridas.
    -   `APP_BUILD` debe ser numérico + monótono (sin `-beta`) para que Sparkle compare las versiones correctamente.
    -   Si se realiza notarización, usa el perfil de cadena de claves `openclaw-notary` creado a partir de las variables de entorno de la API de App Store Connect (ver [Lanzamiento de macOS](../platforms/mac/release.md)).

6.  **Publicar (npm)**

-   [ ]  Confirma que el estado de git esté limpio; haz commit y push según sea necesario.
-   [ ]  `npm login` (verifica 2FA) si es necesario.
-   [ ]  `npm publish --access public` (usa `--tag beta` para pre-lanzamientos).
-   [ ]  Verifica el registro: `npm view openclaw version`, `npm view openclaw dist-tags`, y `npx -y openclaw@X.Y.Z --version` (o `--help`).

### Solución de problemas (notas del lanzamiento 2.0.0-beta2)

-   **`npm pack`/`publish` se cuelga o produce un tarball enorme**: el paquete de la aplicación macOS en `dist/OpenClaw.app` (y los zips de lanzamiento) se incluyen en el paquete. Solución: lista blanca del contenido a publicar vía `package.json` `files` (incluye subdirectorios de dist, docs, skills; excluye paquetes de aplicación). Confirma con `npm pack --dry-run` que `dist/OpenClaw.app` no esté listado.
-   **Bucle web de autenticación de npm para dist-tags**: usa autenticación heredada para obtener un prompt de OTP:
    -   `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
-   **La verificación con `npx` falla con `ECOMPROMISED: Lock compromised`**: reintenta con una caché nueva:
    -   `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
-   **La etiqueta necesita reasignación después de una corrección tardía**: actualiza y empuja la etiqueta forzosamente, luego asegúrate de que los assets del lanzamiento de GitHub aún coincidan:
    -   `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7.  **Lanzamiento en GitHub + appcast**

-   [ ]  Etiqueta y empuja: `git tag vX.Y.Z && git push origin vX.Y.Z` (o `git push --tags`).
-   [ ]  Crea/actualiza el lanzamiento en GitHub para `vX.Y.Z` con el **título `openclaw X.Y.Z`** (no solo la etiqueta); el cuerpo debe incluir la sección **completa** del registro de cambios para esa versión (Aspectos destacados + Cambios + Correcciones), en línea (sin enlaces desnudos), y **no debe repetir el título dentro del cuerpo**.
-   [ ]  Adjunta artefactos: tarball de `npm pack` (opcional), `OpenClaw-X.Y.Z.zip`, y `OpenClaw-X.Y.Z.dSYM.zip` (si se generó).
-   [ ]  Haz commit del `appcast.xml` actualizado y empújalo (Sparkle se alimenta desde main).
-   [ ]  Desde un directorio temporal limpio (sin `package.json`), ejecuta `npx -y openclaw@X.Y.Z send --help` para confirmar que los puntos de entrada de instalación/CLI funcionan.
-   [ ]  Anuncia/comparte las notas de la versión.

## Ámbito de publicación de plugins (npm)

Solo publicamos **plugins npm existentes** bajo el ámbito `@openclaw/*`. Los plugins empaquetados que no están en npm permanecen **solo en el árbol de disco** (aún se envían en `extensions/**`). Proceso para derivar la lista:

1.  `npm search @openclaw --json` y captura los nombres de los paquetes.
2.  Compara con los nombres en `extensions/*/package.json`.
3.  Publica solo la **intersección** (ya están en npm).

Lista actual de plugins en npm (actualiza según sea necesario):

-   @openclaw/bluebubbles
-   @openclaw/diagnostics-otel
-   @openclaw/discord
-   @openclaw/feishu
-   @openclaw/lobster
-   @openclaw/matrix
-   @openclaw/msteams
-   @openclaw/nextcloud-talk
-   @openclaw/nostr
-   @openclaw/voice-call
-   @openclaw/zalo
-   @openclaw/zalouser

Las notas de la versión también deben destacar **nuevos plugins empaquetados opcionales** que **no están activados por defecto** (ejemplo: `tlon`).

[Créditos](./credits.md)[Pruebas](./test.md)