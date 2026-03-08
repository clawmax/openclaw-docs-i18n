

  Plataformas de mensajería

  
# Matrix

Matrix es un protocolo de mensajería abierto y descentralizado. OpenClaw se conecta como un **usuario** de Matrix en cualquier servidor de inicio (homeserver), por lo que necesitas una cuenta de Matrix para el bot. Una vez que haya iniciado sesión, puedes enviarle mensajes directos (DM) o invitarlo a salas (los "grupos" de Matrix). Beeper también es una opción de cliente válida, pero requiere que E2EE esté habilitado. Estado: soportado vía plugin (@vector-im/matrix-bot-sdk). Mensajes directos, salas, hilos, multimedia, reacciones, encuestas (enviar + inicio de encuesta como texto), ubicación y E2EE (con soporte criptográfico).

## Plugin requerido

Matrix se distribuye como un plugin y no está incluido en la instalación principal. Instálalo mediante CLI (registro npm):

```bash
openclaw plugins install @openclaw/matrix
```

Desde un repositorio local (cuando se ejecuta desde un clon de git):

```bash
openclaw plugins install ./extensions/matrix
```

Si eliges Matrix durante la configuración/onboarding y se detecta un clon de git, OpenClaw ofrecerá automáticamente la ruta de instalación local. Detalles: [Plugins](../tools/plugin.md)

## Configuración

1.  Instala el plugin de Matrix:
    -   Desde npm: `openclaw plugins install @openclaw/matrix`
    -   Desde un clon local: `openclaw plugins install ./extensions/matrix`
2.  Crea una cuenta de Matrix en un servidor de inicio (homeserver):
    -   Explora opciones de alojamiento en [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
    -   O alójalo tú mismo.
3.  Obtén un token de acceso para la cuenta del bot:
    
    -   Usa la API de inicio de sesión de Matrix con `curl` en tu servidor de inicio:
    
    Copiar
    
    ```bash
    curl --request POST \
      --url https://matrix.example.org/_matrix/client/v3/login \
      --header 'Content-Type: application/json' \
      --data '{
      "type": "m.login.password",
      "identifier": {
        "type": "m.id.user",
        "user": "your-user-name"
      },
      "password": "your-password"
    }'
    ```
    
    -   Reemplaza `matrix.example.org` con la URL de tu servidor de inicio.
    -   O configura `channels.matrix.userId` + `channels.matrix.password`: OpenClaw llama al mismo endpoint de inicio de sesión, almacena el token de acceso en `~/.openclaw/credentials/matrix/credentials.json` y lo reutiliza en el próximo inicio.
4.  Configura las credenciales:
    -   Variables de entorno: `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (o `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
    -   O en configuración: `channels.matrix.*`
    -   Si ambos están configurados, la configuración tiene prioridad.
    -   Con token de acceso: el ID de usuario se obtiene automáticamente vía `/whoami`.
    -   Cuando se configura, `channels.matrix.userId` debe ser el ID completo de Matrix (ejemplo: `@bot:example.org`).
5.  Reinicia la pasarela (o finaliza el onboarding).
6.  Inicia un mensaje directo con el bot o invítalo a una sala desde cualquier cliente de Matrix (Element, Beeper, etc.; consulta [https://matrix.org/ecosystem/clients/](https://matrix.org/ecosystem/clients/)). Beeper requiere E2EE, así que configura `channels.matrix.encryption: true` y verifica el dispositivo.

Configuración mínima (token de acceso, ID de usuario obtenido automáticamente):

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" },
    },
  },
}
```

Configuración E2EE (cifrado de extremo a extremo habilitado):

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" },
    },
  },
}
```

## Cifrado (E2EE)

El cifrado de extremo a extremo es **soportado** mediante el SDK criptográfico Rust. Habilítalo con `channels.matrix.encryption: true`:

-   Si el módulo criptográfico se carga, las salas cifradas se descifran automáticamente.
-   Los medios salientes se cifran al enviar a salas cifradas.
-   En la primera conexión, OpenClaw solicita la verificación del dispositivo desde tus otras sesiones.
-   Verifica el dispositivo en otro cliente de Matrix (Element, etc.) para habilitar el intercambio de claves.
-   Si el módulo criptográfico no se puede cargar, E2EE se deshabilita y las salas cifradas no se descifrarán; OpenClaw registrará una advertencia.
-   Si ves errores de módulo criptográfico faltante (por ejemplo, `@matrix-org/matrix-sdk-crypto-nodejs-*`), permite los scripts de compilación para `@matrix-org/matrix-sdk-crypto-nodejs` y ejecuta `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` o obtén el binario con `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`.

