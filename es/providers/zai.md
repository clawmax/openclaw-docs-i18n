

  Proveedores

  
# Z.AI

Z.AI es la plataforma API para modelos **GLM**. Proporciona APIs REST para GLM y utiliza claves API para autenticación. Crea tu clave API en la consola de Z.AI. OpenClaw utiliza el proveedor `zai` con una clave API de Z.AI.

## Configuración por CLI

```bash
openclaw onboard --auth-choice zai-api-key
# o no interactivo
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## Fragmento de configuración

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Notas

-   Los modelos GLM están disponibles como `zai/` (ejemplo: `zai/glm-5`).
-   `tool_stream` está habilitado por defecto para el streaming de llamadas a herramientas de Z.AI. Establece `agents.defaults.models["zai/"].params.tool_stream` en `false` para desactivarlo.
-   Consulta [/providers/glm](./glm.md) para ver la descripción general de la familia de modelos.
-   Z.AI utiliza autenticación Bearer con tu clave API.

[Xiaomi MiMo](./xiaomi.md)

---