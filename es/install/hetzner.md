

  Alojamiento y despliegue

  
# Hetzner

## Objetivo

Ejecutar un OpenClaw Gateway persistente en un VPS de Hetzner usando Docker, con estado duradero, binarios integrados y comportamiento seguro ante reinicios. Si quieres "OpenClaw 24/7 por ~$5", esta es la configuración confiable más simple. Los precios de Hetzner cambian; elige el VPS Debian/Ubuntu más pequeño y escala si tienes problemas de memoria (OOM). Recordatorio del modelo de seguridad:

-   Los agentes compartidos en una empresa están bien cuando todos están en el mismo límite de confianza y el entorno de ejecución es solo para negocios.
-   Mantén una separación estricta: VPS/entorno de ejecución dedicado + cuentas dedicadas; sin perfiles personales de Apple/Google/navegador/gestor de contraseñas en ese host.
-   Si los usuarios son adversarios entre sí, sepáralos por gateway/host/usuario del sistema.

Consulta [Seguridad](../gateway/security.md) y [Alojamiento en VPS](../vps.md).

## ¿Qué estamos haciendo (en términos simples)?

-   Alquilar un pequeño servidor Linux (VPS de Hetzner)
-   Instalar Docker (entorno de ejecución de aplicaciones aislado)
-   Iniciar el OpenClaw Gateway en Docker
-   Persistir `~/.openclaw` + `~/.openclaw/workspace` en el host (sobrevive a reinicios/reconstrucciones)
-   Acceder a la Interfaz de Control desde tu portátil mediante un túnel SSH

Se puede acceder al Gateway mediante:

-   Reenvío de puertos SSH desde tu portátil
-   Exposición directa del puerto si gestionas el firewall y los tokens tú mismo

Esta guía asume Ubuntu o Debian en Hetzner.  
Si estás en otro VPS Linux, adapta los paquetes en consecuencia. Para el flujo genérico de Docker, consulta [Docker](./docker.md).

* * *

## Ruta rápida (operadores experimentados)

1.  Provisionar VPS de Hetzner
2.  Instalar Docker
3.  Clonar el repositorio de OpenClaw
4.  Crear directorios persistentes en el host
5.  Configurar `.env` y `docker-compose.yml`
6.  Integrar los binarios necesarios en la imagen
7.  `docker compose up -d`
8.  Verificar la persistencia y el acceso al Gateway

* * *

## Lo que necesitas

-   VPS de Hetzner con acceso root
-   Acceso SSH desde tu portátil
-   Comodidad básica con SSH + copiar/pegar
-   ~20 minutos
-   Docker y Docker Compose
-   Credenciales de autenticación del modelo
-   Credenciales opcionales del proveedor
    -   Código QR de WhatsApp
    -   Token del bot de Telegram
    -   OAuth de Gmail

* * *

## 1) Provisionar el VPS

Crea un VPS Ubuntu o Debian en Hetzner. Conéctate como root:

```bash
ssh root@YOUR_VPS_IP
```

Esta guía asume que el VPS es con estado. No lo trates como infraestructura desechable.

* * *

## 2) Instalar Docker (en el VPS)

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Verifica:

```bash
docker --version
docker compose version
```

* * *

## 3) Clonar el repositorio de OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Esta guía asume que construirás una imagen personalizada para garantizar la persistencia de los binarios.

* * *

## 4) Crear directorios persistentes en el host

Los contenedores Docker son efímeros. Todo el estado de larga duración debe residir en el host.

```bash
mkdir -p /root/.openclaw/workspace

# Establecer propiedad al usuario del contenedor (uid 1000):
chown -R 1000:1000 /root/.openclaw
```

* * *

## 5) Configurar variables de entorno

Crea `.env` en la raíz del repositorio.

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Genera secretos fuertes:

```bash
openssl rand -hex 32
```

**No hagas commit de este archivo.**

* * *

## 6) Configuración de Docker Compose

Crea o actualiza `docker-compose.yml`.

```
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Recomendado: mantener el Gateway solo en loopback en el VPS; acceder mediante túnel SSH.
      # Para exponerlo públicamente, elimina el prefijo `127.0.0.1:` y configura el firewall en consecuencia.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
        "--allow-unconfigured",
      ]
```

