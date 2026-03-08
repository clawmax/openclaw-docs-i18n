

  Alojamiento y despliegue

  
# GCP

## Objetivo

Ejecutar un OpenClaw Gateway persistente en una VM de GCP Compute Engine usando Docker, con estado duradero, binarios integrados y comportamiento seguro ante reinicios. Si quieres "OpenClaw 24/7 por ~$5-12/mes", esta es una configuración confiable en Google Cloud. El precio varía según el tipo de máquina y región; elige la VM más pequeña que se ajuste a tu carga de trabajo y escala si tienes problemas de memoria (OOM).

## ¿Qué vamos a hacer (en términos simples)?

-   Crear un proyecto en GCP y habilitar facturación
-   Crear una VM de Compute Engine
-   Instalar Docker (entorno de ejecución aislado)
-   Iniciar el OpenClaw Gateway en Docker
-   Persistir `~/.openclaw` + `~/.openclaw/workspace` en el host (sobrevive a reinicios/reconstrucciones)
-   Acceder a la UI de Control desde tu portátil mediante un túnel SSH

El Gateway se puede acceder mediante:

-   Reenvío de puertos SSH desde tu portátil
-   Exposición directa de puertos si gestionas el firewall y los tokens tú mismo

Esta guía usa Debian en GCP Compute Engine. Ubuntu también funciona; adapta los paquetes en consecuencia. Para el flujo genérico de Docker, consulta [Docker](./docker.md).

* * *

## Ruta rápida (operadores experimentados)

1.  Crear proyecto GCP + habilitar API de Compute Engine
2.  Crear VM de Compute Engine (e2-small, Debian 12, 20GB)
3.  Conectarse por SSH a la VM
4.  Instalar Docker
5.  Clonar el repositorio de OpenClaw
6.  Crear directorios persistentes en el host
7.  Configurar `.env` y `docker-compose.yml`
8.  Integrar los binarios necesarios, construir y lanzar

* * *

## Lo que necesitas

-   Cuenta de GCP (elegible para el nivel gratuito con e2-micro)
-   CLI de gcloud instalada (o usar la Consola de Cloud)
-   Acceso SSH desde tu portátil
-   Comodidad básica con SSH + copiar/pegar
-   ~20-30 minutos
-   Docker y Docker Compose
-   Credenciales de autenticación de modelos
-   Credenciales opcionales de proveedores
    -   QR de WhatsApp
    -   Token de bot de Telegram
    -   OAuth de Gmail

* * *

## 1) Instalar la CLI de gcloud (o usar la Consola)

**Opción A: CLI de gcloud** (recomendado para automatización) Instalar desde [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install) Inicializar y autenticar:

```bash
gcloud init
gcloud auth login
```

