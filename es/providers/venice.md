

  Proveedores

  
# Venice AI

**Venice** es nuestra configuración destacada para inferencia centrada en la privacidad con acceso opcional anonimizado a modelos propietarios. Venice AI proporciona inferencia de IA centrada en la privacidad con soporte para modelos sin censura y acceso a los principales modelos propietarios a través de su proxy anonimizado. Toda la inferencia es privada por defecto: sin entrenamiento con tus datos, sin registro.

## Por qué Venice en OpenClaw

-   **Inferencia privada** para modelos de código abierto (sin registro).
-   **Modelos sin censura** cuando los necesitas.
-   **Acceso anonimizado** a modelos propietarios (Opus/GPT/Gemini) cuando la calidad importa.
-   Endpoints compatibles con OpenAI `/v1`.

## Modos de Privacidad

Venice ofrece dos niveles de privacidad — entender esto es clave para elegir tu modelo:

| Modo | Descripción | Modelos |
| --- | --- | --- |
| **Privado** | Totalmente privado. Los prompts/respuestas **nunca se almacenan ni registran**. Efímero. | Llama, Qwen, DeepSeek, Kimi, MiniMax, Venice Uncensored, etc. |
| **Anonimizado** | Proxificado a través de Venice con metadatos eliminados. El proveedor subyacente (OpenAI, Anthropic, Google, xAI) ve solicitudes anonimizadas. | Claude, GPT, Gemini, Grok |

## Características

-   **Centrado en la privacidad**: Elige entre modos "privado" (totalmente privado) y "anonimizado" (proxificado)
-   **Modelos sin censura**: Acceso a modelos sin restricciones de contenido
-   **Acceso a modelos principales**: Usa Claude, GPT, Gemini y Grok a través del proxy anonimizado de Venice
-   **API compatible con OpenAI**: Endpoints estándar `/v1` para una fácil integración
-   **Streaming**: ✅ Soportado en todos los modelos
-   **Llamada a funciones**: ✅ Soportado en modelos seleccionados (verifica las capacidades del modelo)
-   **Visión**: ✅ Soportado en modelos con capacidad de visión
-   **Sin límites de tasa estrictos**: Puede aplicarse limitación por uso justo para uso extremo

## Configuración

### 1\. Obtén la Clave API

