

  Experimentos

  
# Investigación de Memoria del Espacio de Trabajo

Objetivo: Espacio de trabajo estilo Clawd (`agents.defaults.workspace`, por defecto `~/.openclaw/workspace`) donde la "memoria" se almacena como un archivo Markdown por día (`memory/YYYY-MM-DD.md`) más un pequeño conjunto de archivos estables (ej. `memory.md`, `SOUL.md`). Este documento propone una arquitectura de memoria **offline-first** que mantiene Markdown como la fuente de verdad canónica y revisable, pero añade **recuerdo estructurado** (búsqueda, resúmenes de entidades, actualizaciones de confianza) mediante un índice derivado.

## ¿Por qué cambiar?

La configuración actual (un archivo por día) es excelente para:

-   llevar un diario de "solo añadir"
-   edición humana
-   durabilidad y auditabilidad respaldada por git
-   captura de baja fricción ("simplemente escríbelo")

Es débil para:

-   recuperación de alta precisión ("¿qué decidimos sobre X?", "¿la última vez que intentamos Y?")
-   respuestas centradas en entidades ("háblame sobre Alice / El Castillo / warelay") sin releer muchos archivos
-   estabilidad de opiniones/preferencias (y evidencia cuando cambian)
-   restricciones de tiempo ("¿qué era cierto durante Nov 2025?") y resolución de conflictos

## Objetivos de diseño

-   **Offline**: funciona sin red; puede ejecutarse en portátil/Castle; sin dependencia de la nube.
-   **Explicable**: los elementos recuperados deben ser atribuibles (archivo + ubicación) y separables de la inferencia.
-   **Baja ceremonia**: el registro diario sigue siendo Markdown, sin trabajo pesado de esquemas.
-   **Incremental**: la v1 es útil solo con FTS; las mejoras semánticas/vectoriales y de grafos son opcionales.
-   **Amigable para agentes**: facilita el "recuerdo dentro de presupuestos de tokens" (devuelve pequeños paquetes de hechos).

## Modelo estrella del norte (Hindsight × Letta)

Dos piezas para combinar:

1.  **Bucle de control estilo Letta/MemGPT**

-   mantener un "núcleo" pequeño siempre en contexto (persona + datos clave del usuario)
-   todo lo demás está fuera de contexto y se recupera mediante herramientas
-   las escrituras de memoria son llamadas explícitas a herramientas (añadir/reemplazar/insertar), que se persisten y luego se reinyectan en el siguiente turno

2.  **Sustrato de memoria estilo Hindsight**

-   separar lo observado vs lo creído vs lo resumido
-   soportar retener/recordar/reflexionar
-   opiniones con confianza que pueden evolucionar con evidencia
-   recuperación consciente de entidades + consultas temporales (incluso sin grafos de conocimiento completos)

## Arquitectura propuesta (Markdown como fuente de verdad + índice derivado)

### Almacén canónico (amigable con git)

Mantener `~/.openclaw/workspace` como memoria canónica legible por humanos. Diseño sugerido del espacio de trabajo:

```
~/.openclaw/workspace/
  memory.md                    # pequeño: hechos duraderos + preferencias (similar al núcleo)
  memory/
    YYYY-MM-DD.md              # registro diario (añadir; narrativa)
  bank/                        # páginas de memoria "tipadas" (estables, revisables)
    world.md                   # hechos objetivos sobre el mundo
    experience.md              # lo que hizo el agente (en primera persona)
    opinions.md                # preferencias/juicios subjetivos + confianza + punteros a evidencia
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Notas:

-   **El registro diario sigue siendo registro diario**. No es necesario convertirlo en JSON.
-   Los archivos en `bank/` están **curados**, producidos por trabajos de reflexión, y aún pueden ser editados a mano.
-   `memory.md` permanece "pequeño + similar al núcleo": las cosas que quieres que Clawd vea en cada sesión.

### Almacén derivado (recuerdo de máquina)

Añadir un índice derivado bajo el espacio de trabajo (no necesariamente rastreado por git):

```
~/.openclaw/workspace/.memory/index.sqlite
```

Respaldarlo con:

-   Esquema SQLite para hechos + enlaces de entidades + metadatos de opiniones
-   **FTS5** de SQLite para recuerdo léxico (rápido, pequeño, offline)
-   tabla opcional de embeddings para recuerdo semántico (todavía offline)

El índice siempre es **reconstruible desde Markdown**.

## Retener / Recordar / Reflexionar (bucle operativo)

### Retener: normalizar registros diarios en "hechos"

La idea clave de Hindsight que importa aquí: almacenar **hechos narrativos y autocontenidos**, no fragmentos pequeños. Regla práctica para `memory/YYYY-MM-DD.md`:

-   al final del día (o durante), añadir una sección `## Retain` con 2–5 viñetas que sean:
    -   narrativas (contexto entre turnos preservado)
    -   autocontenidas (tienen sentido por sí solas más tarde)
    -   etiquetadas con tipo + menciones de entidades

Ejemplo:

```markdown
## Retain
- W @Peter: Actualmente en Marrakech (27 nov – 1 dic, 2025) para el cumpleaños de Andy.
- B @warelay: Arreglé el fallo WS de Baileys envolviendo los manejadores connection.update en try/catch (ver memory/2025-11-27.md).
- O(c=0.95) @Peter: Prefiere respuestas concisas (<1500 caracteres) en WhatsApp; el contenido largo va a archivos.
```

Análisis mínimo:

