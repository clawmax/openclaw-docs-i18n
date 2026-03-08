title: "Guía de Configuración y Configuración del Proveedor OpenClaw vLLM"
description: "Aprende cómo conectar OpenClaw a vLLM para servir modelos de código abierto a través de una API compatible con OpenAI. Incluye inicio rápido, descubrimiento automático y configuración explícita."
keywords: ["vllm", "api compatible con openai", "servicio de modelos", "proveedores de openclaw", "llm local", "descubrimiento de modelos", "completaciones de openai", "configuración de vllm"]
---

  Proveedores

  
# vLLM

vLLM puede servir modelos de código abierto (y algunos personalizados) a través de una API HTTP **compatible con OpenAI**. OpenClaw puede conectarse a vLLM usando la API `openai-completions`. OpenClaw también puede **descubrir automáticamente** los modelos disponibles de vLLM cuando optas por activarlo con `VLLM_API_KEY` (cualquier valor funciona si tu servidor no aplica autenticación) y no defines una entrada explícita `models.providers.vllm`.

## Inicio rápido

1.  Inicia vLLM con un servidor compatible con OpenAI.

Tu URL base debe exponer los endpoints `/v1` (por ejemplo, `/v1/models`, `/v1/chat/completions`). vLLM comúnmente se ejecuta en:

-   `http://127.0.0.1:8000/v1`

2.  Opta por activarlo (cualquier valor funciona si no hay autenticación configurada):

```bash
export VLLM_API_KEY="vllm-local"
```

3.  Selecciona un modelo (reemplaza con uno de tus IDs de modelo de vLLM):

```json
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Descubrimiento de modelos (proveedor implícito)

Cuando `VLLM_API_KEY` está configurada (o existe un perfil de autenticación) y **no** defines `models.providers.vllm`, OpenClaw consultará:

-   `GET http://127.0.0.1:8000/v1/models`

…y convertirá los IDs devueltos en entradas de modelo. Si configuras `models.providers.vllm` explícitamente, se omite el descubrimiento automático y debes definir los modelos manualmente.

## Configuración explícita (modelos manuales)

Usa la configuración explícita cuando:

-   vLLM se ejecuta en un host/puerto diferente.
-   Quieres fijar valores de `contextWindow`/`maxTokens`.
-   Tu servidor requiere una clave API real (o quieres controlar los encabezados).

```json
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "Modelo vLLM Local",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Solución de problemas

-   Verifica que el servidor sea accesible:

```bash
curl http://127.0.0.1:8000/v1/models
```

-   Si las solicitudes fallan con errores de autenticación, configura una `VLLM_API_KEY` real que coincida con la configuración de tu servidor, o configura el proveedor explícitamente bajo `models.providers.vllm`.

[Venice AI](./venice.md)[Xiaomi MiMo](./xiaomi.md)

---