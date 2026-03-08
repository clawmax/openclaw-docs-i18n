

  Mensajes y entrega

  
# Política de Reintentos

## Objetivos

-   Reintentar por solicitud HTTP, no por flujo de múltiples pasos.
-   Preservar el orden reintentando solo el paso actual.
-   Evitar duplicar operaciones no idempotentes.

## Valores Predeterminados

-   Intentos: 3
-   Límite máximo de retraso: 30000 ms
-   Jitter: 0.1 (10 por ciento)
-   Valores predeterminados del proveedor:
    -   Telegram retraso mínimo: 400 ms
    -   Discord retraso mínimo: 500 ms

## Comportamiento

### Discord

-   Reintenta solo en errores de límite de tasa (HTTP 429).
-   Usa `retry_after` de Discord cuando está disponible; de lo contrario, usa retroceso exponencial.

### Telegram

-   Reintenta en errores transitorios (429, timeout, connect/reset/closed, temporalmente no disponible).
-   Usa `retry_after` cuando está disponible; de lo contrario, usa retroceso exponencial.
-   Los errores de análisis de Markdown no se reintentan; se recurre a texto plano.

## Configuración

Establece la política de reintentos por proveedor en `~/.openclaw/openclaw.json`:

```json
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

## Notas

-   Los reintentos se aplican por solicitud (envío de mensaje, carga de medios, reacción, encuesta, sticker).
-   Los flujos compuestos no reintentan pasos completados.

[Streaming y Fragmentación](./streaming.md)[Cola de Comandos](./queue.md)