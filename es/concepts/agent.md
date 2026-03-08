

  Fundamentos

  
# Entorno de Ejecución del Agente

OpenClaw ejecuta un único entorno de ejecución de agente integrado derivado de **pi-mono**.

## Espacio de trabajo (requerido)

OpenClaw utiliza un único directorio de espacio de trabajo del agente (`agents.defaults.workspace`) como el **único** directorio de trabajo (`cwd`) del agente para herramientas y contexto. Recomendado: usa `openclaw setup` para crear `~/.openclaw/openclaw.json` si falta e inicializar los archivos del espacio de trabajo. Diseño completo del espacio de trabajo + guía de respaldo: [Espacio de trabajo del agente](./agent-workspace.md) Si `agents.defaults.sandbox` está habilitado, las sesiones no principales pueden anular esto con espacios de trabajo por sesión bajo `agents.defaults.sandbox.workspaceRoot` (ver [Configuración de Gateway](../gateway/configuration.md)).

## Archivos de arranque (inyectados)

Dentro de `agents.defaults.workspace`, OpenClaw espera estos archivos editables por el usuario:

-   `AGENTS.md` — instrucciones de operación + "memoria"
-   `SOUL.md` — persona, límites, tono
-   `TOOLS.md` — notas de herramientas mantenidas por el usuario (ej. `imsg`, `sag`, convenciones)
-   `BOOTSTRAP.md` — ritual de primera ejecución único (se elimina después de completarse)
-   `IDENTITY.md` — nombre/vibra/emoji del agente
-   `USER.md` — perfil del usuario + forma de tratamiento preferida

En el primer turno de una nueva sesión, OpenClaw inyecta el contenido de estos archivos directamente en el contexto del agente. Los archivos en blanco se omiten. Los archivos grandes se recortan y truncan con un marcador para que las indicaciones se mantengan ligeras (lee el archivo para el contenido completo). Si falta un archivo, OpenClaw inyecta una única línea de marcador de "archivo faltante" (y `openclaw setup` creará una plantilla predeterminada segura). `BOOTSTRAP.md` solo se crea para un **espacio de trabajo completamente nuevo** (no hay otros archivos de arranque presentes). Si lo eliminas después de completar el ritual, no debería recrearse en reinicios posteriores. Para deshabilitar completamente la creación de archivos de arranque (para espacios de trabajo preconfigurados), establece:

```json
{ agent: { skipBootstrap: true } }
```

## Herramientas integradas

Las herramientas principales (read/exec/edit/write y herramientas del sistema relacionadas) están siempre disponibles, sujetas a la política de herramientas. `apply_patch` es opcional y controlado por `tools.exec.applyPatch`. `TOOLS.md` **no** controla qué herramientas existen; es una guía sobre cómo *tú* quieres que se usen.

## Habilidades

OpenClaw carga habilidades desde tres ubicaciones (el espacio de trabajo gana en caso de conflicto de nombre):

-   Incluidas (incluidas en la instalación)
-   Gestionadas/locales: `~/.openclaw/skills`
-   Espacio de trabajo: `/skills`

Las habilidades pueden estar controladas por configuración/entorno (ver `skills` en [Configuración de Gateway](../gateway/configuration.md)).

## Integración con pi-mono

OpenClaw reutiliza partes de la base de código de pi-mono (modelos/herramientas), pero **la gestión de sesiones, el descubrimiento y el cableado de herramientas son propiedad de OpenClaw**.

-   No hay entorno de ejecución de agente de pi-coding.
-   No se consultan configuraciones de `~/.pi/agent` o `/.pi`.

## Sesiones

Las transcripciones de sesión se almacenan como JSONL en:

-   `~/.openclaw/agents//sessions/.jsonl`

El ID de sesión es estable y lo elige OpenClaw. Las carpetas de sesión heredadas de Pi/Tau **no** se leen.

## Direccionamiento durante transmisión

Cuando el modo de cola es `steer`, los mensajes entrantes se inyectan en la ejecución actual. La cola se verifica **después de cada llamada a herramienta**; si hay un mensaje en cola, se omiten las llamadas a herramientas restantes del mensaje actual del asistente (los resultados de herramientas de error con "Omitido debido a mensaje de usuario en cola."), luego el mensaje de usuario en cola se inyecta antes de la siguiente respuesta del asistente. Cuando el modo de cola es `followup` o `collect`, los mensajes entrantes se retienen hasta que termina el turno actual, luego comienza un nuevo turno del agente con las cargas útiles en cola. Ver [Cola](./queue.md) para el comportamiento de modo + debounce/límite. La transmisión por bloques envía los bloques de asistente completados tan pronto como terminan; está **desactivada por defecto** (`agents.defaults.blockStreamingDefault: "off"`). Ajusta el límite mediante `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; por defecto es text_end). Controla la división en fragmentos de bloques suaves con `agents.defaults.blockStreamingChunk` (por defecto 800–1200 caracteres; prefiere saltos de párrafo, luego saltos de línea; las oraciones al final). Fusiona fragmentos transmitidos con `agents.defaults.blockStreamingCoalesce` para reducir el spam de una sola línea (fusión basada en inactividad antes de enviar). Los canales que no son Telegram requieren `*.blockStreaming: true` explícito para habilitar respuestas por bloques. Los resúmenes detallados de herramientas se emiten al inicio de la herramienta (sin debounce); la interfaz de usuario de Control transmite la salida de la herramienta mediante eventos del agente cuando está disponible. Más detalles: [Transmisión + fragmentación](./streaming.md).

## Referencias de modelos

Las referencias de modelos en la configuración (por ejemplo `agents.defaults.model` y `agents.defaults.models`) se analizan dividiendo en la **primera** `/`.

-   Usa `proveedor/modelo` al configurar modelos.
-   Si el ID del modelo en sí contiene `/` (estilo OpenRouter), incluye el prefijo del proveedor (ejemplo: `openrouter/moonshotai/kimi-k2`).
-   Si omites el proveedor, OpenClaw trata la entrada como un alias o un modelo para el **proveedor predeterminado** (solo funciona cuando no hay `/` en el ID del modelo).

## Configuración (mínima)

Como mínimo, establece:

-   `agents.defaults.workspace`
-   `channels.whatsapp.allowFrom` (muy recomendado)

* * *

*Siguiente: [Chats Grupales](../channels/group-messages.md)* 🦞

[Arquitectura de Gateway](./architecture.md)[Ciclo del Agente](./agent-loop.md)

---