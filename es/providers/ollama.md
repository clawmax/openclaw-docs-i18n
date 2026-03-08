

  Proveedores

  
# Ollama

Ollama es un entorno de ejecución local para LLM que facilita la ejecución de modelos de código abierto en tu máquina. OpenClaw se integra con la API nativa de Ollama (`/api/chat`), soportando streaming y llamada de herramientas, y puede **descubrir automáticamente modelos con capacidad para herramientas** cuando optas por ello con `OLLAMA_API_KEY` (o un perfil de autenticación) y no defines una entrada explícita `models.providers.ollama`.

> **⚠️** **Usuarios de Ollama remoto**: No uses la URL compatible con OpenAI `/v1` (`http://host:11434/v1`) con OpenClaw. Esto rompe la llamada de herramientas y los modelos pueden generar JSON crudo de herramientas como texto plano. Usa la URL de la API nativa de Ollama en su lugar: `baseUrl: "http://host:11434"` (sin `/v1`).

## Inicio rápido

1.  Instala Ollama: [https://ollama.ai](https://ollama.ai)
2.  Descarga un modelo:

```bash
ollama pull gpt-oss:20b
# o
ollama pull llama3.3
# o
ollama pull qwen2.5-coder:32b
# o
ollama pull deepseek-r1:32b
```

3.  Habilita Ollama para OpenClaw (cualquier valor funciona; Ollama no requiere una clave real):

```bash
# Establece la variable de entorno
export OLLAMA_API_KEY="ollama-local"

# O configura en tu archivo de configuración
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4.  Usa modelos de Ollama:

```json
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Descubrimiento de modelos (proveedor implícito)

Cuando estableces `OLLAMA_API_KEY` (o un perfil de autenticación) y **no** defines `models.providers.ollama`, OpenClaw descubre modelos desde la instancia local de Ollama en `http://127.0.0.1:11434`:

-   Consulta `/api/tags` y `/api/show`
-   Mantiene solo los modelos que reportan capacidad `tools`
-   Marca `reasoning` cuando el modelo reporta `thinking`
-   Lee `contextWindow` de `model_info[".context_length"]` cuando está disponible
-   Establece `maxTokens` a 10× la ventana de contexto
-   Establece todos los costos a `0`

Esto evita entradas manuales de modelos mientras mantiene el catálogo alineado con las capacidades de Ollama. Para ver qué modelos están disponibles:

```bash
ollama list
openclaw models list
```

Para agregar un nuevo modelo, simplemente descárgalo con Ollama:

```bash
ollama pull mistral
```

El nuevo modelo será descubierto automáticamente y estará disponible para usar. Si estableces `models.providers.ollama` explícitamente, se omite el descubrimiento automático y debes definir los modelos manualmente (ver abajo).

## Configuración

### Configuración básica (descubrimiento implícito)

La forma más simple de habilitar Ollama es mediante una variable de entorno:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Configuración explícita (modelos manuales)

Usa configuración explícita cuando:

-   Ollama se ejecuta en otro host/puerto.
-   Quieres forzar ventanas de contexto o listas de modelos específicas.
-   Quieres incluir modelos que no reportan soporte para herramientas.

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434",
        apiKey: "ollama-local",
        api: "ollama",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Si `OLLAMA_API_KEY` está establecida, puedes omitir `apiKey` en la entrada del proveedor y OpenClaw la completará para las comprobaciones de disponibilidad.

### URL base personalizada (configuración explícita)

Si Ollama se está ejecutando en un host o puerto diferente (la configuración explícita desactiva el descubrimiento automático, así que define los modelos manualmente):

```json
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434", // Sin /v1 - usa la URL de la API nativa de Ollama
        api: "ollama", // Establece explícitamente para garantizar el comportamiento nativo de llamada de herramientas
      },
    },
  },
}
```

> **⚠️** No agregues `/v1` a la URL. La ruta `/v1` usa el modo compatible con OpenAI, donde la llamada de herramientas no es confiable. Usa la URL base de Ollama sin un sufijo de ruta.

### Selección de modelos

Una vez configurado, todos tus modelos de Ollama están disponibles:

```json
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## Avanzado

### Modelos de razonamiento

OpenClaw marca los modelos como capaces de razonamiento cuando Ollama reporta `thinking` en `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

### Costos de modelos

Ollama es gratuito y se ejecuta localmente, por lo que todos los costos de los modelos se establecen en $0.

### Configuración de Streaming

La integración de Ollama en OpenClaw usa la **API nativa de Ollama** (`/api/chat`) por defecto, que soporta completamente streaming y llamada de herramientas simultáneamente. No se necesita configuración especial.

#### Modo Compatible con OpenAI (Legado)

> **⚠️** **La llamada de herramientas no es confiable en el modo compatible con OpenAI.** Usa este modo solo si necesitas el formato OpenAI para un proxy y no dependes del comportamiento nativo de llamada de herramientas.

 Si necesitas usar el endpoint compatible con OpenAI en su lugar (por ejemplo, detrás de un proxy que solo soporta formato OpenAI), establece `api: "openai-completions"` explícitamente:

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: true, // por defecto: true
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

Este modo puede no soportar streaming + llamada de herramientas simultáneamente. Puede que necesites desactivar el streaming con `params: { streaming: false }` en la configuración del modelo. Cuando se usa `api: "openai-completions"` con Ollama, OpenClaw inyecta `options.num_ctx` por defecto para que Ollama no recurra silenciosamente a una ventana de contexto de 4096. Si tu proxy/upstream rechaza campos `options` desconocidos, desactiva este comportamiento:

```json
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        injectNumCtxForOpenAICompat: false,
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

### Ventanas de contexto

Para los modelos descubiertos automáticamente, OpenClaw usa la ventana de contexto reportada por Ollama cuando está disponible; de lo contrario, usa `8192` por defecto. Puedes anular `contextWindow` y `maxTokens` en la configuración explícita del proveedor.

## Solución de problemas

### Ollama no detectado

Asegúrate de que Ollama esté ejecutándose y de que hayas establecido `OLLAMA_API_KEY` (o un perfil de autenticación), y de que **no** hayas definido una entrada explícita `models.providers.ollama`:

```bash
ollama serve
```

Y que la API sea accesible:

```bash
curl http://localhost:11434/api/tags
```

### No hay modelos disponibles

OpenClaw solo descubre automáticamente modelos que reportan soporte para herramientas. Si tu modelo no aparece en la lista, puedes:

-   Descargar un modelo con capacidad para herramientas, o
-   Definir el modelo explícitamente en `models.providers.ollama`.

Para agregar modelos:

```bash
ollama list  # Ver qué está instalado
ollama pull gpt-oss:20b  # Descargar un modelo con capacidad para herramientas
ollama pull llama3.3     # O otro modelo
```

### Conexión rechazada

Verifica que Ollama se esté ejecutando en el puerto correcto:

```bash
# Verifica si Ollama está ejecutándose
ps aux | grep ollama

# O reinicia Ollama
ollama serve
```

## Ver también

-   [Proveedores de Modelos](../concepts/model-providers.md) - Descripción general de todos los proveedores
-   [Selección de Modelos](../concepts/models.md) - Cómo elegir modelos
-   [Configuración](../gateway/configuration.md) - Referencia completa de configuración

[NVIDIA](./nvidia.md)[OpenAI](./openai.md)