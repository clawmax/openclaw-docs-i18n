

  Protocolos y APIs

  
# Protocolo Gateway

El protocolo Gateway WS es el **único plano de control + transporte de nodos** para OpenClaw. Todos los clientes (CLI, interfaz web, aplicación macOS, nodos iOS/Android, nodos headless) se conectan a través de WebSocket y declaran su **rol** + **alcance** en el momento del handshake.

## Transporte

-   WebSocket, tramas de texto con cargas útiles JSON.
-   La primera trama **debe** ser una solicitud `connect`.

## Handshake (connect)

Gateway → Cliente (desafío pre-conexión):

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

Cliente → Gateway:

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

Gateway → Cliente:

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

Cuando se emite un token de dispositivo, `hello-ok` también incluye:

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```

### Ejemplo de nodo

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

## Estructura de tramas

-   **Solicitud**: `{type:"req", id, method, params}`
-   **Respuesta**: `{type:"res", id, ok, payload|error}`
-   **Evento**: `{type:"event", event, payload, seq?, stateVersion?}`

Los métodos con efectos secundarios requieren **claves de idempotencia** (ver esquema).

## Roles + alcances

### Roles

-   `operator` = cliente del plano de control (CLI/UI/automatización).
-   `node` = host de capacidades (cámara/pantalla/canvas/system.run).

### Alcances (operador)

Alcances comunes:

-   `operator.read`
-   `operator.write`
-   `operator.admin`
-   `operator.approvals`
-   `operator.pairing`

El alcance del método es solo la primera puerta. Algunos comandos de barra diagonal accedidos a través de `chat.send` aplican comprobaciones más estrictas a nivel de comando además. Por ejemplo, las escrituras persistentes `/config set` y `/config unset` requieren `operator.admin`.

### Capacidades/comandos/permisos (nodo)

Los nodos declaran afirmaciones de capacidad en el momento de conexión:

-   `caps`: categorías de capacidades de alto nivel.
-   `commands`: lista de comandos permitidos para invocar.
-   `permissions`: interruptores granulares (ej. `screen.record`, `camera.capture`).

El Gateway trata estos como **afirmaciones** y aplica listas de permisos del lado del servidor.

## Presencia

-   `system-presence` devuelve entradas indexadas por identidad del dispositivo.
-   Las entradas de presencia incluyen `deviceId`, `roles` y `scopes` para que las UIs puedan mostrar una sola fila por dispositivo incluso cuando se conecta como **operador** y **nodo**.

### Métodos auxiliares de nodo

-   Los nodos pueden llamar a `skills.bins` para obtener la lista actual de ejecutables de habilidades para comprobaciones de auto-permiso.

### Métodos auxiliares de operador

-   Los operadores pueden llamar a `tools.catalog` (`operator.read`) para obtener el catálogo de herramientas en tiempo de ejecución para un agente. La respuesta incluye herramientas agrupadas y metadatos de procedencia:
    -   `source`: `core` o `plugin`
    -   `pluginId`: propietario del plugin cuando `source="plugin"`
    -   `optional`: si una herramienta de plugin es opcional

## Aprobaciones de ejecución

-   Cuando una solicitud de ejecución necesita aprobación, el gateway transmite `exec.approval.requested`.
-   Los clientes operador resuelven llamando a `exec.approval.resolve` (requiere alcance `operator.approvals`).
-   Para `host=node`, `exec.approval.request` debe incluir `systemRunPlan` (`argv`/`cwd`/`rawCommand`/metadatos de sesión canónicos). Las solicitudes que carezcan de `systemRunPlan` son rechazadas.

## Control de versiones

-   `PROTOCOL_VERSION` reside en `src/gateway/protocol/schema.ts`.
-   Los clientes envían `minProtocol` + `maxProtocol`; el servidor rechaza desajustes.
-   Los esquemas + modelos se generan a partir de definiciones TypeBox:
    -   `pnpm protocol:gen`
    -   `pnpm protocol:gen:swift`
    -   `pnpm protocol:check`

## Autenticación

-   Si `OPENCLAW_GATEWAY_TOKEN` (o `--token`) está configurado, `connect.params.auth.token` debe coincidir o el socket se cierra.
-   Después del emparejamiento, el Gateway emite un **token de dispositivo** con alcance al rol + alcances de la conexión. Se devuelve en `hello-ok.auth.deviceToken` y debe ser persistido por el cliente para futuras conexiones.
-   Los tokens de dispositivo pueden rotarse/revocarse mediante `device.token.rotate` y `device.token.revoke` (requiere alcance `operator.pairing`).

## Identidad del dispositivo + emparejamiento

-   Los nodos deben incluir una identidad de dispositivo estable (`device.id`) derivada de una huella de par de claves.
-   Los Gateways emiten tokens por dispositivo + rol.
-   Se requieren aprobaciones de emparejamiento para nuevos IDs de dispositivo a menos que esté habilitada la auto-aprobación local.
-   Las conexiones **locales** incluyen loopback y la dirección tailnet del propio host del gateway (para que los enlaces tailnet del mismo host aún puedan auto-aprobarse).
-   Todos los clientes WS deben incluir identidad `device` durante `connect` (operador + nodo). La UI de control puede omitirla **solo** cuando `gateway.controlUi.dangerouslyDisableDeviceAuth` está habilitado para uso de emergencia.
-   Todas las conexiones deben firmar el nonce `connect.challenge` proporcionado por el servidor.

### Diagnóstico de migración de autenticación de dispositivo

Para clientes heredados que aún usan el comportamiento de firma pre-desafío, `connect` ahora devuelve códigos de detalle `DEVICE_AUTH_*` bajo `error.details.code` con un `error.details.reason` estable. Fallos de migración comunes:

| Mensaje | details.code | details.reason | Significado |
| --- | --- | --- | --- |
| `device nonce required` | `DEVICE_AUTH_NONCE_REQUIRED` | `device-nonce-missing` | El cliente omitió `device.nonce` (o lo envió vacío). |
| `device nonce mismatch` | `DEVICE_AUTH_NONCE_MISMATCH` | `device-nonce-mismatch` | El cliente firmó con un nonce obsoleto/incorrecto. |
| `device signature invalid` | `DEVICE_AUTH_SIGNATURE_INVALID` | `device-signature` | La carga útil de la firma no coincide con la carga útil v2. |
| `device signature expired` | `DEVICE_AUTH_SIGNATURE_EXPIRED` | `device-signature-stale` | La marca de tiempo firmada está fuera del margen permitido. |
| `device identity mismatch` | `DEVICE_AUTH_DEVICE_ID_MISMATCH` | `device-id-mismatch` | `device.id` no coincide con la huella de la clave pública. |
| `device public key invalid` | `DEVICE_AUTH_PUBLIC_KEY_INVALID` | `device-public-key` | Falló el formato/canonicalización de la clave pública. |

Objetivo de migración:

-   Esperar siempre a `connect.challenge`.
-   Firmar la carga útil v2 que incluye el nonce del servidor.
-   Enviar el mismo nonce en `connect.params.device.nonce`.
-   La carga útil de firma preferida es `v3`, que vincula `platform` y `deviceFamily` además de los campos device/client/role/scopes/token/nonce.
-   Las firmas heredadas `v2` siguen siendo aceptadas por compatibilidad, pero el anclaje de metadatos de dispositivo emparejado aún controla la política de comandos al reconectar.

## TLS + anclaje

-   Se admite TLS para conexiones WS.
-   Los clientes pueden opcionalmente anclar la huella del certificado del gateway (ver configuración `gateway.tls` más `gateway.remote.tlsFingerprint` o CLI `--tls-fingerprint`).

## Alcance

Este protocolo expone la **API completa del gateway** (estado, canales, modelos, chat, agente, sesiones, nodos, aprobaciones, etc.). La superficie exacta está definida por los esquemas TypeBox en `src/gateway/protocol/schema.ts`.

[Sandbox vs Política de Herramientas vs Elevado](./sandbox-vs-tool-policy-vs-elevated.md)[Protocolo Bridge](./bridge-protocol.md)