

  Proveedores

  
# Together

[Together AI](https://together.ai) proporciona acceso a modelos de código abierto líderes, incluyendo Llama, DeepSeek, Kimi y más, a través de una API unificada.

-   Proveedor: `together`
-   Autenticación: `TOGETHER_API_KEY`
-   API: Compatible con OpenAI

## Inicio rápido

1.  Establece la clave API (recomendado: guárdala para el Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2.  Establece un modelo predeterminado:

```json
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Ejemplo no interactivo

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Esto establecerá `together/moonshotai/Kimi-K2.5` como el modelo predeterminado.

## Nota sobre el entorno

Si el Gateway se ejecuta como un daemon (launchd/systemd), asegúrate de que `TOGETHER_API_KEY` esté disponible para ese proceso (por ejemplo, en `~/.openclaw/.env` o a través de `env.shellEnv`).

## Modelos disponibles

Together AI proporciona acceso a muchos modelos de código abierto populares:

-   **GLM 4.7 Fp8** - Modelo predeterminado con ventana de contexto de 200K
-   **Llama 3.3 70B Instruct Turbo** - Seguimiento de instrucciones rápido y eficiente
-   **Llama 4 Scout** - Modelo de visión con comprensión de imágenes
-   **Llama 4 Maverick** - Visión y razonamiento avanzados
-   **DeepSeek V3.1** - Modelo potente para programación y razonamiento
-   **DeepSeek R1** - Modelo de razonamiento avanzado
-   **Kimi K2 Instruct** - Modelo de alto rendimiento con ventana de contexto de 262K

Todos los modelos admiten finalizaciones de chat estándar y son compatibles con la API de OpenAI.

[Sintético](./synthetic.md)[Vercel AI Gateway](./vercel-ai-gateway.md)

---