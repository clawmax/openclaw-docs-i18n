

  Seguridad

  
# Verificación Formal (Modelos de Seguridad)

Esta página rastrea los **modelos de seguridad formal** de OpenClaw (TLA+/TLC hoy; más según sea necesario).

> Nota: algunos enlaces antiguos pueden referirse al nombre anterior del proyecto.

**Objetivo (estrella polar):** proporcionar un argumento verificado por máquina de que OpenClaw aplica su política de seguridad prevista (autorización, aislamiento de sesión, control de herramientas y seguridad contra configuraciones erróneas), bajo supuestos explícitos. **Qué es esto (hoy):** una **suite de regresión de seguridad** ejecutable y dirigida por atacantes:

-   Cada afirmación tiene una verificación de modelo ejecutable sobre un espacio de estados finito.
-   Muchas afirmaciones tienen un **modelo negativo** emparejado que produce un rastro de contraejemplo para una clase de error realista.

**Qué no es esto (todavía):** una prueba de que "OpenClaw es seguro en todos los aspectos" o de que la implementación completa en TypeScript es correcta.

## Dónde viven los modelos

Los modelos se mantienen en un repositorio separado: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

## Advertencias importantes

-   Estos son **modelos**, no la implementación completa en TypeScript. Es posible que haya divergencias entre el modelo y el código.
-   Los resultados están limitados por el espacio de estados explorado por TLC; "verde" no implica seguridad más allá de los supuestos y límites modelados.
-   Algunas afirmaciones dependen de supuestos ambientales explícitos (por ejemplo, despliegue correcto, entradas de configuración correctas).

## Reproduciendo resultados

Hoy, los resultados se reproducen clonando el repositorio de modelos localmente y ejecutando TLC (ver abajo). Una iteración futura podría ofrecer:

-   Modelos ejecutados en CI con artefactos públicos (rastros de contraejemplo, registros de ejecución)
-   un flujo de trabajo alojado de "ejecuta este modelo" para verificaciones pequeñas y acotadas

Para comenzar:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Se requiere Java 11+ (TLC se ejecuta en la JVM).
# El repositorio incluye una versión fija de `tla2tools.jar` (herramientas TLA+) y proporciona `bin/tlc` + objetivos de Make.

make <target>
```

### Exposición de puerta de enlace y configuración errónea de puerta de enlace abierta

**Afirmación:** enlazar más allá del localhost sin autenticación puede hacer posible un compromiso remoto / aumenta la exposición; el token/contraseña bloquea a atacantes no autenticados (según los supuestos del modelo).

-   Ejecuciones verdes:
    -   `make gateway-exposure-v2`
    -   `make gateway-exposure-v2-protected`
-   Rojo (esperado):
    -   `make gateway-exposure-v2-negative`

Ver también: `docs/gateway-exposure-matrix.md` en el repositorio de modelos.

### Pipeline nodes.run (capacidad de mayor riesgo)

**Afirmación:** `nodes.run` requiere (a) lista de comandos permitidos del nodo más comandos declarados y (b) aprobación en vivo cuando está configurado; las aprobaciones tienen token para prevenir repetición (en el modelo).

-   Ejecuciones verdes:
    -   `make nodes-pipeline`
    -   `make approvals-token`
-   Rojo (esperado):
    -   `make nodes-pipeline-negative`
    -   `make approvals-token-negative`

### Almacén de emparejamiento (control de DM)

**Afirmación:** las solicitudes de emparejamiento respetan el TTL y los límites de solicitudes pendientes.

-   Ejecuciones verdes:
    -   `make pairing`
    -   `make pairing-cap`
-   Rojo (esperado):
    -   `make pairing-negative`
    -   `make pairing-cap-negative`

### Control de ingreso (omisión de menciones + comandos de control)

**Afirmación:** en contextos de grupo que requieren mención, un "comando de control" no autorizado no puede omitir el control por mención.

-   Verde:
    -   `make ingress-gating`
-   Rojo (esperado):
    -   `make ingress-gating-negative`

### Aislamiento de enrutamiento/clave de sesión

**Afirmación:** los DM de pares distintos no colapsan en la misma sesión a menos que estén explícitamente vinculados/configurados.

-   Verde:
    -   `make routing-isolation`
-   Rojo (esperado):
    -   `make routing-isolation-negative`

## v1++: modelos acotados adicionales (concurrencia, reintentos, corrección de trazas)

Estos son modelos de seguimiento que aumentan la fidelidad en torno a modos de fallo del mundo real (actualizaciones no atómicas, reintentos y distribución de mensajes).

### Concurrencia / idempotencia del almacén de emparejamiento

**Afirmación:** un almacén de emparejamiento debe hacer cumplir `MaxPending` e idempotencia incluso bajo intercalados (es decir, "verificar-entonces-escribir" debe ser atómico / bloqueado; la actualización no debe crear duplicados). Lo que significa:

-   Bajo solicitudes concurrentes, no se puede exceder `MaxPending` para un canal.
-   Las solicitudes/actualizaciones repetidas para el mismo `(canal, remitente)` no deben crear filas pendientes en vivo duplicadas.
-   Ejecuciones verdes:
    -   `make pairing-race` (verificación de límite atómica/bloqueada)
    -   `make pairing-idempotency`
    -   `make pairing-refresh`
    -   `make pairing-refresh-race`
-   Rojo (esperado):
    -   `make pairing-race-negative` (carrera de límite no atómica begin/commit)
    -   `make pairing-idempotency-negative`
    -   `make pairing-refresh-negative`
    -   `make pairing-refresh-race-negative`

### Correlación de trazas de ingreso / idempotencia

**Afirmación:** la ingesta debe preservar la correlación de trazas a través de la distribución y ser idempotente bajo reintentos del proveedor. Lo que significa:

-   Cuando un evento externo se convierte en múltiples mensajes internos, cada parte mantiene la misma identidad de traza/evento.
-   Los reintentos no resultan en doble procesamiento.
-   Si faltan IDs de evento del proveedor, la deduplicación recurre a una clave segura (por ejemplo, ID de traza) para evitar descartar eventos distintos.
-   Verde:
    -   `make ingress-trace`
    -   `make ingress-trace2`
    -   `make ingress-idempotency`
    -   `make ingress-dedupe-fallback`
-   Rojo (esperado):
    -   `make ingress-trace-negative`
    -   `make ingress-trace2-negative`
    -   `make ingress-idempotency-negative`
    -   `make ingress-dedupe-fallback-negative`

### Precedencia de dmScope de enrutamiento + identityLinks

**Afirmación:** el enrutamiento debe mantener las sesiones DM aisladas por defecto, y solo colapsar sesiones cuando esté explícitamente configurado (precedencia de canal + enlaces de identidad). Lo que significa:

-   Las anulaciones de dmScope específicas del canal deben prevalecer sobre los valores predeterminados globales.
-   identityLinks solo debe colapsar dentro de grupos vinculados explícitos, no entre pares no relacionados.
-   Verde:
    -   `make routing-precedence`
    -   `make routing-identitylinks`
-   Rojo (esperado):
    -   `make routing-precedence-negative`
    -   `make routing-identitylinks-negative`

[Tailscale](../gateway/tailscale.md)[README](./README.md)