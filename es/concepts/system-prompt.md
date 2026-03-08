

  Fundamentos

  
# Prompt del Sistema

OpenClaw construye un prompt del sistema personalizado para cada ejecución de agente. El prompt es **propiedad de OpenClaw** y no utiliza el prompt por defecto de pi-coding-agent. El prompt es ensamblado por OpenClaw e inyectado en cada ejecución de agente.

## Estructura

El prompt es intencionalmente compacto y utiliza secciones fijas:

-   **Herramientas**: lista actual de herramientas + descripciones breves.
-   **Seguridad**: recordatorio breve de barreras de seguridad para evitar comportamientos de búsqueda de poder o eludir la supervisión.
-   **Habilidades** (cuando están disponibles): le indica al modelo cómo cargar instrucciones de habilidades bajo demanda.
-   **Auto-actualización de OpenClaw**: cómo ejecutar `config.apply` y `update.run`.
-   **Espacio de trabajo**: directorio de trabajo (`agents.defaults.workspace`).
-   **Documentación**: ruta local a la documentación de OpenClaw (repositorio o paquete npm) y cuándo leerla.
-   **Archivos del Espacio de Trabajo (inyectados)**: indica que los archivos de arranque se incluyen a continuación.
-   **Sandbox** (cuando está habilitado): indica entorno de ejecución aislado, rutas del sandbox, y si hay ejecución elevada disponible.
-   **Fecha y Hora Actuales**: hora local del usuario, zona horaria y formato de hora.
-   **Etiquetas de Respuesta**: sintaxis opcional de etiquetas de respuesta para proveedores compatibles.
-   **Latidos**: prompt de latido y comportamiento de confirmación.
-   **Entorno de Ejecución**: host, SO, node, modelo, raíz del repositorio (cuando se detecta), nivel de pensamiento (una línea).
-   **Razonamiento**: nivel de visibilidad actual + pista del conmutador /reasoning.

Las barreras de seguridad en el prompt del sistema son de carácter consultivo. Guían el comportamiento del modelo pero no hacen cumplir la política. Utiliza la política de herramientas, aprobaciones de ejecución, aislamiento en sandbox y listas de canales permitidos para una aplicación estricta; los operadores pueden desactivar estos por diseño.

## Modos de prompt

OpenClaw puede renderizar prompts del sistema más pequeños para subagentes. El entorno de ejecución establece un `promptMode` para cada ejecución (no es una configuración visible para el usuario):

-   `full` (por defecto): incluye todas las secciones anteriores.
-   `minimal`: utilizado para subagentes; omite **Habilidades**, **Recuperación de Memoria**, **Auto-actualización de OpenClaw**, **Alias de Modelos**, **Identidad del Usuario**, **Etiquetas de Respuesta**, **Mensajería**, **Respuestas Silenciosas** y **Latidos**. Las Herramientas, **Seguridad**, Espacio de Trabajo, Sandbox, Fecha y Hora Actuales (cuando se conocen), Entorno de Ejecución y el contexto inyectado permanecen disponibles.
-   `none`: devuelve solo la línea de identidad base.

Cuando `promptMode=minimal`, los prompts inyectados adicionales se etiquetan como **Contexto del Subagente** en lugar de **Contexto del Chat Grupal**.

## Inyección de arranque del espacio de trabajo

Los archivos de arranque se recortan y se añaden bajo **Contexto del Proyecto** para que el modelo vea la identidad y el contexto del perfil sin necesidad de lecturas explícitas:

-   `AGENTS.md`
-   `SOUL.md`
-   `TOOLS.md`
-   `IDENTITY.md`
-   `USER.md`
-   `HEARTBEAT.md`
-   `BOOTSTRAP.md` (solo en espacios de trabajo nuevos)
-   `MEMORY.md` y/o `memory.md` (cuando están presentes en el espacio de trabajo; uno o ambos pueden ser inyectados)

Todos estos archivos son **inyectados en la ventana de contexto** en cada turno, lo que significa que consumen tokens. Mantenlos concisos — especialmente `MEMORY.md`, que puede crecer con el tiempo y llevar a un uso de contexto inesperadamente alto y a una compactación más frecuente.

> **Nota:** Los archivos diarios `memory/*.md` **no** se inyectan automáticamente. Se accede a ellos bajo demanda a través de las herramientas `memory_search` y `memory_get`, por lo que no cuentan contra la ventana de contexto a menos que el modelo los lea explícitamente.

Los archivos grandes se truncan con un marcador. El tamaño máximo por archivo se controla con `agents.defaults.bootstrapMaxChars` (por defecto: 20000). El contenido total inyectado de arranque entre archivos está limitado por `agents.defaults.bootstrapTotalMaxChars` (por defecto: 150000). Los archivos faltantes inyectan un marcador breve de archivo faltante. Cuando ocurre truncamiento, OpenClaw puede inyectar un bloque de advertencia en el Contexto del Proyecto; controla esto con `agents.defaults.bootstrapPromptTruncationWarning` (`off`, `once`, `always`; por defecto: `once`). Las sesiones de subagentes solo inyectan `AGENTS.md` y `TOOLS.md` (otros archivos de arranque se filtran para mantener el contexto del subagente pequeño). Los hooks internos pueden interceptar este paso a través de `agent:bootstrap` para mutar o reemplazar los archivos de arranque inyectados (por ejemplo, intercambiando `SOUL.md` por una persona alternativa). Para inspeccionar cuánto contribuye cada archivo inyectado (crudo vs inyectado, truncamiento, más la sobrecarga del esquema de herramientas), usa `/context list` o `/context detail`. Consulta [Contexto](./context.md).

## Manejo del tiempo

El prompt del sistema incluye una sección dedicada **Fecha y Hora Actuales** cuando se conoce la zona horaria del usuario. Para mantener el prompt estable en caché, ahora solo incluye la **zona horaria** (sin reloj dinámico o formato de hora). Usa `session_status` cuando el agente necesite la hora actual; la tarjeta de estado incluye una línea de marca de tiempo. Configura con:

-   `agents.defaults.userTimezone`
-   `agents.defaults.timeFormat` (`auto` | `12` | `24`)

Consulta [Fecha y Hora](../date-time.md) para detalles completos del comportamiento.

## Habilidades

Cuando existen habilidades elegibles, OpenClaw inyecta una **lista compacta de habilidades disponibles** (`formatSkillsForPrompt`) que incluye la **ruta del archivo** para cada habilidad. El prompt instruye al modelo a usar `read` para cargar el SKILL.md en la ubicación listada (espacio de trabajo, gestionado o incluido). Si no hay habilidades elegibles, la sección Habilidades se omite.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

Esto mantiene el prompt base pequeño mientras aún permite el uso dirigido de habilidades.

## Documentación

Cuando está disponible, el prompt del sistema incluye una sección **Documentación** que apunta al directorio local de documentación de OpenClaw (ya sea `docs/` en el espacio de trabajo del repositorio o la documentación del paquete npm incluido) y también menciona el espejo público, el repositorio fuente, la comunidad de Discord y ClawHub ([https://clawhub.com](https://clawhub.com)) para el descubrimiento de habilidades. El prompt instruye al modelo a consultar primero la documentación local para el comportamiento, comandos, configuración o arquitectura de OpenClaw, y a ejecutar `openclaw status` él mismo cuando sea posible (preguntando al usuario solo cuando no tenga acceso).

[Bucle del Agente](./agent-loop.md)[Contexto](./context.md)