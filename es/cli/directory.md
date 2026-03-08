

  Comandos CLI

  
# directorio

Búsquedas en el directorio para canales que lo admiten (contactos/pares, grupos y "yo").

## Banderas comunes

-   `--channel `: id/alias del canal (requerido cuando hay múltiples canales configurados; automático cuando solo hay uno configurado)
-   `--account `: id de la cuenta (predeterminado: el predeterminado del canal)
-   `--json`: salida en JSON

## Notas

-   `directory` está diseñado para ayudarte a encontrar IDs que puedas pegar en otros comandos (especialmente `openclaw message send --target ...`).
-   Para muchos canales, los resultados están respaldados por configuración (listas permitidas / grupos configurados) en lugar de ser un directorio activo del proveedor.
-   La salida predeterminada es `id` (y a veces `nombre`) separados por una tabulación; usa `--json` para scripting.

## Usando resultados con message send

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## Formatos de ID (por canal)

-   WhatsApp: `+15551234567` (DM), `1234567890-1234567890@g.us` (grupo)
-   Telegram: `@nombredeusuario` o id de chat numérico; los grupos son ids numéricos
-   Slack: `user:U…` y `channel:C…`
-   Discord: `user:` y `channel:`
-   Matrix (plugin): `user:@user:server`, `room:!roomId:server`, o `#alias:server`
-   Microsoft Teams (plugin): `user:` y `conversation:`
-   Zalo (plugin): id de usuario (Bot API)
-   Zalo Personal / `zalouser` (plugin): id de hilo (DM/grupo) de `zca` (`me`, `friend list`, `group list`)

## Yo mismo (“me”)

```bash
openclaw directory self --channel zalouser
```

## Pares (contactos/usuarios)

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "nombre"
openclaw directory peers list --channel zalouser --limit 50
```

## Grupos

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "trabajo"
openclaw directory groups members --channel zalouser --group-id <id>
```

[dispositivos](./devices.md)[dns](./dns.md)