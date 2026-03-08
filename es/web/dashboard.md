

  Interfaces web

  
# Panel de control

El panel de control del Gateway es la Interfaz de Control del navegador servida en `/` por defecto (se puede sobrescribir con `gateway.controlUi.basePath`). Apertura rápida (Gateway local):

-   [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (o [http://localhost:18789/](http://localhost:18789/))

Referencias clave:

-   [Interfaz de Control](./control-ui.md) para uso y capacidades de la UI.
-   [Tailscale](../gateway/tailscale.md) para automatización de Serve/Funnel.
-   [Superficies web](../web.md) para modos de enlace y notas de seguridad.

La autenticación se aplica en el handshake del WebSocket a través de `connect.params.auth` (token o contraseña). Consulta `gateway.auth` en la [configuración del Gateway](../gateway/configuration.md). Nota de seguridad: la Interfaz de Control es una **superficie de administración** (chat, configuración, aprobaciones de ejecución). No la expongas públicamente. La UI mantiene los tokens de la URL del panel en memoria para la pestaña actual y los elimina de la URL después de la carga. Prefiere localhost, Tailscale Serve o un túnel SSH.

## Ruta rápida (recomendado)

-   Después de la incorporación, la CLI abre automáticamente el panel e imprime un enlace limpio (sin token).
-   Reabrir en cualquier momento: `openclaw dashboard` (copia el enlace, abre el navegador si es posible, muestra una pista SSH si es headless).
-   Si la UI solicita autenticación, pega el token de `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`) en la configuración de la Interfaz de Control.

## Conceptos básicos del token (local vs remoto)

-   **Localhost**: abre `http://127.0.0.1:18789/`.
-   **Fuente del token**: `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`); `openclaw dashboard` puede pasarlo a través del fragmento de URL para un arranque único, pero la Interfaz de Control no persiste los tokens del gateway en localStorage.
-   Si `gateway.auth.token` está gestionado por SecretRef, `openclaw dashboard` imprime/copia/abre una URL sin token por diseño. Esto evita exponer tokens gestionados externamente en registros de shell, historial del portapapeles o argumentos de lanzamiento del navegador.
-   Si `gateway.auth.token` está configurado como un SecretRef y no está resuelto en tu shell actual, `openclaw dashboard` aún imprime una URL sin token más una guía de configuración de autenticación accionable.
-   **No es localhost**: usa Tailscale Serve (sin token para la Interfaz de Control/WebSocket si `gateway.auth.allowTailscale: true`, asume un host de gateway confiable; las APIs HTTP aún necesitan token/contraseña), enlace de tailnet con un token, o un túnel SSH. Consulta [Superficies web](../web.md).

## Si ves "unauthorized" / 1008

-   Asegúrate de que el gateway sea accesible (local: `openclaw status`; remoto: túnel SSH `ssh -N -L 18789:127.0.0.1:18789 user@host` luego abre `http://127.0.0.1:18789/`).
-   Recupera o proporciona el token desde el host del gateway:
    -   Configuración en texto plano: `openclaw config get gateway.auth.token`
    -   Configuración gestionada por SecretRef: resuelve el proveedor de secretos externo o exporta `OPENCLAW_GATEWAY_TOKEN` en este shell, luego ejecuta de nuevo `openclaw dashboard`
    -   No hay token configurado: `openclaw doctor --generate-gateway-token`
-   En la configuración del panel, pega el token en el campo de autenticación, luego conéctate.

[Interfaz de Control](./control-ui.md)[WebChat](./webchat.md)

---