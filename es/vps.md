

  Alojamiento y despliegue

  
# Alojamiento en VPS

Este centro enlaza a las guías de alojamiento/VPS soportadas y explica cómo funcionan los despliegues en la nube a alto nivel.

## Elige un proveedor

-   **Railway** (un clic + configuración en navegador): [Railway](./install/railway.md)
-   **Northflank** (un clic + configuración en navegador): [Northflank](./install/northflank.md)
-   **Oracle Cloud (Siempre Gratis)**: [Oracle](./platforms/oracle.md) — $0/mes (Siempre Gratis, ARM; la capacidad/registro puede ser complicado)
-   **Fly.io**: [Fly.io](./install/fly.md)
-   **Hetzner (Docker)**: [Hetzner](./install/hetzner.md)
-   **GCP (Compute Engine)**: [GCP](./install/gcp.md)
-   **exe.dev** (VM + proxy HTTPS): [exe.dev](./install/exe-dev.md)
-   **AWS (EC2/Lightsail/nivel gratuito)**: también funciona bien. Guía en video: [https://x.com/techfrenAJ/status/2014934471095812547](https://x.com/techfrenAJ/status/2014934471095812547)

## Cómo funcionan las configuraciones en la nube

-   La **Puerta de Enlace (Gateway) se ejecuta en el VPS** y posee el estado y el espacio de trabajo.
-   Te conectas desde tu portátil/teléfono a través de la **Interfaz de Control (Control UI)** o **Tailscale/SSH**.
-   Trata el VPS como la fuente de verdad y **haz copias de seguridad** del estado y el espacio de trabajo.
-   Seguridad por defecto: mantén la Puerta de Enlace en loopback y accede a ella a través de túnel SSH o Tailscale Serve. Si la enlazas a `lan`/`tailnet`, exige `gateway.auth.token` o `gateway.auth.password`.

Acceso remoto: [Puerta de Enlace remota](./gateway/remote.md)  
Centro de plataformas: [Plataformas](./platforms.md)

## Agente compartido de empresa en un VPS

Esta es una configuración válida cuando los usuarios están en un mismo límite de confianza (por ejemplo, un equipo de empresa) y el agente es solo para negocios.

-   Mantenlo en un entorno de ejecución dedicado (VPS/VM/contenedor + usuario/cuentas de SO dedicados).
-   No inicies sesión en ese entorno con cuentas personales de Apple/Google o perfiles personales de navegador/gestor de contraseñas.
-   Si los usuarios son adversarios entre sí, sepáralos por puerta de enlace/host/usuario del SO.

Detalles del modelo de seguridad: [Seguridad](./gateway/security.md)

## Usar nodos con un VPS

Puedes mantener la Puerta de Enlace en la nube y emparejar **nodos** en tus dispositivos locales (Mac/iOS/Android/sin interfaz). Los nodos proporcionan pantalla local/cámara/lienzo y capacidades `system.run` mientras la Puerta de Enlace permanece en la nube. Documentación: [Nodos](./nodes.md), [CLI de Nodos](./cli/nodes.md)

## Ajustes de arranque para VMs pequeñas y hosts ARM

Si los comandos CLI se sienten lentos en VMs de baja potencia (o hosts ARM), habilita la caché de compilación de módulos de Node:

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF'
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

-   `NODE_COMPILE_CACHE` mejora los tiempos de arranque de comandos repetidos.
-   `OPENCLAW_NO_RESPAWN=1` evita la sobrecarga adicional de arranque de una ruta de auto-reinicio.
-   La primera ejecución del comando calienta la caché; las ejecuciones posteriores son más rápidas.
-   Para detalles específicos de Raspberry Pi, consulta [Raspberry Pi](./platforms/raspberry-pi.md).

### Lista de verificación de ajustes systemd (opcional)

Para hosts VM que usan `systemd`, considera:

-   Añadir variables de entorno al servicio para una ruta de arranque estable:
    -   `OPENCLAW_NO_RESPAWN=1`
    -   `NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache`
-   Mantener el comportamiento de reinicio explícito:
    -   `Restart=always`
    -   `RestartSec=2`
    -   `TimeoutStartSec=90`
-   Preferir discos respaldados por SSD para rutas de estado/caché para reducir penalizaciones de arranque en frío por E/S aleatorias.

Ejemplo:

```bash
sudo systemctl edit openclaw
```

```ini
[Service]
Environment=OPENCLAW_NO_RESPAWN=1
Environment=NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
Restart=always
RestartSec=2
TimeoutStartSec=90
```

Cómo las políticas `Restart=` ayudan en la recuperación automatizada: [systemd puede automatizar la recuperación del servicio](https://www.redhat.com/en/blog/systemd-automate-recovery).

[Desinstalar](./install/uninstall.md)[Fly.io](./install/fly.md)

---