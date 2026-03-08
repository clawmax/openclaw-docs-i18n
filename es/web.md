

  Interfaces web

  
# Web

El Gateway sirve una pequeña **Control UI para navegador** (Vite + Lit) desde el mismo puerto que el WebSocket del Gateway:

-   por defecto: `http://:18789/`
-   prefijo opcional: establece `gateway.controlUi.basePath` (ej. `/openclaw`)

Las capacidades viven en [Control UI](./web/control-ui.md). Esta página se centra en los modos de enlace, seguridad y superficies orientadas a la web.

## Webhooks

Cuando `hooks.enabled=true`, el Gateway también expone un pequeño endpoint de webhook en el mismo servidor HTTP. Consulta [Configuración del Gateway](./gateway/configuration.md) → `hooks` para autenticación + cargas útiles.

## Configuración (activado por defecto)

La Control UI está **activada por defecto** cuando los recursos están presentes (`dist/control-ui`). Puedes controlarla mediante configuración:

```json
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath opcional
  },
}
```

## Acceso Tailscale

### Servir Integrado (recomendado)

Mantén el Gateway en loopback y deja que Tailscale Serve lo proxie:

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Luego inicia el gateway:

```bash
openclaw gateway
```

Abre:

-   `https:///` (o tu `gateway.controlUi.basePath` configurado)

### Enlace Tailnet + token

```json
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

Luego inicia el gateway (se requiere token para enlaces que no sean loopback):

```bash
openclaw gateway
```

Abre:

-   `http://<tailscale-ip>:18789/` (o tu `gateway.controlUi.basePath` configurado)

### Internet público (Funnel)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // o OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## Notas de seguridad

-   La autenticación del Gateway es requerida por defecto (token/contraseña o cabeceras de identidad de Tailscale).
-   Los enlaces que no sean loopback aún **requieren** un token/contraseña compartido (`gateway.auth` o variable de entorno).
-   El asistente genera un token de gateway por defecto (incluso en loopback).
-   La UI envía `connect.params.auth.token` o `connect.params.auth.password`.
-   Para despliegues de Control UI que no sean loopback, establece `gateway.controlUi.allowedOrigins` explícitamente (orígenes completos). Sin esto, el inicio del gateway se rechaza por defecto.
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` habilita el modo de fallback de origen por cabecera Host, pero es una degradación de seguridad peligrosa.
-   Con Serve, las cabeceras de identidad de Tailscale pueden satisfacer la autenticación de Control UI/WebSocket cuando `gateway.auth.allowTailscale` es `true` (no se requiere token/contraseña). Los endpoints de la API HTTP aún requieren token/contraseña. Establece `gateway.auth.allowTailscale: false` para requerir credenciales explícitas. Consulta [Tailscale](./gateway/tailscale.md) y [Seguridad](./gateway/security.md). Este flujo sin token asume que el host del gateway es confiable.
-   `gateway.tailscale.mode: "funnel"` requiere `gateway.auth.mode: "password"` (contraseña compartida).

## Construyendo la UI

El Gateway sirve archivos estáticos desde `dist/control-ui`. Construyelos con:

```bash
pnpm ui:build # instala automáticamente las dependencias de la UI en la primera ejecución
```

[CONTRIBUCIÓN MODELO DE AMENAZAS](./security/CONTRIBUTING-THREAT-MODEL.md)[Control UI](./web/control-ui.md)