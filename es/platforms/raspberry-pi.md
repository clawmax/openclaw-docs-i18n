

  Descripción general de plataformas

  
# Raspberry Pi

## Objetivo

Ejecutar un Gateway de OpenClaw persistente y siempre activo en una Raspberry Pi por un costo único de **~$35-80** (sin tarifas mensuales). Perfecto para:

-   Asistente de IA personal 24/7
-   Centro de automatización del hogar
-   Bot de Telegram/WhatsApp de bajo consumo, siempre disponible

## Requisitos de Hardware

| Modelo de Pi | RAM | ¿Funciona? | Notas |
| --- | --- | --- | --- |
| **Pi 5** | 4GB/8GB | ✅ Mejor | El más rápido, recomendado |
| **Pi 4** | 4GB | ✅ Bueno | Punto óptimo para la mayoría de usuarios |
| **Pi 4** | 2GB | ✅ Aceptable | Funciona, añadir swap |
| **Pi 4** | 1GB | ⚠️ Ajustado | Posible con swap, configuración mínima |
| **Pi 3B+** | 1GB | ⚠️ Lento | Funciona pero es lento |
| **Pi Zero 2 W** | 512MB | ❌ | No recomendado |

**Especificaciones mínimas:** 1GB de RAM, 1 núcleo, 500MB de disco  
**Recomendado:** 2GB+ de RAM, SO de 64 bits, tarjeta SD de 16GB+ (o SSD USB)

## Lo que necesitarás

-   Raspberry Pi 4 o 5 (2GB+ recomendado)
-   Tarjeta MicroSD (16GB+) o SSD USB (mejor rendimiento)
-   Fuente de alimentación (se recomienda la PSU oficial de Pi)
-   Conexión de red (Ethernet o WiFi)
-   ~30 minutos

## 1) Grabar el Sistema Operativo

Usa **Raspberry Pi OS Lite (64-bit)** — no se necesita escritorio para un servidor sin cabeza (headless).

