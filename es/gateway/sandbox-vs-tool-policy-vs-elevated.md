

  Seguridad y sandboxing

  
# Sandbox vs Política de Herramientas vs Elevado

OpenClaw tiene tres controles relacionados (pero diferentes):

1.  **Sandbox** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) decide **dónde se ejecutan las herramientas** (Docker vs host).
2.  **Política de herramientas** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) decide **qué herramientas están disponibles/permitidas**.
3.  **Elevado** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) es una **escotilla de escape solo para exec** para ejecutar en el host cuando estás en un sandbox.

## Depuración rápida

Usa el inspector para ver qué está haciendo OpenClaw *realmente*:

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

Imprime:

-   modo/alcance/acceso al workspace efectivo del sandbox
-   si la sesión está actualmente en sandbox (main vs non-main)
-   lista de permitidos/denegados de herramientas del sandbox efectiva (y si provino de agente/global/por defecto)
-   puertas elevadas y rutas de claves de solución

## Sandbox: dónde se ejecutan las herramientas

El sandboxing se controla mediante `agents.defaults.sandbox.mode`:

-   `"off"`: todo se ejecuta en el host.
-   `"non-main"`: solo las sesiones no principales (non-main) están en sandbox (sorpresa común para grupos/canales).
-   `"all"`: todo está en sandbox.

Consulta [Sandboxing](./sandboxing.md) para ver la matriz completa (alcance, montajes del workspace, imágenes).

### Montajes bind (comprobación rápida de seguridad)

-   `docker.binds` *perfora* el sistema de archivos del sandbox: lo que montes será visible dentro del contenedor con el modo que configures (`:ro` o `:rw`).
-   Por defecto es lectura-escritura si omites el modo; prefiere `:ro` para código fuente/secretos.
-   `scope: "shared"` ignora los montajes bind por agente (solo aplican los montajes globales).
-   Montar `/var/run/docker.sock` efectivamente entrega el control del host al sandbox; hazlo solo intencionalmente.
-   El acceso al workspace (`workspaceAccess: "ro"`/`"rw"`) es independiente de los modos de montaje bind.

## Política de herramientas: qué herramientas existen/se pueden llamar

Dos capas importan:

-   **Perfil de herramienta**: `tools.profile` y `agents.list[].tools.profile` (lista de permitidos base)
-   **Perfil de herramienta del proveedor**: `tools.byProvider[provider].profile` y `agents.list[].tools.byProvider[provider].profile`
-   **Política de herramientas global/por agente**: `tools.allow`/`tools.deny` y `agents.list[].tools.allow`/`agents.list[].tools.deny`
-   **Política de herramientas del proveedor**: `tools.byProvider[provider].allow/deny` y `agents.list[].tools.byProvider[provider].allow/deny`
-   **Política de herramientas del sandbox** (solo aplica cuando estás en sandbox): `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` y `agents.list[].tools.sandbox.tools.*`

Reglas generales:

-   `deny` siempre gana.
-   Si `allow` no está vacío, todo lo demás se trata como bloqueado.
-   La política de herramientas es la parada definitiva: `/exec` no puede anular una herramienta `exec` denegada.
-   `/exec` solo cambia los valores por defecto de la sesión para remitentes autorizados; no otorga acceso a herramientas. Las claves de herramientas del proveedor aceptan `provider` (ej. `google-antigravity`) o `provider/model` (ej. `openai/gpt-5.2`).

### Grupos de herramientas (abreviaturas)

Las políticas de herramientas (global, agente, sandbox) admiten entradas `group:*` que se expanden a múltiples herramientas:

```json
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"],
      },
    },
  },
}
```

Grupos disponibles:

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: todas las herramientas integradas de OpenClaw (excluye plugins de proveedores)

## Elevado: "ejecutar en el host" solo para exec

Elevado **no** otorga herramientas adicionales; solo afecta a `exec`.

-   Si estás en sandbox, `/elevated on` (o `exec` con `elevated: true`) se ejecuta en el host (pueden seguir aplicándose aprobaciones).
-   Usa `/elevated full` para omitir las aprobaciones de exec para la sesión.
-   Si ya estás ejecutando directamente, elevado es efectivamente una no-operación (aún con puertas).
-   Elevado **no** está limitado por habilidad y **no** anula la lista de permitidos/denegados de herramientas.
-   `/exec` es independiente de elevado. Solo ajusta los valores por defecto de exec por sesión para remitentes autorizados.

Puertas:

-   Habilitación: `tools.elevated.enabled` (y opcionalmente `agents.list[].tools.elevated.enabled`)
-   Listas de remitentes permitidos: `tools.elevated.allowFrom.` (y opcionalmente `agents.list[].tools.elevated.allowFrom.`)

Consulta [Modo Elevado](../tools/elevated.md).

## Soluciones comunes para la "cárcel del sandbox"

### "La herramienta X está bloqueada por la política de herramientas del sandbox"

Claves de solución (elige una):

-   Deshabilitar sandbox: `agents.defaults.sandbox.mode=off` (o por agente `agents.list[].sandbox.mode=off`)
-   Permitir la herramienta dentro del sandbox:
    -   quitarla de `tools.sandbox.tools.deny` (o por agente `agents.list[].tools.sandbox.tools.deny`)
    -   o agregarla a `tools.sandbox.tools.allow` (o la lista de permitidos por agente)

### "Pensé que esta era la sesión principal (main), ¿por qué está en sandbox?"

En modo `"non-main"`, las claves de grupo/canal *no* son main. Usa la clave de sesión principal (mostrada por `sandbox explain`) o cambia el modo a `"off"`.

[Sandboxing](./sandboxing.md)[Protocolo de Gateway](./protocol.md)