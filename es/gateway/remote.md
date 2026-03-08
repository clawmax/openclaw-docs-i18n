title: "Configuración de Acceso Remoto para OpenClaw Gateway mediante SSH y VPN"
description: "Aprende a configurar el acceso remoto al OpenClaw Gateway usando túneles SSH, Tailscale y VPNs para una conectividad de agente segura y siempre activa."
keywords: ["acceso remoto openclaw", "túnel ssh gateway", "tailscale vpn", "websocket remoto", "agente siempre activo", "ssh remoto macos", "seguridad gateway", "configuración remota cli"]
---

  Acceso remoto

  
# Acceso Remoto

Este repositorio admite el "acceso remoto por SSH" manteniendo un único Gateway (el maestro) ejecutándose en un host dedicado (escritorio/servidor) y conectando clientes a él.

-   Para **operadores (tú / la aplicación de macOS)**: el túnel SSH es el respaldo universal.
-   Para **nodos (iOS/Android y dispositivos futuros)**: conéctate al **WebSocket** del Gateway (LAN/tailnet o túnel SSH según sea necesario).

## La idea central

-   El WebSocket del Gateway se vincula a **loopback** en el puerto configurado (por defecto 18789).
-   Para uso remoto, reenvías ese puerto loopback a través de SSH (o usas una tailnet/VPN y reduces el túnel).

## Configuraciones comunes de VPN/tailnet (donde vive el agente)

Piensa en el **host del Gateway** como "donde vive el agente". Es dueño de las sesiones, perfiles de autenticación, canales y estado. Tu portátil/escritorio (y los nodos) se conectan a ese host.

### 1) Gateway siempre activo en tu tailnet (VPS o servidor doméstico)

Ejecuta el Gateway en un host persistente y accede a él a través de **Tailscale** o SSH.

-   **Mejor experiencia de usuario:** mantén `gateway.bind: "loopback"` y usa **Tailscale Serve** para la Interfaz de Control.
-   **Respaldo:** mantén loopback + túnel SSH desde cualquier máquina que necesite acceso.
-   **Ejemplos:** [exe.dev](../install/exe-dev.md) (VM fácil) o [Hetzner](../install/hetzner.md) (VPS de producción).

Esto es ideal cuando tu portátil se suspende a menudo pero quieres que el agente esté siempre activo.

### 2) El escritorio doméstico ejecuta el Gateway, el portátil es control remoto

El portátil **no** ejecuta el agente. Se conecta de forma remota:

-   Usa el modo **Remoto por SSH** de la aplicación de macOS (Configuración → General → "OpenClaw runs").
-   La aplicación abre y gestiona el túnel, por lo que WebChat + comprobaciones de estado "simplemente funcionan".

Procedimiento: [Acceso remoto en macOS](../platforms/mac/remote.md).

### 3) El portátil ejecuta el Gateway, acceso remoto desde otras máquinas

Mantén el Gateway local pero exponlo de forma segura:

-   Túnel SSH al portátil desde otras máquinas, o
-   Usa Tailscale Serve para la Interfaz de Control y mantén el Gateway solo en loopback.

Guía: [Tailscale](./tailscale.md) y [Vista general web](../web.md).

## Flujo de comandos (qué se ejecuta dónde)

Un servicio de gateway posee el estado + canales. Los nodos son periféricos. Ejemplo de flujo (Telegram → nodo):

-   El mensaje de Telegram llega al **Gateway**.
-   El Gateway ejecuta el **agente** y decide si llamar a una herramienta del nodo.
-   El Gateway llama al **nodo** a través del WebSocket del Gateway (RPC `node.*`).
-   El nodo devuelve el resultado; el Gateway responde de vuelta a Telegram.

Notas:

-   **Los nodos no ejecutan el servicio de gateway.** Solo un gateway debe ejecutarse por host a menos que ejecutes perfiles aislados intencionalmente (ver [Múltiples gateways](./multiple-gateways.md)).
-   El "modo nodo" de la aplicación de macOS es solo un cliente nodo sobre el WebSocket del Gateway.

## Túnel SSH (CLI + herramientas)

Crea un túnel local al WebSocket del Gateway remoto:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Con el túnel activo:

