

  Proveedores

  
# MiniMax

MiniMax es una empresa de IA que construye la familia de modelos **M2/M2.5**. La versión actual centrada en codificación es **MiniMax M2.5** (23 de diciembre de 2025), construida para tareas complejas del mundo real. Fuente: [Nota de lanzamiento de MiniMax M2.5](https://www.minimax.io/news/minimax-m25)

## Resumen del modelo (M2.5)

MiniMax destaca estas mejoras en M2.5:

-   **Codificación multilingüe** más sólida (Rust, Java, Go, C++, Kotlin, Objective-C, TS/JS).
-   Mejor **desarrollo web/aplicaciones** y calidad de salida estética (incluyendo móvil nativo).
-   Manejo mejorado de **instrucciones compuestas** para flujos de trabajo de tipo oficina, basándose en pensamiento intercalado y ejecución integrada de restricciones.
-   **Respuestas más concisas** con menor uso de tokens y ciclos de iteración más rápidos.
-   Mayor compatibilidad con **marcos de herramientas/agentes** y gestión de contexto (Claude Code, Droid/Factory AI, Cline, Kilo Code, Roo Code, BlackBox).
-   Salidas de **diálogo y escritura técnica** de mayor calidad.

## MiniMax M2.5 vs MiniMax M2.5 Highspeed

-   **Velocidad:** `MiniMax-M2.5-highspeed` es el nivel rápido oficial en la documentación de MiniMax.
-   **Costo:** La tarificación de MiniMax lista el mismo costo de entrada y un costo de salida mayor para highspeed.
-   **Compatibilidad:** OpenClaw aún acepta configuraciones heredadas `MiniMax-M2.5-Lightning`, pero prefiere `MiniMax-M2.5-highspeed` para configuraciones nuevas.

## Elige una configuración

### OAuth de MiniMax (Plan de Codificación) — recomendado

**Ideal para:** configuración rápida con el Plan de Codificación de MiniMax vía OAuth, sin necesidad de clave API. Habilita el plugin OAuth incluido y autentícate:

```bash
openclaw plugins enable minimax-portal-auth  # omitir si ya está cargado.
openclaw gateway restart  # reiniciar si el gateway ya está en ejecución
openclaw onboard --auth-choice minimax-portal
```

Se te pedirá que selecciones un endpoint:

-   **Global** - Usuarios internacionales (`api.minimax.io`)
-   **CN** - Usuarios en China (`api.minimaxi.com`)

Consulta el [README del plugin OAuth de MiniMax](https://github.com/openclaw/openclaw/tree/main/extensions/minimax-portal-auth) para más detalles.

### MiniMax M2.5 (Clave API)

**Ideal para:** MiniMax alojado con API compatible con Anthropic. Configura mediante CLI:

-   Ejecuta `openclaw configure`
-   Selecciona **Model/auth**
-   Elige **MiniMax M2.5**

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.5" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.5",
            name: "MiniMax M2.5",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
          {
            id: "MiniMax-M2.5-highspeed",
            name: "MiniMax M2.5 Highspeed",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### MiniMax M2.5 como respaldo (ejemplo)

**Ideal para:** mantener tu modelo de última generación más potente como primario, con conmutación por error a MiniMax M2.5. El ejemplo a continuación usa Opus como primario concreto; cámbialo por tu modelo primario de última generación preferido.

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "primary" },
        "minimax/MiniMax-M2.5": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.5"],
      },
    },
  },
}
```

### Opcional: Local vía LM Studio (manual)

**Ideal para:** inferencia local con LM Studio. Hemos visto resultados sólidos con MiniMax M2.5 en hardware potente (por ejemplo, un escritorio/servidor) usando el servidor local de LM Studio. Configura manualmente mediante `openclaw.json`:

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
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

## Configurar vía openclaw configure

Usa el asistente de configuración interactivo para configurar MiniMax sin editar JSON:

1.  Ejecuta `openclaw configure`.
2.  Selecciona **Model/auth**.
3.  Elige **MiniMax M2.5**.
4.  Elige tu modelo predeterminado cuando se te solicite.

## Opciones de configuración

-   `models.providers.minimax.baseUrl`: prefiere `https://api.minimax.io/anthropic` (compatible con Anthropic); `https://api.minimax.io/v1` es opcional para cargas útiles compatibles con OpenAI.
-   `models.providers.minimax.api`: prefiere `anthropic-messages`; `openai-completions` es opcional para cargas útiles compatibles con OpenAI.
-   `models.providers.minimax.apiKey`: Clave API de MiniMax (`MINIMAX_API_KEY`).
-   `models.providers.minimax.models`: define `id`, `name`, `reasoning`, `contextWindow`, `maxTokens`, `cost`.
-   `agents.defaults.models`: asigna alias a los modelos que quieras en la lista de permitidos.
-   `models.mode`: mantén `merge` si quieres agregar MiniMax junto con los modelos incorporados.

## Notas

-   Las referencias de modelo son `minimax/`.
-   IDs de modelo recomendados: `MiniMax-M2.5` y `MiniMax-M2.5-highspeed`.
-   API de uso del Plan de Codificación: `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (requiere una clave del plan de codificación).
-   Actualiza los valores de tarificación en `models.json` si necesitas un seguimiento exacto de costos.
-   Enlace de referencia para el Plan de Codificación de MiniMax (10% de descuento): [https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link](https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link)
-   Consulta [/concepts/model-providers](../concepts/model-providers.md) para las reglas de proveedores.
-   Usa `openclaw models list` y `openclaw models set minimax/MiniMax-M2.5` para cambiar.

## Solución de problemas

### “Modelo desconocido: minimax/MiniMax-M2.5”

Esto generalmente significa que **el proveedor MiniMax no está configurado** (no hay entrada de proveedor y no se encontró perfil de autenticación/clave de entorno de MiniMax). Una corrección para esta detección está en **2026.1.12** (no lanzada al momento de escribir). Soluciónalo:

-   Actualizando a **2026.1.12** (o ejecutando desde el código fuente `main`), luego reiniciando el gateway.
-   Ejecutando `openclaw configure` y seleccionando **MiniMax M2.5**, o
-   Agregando el bloque `models.providers.minimax` manualmente, o
-   Estableciendo `MINIMAX_API_KEY` (o un perfil de autenticación de MiniMax) para que el proveedor pueda ser inyectado.

Asegúrate de que el ID del modelo sea **sensible a mayúsculas y minúsculas**:

-   `minimax/MiniMax-M2.5`
-   `minimax/MiniMax-M2.5-highspeed`
-   `minimax/MiniMax-M2.5-Lightning` (heredado)

Luego verifica de nuevo con:

```bash
openclaw models list
```

[Modelos GLM](./glm.md)[Moonshot AI](./moonshot.md)