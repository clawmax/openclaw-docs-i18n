

  Fundamentos

  
# Arquitectura del Gateway

Última actualización: 2026-01-22

## Descripción General

-   Un único **Gateway** de larga duración posee todas las superficies de mensajería (WhatsApp vía Baileys, Telegram vía grammY, Slack, Discord, Signal, iMessage, WebChat).
-   Los clientes del plano de control (aplicación macOS, CLI, interfaz web, automatizaciones) se conectan al Gateway a través de **WebSocket** en el host de enlace configurado (por defecto `127.0.0.1:18789`).
-   Los **Nodos** (macOS/iOS/Android/headless) también se conectan a través de **WebSocket**, pero declaran `role: node` con capacidades/comandos explícitos.
-   Un Gateway por host; es el único lugar que abre una sesión de WhatsApp.
-   El **host del canvas** es servido por el servidor HTTP del Gateway en:
    -   `/__openclaw__/canvas/` (HTML/CSS/JS editable por el agente)
    -   `/__openclaw__/a2ui/` (host A2UI) Utiliza el mismo puerto que el Gateway (por defecto `18789`).

## Componentes y flujos

### Gateway (demonio)

-   Mantiene las conexiones de los proveedores.
-   Expone una API WS tipada (solicitudes, respuestas, eventos push del servidor).
-   Valida los frames entrantes contra JSON Schema.
-   Emite eventos como `agent`, `chat`, `presence`, `health`, `heartbeat`, `cron`.

### Clientes (aplicación macOS / CLI / administrador web)

-   Una conexión WS por cliente.
-   Envían solicitudes (`health`, `status`, `send`, `agent`, `system-presence`).
-   Se suscriben a eventos (`tick`, `agent`, `presence`, `shutdown`).

### Nodos (macOS / iOS / Android / headless)

-   Se conectan al **mismo servidor WS** con `role: node`.
-   Proporcionan una identidad de dispositivo en `connect`; el emparejamiento es **basado en dispositivo** (rol `node`) y la aprobación reside en el almacén de emparejamiento del dispositivo.
-   Exponen comandos como `canvas.*`, `camera.*`, `screen.record`, `location.get`.

Detalles del protocolo:

-   [Protocolo del Gateway](../gateway/protocol.md)

### WebChat

-   Interfaz de usuario estática que utiliza la API WS del Gateway para el historial de chat y envíos.
-   En configuraciones remotas, se conecta a través del mismo túnel SSH/Tailscale que otros clientes.

## Ciclo de vida de la conexión (cliente único)

## Protocolo de comunicación (resumen)

-   Transporte: WebSocket, frames de texto con cargas útiles JSON.
-   El primer frame **debe** ser `connect`.
-   Después del handshake:
    -   Solicitudes: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
    -   Eventos: `{type:"event", event, payload, seq?, stateVersion?}`
-   Si `OPENCLAW_GATEWAY_TOKEN` (o `--token`) está configurado, `connect.params.auth.token` debe coincidir o el socket se cierra.
-   Se requieren claves de idempotencia para métodos con efectos secundarios (`send`, `agent`) para reintentos seguros; el servidor mantiene una caché de deduplicación de corta duración.
-   Los nodos deben incluir `role: "node"` más capacidades/comandos/permisos en `connect`.

## Emparejamiento + confianza local

-   Todos los clientes WS (operadores + nodos) incluyen una **identidad de dispositivo** en `connect`.
-   Los nuevos IDs de dispositivo requieren aprobación de emparejamiento; el Gateway emite un **token de dispositivo** para conexiones posteriores.
-   Las conexiones **locales** (loopback o la dirección tailnet del propio host del gateway) pueden ser aprobadas automáticamente para mantener una experiencia de usuario fluida en el mismo host.
-   Todas las conexiones deben firmar el nonce `connect.challenge`.
-   La carga útil de firma `v3` también vincula `platform` + `deviceFamily`; el gateway fija los metadatos emparejados al reconectar y requiere reemparejamiento para cambios en los metadatos.
-   Las conexiones **no locales** aún requieren aprobación explícita.
-   La autenticación del Gateway (`gateway.auth.*`) aún se aplica a **todas** las conexiones, locales o remotas.

Detalles: [Protocolo del Gateway](../gateway/protocol.md), [Emparejamiento](../channels/pairing.md), [Seguridad](../gateway/security.md).

## Tipado del protocolo y generación de código

-   Los esquemas TypeBox definen el protocolo.
-   JSON Schema se genera a partir de esos esquemas.
-   Los modelos Swift se generan a partir del JSON Schema.

## Acceso remoto

-   Preferido: Tailscale o VPN.
-   Alternativa: Túnel SSH
    
    Copiar
    
    ```bash
    ssh -N -L 18789:127.0.0.1:18789 user@host
    ```
    
-   El mismo handshake + token de autenticación se aplican sobre el túnel.
-   TLS + fijación opcional se puede habilitar para WS en configuraciones remotas.

## Instantánea de operaciones

-   Iniciar: `openclaw gateway` (primer plano, registros a stdout).
-   Salud: `health` sobre WS (también incluido en `hello-ok`).
-   Supervisión: launchd/systemd para reinicio automático.

## Invariantes

-   Exactamente un Gateway controla una única sesión de Baileys por host.
-   El handshake es obligatorio; cualquier primer frame que no sea JSON o no sea `connect` resulta en un cierre forzoso.
-   Los eventos no se reproducen; los clientes deben refrescar ante brechas.

[Arquitectura de Integración Pi](../pi.md)[Tiempo de Ejecución del Agente](./agent.md)

---