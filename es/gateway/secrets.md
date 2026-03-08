

  Configuración y operaciones

  
# Gestión de Secretos

OpenClaw admite SecretRefs aditivos, por lo que las credenciales compatibles no necesitan almacenarse en texto plano en la configuración. El texto plano sigue funcionando. Los SecretRefs son opcionales por credencial.

## Objetivos y modelo de tiempo de ejecución

Los secretos se resuelven en una instantánea en memoria en tiempo de ejecución.

-   La resolución es ansiosa durante la activación, no perezosa en las rutas de solicitud.
-   El inicio falla rápidamente cuando un SecretRef efectivamente activo no puede resolverse.
-   La recarga utiliza intercambio atómico: éxito completo o se mantiene la última instantánea buena conocida.
-   Las solicitudes en tiempo de ejecución leen solo de la instantánea activa en memoria.

Esto mantiene las interrupciones del proveedor de secretos fuera de las rutas de solicitud activas.

## Filtrado de superficie activa

Los SecretRefs se validan solo en superficies efectivamente activas.

-   Superficies habilitadas: las referencias no resueltas bloquean el inicio/recarga.
-   Superficies inactivas: las referencias no resueltas no bloquean el inicio/recarga.
-   Las referencias inactivas emiten diagnósticos no fatales con el código `SECRETS_REF_IGNORED_INACTIVE_SURFACE`.

Ejemplos de superficies inactivas:

-   Entradas de canal/cuenta deshabilitadas.
-   Credenciales de canal de nivel superior que ninguna cuenta habilitada hereda.
-   Superficies de herramienta/funcionalidad deshabilitadas.
-   Claves específicas del proveedor de búsqueda web que no están seleccionadas por `tools.web.search.provider`. En modo automático (proveedor no establecido), las claves específicas del proveedor también están activas para la detección automática del proveedor.
-   Los SecretRefs `gateway.remote.token` / `gateway.remote.password` están activos (cuando `gateway.remote.enabled` no es `false`) si una de estas condiciones es verdadera:
    -   `gateway.mode=remote`
    -   `gateway.remote.url` está configurado
    -   `gateway.tailscale.mode` es `serve` o `funnel` En modo local sin esas superficies remotas:
    -   `gateway.remote.token` está activo cuando la autenticación por token puede ganar y no hay token de entorno/auth configurado.
    -   `gateway.remote.password` está activo solo cuando la autenticación por contraseña puede ganar y no hay contraseña de entorno/auth configurada.
-   El SecretRef `gateway.auth.token` está inactivo para la resolución de autenticación de inicio cuando `OPENCLAW_GATEWAY_TOKEN` (o `CLAWDBOT_GATEWAY_TOKEN`) está establecido, porque la entrada del token de entorno gana para ese tiempo de ejecución.

## Diagnósticos de superficie de autenticación del gateway

Cuando se configura un SecretRef en `gateway.auth.token`, `gateway.auth.password`, `gateway.remote.token` o `gateway.remote.password`, el inicio/recarga del gateway registra el estado de la superficie explícitamente:

-   `active`: el SecretRef es parte de la superficie de autenticación efectiva y debe resolverse.
-   `inactive`: el SecretRef se ignora para este tiempo de ejecución porque otra superficie de autenticación gana, o porque la autenticación remota está deshabilitada/no activa.

Estas entradas se registran con `SECRETS_GATEWAY_AUTH_SURFACE` e incluyen la razón utilizada por la política de superficie activa, para que puedas ver por qué una credencial fue tratada como activa o inactiva.

## Preflight de referencia de incorporación

Cuando la incorporación se ejecuta en modo interactivo y eliges almacenamiento SecretRef, OpenClaw ejecuta validación preflight antes de guardar:

-   Referencias de entorno: valida el nombre de la variable de entorno y confirma que un valor no vacío es visible durante la incorporación.
-   Referencias de proveedor (`file` o `exec`): valida la selección del proveedor, resuelve `id` y verifica el tipo de valor resuelto.
-   Ruta de reutilización de inicio rápido: cuando `gateway.auth.token` ya es un SecretRef, la incorporación lo resuelve antes del sondeo/bootstrap del panel de control (para referencias `env`, `file` y `exec`) usando la misma compuerta de fallo rápido.

Si la validación falla, la incorporación muestra el error y te permite reintentar.

## Contrato SecretRef

Usa una forma de objeto en todas partes:

