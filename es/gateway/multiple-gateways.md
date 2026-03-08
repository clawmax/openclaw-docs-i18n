

  Configuración y operaciones

  
# Múltiples Gateways

La mayoría de las configuraciones deben usar un solo Gateway, ya que un único Gateway puede manejar múltiples conexiones de mensajería y agentes. Si necesitas un aislamiento más fuerte o redundancia (por ejemplo, un bot de rescate), ejecuta Gateways separados con perfiles/puertos aislados.

## Lista de verificación para aislamiento (requerido)

-   `OPENCLAW_CONFIG_PATH` — archivo de configuración por instancia
-   `OPENCLAW_STATE_DIR` — sesiones, credenciales, cachés por instancia
-   `agents.defaults.workspace` — raíz del espacio de trabajo por instancia
-   `gateway.port` (o `--port`) — único por instancia
-   Los puertos derivados (navegador/canvas) no deben superponerse

Si estos se comparten, encontrarás carreras de configuración y conflictos de puertos.

## Recomendado: perfiles (--profile)

Los perfiles delimitan automáticamente `OPENCLAW_STATE_DIR` + `OPENCLAW_CONFIG_PATH` y añaden un sufijo a los nombres de los servicios.

```bash
# principal
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# rescate
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

Servicios por perfil:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

## Guía para el bot de rescate

Ejecuta un segundo Gateway en el mismo host con su propio:

-   perfil/configuración
-   directorio de estado
-   espacio de trabajo
-   puerto base (más los puertos derivados)

Esto mantiene el bot de rescate aislado del bot principal para que pueda depurar o aplicar cambios de configuración si el bot primario está caído. Espaciado de puertos: deja al menos 20 puertos entre los puertos base para que los puertos derivados del navegador/canvas/CDP nunca colisionen.

### Cómo instalar (bot de rescate)

```bash
# Bot principal (existente o nuevo, sin el parámetro --profile)
# Se ejecuta en el puerto 18789 + Puertos CDC/Canvas/... de Chrome
openclaw onboard
openclaw gateway install

# Bot de rescate (perfil aislado + puertos)
openclaw --profile rescue onboard
# Notas:
# - el nombre del espacio de trabajo se añadirá con -rescue por defecto
# - El puerto debe ser al menos 18789 + 20 Puertos,
#   es mejor elegir un puerto base completamente diferente, como 19789,
# - el resto del proceso de incorporación es igual que el normal

# Para instalar el servicio (si no ocurrió automáticamente durante la incorporación)
openclaw --profile rescue gateway install
```

## Mapeo de puertos (derivados)

Puerto base = `gateway.port` (o `OPENCLAW_GATEWAY_PORT` / `--port`).

-   puerto del servicio de control del navegador = base + 2 (solo loopback)
-   el host del canvas se sirve en el servidor HTTP del Gateway (mismo puerto que `gateway.port`)
-   Los puertos CDP del perfil del navegador se asignan automáticamente desde `browser.controlPort + 9 .. + 108`

Si anulas alguno de estos en la configuración o variables de entorno, debes mantenerlos únicos por instancia.

## Notas sobre Navegador/CDP (error común)

-   **No** fijes `browser.cdpUrl` a los mismos valores en múltiples instancias.
-   Cada instancia necesita su propio puerto de control del navegador y rango CDP (derivados de su puerto de gateway).
-   Si necesitas puertos CDP explícitos, configura `browser.profiles..cdpPort` por instancia.
-   Chrome remoto: usa `browser.profiles..cdpUrl` (por perfil, por instancia).

## Ejemplo manual con variables de entorno

```
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```

## Comprobaciones rápidas

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```

[Ejecución en Segundo Plano y Herramienta de Procesos](./background-process.md)[Solución de Problemas](./troubleshooting.md)