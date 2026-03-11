

  Proveedores

  
# Kilocode

Kilo Gateway proporciona una **API unificada** que enruta las solicitudes a muchos modelos detrás de un único endpoint y clave API. Es compatible con OpenAI, por lo que la mayoría de los SDKs de OpenAI funcionan cambiando la URL base.

## Obtener una clave API

1.  Ve a [app.kilo.ai](https://app.kilo.ai)
2.  Inicia sesión o crea una cuenta
3.  Navega a Claves API y genera una nueva clave

## Configuración CLI

```bash
openclaw onboard --kilocode-api-key <key>
```

O establece la variable de entorno:

```bash
export KILOCODE_API_KEY="<your-kilocode-api-key>" # pragma: allowlist secret
```

## Fragmento de configuración

```json
{
  env: { KILOCODE_API_KEY: "<your-kilocode-api-key>" }, // pragma: allowlist secret
  agents: {
    defaults: {
      model: { primary: "kilocode/kilo/auto" },
    },
  },
}
```

## Modelo por defecto

El modelo por defecto es `kilocode/kilo/auto`, un modelo de enrutamiento inteligente que selecciona automáticamente el mejor modelo subyacente según la tarea:

-   Las tareas de planificación, depuración y orquestación se enrutan a Claude Opus
-   Las tareas de escritura y exploración de código se enrutan a Claude Sonnet

## Modelos disponibles

OpenClaw descubre dinámicamente los modelos disponibles desde Kilo Gateway al iniciar. Usa `/models kilocode` para ver la lista completa de modelos disponibles con tu cuenta. Cualquier modelo disponible en la puerta de enlace se puede usar con el prefijo `kilocode/`:

```
kilocode/kilo/auto              (por defecto - enrutamiento inteligente)
kilocode/anthropic/claude-sonnet-4
kilocode/openai/gpt-5.2
kilocode/google/gemini-3-pro-preview
...y muchos más
```

## Notas

-   Las referencias de modelo son `kilocode/<model-id>` (ej., `kilocode/anthropic/claude-sonnet-4`).
-   Modelo por defecto: `kilocode/kilo/auto`
-   URL base: `https://api.kilo.ai/api/gateway/`
-   Para más opciones de modelos/proveedores, consulta [/concepts/model-providers](../concepts/model-providers.md).
-   Kilo Gateway utiliza un token Bearer con tu clave API internamente.

[Hugging Face (Inferencia)](./huggingface.md)[Litellm](./litellm.md)