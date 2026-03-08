

  Redes y descubrimiento

  
# Modelo de red

La mayoría de las operaciones fluyen a través del Gateway (`openclaw gateway`), un único proceso de larga duración que posee las conexiones de canales y el plano de control WebSocket.

## Reglas fundamentales

-   Se recomienda un Gateway por host. Es el único proceso autorizado para poseer la sesión de WhatsApp Web. Para bots de rescate o aislamiento estricto, ejecuta múltiples gateways con perfiles y puertos aislados. Consulta [Múltiples gateways](./multiple-gateways.md).
-   Primero el bucle local (loopback): el WS del Gateway por defecto es `ws://127.0.0.1:18789`. El asistente genera un token de gateway por defecto, incluso para loopback. Para acceso desde la tailnet, ejecuta `openclaw gateway --bind tailnet --token ...` porque los tokens son obligatorios para enlaces no locales.
-   Los nodos se conectan al WS del Gateway a través de LAN, tailnet o SSH según sea necesario. El puente TCP heredado está obsoleto.
-   El host del canvas es servido por el servidor HTTP del Gateway en el **mismo puerto** que el Gateway (por defecto `18789`):
    -   `/__openclaw__/canvas/`
    -   `/__openclaw__/a2ui/` Cuando `gateway.auth` está configurado y el Gateway se enlaza más allá del loopback, estas rutas están protegidas por la autenticación del Gateway. Los clientes nodo usan URLs de capacidad con alcance de nodo vinculadas a su sesión WS activa. Consulta [Configuración del Gateway](./configuration.md) (`canvasHost`, `gateway`).
-   El uso remoto es típicamente un túnel SSH o una VPN tailnet. Consulta [Acceso remoto](./remote.md) y [Descubrimiento](./discovery.md).

[Modelos locales](./local-models.md)[Emparejamiento propiedad del Gateway](./pairing.md)

---