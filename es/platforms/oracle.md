

  Resumen de plataformas

  
# Oracle Cloud

## Objetivo

Ejecutar un OpenClaw Gateway persistente en el nivel gratuito **Always Free** ARM de Oracle Cloud. El nivel gratuito de Oracle puede ser una gran opción para OpenClaw (especialmente si ya tienes una cuenta de OCI), pero tiene sus contrapartidas:

-   Arquitectura ARM (la mayoría de las cosas funcionan, pero algunos binarios pueden ser solo para x86)
-   La capacidad y el registro pueden ser complicados

## Comparación de Costos (2026)

| Proveedor | Plan | Especificaciones | Precio/mes | Notas |
| --- | --- | --- | --- | --- |
| Oracle Cloud | Always Free ARM | hasta 4 OCPU, 24GB RAM | $0 | ARM, capacidad limitada |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | ~ $4 | Opción de pago más económica |
| DigitalOcean | Basic | 1 vCPU, 1GB RAM | $6 | Interfaz fácil, buena documentación |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | Muchas ubicaciones |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | Ahora parte de Akamai |

* * *

## Prerrequisitos

-   Cuenta de Oracle Cloud ([registro](https://www.oracle.com/cloud/free/)) — consulta la [guía de registro de la comunidad](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) si tienes problemas
-   Cuenta de Tailscale (gratuita en [tailscale.com](https://tailscale.com))
-   ~30 minutos

## 1) Crear una Instancia en OCI

1.  Inicia sesión en [Oracle Cloud Console](https://cloud.oracle.com/)
2.  Navega a **Compute → Instances → Create Instance**
3.  Configura:
    -   **Nombre:** `openclaw`
    -   **Imagen:** Ubuntu 24.04 (aarch64)
    -   **Forma:** `VM.Standard.A1.Flex` (Ampere ARM)
    -   **OCPUs:** 2 (o hasta 4)
    -   **Memoria:** 12 GB (o hasta 24 GB)
    -   **Volumen de arranque:** 50 GB (hasta 200 GB gratis)
    -   **Clave SSH:** Añade tu clave pública
4.  Haz clic en **Create**
5.  Anota la dirección IP pública

**Consejo:** Si la creación de la instancia falla con "Out of capacity", prueba un dominio de disponibilidad diferente o reinténtalo más tarde. La capacidad del nivel gratuito es limitada.

## 2) Conectar y Actualizar

```bash
# Conectar vía IP pública
ssh ubuntu@YOUR_PUBLIC_IP

# Actualizar el sistema
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**Nota:** `build-essential` es necesario para la compilación ARM de algunas dependencias.

## 3) Configurar Usuario y Nombre de Host

```bash
# Establecer nombre de host
sudo hostnamectl set-hostname openclaw

# Establecer contraseña para el usuario ubuntu
sudo passwd ubuntu

# Habilitar lingering (mantiene los servicios del usuario ejecutándose después del cierre de sesión)
sudo loginctl enable-linger ubuntu
```

## 4) Instalar Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

Esto habilita Tailscale SSH, por lo que puedes conectarte vía `ssh openclaw` desde cualquier dispositivo en tu tailnet — no se necesita IP pública. Verifica:

```bash
tailscale status
```

**De ahora en adelante, conéctate vía Tailscale:** `ssh ubuntu@openclaw` (o usa la IP de Tailscale).

## 5) Instalar OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
source ~/.bashrc
```

Cuando se te pregunte "How do you want to hatch your bot?", selecciona **"Do this later"**.

> Nota: Si encuentras problemas de compilación nativa para ARM, comienza con los paquetes del sistema (ej. `sudo apt install -y build-essential`) antes de recurrir a Homebrew.

## 6) Configurar Gateway (loopback + autenticación por token) y habilitar Tailscale Serve

Usa la autenticación por token como predeterminada. Es predecible y evita necesitar cualquier bandera "insecure auth" en la Interfaz de Control.

```bash
# Mantén el Gateway privado en la VM
openclaw config set gateway.bind loopback

# Requiere autenticación para el Gateway + Interfaz de Control
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Exponer a través de Tailscale Serve (HTTPS + acceso desde la tailnet)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

## 7) Verificar

```bash
# Comprobar versión
openclaw --version

# Comprobar estado del demonio
systemctl --user status openclaw-gateway

# Comprobar Tailscale Serve
tailscale serve status

# Probar respuesta local
curl http://localhost:18789
```

## 8) Bloquear la Seguridad de la VCN

Ahora que todo funciona, bloquea la VCN para denegar todo el tráfico excepto Tailscale. La Virtual Cloud Network de OCI actúa como un firewall en el borde de la red — el tráfico se bloquea antes de llegar a tu instancia.

1.  Ve a **Networking → Virtual Cloud Networks** en la Consola de OCI
2.  Haz clic en tu VCN → **Security Lists** → Default Security List
3.  **Elimina** todas las reglas de entrada excepto:
    -   `0.0.0.0/0 UDP 41641` (Tailscale)
4.  Mantén las reglas de salida predeterminadas (permitir toda la salida)

Esto bloquea SSH en el puerto 22, HTTP, HTTPS y todo lo demás en el borde de la red. De ahora en adelante, solo podrás conectarte vía Tailscale.

* * *

## Acceder a la Interfaz de Control

Desde cualquier dispositivo en tu red de Tailscale:

```
https://openclaw.<tailnet-name>.ts.net/
```

Reemplaza `<tailnet-name>` con el nombre de tu tailnet (visible en `tailscale status`). No se necesita túnel SSH. Tailscale proporciona:

-   Cifrado HTTPS (certificados automáticos)
-   Autenticación vía identidad de Tailscale
-   Acceso desde cualquier dispositivo en tu tailnet (portátil, teléfono, etc.)

* * *

## Seguridad: VCN + Tailscale (línea base recomendada)

Con la VCN bloqueada (solo UDP 41641 abierto) y el Gateway vinculado a loopback, obtienes una fuerte defensa en profundidad: el tráfico público se bloquea en el borde de la red, y el acceso de administración ocurre a través de tu tailnet. Esta configuración a menudo elimina la *necesidad* de reglas de firewall adicionales basadas en el host solo para detener la fuerza bruta SSH desde toda Internet — pero aún debes mantener el SO actualizado, ejecutar `openclaw security audit`, y verificar que no estás escuchando accidentalmente en interfaces públicas.

### Lo que ya está Protegido

| Paso Tradicional | ¿Necesario? | Por qué |
| --- | --- | --- |
| Firewall UFW | No | La VCN bloquea antes de que el tráfico llegue a la instancia |
| fail2ban | No | No hay fuerza bruta si el puerto 22 está bloqueado en la VCN |
| Endurecimiento de sshd | No | Tailscale SSH no usa sshd |
| Deshabilitar inicio de sesión root | No | Tailscale usa identidad de Tailscale, no usuarios del sistema |
| Autenticación solo por clave SSH | No | Tailscale autentica a través de tu tailnet |
| Endurecimiento IPv6 | Generalmente no | Depende de la configuración de tu VCN/subred; verifica qué está realmente asignado/expuesto |

### Aún Recomendado

-   **Permisos de credenciales:** `chmod 700 ~/.openclaw`
-   **Auditoría de seguridad:** `openclaw security audit`
-   **Actualizaciones del sistema:** `sudo apt update && sudo apt upgrade` regularmente
-   **Monitorear Tailscale:** Revisa los dispositivos en la [consola de administración de Tailscale](https://login.tailscale.com/admin)

### Verificar Postura de Seguridad

```bash
# Confirmar que no hay puertos públicos escuchando
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Verificar que Tailscale SSH está activo
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH activo"

# Opcional: deshabilitar sshd completamente
sudo systemctl disable --now ssh
```

* * *

## Plan de Respaldo: Túnel SSH

Si Tailscale Serve no funciona, usa un túnel SSH:

```bash
# Desde tu máquina local (vía Tailscale)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

Luego abre `http://localhost:18789`.

* * *

## Solución de Problemas

### La creación de la instancia falla ("Out of capacity")

Las instancias ARM de nivel gratuito son populares. Prueba:

-   Un dominio de disponibilidad diferente
-   Reintentar durante horas de menor actividad (madrugada)
-   Usar el filtro "Always Free" al seleccionar la forma

### Tailscale no se conecta

```bash
# Comprobar estado
sudo tailscale status

# Reautenticar
sudo tailscale up --ssh --hostname=openclaw --reset
```

### El Gateway no inicia

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

### No se puede acceder a la Interfaz de Control

```bash
# Verificar que Tailscale Serve está ejecutándose
tailscale serve status

# Comprobar que el gateway está escuchando
curl http://localhost:18789

# Reiniciar si es necesario
systemctl --user restart openclaw-gateway
```

### Problemas con binarios ARM

Algunas herramientas pueden no tener compilaciones para ARM. Comprueba:

```bash
uname -m  # Debería mostrar aarch64
```

La mayoría de los paquetes npm funcionan bien. Para binarios, busca versiones `linux-arm64` o `aarch64`.

* * *

## Persistencia

Todo el estado reside en:

-   `~/.openclaw/` — configuración, credenciales, datos de sesión
-   `~/.openclaw/workspace/` — espacio de trabajo (SOUL.md, memoria, artefactos)

Haz copias de seguridad periódicamente:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## Ver También

-   [Acceso remoto al Gateway](../gateway/remote.md) — otros patrones de acceso remoto
-   [Integración con Tailscale](../gateway/tailscale.md) — documentación completa de Tailscale
-   [Configuración del Gateway](../gateway/configuration.md) — todas las opciones de configuración
-   [Guía de DigitalOcean](./digitalocean.md) — si quieres una opción de pago + registro más fácil
-   [Guía de Hetzner](../install/hetzner.md) — alternativa basada en Docker

[DigitalOcean](./digitalocean.md)[Raspberry Pi](./raspberry-pi.md)