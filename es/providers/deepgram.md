

  Proveedores

  
# Deepgram

Deepgram es una API de voz a texto. En OpenClaw se utiliza para la **transcripción de audio/notas de voz entrantes** a través de `tools.media.audio`. Cuando está habilitado, OpenClaw sube el archivo de audio a Deepgram e inyecta la transcripción en el pipeline de respuesta (`{{Transcript}}` + bloque `[Audio]`). Esto **no es en streaming**; utiliza el endpoint de transcripción pregrabada. Sitio web: [https://deepgram.com](https://deepgram.com)  
Documentación: [https://developers.deepgram.com](https://developers.deepgram.com)

## Inicio rápido

1.  Establece tu clave API:

```
DEEPGRAM_API_KEY=dg_...
```

2.  Habilita el proveedor:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Opciones

-   `model`: ID del modelo de Deepgram (predeterminado: `nova-3`)
-   `language`: sugerencia de idioma (opcional)
-   `tools.media.audio.providerOptions.deepgram.detect_language`: habilita la detección de idioma (opcional)
-   `tools.media.audio.providerOptions.deepgram.punctuate`: habilita la puntuación (opcional)
-   `tools.media.audio.providerOptions.deepgram.smart_format`: habilita el formato inteligente (opcional)

Ejemplo con idioma:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3", language: "en" }],
      },
    },
  },
}
```

Ejemplo con opciones de Deepgram:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true,
          },
        },
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Notas

-   La autenticación sigue el orden estándar de autenticación de proveedores; `DEEPGRAM_API_KEY` es la ruta más simple.
-   Anula los endpoints o encabezados con `tools.media.audio.baseUrl` y `tools.media.audio.headers` cuando uses un proxy.
-   La salida sigue las mismas reglas de audio que otros proveedores (límites de tamaño, tiempos de espera, inyección de transcripción).

[Proxy de API Claude Max](./claude-max-api-proxy.md)[GitHub Copilot](./github-copilot.md)