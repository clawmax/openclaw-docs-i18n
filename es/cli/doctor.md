

  Comandos CLI

  
# doctor

Comprobaciones de salud + soluciones rápidas para la puerta de enlace y los canales. Relacionado:

-   Solución de problemas: [Solución de problemas](../gateway/troubleshooting.md)
-   Auditoría de seguridad: [Seguridad](../gateway/security.md)

## Ejemplos

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Notas:

-   Los mensajes interactivos (como correcciones de llavero/OAuth) solo se ejecutan cuando stdin es un TTY y `--non-interactive` **no** está establecido. Las ejecuciones sin interfaz (cron, Telegram, sin terminal) omitirán los mensajes.
-   `--fix` (alias de `--repair`) escribe una copia de seguridad en `~/.openclaw/openclaw.json.bak` y elimina las claves de configuración desconocidas, listando cada eliminación.
-   Las comprobaciones de integridad del estado ahora detectan archivos de transcripción huérfanos en el directorio de sesiones y pueden archivarlos como `.deleted.` para recuperar espacio de forma segura.
-   Doctor incluye una comprobación de preparación de búsqueda en memoria y puede recomendar `openclaw configure --section model` cuando faltan credenciales de incrustación.
-   Si el modo sandbox está habilitado pero Docker no está disponible, doctor reporta una advertencia de alta señal con remediación (`instalar Docker` o `openclaw config set agents.defaults.sandbox.mode off`).

## macOS: anulaciones de entorno de launchctl

Si anteriormente ejecutaste `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (o `...PASSWORD`), ese valor anula tu archivo de configuración y puede causar errores persistentes de "no autorizado".

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```

[documentación](./docs.md)[puerta de enlace](./gateway.md)