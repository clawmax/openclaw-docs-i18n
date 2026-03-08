

  Extensiones

  
# OpenProse

OpenProse es un formato de flujo de trabajo portátil y basado en markdown para orquestar sesiones de IA. En OpenClaw se incluye como un plugin que instala un paquete de habilidades de OpenProse más un comando de barra `/prose`. Los programas residen en archivos `.prose` y pueden generar múltiples subagentes con un flujo de control explícito. Sitio oficial: [https://www.prose.md](https://www.prose.md)

## Qué puede hacer

-   Investigación y síntesis multiagente con paralelismo explícito.
-   Flujos de trabajo repetibles y seguros para aprobación (revisión de código, triaje de incidentes, canalizaciones de contenido).
-   Programas `.prose` reutilizables que puedes ejecutar en entornos de ejecución de agentes compatibles.

## Instalar + habilitar

Los plugins incluidos están deshabilitados por defecto. Habilita OpenProse:

```bash
openclaw plugins enable open-prose
```

Reinicia el Gateway después de habilitar el plugin. Para instalación local/desarrollo: `openclaw plugins install ./extensions/open-prose` Documentación relacionada: [Plugins](./tools/plugin.md), [Manifiesto del plugin](./plugins/manifest.md), [Habilidades](./tools/skills.md).

## Comando de barra

OpenProse registra `/prose` como un comando de habilidad invocable por el usuario. Enruta a las instrucciones de la máquina virtual de OpenProse y utiliza herramientas de OpenClaw internamente. Comandos comunes:

```bash
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## Ejemplo: un archivo .prose simple

```bash
# Investigación + síntesis con dos agentes ejecutándose en paralelo.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## Ubicaciones de archivos

OpenProse mantiene el estado en `.prose/` dentro de tu espacio de trabajo:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

Los agentes persistentes a nivel de usuario residen en:

```
~/.prose/agents/
```

## Modos de estado

OpenProse admite múltiples backends de estado:

-   **sistema de archivos** (predeterminado): `.prose/runs/...`
-   **en contexto**: transitorio, para programas pequeños
-   **sqlite** (experimental): requiere el binario `sqlite3`
-   **postgres** (experimental): requiere `psql` y una cadena de conexión

Notas:

-   sqlite/postgres son opcionales y experimentales.
-   Las credenciales de postgres fluyen hacia los registros de subagentes; usa una base de datos dedicada y con privilegios mínimos.

## Programas remotos

`/prose run <handle/slug>` se resuelve a `https://p.prose.md//`. Las URLs directas se obtienen tal cual. Esto utiliza la herramienta `web_fetch` (o `exec` para POST).

## Mapeo del entorno de ejecución de OpenClaw

Los programas de OpenProse se mapean a las primitivas de OpenClaw:

| Concepto de OpenProse | Herramienta de OpenClaw |
| --- | --- |
| Generar sesión / Herramienta de tarea | `sessions_spawn` |
| Lectura/escritura de archivos | `read` / `write` |
| Obtención web | `web_fetch` |

Si tu lista de herramientas permitidas bloquea estas herramientas, los programas de OpenProse fallarán. Consulta [Configuración de habilidades](./tools/skills-config.md).

## Seguridad + aprobaciones

Trata los archivos `.prose` como código. Revísalos antes de ejecutarlos. Usa las listas de herramientas permitidas de OpenClaw y las puertas de aprobación para controlar los efectos secundarios. Para flujos de trabajo deterministas y con aprobación, compara con [Lobster](./tools/lobster.md).

[Herramientas de Agente de Plugin](./plugins/agent-tools.md)[Hooks](./automation/hooks.md)