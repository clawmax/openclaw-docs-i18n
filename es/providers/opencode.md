

  Proveedores

  
# OpenCode Zen

OpenCode Zen es una **lista curada de modelos** recomendada por el equipo de OpenCode para agentes de codificación. Es una ruta de acceso opcional y alojada a modelos que utiliza una clave API y el proveedor `opencode`. Zen se encuentra actualmente en versión beta.

## Configuración CLI

```bash
openclaw onboard --auth-choice opencode-zen
# o no interactivo
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## Fragmento de configuración

```json
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## Notas

-   `OPENCODE_ZEN_API_KEY` también es compatible.
-   Inicias sesión en Zen, agregas detalles de facturación y copias tu clave API.
-   OpenCode Zen factura por solicitud; consulta el panel de control de OpenCode para más detalles.

[OpenAI](./openai.md)[OpenRouter](./openrouter.md)

---