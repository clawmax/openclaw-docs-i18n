

  Protocolos y APIs

  
# Backends CLI

OpenClaw puede ejecutar **CLIs de IA local** como un **fallback de solo texto** cuando los proveedores de API están caídos, limitados por tasa o funcionando mal temporalmente. Esto es intencionalmente conservador:

-   **Las herramientas están deshabilitadas** (sin llamadas a herramientas).
-   **Texto de entrada → texto de salida** (confiable).
-   **Las sesiones son compatibles** (para que los turnos de seguimiento se mantengan coherentes).
-   **Las imágenes pueden pasarse** si el CLI acepta rutas de imagen.

Está diseñado como una **red de seguridad** en lugar de una ruta principal. Úsalo cuando quieras respuestas de texto que "siempre funcionen" sin depender de APIs externas.

## Inicio rápido para principiantes

Puedes usar Claude Code CLI **sin ninguna configuración** (OpenClaw incluye un valor predeterminado integrado):

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.6
```

Codex CLI también funciona listo para usar:

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.4
```

Si tu puerta de enlace se ejecuta bajo launchd/systemd y el PATH es mínimo, agrega solo la ruta del comando:

```json
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
      },
    },
  },
}
```

Eso es todo. Sin claves, sin configuración de autenticación adicional más allá del propio CLI.

## Usándolo como fallback

Agrega un backend CLI a tu lista de fallback para que solo se ejecute cuando fallen los modelos principales:

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["claude-cli/opus-4.6", "claude-cli/opus-4.5"],
      },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "claude-cli/opus-4.6": {},
        "claude-cli/opus-4.5": {},
      },
    },
  },
}
```

Notas:

-   Si usas `agents.defaults.models` (lista de permitidos), debes incluir `claude-cli/...`.
-   Si el proveedor principal falla (autenticación, límites de tasa, tiempos de espera), OpenClaw intentará el backend CLI a continuación.

## Resumen de configuración

Todos los backends CLI residen bajo:

```
agents.defaults.cliBackends
```

Cada entrada se identifica con un **id de proveedor** (ej. `claude-cli`, `my-cli`). El id del proveedor se convierte en el lado izquierdo de tu referencia de modelo:

```
<proveedor>/<modelo>
```

### Ejemplo de configuración

```json
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-6": "opus",
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet",
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true,
        },
      },
    },
  },
}
```

## Cómo funciona

1.  **Selecciona un backend** basado en el prefijo del proveedor (`claude-cli/...`).
2.  **Construye un prompt del sistema** usando el mismo prompt de OpenClaw + contexto del espacio de trabajo.
3.  **Ejecuta el CLI** con un id de sesión (si es compatible) para que el historial se mantenga consistente.
4.  **Analiza la salida** (JSON o texto plano) y devuelve el texto final.
5.  **Persiste los ids de sesión** por backend, para que las continuaciones reutilicen la misma sesión CLI.

## Sesiones

-   Si el CLI admite sesiones, configura `sessionArg` (ej. `--session-id`) o `sessionArgs` (marcador de posición `{sessionId}`) cuando el ID deba insertarse en múltiples banderas.
-   Si el CLI usa un **subcomando de reanudación** con banderas diferentes, configura `resumeArgs` (reemplaza `args` al reanudar) y opcionalmente `resumeOutput` (para reanudaciones no JSON).
-   `sessionMode`:
    -   `always`: siempre envía un id de sesión (nuevo UUID si no hay ninguno almacenado).
    -   `existing`: solo envía un id de sesión si uno fue almacenado antes.
    -   `none`: nunca envía un id de sesión.

## Imágenes (paso a través)

Si tu CLI acepta rutas de imagen, configura `imageArg`:

```yaml
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw escribirá imágenes en base64 en archivos temporales. Si `imageArg` está configurado, esas rutas se pasan como argumentos CLI. Si `imageArg` falta, OpenClaw agrega las rutas de archivo al prompt (inyección de ruta), lo cual es suficiente para CLIs que cargan automáticamente archivos locales desde rutas planas (comportamiento de Claude Code CLI).

## Entradas / Salidas

-   `output: "json"` (predeterminado) intenta analizar JSON y extraer texto + id de sesión.
-   `output: "jsonl"` analiza flujos JSONL (Codex CLI `--json`) y extrae el último mensaje del agente más `thread_id` cuando está presente.
-   `output: "text"` trata stdout como la respuesta final.

Modos de entrada:

-   `input: "arg"` (predeterminado) pasa el prompt como el último argumento CLI.
-   `input: "stdin"` envía el prompt a través de stdin.
-   Si el prompt es muy largo y `maxPromptArgChars` está configurado, se usa stdin.

## Valores predeterminados (integrados)

OpenClaw incluye un valor predeterminado para `claude-cli`:

-   `command: "claude"`
-   `args: ["-p", "--output-format", "json", "--permission-mode", "bypassPermissions"]`
-   `resumeArgs: ["-p", "--output-format", "json", "--permission-mode", "bypassPermissions", "--resume", "{sessionId}"]`
-   `modelArg: "--model"`
-   `systemPromptArg: "--append-system-prompt"`
-   `sessionArg: "--session-id"`
-   `systemPromptWhen: "first"`
-   `sessionMode: "always"`

OpenClaw también incluye un valor predeterminado para `codex-cli`:

-   `command: "codex"`
-   `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
-   `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
-   `output: "jsonl"`
-   `resumeOutput: "text"`
-   `modelArg: "--model"`
-   `imageArg: "--image"`
-   `sessionMode: "existing"`

Sobrescribe solo si es necesario (común: ruta absoluta de `command`).

## Limitaciones

-   **Sin herramientas de OpenClaw** (el backend CLI nunca recibe llamadas a herramientas). Algunos CLIs aún pueden ejecutar sus propias herramientas de agente.
-   **Sin transmisión en tiempo real** (la salida CLI se recopila y luego se devuelve).
-   **Las salidas estructuradas** dependen del formato JSON del CLI.
-   **Las sesiones de Codex CLI** se reanudan a través de salida de texto (sin JSONL), lo cual es menos estructurado que la ejecución inicial `--json`. Las sesiones de OpenClaw aún funcionan normalmente.

## Solución de problemas

-   **CLI no encontrado**: configura `command` a una ruta completa.
-   **Nombre de modelo incorrecto**: usa `modelAliases` para mapear `proveedor/modelo` → modelo CLI.
-   **Sin continuidad de sesión**: asegúrate de que `sessionArg` esté configurado y `sessionMode` no sea `none` (Codex CLI actualmente no puede reanudar con salida JSON).
-   **Imágenes ignoradas**: configura `imageArg` (y verifica que el CLI admita rutas de archivo).

[API de Invocación de Herramientas](./tools-invoke-http-api.md)[Modelos Locales](./local-models.md)