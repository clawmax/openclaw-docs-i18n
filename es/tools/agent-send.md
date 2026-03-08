title: "Comando Agent Send para Coordinación de Agentes OpenClaw"
description: "Aprende a usar el comando openclaw agent send para ejecutar un solo turno de agente, gestionar sesiones y entregar respuestas a varios canales."
keywords: ["agente openclaw", "coordinación de agentes", "comando cli", "agent send", "gestión de sesiones", "entrega de mensajes", "tiempo de ejecución embebido"]
---

  Coordinación de agentes

  
# Agent Send

`openclaw agent` ejecuta un solo turno de agente sin necesidad de un mensaje de chat entrante. Por defecto pasa **a través del Gateway**; añade `--local` para forzar el tiempo de ejecución embebido en la máquina actual.

## Comportamiento

-   Requerido: `--message `
-   Selección de sesión:
    -   `--to ` deriva la clave de sesión (los destinos de grupo/canal preservan el aislamiento; los chats directos se colapsan a `main`), **o**
    -   `--session-id ` reutiliza una sesión existente por id, **o**
    -   `--agent ` apunta directamente a un agente configurado (usa la clave de sesión `main` de ese agente)
-   Ejecuta el mismo tiempo de ejecución de agente embebido que las respuestas entrantes normales.
-   Las banderas de pensamiento/verbose persisten en el almacén de sesiones.
-   Salida:
    -   por defecto: imprime el texto de respuesta (más líneas `MEDIA:`)
    -   `--json`: imprime la carga útil estructurada + metadatos
-   Entrega opcional de vuelta a un canal con `--deliver` + `--channel` (los formatos de destino coinciden con `openclaw message --target`).
-   Usa `--reply-channel`/`--reply-to`/`--reply-account` para anular la entrega sin cambiar la sesión.

Si el Gateway es inalcanzable, la CLI **recurre** a la ejecución local embebida.

## Ejemplos

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## Banderas

-   `--local`: ejecutar localmente (requiere claves API del proveedor del modelo en tu shell)
-   `--deliver`: enviar la respuesta al canal elegido
-   `--channel`: canal de entrega (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, por defecto: `whatsapp`)
-   `--reply-to`: anulación del destino de entrega
-   `--reply-channel`: anulación del canal de entrega
-   `--reply-account`: anulación del id de cuenta de entrega
-   `--thinking <off|minimal|low|medium|high|xhigh>`: persistir nivel de pensamiento (solo modelos GPT-5.2 + Codex)
-   `--verbose <on|full|off>`: persistir nivel de verbose
-   `--timeout `: anular el tiempo de espera del agente
-   `--json`: salida JSON estructurada

[Solución de Problemas del Navegador](./browser-linux-troubleshooting.md)[Sub-Agentes](./subagents.md)

---