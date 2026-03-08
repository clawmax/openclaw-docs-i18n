

  Comandos CLI

  
# agents

Gestiona agentes aislados (espacio de trabajo + autenticación + enrutamiento). Relacionado:

-   Enrutamiento multi-agente: [Enrutamiento Multi-Agente](../concepts/multi-agent.md)
-   Espacio de trabajo del agente: [Espacio de trabajo del agente](../concepts/agent-workspace.md)

## Ejemplos

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents bindings
openclaw agents bind --agent work --bind telegram:ops
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Enlaces de enrutamiento

Usa enlaces de enrutamiento para dirigir el tráfico entrante de un canal a un agente específico. Lista los enlaces:

```bash
openclaw agents bindings
openclaw agents bindings --agent work
openclaw agents bindings --json
```

Añade enlaces:

```bash
openclaw agents bind --agent work --bind telegram:ops --bind discord:guild-a
```

Si omites `accountId` (`--bind `), OpenClaw lo resuelve a partir de los valores predeterminados del canal y los hooks de configuración del plugin cuando están disponibles.

### Comportamiento del alcance del enlace

-   Un enlace sin `accountId` coincide solo con la cuenta predeterminada del canal.
-   `accountId: "*"` es el fallback para todo el canal (todas las cuentas) y es menos específico que un enlace con una cuenta explícita.
-   Si el mismo agente ya tiene un enlace de canal coincidente sin `accountId`, y luego enlazas con un `accountId` explícito o resuelto, OpenClaw actualiza ese enlace existente en lugar de añadir un duplicado.

Ejemplo:

```bash
# enlace inicial solo para el canal
openclaw agents bind --agent work --bind telegram

# actualización posterior a un enlace con alcance de cuenta
openclaw agents bind --agent work --bind telegram:ops
```

Después de la actualización, el enrutamiento para ese enlace tiene alcance a `telegram:ops`. Si también quieres enrutamiento para la cuenta predeterminada, añádelo explícitamente (por ejemplo `--bind telegram:default`). Elimina enlaces:

```bash
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents unbind --agent work --all
```

## Archivos de identidad

Cada espacio de trabajo de agente puede incluir un `IDENTITY.md` en la raíz del espacio de trabajo:

-   Ruta de ejemplo: `~/.openclaw/workspace/IDENTITY.md`
-   `set-identity --from-identity` lee desde la raíz del espacio de trabajo (o desde un `--identity-file` explícito)

Las rutas de los avatares se resuelven relativas a la raíz del espacio de trabajo.

## Establecer identidad

`set-identity` escribe campos en `agents.list[].identity`:

-   `name`
-   `theme`
-   `emoji`
-   `avatar` (ruta relativa al espacio de trabajo, URL http(s) o URI de datos)

Cargar desde `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Sobrescribir campos explícitamente:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Ejemplo de configuración:

```json
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```

[agent](./agent.md)[approvals](./approvals.md)