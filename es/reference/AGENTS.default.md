

  Plantillas

  
# AGENTS.md predeterminado

## Primera ejecución (recomendado)

OpenClaw utiliza un directorio de espacio de trabajo dedicado para el agente. Predeterminado: `~/.openclaw/workspace` (configurable mediante `agents.defaults.workspace`).

1.  Crea el espacio de trabajo (si aún no existe):

```bash
mkdir -p ~/.openclaw/workspace
```

2.  Copia las plantillas predeterminadas del espacio de trabajo:

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3.  Opcional: si quieres el listado de habilidades del asistente personal, reemplaza AGENTS.md con este archivo:

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4.  Opcional: elige un espacio de trabajo diferente configurando `agents.defaults.workspace` (admite `~`):

```json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

## Valores de seguridad por defecto

-   No vuelques directorios o secretos en el chat.
-   No ejecutes comandos destructivos a menos que se te pida explícitamente.
-   No envíes respuestas parciales/en flujo a superficies de mensajería externas (solo respuestas finales).

## Inicio de sesión (requerido)

-   Lee `SOUL.md`, `USER.md`, `memory.md`, y los archivos de hoy+ayer en `memory/`.
-   Hazlo antes de responder.

## Alma (requerido)

-   `SOUL.md` define la identidad, el tono y los límites. Mantenlo actualizado.
-   Si cambias `SOUL.md`, informa al usuario.
-   Eres una instancia nueva cada sesión; la continuidad reside en estos archivos.

## Espacios compartidos (recomendado)

-   No eres la voz del usuario; ten cuidado en chats grupales o canales públicos.
-   No compartas datos privados, información de contacto o notas internas.

## Sistema de memoria (recomendado)

-   Registro diario: `memory/YYYY-MM-DD.md` (crea `memory/` si es necesario).
-   Memoria a largo plazo: `memory.md` para hechos duraderos, preferencias y decisiones.
-   Al inicio de sesión, lee hoy + ayer + `memory.md` si está presente.
-   Captura: decisiones, preferencias, restricciones, asuntos pendientes.
-   Evita secretos a menos que se soliciten explícitamente.

## Herramientas y habilidades

-   Las herramientas viven en habilidades; sigue el `SKILL.md` de cada habilidad cuando la necesites.
-   Mantén notas específicas del entorno en `TOOLS.md` (Notas para Habilidades).

## Consejo de respaldo (recomendado)

Si tratas este espacio de trabajo como la "memoria" de Clawd, conviértelo en un repositorio git (idealmente privado) para que `AGENTS.md` y tus archivos de memoria estén respaldados.

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Add Clawd workspace"
# Opcional: añade un repositorio remoto privado + push
```

## Qué hace OpenClaw

-   Ejecuta la pasarela de WhatsApp + el agente de codificación Pi para que el asistente pueda leer/escribir chats, obtener contexto y ejecutar habilidades a través del Mac anfitrión.
-   La aplicación de macOS gestiona los permisos (grabación de pantalla, notificaciones, micrófono) y expone la CLI `openclaw` a través de su binario incluido.
-   Los chats directos se colapsan en la sesión `main` del agente por defecto; los grupos permanecen aislados como `agent:::group:` (salas/canales: `agent:::channel:`); los latidos mantienen vivas las tareas en segundo plano.

## Habilidades principales (habilitar en Configuración → Habilidades)

-   **mcporter** — Runtime/CLI del servidor de herramientas para gestionar backends de habilidades externas.
-   **Peekaboo** — Capturas de pantalla rápidas de macOS con análisis de visión por IA opcional.
-   **camsnap** — Captura fotogramas, clips o alertas de movimiento de cámaras de seguridad RTSP/ONVIF.
-   **oracle** — CLI de agente listo para OpenAI con reproducción de sesión y control del navegador.
-   **eightctl** — Controla tu sueño, desde la terminal.
-   **imsg** — Envía, lee, transmite iMessage y SMS.
-   **wacli** — WhatsApp CLI: sincroniza, busca, envía.
-   **discord** — Acciones de Discord: reaccionar, stickers, encuestas. Usa destinos `user:` o `channel:` (los IDs numéricos simples son ambiguos).
-   **gog** — CLI de Google Suite: Gmail, Calendar, Drive, Contacts.
-   **spotify-player** — Cliente de Spotify en terminal para buscar/encolar/controlar la reproducción.
-   **sag** — Voz de ElevenLabs con UX tipo `say` de mac; transmite a altavoces por defecto.
-   **Sonos CLI** — Controla altavoces Sonos (descubrir/estado/reproducción/volumen/agrupación) desde scripts.
-   **blucli** — Reproduce, agrupa y automatiza reproductores BluOS desde scripts.
-   **OpenHue CLI** — Control de iluminación Philips Hue para escenas y automatizaciones.
-   **OpenAI Whisper** — Voz a texto local para dictados rápidos y transcripciones de buzón de voz.
-   **Gemini CLI** — Modelos Google Gemini desde la terminal para preguntas y respuestas rápidas.
-   **agent-tools** — Kit de herramientas de utilidad para automatizaciones y scripts auxiliares.

## Notas de uso

-   Prefiere la CLI `openclaw` para scripting; la aplicación de mac maneja los permisos.
-   Ejecuta las instalaciones desde la pestaña Habilidades; oculta el botón si un binario ya está presente.
-   Mantén los latidos habilitados para que el asistente pueda programar recordatorios, monitorear bandejas de entrada y activar capturas de cámara.
-   La interfaz de usuario Canvas se ejecuta a pantalla completa con superposiciones nativas. Evita colocar controles críticos en los bordes superior-izquierdo/superior-derecho/inferior; añade márgenes explícitos en el diseño y no dependas de los márgenes de área segura.
-   Para verificación impulsada por navegador, usa `openclaw browser` (pestañas/estado/captura de pantalla) con el perfil de Chrome gestionado por OpenClaw.
-   Para inspección del DOM, usa `openclaw browser eval|query|dom|snapshot` (y `--json`/`--out` cuando necesites salida de máquina).
-   Para interacciones, usa `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run` (click/type requieren referencias de snapshot; usa `evaluate` para selectores CSS).

[Base de datos de modelos de dispositivos](./device-models.md)[Plantilla AGENTS.md](./templates/AGENTS.md)

---