title: "Guía de Firma de Aplicaciones macOS para Permisos Estables de OpenClaw"
description: "Aprende a firmar la aplicación macOS de OpenClaw para permisos TCC estables. Configura identidades de firma, maneja builds ad-hoc y gestiona identificadores de paquete."
keywords: ["firma macos", "codesign", "permisos tcc", "identificador de paquete", "developer id", "firma ad-hoc", "hardened runtime", "aplicación macos"]
---

  Aplicación complementaria para macOS

  
# Firma en macOS

Esta aplicación generalmente se construye desde [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh), que ahora:

-   establece un identificador de paquete de depuración estable: `ai.openclaw.mac.debug`
-   escribe el Info.plist con ese identificador de paquete (se puede sobrescribir con `BUNDLE_ID=...`)
-   llama a [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) para firmar el binario principal y el paquete de la aplicación, de modo que macOS trate cada reconstrucción como el mismo paquete firmado y mantenga los permisos TCC (notificaciones, accesibilidad, grabación de pantalla, micrófono, voz). Para permisos estables, usa una identidad de firma real; la firma ad-hoc es opcional y frágil (ver [permisos de macOS](./permissions.md)).
-   usa `CODESIGN_TIMESTAMP=auto` por defecto; habilita marcas de tiempo confiables para firmas Developer ID. Establece `CODESIGN_TIMESTAMP=off` para omitir las marcas de tiempo (builds de depuración sin conexión).
-   inyecta metadatos de compilación en Info.plist: `OpenClawBuildTimestamp` (UTC) y `OpenClawGitCommit` (hash corto) para que el panel Acerca de pueda mostrar la compilación, git y el canal de depuración/lanzamiento.
-   **El empaquetado requiere Node 22+**: el script ejecuta compilaciones TS y la compilación de la Interfaz de Usuario de Control.
-   lee `SIGN_IDENTITY` del entorno. Agrega `export SIGN_IDENTITY="Apple Development: Tu Nombre (TEAMID)"` (o tu certificado Developer ID Application) a tu shell rc para firmar siempre con tu certificado. La firma ad-hoc requiere una aceptación explícita mediante `ALLOW_ADHOC_SIGNING=1` o `SIGN_IDENTITY="-"` (no recomendado para pruebas de permisos).
-   ejecuta una auditoría del Team ID después de firmar y falla si algún Mach-O dentro del paquete de la aplicación está firmado por un Team ID diferente. Establece `SKIP_TEAM_ID_CHECK=1` para omitirla.

## Uso

```bash
# desde la raíz del repositorio
scripts/package-mac-app.sh               # selecciona identidad automáticamente; error si no se encuentra ninguna
SIGN_IDENTITY="Developer ID Application: Tu Nombre" scripts/package-mac-app.sh   # certificado real
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (los permisos no persistirán)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # ad-hoc explícita (misma advertencia)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # solución alternativa para discrepancia de Team ID de Sparkle (solo desarrollo)
```

### Nota sobre Firma Ad-hoc

Al firmar con `SIGN_IDENTITY="-"` (ad-hoc), el script deshabilita automáticamente el **Hardened Runtime** (`--options runtime`). Esto es necesario para evitar fallos cuando la aplicación intenta cargar frameworks embebidos (como Sparkle) que no comparten el mismo Team ID. Las firmas ad-hoc también rompen la persistencia de permisos TCC; consulta [permisos de macOS](./permissions.md) para los pasos de recuperación.

## Metadatos de compilación para Acerca de

`package-mac-app.sh` marca el paquete con:

-   `OpenClawBuildTimestamp`: ISO8601 UTC en el momento del empaquetado
-   `OpenClawGitCommit`: hash corto de git (o `unknown` si no está disponible)

La pestaña Acerca de lee estas claves para mostrar la versión, fecha de compilación, commit de git y si es una compilación de depuración (mediante `#if DEBUG`). Ejecuta el empaquetador para actualizar estos valores después de cambios en el código.

## Por qué

Los permisos TCC están vinculados al identificador de paquete *y* a la firma de código. Las compilaciones de depuración sin firmar con UUID cambiantes hacían que macOS olvidara las concesiones después de cada reconstrucción. Firmar los binarios (ad‑hoc por defecto) y mantener un identificador/ruta fijos (`dist/OpenClaw.app`) preserva las concesiones entre compilaciones, coincidiendo con el enfoque de VibeTunnel.

[Control Remoto](./remote.md)[Lanzamiento en macOS](./release.md)

---