1.  Regístrate en [venice.ai](https://venice.ai)
2.  Ve a **Configuración → Claves API → Crear nueva clave**
3.  Copia tu clave API (formato: `vapi_xxxxxxxxxxxx`)

### 2\. Configura OpenClaw

**Opción A: Variable de Entorno**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**Opción B: Configuración Interactiva (Recomendada)**

```bash
openclaw onboard --auth-choice venice-api-key
```

Esto hará:

1.  Solicitará tu clave API (o usará la `VENICE_API_KEY` existente)
2.  Mostrará todos los modelos Venice disponibles
3.  Te permitirá elegir tu modelo por defecto
4.  Configurará el proveedor automáticamente

**Opción C: No Interactiva**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```

### 3\. Verifica la Configuración

```bash
openclaw agent --model venice/kimi-k2-5 --message "Hello, are you working?"
```

## Selección de Modelo

Después de la configuración, OpenClaw muestra todos los modelos Venice disponibles. Elige según tus necesidades:

-   **Modelo por defecto**: `venice/kimi-k2-5` para razonamiento privado fuerte más visión.
-   **Opción de alta capacidad**: `venice/claude-opus-4-6` para la ruta Venice anonimizada más potente.
-   **Privacidad**: Elige modelos "privados" para inferencia totalmente privada.
-   **Capacidad**: Elige modelos "anonimizados" para acceder a Claude, GPT, Gemini a través del proxy de Venice.

Cambia tu modelo por defecto en cualquier momento:

```bash
openclaw models set venice/kimi-k2-5
openclaw models set venice/claude-opus-4-6
```

Lista todos los modelos disponibles:

```bash
openclaw models list | grep venice
```

## Configurar vía openclaw configure

1.  Ejecuta `openclaw configure`
2.  Selecciona **Modelo/autenticación**
3.  Elige **Venice AI**

## ¿Qué Modelo Debo Usar?

| Caso de Uso | Modelo Recomendado | Por qué |
| --- | --- | --- |
| **Chat general (por defecto)** | `kimi-k2-5` | Razonamiento privado fuerte más visión |
| **Mejor calidad general** | `claude-opus-4-6` | Opción Venice anonimizada más potente |
| **Privacidad + programación** | `qwen3-coder-480b-a35b-instruct` | Modelo de programación privado con contexto grande |
| **Visión privada** | `kimi-k2-5` | Soporte de visión sin salir del modo privado |
| **Rápido + económico** | `qwen3-4b` | Modelo de razonamiento ligero |
| **Tareas privadas complejas** | `deepseek-v3.2` | Razonamiento fuerte, pero sin soporte de herramientas Venice |
| **Sin censura** | `venice-uncensored` | Sin restricciones de contenido |

## Modelos Disponibles (41 en Total)

### Modelos Privados (26) — Totalmente Privados, Sin Registro

| ID del Modelo | Nombre | Contexto | Características |
| --- | --- | --- | --- |
| `kimi-k2-5` | Kimi K2.5 | 256k | Por defecto, razonamiento, visión |
| `kimi-k2-thinking` | Kimi K2 Thinking | 256k | Razonamiento |
| `llama-3.3-70b` | Llama 3.3 70B | 128k | General |
| `llama-3.2-3b` | Llama 3.2 3B | 128k | General |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 128k | General, herramientas deshabilitadas |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 128k | Razonamiento |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 128k | General |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 256k | Programación |
| `qwen3-coder-480b-a35b-instruct-turbo` | Qwen3 Coder 480B Turbo | 256k | Programación |
| `qwen3-5-35b-a3b` | Qwen3.5 35B A3B | 256k | Razonamiento, visión |
| `qwen3-next-80b` | Qwen3 Next 80B | 256k | General |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B (Visión) | 256k | Visión |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | Rápido, razonamiento |
| `deepseek-v3.2` | DeepSeek V3.2 | 160k | Razonamiento, herramientas deshabilitadas |
| `venice-uncensored` | Venice Uncensored (Dolphin-Mistral) | 32k | Sin censura, herramientas deshabilitadas |
| `mistral-31-24b` | Venice Medium (Mistral) | 128k | Visión |
| `google-gemma-3-27b-it` | Google Gemma 3 27B Instruct | 198k | Visión |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 128k | General |
| `nvidia-nemotron-3-nano-30b-a3b` | NVIDIA Nemotron 3 Nano 30B | 128k | General |
| `olafangensan-glm-4.7-flash-heretic` | GLM 4.7 Flash Heretic | 128k | Razonamiento |
| `zai-org-glm-4.6` | GLM 4.6 | 198k | General |
| `zai-org-glm-4.7` | GLM 4.7 | 198k | Razonamiento |
| `zai-org-glm-4.7-flash` | GLM 4.7 Flash | 128k | Razonamiento |
| `zai-org-glm-5` | GLM 5 | 198k | Razonamiento |
| `minimax-m21` | MiniMax M2.1 | 198k | Razonamiento |
| `minimax-m25` | MiniMax M2.5 | 198k | Razonamiento |

### Modelos Anonimizados (15) — A través del Proxy de Venice

| ID del Modelo | Nombre | Contexto | Características |
| --- | --- | --- | --- |
| `claude-opus-4-6` | Claude Opus 4.6 (vía Venice) | 1M | Razonamiento, visión |
| `claude-opus-4-5` | Claude Opus 4.5 (vía Venice) | 198k | Razonamiento, visión |
| `claude-sonnet-4-6` | Claude Sonnet 4.6 (vía Venice) | 1M | Razonamiento, visión |
| `claude-sonnet-4-5` | Claude Sonnet 4.5 (vía Venice) | 198k | Razonamiento, visión |
| `openai-gpt-54` | GPT-5.4 (vía Venice) | 1M | Razonamiento, visión |
| `openai-gpt-53-codex` | GPT-5.3 Codex (vía Venice) | 400k | Razonamiento, visión, programación |
| `openai-gpt-52` | GPT-5.2 (vía Venice) | 256k | Razonamiento |
| `openai-gpt-52-codex` | GPT-5.2 Codex (vía Venice) | 256k | Razonamiento, visión, programación |
| `openai-gpt-4o-2024-11-20` | GPT-4o (vía Venice) | 128k | Visión |
| `openai-gpt-4o-mini-2024-07-18` | GPT-4o Mini (vía Venice) | 128k | Visión |
| `gemini-3-1-pro-preview` | Gemini 3.1 Pro (vía Venice) | 1M | Razonamiento, visión |
| `gemini-3-pro-preview` | Gemini 3 Pro (vía Venice) | 198k | Razonamiento, visión |
| `gemini-3-flash-preview` | Gemini 3 Flash (vía Venice) | 256k | Razonamiento, visión |
| `grok-41-fast` | Grok 4.1 Fast (vía Venice) | 1M | Razonamiento, visión |
| `grok-code-fast-1` | Grok Code Fast 1 (vía Venice) | 256k | Razonamiento, programación |

## Descubrimiento de Modelos

OpenClaw descubre automáticamente modelos desde la API de Venice cuando `VENICE_API_KEY` está configurada. Si la API no es accesible, recurre a un catálogo estático. El endpoint `/models` es público (no se necesita autenticación para listar), pero la inferencia requiere una clave API válida.

## Streaming y Soporte de Herramientas

| Característica | Soporte |
| --- | --- |
| **Streaming** | ✅ Todos los modelos |
| **Llamada a funciones** | ✅ La mayoría de modelos (verifica `supportsFunctionCalling` en la API) |
| **Visión/Imágenes** | ✅ Modelos marcados con la característica "Visión" |
| **Modo JSON** | ✅ Soportado vía `response_format` |

## Precios

Venice usa un sistema basado en créditos. Consulta [venice.ai/pricing](https://venice.ai/pricing) para las tarifas actuales:

-   **Modelos privados**: Generalmente costo más bajo
-   **Modelos anonimizados**: Similar a los precios de API directos + pequeña tarifa de Venice

## Comparación: Venice vs API Directa

| Aspecto | Venice (Anonimizado) | API Directa |
| --- | --- | --- |
| **Privacidad** | Metadatos eliminados, anonimizado | Tu cuenta vinculada |
| **Latencia** | +10-50ms (proxy) | Directa |
| **Características** | La mayoría de características soportadas | Todas las características |
| **Facturación** | Créditos Venice | Facturación del proveedor |

## Ejemplos de Uso

```bash
# Usa el modelo privado por defecto
openclaw agent --model venice/kimi-k2-5 --message "Quick health check"

# Usa Claude Opus vía Venice (anonimizado)
openclaw agent --model venice/claude-opus-4-6 --message "Summarize this task"

# Usa modelo sin censura
openclaw agent --model venice/venice-uncensored --message "Draft options"

# Usa modelo de visión con imagen
openclaw agent --model venice/qwen3-vl-235b-a22b --message "Review attached image"

# Usa modelo de programación
openclaw agent --model venice/qwen3-coder-480b-a35b-instruct --message "Refactor this function"
```

## Resolución de Problemas

### Clave API no reconocida

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

Asegúrate de que la clave comience con `vapi_`.

### Modelo no disponible

El catálogo de modelos Venice se actualiza dinámicamente. Ejecuta `openclaw models list` para ver los modelos actualmente disponibles. Algunos modelos pueden estar temporalmente fuera de línea.

### Problemas de conexión

La API de Venice está en `https://api.venice.ai/api/v1`. Asegúrate de que tu red permita conexiones HTTPS.

## Ejemplo de archivo de configuración

```json
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/kimi-k2-5" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2-5",
            name: "Kimi K2.5",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

## Enlaces

-   [Venice AI](https://venice.ai)
-   [Documentación de la API](https://docs.venice.ai)
-   [Precios](https://venice.ai/pricing)
-   [Estado](https://status.venice.ai)

[Vercel AI Gateway](./vercel-ai-gateway.md)[vLLM](./vllm.md)