```json
{ source: "env" | "file" | "exec", provider: "default", id: "..." }
```

### source: "env"

```json
{ source: "env", provider: "default", id: "OPENAI_API_KEY" }
```

Validación:

-   `provider` debe coincidir con `^[a-z][a-z0-9_-]{0,63}$`
-   `id` debe coincidir con `^[A-Z][A-Z0-9_]{0,127}$`

### source: "file"

```json
{ source: "file", provider: "filemain", id: "/providers/openai/apiKey" }
```

Validación:

-   `provider` debe coincidir con `^[a-z][a-z0-9_-]{0,63}$`
-   `id` debe ser un puntero JSON absoluto (`/...`)
-   Escapado RFC6901 en segmentos: `~` => `~0`, `/` => `~1`

### source: "exec"

```json
{ source: "exec", provider: "vault", id: "providers/openai/apiKey" }
```

Validación:

-   `provider` debe coincidir con `^[a-z][a-z0-9_-]{0,63}$`
-   `id` debe coincidir con `^[A-Za-z0-9][A-Za-z0-9._:/-]{0,255}$`

## Configuración del proveedor

Define proveedores bajo `secrets.providers`:

```json
{
  secrets: {
    providers: {
      default: { source: "env" },
      filemain: {
        source: "file",
        path: "~/.openclaw/secrets.json",
        mode: "json", // o "singleValue"
      },
      vault: {
        source: "exec",
        command: "/usr/local/bin/openclaw-vault-resolver",
        args: ["--profile", "prod"],
        passEnv: ["PATH", "VAULT_ADDR"],
        jsonOnly: true,
      },
    },
    defaults: {
      env: "default",
      file: "filemain",
      exec: "vault",
    },
    resolution: {
      maxProviderConcurrency: 4,
      maxRefsPerProvider: 512,
      maxBatchBytes: 262144,
    },
  },
}
```

### Proveedor de entorno

-   Lista blanca opcional mediante `allowlist`.
-   Los valores de entorno faltantes/vacíos fallan la resolución.

### Proveedor de archivo

-   Lee el archivo local desde `path`.
-   `mode: "json"` espera un payload de objeto JSON y resuelve `id` como puntero.
-   `mode: "singleValue"` espera el id de referencia `"value"` y devuelve el contenido del archivo.
-   La ruta debe pasar las comprobaciones de propiedad/permisos.
-   Nota de fallo cerrado en Windows: si la verificación de ACL no está disponible para una ruta, la resolución falla. Solo para rutas confiables, establece `allowInsecurePath: true` en ese proveedor para omitir las comprobaciones de seguridad de ruta.

### Proveedor de ejecución

-   Ejecuta la ruta binaria absoluta configurada, sin shell.
-   Por defecto, `command` debe apuntar a un archivo regular (no un enlace simbólico).
-   Establece `allowSymlinkCommand: true` para permitir rutas de comando de enlace simbólico (por ejemplo, shims de Homebrew). OpenClaw valida la ruta de destino resuelta.
-   Combina `allowSymlinkCommand` con `trustedDirs` para rutas de gestores de paquetes (por ejemplo `["/opt/homebrew"]`).
-   Admite tiempo de espera, tiempo de espera sin salida, límites de bytes de salida, lista blanca de entorno y directorios confiables.
-   Nota de fallo cerrado en Windows: si la verificación de ACL no está disponible para la ruta del comando, la resolución falla. Solo para rutas confiables, establece `allowInsecurePath: true` en ese proveedor para omitir las comprobaciones de seguridad de ruta.

Payload de solicitud (stdin):

```json
{ "protocolVersion": 1, "provider": "vault", "ids": ["providers/openai/apiKey"] }
```

Payload de respuesta (stdout):

```json
{ "protocolVersion": 1, "values": { "providers/openai/apiKey": "<openai-api-key>" } } // pragma: allowlist secret
```

Errores opcionales por id:

```json
{
  "protocolVersion": 1,
  "values": {},
  "errors": { "providers/openai/apiKey": { "message": "not found" } }
}
```

## Ejemplos de integración de ejecución

### CLI de 1Password

```json
{
  secrets: {
    providers: {
      onepassword_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/op",
        allowSymlinkCommand: true, // requerido para binarios enlazados simbólicamente de Homebrew
        trustedDirs: ["/opt/homebrew"],
        args: ["read", "op://Personal/OpenClaw QA API Key/password"],
        passEnv: ["HOME"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "onepassword_openai", id: "value" },
      },
    },
  },
}
```

