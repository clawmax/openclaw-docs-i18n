title: "Guía y Ejemplos del Comando Agente de la CLI de OpenClaw"
description: "Aprende a ejecutar un turno de agente mediante la CLI de OpenClaw. Usa ejemplos para enviar mensajes, dirigirte a agentes específicos y gestionar opciones de entrega."
keywords: ["openclaw cli", "comando agente", "cli agente", "ejecutar agente", "ejemplos de agente", "enviar agente", "agente gateway", "herramientas cli"]
---

  Comandos CLI

  
# agent

Ejecuta un turno de agente a través del Gateway (usa `--local` para el modo embebido). Usa `--agent ` para dirigirte directamente a un agente configurado. Relacionado:

-   Herramienta de envío de agente: [Envío de agente](../tools/agent-send.md)

## Ejemplos

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## Notas

-   Cuando este comando activa la regeneración de `models.json`, las credenciales de proveedor gestionadas por SecretRef se conservan como marcadores no secretos (por ejemplo, nombres de variables de entorno o `secretref-managed`), no como texto plano de secretos resueltos.

[acp](./acp.md)[agents](./agents.md)

---