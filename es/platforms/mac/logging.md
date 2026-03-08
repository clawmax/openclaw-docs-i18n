

  Aplicación complementaria para macOS

  
# Registro en macOS

## Registro de archivo de diagnóstico rotativo (Panel de depuración)

OpenClaw dirige los registros de la aplicación de macOS a través de swift-log (registro unificado por defecto) y puede escribir un registro de archivo local y rotativo en el disco cuando necesites una captura duradera.

-   Nivel de detalle: **Panel de depuración → Registros → Registro de la aplicación → Nivel de detalle**
-   Habilitar: **Panel de depuración → Registros → Registro de la aplicación → “Escribir registro de diagnóstico rotativo (JSONL)”**
-   Ubicación: `~/Library/Logs/OpenClaw/diagnostics.jsonl` (rota automáticamente; los archivos antiguos tienen el sufijo `.1`, `.2`, …)
-   Borrar: **Panel de depuración → Registros → Registro de la aplicación → “Borrar”**

Notas:

-   Esto está **deshabilitado por defecto**. Habilítalo solo mientras depures activamente.
-   Trata el archivo como sensible; no lo compartas sin revisarlo.

## Datos privados en el registro unificado de macOS

El registro unificado redacta la mayoría de las cargas útiles a menos que un subsistema opte por `privacy -off`. Según el artículo de Peter sobre las [trampas de privacidad en el registro de macOS](https://steipete.me/posts/2025/logging-privacy-shenanigans) (2025), esto se controla mediante un archivo plist en `/Library/Preferences/Logging/Subsystems/` indexado por el nombre del subsistema. Solo las nuevas entradas de registro recogen la bandera, así que habilítala antes de reproducir un problema.

## Habilitar para OpenClaw (ai.openclaw)

-   Primero escribe el archivo plist en un archivo temporal, luego instálalo atómicamente como root:

```bash
cat <<'EOF' >/tmp/ai.openclaw.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/ai.openclaw.plist /Library/Preferences/Logging/Subsystems/ai.openclaw.plist
```

-   No se requiere reinicio; logd detecta el archivo rápidamente, pero solo las nuevas líneas de registro incluirán cargas útiles privadas.
-   Visualiza la salida más detallada con el asistente existente, por ejemplo: `./scripts/clawlog.sh --category WebChat --last 5m`.

## Deshabilitar después de la depuración

-   Elimina la anulación: `sudo rm /Library/Preferences/Logging/Subsystems/ai.openclaw.plist`.
-   Opcionalmente, ejecuta `sudo log config --reload` para forzar a logd a descartar la anulación inmediatamente.
-   Recuerda que esta superficie puede incluir números de teléfono y cuerpos de mensajes; mantén el archivo plist en su lugar solo mientras necesites activamente el detalle adicional.

[Icono de la Barra de Menú](./icon.md)[Permisos de macOS](./permissions.md)

---