### CLI de HashiCorp Vault

```json
{
  secrets: {
    providers: {
      vault_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/vault",
        allowSymlinkCommand: true, // requerido para binarios enlazados simbólicamente de Homebrew
        trustedDirs: ["/opt/homebrew"],
        args: ["kv", "get", "-field=OPENAI_API_KEY", "secret/openclaw"],
        passEnv: ["VAULT_ADDR", "VAULT_TOKEN"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "vault_openai", id: "value" },
      },
    },
  },
}
```

### sops

```json
{
  secrets: {
    providers: {
      sops_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/sops",
        allowSymlinkCommand: true, // requerido para binarios enlazados simbólicamente de Homebrew
        trustedDirs: ["/opt/homebrew"],
        args: ["-d", "--extract", '["providers"]["openai"]["apiKey"]', "/path/to/secrets.enc.json"],
        passEnv: ["SOPS_AGE_KEY_FILE"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "sops_openai", id: "value" },
      },
    },
  },
}
```

## Superficie de credenciales compatible

Las credenciales canónicamente compatibles y no compatibles se enumeran en:

-   [Superficie de Credenciales SecretRef](../reference/secretref-credential-surface.md)

Las credenciales generadas en tiempo de ejecución o rotativas y el material de actualización de OAuth están intencionalmente excluidos de la resolución de solo lectura de SecretRef.

## Comportamiento requerido y precedencia

-   Campo sin referencia: sin cambios.
-   Campo con referencia: requerido en superficies activas durante la activación.
-   Si están presentes tanto texto plano como referencia, la referencia tiene precedencia en las rutas de precedencia compatibles.

Señales de advertencia y auditoría:

-   `SECRETS_REF_OVERRIDES_PLAINTEXT` (advertencia de tiempo de ejecución)
-   `REF_SHADOWED` (hallazgo de auditoría cuando las credenciales de `auth-profiles.json` toman precedencia sobre las referencias de `openclaw.json`)

Comportamiento de compatibilidad de Google Chat:

-   `serviceAccountRef` tiene precedencia sobre el texto plano `serviceAccount`.
-   El valor en texto plano se ignora cuando se establece una referencia hermana.

## Desencadenantes de activación

La activación de secretos se ejecuta en:

-   Inicio (preflight más activación final)
-   Ruta de aplicación en caliente de recarga de configuración
-   Ruta de verificación de reinicio de recarga de configuración
-   Recarga manual mediante `secrets.reload`

Contrato de activación:

-   El éxito intercambia la instantánea atómicamente.
-   El fallo de inicio aborta el inicio del gateway.
-   El fallo de recarga en tiempo de ejecución mantiene la última instantánea buena conocida.

## Señales degradadas y recuperadas

Cuando la activación en tiempo de recarga falla después de un estado saludable, OpenClaw entra en estado degradado de secretos. Evento de sistema único y códigos de registro:

-   `SECRETS_RELOADER_DEGRADED`
-   `SECRETS_RELOADER_RECOVERED`

Comportamiento:

-   Degradado: el tiempo de ejecución mantiene la última instantánea buena conocida.
-   Recuperado: se emite una vez después de la siguiente activación exitosa.
-   Los fallos repetidos mientras ya está degradado registran advertencias pero no generan spam de eventos.
-   El fallo rápido de inicio no emite eventos degradados porque el tiempo de ejecución nunca se volvió activo.

## Resolución de ruta de comando

Las rutas de comando pueden optar por la resolución SecretRef compatible a través de RPC de instantánea del gateway. Hay dos comportamientos amplios:

-   Rutas de comando estrictas (por ejemplo, rutas de memoria remota `openclaw memory` y `openclaw qr --remote`) leen de la instantánea activa y fallan rápidamente cuando un SecretRef requerido no está disponible.
-   Rutas de comando de solo lectura (por ejemplo, `openclaw status`, `openclaw status --all`, `openclaw channels status`, `openclaw channels resolve` y flujos de reparación de configuración/doctor de solo lectura) también prefieren la instantánea activa, pero se degradan en lugar de abortar cuando un SecretRef específico no está disponible en esa ruta de comando.

Comportamiento de solo lectura:

