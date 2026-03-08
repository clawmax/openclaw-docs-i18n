

  Entorno y depuración

  
# Variables de Entorno

OpenClaw obtiene variables de entorno desde múltiples fuentes. La regla es **nunca sobrescribir valores existentes**.

## Precedencia (mayor → menor)

1.  **Entorno del proceso** (lo que el proceso Gateway ya tiene del shell/daemon padre).
2.  **`.env` en el directorio de trabajo actual** (dotenv por defecto; no sobrescribe).
3.  **`.env` global** en `~/.openclaw/.env` (también `$OPENCLAW_STATE_DIR/.env`; no sobrescribe).
4.  **Bloque `env` de configuración** en `~/.openclaw/openclaw.json` (se aplica solo si falta).
5.  **Importación opcional del shell de inicio de sesión** (`env.shellEnv.enabled` o `OPENCLAW_LOAD_SHELL_ENV=1`), aplicada solo para claves esperadas faltantes.

Si el archivo de configuración falta por completo, se omite el paso 4; la importación del shell aún se ejecuta si está habilitada.

## Bloque env de configuración

Dos formas equivalentes de establecer variables de entorno en línea (ambas no sobrescriben):

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

## Importación de entorno del shell

`env.shellEnv` ejecuta tu shell de inicio de sesión e importa solo las claves esperadas **faltantes**:

```json
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Variables de entorno equivalentes:

-   `OPENCLAW_LOAD_SHELL_ENV=1`
-   `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## Variables de entorno inyectadas en tiempo de ejecución

OpenClaw también inyecta marcadores de contexto en los procesos hijos generados:

-   `OPENCLAW_SHELL=exec`: se establece para comandos ejecutados a través de la herramienta `exec`.
-   `OPENCLAW_SHELL=acp`: se establece para los procesos generados por el backend de tiempo de ejecución ACP (por ejemplo `acpx`).
-   `OPENCLAW_SHELL=acp-client`: se establece para `openclaw acp client` cuando genera el proceso puente ACP.
-   `OPENCLAW_SHELL=tui-local`: se establece para comandos de shell `!` locales en la TUI.

Estos son marcadores de tiempo de ejecución (no configuración requerida del usuario). Se pueden usar en la lógica del shell/perfil para aplicar reglas específicas del contexto.

## Sustitución de variables de entorno en la configuración

Puedes hacer referencia a variables de entorno directamente en los valores de cadena de configuración usando la sintaxis `${VAR_NAME}`:

```json
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

Consulta [Configuración: Sustitución de variables de entorno](../gateway/configuration.md#env-var-substitution-in-config) para más detalles.

## Referencias secretas vs cadenas `\${ENV}`

OpenClaw admite dos patrones basados en entorno:

-   Sustitución de cadena `${VAR}` en valores de configuración.
-   Objetos SecretRef (`{ source: "env", provider: "default", id: "VAR" }`) para campos que admiten referencias secretas.

Ambos se resuelven desde el entorno del proceso en el momento de la activación. Los detalles de SecretRef están documentados en [Gestión de Secretos](../gateway/secrets.md).

## Variables de entorno relacionadas con rutas

| Variable | Propósito |
| --- | --- |
| `OPENCLAW_HOME` | Anula el directorio principal utilizado para toda la resolución de rutas internas (`~/.openclaw/`, directorios de agentes, sesiones, credenciales). Útil cuando se ejecuta OpenClaw como un usuario de servicio dedicado. |
| `OPENCLAW_STATE_DIR` | Anula el directorio de estado (por defecto `~/.openclaw`). |
| `OPENCLAW_CONFIG_PATH` | Anula la ruta del archivo de configuración (por defecto `~/.openclaw/openclaw.json`). |

## Registro (Logging)

| Variable | Propósito |
| --- | --- |
| `OPENCLAW_LOG_LEVEL` | Anula el nivel de registro tanto para archivo como para consola (ej. `debug`, `trace`). Tiene precedencia sobre `logging.level` y `logging.consoleLevel` en la configuración. Los valores no válidos se ignoran con una advertencia. |

### OPENCLAW\_HOME

Cuando se establece, `OPENCLAW_HOME` reemplaza el directorio principal del sistema (`$HOME` / `os.homedir()`) para toda la resolución de rutas internas. Esto permite un aislamiento completo del sistema de archivos para cuentas de servicio sin interfaz gráfica. **Precedencia:** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()` **Ejemplo** (macOS LaunchDaemon):

```
<key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

`OPENCLAW_HOME` también se puede establecer como una ruta con tilde (ej. `~/svc`), que se expande usando `$HOME` antes de su uso.

## Relacionado

-   [Configuración de Gateway](../gateway/configuration.md)
-   [Preguntas frecuentes: variables de entorno y carga de .env](./faq.md#env-vars-and-env-loading)
-   [Descripción general de Modelos](../concepts/models.md)

[Lore de OpenClaw](../start/lore.md)[Depuración](./debugging.md)