

  App complementaria para macOS

  
# Lanzamiento para macOS

Esta aplicación ahora incluye actualizaciones automáticas de Sparkle. Las compilaciones de lanzamiento deben estar firmadas con Developer ID, comprimidas en zip y publicadas con una entrada de appcast firmada.

## Requisitos previos

-   Certificado Developer ID Application instalado (ejemplo: `Developer ID Application:  ()`).
-   Ruta de la clave privada de Sparkle configurada en el entorno como `SPARKLE_PRIVATE_KEY_FILE` (ruta a tu clave privada ed25519 de Sparkle; la clave pública está integrada en Info.plist). Si falta, revisa `~/.profile`.
-   Credenciales de notarización (perfil de llavero o clave API) para `xcrun notarytool` si deseas distribución DMG/zip segura para Gatekeeper.
    -   Usamos un perfil de llavero llamado `openclaw-notary`, creado a partir de las variables de entorno de la clave API de App Store Connect en tu perfil de shell:
        -   `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
        -   `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
        -   `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
-   Dependencias de `pnpm` instaladas (`pnpm install --config.node-linker=hoisted`).
-   Las herramientas de Sparkle se obtienen automáticamente vía SwiftPM en `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` (`sign_update`, `generate_appcast`, etc.).

## Compilar y empaquetar

Notas:

-   `APP_BUILD` se asigna a `CFBundleVersion`/`sparkle:version`; mantenlo numérico y monótono (sin `-beta`), o Sparkle lo comparará como igual.
-   Si se omite `APP_BUILD`, `scripts/package-mac-app.sh` deriva un valor predeterminado seguro para Sparkle a partir de `APP_VERSION` (`YYYYMMDDNN`: las versiones estables usan `90` por defecto, las pre-lanzamientos usan un canal derivado del sufijo) y usa el valor más alto entre ese y el conteo de commits de git.
-   Aún puedes anular `APP_BUILD` explícitamente cuando la ingeniería de lanzamiento necesite un valor monótono específico.
-   Por defecto usa la arquitectura actual (`$(uname -m)`). Para compilaciones universales/de lanzamiento, establece `BUILD_ARCHS="arm64 x86_64"` (o `BUILD_ARCHS=all`).
-   Usa `scripts/package-mac-dist.sh` para artefactos de lanzamiento (zip + DMG + notarización). Usa `scripts/package-mac-app.sh` para empaquetado local/de desarrollo.

```bash
# Desde la raíz del repositorio; establece los IDs de lanzamiento para que el feed de Sparkle esté habilitado.
# APP_BUILD debe ser numérico y monótono para la comparación de Sparkle.
# El valor predeterminado se deriva automáticamente de APP_VERSION cuando se omite.
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# Zip para distribución (incluye resource forks para soporte de delta de Sparkle)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.3.7.zip

# Opcional: también crea un DMG con estilo para humanos (arrastrar a /Applications)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.3.7.dmg

# Recomendado: compilar + notarizar/grapillar zip + DMG
# Primero, crea un perfil de llavero una vez:
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# Opcional: enviar dSYM junto con el lanzamiento
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.3.7.dSYM.zip
```

## Entrada en el Appcast

Usa el generador de notas de lanzamiento para que Sparkle renderice notas HTML formateadas:

```
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.3.7.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

Genera notas de lanzamiento en HTML a partir de `CHANGELOG.md` (vía [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)) y las incrusta en la entrada del appcast. Confirma el `appcast.xml` actualizado junto con los activos de lanzamiento (zip + dSYM) al publicar.

## Publicar y verificar

-   Sube `OpenClaw-2026.3.7.zip` (y `OpenClaw-2026.3.7.dSYM.zip`) al lanzamiento de GitHub para la etiqueta `v2026.3.7`.
-   Asegúrate de que la URL del appcast en bruto coincida con el feed integrado: `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
-   Comprobaciones de sentido común:
    -   `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` devuelve 200.
    -   `curl -I ` devuelve 200 después de subir los activos.
    -   En una compilación pública anterior, ejecuta "Buscar actualizaciones…" desde la pestaña Acerca de y verifica que Sparkle instale la nueva compilación sin problemas.

Definición de completado: la app firmada y el appcast están publicados, el flujo de actualización funciona desde una versión instalada anterior, y los activos de lanzamiento están adjuntos al lanzamiento de GitHub.

[Firma en macOS](./signing.md)[Gateway en macOS](./bundled-gateway.md)