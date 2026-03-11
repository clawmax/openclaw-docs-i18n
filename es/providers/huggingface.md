

  Proveedores

  
# Hugging Face (Inferencia)

Los [Proveedores de Inferencia de Hugging Face](https://huggingface.co/docs/inference-providers) ofrecen completados de chat compatibles con OpenAI a través de una única API de router. Obtienes acceso a muchos modelos (DeepSeek, Llama y más) con un solo token. OpenClaw utiliza el **endpoint compatible con OpenAI** (solo completados de chat); para texto a imagen, embeddings o voz, usa los [clientes de inferencia de HF](https://huggingface.co/docs/api-inference/quicktour) directamente.

-   Proveedor: `huggingface`
-   Autenticación: `HUGGINGFACE_HUB_TOKEN` o `HF_TOKEN` (token de grano fino con el permiso **Make calls to Inference Providers**)
-   API: Compatible con OpenAI (`https://router.huggingface.co/v1`)
-   Facturación: Token único de HF; el [precio](https://huggingface.co/docs/inference-providers/pricing) sigue las tarifas del proveedor con un nivel gratuito.

## Inicio rápido

1.  Crea un token de grano fino en [Hugging Face → Configuración → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) con el permiso **Make calls to Inference Providers**.
2.  Ejecuta la configuración inicial y elige **Hugging Face** en el menú desplegable de proveedores, luego ingresa tu clave API cuando se solicite:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3.  En el menú desplegable **Modelo predeterminado de Hugging Face**, elige el modelo que deseas (la lista se carga desde la API de Inferencia cuando tienes un token válido; de lo contrario, se muestra una lista integrada). Tu elección se guarda como el modelo predeterminado.
4.  También puedes establecer o cambiar el modelo predeterminado más tarde en la configuración:

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Ejemplo no interactivo

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Esto establecerá `huggingface/deepseek-ai/DeepSeek-R1` como el modelo predeterminado.

## Nota sobre el entorno

Si el Gateway se ejecuta como un daemon (launchd/systemd), asegúrate de que `HUGGINGFACE_HUB_TOKEN` o `HF_TOKEN` esté disponible para ese proceso (por ejemplo, en `~/.openclaw/.env` o a través de `env.shellEnv`).

## Descubrimiento de modelos y menú desplegable de configuración inicial

OpenClaw descubre modelos llamando directamente al **endpoint de Inferencia**:

```bash
GET https://router.huggingface.co/v1/models
```

(Opcional: envía `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` o `$HF_TOKEN` para la lista completa; algunos endpoints devuelven un subconjunto sin autenticación). La respuesta es estilo OpenAI `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`. Cuando configuras una clave API de Hugging Face (a través de la configuración inicial, `HUGGINGFACE_HUB_TOKEN` o `HF_TOKEN`), OpenClaw usa este GET para descubrir los modelos de completado de chat disponibles. Durante la **configuración inicial interactiva**, después de ingresar tu token, ves un menú desplegable **Modelo predeterminado de Hugging Face** poblado desde esa lista (o el catálogo integrado si la solicitud falla). En tiempo de ejecución (por ejemplo, al iniciar el Gateway), cuando hay una clave presente, OpenClaw vuelve a llamar a **GET** `https://router.huggingface.co/v1/models` para actualizar el catálogo. La lista se fusiona con un catálogo integrado (para metadatos como ventana de contexto y costo). Si la solicitud falla o no se establece ninguna clave, solo se usa el catálogo integrado.

## Nombres de modelos y opciones editables

-   **Nombre desde la API:** El nombre para mostrar del modelo se **hidrata desde GET /v1/models** cuando la API devuelve `name`, `title` o `display_name`; de lo contrario, se deriva del id del modelo (por ejemplo, `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
-   **Anular nombre para mostrar:** Puedes establecer una etiqueta personalizada por modelo en la configuración para que aparezca como desees en la CLI y la UI:

```json
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (rápido)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (económico)" },
      },
    },
  },
}
```

-   **Selección de proveedor / política:** Añade un sufijo al **id del modelo** para elegir cómo el router selecciona el backend:
    
    -   **`:fastest`** — mayor rendimiento (el router elige; la elección del proveedor está **bloqueada** — no hay selector interactivo de backend).
    -   **`:cheapest`** — menor costo por token de salida (el router elige; la elección del proveedor está **bloqueada**).
    -   **`:provider`** — forzar un backend específico (por ejemplo, `:sambanova`, `:together`).
    
    Cuando seleccionas **:cheapest** o **:fastest** (por ejemplo, en el menú desplegable de modelos de la configuración inicial), el proveedor se bloquea: el router decide por costo o velocidad y no se muestra el paso opcional “preferir backend específico”. Puedes agregar estos como entradas separadas en `models.providers.huggingface.models` o establecer `model.primary` con el sufijo. También puedes establecer tu orden predeterminado en la [configuración de Proveedores de Inferencia](https://hf.co/settings/inference-providers) (sin sufijo = usar ese orden).
-   **Fusión de configuración:** Las entradas existentes en `models.providers.huggingface.models` (por ejemplo, en `models.json`) se mantienen cuando se fusiona la configuración. Por lo tanto, cualquier `name`, `alias` personalizado u opciones de modelo que hayas establecido allí se conservan.

## IDs de modelo y ejemplos de configuración

Las referencias de modelo usan la forma `huggingface//` (IDs estilo Hub). La lista a continuación es de **GET** `https://router.huggingface.co/v1/models`; tu catálogo puede incluir más. **Ejemplos de IDs (desde el endpoint de inferencia):**

| Modelo | Ref (prefijar con `huggingface/`) |
| --- | --- |
| DeepSeek R1 | `deepseek-ai/DeepSeek-R1` |
| DeepSeek V3.2 | `deepseek-ai/DeepSeek-V3.2` |
| Qwen3 8B | `Qwen/Qwen3-8B` |
| Qwen2.5 7B Instruct | `Qwen/Qwen2.5-7B-Instruct` |
| Qwen3 32B | `Qwen/Qwen3-32B` |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct` |
| Llama 3.1 8B Instruct | `meta-llama/Llama-3.1-8B-Instruct` |
| GPT-OSS 120B | `openai/gpt-oss-120b` |
| GLM 4.7 | `zai-org/GLM-4.7` |
| Kimi K2.5 | `moonshotai/Kimi-K2.5` |

Puedes añadir `:fastest`, `:cheapest` o `:provider` (por ejemplo, `:together`, `:sambanova`) al id del modelo. Establece tu orden predeterminado en la [configuración de Proveedores de Inferencia](https://hf.co/settings/inference-providers); consulta [Proveedores de Inferencia](https://huggingface.co/docs/inference-providers) y **GET** `https://router.huggingface.co/v1/models` para la lista completa.

### Ejemplos de configuración completos

**DeepSeek R1 principal con Qwen como respaldo:**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen como predeterminado, con variantes :cheapest y :fastest:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (más económico)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (más rápido)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS con alias:**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**Forzar un backend específico con :provider:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**Múltiples modelos Qwen y DeepSeek con sufijos de política:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (económico)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (rápido)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```

[GitHub Copilot](./github-copilot.md)[Kilocode](./kilocode.md)