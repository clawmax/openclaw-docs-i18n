

  Proveedores

  
# Mistral

OpenClaw soporta Mistral tanto para el enrutamiento de modelos de texto/imagen (`mistral/...`) como para la transcripción de audio a través de Voxtral en la comprensión de medios. Mistral también se puede usar para incrustaciones de memoria (`memorySearch.provider = "mistral"`).

## Configuración por CLI

```bash
openclaw onboard --auth-choice mistral-api-key
# o no interactivo
openclaw onboard --mistral-api-key "$MISTRAL_API_KEY"
```

## Fragmento de configuración (proveedor LLM)

```json
{
  env: { MISTRAL_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "mistral/mistral-large-latest" } } },
}
```

## Fragmento de configuración (transcripción de audio con Voxtral)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## Notas

-   La autenticación de Mistral usa `MISTRAL_API_KEY`.
-   La URL base del proveedor por defecto es `https://api.mistral.ai/v1`.
-   El modelo por defecto en la incorporación es `mistral/mistral-large-latest`.
-   El modelo de audio por defecto para Mistral en comprensión de medios es `voxtral-mini-latest`.
-   La ruta de transcripción de medios usa `/v1/audio/transcriptions`.
-   La ruta de incrustaciones de memoria usa `/v1/embeddings` (modelo por defecto: `mistral-embed`).

[Moonshot AI](./moonshot.md)[NVIDIA](./nvidia.md)