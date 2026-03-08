title: "Guía de Comandos de Canales de OpenClaw CLI para Gestionar Cuentas de Chat"
description: "Aprende a gestionar cuentas de canales de chat, añadir o eliminar cuentas, resolver nombres a IDs y solucionar problemas de estado en tiempo de ejecución usando comandos de OpenClaw CLI."
keywords: ["openclaw cli", "comandos de canales", "cuentas de chat", "estado del gateway", "añadir cuenta de canal", "solución de problemas de canales", "resolver ids de canal", "capacidades del canal"]
---

  Comandos CLI

  
# channels

Gestiona cuentas de canales de chat y su estado de ejecución en el Gateway. Documentación relacionada:

-   Guías de canales: [Canales](../channels/index.md)
-   Configuración del Gateway: [Configuración](../gateway/configuration.md)

## Comandos comunes

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

## Añadir / eliminar cuentas

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

Consejo: `openclaw channels add --help` muestra las banderas por canal (token, app token, rutas de signal-cli, etc). Cuando ejecutas `openclaw channels add` sin banderas, el asistente interactivo puede preguntar:

-   ids de cuenta por canal seleccionado
-   nombres para mostrar opcionales para esas cuentas
-   `¿Vincular las cuentas de canal configuradas a agentes ahora?`

Si confirmas vincular ahora, el asistente pregunta qué agente debe poseer cada cuenta de canal configurada y escribe reglas de enrutamiento con alcance de cuenta. También puedes gestionar las mismas reglas de enrutamiento más tarde con `openclaw agents bindings`, `openclaw agents bind` y `openclaw agents unbind` (ver [agents](./agents.md)). Cuando añades una cuenta no predeterminada a un canal que aún usa configuraciones de nivel superior de cuenta única (sin entradas `channels..accounts` todavía), OpenClaw mueve los valores de nivel superior de cuenta única con alcance de cuenta a `channels..accounts.default`, luego escribe la nueva cuenta. Esto preserva el comportamiento original de la cuenta mientras se pasa a la forma multi-cuenta. El comportamiento de enrutamiento se mantiene consistente:

-   Los enlaces existentes solo de canal (sin `accountId`) continúan coincidiendo con la cuenta predeterminada.
-   `channels add` no crea automáticamente ni reescribe enlaces en modo no interactivo.
-   La configuración interactiva puede añadir opcionalmente enlaces con alcance de cuenta.

Si tu configuración ya estaba en un estado mixto (cuentas nombradas presentes, falta `default`, y aún estaban establecidos los valores de cuenta única de nivel superior), ejecuta `openclaw doctor --fix` para mover los valores con alcance de cuenta a `accounts.default`.

## Iniciar / cerrar sesión (interactivo)

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

## Solución de problemas

-   Ejecuta `openclaw status --deep` para una sonda amplia.
-   Usa `openclaw doctor` para correcciones guiadas.
-   `openclaw channels list` imprime `Claude: HTTP 403 ... user:profile` → la instantánea de uso necesita el alcance `user:profile`. Usa `--no-usage`, o proporciona una clave de sesión de claude.ai (`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`), o reautentícate a través de Claude Code CLI.
-   `openclaw channels status` recurre a resúmenes solo de configuración cuando el gateway es inalcanzable. Si una credencial de canal compatible está configurada vía SecretRef pero no está disponible en la ruta de comando actual, reporta esa cuenta como configurada con notas degradadas en lugar de mostrarla como no configurada.

## Sonda de capacidades

Obtén pistas de capacidades del proveedor (intenciones/alcances donde estén disponibles) más soporte de características estáticas:

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

Notas:

-   `--channel` es opcional; omítelo para listar cada canal (incluyendo extensiones).
-   `--target` acepta `channel:` o un id de canal numérico crudo y solo se aplica a Discord.
-   Las sondas son específicas del proveedor: intenciones de Discord + permisos de canal opcionales; alcances de bot + usuario de Slack; banderas de bot de Telegram + webhook; versión del daemon de Signal; token de aplicación de MS Teams + roles/alcances de Graph (anotados donde se conozcan). Los canales sin sondas reportan `Probe: unavailable`.

## Resolver nombres a IDs

Resuelve nombres de canal/usuario a IDs usando el directorio del proveedor:

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

Notas:

-   Usa `--kind user|group|auto` para forzar el tipo de destino.
-   La resolución prefiere coincidencias activas cuando múltiples entradas comparten el mismo nombre.
-   `channels resolve` es de solo lectura. Si una cuenta seleccionada está configurada vía SecretRef pero esa credencial no está disponible en la ruta de comando actual, el comando devuelve resultados degradados no resueltos con notas en lugar de abortar toda la ejecución.

[browser](./browser.md)[clawbot](./clawbot.md)