

  Configuración y operaciones

  
# Autenticación

OpenClaw admite OAuth y claves API para proveedores de modelos. Para hosts de puerta de enlace siempre activos, las claves API suelen ser la opción más predecible. Los flujos de suscripción/OAuth también son compatibles cuando coinciden con el modelo de cuenta de tu proveedor. Consulta [/concepts/oauth](../concepts/oauth.md) para ver el flujo completo de OAuth y el diseño de almacenamiento. Para la autenticación basada en SecretRef (proveedores `env`/`file`/`exec`), consulta [Gestión de Secretos](./secrets.md). Para las reglas de elegibilidad/código de motivo de credenciales utilizadas por `models status --probe`, consulta [Semántica de Credenciales de Autenticación](../auth-credential-semantics.md).

## Configuración recomendada (Clave API, cualquier proveedor)

Si ejecutas una puerta de enlace de larga duración, comienza con una clave API para tu proveedor elegido. Para Anthropic específicamente, la autenticación por clave API es el camino seguro y se recomienda sobre la autenticación por token de configuración de suscripción.

1.  Crea una clave API en la consola de tu proveedor.
2.  Colócala en el **host de la puerta de enlace** (la máquina que ejecuta `openclaw gateway`).

```bash
export <PROVIDER>_API_KEY="..."
openclaw models status
```

3.  Si la Puerta de enlace se ejecuta bajo systemd/launchd, prefiere colocar la clave en `~/.openclaw/.env` para que el demonio pueda leerla:

```bash
cat >> ~/.openclaw/.env <<'EOF'
<PROVIDER>_API_KEY=...
EOF
```

Luego reinicia el demonio (o reinicia tu proceso de Puerta de enlace) y verifica de nuevo:

```bash
openclaw models status
openclaw doctor
```

Si prefieres no gestionar las variables de entorno tú mismo, el asistente de incorporación puede almacenar claves API para uso del demonio: `openclaw onboard`. Consulta [Ayuda](../help.md) para detalles sobre la herencia de variables de entorno (`env.shellEnv`, `~/.openclaw/.env`, systemd/launchd).

## Anthropic: token de configuración (autenticación por suscripción)

Si estás usando una suscripción a Claude, el flujo de token de configuración es compatible. Ejecútalo en el **host de la puerta de enlace**:

```bash
claude setup-token
```

Luego pégala en OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

Si el token se creó en otra máquina, pégala manualmente:

```bash
openclaw models auth paste-token --provider anthropic
```

Si ves un error de Anthropic como:

```bash
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

…usa una clave API de Anthropic en su lugar.

> **⚠️** La compatibilidad con el token de configuración de Anthropic es solo técnica. Anthropic ha bloqueado algunos usos de suscripción fuera de Claude Code en el pasado. Úsalo solo si decides que el riesgo de política es aceptable, y verifica los términos actuales de Anthropic tú mismo.

 Entrada manual de token (cualquier proveedor; escribe `auth-profiles.json` + actualiza la configuración):

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

Las referencias a perfiles de autenticación también son compatibles para credenciales estáticas:

-   Las credenciales `api_key` pueden usar `keyRef: { source, provider, id }`
-   Las credenciales `token` pueden usar `tokenRef: { source, provider, id }`

Verificación apta para automatización (salida `1` cuando caduca/falta, `2` cuando está por caducar):

```bash
openclaw models status --check
```

Los scripts de operaciones opcionales (systemd/Termux) están documentados aquí: [/automation/auth-monitoring](../automation/auth-monitoring.md)

> `claude setup-token` requiere una TTY interactiva.

## Verificando el estado de autenticación del modelo

```bash
openclaw models status
openclaw doctor
```

## Comportamiento de rotación de clave API (puerta de enlace)

Algunos proveedores admiten reintentar una solicitud con claves alternativas cuando una llamada API alcanza un límite de tasa del proveedor.

-   Orden de prioridad:
    -   `OPENCLAW_LIVE__KEY` (anulación única)
    -   `_API_KEYS`
    -   `_API_KEY`
    -   `_API_KEY_*`
-   Los proveedores de Google también incluyen `GOOGLE_API_KEY` como respaldo adicional.
-   La misma lista de claves se desduplica antes de su uso.
-   OpenClaw reintenta con la siguiente clave solo para errores de límite de tasa (por ejemplo `429`, `rate_limit`, `quota`, `resource exhausted`).
-   Los errores que no son de límite de tasa no se reintentan con claves alternativas.
-   Si todas las claves fallan, se devuelve el último error del último intento.

## Controlando qué credencial se usa

### Por sesión (comando de chat)

Usa `/model <alias-or-id>@` para fijar una credencial de proveedor específica para la sesión actual (ejemplos de ids de perfil: `anthropic:default`, `anthropic:work`). Usa `/model` (o `/model list`) para un selector compacto; usa `/model status` para la vista completa (candidatos + siguiente perfil de autenticación, más detalles del endpoint del proveedor cuando esté configurado).

### Por agente (anulación CLI)

Establece un orden de perfil de autenticación explícito para un agente (almacenado en el `auth-profiles.json` de ese agente):

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

Usa `--agent ` para apuntar a un agente específico; omítelo para usar el agente predeterminado configurado.

## Resolución de problemas

### “No se encontraron credenciales”

Si falta el perfil de token de Anthropic, ejecuta `claude setup-token` en el **host de la puerta de enlace**, luego verifica de nuevo:

```bash
openclaw models status
```

### Token por caducar/caducado

Ejecuta `openclaw models status` para confirmar qué perfil está por caducar. Si falta el perfil, vuelve a ejecutar `claude setup-token` y pega el token nuevamente.

## Requisitos

-   Cuenta de suscripción de Anthropic (para `claude setup-token`)
-   CLI de Claude Code instalado (comando `claude` disponible)

[Ejemplos de Configuración](./configuration-examples.md)[Semántica de credenciales de autenticación](../auth-credential-semantics.md)

---