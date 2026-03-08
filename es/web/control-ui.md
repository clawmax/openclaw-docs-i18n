

  Interfaces web

  
# Interfaz de Control

La Interfaz de Control es una pequeña aplicación de una sola página **Vite + Lit** servida por la Puerta de Enlace:

-   predeterminado: `http://:18789/`
-   prefijo opcional: establece `gateway.controlUi.basePath` (ej. `/openclaw`)

Se comunica **directamente con el WebSocket de la Puerta de Enlace** en el mismo puerto.

## Apertura rápida (local)

Si la Puerta de Enlace se está ejecutando en la misma computadora, abre:

-   [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (o [http://localhost:18789/](http://localhost:18789/))

Si la página no carga, inicia primero la Puerta de Enlace: `openclaw gateway`. La autenticación se proporciona durante el protocolo de enlace del WebSocket mediante:

-   `connect.params.auth.token`
-   `connect.params.auth.password` El panel de configuración del dashboard te permite almacenar un token; las contraseñas no se conservan. El asistente de configuración inicial genera un token de puerta de enlace por defecto, así que pégalo aquí en la primera conexión.

## Emparejamiento de dispositivo (primera conexión)

Cuando te conectas a la Interfaz de Control desde un nuevo navegador o dispositivo, la Puerta de Enlace requiere una **aprobación de emparejamiento única** — incluso si estás en la misma Tailnet con `gateway.auth.allowTailscale: true`. Esta es una medida de seguridad para prevenir accesos no autorizados. **Lo que verás:** “desconectado (1008): emparejamiento requerido” **Para aprobar el dispositivo:**

```bash
# Listar solicitudes pendientes
openclaw devices list

# Aprobar por ID de solicitud
openclaw devices approve <requestId>
```

Una vez aprobado, el dispositivo se recuerda y no requerirá re-aprobación a menos que lo revoques con `openclaw devices revoke --device  --role `. Consulta [CLI de Dispositivos](../cli/devices.md) para la rotación y revocación de tokens. **Notas:**

-   Las conexiones locales (`127.0.0.1`) se aprueban automáticamente.
-   Las conexiones remotas (LAN, Tailnet, etc.) requieren aprobación explícita.
-   Cada perfil de navegador genera un ID de dispositivo único, por lo que cambiar de navegador o borrar los datos del navegador requerirá re-emparejamiento.

## Soporte de idiomas

La Interfaz de Control puede localizarse en la primera carga según la configuración regional de tu navegador, y puedes anularla más tarde desde el selector de idioma en la tarjeta de Acceso.

-   Idiomas soportados: `en`, `zh-CN`, `zh-TW`, `pt-BR`, `de`, `es`
-   Las traducciones a idiomas no ingleses se cargan de forma diferida en el navegador.
-   La configuración regional seleccionada se guarda en el almacenamiento del navegador y se reutiliza en visitas futuras.
-   Las claves de traducción faltantes recurren al inglés.

## Qué puede hacer (actualmente)

-   Chatear con el modelo a través del WS de la Puerta de Enlace (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
-   Transmitir llamadas a herramientas + tarjetas de salida de herramientas en vivo en el Chat (eventos del agente)
-   Canales: WhatsApp/Telegram/Discord/Slack + canales de complementos (Mattermost, etc.) estado + inicio de sesión QR + configuración por canal (`channels.status`, `web.login.*`, `config.patch`)
-   Instancias: lista de presencia + actualizar (`system-presence`)
-   Sesiones: listar + anulaciones de pensamiento/verboso por sesión (`sessions.list`, `sessions.patch`)
-   Trabajos cron: listar/agregar/editar/ejecutar/habilitar/deshabilitar + historial de ejecución (`cron.*`)
-   Habilidades: estado, habilitar/deshabilitar, instalar, actualizaciones de claves API (`skills.*`)
-   Nodos: listar + capacidades (`node.list`)
-   Aprobaciones de ejecución: editar listas de permitidos de puerta de enlace o nodo + política de solicitud para `exec host=gateway/node` (`exec.approvals.*`)
-   Configuración: ver/editar `~/.openclaw/openclaw.json` (`config.get`, `config.set`)
-   Configuración: aplicar + reiniciar con validación (`config.apply`) y despertar la última sesión activa
-   Las escrituras de configuración incluyen una protección de hash base para evitar sobrescribir ediciones concurrentes
-   Esquema de configuración + renderizado de formularios (`config.schema`, incluyendo esquemas de complementos y canales); El editor JSON crudo sigue disponible
-   Depuración: instantáneas de estado/salud/modelos + registro de eventos + llamadas RPC manuales (`status`, `health`, `models.list`)
-   Registros: seguimiento en vivo de registros de archivos de la puerta de enlace con filtro/exportación (`logs.tail`)
-   Actualización: ejecutar una actualización de paquete/git + reiniciar (`update.run`) con un informe de reinicio

Notas del panel de trabajos cron:

-   Para trabajos aislados, la entrega predeterminada es anunciar el resumen. Puedes cambiar a ninguno si quieres ejecuciones solo internas.
-   Los campos de canal/destino aparecen cuando se selecciona anunciar.
-   El modo webhook usa `delivery.mode = "webhook"` con `delivery.to` establecido en una URL de webhook HTTP(S) válida.
-   Para trabajos de sesión principal, están disponibles los modos de entrega webhook y ninguno.
-   Los controles de edición avanzados incluyen eliminar después de ejecutar, anulación de agente clara, opciones cron exactas/espaciadas, anulaciones de modelo/pensamiento del agente y alternancias de entrega de mejor esfuerzo.
-   La validación del formulario es en línea con errores a nivel de campo; los valores inválidos deshabilitan el botón de guardar hasta que se corrijan.
-   Establece `cron.webhookToken` para enviar un token de portador dedicado; si se omite, el webhook se envía sin una cabecera de autenticación.
-   Retrocompatibilidad obsoleta: los trabajos heredados almacenados con `notify: true` aún pueden usar `cron.webhook` hasta que se migren.

## Comportamiento del Chat

-   `chat.send` es **no bloqueante**: acusa recibo inmediatamente con `{ runId, status: "started" }` y la respuesta se transmite a través de eventos `chat`.
-   Re-enviar con la misma `idempotencyKey` devuelve `{ status: "in_flight" }` mientras se ejecuta, y `{ status: "ok" }` después de la finalización.
-   Las respuestas de `chat.history` tienen un tamaño limitado para seguridad de la UI. Cuando las entradas de la transcripción son demasiado grandes, la Puerta de Enlace puede truncar campos de texto largos, omitir bloques de metadatos pesados y reemplazar mensajes de gran tamaño con un marcador de posición (`[chat.history omitido: mensaje demasiado grande]`).
-   `chat.inject` agrega una nota del asistente a la transcripción de la sesión y transmite un evento `chat` para actualizaciones solo de UI (sin ejecución de agente, sin entrega a canal).
-   Detener:
    -   Haz clic en **Detener** (llama a `chat.abort`)
    -   Escribe `/stop` (o frases de abortar independientes como `stop`, `stop action`, `stop run`, `stop openclaw`, `please stop`) para abortar fuera de banda
    -   `chat.abort` soporta `{ sessionKey }` (sin `runId`) para abortar todas las ejecuciones activas para esa sesión
-   Retención parcial por aborto:
    -   Cuando se aborta una ejecución, el texto parcial del asistente aún puede mostrarse en la UI
    -   La Puerta de Enlace persiste el texto parcial del asistente abortado en el historial de transcripción cuando existe salida en búfer
    -   Las entradas persistentes incluyen metadatos de aborto para que los consumidores de la transcripción puedan distinguir las partes parciales abortadas de la salida de finalización normal

## Acceso a Tailnet (recomendado)

### Tailscale Serve integrado (preferido)

Mantén la Puerta de Enlace en loopback y deja que Tailscale Serve la sirva con HTTPS:

```bash
openclaw gateway --tailscale serve
```

Abre:

-   `https:///` (o tu `gateway.controlUi.basePath` configurado)

Por defecto, las solicitudes Serve de la Interfaz de Control/WebSocket pueden autenticarse a través de las cabeceras de identidad de Tailscale (`tailscale-user-login`) cuando `gateway.auth.allowTailscale` es `true`. OpenClaw verifica la identidad resolviendo la dirección `x-forwarded-for` con `tailscale whois` y comparándola con la cabecera, y solo acepta estas cuando la solicitud llega a loopback con las cabeceras `x-forwarded-*` de Tailscale. Establece `gateway.auth.allowTailscale: false` (o fuerza `gateway.auth.mode: "password"`) si quieres requerir un token/contraseña incluso para el tráfico Serve. La autenticación Serve sin token asume que el host de la puerta de enlace es confiable. Si en ese host puede ejecutarse código local no confiable, requiere autenticación con token/contraseña.

### Vincular a tailnet + token

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

Luego abre:

-   `http://<tailscale-ip>:18789/` (o tu `gateway.controlUi.basePath` configurado)

Pega el token en la configuración de la UI (enviado como `connect.params.auth.token`).

## HTTP inseguro

Si abres el panel a través de HTTP plano (`http://<lan-ip>` o `http://<tailscale-ip>`), el navegador se ejecuta en un **contexto no seguro** y bloquea WebCrypto. Por defecto, OpenClaw **bloquea** las conexiones de la Interfaz de Control sin identidad de dispositivo. **Solución recomendada:** usa HTTPS (Tailscale Serve) o abre la UI localmente:

-   `https:///` (Serve)
-   `http://127.0.0.1:18789/` (en el host de la puerta de enlace)

**Comportamiento del interruptor de autenticación insegura:**

```json
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`allowInsecureAuth` no omite la identidad del dispositivo de la Interfaz de Control ni las comprobaciones de emparejamiento. **Solo para emergencias:**

```json
{
  gateway: {
    controlUi: { dangerouslyDisableDeviceAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`dangerouslyDisableDeviceAuth` deshabilita las comprobaciones de identidad del dispositivo de la Interfaz de Control y es una degradación de seguridad severa. Revierte rápidamente después de un uso de emergencia. Consulta [Tailscale](../gateway/tailscale.md) para orientación sobre configuración HTTPS.

## Construyendo la UI

La Puerta de Enlace sirve archivos estáticos desde `dist/control-ui`. Construyelos con:

```bash
pnpm ui:build # instala automáticamente las dependencias de la UI en la primera ejecución
```

Base absoluta opcional (cuando quieres URLs de activos fijas):

```
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

Para desarrollo local (servidor de desarrollo separado):

```bash
pnpm ui:dev # instala automáticamente las dependencias de la UI en la primera ejecución
```

Luego apunta la UI a tu URL WS de la Puerta de Enlace (ej. `ws://127.0.0.1:18789`).

## Depuración/pruebas: servidor de desarrollo + Puerta de Enlace remota

La Interfaz de Control son archivos estáticos; el objetivo del WebSocket es configurable y puede ser diferente del origen HTTP. Esto es útil cuando quieres el servidor de desarrollo Vite localmente pero la Puerta de Enlace se ejecuta en otro lugar.

1.  Inicia el servidor de desarrollo de la UI: `pnpm ui:dev`
2.  Abre una URL como:

```
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

Autenticación única opcional (si es necesaria):

```
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789#token=<gateway-token>
```

Notas:

-   `gatewayUrl` se almacena en localStorage después de la carga y se elimina de la URL.
-   `token` se importa a la memoria para la pestaña actual y se elimina de la URL; no se almacena en localStorage.
-   `password` se mantiene solo en memoria.
-   Cuando se establece `gatewayUrl`, la UI no recurre a credenciales de configuración o entorno. Proporciona `token` (o `password`) explícitamente. La falta de credenciales explícitas es un error.
-   Usa `wss://` cuando la Puerta de Enlace esté detrás de TLS (Tailscale Serve, proxy HTTPS, etc.).
-   `gatewayUrl` solo se acepta en una ventana de nivel superior (no incrustada) para prevenir clickjacking.
-   Los despliegues de la Interfaz de Control que no sean loopback deben establecer `gateway.controlUi.allowedOrigins` explícitamente (orígenes completos). Esto incluye configuraciones de desarrollo remotas.
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` habilita el modo de recurrencia de origen por cabecera Host, pero es un modo de seguridad peligroso.

Ejemplo:

```json
{
  gateway: {
    controlUi: {
      allowedOrigins: ["http://localhost:5173"],
    },
  },
}
```

Detalles de configuración de acceso remoto: [Acceso remoto](../gateway/remote.md).

[Web](../web.md)[Panel](./dashboard.md)