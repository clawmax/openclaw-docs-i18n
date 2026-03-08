

  Comandos CLI

  
# acp

Ejecuta el puente del [Protocolo de Cliente Agente (ACP)](https://agentclientprotocol.com/) que se comunica con una Puerta de enlace OpenClaw. Este comando habla ACP a través de stdio para IDEs y reenvía solicitudes a la Puerta de enlace a través de WebSocket. Mantiene las sesiones ACP mapeadas a claves de sesión de la Puerta de enlace.

## Uso

```bash
openclaw acp

# Puerta de enlace remota
openclaw acp --url wss://gateway-host:18789 --token <token>

# Puerta de enlace remota (token desde archivo)
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# Conectarse a una clave de sesión existente
openclaw acp --session agent:main:main

# Conectarse por etiqueta (debe existir previamente)
openclaw acp --session-label "support inbox"

# Reiniciar la clave de sesión antes de la primera solicitud
openclaw acp --session agent:main:main --reset-session
```

## Cliente ACP (depuración)

Usa el cliente ACP integrado para verificar el funcionamiento del puente sin un IDE. Genera el puente ACP y te permite escribir solicitudes de forma interactiva.

```bash
openclaw acp client

# Apuntar el puente generado a una Puerta de enlace remota
openclaw acp client --server-args --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# Sobrescribir el comando del servidor (predeterminado: openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

Modelo de permisos (modo de depuración del cliente):

-   La aprobación automática está basada en una lista de permitidos y solo se aplica a IDs de herramientas principales confiables.
-   La aprobación automática de `read` está limitada al directorio de trabajo actual (`--cwd` cuando se establece).
-   Los nombres de herramientas desconocidas/no principales, las lecturas fuera de alcance y las herramientas peligrosas siempre requieren aprobación explícita mediante solicitud.
-   El `toolCall.kind` proporcionado por el servidor se trata como metadatos no confiables (no es una fuente de autorización).

## Cómo usar esto

Usa ACP cuando un IDE (u otro cliente) hable el Protocolo de Cliente Agente y quieras que controle una sesión de una Puerta de enlace OpenClaw.

1.  Asegúrate de que la Puerta de enlace esté en ejecución (local o remota).
2.  Configura el objetivo de la Puerta de enlace (configuración o banderas).
3.  Configura tu IDE para ejecutar `openclaw acp` a través de stdio.

Ejemplo de configuración (persistente):

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

Ejemplo de ejecución directa (sin escribir configuración):

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
# preferible para seguridad del proceso local
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token
```

## Seleccionar agentes

ACP no selecciona agentes directamente. Enruta mediante la clave de sesión de la Puerta de enlace. Usa claves de sesión con alcance de agente para apuntar a un agente específico:

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

Cada sesión ACP se mapea a una única clave de sesión de la Puerta de enlace. Un agente puede tener muchas sesiones; ACP usa por defecto una sesión aislada `acp:` a menos que sobrescribas la clave o la etiqueta.

## Configuración del editor Zed

Añade un agente ACP personalizado en `~/.config/zed/settings.json` (o usa la IU de Configuración de Zed):

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

Para apuntar a una Puerta de enlace o agente específico:

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

En Zed, abre el panel de Agente y selecciona "OpenClaw ACP" para iniciar un hilo.

## Mapeo de sesiones

Por defecto, las sesiones ACP obtienen una clave de sesión de Puerta de enlace aislada con el prefijo `acp:`. Para reutilizar una sesión conocida, pasa una clave o etiqueta de sesión:

-   `--session `: usa una clave de sesión de Puerta de enlace específica.
-   `--session-label `: resuelve una sesión existente por etiqueta.
-   `--reset-session`: genera un nuevo ID de sesión para esa clave (misma clave, nueva transcripción).

Si tu cliente ACP admite metadatos, puedes sobrescribir por sesión:

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

Aprende más sobre las claves de sesión en [/concepts/session](../concepts/session.md).

## Opciones

-   `--url `: URL WebSocket de la Puerta de enlace (por defecto usa gateway.remote.url cuando está configurado).
-   `--token `: token de autenticación de la Puerta de enlace.
-   `--token-file `: leer token de autenticación de la Puerta de enlace desde un archivo.
-   `--password `: contraseña de autenticación de la Puerta de enlace.
-   `--password-file `: leer contraseña de autenticación de la Puerta de enlace desde un archivo.
-   `--session `: clave de sesión por defecto.
-   `--session-label `: etiqueta de sesión por defecto para resolver.
-   `--require-existing`: fallar si la clave/etiqueta de sesión no existe.
-   `--reset-session`: reiniciar la clave de sesión antes del primer uso.
-   `--no-prefix-cwd`: no prefijar las solicitudes con el directorio de trabajo.
-   `--verbose, -v`: registro detallado a stderr.

Nota de seguridad:

-   `--token` y `--password` pueden ser visibles en listados de procesos locales en algunos sistemas.
-   Prefiere `--token-file`/`--password-file` o variables de entorno (`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`).
-   Los procesos secundarios del backend de tiempo de ejecución de ACP reciben `OPENCLAW_SHELL=acp`, que puede usarse para reglas específicas de shell/perfil.
-   `openclaw acp client` establece `OPENCLAW_SHELL=acp-client` en el proceso del puente generado.

### Opciones del cliente acp

-   `--cwd `: directorio de trabajo para la sesión ACP.
-   `--server `: comando del servidor ACP (predeterminado: `openclaw`).
-   `--server-args <args...>`: argumentos adicionales pasados al servidor ACP.
-   `--server-verbose`: habilitar registro detallado en el servidor ACP.
-   `--verbose, -v`: registro detallado del cliente.

[Referencia CLI](../cli.md)[agente](./agent.md)