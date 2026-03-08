

  Automatización

  
# Gmail PubSub

Objetivo: Vigilancia de Gmail -> Push de Pub/Sub -> `gog gmail watch serve` -> Webhook de OpenClaw.

## Requisitos previos

-   `gcloud` instalado y con sesión iniciada ([guía de instalación](https://docs.cloud.google.com/sdk/docs/install-sdk)).
-   `gog` (gogcli) instalado y autorizado para la cuenta de Gmail ([gogcli.sh](https://gogcli.sh/)).
-   Hooks de OpenClaw habilitados (ver [Webhooks](./webhook.md)).
-   `tailscale` con sesión iniciada ([tailscale.com](https://tailscale.com/)). La configuración admitida utiliza Tailscale Funnel para el endpoint HTTPS público. Otros servicios de túnel pueden funcionar, pero son DIY/no admitidos y requieren configuración manual. Por ahora, Tailscale es lo que admitimos.

Ejemplo de configuración de hook (habilitar mapeo predefinido de Gmail):

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"],
  },
}
```

Para entregar el resumen de Gmail a una superficie de chat, anula el predefinido con un mapeo que establezca `deliver` + opcionalmente `channel`/`to`:

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "Nuevo correo de {{messages[0].from}}\nAsunto: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last",
        // to: "+15551234567"
      },
    ],
  },
}
```

Si quieres un canal fijo, establece `channel` + `to`. De lo contrario, `channel: "last"` usa la última ruta de entrega (recurre a WhatsApp). Para forzar un modelo más económico para las ejecuciones de Gmail, establece `model` en el mapeo (`provider/model` o alias). Si aplicas `agents.defaults.models`, inclúyelo allí. Para establecer un modelo y nivel de pensamiento predeterminados específicamente para hooks de Gmail, agrega `hooks.gmail.model` / `hooks.gmail.thinking` en tu configuración:

```json
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

Notas:

-   El `model`/`thinking` por hook en el mapeo aún anula estos valores predeterminados.
-   Orden de respaldo: `hooks.gmail.model` → `agents.defaults.model.fallbacks` → principal (autenticación/límite de tasa/timeouts).
-   Si `agents.defaults.models` está establecido, el modelo de Gmail debe estar en la lista de permitidos.
-   El contenido del hook de Gmail está envuelto con límites de seguridad de contenido externo por defecto. Para deshabilitar (peligroso), establece `hooks.gmail.allowUnsafeExternalContent: true`.

Para personalizar aún más el manejo de la carga útil, agrega `hooks.mappings` o un módulo de transformación JS/TS bajo `~/.openclaw/hooks/transforms` (ver [Webhooks](./webhook.md)).

## Asistente (recomendado)

Usa el asistente de OpenClaw para conectar todo (instala dependencias en macOS via brew):

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

Valores predeterminados:

-   Usa Tailscale Funnel para el endpoint push público.
-   Escribe la configuración `hooks.gmail` para `openclaw webhooks gmail run`.
-   Habilita el predefinido de hook de Gmail (`hooks.presets: ["gmail"]`).

Nota sobre la ruta: cuando `tailscale.mode` está habilitado, OpenClaw establece automáticamente `hooks.gmail.serve.path` en `/` y mantiene la ruta pública en `hooks.gmail.tailscale.path` (predeterminado `/gmail-pubsub`) porque Tailscale elimina el prefijo de ruta establecido antes de redirigir. Si necesitas que el backend reciba la ruta con prefijo, establece `hooks.gmail.tailscale.target` (o `--tailscale-target`) a una URL completa como `http://127.0.0.1:8788/gmail-pubsub` y hazla coincidir con `hooks.gmail.serve.path`. ¿Quieres un endpoint personalizado? Usa `--push-endpoint ` o `--tailscale off`. Nota sobre plataforma: en macOS el asistente instala `gcloud`, `gogcli` y `tailscale` via Homebrew; en Linux instálalos manualmente primero. Inicio automático del Gateway (recomendado):

-   Cuando `hooks.enabled=true` y `hooks.gmail.account` está establecido, el Gateway inicia `gog gmail watch serve` al arrancar y renueva automáticamente la vigilancia.
-   Establece `OPENCLAW_SKIP_GMAIL_WATCHER=1` para excluirte (útil si ejecutas el daemon tú mismo).
-   No ejecutes el daemon manual al mismo tiempo, o obtendrás `listen tcp 127.0.0.1:8788: bind: address already in use`.

Daemon manual (inicia `gog gmail watch serve` + renovación automática):

```bash
openclaw webhooks gmail run
```

## Configuración única

1.  Selecciona el proyecto de GCP **que posee el cliente OAuth** usado por `gog`.

```bash
gcloud auth login
gcloud config set project <project-id>
```

Nota: La vigilancia de Gmail requiere que el tema de Pub/Sub resida en el mismo proyecto que el cliente OAuth.

2.  Habilita las APIs:

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3.  Crea un tema:

```bash
gcloud pubsub topics create gog-gmail-watch
```

4.  Permite que el push de Gmail publique:

```
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

## Iniciar la vigilancia

```
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

Guarda el `history_id` de la salida (para depuración).

## Ejecutar el manejador push

Ejemplo local (autenticación por token compartido):

```
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

Notas:

-   `--token` protege el endpoint push (`x-gog-token` o `?token=`).
-   `--hook-url` apunta a OpenClaw `/hooks/gmail` (mapeado; ejecución aislada + resumen al principal).
-   `--include-body` y `--max-bytes` controlan el fragmento del cuerpo enviado a OpenClaw.

Recomendado: `openclaw webhooks gmail run` envuelve el mismo flujo y renueva automáticamente la vigilancia.

## Exponer el manejador (avanzado, no admitido)

Si necesitas un túnel que no sea Tailscale, configúralo manualmente y usa la URL pública en la suscripción push (no admitido, sin protecciones):

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

Usa la URL generada como endpoint push:

```
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

Producción: usa un endpoint HTTPS estable y configura OIDC JWT de Pub/Sub, luego ejecuta:

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

## Probar

Envía un mensaje a la bandeja de entrada vigilada:

```
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "prueba de vigilancia" \
  --body "ping"
```

Verifica el estado de la vigilancia y el historial:

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

## Solución de problemas

-   `Invalid topicName`: discrepancia del proyecto (el tema no está en el proyecto del cliente OAuth).
-   `User not authorized`: falta `roles/pubsub.publisher` en el tema.
-   Mensajes vacíos: el push de Gmail solo proporciona `historyId`; obtén los datos via `gog gmail history`.

## Limpieza

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```

[Webhooks](./webhook.md)[Encuestas](./poll.md)