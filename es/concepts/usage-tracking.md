

  Conceptos internos

  
# Seguimiento de Uso

## Qué es

-   Obtiene el uso/cuota del proveedor directamente desde sus endpoints de uso.
-   Sin costos estimados; solo los períodos reportados por el proveedor.

## Dónde aparece

-   `/status` en chats: tarjeta de estado con emojis que muestra tokens de sesión + costo estimado (solo clave API). El uso del proveedor se muestra para el **proveedor de modelo actual** cuando está disponible.
-   `/usage off|tokens|full` en chats: pie de página de uso por respuesta (OAuth muestra solo tokens).
-   `/usage cost` en chats: resumen de costos local agregado desde los registros de sesión de OpenClaw.
-   CLI: `openclaw status --usage` imprime un desglose completo por proveedor.
-   CLI: `openclaw channels list` imprime la misma instantánea de uso junto con la configuración del proveedor (usa `--no-usage` para omitir).
-   Barra de menú de macOS: sección "Uso" bajo Contexto (solo si está disponible).

## Proveedores + credenciales

-   **Anthropic (Claude)**: Tokens OAuth en perfiles de autenticación.
-   **GitHub Copilot**: Tokens OAuth en perfiles de autenticación.
-   **Gemini CLI**: Tokens OAuth en perfiles de autenticación.
-   **Antigravity**: Tokens OAuth en perfiles de autenticación.
-   **OpenAI Codex**: Tokens OAuth en perfiles de autenticación (se usa accountId cuando está presente).
-   **MiniMax**: Clave API (clave del plan de codificación; `MINIMAX_CODE_PLAN_KEY` o `MINIMAX_API_KEY`); usa la ventana del plan de codificación de 5 horas.
-   **z.ai**: Clave API a través de entorno/configuración/almacén de autenticación.

El uso está oculto si no existen credenciales OAuth/API coincidentes.

[Indicadores de escritura](./typing-indicators.md)[Zonas horarias](./timezone.md)