

  Mantenimiento

  
# Actualizando

OpenClaw se mueve rápido (pre "1.0"). Trata las actualizaciones como enviar infraestructura: actualizar → ejecutar verificaciones → reiniciar (o usar `openclaw update`, que reinicia) → verificar.

## Recomendado: volver a ejecutar el instalador del sitio web (actualización en el lugar)

La ruta de actualización **preferida** es volver a ejecutar el instalador desde el sitio web. Detecta instalaciones existentes, actualiza en el lugar y ejecuta `openclaw doctor` cuando es necesario.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Notas:

-   Añade `--no-onboard` si no quieres que el asistente de configuración inicial se ejecute de nuevo.
-   Para **instalaciones desde fuente**, usa:
    
    Copiar
    
    ```bash
    curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --no-onboard
    ```
    
    El instalador hará `git pull --rebase` **solo** si el repositorio está limpio.
-   Para **instalaciones globales**, el script usa `npm install -g openclaw@latest` internamente.
-   Nota de compatibilidad: `clawdbot` permanece disponible como un envoltorio de compatibilidad.

## Antes de actualizar

-   Saber cómo instalaste: **global** (npm/pnpm) vs **desde fuente** (git clone).
-   Saber cómo se ejecuta tu Gateway: **terminal en primer plano** vs **servicio supervisado** (launchd/systemd).
-   Haz una instantánea de tu personalización:
    -   Configuración: `~/.openclaw/openclaw.json`
    -   Credenciales: `~/.openclaw/credentials/`
    -   Espacio de trabajo: `~/.openclaw/workspace`

## Actualizar (instalación global)

Instalación global (elige una):

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

**No** recomendamos Bun para el tiempo de ejecución del Gateway (errores en WhatsApp/Telegram). Para cambiar canales de actualización (instalaciones git + npm):

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

Usa `--tag <dist-tag|version>` para una etiqueta/versión de instalación puntual. Consulta [Canales de desarrollo](./development-channels.md) para la semántica de canales y notas de la versión. Nota: en instalaciones npm, el gateway registra una sugerencia de actualización al iniciar (verifica la etiqueta del canal actual). Desactívalo con `update.checkOnStart: false`.

### Auto-actualizador del núcleo (opcional)

El auto-actualizador está **desactivado por defecto** y es una característica central del Gateway (no un plugin).

```json
{
  "update": {
    "channel": "stable",
    "auto": {
      "enabled": true,
      "stableDelayHours": 6,
      "stableJitterHours": 12,
      "betaCheckIntervalHours": 1
    }
  }
}
```

Comportamiento:

-   `stable`: cuando se detecta una nueva versión, OpenClaw espera `stableDelayHours` y luego aplica un jitter determinístico por instalación en `stableJitterHours` (despliegue escalonado).
-   `beta`: verifica en el intervalo `betaCheckIntervalHours` (por defecto: cada hora) y aplica cuando hay una actualización disponible.
-   `dev`: no aplica automáticamente; usa `openclaw update` manual.

Usa `openclaw update --dry-run` para previsualizar las acciones de actualización antes de habilitar la automatización. Luego:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Notas:

-   Si tu Gateway se ejecuta como un servicio, `openclaw gateway restart` es preferible a matar PIDs.
-   Si estás fijado a una versión específica, consulta "Revertir / fijar versión" más abajo.

## Actualizar (openclaw update)

Para **instalaciones desde fuente** (checkout de git), prefiere:

```bash
openclaw update
```

Ejecuta un flujo de actualización seguro:

-   Requiere un árbol de trabajo limpio.
-   Cambia al canal seleccionado (etiqueta o rama).
-   Obtiene y reubica (rebase) contra el upstream configurado (canal dev).
-   Instala dependencias, construye, construye la Interfaz de Control y ejecuta `openclaw doctor`.
-   Reinicia el gateway por defecto (usa `--no-restart` para omitir).

Si instalaste vía **npm/pnpm** (sin metadatos de git), `openclaw update` intentará actualizar a través de tu gestor de paquetes. Si no puede detectar la instalación, usa "Actualizar (instalación global)" en su lugar.

