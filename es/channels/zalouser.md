

  Plataformas de mensajería

  
# Zalo Personal

Estado: experimental. Esta integración automatiza una **cuenta personal de Zalo** mediante `zca-js` nativo dentro de OpenClaw.

> **Advertencia:** Esta es una integración no oficial y puede resultar en la suspensión/bloqueo de la cuenta. Úsalo bajo tu propio riesgo.

## Plugin requerido

Zalo Personal se distribuye como un plugin y no está incluido en la instalación principal.

-   Instalar vía CLI: `openclaw plugins install @openclaw/zalouser`
-   O desde un repositorio fuente: `openclaw plugins install ./extensions/zalouser`
-   Detalles: [Plugins](../tools/plugin.md)

No se requiere ningún binario CLI externo `zca`/`openzca`.

## Configuración rápida (principiante)

1.  Instala el plugin (ver arriba).
2.  Inicia sesión (QR, en la máquina del Gateway):
    -   `openclaw channels login --channel zalouser`
    -   Escanea el código QR con la aplicación móvil de Zalo.
3.  Habilita el canal:

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

4.  Reinicia el Gateway (o finaliza el proceso de incorporación).
5.  El acceso por DM por defecto es por emparejamiento; aprueba el código de emparejamiento en el primer contacto.

## Qué es

-   Se ejecuta completamente en proceso vía `zca-js`.
-   Utiliza detectores de eventos nativos para recibir mensajes entrantes.
-   Envía respuestas directamente a través de la API JS (texto/media/enlace).
-   Diseñado para casos de uso de "cuenta personal" donde la API oficial de bots de Zalo no está disponible.

## Nomenclatura

El id del canal es `zalouser` para dejar claro que automatiza una **cuenta de usuario personal de Zalo** (no oficial). Reservamos `zalo` para una posible futura integración con la API oficial de Zalo.

## Encontrar IDs (directorio)

Usa la CLI de directorio para descubrir contactos/grupos y sus IDs:

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

## Límites

-   El texto saliente se divide en fragmentos de ~2000 caracteres (límites del cliente de Zalo).
-   El streaming está bloqueado por defecto.

## Control de acceso (DMs)

`channels.zalouser.dmPolicy` admite: `pairing | allowlist | open | disabled` (por defecto: `pairing`). `channels.zalouser.allowFrom` acepta IDs de usuario o nombres. Durante la incorporación, los nombres se resuelven a IDs usando la búsqueda de contactos en proceso del plugin. Aprueba mediante:

-   `openclaw pairing list zalouser`
-   `openclaw pairing approve zalouser `

## Acceso a grupos (opcional)

-   Por defecto: `channels.zalouser.groupPolicy = "open"` (grupos permitidos). Usa `channels.defaults.groupPolicy` para anular el valor por defecto cuando no esté establecido.
-   Restringe a una lista permitida con:
    -   `channels.zalouser.groupPolicy = "allowlist"`
    -   `channels.zalouser.groups` (las claves son IDs de grupo o nombres)
-   Bloquea todos los grupos: `channels.zalouser.groupPolicy = "disabled"`.
-   El asistente de configuración puede solicitar listas permitidas de grupos.
-   Al iniciar, OpenClaw resuelve los nombres de grupos/usuarios en las listas permitidas a IDs y registra el mapeo; las entradas no resueltas se mantienen como se escribieron.

Ejemplo:

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true },
      },
    },
  },
}
```

### Control de menciones en grupos

-   `channels.zalouser.groups..requireMention` controla si las respuestas en grupo requieren una mención.
-   Orden de resolución: id/nombre exacto del grupo -> slug normalizado del grupo -> `*` -> valor por defecto (`true`).
-   Esto se aplica tanto a grupos en la lista permitida como al modo de grupo abierto.

Ejemplo:

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "*": { allow: true, requireMention: true },
        "Work Chat": { allow: true, requireMention: false },
      },
    },
  },
}
```

## Multi-cuenta

Las cuentas se asignan a perfiles `zalouser` en el estado de OpenClaw. Ejemplo:

```json
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
      },
    },
  },
}
```

## Escribiendo, reacciones y acuses de recibo

-   OpenClaw envía un evento de "escribiendo" antes de enviar una respuesta (mejor esfuerzo).
-   La acción de reacción a mensajes `react` es compatible con `zalouser` en las acciones del canal.
    -   Usa `remove: true` para eliminar un emoji de reacción específico de un mensaje.
    -   Semántica de reacciones: [Reacciones](../tools/reactions.md)
-   Para mensajes entrantes que incluyen metadatos de evento, OpenClaw envía acuses de recibo de entregado y visto (mejor esfuerzo).

## Solución de problemas

**El inicio de sesión no persiste:**

-   `openclaw channels status --probe`
-   Vuelve a iniciar sesión: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`

**El nombre de la lista permitida/grupo no se resolvió:**

-   Usa IDs numéricos en `allowFrom`/`groups`, o nombres exactos de amigos/grupos.

**Actualizado desde una configuración antigua basada en CLI:**

-   Elimina cualquier suposición antigua sobre procesos externos `zca`.
-   El canal ahora se ejecuta completamente en OpenClaw sin binarios CLI externos.

[Zalo](./zalo.md)[Emparejamiento](./pairing.md)