El estado criptográfico se almacena por cuenta + token de acceso en `~/.openclaw/matrix/accounts//__/<token-hash>/crypto/` (base de datos SQLite). El estado de sincronización se guarda junto a él en `bot-storage.json`. Si el token de acceso (dispositivo) cambia, se crea un nuevo almacén y el bot debe ser re-verificado para las salas cifradas. **Verificación del dispositivo:** Cuando E2EE está habilitado, el bot solicitará verificación desde tus otras sesiones al iniciar. Abre Element (u otro cliente) y aprueba la solicitud de verificación para establecer confianza. Una vez verificado, el bot puede descifrar mensajes en salas cifradas.

## Multi-cuenta

Soporte multi-cuenta: usa `channels.matrix.accounts` con credenciales por cuenta y un `name` opcional. Consulta [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) para el patrón compartido. Cada cuenta se ejecuta como un usuario de Matrix separado en cualquier servidor de inicio. La configuración por cuenta hereda de los ajustes de nivel superior `channels.matrix` y puede anular cualquier opción (política de DM, grupos, cifrado, etc.).

```json
{
  channels: {
    matrix: {
      enabled: true,
      dm: { policy: "pairing" },
      accounts: {
        assistant: {
          name: "Asistente principal",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_assistant_***",
          encryption: true,
        },
        alerts: {
          name: "Bot de alertas",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_alerts_***",
          dm: { policy: "allowlist", allowFrom: ["@admin:example.org"] },
        },
      },
    },
  },
}
```

Notas:

-   El inicio de cuentas se serializa para evitar condiciones de carrera con importaciones concurrentes de módulos.
-   Las variables de entorno (`MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`, etc.) solo se aplican a la cuenta **predeterminada**.
-   Los ajustes base del canal (política de DM, política de grupo, mención condicional, etc.) se aplican a todas las cuentas a menos que se anulen por cuenta.
-   Usa `bindings[].match.accountId` para enrutar cada cuenta a un agente diferente.
-   El estado criptográfico se almacena por cuenta + token de acceso (almacenes de claves separados por cuenta).

## Modelo de enrutamiento

-   Las respuestas siempre regresan a Matrix.
-   Los mensajes directos (DM) comparten la sesión principal del agente; las salas se asignan a sesiones de grupo.

## Control de acceso (Mensajes directos)

-   Predeterminado: `channels.matrix.dm.policy = "pairing"`. Los remitentes desconocidos reciben un código de emparejamiento.
-   Aprueba mediante:
    -   `openclaw pairing list matrix`
    -   `openclaw pairing approve matrix <CÓDIGO>`
-   Mensajes directos públicos: `channels.matrix.dm.policy="open"` más `channels.matrix.dm.allowFrom=["*"]`.
-   `channels.matrix.dm.allowFrom` acepta IDs de usuario completos de Matrix (ejemplo: `@user:server`). El asistente resuelve los nombres para mostrar a IDs de usuario cuando la búsqueda en el directorio encuentra una coincidencia exacta única.
-   No uses nombres para mostrar o partes locales simples (ejemplo: `"Alice"` o `"alice"`). Son ambiguos y se ignoran para la coincidencia de la lista de permitidos. Usa IDs completos `@user:server`.

## Salas (grupos)

-   Predeterminado: `channels.matrix.groupPolicy = "allowlist"` (acceso por mención). Usa `channels.defaults.groupPolicy` para anular el valor predeterminado cuando no esté configurado.
-   Nota de tiempo de ejecución: si `channels.matrix` falta por completo, el tiempo de ejecución recurre a `groupPolicy="allowlist"` para las comprobaciones de sala (incluso si `channels.defaults.groupPolicy` está configurado).
-   Permite salas con `channels.matrix.groups` (IDs de sala o alias; los nombres se resuelven a IDs cuando la búsqueda en el directorio encuentra una coincidencia exacta única):

```json
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
      groupAllowFrom: ["@owner:example.org"],
    },
  },
}
```

-   `requireMention: false` habilita la respuesta automática en esa sala.
-   `groups."*"` puede establecer valores predeterminados para el acceso por mención en todas las salas.
-   `groupAllowFrom` restringe qué remitentes pueden activar el bot en las salas (IDs de usuario completos de Matrix).
-   Las listas de permitidos `users` por sala pueden restringir aún más los remitentes dentro de una sala específica (usa IDs de usuario completos de Matrix).
-   El asistente de configuración solicita listas de salas permitidas (IDs de sala, alias o nombres) y resuelve nombres solo en una coincidencia exacta y única.
-   Al iniciar, OpenClaw resuelve los nombres de sala/usuario en las listas de permitidos a IDs y registra la asignación; las entradas no resueltas se ignoran para la coincidencia de la lista de permitidos.
-   Las invitaciones se aceptan automáticamente por defecto; controla esto con `channels.matrix.autoJoin` y `channels.matrix.autoJoinAllowlist`.
-   Para no permitir **ninguna sala**, configura `channels.matrix.groupPolicy: "disabled"` (o mantén una lista de permitidos vacía).
-   Clave heredada: `channels.matrix.rooms` (misma forma que `groups`).

