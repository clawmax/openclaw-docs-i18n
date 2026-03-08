

  Redes y descubrimiento

  
# Descubrimiento y Transportes

OpenClaw tiene dos problemas distintos que parecen similares en la superficie:

1.  **Control remoto del operador**: la aplicación de la barra de menú de macOS controlando un gateway que se ejecuta en otro lugar.
2.  **Emparejamiento de nodos**: iOS/Android (y futuros nodos) encontrando un gateway y emparejándose de forma segura.

El objetivo de diseño es mantener todo el descubrimiento/anuncio de red en el **Node Gateway** (`openclaw gateway`) y mantener a los clientes (aplicación mac, iOS) como consumidores.

## Términos

-   **Gateway**: un único proceso de gateway de larga duración que posee el estado (sesiones, emparejamiento, registro de nodos) y ejecuta canales. La mayoría de las configuraciones usan uno por host; son posibles configuraciones multi-gateway aisladas.
-   **Gateway WS (plano de control)**: el endpoint WebSocket en `127.0.0.1:18789` por defecto; puede vincularse a LAN/tailnet mediante `gateway.bind`.
-   **Transporte WS directo**: un endpoint Gateway WS orientado a LAN/tailnet (sin SSH).
-   **Transporte SSH (respaldo)**: control remoto reenviando `127.0.0.1:18789` a través de SSH.
-   **Puente TCP heredado (obsoleto/eliminado)**: transporte de nodo antiguo (ver [Protocolo de puente](./bridge-protocol.md)); ya no se anuncia para descubrimiento.

Detalles del protocolo:

-   [Protocolo de gateway](./protocol.md)
-   [Protocolo de puente (heredado)](./bridge-protocol.md)

## Por qué mantenemos tanto "directo" como SSH

-   **WS directo** es la mejor experiencia de usuario en la misma red y dentro de una tailnet:
    -   auto-descubrimiento en LAN vía Bonjour
    -   tokens de emparejamiento + ACLs propiedad del gateway
    -   no se requiere acceso a shell; la superficie del protocolo puede mantenerse ajustada y auditable
-   **SSH** sigue siendo el respaldo universal:
    -   funciona en cualquier lugar donde tengas acceso SSH (incluso a través de redes no relacionadas)
    -   sobrevive a problemas de multicast/mDNS
    -   no requiere nuevos puertos de entrada además de SSH

## Entradas de descubrimiento (cómo los clientes aprenden dónde está el gateway)

### 1) Bonjour / mDNS (solo LAN)

Bonjour es de mejor esfuerzo y no cruza redes. Solo se usa por conveniencia en la "misma LAN". Dirección objetivo:

-   El **gateway** anuncia su endpoint WS vía Bonjour.
-   Los clientes exploran y muestran una lista "elegir un gateway", luego almacenan el endpoint elegido.

Solución de problemas y detalles del beacon: [Bonjour](./bonjour.md).

#### Detalles del beacon de servicio

-   Tipos de servicio:
    -   `_openclaw-gw._tcp` (beacon de transporte de gateway)
-   Claves TXT (no secretas):
    -   `role=gateway`
    -   `lanHost=.local`
    -   `sshPort=22` (o lo que sea anunciado)
    -   `gatewayPort=18789` (Gateway WS + HTTP)
    -   `gatewayTls=1` (solo cuando TLS está habilitado)
    -   `gatewayTlsSha256=` (solo cuando TLS está habilitado y la huella digital está disponible)
    -   `canvasPort=` (puerto del host del canvas; actualmente el mismo que `gatewayPort` cuando el host del canvas está habilitado)
    -   `cliPath=` (opcional; ruta absoluta a un punto de entrada o binario ejecutable `openclaw`)
    -   `tailnetDns=` (pista opcional; detectada automáticamente cuando Tailscale está disponible)

Notas de seguridad:

-   Los registros TXT de Bonjour/mDNS **no están autenticados**. Los clientes deben tratar los valores TXT solo como pistas para la UX.
-   El enrutamiento (host/puerto) debe preferir el **endpoint de servicio resuelto** (SRV + A/AAAA) sobre el `lanHost`, `tailnetDns` o `gatewayPort` proporcionado por TXT.
-   La fijación TLS nunca debe permitir que un `gatewayTlsSha256` anunciado anule una fijación almacenada previamente.
-   Los nodos iOS/Android deben tratar las conexiones directas basadas en descubrimiento como **solo TLS** y requerir una confirmación explícita de "confiar en esta huella digital" antes de almacenar una fijación por primera vez (verificación fuera de banda).

Deshabilitar/anular:

-   `OPENCLAW_DISABLE_BONJOUR=1` deshabilita la publicidad.
-   `gateway.bind` en `~/.openclaw/openclaw.json` controla el modo de vinculación del Gateway.
-   `OPENCLAW_SSH_PORT` anula el puerto SSH anunciado en TXT (por defecto 22).
-   `OPENCLAW_TAILNET_DNS` publica una pista `tailnetDns` (MagicDNS).
-   `OPENCLAW_CLI_PATH` anula la ruta CLI anunciada.

### 2) Tailnet (entre redes)

Para configuraciones estilo Londres/Viena, Bonjour no ayudará. El objetivo "directo" recomendado es:

-   Nombre Tailscale MagicDNS (preferido) o una IP estable de tailnet.

Si el gateway puede detectar que se está ejecutando bajo Tailscale, publica `tailnetDns` como una pista opcional para clientes (incluyendo beacons de área amplia).

### 3) Objetivo manual / SSH

Cuando no hay una ruta directa (o la directa está deshabilitada), los clientes siempre pueden conectarse vía SSH reenviando el puerto del gateway de loopback. Ver [Acceso remoto](./remote.md).

## Selección de transporte (política del cliente)

Comportamiento recomendado del cliente:

1.  Si un endpoint directo emparejado está configurado y es accesible, usarlo.
2.  De lo contrario, si Bonjour encuentra un gateway en LAN, ofrecer una opción de un toque "Usar este gateway" y guardarlo como el endpoint directo.
3.  De lo contrario, si un DNS/IP de tailnet está configurado, intentar directo.
4.  De lo contrario, recurrir a SSH.

## Emparejamiento + autenticación (transporte directo)

El gateway es la fuente de verdad para la admisión de nodos/clientes.

-   Las solicitudes de emparejamiento se crean/aprueban/rechazan en el gateway (ver [Emparejamiento de gateway](./pairing.md)).
-   El gateway aplica:
    -   autenticación (token / par de claves)
    -   alcances/ACLs (el gateway no es un proxy crudo para cada método)
    -   límites de tasa

## Responsabilidades por componente

-   **Gateway**: anuncia beacons de descubrimiento, posee decisiones de emparejamiento y aloja el endpoint WS.
-   **Aplicación macOS**: te ayuda a elegir un gateway, muestra mensajes de emparejamiento y usa SSH solo como respaldo.
-   **Nodos iOS/Android**: exploran Bonjour como conveniencia y se conectan al Gateway WS emparejado.

[Emparejamiento Propiedad del Gateway](./pairing.md)[Descubrimiento Bonjour](./bonjour.md)