

  Acceso remoto

  
# Tailscale

OpenClaw puede configurar automáticamente Tailscale **Serve** (tailnet) o **Funnel** (público) para el panel de control del Gateway y el puerto WebSocket. Esto mantiene el Gateway vinculado al loopback mientras Tailscale proporciona HTTPS, enrutamiento y (para Serve) cabeceras de identidad.

## Modos

-   `serve`: Serve solo para la tailnet mediante `tailscale serve`. El gateway permanece en `127.0.0.1`.
-   `funnel`: HTTPS público mediante `tailscale funnel`. OpenClaw requiere una contraseña compartida.
-   `off`: Por defecto (sin automatización de Tailscale).

## Autenticación

Configura `gateway.auth.mode` para controlar el handshake:

-   `token` (por defecto cuando `OPENCLAW_GATEWAY_TOKEN` está establecido)
-   `password` (secreto compartido mediante `OPENCLAW_GATEWAY_PASSWORD` o configuración)

Cuando `tailscale.mode = "serve"` y `gateway.auth.allowTailscale` es `true`, la autenticación de la UI de Control/WebSocket puede usar las cabeceras de identidad de Tailscale (`tailscale-user-login`) sin proporcionar un token/contraseña. OpenClaw verifica la identidad resolviendo la dirección `x-forwarded-for` a través del daemon local de Tailscale (`tailscale whois`) y comparándola con la cabecera antes de aceptarla. OpenClaw solo trata una solicitud como Serve cuando llega desde loopback con las cabeceras `x-forwarded-for`, `x-forwarded-proto` y `x-forwarded-host` de Tailscale. Los endpoints de la API HTTP (por ejemplo `/v1/*`, `/tools/invoke` y `/api/channels/*`) aún requieren autenticación con token/contraseña. Este flujo sin token asume que el host del gateway es de confianza. Si código local no confiable puede ejecutarse en el mismo host, deshabilita `gateway.auth.allowTailscale` y requiere autenticación con token/contraseña en su lugar. Para requerir credenciales explícitas, establece `gateway.auth.allowTailscale: false` o fuerza `gateway.auth.mode: "password"`.

## Ejemplos de configuración

### Solo tailnet (Serve)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Abrir: `https:///` (o tu `gateway.controlUi.basePath` configurado)

### Solo tailnet (vincular a IP de Tailnet)

Usa esto cuando quieras que el Gateway escuche directamente en la IP de la tailnet (sin Serve/Funnel).

```json
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

Conectar desde otro dispositivo de la tailnet:

-   UI de Control: `http://<tailscale-ip>:18789/`
-   WebSocket: `ws://<tailscale-ip>:18789`

Nota: loopback (`http://127.0.0.1:18789`) **no** funcionará en este modo.

### Internet público (Funnel + contraseña compartida)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

Prefiere `OPENCLAW_GATEWAY_PASSWORD` sobre guardar una contraseña en disco.

## Ejemplos de CLI

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## Notas

-   Tailscale Serve/Funnel requiere que la CLI `tailscale` esté instalada y con sesión iniciada.
-   `tailscale.mode: "funnel"` se niega a iniciar a menos que el modo de autenticación sea `password` para evitar exposición pública.
-   Establece `gateway.tailscale.resetOnExit` si quieres que OpenClaw deshaga la configuración de `tailscale serve` o `tailscale funnel` al apagarse.
-   `gateway.bind: "tailnet"` es un enlace directo a la tailnet (sin HTTPS, sin Serve/Funnel).
-   `gateway.bind: "auto"` prefiere loopback; usa `tailnet` si quieres solo tailnet.
-   Serve/Funnel solo exponen la **UI de control del Gateway + WS**. Los nodos se conectan a través del mismo endpoint WS del Gateway, por lo que Serve puede funcionar para acceso a nodos.

## Control desde navegador (Gateway remoto + navegador local)

Si ejecutas el Gateway en una máquina pero quieres controlar un navegador en otra máquina, ejecuta un **host de nodo** en la máquina del navegador y mantén ambos en la misma tailnet. El Gateway enviará por proxy las acciones del navegador al nodo; no se necesita un servidor de control separado ni una URL de Serve. Evita Funnel para el control del navegador; trata el emparejamiento de nodos como acceso de operador.

## Prerrequisitos y límites de Tailscale

-   Serve requiere HTTPS habilitado para tu tailnet; la CLI lo solicita si falta.
-   Serve inyecta cabeceras de identidad de Tailscale; Funnel no.
-   Funnel requiere Tailscale v1.38.3+, MagicDNS, HTTPS habilitado y un atributo de nodo funnel.
-   Funnel solo admite los puertos `443`, `8443` y `10000` sobre TLS.
-   Funnel en macOS requiere la variante de aplicación de Tailscale de código abierto.

## Aprende más

-   Descripción general de Tailscale Serve: [https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
-   Comando `tailscale serve`: [https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
-   Descripción general de Tailscale Funnel: [https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
-   Comando `tailscale funnel`: [https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)

[Configuración de Gateway Remoto](./remote-gateway-readme.md)[Verificación Formal (Modelos de Seguridad)](../security/formal-verification.md)