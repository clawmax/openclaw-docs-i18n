

  Redes y descubrimiento

  
# Descubrimiento Bonjour

OpenClaw usa Bonjour (mDNS / DNS‑SD) como una **conveniencia solo para LAN** para descubrir un Gateway activo (endpoint WebSocket). Es de mejor esfuerzo y **no** reemplaza la conectividad basada en SSH o Tailnet.

## Bonjour de área amplia (DNS‑SD Unicast) sobre Tailscale

Si el nodo y el gateway están en redes diferentes, el mDNS multicast no cruzará el límite. Puedes mantener la misma experiencia de usuario de descubrimiento cambiando a **DNS‑SD unicast** ("Bonjour de Área Amplia") sobre Tailscale. Pasos generales:

1.  Ejecuta un servidor DNS en el host del gateway (accesible a través de Tailnet).
2.  Publica registros DNS‑SD para `_openclaw-gw._tcp` bajo una zona dedicada (ejemplo: `openclaw.internal.`).
3.  Configura **DNS dividido** de Tailscale para que tu dominio elegido se resuelva a través de ese servidor DNS para los clientes (incluyendo iOS).

OpenClaw admite cualquier dominio de descubrimiento; `openclaw.internal.` es solo un ejemplo. Los nodos iOS/Android exploran tanto `local.` como tu dominio de área amplia configurado.

### Configuración del gateway (recomendado)

```json
{
  gateway: { bind: "tailnet" }, // solo tailnet (recomendado)
  discovery: { wideArea: { enabled: true } }, // habilita la publicación de DNS-SD de área amplia
}
```

### Configuración única del servidor DNS (host del gateway)

```bash
openclaw dns setup --apply
```

Esto instala CoreDNS y lo configura para:

-   escuchar en el puerto 53 solo en las interfaces Tailscale del gateway
-   servir tu dominio elegido (ejemplo: `openclaw.internal.`) desde `~/.openclaw/dns/.db`

Valida desde una máquina conectada a la tailnet:

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### Configuración de DNS de Tailscale

En la consola de administración de Tailscale:

-   Añade un servidor de nombres que apunte a la IP de tailnet del gateway (UDP/TCP 53).
-   Añade DNS dividido para que tu dominio de descubrimiento use ese servidor de nombres.

Una vez que los clientes acepten el DNS de tailnet, los nodos iOS pueden explorar `_openclaw-gw._tcp` en tu dominio de descubrimiento sin multicast.

### Seguridad del listener del gateway (recomendado)

El puerto WS del Gateway (por defecto `18789`) se vincula al loopback por defecto. Para acceso LAN/tailnet, vincula explícitamente y mantén la autenticación habilitada. Para configuraciones solo tailnet:

-   Establece `gateway.bind: "tailnet"` en `~/.openclaw/openclaw.json`.
-   Reinicia el Gateway (o reinicia la aplicación de la barra de menús de macOS).

## Qué anuncia

Solo el Gateway anuncia `_openclaw-gw._tcp`.

## Tipos de servicio

-   `_openclaw-gw._tcp` — baliza de transporte del gateway (usada por nodos macOS/iOS/Android).

## Claves TXT (pistas no secretas)

El Gateway anuncia pequeñas pistas no secretas para hacer los flujos de UI convenientes:

-   `role=gateway`
-   `displayName=`
-   `lanHost=.local`
-   `gatewayPort=` (Gateway WS + HTTP)
-   `gatewayTls=1` (solo cuando TLS está habilitado)
-   `gatewayTlsSha256=` (solo cuando TLS está habilitado y la huella digital está disponible)
-   `canvasPort=` (solo cuando el host del canvas está habilitado; actualmente el mismo que `gatewayPort`)
-   `sshPort=` (por defecto 22 cuando no se sobrescribe)
-   `transport=gateway`
-   `cliPath=` (opcional; ruta absoluta a un punto de entrada ejecutable `openclaw`)
-   `tailnetDns=` (pista opcional cuando Tailnet está disponible)

Notas de seguridad:

