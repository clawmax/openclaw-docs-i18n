

  Resumen de plataformas

  
# DigitalOcean

## Objetivo

Ejecutar un OpenClaw Gateway persistente en DigitalOcean por **$6/mes** (o $4/mes con precios reservados). Si quieres una opción de $0/mes y no te importa ARM + configuración específica del proveedor, consulta la [guía de Oracle Cloud](./oracle.md).

## Comparación de Costos (2026)

| Proveedor | Plan | Especificaciones | Precio/mes | Notas |
| --- | --- | --- | --- | --- |
| Oracle Cloud | Always Free ARM | hasta 4 OCPU, 24GB RAM | $0 | ARM, capacidad limitada / peculiaridades de registro |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | €3.79 (~$4) | Opción de pago más barata |
| DigitalOcean | Básico | 1 vCPU, 1GB RAM | $6 | Interfaz sencilla, buena documentación |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | Muchas ubicaciones |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | Ahora parte de Akamai |

**Elegir un proveedor:**

-   DigitalOcean: UX más simple + configuración predecible (esta guía)
-   Hetzner: buena relación precio/rendimiento (ver [guía de Hetzner](../install/hetzner.md))
-   Oracle Cloud: puede ser $0/mes, pero es más complicado y solo ARM (ver [guía de Oracle](./oracle.md))

* * *

## Prerrequisitos

-   Cuenta de DigitalOcean ([registro con $200 de crédito gratis](https://m.do.co/c/signup))
-   Par de claves SSH (o disposición a usar autenticación por contraseña)
-   ~20 minutos

## 1) Crear un Droplet

> **⚠️** Usa una imagen base limpia (Ubuntu 24.04 LTS). Evita las imágenes de 1 clic del Marketplace de terceros a menos que hayas revisado sus scripts de inicio y configuraciones predeterminadas del firewall.

1.  Inicia sesión en [DigitalOcean](https://cloud.digitalocean.com/)
2.  Haz clic en **Create → Droplets**
3.  Elige:
    -   **Región:** La más cercana a ti (o a tus usuarios)
    -   **Imagen:** Ubuntu 24.04 LTS
    -   **Tamaño:** Básico → Regular → **$6/mes** (1 vCPU, 1GB RAM, 25GB SSD)
    -   **Autenticación:** Clave SSH (recomendado) o contraseña
4.  Haz clic en **Create Droplet**
5.  Anota la dirección IP

## 2) Conectar vía SSH

```bash
ssh root@TU_IP_DEL_DROPLET
```

## 3) Instalar OpenClaw

```bash
# Actualizar sistema
apt update && apt upgrade -y

# Instalar Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Instalar OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# Verificar
openclaw --version
```

## 4) Ejecutar la Configuración Inicial

```bash
openclaw onboard --install-daemon
```

El asistente te guiará a través de:

-   Autenticación de modelos (claves API u OAuth)
-   Configuración de canales (Telegram, WhatsApp, Discord, etc.)
-   Token del gateway (generado automáticamente)
-   Instalación del daemon (systemd)

## 5) Verificar el Gateway

```bash
# Verificar estado
openclaw status

# Verificar servicio
systemctl --user status openclaw-gateway.service

# Ver registros
journalctl --user -u openclaw-gateway.service -f
```

## 6) Acceder al Panel de Control

El gateway se vincula al loopback por defecto. Para acceder a la Interfaz de Control: **Opción A: Túnel SSH (recomendado)**

```bash
# Desde tu máquina local
ssh -L 18789:localhost:18789 root@TU_IP_DEL_DROPLET

# Luego abre: http://localhost:18789
```

**Opción B: Tailscale Serve (HTTPS, solo loopback)**

```bash
# En el droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configurar el Gateway para usar Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

Abre: `https:///` Notas:

-   Serve mantiene el Gateway solo en loopback y autentica el tráfico de la Interfaz de Control/WebSocket mediante cabeceras de identidad de Tailscale (la autenticación sin token asume un host de gateway confiable; las APIs HTTP aún requieren token/contraseña).
-   Para requerir token/contraseña en su lugar, establece `gateway.auth.allowTailscale: false` o usa `gateway.auth.mode: "password"`.

**Opción C: Vinculación a la red Tailnet (sin Serve)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Abre: `http://<tailscale-ip>:18789` (se requiere token).

## 7) Conectar Tus Canales

### Telegram

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODIGO>
```

### WhatsApp

```bash
openclaw channels login whatsapp
# Escanear código QR
```

Consulta [Canales](../channels.md) para otros proveedores.

* * *

## Optimizaciones para 1GB de RAM

El droplet de $6 solo tiene 1GB de RAM. Para mantener todo funcionando sin problemas:

### Agregar swap (recomendado)

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### Usar un modelo más ligero

Si tienes problemas de falta de memoria (OOM), considera:

-   Usar modelos basados en API (Claude, GPT) en lugar de modelos locales
-   Establecer `agents.defaults.model.primary` en un modelo más pequeño

### Monitorear la memoria

```
free -h
htop
```

* * *

## Persistencia

Todo el estado reside en:

-   `~/.openclaw/` — configuración, credenciales, datos de sesión
-   `~/.openclaw/workspace/` — espacio de trabajo (SOUL.md, memoria, etc.)

Estos sobreviven a reinicios. Haz copias de seguridad periódicamente:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## Alternativa Gratuita de Oracle Cloud

Oracle Cloud ofrece instancias **Always Free** ARM que son significativamente más potentes que cualquier opción de pago aquí — por $0/mes.

| Lo que obtienes | Especificaciones |
| --- | --- |
| **4 OCPUs** | ARM Ampere A1 |
| **24GB RAM** | Más que suficiente |
| **200GB almacenamiento** | Volumen en bloque |
| **Gratis para siempre** | Sin cargos en tarjeta de crédito |

**Advertencias:**

-   El registro puede ser complicado (reintenta si falla)
-   Arquitectura ARM — la mayoría de las cosas funcionan, pero algunos binarios necesitan compilaciones para ARM

Para la guía de configuración completa, consulta [Oracle Cloud](./oracle.md). Para consejos de registro y solución de problemas del proceso de inscripción, consulta esta [guía de la comunidad](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd).

* * *

## Resolución de Problemas

### El Gateway no inicia

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

### Puerto ya en uso

```bash
lsof -i :18789
kill <PID>
```

### Falta de memoria

```bash
# Verificar memoria
free -h

# Agregar más swap
# O actualizar a un droplet de $12/mes (2GB RAM)
```

* * *

## Ver También

-   [Guía de Hetzner](../install/hetzner.md) — más barato, más potente
-   [Instalación con Docker](../install/docker.md) — configuración en contenedores
-   [Tailscale](../gateway/tailscale.md) — acceso remoto seguro
-   [Configuración](../gateway/configuration.md) — referencia completa de configuración

[iOS App](./ios.md)[Oracle Cloud](./oracle.md)