

  Automatización

  
# Cron vs Heartbeat

Tanto los heartbeats como los cron jobs te permiten ejecutar tareas en un horario. Esta guía te ayuda a elegir el mecanismo correcto para tu caso de uso.

## Guía Rápida de Decisión

| Caso de Uso | Recomendado | Por qué |
| --- | --- | --- |
| Revisar bandeja de entrada cada 30 min | Heartbeat | Agrupa con otras revisiones, consciente del contexto |
| Enviar informe diario a las 9am en punto | Cron (aislado) | Se necesita sincronización exacta |
| Monitorear calendario para eventos próximos | Heartbeat | Ajuste natural para conciencia periódica |
| Ejecutar análisis profundo semanal | Cron (aislado) | Tarea independiente, puede usar un modelo diferente |
| Recordarme en 20 minutos | Cron (principal, `--at`) | Tarea única con sincronización precisa |
| Revisión de salud de proyecto en segundo plano | Heartbeat | Se aprovecha del ciclo existente |

## Heartbeat: Conciencia Periódica

Los heartbeats se ejecutan en la **sesión principal** a intervalos regulares (por defecto: 30 min). Están diseñados para que el agente revise cosas y destaque cualquier asunto importante.

### Cuándo usar heartbeat

-   **Múltiples revisiones periódicas**: En lugar de 5 cron jobs separados revisando bandeja de entrada, calendario, clima, notificaciones y estado del proyecto, un solo heartbeat puede agruparlos todos.
-   **Decisiones conscientes del contexto**: El agente tiene el contexto completo de la sesión principal, por lo que puede tomar decisiones inteligentes sobre qué es urgente vs. qué puede esperar.
-   **Continuidad conversacional**: Los heartbeats comparten la misma sesión, por lo que el agente recuerda conversaciones recientes y puede hacer seguimiento de forma natural.
-   **Monitoreo de baja sobrecarga**: Un heartbeat reemplaza muchas tareas pequeñas de sondeo.

### Ventajas del heartbeat

-   **Agrupa múltiples revisiones**: Un turno del agente puede revisar bandeja de entrada, calendario y notificaciones juntos.
-   **Reduce llamadas a la API**: Un solo heartbeat es más económico que 5 cron jobs aislados.
-   **Consciente del contexto**: El agente sabe en qué has estado trabajando y puede priorizar en consecuencia.
-   **Supresión inteligente**: Si nada necesita atención, el agente responde `HEARTBEAT_OK` y no se entrega ningún mensaje.
-   **Sincronización natural**: Se desvía ligeramente según la carga de la cola, lo cual está bien para la mayoría del monitoreo.

### Ejemplo de heartbeat: Lista de verificación HEARTBEAT.md

```bash
# Heartbeat checklist

- Check email for urgent messages
- Review calendar for events in next 2 hours
- If a background task finished, summarize results
- If idle for 8+ hours, send a brief check-in
```

El agente lee esto en cada heartbeat y maneja todos los elementos en un turno.

### Configurar heartbeat

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // intervalo
        target: "last", // destino explícito de entrega de alertas (por defecto es "none")
        activeHours: { start: "08:00", end: "22:00" }, // opcional
      },
    },
  },
}
```

Consulta [Heartbeat](../gateway/heartbeat.md) para la configuración completa.

## Cron: Programación Precisa

Los cron jobs se ejecutan en momentos precisos y pueden ejecutarse en sesiones aisladas sin afectar el contexto principal. Los horarios recurrentes de cada hora en punto se distribuyen automáticamente mediante un desplazamiento determinista por trabajo en una ventana de 0 a 5 minutos.

### Cuándo usar cron

-   **Se requiere sincronización exacta**: "Envía esto a las 9:00 AM todos los lunes" (no "alrededor de las 9").
-   **Tareas independientes**: Tareas que no necesitan contexto conversacional.
-   **Modelo/pensamiento diferente**: Análisis pesado que justifica un modelo más potente.
-   **Recordatorios únicos**: "Recuérdame en 20 minutos" con `--at`.
-   **Tareas ruidosas/frecuentes**: Tareas que desordenarían el historial de la sesión principal.
-   **Disparadores externos**: Tareas que deben ejecutarse independientemente de si el agente está activo o no.

### Ventajas de cron

-   **Sincronización precisa**: Expresiones cron de 5 o 6 campos (segundos) con soporte de zona horaria.
-   **Distribución de carga incorporada**: los horarios recurrentes de cada hora en punto se escalonan hasta 5 minutos por defecto.
-   **Control por trabajo**: anula el escalonamiento con `--stagger ` o fuerza la sincronización exacta con `--exact`.
-   **Aislamiento de sesión**: Se ejecuta en `cron:` sin contaminar el historial principal.
-   **Anulaciones de modelo**: Usa un modelo más económico o más potente por trabajo.
-   **Control de entrega**: Los trabajos aislados por defecto usan `announce` (resumen); elige `none` según sea necesario.
-   **Entrega inmediata**: El modo announce publica directamente sin esperar al heartbeat.
-   **No necesita contexto del agente**: Se ejecuta incluso si la sesión principal está inactiva o compactada.
-   **Soporte para tareas únicas**: `--at` para marcas de tiempo futuras precisas.

### Ejemplo de cron: Resumen matutino diario

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Esto se ejecuta exactamente a las 7:00 AM hora de Nueva York, usa Opus para calidad y anuncia un resumen directamente a WhatsApp.

### Ejemplo de cron: Recordatorio único

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

Consulta [Cron jobs](./cron-jobs.md) para la referencia completa de CLI.

## Diagrama de Flujo de Decisión

```
¿La tarea necesita ejecutarse en un momento EXACTO?
  SÍ -> Usa cron
  NO  -> Continúa...

