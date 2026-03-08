

  Comandos CLI

  
# status

Diagnósticos para canales + sesiones.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Notas:

-   `--deep` ejecuta sondeos en vivo (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
-   La salida incluye almacenes de sesión por agente cuando se configuran múltiples agentes.
-   La visión general incluye el estado de instalación/ejecución del servicio Gateway + host del nodo cuando está disponible.
-   La visión general incluye el canal de actualización + SHA de git (para instalaciones desde código fuente).
-   La información de actualización aparece en la Visión General; si hay una actualización disponible, status imprime una sugerencia para ejecutar `openclaw update` (ver [Actualizando](../install/updating.md)).
-   Las superficies de estado de solo lectura (`status`, `status --json`, `status --all`) resuelven los SecretRefs admitidos para sus rutas de configuración objetivo cuando es posible.
-   Si se configura un SecretRef de canal admitido pero no está disponible en la ruta del comando actual, el estado permanece de solo lectura y reporta una salida degradada en lugar de fallar. La salida humana muestra advertencias como "token configurado no disponible en esta ruta de comando", y la salida JSON incluye `secretDiagnostics`.

[skills](./skills.md)[system](./system.md)

---