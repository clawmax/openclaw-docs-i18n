

  Protocolos y APIs

  
# Modelos Locales

Es posible usar modelos locales, pero OpenClaw requiere contexto grande y defensas fuertes contra inyección de prompts. Las tarjetas pequeñas truncan el contexto y filtran seguridad. Apunta alto: **≥2 Mac Studios al máximo o una configuración de GPU equivalente (~$30k+)**. Una sola GPU de **24 GB** solo funciona para prompts más ligeros con mayor latencia. Usa **la variante de modelo más grande / de tamaño completo que puedas ejecutar**; los checkpoints fuertemente cuantizados o "pequeños" aumentan el riesgo de inyección de prompts (ver [Seguridad](./security.md)).

## Recomendado: LM Studio + MiniMax M2.5 (API de Respuestas, tamaño completo)

La mejor pila local actual. Carga MiniMax M2.5 en LM Studio, habilita el servidor local (por defecto `http://127.0.0.1:1234`), y usa la API de Respuestas para mantener el razonamiento separado del texto final.

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**Lista de verificación de configuración**

-   Instala LM Studio: [https://lmstudio.ai](https://lmstudio.ai)
-   En LM Studio, descarga **la compilación más grande de MiniMax M2.5 disponible** (evita variantes "pequeñas"/fuertemente cuantizadas), inicia el servidor, confirma que `http://127.0.0.1:1234/v1/models` la liste.
-   Mantén el modelo cargado; la carga en frío añade latencia de inicio.
-   Ajusta `contextWindow`/`maxTokens` si tu compilación de LM Studio difiere.
-   Para WhatsApp, usa la API de Respuestas para que solo se envíe el texto final.

Mantén los modelos alojados configurados incluso cuando ejecutes localmente; usa `models.mode: "merge"` para que los respaldos sigan disponibles.

### Configuración híbrida: primario alojado, respaldo local

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.5-gs32", "anthropic/claude-opus-4-6"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.5-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### Primero local con red de seguridad alojada

Intercambia el orden del primario y el respaldo; mantén el mismo bloque de proveedores y `models.mode: "merge"` para poder recurrir a Sonnet u Opus cuando el equipo local esté caído.

### Alojamiento regional / enrutamiento de datos

-   Las variantes alojadas de MiniMax/Kimi/GLM también existen en OpenRouter con endpoints fijados por región (ej., alojados en EE. UU.). Elige la variante regional allí para mantener el tráfico en tu jurisdicción elegida mientras aún usas `models.mode: "merge"` para respaldos de Anthropic/OpenAI.
-   Solo local sigue siendo la ruta de privacidad más fuerte; el enrutamiento regional alojado es el punto medio cuando necesitas funciones del proveedor pero quieres control sobre el flujo de datos.

## Otros proxies locales compatibles con OpenAI

vLLM, LiteLLM, OAI-proxy o gateways personalizados funcionan si exponen un endpoint `/v1` estilo OpenAI. Reemplaza el bloque de proveedor anterior con tu endpoint e ID de modelo:

```json
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Mantén `models.mode: "merge"` para que los modelos alojados sigan disponibles como respaldos.

## Solución de problemas

-   ¿El Gateway puede alcanzar el proxy? `curl http://127.0.0.1:1234/v1/models`.
-   ¿Modelo de LM Studio descargado? Recárgalo; el inicio en frío es una causa común de "colgado".
-   ¿Errores de contexto? Reduce `contextWindow` o aumenta el límite de tu servidor.
-   Seguridad: los modelos locales omiten los filtros del lado del proveedor; mantén los agentes estrechos y la compactación activada para limitar el radio de explosión de la inyección de prompts.

[Backends CLI](./cli-backends.md)[Modelo de red](./network-model.md)