-   Cuando el gateway está en ejecución, estos comandos leen primero de la instantánea activa.
-   Si la resolución del gateway está incompleta o el gateway no está disponible, intentan un respaldo local específico para la superficie de comando específica.
-   Si un SecretRef específico aún no está disponible, el comando continúa con salida degradada de solo lectura y diagnósticos explícitos como "configurado pero no disponible en esta ruta de comando".
-   Este comportamiento degradado es solo local al comando. No debilita el inicio, recarga o rutas de envío/autenticación del tiempo de ejecución.

Otras notas:

-   La actualización de instantánea después de la rotación de secretos del backend se maneja con `openclaw secrets reload`.
-   Método RPC del gateway utilizado por estas rutas de comando: `secrets.resolve`.

## Flujo de trabajo de auditoría y configuración

Flujo de operador predeterminado:

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

### secrets audit

Los hallazgos incluyen:

-   valores en texto plano en reposo (`openclaw.json`, `auth-profiles.json`, `.env` y `agents/*/agent/models.json` generados)
-   residuos de encabezados sensibles de proveedor en texto plano en entradas `models.json` generadas
-   referencias no resueltas
-   sombreado de precedencia (`auth-profiles.json` tomando prioridad sobre referencias de `openclaw.json`)
-   residuos heredados (`auth.json`, recordatorios de OAuth)

Nota sobre residuos de encabezado:

-   La detección de encabezados sensibles de proveedor se basa en heurística de nombres (nombres y fragmentos comunes de encabezados de autenticación/credenciales como `authorization`, `x-api-key`, `token`, `secret`, `password` y `credential`).

### secrets configure

Ayudante interactivo que:

-   configura primero `secrets.providers` (`env`/`file`/`exec`, agregar/editar/eliminar)
-   te permite seleccionar campos que contienen secretos compatibles en `openclaw.json` más `auth-profiles.json` para un alcance de agente
-   puede crear un nuevo mapeo `auth-profiles.json` directamente en el selector de destino
-   captura detalles de SecretRef (`source`, `provider`, `id`)
-   ejecuta resolución preflight
-   puede aplicar inmediatamente

Modos útiles:

-   `openclaw secrets configure --providers-only`
-   `openclaw secrets configure --skip-provider-setup`
-   `openclaw secrets configure --agent `

Aplicar valores predeterminados de `configure`:

-   limpia las credenciales estáticas coincidentes de `auth-profiles.json` para proveedores específicos
-   limpia las entradas estáticas heredadas `api_key` de `auth.json`
-   limpia las líneas secretas conocidas coincidentes de `<config-dir>/.env`

### secrets apply

Aplica un plan guardado:

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
```

Para detalles estrictos de contrato de ruta/objetivo y reglas de rechazo exactas, consulta:

-   [Contrato de Plan de Aplicación de Secretos](./secrets-plan-contract.md)

## Política de seguridad unidireccional

OpenClaw intencionalmente no escribe copias de seguridad de reversión que contengan valores de secretos en texto plano históricos. Modelo de seguridad:

-   el preflight debe tener éxito antes del modo de escritura
-   la activación del tiempo de ejecución se valida antes del commit
-   la aplicación actualiza archivos usando reemplazo atómico de archivos y restauración de mejor esfuerzo en caso de fallo

## Notas de compatibilidad de autenticación heredada

Para credenciales estáticas, el tiempo de ejecución ya no depende del almacenamiento de autenticación heredado en texto plano.

-   La fuente de credenciales del tiempo de ejecución es la instantánea en memoria resuelta.
-   Las entradas estáticas heredadas `api_key` se limpian cuando se descubren.
-   El comportamiento de compatibilidad relacionado con OAuth permanece separado.

## Nota sobre la interfaz de usuario web

Algunas uniones SecretInput son más fáciles de configurar en modo editor de texto sin formato que en modo formulario.

## Documentación relacionada

-   Comandos CLI: [secrets](../cli/secrets.md)
-   Detalles del contrato del plan: [Contrato de Plan de Aplicación de Secretos](./secrets-plan-contract.md)
-   Superficie de credenciales: [Superficie de Credenciales SecretRef](../reference/secretref-credential-surface.md)
-   Configuración de autenticación: [Autenticación](./authentication.md)
-   Postura de seguridad: [Seguridad](./security.md)
-   Precedencia de entorno: [Variables de Entorno](../help/environment.md)

[Semántica de Credenciales de Autenticación](../auth-credential-semantics.md)[Contrato de Plan de Aplicación de Secretos](./secrets-plan-contract.md)