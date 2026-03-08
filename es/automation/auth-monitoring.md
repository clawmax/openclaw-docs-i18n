

  Automatización

  
# Monitoreo de Autenticación

OpenClaw expone el estado de salud de la caducidad OAuth a través de `openclaw models status`. Úsalo para automatización y alertas; los scripts son extras opcionales para flujos de trabajo en teléfono.

## Preferido: Comprobación CLI (portable)

```bash
openclaw models status --check
```

Códigos de salida:

-   `0`: OK
-   `1`: credenciales caducadas o faltantes
-   `2`: caducando pronto (dentro de 24h)

Esto funciona en cron/systemd y no requiere scripts adicionales.

## Scripts opcionales (flujos de trabajo para operaciones / teléfono)

Estos se encuentran en `scripts/` y son **opcionales**. Asumen acceso SSH al host de la puerta de enlace y están ajustados para systemd + Termux.

-   `scripts/claude-auth-status.sh` ahora usa `openclaw models status --json` como la fuente de verdad (recurriendo a lecturas directas de archivo si la CLI no está disponible), así que mantén `openclaw` en el `PATH` para los temporizadores.
-   `scripts/auth-monitor.sh`: objetivo del temporizador cron/systemd; envía alertas (ntfy o teléfono).
-   `scripts/systemd/openclaw-auth-monitor.{service,timer}`: temporizador de usuario de systemd.
-   `scripts/claude-auth-status.sh`: comprobador de autenticación de Claude Code + OpenClaw (completo/json/simple).
-   `scripts/mobile-reauth.sh`: flujo de re-autenticación guiado por SSH.
-   `scripts/termux-quick-auth.sh`: widget de un toque para estado + abrir URL de autenticación.
-   `scripts/termux-auth-widget.sh`: flujo completo guiado por widget.
-   `scripts/termux-sync-widget.sh`: sincronizar credenciales de Claude Code → OpenClaw.

Si no necesitas automatización en el teléfono o temporizadores de systemd, omite estos scripts.

[Encuestas](./poll.md)[Nodos](../nodes.md)