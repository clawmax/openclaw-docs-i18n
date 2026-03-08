

  Configuración

  
# Mensajes de Grupo

Objetivo: permitir que Clawd esté en grupos de WhatsApp, se active solo cuando se le mencione y mantenga ese hilo separado de la sesión de mensajes directos personales. Nota: `agents.list[].groupChat.mentionPatterns` ahora también lo usan Telegram/Discord/Slack/iMessage; este documento se centra en el comportamiento específico de WhatsApp. Para configuraciones multi-agente, establece `agents.list[].groupChat.mentionPatterns` por agente (o usa `messages.groupChat.mentionPatterns` como valor global de respaldo).

## Lo que está implementado (2025-12-03)

-   Modos de activación: `mention` (predeterminado) o `always`. `mention` requiere una mención (menciones reales de WhatsApp @ a través de `mentionedJids`, patrones regex, o el E.164 del bot en cualquier parte del texto). `always` activa al agente en cada mensaje, pero solo debe responder cuando pueda aportar valor significativo; de lo contrario, devuelve el token silencioso `NO_REPLY`. Los valores predeterminados se pueden establecer en la configuración (`channels.whatsapp.groups`) y anular por grupo mediante `/activation`. Cuando `channels.whatsapp.groups` está configurado, también actúa como una lista de permitidos de grupos (incluye `"*"` para permitir todos).
-   Política de grupo: `channels.whatsapp.groupPolicy` controla si se aceptan mensajes de grupo (`open|disabled|allowlist`). `allowlist` usa `channels.whatsapp.groupAllowFrom` (respaldo: `channels.whatsapp.allowFrom` explícito). El valor predeterminado es `allowlist` (bloqueado hasta que agregues remitentes).
-   Sesiones por grupo: las claves de sesión tienen el formato `agent::whatsapp:group:`, por lo que comandos como `/verbose on` o `/think high` (enviados como mensajes independientes) se limitan a ese grupo; el estado de los mensajes directos personales no se ve afectado. Los latidos del corazón se omiten para los hilos de grupo.
-   Inyección de contexto: los mensajes de grupo **solo pendientes** (predeterminado 50) que *no* desencadenaron una ejecución se prefijan con `[Mensajes del chat desde tu última respuesta - para contexto]`, y la línea desencadenante con `[Mensaje actual - responde a esto]`. Los mensajes que ya están en la sesión no se reinyectan.
-   Visualización del remitente: cada lote de grupo ahora termina con `[de: Nombre del Remitente (+E164)]` para que Pi sepa quién está hablando.
-   Efímeros/ver-una-vez: los desempaquetamos antes de extraer texto/menciones, por lo que las menciones dentro de ellos aún activan al agente.
-   Prompt del sistema para grupos: en el primer turno de una sesión de grupo (y cada vez que `/activation` cambie el modo) inyectamos una breve nota en el prompt del sistema como `Estás respondiendo dentro del grupo de WhatsApp "". Miembros del grupo: Alice (+44...), Bob (+43...), … Activación: solo por mención … Dirígete al remitente específico indicado en el contexto del mensaje.` Si los metadatos no están disponibles, aún le decimos al agente que es un chat grupal.

## Ejemplo de configuración (WhatsApp)

Añade un bloque `groupChat` a `~/.openclaw/openclaw.json` para que las menciones por nombre de visualización funcionen incluso cuando WhatsApp elimine el `@` visual en el cuerpo del texto:

```json
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: ["@?openclaw", "\\+?15555550123"],
        },
      },
    ],
  },
}
```

Notas:

-   Las expresiones regulares no distinguen entre mayúsculas y minúsculas; cubren una mención por nombre de visualización como `@openclaw` y el número crudo con o sin `+`/espacios.
-   WhatsApp aún envía menciones canónicas a través de `mentionedJids` cuando alguien toca el contacto, por lo que el respaldo del número rara vez es necesario pero es una red de seguridad útil.

### Comando de activación (solo para el propietario)

Usa el comando de chat grupal:

-   `/activation mention`
-   `/activation always`

Solo el número del propietario (de `channels.whatsapp.allowFrom`, o el propio E.164 del bot cuando no esté configurado) puede cambiar esto. Envía `/status` como un mensaje independiente en el grupo para ver el modo de activación actual.

## Cómo usarlo

1.  Añade tu cuenta de WhatsApp (la que ejecuta OpenClaw) al grupo.
2.  Di `@openclaw …` (o incluye el número). Solo los remitentes en la lista de permitidos pueden activarlo a menos que establezcas `groupPolicy: "open"`.
3.  El prompt del agente incluirá el contexto reciente del grupo más el marcador final `[de: …]` para que pueda dirigirse a la persona correcta.
4.  Las directivas a nivel de sesión (`/verbose on`, `/think high`, `/new` o `/reset`, `/compact`) se aplican solo a la sesión de ese grupo; envíalas como mensajes independientes para que se registren. Tu sesión de mensajes directos personales permanece independiente.

## Pruebas / verificación

-   Prueba manual:
    -   Envía una mención `@openclaw` en el grupo y confirma una respuesta que haga referencia al nombre del remitente.
    -   Envía una segunda mención y verifica que el bloque de historial se incluya y luego se borre en el siguiente turno.
-   Revisa los registros de la pasarela (ejecuta con `--verbose`) para ver las entradas `inbound web message` que muestren `from: ` y el sufijo `[de: …]`.

## Consideraciones conocidas

-   Los latidos del corazón se omiten intencionalmente para grupos para evitar transmisiones ruidosas.
-   La supresión de eco usa la cadena combinada del lote; si envías texto idéntico dos veces sin menciones, solo el primero obtendrá una respuesta.
-   Las entradas del almacén de sesiones aparecerán como `agent::whatsapp:group:` en el almacén de sesiones (`~/.openclaw/agents//sessions/sessions.json` por defecto); una entrada faltante solo significa que el grupo aún no ha desencadenado una ejecución.
-   Los indicadores de escritura en grupos siguen `agents.defaults.typingMode` (predeterminado: `message` cuando no se menciona).

[Emparejamiento](./pairing.md)[Grupos](./groups.md)