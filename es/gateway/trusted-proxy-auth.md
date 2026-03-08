

  Configuración y operaciones

  
# Autenticación de proxy confiable

> ⚠️ **Característica sensible a la seguridad.** Este modo delega la autenticación completamente a tu proxy inverso. Una mala configuración puede exponer tu Gateway a acceso no autorizado. Lee esta página cuidadosamente antes de habilitarlo.

## Cuándo usar

Usa el modo de autenticación `trusted-proxy` cuando:

-   Ejecutas OpenClaw detrás de un **proxy con identidad** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
-   Tu proxy maneja toda la autenticación y pasa la identidad del usuario mediante cabeceras
-   Estás en un entorno Kubernetes o de contenedores donde el proxy es la única ruta hacia el Gateway
-   Encuentras errores WebSocket `1008 unauthorized` porque los navegadores no pueden pasar tokens en la carga útil de WS

## Cuándo NO usar

-   Si tu proxy no autentica usuarios (solo es un terminador TLS o balanceador de carga)
-   Si existe alguna ruta hacia el Gateway que evite el proxy (agujeros en el firewall, acceso a la red interna)
-   Si no estás seguro de si tu proxy elimina/sobrescribe correctamente las cabeceras reenviadas
-   Si solo necesitas acceso personal de un solo usuario (considera Tailscale Serve + loopback para una configuración más simple)

## Cómo funciona

1.  Tu proxy inverso autentica a los usuarios (OAuth, OIDC, SAML, etc.)
2.  El proxy agrega una cabecera con la identidad del usuario autenticado (ej., `x-forwarded-user: nick@example.com`)
3.  OpenClaw verifica que la solicitud provino de una **IP de proxy confiable** (configurada en `gateway.trustedProxies`)
4.  OpenClaw extrae la identidad del usuario de la cabecera configurada
5.  Si todo verifica, la solicitud está autorizada

## Comportamiento del emparejamiento de la UI de Control

Cuando `gateway.auth.mode = "trusted-proxy"` está activo y la solicitud pasa las verificaciones de proxy confiable, las sesiones WebSocket de la UI de Control pueden conectarse sin la identidad de emparejamiento de dispositivo. Implicaciones:

-   El emparejamiento ya no es la puerta principal para el acceso a la UI de Control en este modo.
-   Tu política de autenticación del proxy inverso y `allowUsers` se convierten en el control de acceso efectivo.
-   Mantén el ingreso del Gateway bloqueado solo a las IPs del proxy confiable (`gateway.trustedProxies` + firewall).

## Configuración

```json
{
  gateway: {
    // Usa loopback para configuraciones de proxy en el mismo host; usa lan/custom para hosts proxy remotos
    bind: "loopback",

    // CRÍTICO: Solo agrega la(s) IP(s) de tu proxy aquí
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Cabecera que contiene la identidad del usuario autenticado (requerido)
        userHeader: "x-forwarded-user",

        // Opcional: cabeceras que DEBEN estar presentes (verificación del proxy)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Opcional: restringir a usuarios específicos (vacío = permitir todos)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

Si `gateway.bind` es `loopback`, incluye una dirección de proxy de loopback en `gateway.trustedProxies` (`127.0.0.1`, `::1`, o un CIDR de loopback equivalente).

### Referencia de configuración

| Campo | Requerido | Descripción |
| --- | --- | --- |
| `gateway.trustedProxies` | Sí | Arreglo de direcciones IP de proxy en las que confiar. Las solicitudes de otras IPs son rechazadas. |
| `gateway.auth.mode` | Sí | Debe ser `"trusted-proxy"` |
| `gateway.auth.trustedProxy.userHeader` | Sí | Nombre de la cabecera que contiene la identidad del usuario autenticado |
| `gateway.auth.trustedProxy.requiredHeaders` | No | Cabeceras adicionales que deben estar presentes para que la solicitud sea confiable |
| `gateway.auth.trustedProxy.allowUsers` | No | Lista de permitidos de identidades de usuario. Vacío significa permitir todos los usuarios autenticados. |

## Terminación TLS y HSTS

Usa un punto de terminación TLS y aplica HSTS allí.

### Patrón recomendado: terminación TLS en el proxy

Cuando tu proxy inverso maneja HTTPS para `https://control.example.com`, configura `Strict-Transport-Security` en el proxy para ese dominio.

-   Buen ajuste para despliegues orientados a internet.
-   Mantiene el certificado + la política de endurecimiento HTTP en un solo lugar.
-   OpenClaw puede permanecer en HTTP de loopback detrás del proxy.

Valor de cabecera de ejemplo:

