

  Plataformas de mensajería

  
# Nostr

**Estado:** Plugin opcional (deshabilitado por defecto). Nostr es un protocolo descentralizado para redes sociales. Este canal permite a OpenClaw recibir y responder a mensajes directos cifrados (DMs) a través de NIP-04.

## Instalar (bajo demanda)

### Asistente de configuración (recomendado)

-   El asistente de configuración (`openclaw onboard`) y `openclaw channels add` listan los plugins de canal opcionales.
-   Seleccionar Nostr te pedirá que instales el plugin bajo demanda.

Instalación por defecto:

-   **Canal Dev + checkout git disponible:** usa la ruta local del plugin.
-   **Estable/Beta:** descarga desde npm.

Siempre puedes anular la elección en el prompt.

### Instalación manual

```bash
openclaw plugins install @openclaw/nostr
```

Usar un checkout local (flujos de trabajo de desarrollo):

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

Reinicia el Gateway después de instalar o habilitar plugins.

## Configuración rápida

1.  Genera un par de claves de Nostr (si es necesario):

```bash
# Usando nak
nak key generate
```

2.  Añade a la configuración:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3.  Exporta la clave:

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4.  Reinicia el Gateway.

## Referencia de configuración

| Clave | Tipo | Por defecto | Descripción |
| --- | --- | --- | --- |
| `privateKey` | string | requerida | Clave privada en formato `nsec` o hexadecimal |
| `relays` | string\[\] | `['wss://relay.damus.io', 'wss://nos.lol']` | URLs de relays (WebSocket) |
| `dmPolicy` | string | `pairing` | Política de acceso a DMs |
| `allowFrom` | string\[\] | `[]` | Claves públicas de remitentes permitidos |
| `enabled` | boolean | `true` | Habilitar/deshabilitar canal |
| `name` | string | \- | Nombre para mostrar |
| `profile` | object | \- | Metadatos de perfil NIP-01 |

## Metadatos del perfil

Los datos del perfil se publican como un evento NIP-01 `kind:0`. Puedes gestionarlos desde la Interfaz de Control (Canales -> Nostr -> Perfil) o configurarlos directamente en la configuración. Ejemplo:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "Bot de DM de asistente personal",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

Notas:

-   Las URLs del perfil deben usar `https://`.
-   La importación desde relays fusiona los campos y preserva las anulaciones locales.

## Control de acceso

### Políticas de DM

-   **pairing** (por defecto): los remitentes desconocidos reciben un código de emparejamiento.
-   **allowlist**: solo las claves públicas en `allowFrom` pueden enviar DMs.
-   **open**: DMs entrantes públicos (requiere `allowFrom: ["*"]`).
-   **disabled**: ignorar DMs entrantes.

### Ejemplo de lista de permitidos

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```

## Formatos de clave

Formatos aceptados:

-   **Clave privada:** `nsec...` o hexadecimal de 64 caracteres
-   **Claves públicas (`allowFrom`):** `npub...` o hexadecimal

## Relays

Por defecto: `relay.damus.io` y `nos.lol`.

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["wss://relay.damus.io", "wss://relay.primal.net", "wss://nostr.wine"]
    }
  }
}
```

Consejos:

-   Usa 2-3 relays para redundancia.
-   Evita demasiados relays (latencia, duplicación).
-   Los relays de pago pueden mejorar la fiabilidad.
-   Los relays locales son adecuados para pruebas (`ws://localhost:7777`).

## Soporte de protocolo

| NIP | Estado | Descripción |
| --- | --- | --- |
| NIP-01 | Soportado | Formato básico de evento + metadatos de perfil |
| NIP-04 | Soportado | DMs cifrados (`kind:4`) |
| NIP-17 | Planeado | DMs envueltos como regalo |
| NIP-44 | Planeado | Cifrado versionado |

## Pruebas

### Relay local

```bash
# Iniciar strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```

### Prueba manual

1.  Anota la clave pública del bot (npub) de los registros.
2.  Abre un cliente de Nostr (Damus, Amethyst, etc.).
3.  Envía un DM a la clave pública del bot.
4.  Verifica la respuesta.

## Solución de problemas

### No se reciben mensajes

-   Verifica que la clave privada sea válida.
-   Asegúrate de que las URLs de los relays sean accesibles y usen `wss://` (o `ws://` para local).
-   Confirma que `enabled` no sea `false`.
-   Revisa los registros del Gateway en busca de errores de conexión al relay.

### No se envían respuestas

-   Comprueba que el relay acepte escrituras.
-   Verifica la conectividad saliente.
-   Vigila los límites de tasa del relay.

### Respuestas duplicadas

-   Es esperado cuando se usan múltiples relays.
-   Los mensajes se deduplican por ID de evento; solo la primera entrega desencadena una respuesta.

## Seguridad

-   Nunca comprometas claves privadas.
-   Usa variables de entorno para las claves.
-   Considera `allowlist` para bots en producción.

## Limitaciones (MVP)

-   Solo mensajes directos (sin chats grupales).
-   Sin archivos adjuntos multimedia.
-   Solo NIP-04 (NIP-17 gift-wrap planeado).

[Nextcloud Talk](./nextcloud-talk.md)[Signal](./signal.md)