## Actualizar (Interfaz de Control / RPC)

La Interfaz de Control tiene **Actualizar y Reiniciar** (RPC: `update.run`). Esto:

1.  Ejecuta el mismo flujo de actualización desde fuente que `openclaw update` (solo checkout de git).
2.  Escribe un centinela de reinicio con un informe estructurado (cola de stdout/stderr).
3.  Reinicia el gateway y notifica a la última sesión activa con el informe.

Si la reubicación (rebase) falla, el gateway aborta y se reinicia sin aplicar la actualización.

## Actualizar (desde fuente)

Desde el checkout del repositorio: Preferido:

```bash
openclaw update
```

Manual (equivalente):

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # instala automáticamente las dependencias de la UI en la primera ejecución
openclaw doctor
openclaw health
```

Notas:

-   `pnpm build` es importante cuando ejecutas el binario empaquetado `openclaw` ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) o usas Node para ejecutar `dist/`.
-   Si ejecutas desde un checkout del repositorio sin una instalación global, usa `pnpm openclaw ...` para comandos CLI.
-   Si ejecutas directamente desde TypeScript (`pnpm openclaw ...`), una reconstrucción generalmente no es necesaria, pero **las migraciones de configuración aún aplican** → ejecuta doctor.
-   Cambiar entre instalaciones globales y de git es fácil: instala la otra variante, luego ejecuta `openclaw doctor` para que el punto de entrada del servicio del gateway se reescriba a la instalación actual.

## Siempre Ejecuta: openclaw doctor

Doctor es el comando de "actualización segura". Es intencionalmente simple: reparar + migrar + advertir. Nota: si estás en una **instalación desde fuente** (checkout de git), `openclaw doctor` ofrecerá ejecutar `openclaw update` primero. Cosas típicas que hace:

-   Migrar claves de configuración obsoletas / ubicaciones de archivos de configuración heredados.
-   Auditar políticas de DM y advertir sobre configuraciones "abiertas" riesgosas.
-   Verificar la salud del Gateway y puede ofrecer reiniciarlo.
-   Detectar y migrar servicios de gateway más antiguos (launchd/systemd; schtasks heredados) a los servicios actuales de OpenClaw.
-   En Linux, asegurar la persistencia del usuario en systemd (para que el Gateway sobreviva al cierre de sesión).

Detalles: [Doctor](../gateway/doctor.md)

## Iniciar / detener / reiniciar el Gateway

CLI (funciona independientemente del SO):

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

Si estás supervisado:

-   macOS launchd (LaunchAgent incluido en la app): `launchctl kickstart -k gui/$UID/ai.openclaw.gateway` (usa `ai.openclaw.`; el heredado `com.openclaw.*` aún funciona)
-   Servicio de usuario systemd en Linux: `systemctl --user restart openclaw-gateway[-].service`
-   Windows (WSL2): `systemctl --user restart openclaw-gateway[-].service`
    -   `launchctl`/`systemctl` solo funcionan si el servicio está instalado; de lo contrario ejecuta `openclaw gateway install`.

Runbook + etiquetas exactas del servicio: [Runbook del Gateway](../gateway.md)

## Revertir / fijar versión (cuando algo falla)

### Fijar versión (instalación global)

Instala una versión conocida como buena (reemplaza `` con la última que funcionó):

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

Consejo: para ver la versión publicada actual, ejecuta `npm view openclaw version`. Luego reinicia y vuelve a ejecutar doctor:

```bash
openclaw doctor
openclaw gateway restart
```

### Fijar versión (desde fuente) por fecha

Elige un commit de una fecha (ejemplo: "estado de main al 2026-01-01"):

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

Luego reinstala dependencias + reinicia:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

Si quieres volver a la última versión más tarde:

```bash
git checkout main
git pull
```

## Si estás atascado

-   Ejecuta `openclaw doctor` de nuevo y lee la salida cuidadosamente (a menudo te dice la solución).
-   Consulta: [Solución de problemas](../gateway/troubleshooting.md)
-   Pregunta en Discord: [https://discord.gg/clawd](https://discord.gg/clawd)

[Bun (Experimental)](./bun.md)[Guía de Migración](./migrating.md)