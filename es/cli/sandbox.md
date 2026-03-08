

  Comandos CLI

  
# Sandbox CLI

Gestiona contenedores sandbox basados en Docker para la ejecución aislada de agentes.

## Descripción general

OpenClaw puede ejecutar agentes en contenedores Docker aislados por seguridad. Los comandos `sandbox` te ayudan a gestionar estos contenedores, especialmente después de actualizaciones o cambios de configuración.

## Comandos

### openclaw sandbox explain

Inspecciona el modo/alcance/acceso al espacio de trabajo **efectivo** del sandbox, la política de herramientas del sandbox y las puertas elevadas (con las rutas de clave de configuración para solucionarlo).

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

### openclaw sandbox list

Lista todos los contenedores sandbox con su estado y configuración.

```bash
openclaw sandbox list
openclaw sandbox list --browser  # Lista solo contenedores de navegador
openclaw sandbox list --json     # Salida en JSON
```

**La salida incluye:**

-   Nombre del contenedor y estado (en ejecución/detenido)
-   Imagen de Docker y si coincide con la configuración
-   Antigüedad (tiempo desde su creación)
-   Tiempo inactivo (tiempo desde el último uso)
-   Sesión/agente asociado

### openclaw sandbox recreate

Elimina contenedores sandbox para forzar su recreación con imágenes/configuración actualizadas.

```bash
openclaw sandbox recreate --all                # Recrea todos los contenedores
openclaw sandbox recreate --session main       # Sesión específica
openclaw sandbox recreate --agent mybot        # Agente específico
openclaw sandbox recreate --browser            # Solo contenedores de navegador
openclaw sandbox recreate --all --force        # Omite la confirmación
```

**Opciones:**

-   `--all`: Recrea todos los contenedores sandbox
-   `--session `: Recrea el contenedor para una sesión específica
-   `--agent `: Recrea contenedores para un agente específico
-   `--browser`: Solo recrea contenedores de navegador
-   `--force`: Omite el mensaje de confirmación

**Importante:** Los contenedores se recrean automáticamente la próxima vez que se use el agente.

## Casos de uso

### Después de actualizar imágenes de Docker

```bash
# Descargar nueva imagen
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# Actualizar configuración para usar la nueva imagen
# Editar config: agents.defaults.sandbox.docker.image (o agents.list[].sandbox.docker.image)

# Recrear contenedores
openclaw sandbox recreate --all
```

### Después de cambiar la configuración del sandbox

```bash
# Editar config: agents.defaults.sandbox.* (o agents.list[].sandbox.*)

# Recrear para aplicar la nueva configuración
openclaw sandbox recreate --all
```

### Después de cambiar setupCommand

```bash
openclaw sandbox recreate --all
# o solo un agente:
openclaw sandbox recreate --agent family
```

### Solo para un agente específico

```bash
# Actualiza solo los contenedores de un agente
openclaw sandbox recreate --agent alfred
```

## ¿Por qué es necesario?

**Problema:** Cuando actualizas las imágenes de Docker o la configuración del sandbox:

-   Los contenedores existentes siguen ejecutándose con la configuración antigua
-   Los contenedores solo se eliminan después de 24h de inactividad
-   Los agentes de uso regular mantienen los contenedores antiguos ejecutándose indefinidamente

**Solución:** Usa `openclaw sandbox recreate` para forzar la eliminación de los contenedores antiguos. Se recrearán automáticamente con la configuración actual cuando se necesiten la próxima vez. Consejo: prefiere `openclaw sandbox recreate` sobre `docker rm` manual. Usa la nomenclatura de contenedores del Gateway y evita desajustes cuando cambian las claves de alcance/sesión.

## Configuración

Los ajustes del sandbox se encuentran en `~/.openclaw/openclaw.json` bajo `agents.defaults.sandbox` (las anulaciones por agente van en `agents.list[].sandbox`):

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all", // off, non-main, all
        "scope": "agent", // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-",
          // ... más opciones de Docker
        },
        "prune": {
          "idleHours": 24, // Eliminación automática tras 24h inactivo
          "maxAgeDays": 7, // Eliminación automática tras 7 días
        },
      },
    },
  },
}
```

## Ver también

-   [Documentación del Sandbox](../gateway/sandboxing.md)
-   [Configuración de Agentes](../concepts/agent-workspace.md)
-   [Comando Doctor](../gateway/doctor.md) - Verifica la configuración del sandbox

[reset](./reset.md)[secrets](./secrets.md)

---