

  Configuración para desarrolladores

  
# Flujo de Trabajo de Desarrollo Pi

Esta guía resume un flujo de trabajo sensato para trabajar en la integración de pi en OpenClaw.

## Verificación de Tipos y Linting

-   Verificación de tipos y construcción: `pnpm build`
-   Lint: `pnpm lint`
-   Verificación de formato: `pnpm format`
-   Verificación completa antes de subir cambios: `pnpm lint && pnpm build && pnpm test`

## Ejecución de Pruebas Pi

Ejecuta el conjunto de pruebas enfocado en Pi directamente con Vitest:

```bash
pnpm test -- \
  "src/agents/pi-*.test.ts" \
  "src/agents/pi-embedded-*.test.ts" \
  "src/agents/pi-tools*.test.ts" \
  "src/agents/pi-settings.test.ts" \
  "src/agents/pi-tool-definition-adapter*.test.ts" \
  "src/agents/pi-extensions/**/*.test.ts"
```

Para incluir el ejercicio del proveedor en vivo:

```
OPENCLAW_LIVE_TEST=1 pnpm test -- src/agents/pi-embedded-runner-extraparams.live.test.ts
```

Esto cubre las principales suites unitarias de Pi:

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-tool-definition-adapter.test.ts`
-   `src/agents/pi-extensions/*.test.ts`

## Pruebas Manuales

Flujo recomendado:

-   Ejecutar la pasarela en modo desarrollo:
    -   `pnpm gateway:dev`
-   Activar el agente directamente:
    -   `pnpm openclaw agent --message "Hello" --thinking low`
-   Usar la TUI para depuración interactiva:
    -   `pnpm tui`

Para el comportamiento de llamadas a herramientas, solicita una acción `read` o `exec` para poder ver el streaming de herramientas y el manejo de payloads.

## Restablecimiento a Estado Inicial

El estado reside en el directorio de estado de OpenClaw. Por defecto es `~/.openclaw`. Si `OPENCLAW_STATE_DIR` está configurado, usa ese directorio en su lugar. Para restablecer todo:

-   `openclaw.json` para la configuración
-   `credentials/` para perfiles de autenticación y tokens
-   `agents//sessions/` para el historial de sesiones del agente
-   `agents//sessions.json` para el índice de sesiones
-   `sessions/` si existen rutas heredadas
-   `workspace/` si quieres un espacio de trabajo en blanco

Si solo quieres restablecer las sesiones, elimina `agents//sessions/` y `agents//sessions.json` para ese agente. Conserva `credentials/` si no quieres volver a autenticarte.

## Referencias

-   [https://docs.openclaw.ai/testing](https://docs.openclaw.ai/testing)
-   [https://docs.openclaw.ai/start/getting-started](https://docs.openclaw.ai/start/getting-started)

[Configuración](./start/setup.md)[Canalización de CI](./ci.md)

---