

  Otros métodos de instalación

  
# Ansible

La forma recomendada de implementar OpenClaw en servidores de producción es mediante **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — un instalador automatizado con arquitectura de seguridad primero.

## Inicio Rápido

Instalación con un comando:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Guía completa: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** El repositorio openclaw-ansible es la fuente de verdad para la implementación con Ansible. Esta página es una descripción general rápida.

## Lo que obtienes

-   🔒 **Seguridad primero con firewall**: UFW + aislamiento Docker (solo SSH + Tailscale accesibles)
-   🔐 **VPN Tailscale**: Acceso remoto seguro sin exponer servicios públicamente
-   🐳 **Docker**: Contenedores sandbox aislados, enlaces solo a localhost
-   🛡️ **Defensa en profundidad**: Arquitectura de seguridad de 4 capas
-   🚀 **Configuración con un comando**: Implementación completa en minutos
-   🔧 **Integración con Systemd**: Inicio automático al arrancar con endurecimiento

## Requisitos

-   **Sistema Operativo**: Debian 11+ o Ubuntu 20.04+
-   **Acceso**: Privilegios de root o sudo
-   **Red**: Conexión a Internet para la instalación de paquetes
-   **Ansible**: 2.14+ (instalado automáticamente por el script de inicio rápido)

## Qué se instala

El playbook de Ansible instala y configura:

1.  **Tailscale** (VPN en malla para acceso remoto seguro)
2.  **Firewall UFW** (solo puertos SSH + Tailscale)
3.  **Docker CE + Compose V2** (para sandboxes de agentes)
4.  **Node.js 22.x + pnpm** (dependencias de tiempo de ejecución)
5.  **OpenClaw** (basado en el host, no contenerizado)
6.  **Servicio Systemd** (inicio automático con endurecimiento de seguridad)

Nota: La puerta de enlace se ejecuta **directamente en el host** (no en Docker), pero los sandboxes de agentes usan Docker para aislamiento. Consulta [Sandboxing](../gateway/sandboxing.md) para más detalles.

## Configuración posterior a la instalación

Una vez completada la instalación, cambia al usuario openclaw:

```bash
sudo -i -u openclaw
```

El script posterior a la instalación te guiará a través de:

1.  **Asistente de incorporación**: Configura los ajustes de OpenClaw
2.  **Inicio de sesión del proveedor**: Conecta WhatsApp/Telegram/Discord/Signal
3.  **Prueba de la puerta de enlace**: Verifica la instalación
4.  **Configuración de Tailscale**: Conéctate a tu malla VPN

### Comandos rápidos

```bash
# Verificar estado del servicio
sudo systemctl status openclaw

# Ver registros en vivo
sudo journalctl -u openclaw -f

# Reiniciar la puerta de enlace
sudo systemctl restart openclaw

# Inicio de sesión del proveedor (ejecutar como usuario openclaw)
sudo -i -u openclaw
openclaw channels login
```

## Arquitectura de seguridad

### Defensa de 4 capas

1.  **Firewall (UFW)**: Solo SSH (22) + Tailscale (41641/udp) expuestos públicamente
2.  **VPN (Tailscale)**: La puerta de enlace es accesible solo a través de la malla VPN
3.  **Aislamiento Docker**: La cadena DOCKER-USER de iptables evita la exposición de puertos externos
4.  **Endurecimiento de Systemd**: NoNewPrivileges, PrivateTmp, usuario sin privilegios

### Verificación

Prueba la superficie de ataque externa:

```bash
nmap -p- TU_IP_DEL_SERVIDOR
```

Debería mostrar **solo el puerto 22** (SSH) abierto. Todos los demás servicios (puerta de enlace, Docker) están bloqueados.

### Disponibilidad de Docker

Docker se instala para **sandboxes de agentes** (ejecución aislada de herramientas), no para ejecutar la puerta de enlace en sí. La puerta de enlace se enlaza solo a localhost y es accesible a través de la VPN Tailscale. Consulta [Sandbox y herramientas multi-agente](../tools/multi-agent-sandbox-tools.md) para la configuración del sandbox.

## Instalación manual

Si prefieres control manual sobre la automatización:

```bash
# 1. Instalar requisitos previos
sudo apt update && sudo apt install -y ansible git

# 2. Clonar repositorio
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Instalar colecciones de Ansible
ansible-galaxy collection install -r requirements.yml

# 4. Ejecutar playbook
./run-playbook.sh

# O ejecutar directamente (luego ejecutar manualmente /tmp/openclaw-setup.sh después)
# ansible-playbook playbook.yml --ask-become-pass
```

## Actualizando OpenClaw

El instalador de Ansible configura OpenClaw para actualizaciones manuales. Consulta [Actualizando](./updating.md) para el flujo de actualización estándar. Para volver a ejecutar el playbook de Ansible (por ejemplo, para cambios de configuración):

```bash
cd openclaw-ansible
./run-playbook.sh
```

Nota: Esto es idempotente y seguro de ejecutar múltiples veces.

## Solución de problemas

### El firewall bloquea mi conexión

Si te quedas bloqueado fuera:

-   Asegúrate de poder acceder primero a través de la VPN Tailscale
-   El acceso SSH (puerto 22) siempre está permitido
-   La puerta de enlace es **solo** accesible a través de Tailscale por diseño

### El servicio no se inicia

```bash
# Verificar registros
sudo journalctl -u openclaw -n 100

# Verificar permisos
sudo ls -la /opt/openclaw

# Probar inicio manual
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Problemas con el sandbox de Docker

```bash
# Verificar que Docker esté ejecutándose
sudo systemctl status docker

# Verificar imagen del sandbox
sudo docker images | grep openclaw-sandbox

# Construir imagen del sandbox si falta
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### Fallo en el inicio de sesión del proveedor

Asegúrate de estar ejecutando como el usuario `openclaw`:

```bash
sudo -i -u openclaw
openclaw channels login
```

## Configuración avanzada

Para detalles de arquitectura de seguridad y solución de problemas:

-   [Arquitectura de seguridad](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
-   [Detalles técnicos](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
-   [Guía de solución de problemas](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## Relacionado

-   [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — guía completa de implementación
-   [Docker](./docker.md) — configuración de puerta de enlace contenerizada
-   [Sandboxing](../gateway/sandboxing.md) — configuración de sandbox de agentes
-   [Sandbox y herramientas multi-agente](../tools/multi-agent-sandbox-tools.md) — aislamiento por agente

[Nix](./nix.md)[Bun (Experimental)](./bun.md)