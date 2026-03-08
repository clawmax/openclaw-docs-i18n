title: "Uso y Ejemplos del Comando voicecall de OpenClaw CLI"
description: "Aprende a usar los comandos CLI de voicecall de OpenClaw para realizar, gestionar y finalizar llamadas de voz con IA, y exponer webhooks de forma segura mediante Tailscale."
keywords: ["openclaw voicecall", "comandos de voz cli", "llamada de voz con ia", "webhooks tailscale", "plugin voicecall", "comandos cli", "automatización de llamadas de voz", "exposición de webhooks"]
---

  Comandos CLI

  
# voicecall

`voicecall` es un comando proporcionado por un plugin. Solo aparece si el plugin de llamadas de voz está instalado y habilitado. Documentación principal:

-   Plugin de llamadas de voz: [Llamada de Voz](../plugins/voice-call.md)

## Comandos comunes

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## Exponer webhooks (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall expose --mode off
```

Nota de seguridad: solo exponga el endpoint del webhook a redes de confianza. Prefiera Tailscale Serve sobre Funnel cuando sea posible.

[actualizar](./update.md)[webhooks](./webhooks.md)

---