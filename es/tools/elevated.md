

  Herramientas integradas

  
# Modo Elevado

## Qué hace

-   `/elevated on` se ejecuta en el host de puerta de enlace y mantiene las aprobaciones de exec (igual que `/elevated ask`).
-   `/elevated full` se ejecuta en el host de puerta de enlace **y** aprueba automáticamente exec (omite las aprobaciones de exec).
-   `/elevated ask` se ejecuta en el host de puerta de enlace pero mantiene las aprobaciones de exec (igual que `/elevated on`).
-   `on`/`ask` **no** fuerzan `exec.security=full`; la política de seguridad/ask configurada aún se aplica.
-   Solo cambia el comportamiento cuando el agente está **en sandbox** (de lo contrario, exec ya se ejecuta en el host).
-   Formas de directiva: `/elevated on|off|ask|full`, `/elev on|off|ask|full`.
-   Solo se aceptan `on|off|ask|full`; cualquier otra cosa devuelve una pista y no cambia el estado.

## Qué controla (y qué no)

-   **Puertas de disponibilidad**: `tools.elevated` es la línea base global. `agents.list[].tools.elevated` puede restringir aún más el modo elevado por agente (ambos deben permitirlo).
-   **Estado por sesión**: `/elevated on|off|ask|full` establece el nivel elevado para la clave de sesión actual.
-   **Directiva en línea**: `/elevated on|ask|full` dentro de un mensaje se aplica solo a ese mensaje.
-   **Grupos**: En chats grupales, las directivas elevadas solo se respetan cuando se menciona al agente. Los mensajes solo de comando que omiten los requisitos de mención se tratan como mencionados.
-   **Ejecución en host**: elevated fuerza que `exec` se ejecute en el host de puerta de enlace; `full` también establece `security=full`.
-   **Aprobaciones**: `full` omite las aprobaciones de exec; `on`/`ask` las respetan cuando las reglas de lista de permitidos/ask lo requieren.
-   **Agentes no en sandbox**: no tiene efecto en la ubicación; solo afecta el control de acceso, el registro y el estado.
-   **La política de herramientas aún se aplica**: si `exec` es denegado por la política de herramientas, no se puede usar el modo elevado.
-   **Separado de `/exec`**: `/exec` ajusta los valores predeterminados por sesión para remitentes autorizados y no requiere modo elevado.

## Orden de resolución

1.  Directiva en línea en el mensaje (se aplica solo a ese mensaje).
2.  Sobrescritura de sesión (establecida enviando un mensaje solo de directiva).
3.  Valor predeterminado global (`agents.defaults.elevatedDefault` en la configuración).

## Establecer un valor predeterminado de sesión

-   Envía un mensaje que sea **solo** la directiva (se permite espacio en blanco), por ejemplo, `/elevated full`.
-   Se envía una respuesta de confirmación (`Modo elevado establecido en full...` / `Modo elevado desactivado.`).
-   Si el acceso elevado está deshabilitado o el remitente no está en la lista de permitidos aprobada, la directiva responde con un error procesable y no cambia el estado de la sesión.
-   Envía `/elevated` (o `/elevated:`) sin argumento para ver el nivel elevado actual.

## Disponibilidad + listas de permitidos

-   Puerta de función: `tools.elevated.enabled` (el valor predeterminado puede estar desactivado mediante configuración incluso si el código lo admite).
-   Lista de remitentes permitidos: `tools.elevated.allowFrom` con listas de permitidos por proveedor (por ejemplo, `discord`, `whatsapp`).
-   Las entradas de lista de permitidos sin prefijo coinciden solo con valores de identidad con alcance de remitente (`SenderId`, `SenderE164`, `From`); los campos de enrutamiento de destinatario nunca se usan para autorización elevada.
-   Los metadatos mutables del remitente requieren prefijos explícitos:
    -   `name:` coincide con `SenderName`
    -   `username:` coincide con `SenderUsername`
    -   `tag:` coincide con `SenderTag`
    -   `id:`, `from:`, `e164:` están disponibles para el direccionamiento explícito de identidad
-   Puerta por agente: `agents.list[].tools.elevated.enabled` (opcional; solo puede restringir más).
-   Lista de permitidos por agente: `agents.list[].tools.elevated.allowFrom` (opcional; cuando se establece, el remitente debe coincidir con **ambas** listas de permitidos global y por agente).
-   Respaldo de Discord: si se omite `tools.elevated.allowFrom.discord`, se usa la lista `channels.discord.allowFrom` como respaldo (heredado: `channels.discord.dm.allowFrom`). Establece `tools.elevated.allowFrom.discord` (incluso `[]`) para anular. Las listas de permitidos por agente **no** usan el respaldo.
-   Todas las puertas deben pasar; de lo contrario, el modo elevado se trata como no disponible.

## Registro + estado

-   Las llamadas exec elevadas se registran a nivel de información.
-   El estado de la sesión incluye el modo elevado (por ejemplo, `elevated=ask`, `elevated=full`).

[Herramienta PDF](./pdf.md)[Herramienta Exec](./exec.md)