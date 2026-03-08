

  Herramientas integradas

  
# Detección de bucles en herramientas

OpenClaw puede evitar que los agentes se queden atascados en patrones repetitivos de llamadas a herramientas. Esta protección está **deshabilitada por defecto**. Habilítala solo donde sea necesario, porque con configuraciones estrictas puede bloquear llamadas repetidas legítimas.

## Por qué existe esto

-   Detectar secuencias repetitivas que no generan progreso.
-   Detectar bucles de alta frecuencia sin resultados (misma herramienta, mismos parámetros, errores repetidos).
-   Detectar patrones específicos de llamadas repetidas para herramientas de sondeo conocidas.

## Bloque de configuración

Valores predeterminados globales:

```json
{
  tools: {
    loopDetection: {
      enabled: false,
      historySize: 30,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

Anulación por agente (opcional):

```json
{
  agents: {
    list: [
      {
        id: "safe-runner",
        tools: {
          loopDetection: {
            enabled: true,
            warningThreshold: 8,
            criticalThreshold: 16,
          },
        },
      },
    ],
  },
}
```

### Comportamiento de los campos

-   `enabled`: Interruptor maestro. `false` significa que no se realiza ninguna detección de bucles.
-   `historySize`: número de llamadas a herramientas recientes que se conservan para el análisis.
-   `warningThreshold`: umbral antes de clasificar un patrón como solo advertencia.
-   `criticalThreshold`: umbral para bloquear patrones de bucle repetitivos.
-   `globalCircuitBreakerThreshold`: umbral global del cortacircuitos por falta de progreso.
-   `detectors.genericRepeat`: detecta patrones repetidos de misma-herramienta + mismos-parámetros.
-   `detectors.knownPollNoProgress`: detecta patrones de sondeo conocidos sin cambio de estado.
-   `detectors.pingPong`: detecta patrones alternantes de ping-pong.

## Configuración recomendada

-   Comienza con `enabled: true`, dejando los valores predeterminados sin cambios.
-   Mantén los umbrales ordenados como `warningThreshold < criticalThreshold < globalCircuitBreakerThreshold`.
-   Si ocurren falsos positivos:
    -   aumenta `warningThreshold` y/o `criticalThreshold`
    -   (opcionalmente) aumenta `globalCircuitBreakerThreshold`
    -   deshabilita solo el detector que causa problemas
    -   reduce `historySize` para un contexto histórico menos estricto

## Registros y comportamiento esperado

Cuando se detecta un bucle, OpenClaw reporta un evento de bucle y bloquea o atenúa el siguiente ciclo de herramientas dependiendo de la severidad. Esto protege a los usuarios del gasto descontrolado de tokens y bloqueos, preservando el acceso normal a las herramientas.

-   Prefiere primero la advertencia y la supresión temporal.
-   Escala solo cuando se acumula evidencia repetida.

## Notas

-   `tools.loopDetection` se fusiona con las anulaciones a nivel de agente.
-   La configuración por agente anula o extiende completamente los valores globales.
-   Si no existe configuración, las protecciones permanecen desactivadas.

[Lobster](./lobster.md)[Reacciones](./reactions.md)

---