¿La tarea necesita aislamiento de la sesión principal?
  SÍ -> Usa cron (aislado)
  NO  -> Continúa...

¿Esta tarea puede agruparse con otras revisiones periódicas?
  SÍ -> Usa heartbeat (agrégala a HEARTBEAT.md)
  NO  -> Usa cron

¿Es un recordatorio único?
  SÍ -> Usa cron con --at
  NO  -> Continúa...

¿Necesita un modelo o nivel de pensamiento diferente?
  SÍ -> Usa cron (aislado) con --model/--thinking
  NO  -> Usa heartbeat
```

## Combinando Ambos

La configuración más eficiente usa **ambos**:

1.  **Heartbeat** maneja el monitoreo de rutina (bandeja de entrada, calendario, notificaciones) en un turno agrupado cada 30 minutos.
2.  **Cron** maneja horarios precisos (informes diarios, revisiones semanales) y recordatorios únicos.

### Ejemplo: Configuración eficiente de automatización

**HEARTBEAT.md** (revisado cada 30 min):

```bash
# Heartbeat checklist

- Scan inbox for urgent emails
- Check calendar for events in next 2h
- Review any pending tasks
- Light check-in if quiet for 8+ hours
```

**Cron jobs** (sincronización precisa):

```bash
# Daily morning briefing at 7am
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --announce

# Weekly project review on Mondays at 9am
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# One-shot reminder
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

## Lobster: Flujos de trabajo deterministas con aprobaciones

Lobster es el entorno de ejecución de flujos de trabajo para **canalizaciones de herramientas de múltiples pasos** que necesitan ejecución determinista y aprobaciones explícitas. Úsalo cuando la tarea sea más que un solo turno del agente, y quieras un flujo de trabajo reanudable con puntos de control humanos.

### Cuándo encaja Lobster

-   **Automatización de múltiples pasos**: Necesitas una canalización fija de llamadas a herramientas, no una instrucción única.
-   **Puertas de aprobación**: Los efectos secundarios deben pausarse hasta que apruebes, luego reanudarse.
-   **Ejecuciones reanudables**: Continúa un flujo de trabajo pausado sin volver a ejecutar pasos anteriores.

### Cómo se combina con heartbeat y cron

-   **Heartbeat/cron** deciden *cuándo* ocurre una ejecución.
-   **Lobster** define *qué pasos* ocurren una vez que comienza la ejecución.

Para flujos de trabajo programados, usa cron o heartbeat para activar un turno del agente que llame a Lobster. Para flujos de trabajo ad-hoc, llama a Lobster directamente.

### Notas operativas (del código)

-   Lobster se ejecuta como un **subproceso local** (CLI `lobster`) en modo herramienta y devuelve un **sobre JSON**.
-   Si la herramienta devuelve `needs_approval`, reanudas con un `resumeToken` y un indicador `approve`.
-   La herramienta es un **complemento opcional**; habilítalo de forma aditiva mediante `tools.alsoAllow: ["lobster"]` (recomendado).
-   Lobster espera que la CLI `lobster` esté disponible en `PATH`.

Consulta [Lobster](../tools/lobster.md) para el uso completo y ejemplos.

## Sesión Principal vs Sesión Aislada

Tanto heartbeat como cron pueden interactuar con la sesión principal, pero de manera diferente:

|  | Heartbeat | Cron (principal) | Cron (aislado) |
| --- | --- | --- | --- |
| Sesión | Principal | Principal (vía evento del sistema) | `cron:` |
| Historial | Compartido | Compartido | Nuevo cada ejecución |
| Contexto | Completo | Completo | Ninguno (comienza limpio) |
| Modelo | Modelo de sesión principal | Modelo de sesión principal | Se puede anular |
| Salida | Se entrega si no es `HEARTBEAT_OK` | Instrucción de heartbeat + evento | Anuncia resumen (por defecto) |

### Cuándo usar cron en sesión principal

Usa `--session main` con `--system-event` cuando quieras:

-   Que el recordatorio/evento aparezca en el contexto de la sesión principal
-   Que el agente lo maneje durante el próximo heartbeat con contexto completo
-   Que no haya una ejecución aislada separada

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

### Cuándo usar cron aislado

Usa `--session isolated` cuando quieras:

-   Un estado limpio sin contexto previo
-   Configuraciones de modelo o pensamiento diferentes
-   Anunciar resúmenes directamente a un canal
-   Un historial que no desordene la sesión principal

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --announce
```

## Consideraciones de Costo

| Mecanismo | Perfil de Costo |
| --- | --- |
| Heartbeat | Un turno cada N minutos; escala con el tamaño de HEARTBEAT.md |
| Cron (principal) | Agrega evento al próximo heartbeat (sin turno aislado) |
| Cron (aislado) | Turno completo del agente por trabajo; puede usar un modelo más económico |

**Consejos**:

-   Mantén `HEARTBEAT.md` pequeño para minimizar la sobrecarga de tokens.
-   Agrupa revisiones similares en heartbeat en lugar de múltiples cron jobs.
-   Usa `target: "none"` en heartbeat si solo quieres procesamiento interno.
-   Usa cron aislado con un modelo más económico para tareas rutinarias.

## Relacionado

-   [Heartbeat](../gateway/heartbeat.md) - configuración completa de heartbeat
-   [Cron jobs](./cron-jobs.md) - referencia completa de CLI y API de cron
-   [System](../cli/system.md) - eventos del sistema + controles de heartbeat

[Cron Jobs](./cron-jobs.md)[Solución de Problemas de Automatización](./troubleshooting.md)