-   `openclaw health` y `openclaw status --deep` ahora alcanzan el gateway remoto a través de `ws://127.0.0.1:18789`.
-   `openclaw gateway {status,health,send,agent,call}` también pueden apuntar a la URL reenviada mediante `--url` cuando sea necesario.

Nota: reemplaza `18789` con tu `gateway.port` configurado (o `--port`/`OPENCLAW_GATEWAY_PORT`). Nota: cuando pasas `--url`, la CLI no recurre a las credenciales de configuración o entorno. Incluye `--token` o `--password` explícitamente. La falta de credenciales explícitas es un error.

## Valores predeterminados remotos de la CLI

Puedes persistir un destino remoto para que los comandos de la CLI lo usen por defecto:

```json
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token",
    },
  },
}
```

Cuando el gateway es solo loopback, mantén la URL en `ws://127.0.0.1:18789` y abre primero el túnel SSH.

## Precedencia de credenciales

La resolución de credenciales para llamadas/sondeos del Gateway ahora sigue un contrato compartido:

-   Las credenciales explícitas (`--token`, `--password`, o `gatewayToken` de la herramienta) siempre ganan.
-   Valores predeterminados del modo local:
    -   token: `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token` -> `gateway.remote.token`
    -   password: `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.auth.password` -> `gateway.remote.password`
-   Valores predeterminados del modo remoto:
    -   token: `gateway.remote.token` -> `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token`
    -   password: `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.remote.password` -> `gateway.auth.password`
-   Las comprobaciones de token para sondeos/estado remotos son estrictas por defecto: usan solo `gateway.remote.token` (sin recurrir al token local) cuando se apunta al modo remoto.
-   Las variables de entorno heredadas `CLAWDBOT_GATEWAY_*` solo son usadas por rutas de llamada de compatibilidad; la resolución de sondeo/estado/autenticación usa solo `OPENCLAW_GATEWAY_*`.

## Interfaz de chat sobre SSH

WebChat ya no usa un puerto HTTP separado. La interfaz de chat SwiftUI se conecta directamente al WebSocket del Gateway.

-   Reenvía `18789` a través de SSH (ver arriba), luego conecta clientes a `ws://127.0.0.1:18789`.
-   En macOS, prefiere el modo "Remoto por SSH" de la aplicación, que gestiona el túnel automáticamente.

## Aplicación de macOS "Remoto por SSH"

La aplicación de la barra de menús de macOS puede impulsar la misma configuración de extremo a extremo (comprobaciones de estado remotas, WebChat y reenvío de Voice Wake). Procedimiento: [Acceso remoto en macOS](../platforms/mac/remote.md).

## Reglas de seguridad (remoto/VPN)

Versión corta: **mantén el Gateway solo en loopback** a menos que estés seguro de que necesitas un enlace.

-   **Loopback + SSH/Tailscale Serve** es el valor predeterminado más seguro (sin exposición pública).
-   El texto plano `ws://` es solo loopback por defecto. Para redes privadas confiables, establece `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` en el proceso cliente como medida de emergencia.
-   Los **enlaces no loopback** (`lan`/`tailnet`/`custom`, o `auto` cuando loopback no está disponible) deben usar tokens/contraseñas de autenticación.
-   `gateway.remote.token` / `.password` son fuentes de credenciales del cliente. Por sí solas **no** configuran la autenticación del servidor.
-   Las rutas de llamada local pueden usar `gateway.remote.*` como respaldo cuando `gateway.auth.*` no está establecido.
-   `gateway.remote.tlsFingerprint` fija el certificado TLS remoto cuando se usa `wss://`.
-   **Tailscale Serve** puede autenticar el tráfico de la Interfaz de Control/WebSocket a través de cabeceras de identidad cuando `gateway.auth.allowTailscale: true`; los endpoints de la API HTTP aún requieren autenticación por token/contraseña. Este flujo sin token asume que el host del gateway es confiable. Establécelo en `false` si quieres tokens/contraseñas en todas partes.
-   Trata el control del navegador como acceso de operador: solo tailnet + emparejamiento deliberado de nodos.

Profundización: [Seguridad](./security.md).

[Descubrimiento Bonjour](./bonjour.md)[Configuración de Gateway Remoto](./remote-gateway-readme.md)