

  Primeros pasos

  
# Resumen de Incorporación

OpenClaw admite múltiples rutas de incorporación dependiendo de dónde se ejecute el Gateway y cómo prefieras configurar los proveedores.

## Elige tu ruta de incorporación

-   **Asistente CLI** para macOS, Linux y Windows (a través de WSL2).
-   **Aplicación macOS** para una primera ejecución guiada en Macs con Apple silicon o Intel.

## Asistente de incorporación CLI

Ejecuta el asistente en una terminal:

```bash
openclaw onboard
```

Usa el asistente CLI cuando quieras control total del Gateway, el espacio de trabajo, los canales y las habilidades. Documentación:

-   [Asistente de Incorporación (CLI)](./wizard.md)
-   [Comando `openclaw onboard`](../cli/onboard.md)

## Incorporación con la aplicación macOS

Usa la aplicación OpenClaw cuando quieras una configuración completamente guiada en macOS. Documentación:

-   [Incorporación (Aplicación macOS)](./onboarding.md)

## Proveedor Personalizado

Si necesitas un endpoint que no esté listado, incluidos proveedores alojados que exponen APIs estándar de OpenAI o Anthropic, elige **Proveedor Personalizado** en el asistente CLI. Se te pedirá:

-   Elegir compatible con OpenAI, compatible con Anthropic o **Desconocido** (detección automática).
-   Ingresar una URL base y una clave API (si el proveedor lo requiere).
-   Proporcionar un ID de modelo y un alias opcional.
-   Elegir un ID de Endpoint para que puedan coexistir múltiples endpoints personalizados.

Para pasos detallados, sigue la documentación de incorporación CLI anterior.

[Primeros Pasos](./getting-started.md)[Incorporación: CLI](./wizard.md)