**Opción B: Consola de Cloud** Todos los pasos se pueden hacer mediante la interfaz web en [https://console.cloud.google.com](https://console.cloud.google.com)

* * *

## 2) Crear un proyecto en GCP

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

Habilitar facturación en [https://console.cloud.google.com/billing](https://console.cloud.google.com/billing) (requerido para Compute Engine). Habilitar la API de Compute Engine:

```bash
gcloud services enable compute.googleapis.com
```

**Consola:**

1.  Ir a IAM y administración > Crear proyecto
2.  Ponerle nombre y crear
3.  Habilitar facturación para el proyecto
4.  Navegar a APIs y servicios > Habilitar APIs > buscar "Compute Engine API" > Habilitar

* * *

## 3) Crear la VM

**Tipos de máquina:**

| Tipo | Especificaciones | Costo | Notas |
| --- | --- | --- | --- |
| e2-medium | 2 vCPU, 4GB RAM | ~$25/mes | Más confiable para construcciones locales de Docker |
| e2-small | 2 vCPU, 2GB RAM | ~$12/mes | Mínimo recomendado para construcción con Docker |
| e2-micro | 2 vCPU (compartidos), 1GB RAM | Elegible para nivel gratuito | Suele fallar por OOM en construcción Docker (código de salida 137) |

**CLI:**

```
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Consola:**

1.  Ir a Compute Engine > Instancias de VM > Crear instancia
2.  Nombre: `openclaw-gateway`
3.  Región: `us-central1`, Zona: `us-central1-a`
4.  Tipo de máquina: `e2-small`
5.  Disco de arranque: Debian 12, 20GB
6.  Crear

* * *

## 4) Conectarse por SSH a la VM

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Consola:** Haz clic en el botón "SSH" junto a tu VM en el panel de Compute Engine. Nota: La propagación de la clave SSH puede tardar 1-2 minutos después de crear la VM. Si la conexión es rechazada, espera y vuelve a intentar.

* * *

## 5) Instalar Docker (en la VM)

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

Cerrar sesión y volver a entrar para que el cambio de grupo surta efecto:

```
exit
```

Luego volver a conectarse por SSH:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

Verificar:

```bash
docker --version
docker compose version
```

* * *

## 6) Clonar el repositorio de OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Esta guía asume que construirás una imagen personalizada para garantizar la persistencia de los binarios.

* * *

## 7) Crear directorios persistentes en el host

Los contenedores de Docker son efímeros. Todo el estado de larga duración debe residir en el host.

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

* * *

## 8) Configurar variables de entorno

Crear `.env` en la raíz del repositorio.

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Generar secretos fuertes:

```bash
openssl rand -hex 32
```

**No comprometer este archivo.**

* * *

## 9) Configuración de Docker Compose

Crear o actualizar `docker-compose.yml`.

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
      # Recomendado: mantener el Gateway solo en loopback en la VM; acceder mediante túnel SSH.
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
      ]
```

* * *

## 10) Integrar los binarios necesarios en la imagen (crítico)

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

## 11) Construir y lanzar

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Si la construcción falla con `Killed` / `código de salida 137` durante `pnpm install --frozen-lockfile`, la VM se quedó sin memoria. Usa `e2-small` como mínimo, o `e2-medium` para construcciones iniciales más confiables. Al enlazar a LAN (`OPENCLAW_GATEWAY_BIND=lan`), configura un origen de navegador confiable antes de continuar:

```bash
docker compose run --rm openclaw-cli config set gateway.controlUi.allowedOrigins '["http://127.0.0.1:18789"]' --strict-json
```

Si cambiaste el puerto del gateway, reemplaza `18789` con tu puerto configurado. Verificar binarios:

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

## 12) Verificar el Gateway

```bash
docker compose logs -f openclaw-gateway
```

Éxito:

```ini
[gateway] listening on ws://0.0.0.0:18789
```

* * *

## 13) Acceder desde tu portátil

Crear un túnel SSH para reenviar el puerto del Gateway:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

Abrir en tu navegador: `http://127.0.0.1:18789/` Obtener un enlace fresco del panel de control con token:

```bash
docker compose run --rm openclaw-cli dashboard --no-open
```

Pega el token de esa URL. Si la UI de Control muestra `unauthorized` o `disconnected (1008): pairing required`, aprueba el dispositivo del navegador:

```bash
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

* * *

## Qué persiste y dónde (fuente de verdad)

OpenClaw se ejecuta en Docker, pero Docker no es la fuente de verdad. Todo el estado de larga duración debe sobrevivir a reinicios, reconstrucciones y apagados.

| Componente | Ubicación | Mecanismo de persistencia | Notas |
| --- | --- | --- | --- |
| Configuración del Gateway | `/home/node/.openclaw/` | Montaje de volumen del host | Incluye `openclaw.json`, tokens |
| Perfiles de autenticación de modelos | `/home/node/.openclaw/` | Montaje de volumen del host | Tokens OAuth, claves API |
| Configuraciones de habilidades | `/home/node/.openclaw/skills/` | Montaje de volumen del host | Estado a nivel de habilidad |
| Espacio de trabajo del agente | `/home/node/.openclaw/workspace/` | Montaje de volumen del host | Código y artefactos del agente |
| Sesión de WhatsApp | `/home/node/.openclaw/` | Montaje de volumen del host | Preserva el inicio de sesión por QR |
| Llavero de Gmail | `/home/node/.openclaw/` | Volumen del host + contraseña | Requiere `GOG_KEYRING_PASSWORD` |
| Binarios externos | `/usr/local/bin/` | Imagen de Docker | Deben integrarse en el momento de la construcción |
| Entorno de ejecución Node | Sistema de archivos del contenedor | Imagen de Docker | Reconstruido en cada construcción de imagen |
| Paquetes del SO | Sistema de archivos del contenedor | Imagen de Docker | No instalar en tiempo de ejecución |
| Contenedor Docker | Efímero | Reiniciable | Seguro de destruir |

* * *

## Actualizaciones

Para actualizar OpenClaw en la VM:

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

* * *

## Solución de problemas

**Conexión SSH rechazada** La propagación de la clave SSH puede tardar 1-2 minutos después de crear la VM. Espera y vuelve a intentar. **Problemas con OS Login** Verifica tu perfil de OS Login:

```bash
gcloud compute os-login describe-profile
```

Asegúrate de que tu cuenta tenga los permisos IAM requeridos (Compute OS Login o Compute OS Admin Login). **Falta de memoria (OOM)** Si la construcción de Docker falla con `Killed` y `código de salida 137`, la VM fue terminada por OOM. Actualiza a e2-small (mínimo) o e2-medium (recomendado para construcciones locales confiables):

```bash
# Detener la VM primero
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# Cambiar tipo de máquina
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# Iniciar la VM
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

* * *

## Cuentas de servicio (mejor práctica de seguridad)

Para uso personal, tu cuenta de usuario predeterminada funciona bien. Para automatización o pipelines de CI/CD, crea una cuenta de servicio dedicada con permisos mínimos:

1.  Crear una cuenta de servicio:
    
    Copy
    
    ```
    gcloud iam service-accounts create openclaw-deploy \
      --display-name="OpenClaw Deployment"
    ```
    
2.  Otorgar rol de Administrador de Instancias de Compute (o un rol personalizado más restringido):
    
    Copy
    
    ```
    gcloud projects add-iam-policy-binding my-openclaw-project \
      --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
      --role="roles/compute.instanceAdmin.v1"
    ```
    

Evita usar el rol de Propietario para automatización. Usa el principio de privilegio mínimo. Consulta [https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles) para detalles sobre roles IAM.

* * *

## Próximos pasos

-   Configurar canales de mensajería: [Canales](../channels.md)
-   Emparejar dispositivos locales como nodos: [Nodos](../nodes.md)
-   Configurar el Gateway: [Configuración del Gateway](../gateway/configuration.md)

[Hetzner](./hetzner.md)[VMs macOS](./macos-vm.md)