

  Extensiones

  
# Herramientas de Agente de Plugin

Los plugins de OpenClaw pueden registrar **herramientas de agente** (funciones JSON‑schema) que se exponen al LLM durante las ejecuciones del agente. Las herramientas pueden ser **requeridas** (siempre disponibles) u **opcionales** (opt‑in). Las herramientas de agente se configuran bajo `tools` en la configuración principal, o por agente bajo `agents.list[].tools`. La política de lista de permitidos/denegados controla qué herramientas puede llamar el agente.

## Herramienta básica

```typescript
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Hacer algo",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```

## Herramienta opcional (opt‑in)

Las herramientas opcionales **nunca** se habilitan automáticamente. Los usuarios deben agregarlas a una lista de permitidos de un agente.

```bash
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Ejecutar un flujo de trabajo local",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

Habilita herramientas opcionales en `agents.list[].tools.allow` (o globalmente en `tools.allow`):

```json
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool", // nombre específico de la herramienta
            "workflow", // id del plugin (habilita todas las herramientas de ese plugin)
            "group:plugins", // todas las herramientas de plugins
          ],
        },
      },
    ],
  },
}
```

Otras opciones de configuración que afectan la disponibilidad de herramientas:

-   Las listas de permitidos que solo nombran herramientas de plugin se tratan como opt-ins de plugin; las herramientas principales permanecen habilitadas a menos que también incluyas herramientas principales o grupos en la lista de permitidos.
-   `tools.profile` / `agents.list[].tools.profile` (lista de permitidos base)
-   `tools.byProvider` / `agents.list[].tools.byProvider` (permitir/denegar específico del proveedor)
-   `tools.sandbox.tools.*` (política de herramientas del sandbox cuando está en sandbox)

## Reglas + consejos

-   Los nombres de las herramientas **no** deben coincidir con los nombres de las herramientas principales; las herramientas en conflicto se omiten.
-   Los ids de plugin usados en listas de permitidos no deben coincidir con nombres de herramientas principales.
-   Prefiere `optional: true` para herramientas que desencadenan efectos secundarios o requieren binarios/credenciales adicionales.

[Manifiesto de Plugin](./manifest.md)[OpenProse](../prose.md)

---