1.  Descarga [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2.  Elige el SO: **Raspberry Pi OS Lite (64-bit)**
3.  Haz clic en el icono de engranaje (⚙️) para preconfigurar:
    -   Establece el nombre de host: `gateway-host`
    -   Habilita SSH
    -   Establece nombre de usuario/contraseña
    -   Configura WiFi (si no usas Ethernet)
4.  Graba en tu tarjeta SD / unidad USB
5.  Insértala y arranca la Pi

## 2) Conectar vía SSH

```bash
ssh user@gateway-host
# o usa la dirección IP
ssh user@192.168.x.x
```

## 3) Configuración del Sistema

```bash
# Actualizar el sistema
sudo apt update && sudo apt upgrade -y

# Instalar paquetes esenciales
sudo apt install -y git curl build-essential

# Establecer zona horaria (importante para cron/recordatorios)
sudo timedatectl set-timezone America/Chicago  # Cámbialo a tu zona horaria
```

## 4) Instalar Node.js 22 (ARM64)

```bash
# Instalar Node.js vía NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Verificar
node --version  # Debería mostrar v22.x.x
npm --version
```

## 5) Añadir Swap (Importante para 2GB o menos)

El swap evita fallos por falta de memoria:

```bash
# Crear archivo de swap de 2GB
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Hacerlo permanente
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Optimizar para poca RAM (reducir swappiness)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## 6) Instalar OpenClaw

### Opción A: Instalación Estándar (Recomendada)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Opción B: Instalación Hackeable (Para experimentar)

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

La instalación hackeable te da acceso directo a los registros y al código — útil para depurar problemas específicos de ARM.

## 7) Ejecutar el Asistente de Configuración (Onboarding)

```bash
openclaw onboard --install-daemon
```

Sigue el asistente:

1.  **Modo del Gateway:** Local
2.  **Autenticación:** Se recomiendan claves API (OAuth puede ser problemático en una Pi sin cabeza)
3.  **Canales:** Telegram es el más fácil para empezar
4.  **Daemon:** Sí (systemd)

## 8) Verificar la Instalación

```bash
# Comprobar estado
openclaw status

# Comprobar servicio
sudo systemctl status openclaw

# Ver registros
journalctl -u openclaw -f
```

## 9) Acceder al Panel de Control

Dado que la Pi no tiene interfaz gráfica, usa un túnel SSH:

```bash
# Desde tu portátil/escritorio
ssh -L 18789:localhost:18789 user@gateway-host

# Luego abre en el navegador
open http://localhost:18789
```

O usa Tailscale para acceso siempre activo:

```bash
# En la Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Actualizar configuración
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

* * *

## Optimizaciones de Rendimiento

### Usar un SSD USB (Mejora Enorme)

Las tarjetas SD son lentas y se desgastan. Un SSD USB mejora drásticamente el rendimiento:

```bash
# Comprobar si arranca desde USB
lsblk
```

Consulta la [guía de arranque USB de Pi](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) para la configuración.

### Acelerar el inicio de la CLI (caché de compilación de módulos)

En hosts Pi de menor potencia, habilita la caché de compilación de módulos de Node para que las ejecuciones repetidas de la CLI sean más rápidas:

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF' # pragma: allowlist secret
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

Notas:

-   `NODE_COMPILE_CACHE` acelera las ejecuciones posteriores (`status`, `health`, `--help`).
-   `/var/tmp` sobrevive mejor a los reinicios que `/tmp`.
-   `OPENCLAW_NO_RESPAWN=1` evita el costo adicional del autoreinicio de la CLI.
-   La primera ejecución calienta la caché; las ejecuciones posteriores se benefician más.

### Ajuste de inicio de systemd (opcional)

Si esta Pi ejecuta principalmente OpenClaw, añade un "drop-in" de servicio para reducir la inestabilidad en los reinicios y mantener estable el entorno de inicio:

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

Luego aplica:

```bash
sudo systemctl daemon-reload
sudo systemctl restart openclaw
```

Si es posible, mantén el estado/caché de OpenClaw en almacenamiento respaldado por SSD para evitar cuellos de botella de E/S aleatoria de la tarjeta SD durante los arranques en frío. Cómo las políticas `Restart=` ayudan a la recuperación automática: [systemd puede automatizar la recuperación de servicios](https://www.redhat.com/en/blog/systemd-automate-recovery).

### Reducir el Uso de Memoria

```bash
# Deshabilitar asignación de memoria de GPU (sin cabeza)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Deshabilitar Bluetooth si no se necesita
sudo systemctl disable bluetooth
```

### Monitorear Recursos

```bash
# Comprobar memoria
free -h

# Comprobar temperatura de la CPU
vcgencmd measure_temp

# Monitoreo en vivo
htop
```

* * *

## Notas Específicas para ARM

### Compatibilidad Binaria

La mayoría de las funciones de OpenClaw funcionan en ARM64, pero algunos binarios externos pueden necesitar compilaciones para ARM:

| Herramienta | Estado ARM64 | Notas |
| --- | --- | --- |
| Node.js | ✅ | Funciona perfectamente |
| WhatsApp (Baileys) | ✅ | JS puro, sin problemas |
| Telegram | ✅ | JS puro, sin problemas |
| gog (CLI de Gmail) | ⚠️ | Verificar si hay versión para ARM |
| Chromium (navegador) | ✅ | `sudo apt install chromium-browser` |

Si una skill falla, verifica si su binario tiene una compilación para ARM. Muchas herramientas de Go/Rust la tienen; otras no.

### 32 bits vs 64 bits

**Usa siempre un SO de 64 bits.** Node.js y muchas herramientas modernas lo requieren. Verifica con:

```bash
uname -m
# Debería mostrar: aarch64 (64-bit) no armv7l (32-bit)
```

* * *

## Configuración de Modelo Recomendada

Dado que la Pi es solo el Gateway (los modelos se ejecutan en la nube), usa modelos basados en API:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**No intentes ejecutar LLMs locales en una Pi** — incluso los modelos pequeños son demasiado lentos. Deja que Claude/GPT hagan el trabajo pesado.

* * *

## Inicio Automático al Arrancar

El asistente de configuración lo prepara, pero para verificar:

```bash
# Comprobar si el servicio está habilitado
sudo systemctl is-enabled openclaw

# Habilitar si no lo está
sudo systemctl enable openclaw

# Iniciar al arrancar
sudo systemctl start openclaw
```

* * *

## Solución de Problemas

### Falta de Memoria (OOM)

```bash
# Comprobar memoria
free -h

# Añadir más swap (ver Paso 5)
# O reducir servicios ejecutándose en la Pi
```

### Rendimiento Lento

-   Usa SSD USB en lugar de tarjeta SD
-   Deshabilita servicios no utilizados: `sudo systemctl disable cups bluetooth avahi-daemon`
-   Comprobar limitación de CPU: `vcgencmd get_throttled` (debería devolver `0x0`)

### El Servicio No Inicia

```bash
# Comprobar registros
journalctl -u openclaw --no-pager -n 100

# Solución común: recompilar
cd ~/openclaw  # si usas la instalación hackeable
npm run build
sudo systemctl restart openclaw
```

### Problemas con Binarios ARM

Si una skill falla con "exec format error":

1.  Verifica si el binario tiene una compilación para ARM64
2.  Intenta compilar desde el código fuente
3.  O usa un contenedor Docker con soporte ARM

### Caídas de WiFi

Para Pis sin cabeza en WiFi:

```bash
# Deshabilitar gestión de energía de WiFi
sudo iwconfig wlan0 power off

# Hacerlo permanente
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

* * *

## Comparación de Costos

| Configuración | Costo Único | Costo Mensual | Notas |
| --- | --- | --- | --- |
| **Pi 4 (2GB)** | ~$45 | $0 | \+ electricidad (~$5/año) |
| **Pi 4 (4GB)** | ~$55 | $0 | Recomendado |
| **Pi 5 (4GB)** | ~$60 | $0 | Mejor rendimiento |
| **Pi 5 (8GB)** | ~$80 | $0 | Excesivo pero a prueba de futuro |
| DigitalOcean | $0 | $6/mes | $72/año |
| Hetzner | $0 | €3.79/mes | ~$50/año |

**Punto de equilibrio:** Una Pi se paga sola en ~6-12 meses comparado con un VPS en la nube.

* * *

## Ver También

-   [Guía de Linux](./linux.md) — configuración general de Linux
-   [Guía de DigitalOcean](./digitalocean.md) — alternativa en la nube
-   [Guía de Hetzner](../install/hetzner.md) — configuración con Docker
-   [Tailscale](../gateway/tailscale.md) — acceso remoto
-   [Nodos](../nodes.md) — empareja tu portátil/teléfono con el gateway Pi

[Oracle Cloud](./oracle.md)[Configuración de Desarrollo en macOS](./mac/dev-setup.md)