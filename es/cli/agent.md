

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