-   Prefijo de tipo: `W` (mundo), `B` (experiencia/biográfico), `O` (opinión), `S` (observación/resumen; usualmente generado)
-   Entidades: `@Peter`, `@warelay`, etc (los slugs se mapean a `bank/entities/*.md`)
-   Confianza de opinión: `O(c=0.0..1.0)` opcional

Si no quieres que los autores piensen en ello: el trabajo de reflexión puede inferir estas viñetas del resto del registro, pero tener una sección explícita `## Retain` es la "palanca de calidad" más fácil.

### Recordar: consultas sobre el índice derivado

El recuerdo debe soportar:

-   **léxico**: "encontrar términos/nombres/comandos exactos" (FTS5)
-   **de entidad**: "háblame sobre X" (páginas de entidad + hechos vinculados a entidades)
-   **temporal**: "qué pasó alrededor del 27 de nov" / "desde la semana pasada"
-   **de opinión**: "¿qué prefiere Peter?" (con confianza + evidencia)

El formato de retorno debe ser amigable para el agente y citar fuentes:

-   `kind` (`world|experience|opinion|observation`)
-   `timestamp` (día fuente, o rango de tiempo extraído si está presente)
-   `entities` (`["Peter","warelay"]`)
-   `content` (el hecho narrativo)
-   `source` (`memory/2025-11-27.md#L12` etc)

### Reflexionar: producir páginas estables + actualizar creencias

La reflexión es un trabajo programado (diario o latido `ultrathink`) que:

-   actualiza `bank/entities/*.md` a partir de hechos recientes (resúmenes de entidades)
-   actualiza la confianza en `bank/opinions.md` basándose en refuerzo/contradicción
-   opcionalmente propone ediciones a `memory.md` (hechos duraderos "similares al núcleo")

Evolución de opiniones (simple, explicable):

-   cada opinión tiene:
    -   declaración
    -   confianza `c ∈ [0,1]`
    -   última_actualización
    -   enlaces de evidencia (IDs de hechos que apoyan + contradicen)
-   cuando llegan nuevos hechos:
    -   encontrar opiniones candidatas por superposición de entidades + similitud (FTS primero, embeddings después)
    -   actualizar la confianza con pequeños deltas; los saltos grandes requieren fuerte contradicción + evidencia repetida

## Integración CLI: independiente vs integración profunda

Recomendación: **integración profunda en OpenClaw**, pero mantener una biblioteca central separable.

### ¿Por qué integrar en OpenClaw?

-   OpenClaw ya conoce:
    -   la ruta del espacio de trabajo (`agents.defaults.workspace`)
    -   el modelo de sesión + latidos
    -   patrones de registro + solución de problemas
-   Quieres que el agente mismo llame a las herramientas:
    -   `openclaw memory recall "…" --k 25 --since 30d`
    -   `openclaw memory reflect --since 7d`

### ¿Por qué aún separar una biblioteca?

-   mantener la lógica de memoria probable sin gateway/runtime
-   reutilizar desde otros contextos (scripts locales, futura aplicación de escritorio, etc.)

Forma: La herramienta de memoria está destinada a ser una pequeña capa de CLI + biblioteca, pero esto es solo exploratorio.

## “S-Collide” / SuCo: cuándo usarlo (investigación)

Si “S-Collide” se refiere a **SuCo (Subspace Collision)**: es un enfoque de recuperación ANN que apunta a fuertes compensaciones de precisión/latencia usando colisiones aprendidas/estructuradas en subespacios (artículo: arXiv 2411.14754, 2024). Enfoque pragmático para `~/.openclaw/workspace`:

-   **no empezar** con SuCo.
-   empezar con SQLite FTS + (opcional) embeddings simples; obtendrás la mayoría de las ventajas de UX inmediatamente.
-   considerar soluciones de clase SuCo/HNSW/ScaNN solo cuando:
    -   el corpus sea grande (decenas/cientos de miles de fragmentos)
    -   la búsqueda por fuerza bruta de embeddings se vuelva demasiado lenta
    -   la calidad del recuerdo esté limitada significativamente por la búsqueda léxica

Alternativas amigables con offline (en complejidad creciente):

-   SQLite FTS5 + filtros de metadatos (cero ML)
-   Embeddings + fuerza bruta (funciona sorprendentemente lejos si el recuento de fragmentos es bajo)
-   Índice HNSW (común, robusto; necesita un enlace a biblioteca)
-   SuCo (grado de investigación; atractivo si hay una implementación sólida que puedas integrar)

Pregunta abierta:

-   ¿cuál es el **mejor** modelo de embedding offline para "memoria de asistente personal" en tus máquinas (portátil + escritorio)?
    -   si ya tienes Ollama: incrustar con un modelo local; de lo contrario, incluir un modelo de embedding pequeño en la cadena de herramientas.

## Piloto más pequeño útil

Si quieres una versión mínima, pero aún útil:

-   Añadir páginas de entidad en `bank/` y una sección `## Retain` en los registros diarios.
-   Usar SQLite FTS para recuerdo con citas (ruta + números de línea).
-   Añadir embeddings solo si la calidad del recuerdo o la escala lo exigen.

## Referencias

-   Conceptos Letta / MemGPT: "bloques de memoria central" + "memoria de archivo" + memoria de autoedición impulsada por herramientas.
-   Informe Técnico de Hindsight: "retener / recordar / reflexionar", memoria de cuatro redes, extracción de hechos narrativos, evolución de confianza de opiniones.
-   SuCo: arXiv 2411.14754 (2024): "Subspace Collision" recuperación aproximada del vecino más cercano.

[Plan de Enlace de Sesión Agnóstico al Canal](../plans/session-binding-channel-agnostic.md)[Exploración de Configuración de Modelo](../proposals/model-config.md)