-   Los registros TXT de Bonjour/mDNS **no están autenticados**. Los clientes no deben tratar TXT como enrutamiento autoritativo.
-   Los clientes deben enrutar usando el endpoint de servicio resuelto (SRV + A/AAAA). Trata `lanHost`, `tailnetDns`, `gatewayPort` y `gatewayTlsSha256` solo como pistas.
-   La fijación TLS nunca debe permitir que un `gatewayTlsSha256` anunciado sobrescriba un pin almacenado previamente.
-   Los nodos iOS/Android deben tratar las conexiones directas basadas en descubrimiento como **solo TLS** y requerir confirmación explícita del usuario antes de confiar en una huella digital por primera vez.

## Depuración en macOS

Herramientas integradas útiles:

-   Explorar instancias:
    
    Copiar
    
    ```bash
    dns-sd -B _openclaw-gw._tcp local.
    ```
    
-   Resolver una instancia (reemplaza ``):
    
    Copiar
    
    ```bash
    dns-sd -L "<instancia>" _openclaw-gw._tcp local.
    ```
    

Si la exploración funciona pero la resolución falla, generalmente estás enfrentando una política de LAN o un problema del resolvedor mDNS.

## Depuración en los registros del Gateway

El Gateway escribe un archivo de registro rotativo (impreso al inicio como `gateway log file: ...`). Busca líneas `bonjour:`, especialmente:

-   `bonjour: advertise failed ...`
-   `bonjour: ... name conflict resolved` / `hostname conflict resolved`
-   `bonjour: watchdog detected non-announced service ...`

## Depuración en el nodo iOS

El nodo iOS usa `NWBrowser` para descubrir `_openclaw-gw._tcp`. Para capturar registros:

-   Ajustes → Gateway → Avanzado → **Registros de depuración de descubrimiento**
-   Ajustes → Gateway → Avanzado → **Registros de descubrimiento** → reproducir → **Copiar**

El registro incluye transiciones de estado del navegador y cambios en el conjunto de resultados.

## Modos de fallo comunes

-   **Bonjour no cruza redes**: usa Tailnet o SSH.
-   **Multicast bloqueado**: algunas redes Wi‑Fi deshabilitan mDNS.
-   **Suspensión / cambios de interfaz**: macOS puede descartar temporalmente resultados mDNS; reintenta.
-   **La exploración funciona pero la resolución falla**: mantén los nombres de máquina simples (evita emojis o puntuación), luego reinicia el Gateway. El nombre de instancia del servicio se deriva del nombre del host, por lo que nombres excesivamente complejos pueden confundir a algunos resolvedores.

## Nombres de instancia escapados (\\032)

Bonjour/DNS‑SD a menudo escapa bytes en nombres de instancia de servicio como secuencias decimales `\DDD` (por ejemplo, los espacios se convierten en `\032`).

-   Esto es normal a nivel de protocolo.
-   Las UIs deben decodificar para mostrar (iOS usa `BonjourEscapes.decode`).

## Deshabilitar / configuración

-   `OPENCLAW_DISABLE_BONJOUR=1` deshabilita la publicidad (heredado: `OPENCLAW_DISABLE_BONJOUR`).
-   `gateway.bind` en `~/.openclaw/openclaw.json` controla el modo de vinculación del Gateway.
-   `OPENCLAW_SSH_PORT` sobrescribe el puerto SSH anunciado en TXT (heredado: `OPENCLAW_SSH_PORT`).
-   `OPENCLAW_TAILNET_DNS` publica una pista MagicDNS en TXT (heredado: `OPENCLAW_TAILNET_DNS`).
-   `OPENCLAW_CLI_PATH` sobrescribe la ruta CLI anunciada (heredado: `OPENCLAW_CLI_PATH`).

## Documentación relacionada

-   Política de descubrimiento y selección de transporte: [Descubrimiento](./discovery.md)
-   Emparejamiento de nodos + aprobaciones: [Emparejamiento de Gateway](./pairing.md)

[Descubrimiento y Transportes](./discovery.md)[Acceso Remoto](./remote.md)