

  Resumen de plataformas

  
# Aplicación Linux

El Gateway es totalmente compatible con Linux. **Node es el entorno de ejecución recomendado**. Bun no es recomendado para el Gateway (errores en WhatsApp/Telegram). Están planeadas aplicaciones complementarias nativas para Linux. Las contribuciones son bienvenidas si quieres ayudar a construir una.

## Ruta rápida para principiantes (VPS)

1.  Instala Node 22+
2.  `npm i -g openclaw@latest`
3.  `openclaw onboard --install-daemon`
4.  Desde tu portátil: `ssh -N -L 18789:127.0.0.1:18789 @`
5.  Abre `http://127.0.0.1:18789/` y pega tu token

Guía paso a paso para VPS: [exe.dev](../install/exe-dev.md)

## Instalar

-   [Primeros pasos](../start/getting-started.md)
-   [Instalación y actualizaciones](../install/updating.md)
-   Flujos opcionales: [Bun (experimental)](../install/bun.md), [Nix](../install/nix.md), [Docker](../install/docker.md)

## Gateway

-   [Manual del Gateway](../gateway.md)
-   [Configuración](../gateway/configuration.md)

## Instalación del servicio Gateway (CLI)

Usa uno de estos comandos:

```bash
openclaw onboard --install-daemon
```

O:

```bash
openclaw gateway install
```

O:

```bash
openclaw configure
```

Selecciona **Servicio Gateway** cuando se te solicite. Reparar/migrar:

```bash
openclaw doctor
```

## Control del sistema (unidad de usuario systemd)

OpenClaw instala un servicio de **usuario** de systemd por defecto. Usa un servicio de **sistema** para servidores compartidos o siempre encendidos. El ejemplo completo de unidad y la guía están en el [Manual del Gateway](../gateway.md). Configuración mínima: Crea `~/.config/systemd/user/openclaw-gateway[-].service`:

```ini
[Unit]
Description=OpenClaw Gateway (perfil: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Habilítalo:

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

[Aplicación macOS](./macos.md)[Windows (WSL2)](./windows.md)