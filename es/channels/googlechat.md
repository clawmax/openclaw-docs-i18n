

  Plataformas de mensajería

  
# Google Chat

Estado: listo para mensajes directos + espacios mediante webhooks de la API de Google Chat (solo HTTP).

## Configuración rápida (principiante)

1.  Crea un proyecto de Google Cloud y habilita la **API de Google Chat**.
    -   Ve a: [Credenciales de la API de Google Chat](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
    -   Habilita la API si no está ya activada.
2.  Crea una **Cuenta de Servicio**:
    -   Presiona **Crear credenciales** > **Cuenta de servicio**.
    -   Nómbrala como quieras (ej., `openclaw-chat`).
    -   Deja los permisos en blanco (presiona **Continuar**).
    -   Deja los principales con acceso en blanco (presiona **Listo**).
3.  Crea y descarga la **Clave JSON**:
    -   En la lista de cuentas de servicio, haz clic en la que acabas de crear.
    -   Ve a la pestaña **Claves**.
    -   Haz clic en **Agregar clave** > **Crear nueva clave**.
    -   Selecciona **JSON** y presiona **Crear**.
4.  Guarda el archivo JSON descargado en tu host de puerta de enlace (ej., `~/.openclaw/googlechat-service-account.json`).
5.  Crea una aplicación de Google Chat en la [Configuración de Chat de la Consola de Google Cloud](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat):
    -   Completa la **Información de la aplicación**:
        -   **Nombre de la aplicación**: (ej. `OpenClaw`)
        -   **URL del avatar**: (ej. `https://openclaw.ai/logo.png`)
        -   **Descripción**: (ej. `Asistente de IA personal`)
    -   Habilita **Funciones interactivas**.
    -   En **Funcionalidad**, marca **Unirse a espacios y conversaciones grupales**.
    -   En **Configuración de conexión**, selecciona **URL de endpoint HTTP**.
    -   En **Disparadores**, selecciona **Usar una URL de endpoint HTTP común para todos los disparadores** y configúrala como la URL pública de tu puerta de enlace seguida de `/googlechat`.
        -   *Consejo: Ejecuta `openclaw status` para encontrar la URL pública de tu puerta de enlace.*
    -   En **Visibilidad**, marca **Hacer que esta aplicación de Chat esté disponible para personas y grupos específicos en `Tu Dominio`**.
    -   Ingresa tu dirección de correo electrónico (ej. `user@example.com`) en el cuadro de texto.
    -   Haz clic en **Guardar** al final.
6.  **Habilita el estado de la aplicación**:
    -   Después de guardar, **actualiza la página**.
    -   Busca la sección **Estado de la aplicación** (generalmente cerca de la parte superior o inferior después de guardar).
    -   Cambia el estado a **En vivo - disponible para usuarios**.
    -   Haz clic en **Guardar** nuevamente.
7.  Configura OpenClaw con la ruta de la cuenta de servicio + la audiencia del webhook:
    -   Variable de entorno: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/ruta/a/service-account.json`
    -   O en configuración: `channels.googlechat.serviceAccountFile: "/ruta/a/service-account.json"`.
8.  Establece el tipo de audiencia del webhook + su valor (debe coincidir con la configuración de tu aplicación de Chat).
9.  Inicia la puerta de enlace. Google Chat enviará POST a tu ruta de webhook.

## Agregar a Google Chat

Una vez que la puerta de enlace esté en ejecución y tu correo electrónico esté en la lista de visibilidad:

1.  Ve a [Google Chat](https://chat.google.com/).
2.  Haz clic en el icono **+** (más) junto a **Mensajes directos**.
3.  En la barra de búsqueda (donde normalmente agregas personas), escribe el **Nombre de la aplicación** que configuraste en la Consola de Google Cloud.
    -   **Nota**: El bot *no* aparecerá en la lista de exploración del "Marketplace" porque es una aplicación privada. Debes buscarlo por su nombre.
4.  Selecciona tu bot de los resultados.
5.  Haz clic en **Agregar** o **Chatear** para iniciar una conversación 1:1.
6.  ¡Envía "Hola" para activar al asistente!

## URL pública (solo Webhook)

Los webhooks de Google Chat requieren un endpoint HTTPS público. Por seguridad, **expón solo la ruta `/googlechat`** a internet. Mantén el panel de control de OpenClaw y otros endpoints sensibles en tu red privada.

### Opción A: Tailscale Funnel (Recomendado)

Usa Tailscale Serve para el panel de control privado y Funnel para la ruta pública del webhook. Esto mantiene `/` privado mientras expone solo `/googlechat`.

1.  **Verifica a qué dirección está vinculada tu puerta de enlace:**
    
    Copiar
    
    ```bash
    ss -tlnp | grep 18789
    ```
    
    Anota la dirección IP (ej., `127.0.0.1`, `0.0.0.0`, o tu IP de Tailscale como `100.x.x.x`).
2.  **Expón el panel de control solo a la tailnet (puerto 8443):**
    
    Copiar
    
    ```bash
    # Si está vinculado a localhost (127.0.0.1 o 0.0.0.0):
    tailscale serve --bg --https 8443 http://127.0.0.1:18789
    
    # Si está vinculado solo a una IP de Tailscale (ej., 100.106.161.80):
    tailscale serve --bg --https 8443 http://100.106.161.80:18789
    ```
    
3.  **Expón solo la ruta del webhook públicamente:**
    
    Copiar
    
    ```bash
    # Si está vinculado a localhost (127.0.0.1 o 0.0.0.0):
    tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat
    
    # Si está vinculado solo a una IP de Tailscale (ej., 100.106.161.80):
    tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
    ```
    
4.  **Autoriza el nodo para acceso a Funnel:** Si se solicita, visita la URL de autorización que se muestra en la salida para habilitar Funnel para este nodo en la política de tu tailnet.
5.  **Verifica la configuración:**
    
    Copiar
    
    ```
    tailscale serve status
    tailscale funnel status
    ```
    

Tu URL pública del webhook será: `https://<nombre-nodo>..ts.net/googlechat` Tu panel de control privado permanece solo para la tailnet: `https://<nombre-nodo>..ts.net:8443/` Usa la URL pública (sin `:8443`) en la configuración de la aplicación de Google Chat.

> Nota: Esta configuración persiste tras reinicios. Para eliminarla más tarde, ejecuta `tailscale funnel reset` y `tailscale serve reset`.

### Opción B: Proxy inverso (Caddy)

Si usas un proxy inverso como Caddy, solo redirige la ruta específica:

```
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

Con esta configuración, cualquier solicitud a `your-domain.com/` será ignorada o devolverá un 404, mientras que `your-domain.com/googlechat` se enruta de forma segura a OpenClaw.

### Opción C: Túnel de Cloudflare

Configura las reglas de ingreso de tu túnel para enrutar solo la ruta del webhook:

-   **Ruta**: `/googlechat` -> `http://localhost:18789/googlechat`
-   **Regla por defecto**: HTTP 404 (No encontrado)

## Cómo funciona

1.  Google Chat envía POST de webhook a la puerta de enlace. Cada solicitud incluye una cabecera `Authorization: Bearer `.
    -   OpenClaw verifica la autenticación bearer antes de leer/analizar los cuerpos completos de los webhooks cuando la cabecera está presente.
    -   Las solicitudes de Complemento de Google Workspace que llevan `authorizationEventObject.systemIdToken` en el cuerpo son compatibles mediante un presupuesto de cuerpo previo a la autenticación más estricto.
2.  OpenClaw verifica el token contra el `audienceType` + `audience` configurado:
    -   `audienceType: "app-url"` → la audiencia es tu URL HTTPS del webhook.
    -   `audienceType: "project-number"` → la audiencia es el número del proyecto de Cloud.
3.  Los mensajes se enrutan por espacio:
    -   Los mensajes directos usan la clave de sesión `agent::googlechat:dm:`.
    -   Los espacios usan la clave de sesión `agent::googlechat:group:`.
4.  El acceso a mensajes directos es por emparejamiento por defecto. Los remitentes desconocidos reciben un código de emparejamiento; aprueba con:
    -   `openclaw pairing approve googlechat `
5.  Los espacios grupales requieren mención con @ por defecto. Usa `botUser` si la detección de menciones necesita el nombre de usuario de la aplicación.

## Destinos

Usa estos identificadores para entrega y listas de permitidos:

-   Mensajes directos: `users/` (recomendado).
-   El correo electrónico crudo `name@example.com` es mutable y solo se usa para coincidencia directa en listas de permitidos cuando `channels.googlechat.dangerouslyAllowNameMatching: true`.
-   Obsoleto: `users/` se trata como un ID de usuario, no como una lista de permitidos por correo.
-   Espacios: `spaces/`.

## Aspectos destacados de la configuración

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      // o serviceAccountRef: { source: "file", provider: "filemain", id: "/channels/googlechat/serviceAccount" }
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // opcional; ayuda a la detección de menciones
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Respuestas cortas solamente.",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

Notas:

-   Las credenciales de la cuenta de servicio también se pueden pasar en línea con `serviceAccount` (cadena JSON).
-   `serviceAccountRef` también es compatible (SecretRef de variable de entorno/archivo), incluyendo referencias por cuenta bajo `channels.googlechat.accounts..serviceAccountRef`.
-   La ruta de webhook por defecto es `/googlechat` si `webhookPath` no está configurada.
-   `dangerouslyAllowNameMatching` reactiva la coincidencia de principal de correo mutable para listas de permitidos (modo de compatibilidad de emergencia).
-   Las reacciones están disponibles a través de la herramienta `reactions` y `channels action` cuando `actions.reactions` está habilitado.
-   `typingIndicator` admite `none`, `message` (predeterminado) y `reaction` (la reacción requiere OAuth de usuario).
-   Los archivos adjuntos se descargan a través de la API de Chat y se almacenan en el pipeline de medios (tamaño limitado por `mediaMaxMb`).

Detalles de referencia de secretos: [Gestión de Secretos](../gateway/secrets.md).

## Solución de problemas

### 405 Método No Permitido

Si el Explorador de registros de Google Cloud muestra errores como:

```bash
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

Esto significa que el manejador del webhook no está registrado. Causas comunes:

1.  **Canal no configurado**: Falta la sección `channels.googlechat` en tu configuración. Verifica con:
    
    Copiar
    
    ```bash
    openclaw config get channels.googlechat
    ```
    
    Si devuelve "Ruta de configuración no encontrada", agrega la configuración (ver [Aspectos destacados de la configuración](#config-highlights)).
2.  **Plugin no habilitado**: Verifica el estado del plugin:
    
    Copiar
    
    ```bash
    openclaw plugins list | grep googlechat
    ```
    
    Si muestra "disabled", agrega `plugins.entries.googlechat.enabled: true` a tu configuración.
3.  **Puerta de enlace no reiniciada**: Después de agregar la configuración, reinicia la puerta de enlace:
    
    Copiar
    
    ```bash
    openclaw gateway restart
    ```
    

Verifica que el canal esté en ejecución:

```bash
openclaw channels status
# Debería mostrar: Google Chat default: enabled, configured, ...
```

### Otros problemas

-   Ejecuta `openclaw channels status --probe` para ver errores de autenticación o configuración de audiencia faltante.
-   Si no llegan mensajes, confirma la URL del webhook de la aplicación de Chat + suscripciones a eventos.
-   Si la compuerta de mención bloquea respuestas, establece `botUser` al nombre de recurso de usuario de la aplicación y verifica `requireMention`.
-   Usa `openclaw logs --follow` mientras envías un mensaje de prueba para ver si las solicitudes llegan a la puerta de enlace.

Documentación relacionada:

-   [Configuración de la puerta de enlace](../gateway/configuration.md)
-   [Seguridad](../gateway/security.md)
-   [Reacciones](../tools/reactions.md)

[Feishu](./feishu.md)[iMessage](./imessage.md)