

  Proveedores

  
# Litellm

[LiteLLM](https://litellm.ai) es una puerta de enlace LLM de código abierto que proporciona una API unificada para más de 100 proveedores de modelos. Enruta OpenClaw a través de LiteLLM para obtener un seguimiento centralizado de costos, registro y la flexibilidad de cambiar backends sin modificar tu configuración de OpenClaw.

## ¿Por qué usar LiteLLM con OpenClaw?

-   **Seguimiento de costos** — Ve exactamente en qué gasta OpenClaw en todos los modelos
-   **Enrutamiento de modelos** — Cambia entre Claude, GPT-4, Gemini, Bedrock sin cambios de configuración
-   **Claves virtuales** — Crea claves con límites de gasto para OpenClaw
-   **Registro** — Registros completos de solicitudes/respuestas para depuración
-   **Respaldo** — Conmutación por error automática si tu proveedor principal está caído

## Inicio rápido

### Mediante incorporación

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Configuración manual

1.  Inicia el proxy de LiteLLM:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2.  Dirige OpenClaw a LiteLLM:

```bash
export LITELLM_API_KEY="tu-clave-litellm"

openclaw
```

Eso es todo. OpenClaw ahora se enruta a través de LiteLLM.

## Configuración

### Variables de entorno

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Archivo de configuración

```json
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## Claves virtuales

Crea una clave dedicada para OpenClaw con límites de gasto:

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

Usa la clave generada como `LITELLM_API_KEY`.

## Enrutamiento de modelos

LiteLLM puede enrutar solicitudes de modelos a diferentes backends. Configúralo en tu `config.yaml` de LiteLLM:

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw sigue solicitando `claude-opus-4-6` — LiteLLM maneja el enrutamiento.

## Visualización del uso

Consulta el panel de control o la API de LiteLLM:

```bash
# Información de la clave
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Registros de gasto
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Notas

-   LiteLLM se ejecuta en `http://localhost:4000` por defecto
-   OpenClaw se conecta a través del endpoint compatible con OpenAI `/v1/chat/completions`
-   Todas las funciones de OpenClaw funcionan a través de LiteLLM — sin limitaciones

## Ver también

-   [Documentación de LiteLLM](https://docs.litellm.ai)
-   [Proveedores de modelos](../concepts/model-providers.md)

[Kilocode](./kilocode.md)[Modelos GLM](./glm.md)