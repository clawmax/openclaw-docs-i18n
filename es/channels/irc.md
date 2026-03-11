

  Plataformas de mensajería

  
# IRC

Usa IRC cuando quieras OpenClaw en canales clásicos (`#room`) y mensajes directos. IRC se distribuye como un plugin de extensión, pero se configura en la configuración principal bajo `channels.irc`.

## Inicio rápido

1.  Habilita la configuración de IRC en `~/.openclaw/openclaw.json`.
2.  Establece al menos:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3.  Inicia/reinicia el gateway:

```bash
openclaw gateway run
```

## Valores predeterminados de seguridad

-   `channels.irc.dmPolicy` tiene como valor predeterminado `"pairing"`.
-   `channels.irc.groupPolicy` tiene como valor predeterminado `"allowlist"`.
-   Con `groupPolicy="allowlist"`, establece `channels.irc.groups` para definir los canales permitidos.
-   Usa TLS (`channels.irc.tls=true`) a menos que aceptes intencionalmente el transporte en texto plano.

## Control de acceso

Hay dos "puertas" separadas para los canales de IRC:

1.  **Acceso al canal** (`groupPolicy` + `groups`): si el bot acepta mensajes de un canal en absoluto.
2.  **Acceso del remitente** (`groupAllowFrom` / `groups["#channel"].allowFrom` por canal): quién tiene permitido activar el bot dentro de ese canal.

Claves de configuración:

-   Lista de permitidos para MD (acceso del remitente en MD): `channels.irc.allowFrom`
-   Lista de permitidos del remitente del grupo (acceso del remitente del canal): `channels.irc.groupAllowFrom`
-   Controles por canal (canal + remitente + reglas de mención): `channels.irc.groups["#channel"]`
-   `channels.irc.groupPolicy="open"` permite canales no configurados (**aún con control de mención por defecto**)

Las entradas de la lista de permitidos deben usar identidades de remitente estables (`nick!user@host`). La coincidencia simple de nick es mutable y solo se habilita cuando `channels.irc.dangerouslyAllowNameMatching: true`.

### Error común: allowFrom es para MD, no para canales

Si ves registros como:

-   `irc: drop group sender alice!ident@host (policy=allowlist)`

…significa que el remitente no estaba permitido para mensajes de **grupo/canal**. Soluciónalo ya sea:

-   estableciendo `channels.irc.groupAllowFrom` (global para todos los canales), o
-   estableciendo listas de remitentes permitidos por canal: `channels.irc.groups["#channel"].allowFrom`

Ejemplo (permitir que cualquiera en `#tuirc-dev` hable con el bot):

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Activación de respuestas (menciones)

Incluso si un canal está permitido (vía `groupPolicy` + `groups`) y el remitente está permitido, OpenClaw tiene como valor predeterminado el **control de menciones** en contextos grupales. Eso significa que puedes ver registros como `drop channel … (missing-mention)` a menos que el mensaje incluya un patrón de mención que coincida con el bot. Para hacer que el bot responda en un canal de IRC **sin necesidad de una mención**, deshabilita el control de menciones para ese canal:

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

O para permitir **todos** los canales de IRC (sin lista de permitidos por canal) y aún responder sin menciones:

```json
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## Nota de seguridad (recomendado para canales públicos)

Si permites `allowFrom: ["*"]` en un canal público, cualquiera puede activar el bot. Para reducir el riesgo, restringe las herramientas para ese canal.

### Mismas herramientas para todos en el canal

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Diferentes herramientas por remitente (el propietario obtiene más poder)

Usa `toolsBySender` para aplicar una política más estricta a `"*"` y una más flexible a tu nick:

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            "id:eigen": {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Notas:

-   Las claves de `toolsBySender` deben usar `id:` para los valores de identidad del remitente de IRC: `id:eigen` o `id:eigen!~eigen@174.127.248.171` para una coincidencia más fuerte.
-   Aún se aceptan claves sin prefijo heredadas y se comparan solo como `id:`.
-   La primera política de remitente coincidente gana; `"*"` es el comodín de respaldo.

Para más información sobre el acceso grupal vs. el control de menciones (y cómo interactúan), consulta: [/channels/groups](./groups.md).

## NickServ

Para identificarse con NickServ después de conectarse:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "tu-contraseña-de-nickserv"
      }
    }
  }
}
```

Registro único opcional al conectarse:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Deshabilita `register` después de que el nick esté registrado para evitar intentos repetidos de REGISTER.

## Variables de entorno

La cuenta predeterminada admite:

-   `IRC_HOST`
-   `IRC_PORT`
-   `IRC_TLS`
-   `IRC_NICK`
-   `IRC_USERNAME`
-   `IRC_REALNAME`
-   `IRC_PASSWORD`
-   `IRC_CHANNELS` (separados por comas)
-   `IRC_NICKSERV_PASSWORD`
-   `IRC_NICKSERV_REGISTER_EMAIL`

## Solución de problemas

-   Si el bot se conecta pero nunca responde en los canales, verifica `channels.irc.groups` **y** si el control de menciones está descartando mensajes (`missing-mention`). Si quieres que responda sin pings, establece `requireMention:false` para el canal.
-   Si el inicio de sesión falla, verifica la disponibilidad del nick y la contraseña del servidor.
-   Si TLS falla en una red personalizada, verifica la configuración de host/puerto y certificado.

[iMessage](./imessage.md)[LINE](./line.md)