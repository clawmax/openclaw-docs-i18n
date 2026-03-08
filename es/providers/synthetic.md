

  Proveedores

  
# Synthetic

Synthetic expone endpoints compatibles con Anthropic. OpenClaw lo registra como el proveedor `synthetic` y utiliza la API de Mensajes de Anthropic.

## Configuración rápida

1.  Establece `SYNTHETIC_API_KEY` (o ejecuta el asistente a continuación).
2.  Ejecuta la incorporación:

```bash
openclaw onboard --auth-choice synthetic-api-key
```

El modelo predeterminado se establece en:

```
synthetic/hf:MiniMaxAI/MiniMax-M2.5
```

## Ejemplo de configuración

```json
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.5" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.5": { alias: "MiniMax M2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.5",
            name: "MiniMax M2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

Nota: El cliente Anthropic de OpenClaw añade `/v1` a la URL base, así que usa `https://api.synthetic.new/anthropic` (no `/anthropic/v1`). Si Synthetic cambia su URL base, sobrescribe `models.providers.synthetic.baseUrl`.

## Catálogo de modelos

Todos los modelos a continuación tienen un costo de `0` (entrada/salida/caché).

| ID del Modelo | Ventana de contexto | Tokens máximos | Razonamiento | Entrada |
| --- | --- | --- | --- | --- |
| `hf:MiniMaxAI/MiniMax-M2.5` | 192000 | 65536 | false | texto |
| `hf:moonshotai/Kimi-K2-Thinking` | 256000 | 8192 | true | texto |
| `hf:zai-org/GLM-4.7` | 198000 | 128000 | false | texto |
| `hf:deepseek-ai/DeepSeek-R1-0528` | 128000 | 8192 | false | texto |
| `hf:deepseek-ai/DeepSeek-V3-0324` | 128000 | 8192 | false | texto |
| `hf:deepseek-ai/DeepSeek-V3.1` | 128000 | 8192 | false | texto |
| `hf:deepseek-ai/DeepSeek-V3.1-Terminus` | 128000 | 8192 | false | texto |
| `hf:deepseek-ai/DeepSeek-V3.2` | 159000 | 8192 | false | texto |
| `hf:meta-llama/Llama-3.3-70B-Instruct` | 128000 | 8192 | false | texto |
| `hf:meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | 524000 | 8192 | false | texto |
| `hf:moonshotai/Kimi-K2-Instruct-0905` | 256000 | 8192 | false | texto |
| `hf:openai/gpt-oss-120b` | 128000 | 8192 | false | texto |
| `hf:Qwen/Qwen3-235B-A22B-Instruct-2507` | 256000 | 8192 | false | texto |
| `hf:Qwen/Qwen3-Coder-480B-A35B-Instruct` | 256000 | 8192 | false | texto |
| `hf:Qwen/Qwen3-VL-235B-A22B-Instruct` | 250000 | 8192 | false | texto + imagen |
| `hf:zai-org/GLM-4.5` | 128000 | 128000 | false | texto |
| `hf:zai-org/GLM-4.6` | 198000 | 128000 | false | texto |
| `hf:deepseek-ai/DeepSeek-V3` | 128000 | 8192 | false | texto |
| `hf:Qwen/Qwen3-235B-A22B-Thinking-2507` | 256000 | 8192 | true | texto |

## Notas

-   Las referencias de modelo usan `synthetic/`.
-   Si habilitas una lista de permitidos de modelos (`agents.defaults.models`), añade cada modelo que planees usar.
-   Consulta [Proveedores de modelos](../concepts/model-providers.md) para las reglas del proveedor.

[Qwen](./qwen.md)[Together](./together.md)

---