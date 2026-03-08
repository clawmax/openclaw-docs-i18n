

  Gateway

  
# Runbook del Gateway

Usa esta página para el inicio del día 1 y las operaciones del día 2 del servicio Gateway.

## Inicio local en 5 minutos

### Paso 1: Iniciar el Gateway

```bash
openclaw gateway --port 18789
# depuración/seguimiento reflejado en stdio
openclaw gateway --port 18789 --verbose
# forzar la terminación del listener en el puerto seleccionado, luego iniciar
openclaw gateway --force
```

### Paso 2: Verificar la salud del servicio

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

Línea base saludable: `Runtime: running` y `RPC probe: ok`.

### Paso 3: Validar la preparación de los canales

```bash
openclaw channels status --probe
```

 

> **ℹ️** La recarga de configuración del Gateway observa la ruta del archivo de configuración activo (resuelta a partir de los valores predeterminados de perfil/estado, o `OPENCLAW_CONFIG_PATH` cuando está establecida). El modo predeterminado es `gateway.reload.mode="hybrid"`.

## Modelo de tiempo de ejecución

-   Un proceso siempre activo para enrutamiento, plano de control y conexiones de canal.
-   Puerto único multiplexado para:
    -   Control/RPC WebSocket
    -   APIs HTTP (compatibles con OpenAI, Respuestas, invocación de herramientas)
    -   UI de control y hooks
-   Modo de enlace predeterminado: `loopback`.
-   La autenticación es obligatoria por defecto (`gateway.auth.token` / `gateway.auth.password`, o `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Precedencia de puerto y enlace

| Configuración | Orden de resolución |
| --- | --- |
| Puerto del Gateway | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Modo de enlace | CLI/sobrescritura → `gateway.bind` → `loopback` |

### Modos de recarga en caliente

| `gateway.reload.mode` | Comportamiento |
| --- | --- |
| `off` | Sin recarga de configuración |
| `hot` | Aplicar solo cambios seguros en caliente |
| `restart` | Reiniciar en cambios que requieren recarga |
| `hybrid` (predeterminado) | Aplicar en caliente cuando sea seguro, reiniciar cuando sea requerido |

## Conjunto de comandos del operador

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw secrets reload
openclaw logs --follow
openclaw doctor
```

## Acceso remoto

Preferido: Tailscale/VPN. Alternativa: túnel SSH.

```bash
ssh -N -L 18789:127.0.0.1:18789 usuario@host
```

Luego conectar clientes a `ws://127.0.0.1:18789` localmente.

> **⚠️** Si la autenticación del gateway está configurada, los clientes aún deben enviar autenticación (`token`/`password`) incluso a través de túneles SSH.

 Ver: [Gateway Remoto](./gateway/remote.md), [Autenticación](./gateway/authentication.md), [Tailscale](./gateway/tailscale.md).

## Supervisión y ciclo de vida del servicio

Usa ejecuciones supervisadas para una fiabilidad similar a la de producción.

].service\nopenclaw gateway status', lang: 'bash' }, { label: 'Linux (system service)', code: 'sudo systemctl daemon-reload\nsudo systemctl enable --now openclaw-gateway[-].service', lang: 'bash' }]} />

## Múltiples gateways en un mismo host

La mayoría de las configuraciones deben ejecutar **un** Gateway. Usa múltiples solo para aislamiento/redundancia estricta (por ejemplo, un perfil de rescate). Lista de comprobación por instancia:

-   `gateway.port` único
-   `OPENCLAW_CONFIG_PATH` único
-   `OPENCLAW_STATE_DIR` único
-   `agents.defaults.workspace` único

Ejemplo:

```
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Ver: [Múltiples gateways](./gateway/multiple-gateways.md).

### Ruta rápida de perfil de desarrollo

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Los valores predeterminados incluyen estado/configuración aislados y puerto base del gateway `19001`.

## Referencia rápida del protocolo (vista del operador)

-   El primer frame del cliente debe ser `connect`.
-   El Gateway devuelve una instantánea `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, límites/política).
-   Solicitudes: `req(method, params)` → `res(ok/payload|error)`.
-   Eventos comunes: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Las ejecuciones de agente son de dos etapas:

1.  Acuse de recibo inmediato aceptado (`status:"accepted"`)
2.  Respuesta de finalización final (`status:"ok"|"error"`), con eventos `agent` transmitidos en el medio.

Ver documentación completa del protocolo: [Protocolo del Gateway](./gateway/protocol.md).

## Comprobaciones operativas

### Actividad

-   Abrir WS y enviar `connect`.
-   Esperar respuesta `hello-ok` con instantánea.

### Preparación

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Recuperación de brechas

Los eventos no se reproducen. En brechas de secuencia, refrescar el estado (`health`, `system-presence`) antes de continuar.

## Firmas de fallo comunes

| Firma | Problema probable |
| --- | --- |
| `refusing to bind gateway ... without auth` | Enlace no loopback sin token/contraseña |
| `another gateway instance is already listening` / `EADDRINUSE` | Conflicto de puerto |
| `Gateway start blocked: set gateway.mode=local` | Configuración establecida en modo remoto |
| `unauthorized` durante la conexión | Desajuste de autenticación entre cliente y gateway |

Para escaleras de diagnóstico completas, usa [Solución de Problemas del Gateway](./gateway/troubleshooting.md).

## Garantías de seguridad

-   Los clientes del protocolo Gateway fallan rápidamente cuando el Gateway no está disponible (sin fallback implícito a canal directo).
-   Los primeros frames inválidos/no connect son rechazados y cerrados.
-   El apagado elegante emite el evento `shutdown` antes de cerrar el socket.

* * *

Relacionado:

-   [Solución de Problemas](./gateway/troubleshooting.md)
-   [Proceso en Segundo Plano](./gateway/background-process.md)
-   [Configuración](./gateway/configuration.md)
-   [Salud](./gateway/health.md)
-   [Doctor](./gateway/doctor.md)
-   [Autenticación](./gateway/authentication.md)

[Configuración](./gateway/configuration.md)