```yaml
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### Terminación TLS en el Gateway

Si OpenClaw mismo sirve HTTPS directamente (sin proxy que termine TLS), configura:

```json
{
  gateway: {
    tls: { enabled: true },
    http: {
      securityHeaders: {
        strictTransportSecurity: "max-age=31536000; includeSubDomains",
      },
    },
  },
}
```

`strictTransportSecurity` acepta un valor de cabecera de cadena, o `false` para deshabilitar explícitamente.

### Guía de implementación

-   Comienza primero con una edad máxima corta (por ejemplo `max-age=300`) mientras validas el tráfico.
-   Incrementa a valores de larga duración (por ejemplo `max-age=31536000`) solo después de tener alta confianza.
-   Agrega `includeSubDomains` solo si cada subdominio está listo para HTTPS.
-   Usa preload solo si cumples intencionalmente con los requisitos de preload para todo tu conjunto de dominios.
-   El desarrollo local solo con loopback no se beneficia de HSTS.

## Ejemplos de configuración de proxy

### Pomerium

Pomerium pasa la identidad en `x-pomerium-claim-email` (u otras cabeceras de claim) y un JWT en `x-pomerium-jwt-assertion`.

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP de Pomerium
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

Fragmento de configuración de Pomerium:

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy con OAuth

Caddy con el plugin `caddy-security` puede autenticar usuarios y pasar cabeceras de identidad.

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // IP de Caddy (si está en el mismo host)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Fragmento de Caddyfile:

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

oauth2-proxy autentica usuarios y pasa la identidad en `x-auth-request-email`.

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP de nginx/oauth2-proxy
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

Fragmento de configuración de nginx:

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik con Forward Auth

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // IP del contenedor Traefik
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## Lista de verificación de seguridad

Antes de habilitar la autenticación trusted-proxy, verifica:

-   [ ]  **El proxy es la única ruta**: El puerto del Gateway está protegido por firewall de todo excepto de tu proxy
-   [ ]  **trustedProxies es mínimo**: Solo las IPs reales de tu proxy, no subredes enteras
-   [ ]  **El proxy elimina cabeceras**: Tu proxy sobrescribe (no agrega) las cabeceras `x-forwarded-*` de los clientes
-   [ ]  **Terminación TLS**: Tu proxy maneja TLS; los usuarios se conectan vía HTTPS
-   [ ]  **allowUsers está configurado** (recomendado): Restringe a usuarios conocidos en lugar de permitir a cualquier persona autenticada

## Auditoría de seguridad

`openclaw security audit` marcará la autenticación trusted-proxy con un hallazgo de severidad **crítica**. Esto es intencional — es un recordatorio de que estás delegando la seguridad a tu configuración de proxy. La auditoría verifica:

-   Configuración faltante de `trustedProxies`
-   Configuración faltante de `userHeader`
-   `allowUsers` vacío (permite cualquier usuario autenticado)

## Resolución de problemas

### ”trusted\_proxy\_untrusted\_source”

La solicitud no provino de una IP en `gateway.trustedProxies`. Verifica:

-   ¿Es correcta la IP del proxy? (Las IPs de contenedores Docker pueden cambiar)
-   ¿Hay un balanceador de carga delante de tu proxy?
-   Usa `docker inspect` o `kubectl get pods -o wide` para encontrar las IPs reales

### ”trusted\_proxy\_user\_missing”

La cabecera de usuario estaba vacía o faltaba. Verifica:

-   ¿Está tu proxy configurado para pasar cabeceras de identidad?
-   ¿Es correcto el nombre de la cabecera? (no sensible a mayúsculas, pero la ortografía importa)
-   ¿Está el usuario realmente autenticado en el proxy?

### “trustedproxy\_missing\_header\*”

Una cabecera requerida no estaba presente. Verifica:

-   Tu configuración de proxy para esas cabeceras específicas
-   Si las cabeceras están siendo eliminadas en algún punto de la cadena

### ”trusted\_proxy\_user\_not\_allowed”

El usuario está autenticado pero no está en `allowUsers`. O agrégalo o elimina la lista de permitidos.

### WebSocket aún fallando

Asegúrate de que tu proxy:

-   Soporte actualizaciones WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
-   Pase las cabeceras de identidad en las solicitudes de actualización WebSocket (no solo HTTP)
-   No tenga una ruta de autenticación separada para conexiones WebSocket

## Migración desde autenticación por token

Si estás migrando de autenticación por token a trusted-proxy:

1.  Configura tu proxy para autenticar usuarios y pasar cabeceras
2.  Prueba la configuración del proxy de forma independiente (curl con cabeceras)
3.  Actualiza la configuración de OpenClaw con autenticación trusted-proxy
4.  Reinicia el Gateway
5.  Prueba las conexiones WebSocket desde la UI de Control
6.  Ejecuta `openclaw security audit` y revisa los hallazgos

## Relacionado

-   [Seguridad](./security.md) — guía completa de seguridad
-   [Configuración](./configuration.md) — referencia de configuración
-   [Acceso remoto](./remote.md) — otros patrones de acceso remoto
-   [Tailscale](./tailscale.md) — alternativa más simple para acceso solo a tailnet

[Contrato de plan de aplicación de secretos](./secrets-plan-contract.md)[Comprobaciones de salud](./health.md)