`--allow-unconfigured` es solo para conveniencia durante el arranque inicial, no es un reemplazo para una configuración adecuada del gateway. Aún así, configura la autenticación (`gateway.auth.token` o contraseña) y usa configuraciones de enlace seguras para tu despliegue.

* * *

## 7) Integrar binarios necesarios en la imagen (crítico)

Instalar binarios dentro de un contenedor en ejecución es una trampa. Cualquier cosa instalada en tiempo de ejecución se perderá al reiniciar. Todos los binarios externos requeridos por las habilidades deben instalarse en el momento de la construcción de la imagen. Los ejemplos a continuación muestran solo tres binarios comunes:

-   `gog` para acceso a Gmail
-   `goplaces` para Google Places
-   `wacli` para WhatsApp

Estos son ejemplos, no una lista completa. Puedes instalar tantos binarios como necesites usando el mismo patrón. Si añades nuevas habilidades más tarde que dependan de binarios adicionales, debes:

1.  Actualizar el Dockerfile
2.  Reconstruir la imagen
3.  Reiniciar los contenedores

**Ejemplo de Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Ejemplo binario 1: CLI de Gmail
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Ejemplo binario 2: CLI de Google Places
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Ejemplo binario 3: CLI de WhatsApp
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Añade más binarios abajo usando el mismo patrón

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

* * *

## 8) Construir y lanzar

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Verifica los binarios:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Salida esperada:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

* * *

## 9) Verificar el Gateway

```bash
docker compose logs -f openclaw-gateway
```

Éxito:

```ini
[gateway] listening on ws://0.0.0.0:18789
```

Desde tu portátil:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Abre: `http://127.0.0.1:18789/` Pega tu token del gateway.

* * *

## Qué persiste y dónde (fuente de verdad)

OpenClaw se ejecuta en Docker, pero Docker no es la fuente de verdad. Todo el estado de larga duración debe sobrevivir a reinicios, reconstrucciones y apagados.

| Componente | Ubicación | Mecanismo de persistencia | Notas |
| --- | --- | --- | --- |
| Configuración del Gateway | `/home/node/.openclaw/` | Montaje de volumen del host | Incluye `openclaw.json`, tokens |
| Perfiles de autenticación del modelo | `/home/node/.openclaw/` | Montaje de volumen del host | Tokens OAuth, claves API |
| Configuraciones de habilidades | `/home/node/.openclaw/skills/` | Montaje de volumen del host | Estado a nivel de habilidad |
| Espacio de trabajo del agente | `/home/node/.openclaw/workspace/` | Montaje de volumen del host | Código y artefactos del agente |
| Sesión de WhatsApp | `/home/node/.openclaw/` | Montaje de volumen del host | Preserva el inicio de sesión por QR |
| Llavero de Gmail | `/home/node/.openclaw/` | Volumen del host + contraseña | Requiere `GOG_KEYRING_PASSWORD` |
| Binarios externos | `/usr/local/bin/` | Imagen de Docker | Deben integrarse en el momento de la construcción |
| Entorno de ejecución Node | Sistema de archivos del contenedor | Imagen de Docker | Reconstruido en cada construcción de imagen |
| Paquetes del sistema | Sistema de archivos del contenedor | Imagen de Docker | No instalar en tiempo de ejecución |
| Contenedor Docker | Efímero | Reiniciable | Seguro de destruir |

* * *

## Infraestructura como Código (Terraform)

Para equipos que prefieren flujos de trabajo de infraestructura-como-código, una configuración de Terraform mantenida por la comunidad proporciona:

-   Configuración modular de Terraform con gestión de estado remoto
-   Provisionamiento automatizado mediante cloud-init
-   Scripts de despliegue (arranque, despliegue, copia de seguridad/restauración)
-   Fortalecimiento de seguridad (firewall, UFW, acceso solo SSH)
-   Configuración de túnel SSH para acceso al gateway

**Repositorios:**

-   Infraestructura: [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)
-   Configuración Docker: [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)

Este enfoque complementa la configuración de Docker anterior con despliegues reproducibles, infraestructura controlada por versión y recuperación ante desastres automatizada.

> **Nota:** Mantenido por la comunidad. Para problemas o contribuciones, consulta los enlaces a los repositorios anteriores.

[Fly.io](./fly.md)[GCP](./gcp.md)