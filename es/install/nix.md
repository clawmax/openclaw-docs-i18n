

  Otros métodos de instalación

  
# Nix

La forma recomendada de ejecutar OpenClaw con Nix es mediante **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — un módulo de Home Manager con todo incluido.

## Inicio Rápido

Pega esto en tu agente de IA (Claude, Cursor, etc.):

```
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, model provider API key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 Guía completa: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)** El repositorio nix-openclaw es la fuente de verdad para la instalación con Nix. Esta página es solo un resumen rápido.

## Lo que obtienes

-   Gateway + aplicación macOS + herramientas (whisper, spotify, cámaras) — todo fijado
-   Servicio launchd que sobrevive a reinicios
-   Sistema de plugins con configuración declarativa
-   Reversión instantánea: `home-manager switch --rollback`

* * *

## Comportamiento en Tiempo de Ejecución del Modo Nix

Cuando `OPENCLAW_NIX_MODE=1` está establecido (automático con nix-openclaw): OpenClaw soporta un **modo Nix** que hace la configuración determinista y desactiva los flujos de auto-instalación. Actívalo exportando:

```
OPENCLAW_NIX_MODE=1
```

En macOS, la aplicación GUI no hereda automáticamente las variables de entorno del shell. También puedes activar el modo Nix vía defaults:

```bash
defaults write ai.openclaw.mac openclaw.nixMode -bool true
```

### Rutas de configuración + estado

OpenClaw lee la configuración JSON5 desde `OPENCLAW_CONFIG_PATH` y almacena datos mutables en `OPENCLAW_STATE_DIR`. Cuando sea necesario, también puedes establecer `OPENCLAW_HOME` para controlar el directorio base utilizado para la resolución de rutas internas.

-   `OPENCLAW_HOME` (precedencia por defecto: `HOME` / `USERPROFILE` / `os.homedir()`)
-   `OPENCLAW_STATE_DIR` (por defecto: `~/.openclaw`)
-   `OPENCLAW_CONFIG_PATH` (por defecto: `$OPENCLAW_STATE_DIR/openclaw.json`)

Cuando se ejecuta bajo Nix, establece estas explícitamente en ubicaciones gestionadas por Nix para que el estado de ejecución y la configuración permanezcan fuera del almacén inmutable.

### Comportamiento en tiempo de ejecución en modo Nix

-   Los flujos de auto-instalación y auto-mutación están desactivados
-   Las dependencias faltantes muestran mensajes de remediación específicos de Nix
-   La interfaz de usuario muestra un banner de solo lectura del modo Nix cuando está presente

## Nota sobre empaquetado (macOS)

El flujo de empaquetado de macOS espera una plantilla estable de Info.plist en:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) copia esta plantilla en el paquete de la aplicación y parchea campos dinámicos (ID del paquete, versión/build, SHA de Git, claves Sparkle). Esto mantiene el plist determinista para el empaquetado con SwiftPM y las compilaciones de Nix (que no dependen de un toolchain completo de Xcode).

## Relacionado

-   [nix-openclaw](https://github.com/openclaw/nix-openclaw) — guía de configuración completa
-   [Asistente](../start/wizard.md) — configuración CLI sin Nix
-   [Docker](./docker.md) — configuración en contenedores

[Podman](./podman.md)[Ansible](./ansible.md)

---