

  Proveedores

  
# Modelos GLM

GLM es una **familia de modelos** (no una empresa) disponible a través de la plataforma Z.AI. En OpenClaw, los modelos GLM se acceden a través del proveedor `zai` e IDs de modelo como `zai/glm-5`.

## Configuración CLI

```bash
openclaw onboard --auth-choice zai-api-key
```

## Fragmento de configuración

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Notas

-   Las versiones y disponibilidad de GLM pueden cambiar; consulta la documentación de Z.AI para lo más reciente.
-   Ejemplos de IDs de modelo incluyen `glm-5`, `glm-4.7` y `glm-4.6`.
-   Para detalles del proveedor, consulta [/providers/zai](./zai.md).

[Litellm](./litellm.md)[MiniMax](./minimax.md)

---