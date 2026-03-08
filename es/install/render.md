

  Alojamiento y despliegue

  
# Desplegar en Render

Despliega OpenClaw en Render usando Infraestructura como Código. El Blueprint `render.yaml` incluido define toda tu pila de forma declarativa: servicio, disco, variables de entorno, para que puedas desplegar con un solo clic y versionar tu infraestructura junto con tu código.

## Requisitos previos

-   Una [cuenta de Render](https://render.com) (hay un plan gratuito disponible)
-   Una clave API de tu [proveedor de modelos](../providers.md) preferido

## Desplegar con un Blueprint de Render

[Desplegar en Render](https://render.com/deploy?repo=https://github.com/openclaw/openclaw) Al hacer clic en este enlace se hará lo siguiente:

1.  Crear un nuevo servicio en Render desde el Blueprint `render.yaml` en la raíz de este repositorio.
2.  Pedirte que configures `SETUP_PASSWORD`
3.  Construir la imagen Docker y desplegarla

Una vez desplegado, la URL de tu servicio sigue el patrón `https://<nombre-del-servicio>.onrender.com`.

## Entendiendo el Blueprint

Los Blueprints de Render son archivos YAML que definen tu infraestructura. El `render.yaml` en este repositorio configura todo lo necesario para ejecutar OpenClaw:

```yaml
services:
  - type: web
    name: openclaw
    runtime: docker
    plan: starter
    healthCheckPath: /health
    envVars:
      - key: PORT
        value: "8080"
      - key: SETUP_PASSWORD
        sync: false # prompts during deploy
      - key: OPENCLAW_STATE_DIR
        value: /data/.openclaw
      - key: OPENCLAW_WORKSPACE_DIR
        value: /data/workspace
      - key: OPENCLAW_GATEWAY_TOKEN
        generateValue: true # auto-generates a secure token
    disk:
      name: openclaw-data
      mountPath: /data
      sizeGB: 1
```

Características clave del Blueprint utilizadas:

| Característica | Propósito |
| --- | --- |
| `runtime: docker` | Construye desde el Dockerfile del repositorio |
| `healthCheckPath` | Render monitorea `/health` y reinicia instancias no saludables |
| `sync: false` | Solicita el valor durante el despliegue (secretos) |
| `generateValue: true` | Genera automáticamente un valor criptográficamente seguro |
| `disk` | Almacenamiento persistente que sobrevive a los re-despliegues |

## Elegir un plan

| Plan | Suspensión | Disco | Recomendado para |
| --- | --- | --- | --- |
| Gratuito | Después de 15 min inactivo | No disponible | Pruebas, demostraciones |
| Starter | Nunca | 1GB+ | Uso personal, equipos pequeños |
| Standard+ | Nunca | 1GB+ | Producción, múltiples canales |

El Blueprint usa por defecto `starter`. Para usar el plan gratuito, cambia a `plan: free` en el `render.yaml` de tu fork (pero nota: sin disco persistente, la configuración se reinicia en cada despliegue).

## Después del despliegue

### Completar el asistente de configuración

1.  Navega a `https://<tu-servicio>.onrender.com/setup`
2.  Ingresa tu `SETUP_PASSWORD`
3.  Selecciona un proveedor de modelos y pega tu clave API
4.  Configura opcionalmente canales de mensajería (Telegram, Discord, Slack)
5.  Haz clic en **Ejecutar configuración**

### Acceder a la Interfaz de Control

El panel web está disponible en `https://<tu-servicio>.onrender.com/openclaw`.

## Características del Panel de Render

### Registros (Logs)

Visualiza registros en tiempo real en **Panel → tu servicio → Registros**. Filtra por:

-   Registros de construcción (creación de imagen Docker)
-   Registros de despliegue (inicio del servicio)
-   Registros de tiempo de ejecución (salida de la aplicación)

### Acceso por Shell

Para depuración, abre una sesión de shell desde **Panel → tu servicio → Shell**. El disco persistente está montado en `/data`.

### Variables de entorno

Modifica las variables en **Panel → tu servicio → Entorno**. Los cambios activan un re-despliegue automático.

### Auto-despliegue

Si usas el repositorio original de OpenClaw, Render no hará auto-despliegue de tu OpenClaw. Para actualizarlo, ejecuta una sincronización manual del Blueprint desde el panel.

## Dominio personalizado

1.  Ve a **Panel → tu servicio → Configuración → Dominios personalizados**
2.  Añade tu dominio
3.  Configura el DNS como se indica (registro CNAME a `*.onrender.com`)
4.  Render provisiona un certificado TLS automáticamente

## Escalado

Render soporta escalado horizontal y vertical:

-   **Vertical**: Cambia el plan para obtener más CPU/RAM
-   **Horizontal**: Aumenta el número de instancias (plan Standard y superiores)

Para OpenClaw, el escalado vertical suele ser suficiente. El escalado horizontal requiere sesiones persistentes (sticky sessions) o gestión de estado externa.

## Copias de seguridad y migración

Exporta tu configuración y espacio de trabajo en cualquier momento:

```
https://<tu-servicio>.onrender.com/setup/export
```

Esto descarga una copia de seguridad portátil que puedes restaurar en cualquier host de OpenClaw.

## Solución de problemas

### El servicio no inicia

Revisa los registros de despliegue en el Panel de Render. Problemas comunes:

-   Falta `SETUP_PASSWORD` — el Blueprint lo solicita, pero verifica que esté configurado
-   Puerto incorrecto — asegúrate de que `PORT=8080` coincida con el puerto expuesto en el Dockerfile

### Inicios en frío lentos (plan gratuito)

Los servicios del plan gratuito se suspenden después de 15 minutos de inactividad. La primera solicitud tras la suspensión tarda unos segundos mientras se inicia el contenedor. Actualiza al plan Starter para tenerlo siempre activo.

### Pérdida de datos tras re-desplegar

Esto sucede en el plan gratuito (sin disco persistente). Actualiza a un plan de pago, o exporta regularmente tu configuración mediante `/setup/export`.

### Fallos en la verificación de salud (health check)

Render espera una respuesta 200 de `/health` en 30 segundos. Si las construcciones tienen éxito pero los despliegues fallan, el servicio podría estar tardando demasiado en iniciar. Verifica:

-   Los registros de construcción en busca de errores
-   Si el contenedor se ejecuta localmente con `docker build && docker run`

[Desplegar en Railway](./railway.md)[Desplegar en Northflank](./northflank.md)