

  Habilidades

  
# Habilidades

OpenClaw utiliza carpetas de habilidades compatibles con **[AgentSkills](https://agentskills.io)** para enseñar al agente cómo usar herramientas. Cada habilidad es un directorio que contiene un `SKILL.md` con frontmatter YAML e instrucciones. OpenClaw carga **habilidades empaquetadas** más anulaciones locales opcionales, y las filtra en tiempo de carga según el entorno, la configuración y la presencia de binarios.

## Ubicaciones y precedencia

Las habilidades se cargan desde **tres** lugares:

1.  **Habilidades empaquetadas**: incluidas en la instalación (paquete npm o OpenClaw.app)
2.  **Habilidades gestionadas/locales**: `~/.openclaw/skills`
3.  **Habilidades del espacio de trabajo**: `/skills`

Si un nombre de habilidad entra en conflicto, la precedencia es: `/skills` (más alta) → `~/.openclaw/skills` → habilidades empaquetadas (más baja) Adicionalmente, puedes configurar carpetas de habilidades extra (precedencia más baja) mediante `skills.load.extraDirs` en `~/.openclaw/openclaw.json`.

## Habilidades por agente vs compartidas

En configuraciones **multi-agente**, cada agente tiene su propio espacio de trabajo. Eso significa:

-   **Habilidades por agente** residen en `/skills` solo para ese agente.
-   **Habilidades compartidas** residen en `~/.openclaw/skills` (gestionadas/locales) y son visibles para **todos los agentes** en la misma máquina.
-   **Carpetas compartidas** también se pueden agregar mediante `skills.load.extraDirs` (precedencia más baja) si deseas un paquete de habilidades común usado por múltiples agentes.

Si el mismo nombre de habilidad existe en más de un lugar, aplica la precedencia habitual: el espacio de trabajo gana, luego gestionadas/locales, luego empaquetadas.

## Plugins + habilidades

Los plugins pueden incluir sus propias habilidades listando directorios `skills` en `openclaw.plugin.json` (rutas relativas a la raíz del plugin). Las habilidades del plugin se cargan cuando el plugin está habilitado y participan en las reglas normales de precedencia de habilidades. Puedes controlar su acceso mediante `metadata.openclaw.requires.config` en la entrada de configuración del plugin. Consulta [Plugins](./plugin.md) para descubrimiento/configuración y [Herramientas](../tools.md) para la superficie de herramientas que esas habilidades enseñan.

## ClawHub (instalar + sincronizar)

ClawHub es el registro público de habilidades para OpenClaw. Navega en [https://clawhub.com](https://clawhub.com). Úsalo para descubrir, instalar, actualizar y respaldar habilidades. Guía completa: [ClawHub](./clawhub.md). Flujos comunes:

-   Instalar una habilidad en tu espacio de trabajo:
    -   `clawhub install <skill-slug>`
-   Actualizar todas las habilidades instaladas:
    -   `clawhub update --all`
-   Sincronizar (escanear + publicar actualizaciones):
    -   `clawhub sync --all`

Por defecto, `clawhub` instala en `./skills` bajo tu directorio de trabajo actual (o recurre al espacio de trabajo de OpenClaw configurado). OpenClaw lo recoge como `/skills` en la siguiente sesión.

## Notas de seguridad

-   Trata las habilidades de terceros como **código no confiable**. Léelas antes de habilitarlas.
-   Prefiere ejecuciones en sandbox para entradas no confiables y herramientas riesgosas. Consulta [Sandboxing](../gateway/sandboxing.md).
-   El descubrimiento de habilidades del espacio de trabajo y directorios extra solo acepta raíces de habilidades y archivos `SKILL.md` cuya ruta real resuelta permanezca dentro de la raíz configurada.
-   `skills.entries.*.env` y `skills.entries.*.apiKey` inyectan secretos en el proceso **host** para ese turno del agente (no en el sandbox). Mantén los secretos fuera de los prompts y logs.
-   Para un modelo de amenazas más amplio y listas de verificación, consulta [Seguridad](../gateway/security.md).

## Formato (compatible con AgentSkills + Pi)

`SKILL.md` debe incluir al menos:

```
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
---
```

Notas:

-   Seguimos la especificación de AgentSkills para diseño/intención.
-   El analizador utilizado por el agente integrado solo admite claves de frontmatter en **una sola línea**.
-   `metadata` debe ser un **objeto JSON de una sola línea**.
-   Usa `{baseDir}` en las instrucciones para referenciar la ruta de la carpeta de la habilidad.
-   Claves de frontmatter opcionales:
    -   `homepage` — URL mostrada como "Sitio web" en la interfaz de usuario de Habilidades de macOS (también admitida mediante `metadata.openclaw.homepage`).
    -   `user-invocable` — `true|false` (predeterminado: `true`). Cuando es `true`, la habilidad se expone como un comando de barra diagonal del usuario.
    -   `disable-model-invocation` — `true|false` (predeterminado: `false`). Cuando es `true`, la habilidad se excluye del prompt del modelo (todavía disponible mediante invocación del usuario).
    -   `command-dispatch` — `tool` (opcional). Cuando se establece en `tool`, el comando de barra diagonal omite el modelo y se envía directamente a una herramienta.
    -   `command-tool` — nombre de la herramienta a invocar cuando `command-dispatch: tool` está establecido.
    -   `command-arg-mode` — `raw` (predeterminado). Para el envío a herramienta, reenvía la cadena de argumentos sin procesar a la herramienta (sin análisis del núcleo). La herramienta se invoca con los parámetros: `{ command: "", commandName: "", skillName: "" }`.

## Control de acceso (filtros en tiempo de carga)

OpenClaw **filtra habilidades en tiempo de carga** usando `metadata` (JSON de una sola línea):

```
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["uv"], "env": ["GEMINI_API_KEY"], "config": ["browser.enabled"] },
        "primaryEnv": "GEMINI_API_KEY",
      },
  }
---
```

Campos bajo `metadata.openclaw`:

-   `always: true` — incluir siempre la habilidad (omitir otros controles).
-   `emoji` — emoji opcional usado por la interfaz de usuario de Habilidades de macOS.
-   `homepage` — URL opcional mostrada como "Sitio web" en la interfaz de usuario de Habilidades de macOS.
-   `os` — lista opcional de plataformas (`darwin`, `linux`, `win32`). Si se establece, la habilidad solo es elegible en esos sistemas operativos.
-   `requires.bins` — lista; cada uno debe existir en `PATH`.
-   `requires.anyBins` — lista; al menos uno debe existir en `PATH`.
-   `requires.env` — lista; la variable de entorno debe existir **o** ser proporcionada en la configuración.
-   `requires.config` — lista de rutas de `openclaw.json` que deben ser verdaderas.
-   `primaryEnv` — nombre de la variable de entorno asociada con `skills.entries..apiKey`.
-   `install` — array opcional de especificaciones de instalador usadas por la interfaz de usuario de Habilidades de macOS (brew/node/go/uv/download).

Nota sobre sandboxing:

-   `requires.bins` se verifica en el **host** en el momento de carga de la habilidad.
-   Si un agente está en sandbox, el binario también debe existir **dentro del contenedor**. Instálalo mediante `agents.defaults.sandbox.docker.setupCommand` (o una imagen personalizada). `setupCommand` se ejecuta una vez después de crear el contenedor. Las instalaciones de paquetes también requieren salida de red, un sistema de archivos raíz escribible y un usuario root en el sandbox. Ejemplo: la habilidad `summarize` (`skills/summarize/SKILL.md`) necesita el CLI `summarize` en el contenedor sandbox para ejecutarse allí.

Ejemplo de instalador:

```
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata:
  {
    "openclaw":
      {
        "emoji": "♊️",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "Install Gemini CLI (brew)",
            },
          ],
      },
  }
---
```

Notas:

-   Si se listan múltiples instaladores, la puerta de enlace elige una **única** opción preferida (brew cuando está disponible, de lo contrario node).
-   Si todos los instaladores son `download`, OpenClaw lista cada entrada para que puedas ver los artefactos disponibles.
-   Las especificaciones del instalador pueden incluir `os: ["darwin"|"linux"|"win32"]` para filtrar opciones por plataforma.
-   Las instalaciones de Node respetan `skills.install.nodeManager` en `openclaw.json` (predeterminado: npm; opciones: npm/pnpm/yarn/bun). Esto solo afecta a las **instalaciones de habilidades**; el tiempo de ejecución de la Puerta de enlace aún debe ser Node (no se recomienda Bun para WhatsApp/Telegram).
-   Instalaciones de Go: si falta `go` y `brew` está disponible, la puerta de enlace instala Go primero mediante Homebrew y establece `GOBIN` en el `bin` de Homebrew cuando es posible.
-   Instalaciones de descarga: `url` (requerida), `archive` (`tar.gz` | `tar.bz2` | `zip`), `extract` (predeterminado: automático cuando se detecta archivo), `stripComponents`, `targetDir` (predeterminado: `~/.openclaw/tools/`).

Si no hay `metadata.openclaw` presente, la habilidad siempre es elegible (a menos que esté deshabilitada en la configuración o bloqueada por `skills.allowBundled` para habilidades empaquetadas).

## Anulaciones de configuración (~/.openclaw/openclaw.json)

Las habilidades empaquetadas/gestionadas se pueden activar/desactivar y proporcionar con valores de entorno:

```json
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // o cadena de texto plano
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

Nota: si el nombre de la habilidad contiene guiones, pon la clave entre comillas (JSON5 permite claves entre comillas). Las claves de configuración coinciden con el **nombre de la habilidad** por defecto. Si una habilidad define `metadata.openclaw.skillKey`, usa esa clave bajo `skills.entries`. Reglas:

-   `enabled: false` deshabilita la habilidad incluso si está empaquetada/instalada.
-   `env`: se inyecta **solo si** la variable no está ya establecida en el proceso.
-   `apiKey`: conveniencia para habilidades que declaran `metadata.openclaw.primaryEnv`. Admite cadena de texto plano u objeto SecretRef (`{ source, provider, id }`).
-   `config`: bolsa opcional para campos personalizados por habilidad; las claves personalizadas deben residir aquí.
-   `allowBundled`: lista blanca opcional solo para habilidades **empaquetadas**. Si se establece, solo las habilidades empaquetadas en la lista son elegibles (las habilidades gestionadas/del espacio de trabajo no se ven afectadas).

## Inyección de entorno (por ejecución de agente)

Cuando comienza una ejecución de agente, OpenClaw:

1.  Lee los metadatos de la habilidad.
2.  Aplica cualquier `skills.entries..env` o `skills.entries..apiKey` a `process.env`.
3.  Construye el prompt del sistema con las habilidades **elegibles**.
4.  Restaura el entorno original después de que finaliza la ejecución.

Esto está **limitado a la ejecución del agente**, no es un entorno de shell global.

## Instantánea de sesión (rendimiento)

OpenClaw toma una instantánea de las habilidades elegibles **cuando comienza una sesión** y reutiliza esa lista para turnos posteriores en la misma sesión. Los cambios en las habilidades o la configuración surten efecto en la siguiente sesión nueva. Las habilidades también pueden actualizarse a mitad de sesión cuando el observador de habilidades está habilitado o cuando aparece un nuevo nodo remoto elegible (ver más abajo). Piensa en esto como una **recarga en caliente**: la lista actualizada se recoge en el siguiente turno del agente.

## Nodos macOS remotos (puerta de enlace Linux)

Si la Puerta de enlace se ejecuta en Linux pero un **nodo macOS** está conectado **con `system.run` permitido** (seguridad de aprobaciones de ejecución no establecida en `deny`), OpenClaw puede tratar las habilidades solo para macOS como elegibles cuando los binarios requeridos están presentes en ese nodo. El agente debe ejecutar esas habilidades mediante la herramienta `nodes` (típicamente `nodes.run`). Esto depende de que el nodo informe su soporte de comandos y de un sondeo de binario mediante `system.run`. Si el nodo macOS se desconecta más tarde, las habilidades permanecen visibles; las invocaciones pueden fallar hasta que el nodo se reconecte.

## Observador de habilidades (actualización automática)

Por defecto, OpenClaw observa las carpetas de habilidades y actualiza la instantánea de habilidades cuando cambian los archivos `SKILL.md`. Configúralo bajo `skills.load`:

```json
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250,
    },
  },
}
```

## Impacto en tokens (lista de habilidades)

Cuando las habilidades son elegibles, OpenClaw inyecta una lista compacta XML de habilidades disponibles en el prompt del sistema (mediante `formatSkillsForPrompt` en `pi-coding-agent`). El costo es determinista:

-   **Sobrecarga base (solo cuando ≥1 habilidad):** 195 caracteres.
-   **Por habilidad:** 97 caracteres + la longitud de los valores ``, `` y `` escapados en XML.

Fórmula (caracteres):

```bash
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

Notas:

-   El escape XML expande `& < > " '` en entidades (`&amp;`, `<`, etc.), aumentando la longitud.
-   Los recuentos de tokens varían según el tokenizador del modelo. Una estimación aproximada al estilo OpenAI es ~4 caracteres/token, por lo que **97 caracteres ≈ 24 tokens** por habilidad más las longitudes reales de tus campos.

## Ciclo de vida de habilidades gestionadas

OpenClaw incluye un conjunto base de habilidades como **habilidades empaquetadas** como parte de la instalación (paquete npm o OpenClaw.app). `~/.openclaw/skills` existe para anulaciones locales (por ejemplo, fijar/parchear una habilidad sin cambiar la copia empaquetada). Las habilidades del espacio de trabajo son propiedad del usuario y anulan a ambas en conflictos de nombre.

## Referencia de configuración

Consulta [Configuración de habilidades](./skills-config.md) para el esquema de configuración completo.

## ¿Buscas más habilidades?

Navega en [https://clawhub.com](https://clawhub.com).

* * *

[Comandos de Barra Diagonal](./slash-commands.md)[Configuración de Habilidades](./skills-config.md)