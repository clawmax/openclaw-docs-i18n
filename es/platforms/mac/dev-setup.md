

  Aplicación complementaria para macOS

  
# Configuración de Desarrollo para macOS

Esta guía cubre los pasos necesarios para compilar y ejecutar la aplicación OpenClaw para macOS desde el código fuente.

## Requisitos Previos

Antes de compilar la aplicación, asegúrate de tener instalado lo siguiente:

1.  **Xcode 26.2+**: Requerido para el desarrollo en Swift.
2.  **Node.js 22+ & pnpm**: Requerido para la puerta de enlace, la CLI y los scripts de empaquetado.

## 1\. Instalar Dependencias

Instala las dependencias del proyecto:

```bash
pnpm install
```

## 2\. Compilar y Empaquetar la Aplicación

Para compilar la aplicación de macOS y empaquetarla en `dist/OpenClaw.app`, ejecuta:

```
./scripts/package-mac-app.sh
```

Si no tienes un certificado de ID de Desarrollador de Apple, el script usará automáticamente la **firma ad-hoc** (`-`). Para modos de ejecución de desarrollo, banderas de firma y solución de problemas del Team ID, consulta el README de la aplicación de macOS: [https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md](https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md)

> **Nota**: Las aplicaciones firmadas ad-hoc pueden activar avisos de seguridad. Si la aplicación se cierra inmediatamente con “Abort trap 6”, consulta la sección [Solución de Problemas](#troubleshooting).

## 3\. Instalar la CLI

La aplicación de macOS espera una instalación global de la CLI `openclaw` para gestionar tareas en segundo plano. **Para instalarla (recomendado):**

1.  Abre la aplicación OpenClaw.
2.  Ve a la pestaña de ajustes **General**.
3.  Haz clic en **“Instalar CLI”**.

Alternativamente, instálala manualmente:

```bash
npm install -g openclaw@<version>
```

## Solución de Problemas

### La Compilación Falla: Incompatibilidad de Toolchain o SDK

La compilación de la aplicación de macOS espera el SDK más reciente de macOS y la toolchain de Swift 6.2. **Dependencias del sistema (requeridas):**

-   **Última versión de macOS disponible en Actualización de Software** (requerida por los SDKs de Xcode 26.2)
-   **Xcode 26.2** (toolchain de Swift 6.2)

**Comprobaciones:**

```bash
xcodebuild -version
xcrun swift --version
```

Si las versiones no coinciden, actualiza macOS/Xcode y vuelve a ejecutar la compilación.

### La Aplicación se Cierra al Otorgar Permiso

Si la aplicación se cierra cuando intentas permitir el acceso al **Reconocimiento de Voz** o al **Micrófono**, puede deberse a una caché TCC corrupta o a una discrepancia en la firma. **Solución:**

1.  Restablece los permisos TCC:
    
    Copiar
    
    ```bash
    tccutil reset All ai.openclaw.mac.debug
    ```
    
2.  Si eso falla, cambia el `BUNDLE_ID` temporalmente en [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) para forzar un "estado limpio" desde macOS.

### La Puerta de Enlace se queda en “Iniciando…” indefinidamente

Si el estado de la puerta de enlace permanece en “Iniciando…”, comprueba si un proceso zombi está ocupando el puerto:

```bash
openclaw gateway status
openclaw gateway stop

# Si no estás usando un LaunchAgent (modo desarrollo / ejecuciones manuales), encuentra el proceso que escucha:
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Si una ejecución manual está ocupando el puerto, detén ese proceso (Ctrl+C). Como último recurso, termina el PID que encontraste arriba.

[Raspberry Pi](../raspberry-pi.md)[Barra de Menú](./menu-bar.md)