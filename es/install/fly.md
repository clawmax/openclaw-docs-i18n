

  Alojamiento y despliegue

  
# Fly.io

**Objetivo:** OpenClaw Gateway ejecutándose en una máquina de [Fly.io](https://fly.io) con almacenamiento persistente, HTTPS automático y acceso a Discord/canales.

## Lo que necesitas

-   [CLI de flyctl](https://fly.io/docs/hands-on/install-flyctl/) instalado
-   Cuenta de Fly.io (el nivel gratuito funciona)
-   Autenticación del modelo: Clave API para tu proveedor de modelos elegido
-   Credenciales del canal: Token de bot de Discord, token de Telegram, etc.

## Ruta rápida para principiantes

1.  Clonar el repositorio → personalizar `fly.toml`
2.  Crear aplicación + volumen → establecer secretos
3.  Desplegar con `fly deploy`
4.  Conectar por SSH para crear la configuración o usar la UI de Control

## 1) Crear la aplicación en Fly

```bash
# Clonar el repositorio
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Crear una nueva aplicación Fly (elige tu propio nombre)
fly apps create my-openclaw

# Crear un volumen persistente (1GB suele ser suficiente)
fly volumes create openclaw_data --size 1 --region iad
```

**Consejo:** Elige una región cercana a ti. Opciones comunes: `lhr` (Londres), `iad` (Virginia), `sjc` (San José).

## 2) Configurar fly.toml

Edita `fly.toml` para que coincida con el nombre de tu aplicación y tus requisitos. **Nota de seguridad:** La configuración por defecto expone una URL pública. Para un despliegue reforzado sin IP pública, consulta [Despliegue Privado](#private-deployment-hardened) o usa `fly.private.toml`.

```
app = "my-openclaw"  # Tu nombre de aplicación
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**Ajustes clave:**

| Ajuste | Por qué |
| --- | --- |
| `--bind lan` | Se enlaza a `0.0.0.0` para que el proxy de Fly pueda alcanzar la puerta de enlace |
| `--allow-unconfigured` | Inicia sin un archivo de configuración (lo crearás después) |
| `internal_port = 3000` | Debe coincidir con `--port 3000` (o `OPENCLAW_GATEWAY_PORT`) para las comprobaciones de salud de Fly |
| `memory = "2048mb"` | 512MB es demasiado pequeño; se recomiendan 2GB |
| `OPENCLAW_STATE_DIR = "/data"` | Persiste el estado en el volumen |

## 3) Establecer secretos

```bash
# Requerido: Token de la puerta de enlace (para enlace no local)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# Claves API del proveedor de modelos
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Opcional: Otros proveedores
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Tokens de canales
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**Notas:**

-   Los enlaces no locales (`--bind lan`) requieren `OPENCLAW_GATEWAY_TOKEN` por seguridad.
-   Trata estos tokens como contraseñas.
-   **Prefiere variables de entorno sobre el archivo de configuración** para todas las claves API y tokens. Esto mantiene los secretos fuera de `openclaw.json` donde podrían exponerse o registrarse accidentalmente.

## 4) Desplegar

```bash
fly deploy
```

El primer despliegue construye la imagen Docker (~2-3 minutos). Los despliegues posteriores son más rápidos. Después del despliegue, verifica:

```bash
fly status
fly logs
```

Deberías ver:

```ini
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```

## 5) Crear archivo de configuración

Conéctate por SSH a la máquina para crear una configuración adecuada:

```bash
fly ssh console
```

Crea el directorio y el archivo de configuración:

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**Nota:** Con `OPENCLAW_STATE_DIR=/data`, la ruta de configuración es `/data/openclaw.json`. **Nota:** El token de Discord puede provenir de:

-   Variable de entorno: `DISCORD_BOT_TOKEN` (recomendado para secretos)
-   Archivo de configuración: `channels.discord.token`

Si usas la variable de entorno, no es necesario añadir el token a la configuración. La puerta de enlace lee `DISCORD_BOT_TOKEN` automáticamente. Reinicia para aplicar:

```
exit
fly machine restart <machine-id>
```

## 6) Acceder a la Puerta de Enlace

### UI de Control

Abre en el navegador:

```bash
fly open
```

O visita `https://my-openclaw.fly.dev/` Pega tu token de puerta de enlace (el de `OPENCLAW_GATEWAY_TOKEN`) para autenticarte.

### Registros

```bash
fly logs              # Registros en vivo
fly logs --no-tail    # Registros recientes
```

### Consola SSH

```bash
fly ssh console
```

## Resolución de problemas

### ”La aplicación no está escuchando en la dirección esperada”

La puerta de enlace se está enlazando a `127.0.0.1` en lugar de `0.0.0.0`. **Solución:** Añade `--bind lan` a tu comando de proceso en `fly.toml`.

### Comprobaciones de salud fallando / conexión rechazada

Fly no puede alcanzar la puerta de enlace en el puerto configurado. **Solución:** Asegúrate de que `internal_port` coincida con el puerto de la puerta de enlace (establece `--port 3000` o `OPENCLAW_GATEWAY_PORT=3000`).

### OOM / Problemas de Memoria

El contenedor se reinicia continuamente o es eliminado. Señales: `SIGABRT`, `v8::internal::Runtime_AllocateInYoungGeneration`, o reinicios silenciosos. **Solución:** Aumenta la memoria en `fly.toml`:

```
[[vm]]
  memory = "2048mb"
```

O actualiza una máquina existente:

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**Nota:** 512MB es demasiado pequeño. 1GB puede funcionar pero puede causar OOM bajo carga o con registros detallados. **Se recomiendan 2GB.**

### Problemas de Bloqueo de la Puerta de Enlace

La puerta de enlace se niega a iniciar con errores de "ya se está ejecutando". Esto sucede cuando el contenedor se reinicia pero el archivo de bloqueo PID persiste en el volumen. **Solución:** Elimina el archivo de bloqueo:

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

El archivo de bloqueo está en `/data/gateway.*.lock` (no en un subdirectorio).

### La Configuración No Se Está Leyendo

Si usas `--allow-unconfigured`, la puerta de enlace crea una configuración mínima. Tu configuración personalizada en `/data/openclaw.json` debería leerse al reiniciar. Verifica que la configuración exista:

```bash
fly ssh console --command "cat /data/openclaw.json"
```

### Escribir Configuración vía SSH

El comando `fly ssh console -C` no admite redirección de shell. Para escribir un archivo de configuración:

```bash
# Usa echo + tee (tubería de local a remoto)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# O usa sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**Nota:** `fly sftp` puede fallar si el archivo ya existe. Elimínalo primero:

```bash
fly ssh console --command "rm /data/openclaw.json"
```

### El Estado No Persiste

Si pierdes credenciales o sesiones después de un reinicio, el directorio de estado está escribiendo en el sistema de archivos del contenedor. **Solución:** Asegúrate de que `OPENCLAW_STATE_DIR=/data` esté establecido en `fly.toml` y vuelve a desplegar.

## Actualizaciones

```bash
# Obtener los últimos cambios
git pull

# Volver a desplegar
fly deploy

# Comprobar salud
fly status
fly logs
```

### Actualizar Comando de la Máquina

Si necesitas cambiar el comando de inicio sin un despliegue completo:

```bash
# Obtener ID de la máquina
fly machines list

# Actualizar comando
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# O con aumento de memoria
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**Nota:** Después de `fly deploy`, el comando de la máquina puede restablecerse a lo que hay en `fly.toml`. Si hiciste cambios manuales, vuelve a aplicarlos después del despliegue.

## Despliegue Privado (Reforzado)

Por defecto, Fly asigna IPs públicas, haciendo que tu puerta de enlace sea accesible en `https://your-app.fly.dev`. Esto es conveniente pero significa que tu despliegue es detectable por escáneres de internet (Shodan, Censys, etc.). Para un despliegue reforzado **sin exposición pública**, usa la plantilla privada.

### Cuándo usar despliegue privado

-   Solo haces llamadas/mensajes **salientes** (sin webhooks entrantes)
-   Usas túneles **ngrok o Tailscale** para cualquier devolución de llamada de webhook
-   Accedes a la puerta de enlace vía **SSH, proxy o WireGuard** en lugar del navegador
-   Quieres que el despliegue esté **oculto de los escáneres de internet**

### Configuración

Usa `fly.private.toml` en lugar de la configuración estándar:

```bash
# Desplegar con configuración privada
fly deploy -c fly.private.toml
```

O convierte un despliegue existente:

```bash
# Listar IPs actuales
fly ips list -a my-openclaw

# Liberar IPs públicas
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# Cambiar a configuración privada para que futuros despliegues no reasignen IPs públicas
# (elimina [http_service] o despliega con la plantilla privada)
fly deploy -c fly.private.toml

# Asignar solo IPv6 privada
fly ips allocate-v6 --private -a my-openclaw
```

Después de esto, `fly ips list` debería mostrar solo una IP de tipo `private`:

```bash
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```

### Acceder a un despliegue privado

Como no hay URL pública, usa uno de estos métodos: **Opción 1: Proxy local (más simple)**

```bash
# Redirigir puerto local 3000 a la aplicación
fly proxy 3000:3000 -a my-openclaw

# Luego abre http://localhost:3000 en el navegador
```

**Opción 2: VPN WireGuard**

```bash
# Crear configuración WireGuard (una vez)
fly wireguard create

# Importar al cliente WireGuard, luego acceder vía IPv6 interna
# Ejemplo: http://[fdaa:x:x:x:x::x]:3000
```

**Opción 3: Solo SSH**

```bash
fly ssh console -a my-openclaw
```

### Webhooks con despliegue privado

Si necesitas devoluciones de llamada de webhook (Twilio, Telnyx, etc.) sin exposición pública:

1.  **Túnel ngrok** - Ejecuta ngrok dentro del contenedor o como sidecar
2.  **Tailscale Funnel** - Expone rutas específicas vía Tailscale
3.  **Solo saliente** - Algunos proveedores (Twilio) funcionan bien para llamadas salientes sin webhooks

Ejemplo de configuración de llamada de voz con ngrok:

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" },
          "webhookSecurity": {
            "allowedHosts": ["example.ngrok.app"]
          }
        }
      }
    }
  }
}
```

El túnel ngrok se ejecuta dentro del contenedor y proporciona una URL de webhook pública sin exponer la aplicación Fly en sí. Establece `webhookSecurity.allowedHosts` en el nombre de host público del túnel para que los encabezados de host reenviados sean aceptados.

### Beneficios de seguridad

| Aspecto | Público | Privado |
| --- | --- | --- |
| Escáneres de internet | Detectable | Oculto |
| Ataques directos | Posibles | Bloqueados |
| Acceso a UI de Control | Navegador | Proxy/VPN |
| Entrega de webhook | Directa | Vía túnel |

## Notas

-   Fly.io usa **arquitectura x86** (no ARM)
-   El Dockerfile es compatible con ambas arquitecturas
-   Para la incorporación de WhatsApp/Telegram, usa `fly ssh console`
-   Los datos persistentes viven en el volumen en `/data`
-   Signal requiere Java + signal-cli; usa una imagen personalizada y mantén la memoria en 2GB+.

## Coste

Con la configuración recomendada (`shared-cpu-2x`, 2GB RAM):

-   ~$10-15/mes dependiendo del uso
-   El nivel gratuito incluye cierta asignación

Consulta [Precios de Fly.io](https://fly.io/docs/about/pricing/) para más detalles.

[Alojamiento VPS](../vps.md)[Hetzner](./hetzner.md)