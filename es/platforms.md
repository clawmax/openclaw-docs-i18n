

  Resumen de plataformas

  
# Plataformas

El núcleo de OpenClaw está escrito en TypeScript. **Node es el entorno de ejecución recomendado**. Bun no es recomendado para el Gateway (errores en WhatsApp/Telegram). Existen aplicaciones compañeras para macOS (aplicación de barra de menú) y nodos móviles (iOS/Android). Las aplicaciones compañeras para Windows y Linux están planificadas, pero el Gateway está totalmente soportado hoy en día. También se planean aplicaciones compañeras nativas para Windows; se recomienda el Gateway a través de WSL2.

## Elige tu sistema operativo

-   macOS: [macOS](./platforms/macos.md)
-   iOS: [iOS](./platforms/ios.md)
-   Android: [Android](./platforms/android.md)
-   Windows: [Windows](./platforms/windows.md)
-   Linux: [Linux](./platforms/linux.md)

## VPS y alojamiento

-   Centro VPS: [Alojamiento VPS](./vps.md)
-   Fly.io: [Fly.io](./install/fly.md)
-   Hetzner (Docker): [Hetzner](./install/hetzner.md)
-   GCP (Compute Engine): [GCP](./install/gcp.md)
-   exe.dev (VM + proxy HTTPS): [exe.dev](./install/exe-dev.md)

## Enlaces comunes

-   Guía de instalación: [Primeros pasos](./start/getting-started.md)
-   Manual del Gateway: [Gateway](./gateway.md)
-   Configuración del Gateway: [Configuración](./gateway/configuration.md)
-   Estado del servicio: `openclaw gateway status`

## Instalación del servicio Gateway (CLI)

Usa uno de estos (todos soportados):

-   Asistente (recomendado): `openclaw onboard --install-daemon`
-   Directo: `openclaw gateway install`
-   Flujo de configuración: `openclaw configure` → selecciona **Servicio Gateway**
-   Reparar/migrar: `openclaw doctor` (ofrece instalar o reparar el servicio)

El objetivo del servicio depende del SO:

-   macOS: LaunchAgent (`ai.openclaw.gateway` o `ai.openclaw.`; legado `com.openclaw.*`)
-   Linux/WSL2: servicio de usuario systemd (`openclaw-gateway[-].service`)

[Aplicación macOS](./platforms/macos.md)

---