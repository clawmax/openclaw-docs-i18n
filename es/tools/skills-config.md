

  Habilidades

  
# Configuración de Habilidades

Toda la configuración relacionada con habilidades se encuentra bajo `skills` en `~/.openclaw/openclaw.json`.

```json
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway runtime still Node; bun not recommended)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // or plaintext string
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## Campos

-   `allowBundled`: lista de permitidos opcional solo para habilidades **incluidas** (bundled). Cuando se establece, solo las habilidades incluidas en la lista son elegibles (las habilidades gestionadas/de espacio de trabajo no se ven afectadas).
-   `load.extraDirs`: directorios de habilidades adicionales para escanear (precedencia más baja).
-   `load.watch`: vigila las carpetas de habilidades y actualiza la instantánea de habilidades (por defecto: true).
-   `load.watchDebounceMs`: tiempo de debounce para eventos del vigilante de habilidades en milisegundos (por defecto: 250).
-   `install.preferBrew`: preferir instaladores de brew cuando estén disponibles (por defecto: true).
-   `install.nodeManager`: preferencia de gestor de node para instalaciones (`npm` | `pnpm` | `yarn` | `bun`, por defecto: npm). Esto solo afecta a las **instalaciones de habilidades**; el runtime de Gateway debe seguir siendo Node (no se recomienda Bun para WhatsApp/Telegram).
-   `entries.`: sobreescrituras por habilidad.

Campos por habilidad:

-   `enabled`: establece `false` para deshabilitar una habilidad incluso si está incluida/instalada.
-   `env`: variables de entorno inyectadas para la ejecución del agente (solo si no están ya establecidas).
-   `apiKey`: conveniencia opcional para habilidades que declaran una variable de entorno principal. Admite cadena de texto plano u objeto SecretRef (`{ source, provider, id }`).

## Notas

-   Las claves bajo `entries` se asignan al nombre de la habilidad por defecto. Si una habilidad define `metadata.openclaw.skillKey`, usa esa clave en su lugar.
-   Los cambios en las habilidades se detectan en el siguiente turno del agente cuando el vigilante está habilitado.

### Habilidades en sandbox + variables de entorno

Cuando una sesión está en **sandbox**, los procesos de las habilidades se ejecutan dentro de Docker. El sandbox **no** hereda el `process.env` del host. Usa una de las siguientes opciones:

-   `agents.defaults.sandbox.docker.env` (o por agente `agents.list[].sandbox.docker.env`)
-   integra las variables de entorno en tu imagen de sandbox personalizada

Las configuraciones globales `env` y `skills.entries..env/apiKey` se aplican solo a ejecuciones en el **host**.

[Habilidades](./skills.md)[ClawHub](./clawhub.md)

---