## Hilos

-   Se soporta el enhebrado de respuestas.
-   `channels.matrix.threadReplies` controla si las respuestas permanecen en hilos:
    -   `off`, `inbound` (predeterminado), `always`
-   `channels.matrix.replyToMode` controla los metadatos de respuesta cuando no se responde en un hilo:
    -   `off` (predeterminado), `first`, `all`

## Capacidades

| Característica | Estado |
| --- | --- |
| Mensajes directos | ✅ Soportado |
| Salas | ✅ Soportado |
| Hilos | ✅ Soportado |
| Multimedia | ✅ Soportado |
| E2EE | ✅ Soportado (se requiere módulo criptográfico) |
| Reacciones | ✅ Soportado (enviar/leer mediante herramientas) |
| Encuestas | ✅ Envío soportado; los inicios de encuestas entrantes se convierten a texto (se ignoran respuestas/finalizaciones) |
| Ubicación | ✅ Soportado (URI geo; se ignora la altitud) |
| Comandos nativos | ✅ Soportado |

## Solución de problemas

Ejecuta primero esta escalera:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Luego confirma el estado de emparejamiento de DM si es necesario:

```bash
openclaw pairing list matrix
```

Fallos comunes:

-   Inició sesión pero se ignoran los mensajes de la sala: la sala está bloqueada por `groupPolicy` o la lista de salas permitidas.
-   Se ignoran los mensajes directos: el remitente está pendiente de aprobación cuando `channels.matrix.dm.policy="pairing"`.
-   Fallan las salas cifradas: falta soporte criptográfico o hay discrepancia en la configuración de cifrado.

Para el flujo de triaje: [/channels/troubleshooting](./troubleshooting.md).

## Referencia de configuración (Matrix)

Configuración completa: [Configuración](../gateway/configuration.md) Opciones del proveedor:

-   `channels.matrix.enabled`: habilita/deshabilita el inicio del canal.
-   `channels.matrix.homeserver`: URL del servidor de inicio.
-   `channels.matrix.userId`: ID de usuario de Matrix (opcional con token de acceso).
-   `channels.matrix.accessToken`: token de acceso.
-   `channels.matrix.password`: contraseña para inicio de sesión (token almacenado).
-   `channels.matrix.deviceName`: nombre para mostrar del dispositivo.
-   `channels.matrix.encryption`: habilita E2EE (predeterminado: false).
-   `channels.matrix.initialSyncLimit`: límite de sincronización inicial.
-   `channels.matrix.threadReplies`: `off | inbound | always` (predeterminado: inbound).
-   `channels.matrix.textChunkLimit`: tamaño de fragmento de texto saliente (caracteres).
-   `channels.matrix.chunkMode`: `length` (predeterminado) o `newline` para dividir en líneas en blanco (límites de párrafo) antes del fragmentado por longitud.
-   `channels.matrix.dm.policy`: `pairing | allowlist | open | disabled` (predeterminado: pairing).
-   `channels.matrix.dm.allowFrom`: lista de permitidos para DM (IDs de usuario completos de Matrix). `open` requiere `"*"`. El asistente resuelve nombres a IDs cuando es posible.
-   `channels.matrix.groupPolicy`: `allowlist | open | disabled` (predeterminado: allowlist).
-   `channels.matrix.groupAllowFrom`: remitentes permitidos para mensajes de grupo (IDs de usuario completos de Matrix).
-   `channels.matrix.allowlistOnly`: fuerza las reglas de lista de permitidos para DM + salas.
-   `channels.matrix.groups`: lista de grupos permitidos + mapa de configuración por sala.
-   `channels.matrix.rooms`: lista de grupos permitidos/configuración heredada.
-   `channels.matrix.replyToMode`: modo de respuesta para hilos/etiquetas.
-   `channels.matrix.mediaMaxMb`: límite de medios entrantes/salientes (MB).
-   `channels.matrix.autoJoin`: manejo de invitaciones (`always | allowlist | off`, predeterminado: always).
-   `channels.matrix.autoJoinAllowlist`: IDs de sala/alias permitidos para auto-unirse.
-   `channels.matrix.accounts`: configuración multi-cuenta indexada por ID de cuenta (cada cuenta hereda los ajustes de nivel superior).
-   `channels.matrix.actions`: control de herramientas por acción (reacciones/mensajes/fijaciones/infoMiembro/infoCanal).

[LINE](./line.md)[Mattermost](./mattermost.md)