

  RPC y API

  
# Base de Datos de Modelos de Dispositivos

La aplicación complementaria de macOS muestra nombres de modelos de dispositivos Apple amigables en la interfaz de usuario de **Instancias** mapeando identificadores de modelos de Apple (por ejemplo, `iPad16,6`, `Mac16,6`) a nombres legibles por humanos. El mapeo se incluye como JSON en:

-   `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

## Fuente de datos

Actualmente incluimos el mapeo desde el repositorio con licencia MIT:

-   `kyle-seongwoo-jun/apple-device-identifiers`

Para mantener las compilaciones deterministas, los archivos JSON están fijados a commits específicos del repositorio fuente (registrados en `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`).

## Actualizando la base de datos

1.  Elige los commits fuente a los que quieres fijarte (uno para iOS, uno para macOS).
2.  Actualiza los hashes de los commits en `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`.
3.  Vuelve a descargar los archivos JSON, fijados a esos commits:

```
IOS_COMMIT="<commit sha for ios-device-identifiers.json>"
MAC_COMMIT="<commit sha for mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4.  Asegúrate de que `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` aún coincida con la fuente (reemplázalo si la licencia fuente cambia).
5.  Verifica que la aplicación de macOS se compile sin errores (sin advertencias):

```bash
swift build --package-path apps/macos
```

[Adaptadores RPC](./rpc.md)[AGENTS.md por defecto](./AGENTS.default.md)