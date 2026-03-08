

  Configuración para desarrolladores

  
# Configuración

> **ℹ️** Si estás configurando por primera vez, comienza con [Primeros pasos](./getting-started.md). Para detalles del asistente, consulta [Asistente de incorporación](./wizard.md).

 Última actualización: 2026-01-01

## TL;DR

-   **La personalización vive fuera del repositorio:** `~/.openclaw/workspace` (espacio de trabajo) + `~/.openclaw/openclaw.json` (configuración).
-   **Flujo de trabajo estable:** instala la aplicación macOS; déjala ejecutar el Gateway incluido.
-   **Flujo de trabajo de vanguardia:** ejecuta el Gateway tú mismo mediante `pnpm gateway:watch`, luego deja que la aplicación macOS se conecte en modo Local.

## Requisitos previos (desde el código fuente)

-   Node `>=22`
-   `pnpm`
-   Docker (opcional; solo para configuración/e2e en contenedores — ver [Docker](../install/docker.md))

## Estrategia de personalización (para que las actualizaciones no afecten)

Si quieres algo "100% personalizado para mí" *y* actualizaciones fáciles, mantén tu personalización en:

-   **Configuración:** `~/.openclaw/openclaw.json` (JSON/JSON5-ish)
-   **Espacio de trabajo:** `~/.openclaw/workspace` (habilidades, prompts, recuerdos; hazlo un repositorio git privado)

Inicializa una vez:

```bash
openclaw setup
```

Desde dentro de este repositorio, usa la entrada CLI local:

```bash
openclaw setup
```

Si aún no tienes una instalación global, ejecútalo mediante `pnpm openclaw setup`.

## Ejecutar el Gateway desde este repositorio

Después de `pnpm build`, puedes ejecutar la CLI empaquetada directamente:

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## Flujo de trabajo estable (primero la aplicación macOS)

1.  Instala + inicia **OpenClaw.app** (barra de menú).
2.  Completa la lista de verificación de incorporación/permisos (solicitudes TCC).
3.  Asegúrate de que el Gateway esté en **Local** y ejecutándose (la aplicación lo gestiona).
4.  Vincula superficies (ejemplo: WhatsApp):

```bash
openclaw channels login
```

5.  Verificación de funcionamiento:

```bash
openclaw health
```

Si la incorporación no está disponible en tu compilación:

-   Ejecuta `openclaw setup`, luego `openclaw channels login`, y luego inicia el Gateway manualmente (`openclaw gateway`).

## Flujo de trabajo de vanguardia (Gateway en una terminal)

Objetivo: trabajar en el Gateway TypeScript, obtener recarga en caliente, mantener la interfaz de usuario de la aplicación macOS conectada.

### 0) (Opcional) Ejecutar también la aplicación macOS desde el código fuente

Si también quieres la aplicación macOS en la vanguardia:

```
./scripts/restart-mac.sh
```

### 1) Iniciar el Gateway de desarrollo

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` ejecuta el gateway en modo de observación y se recarga con los cambios de TypeScript.

### 2) Dirigir la aplicación macOS a tu Gateway en ejecución

En **OpenClaw.app**:

-   Modo de conexión: **Local** La aplicación se conectará al gateway en ejecución en el puerto configurado.

### 3) Verificar

-   El estado del Gateway en la aplicación debería mostrar **"Usando gateway existente …"**
-   O mediante la CLI:

```bash
openclaw health
```

### Errores comunes

-   **Puerto incorrecto:** El WS del Gateway usa por defecto `ws://127.0.0.1:18789`; mantén la aplicación y la CLI en el mismo puerto.
-   **Dónde vive el estado:**
    -   Credenciales: `~/.openclaw/credentials/`
    -   Sesiones: `~/.openclaw/agents//sessions/`
    -   Registros: `/tmp/openclaw/`

## Mapa de almacenamiento de credenciales

Usa esto al depurar autenticación o decidir qué respaldar:

-   **WhatsApp**: `~/.openclaw/credentials/whatsapp//creds.json`
-   **Token de bot de Telegram**: configuración/env o `channels.telegram.tokenFile`
-   **Token de bot de Discord**: configuración/env o SecretRef (proveedores env/file/exec)
-   **Tokens de Slack**: configuración/env (`channels.slack.*`)
-   **Listas de permitidos de emparejamiento**:
    -   `~/.openclaw/credentials/-allowFrom.json` (cuenta predeterminada)
    -   `~/.openclaw/credentials/--allowFrom.json` (cuentas no predeterminadas)
-   **Perfiles de autenticación de modelos**: `~/.openclaw/agents//agent/auth-profiles.json`
-   **Carga útil de secretos respaldada por archivo (opcional)**: `~/.openclaw/secrets.json`
-   **Importación heredada de OAuth**: `~/.openclaw/credentials/oauth.json` Más detalles: [Seguridad](../gateway/security.md#credential-storage-map).

## Actualizar (sin arruinar tu configuración)

-   Mantén `~/.openclaw/workspace` y `~/.openclaw/` como "tus cosas"; no pongas prompts/configuración personal en el repositorio `openclaw`.
-   Actualizar el código fuente: `git pull` + `pnpm install` (cuando cambie el archivo de bloqueo) + sigue usando `pnpm gateway:watch`.

## Linux (servicio de usuario systemd)

Las instalaciones de Linux usan un servicio de **usuario** systemd. Por defecto, systemd detiene los servicios de usuario al cerrar sesión/inactividad, lo que mata el Gateway. La incorporación intenta habilitar la permanencia por ti (puede solicitar sudo). Si sigue desactivado, ejecuta:

```bash
sudo loginctl enable-linger $USER
```

Para servidores siempre activos o multiusuario, considera un servicio de **sistema** en lugar de un servicio de usuario (no se necesita permanencia). Consulta el [Manual del Gateway](../gateway.md) para ver las notas sobre systemd.

## Documentación relacionada

-   [Manual del Gateway](../gateway.md) (banderas, supervisión, puertos)
-   [Configuración del Gateway](../gateway/configuration.md) (esquema de configuración + ejemplos)
-   [Discord](../channels/discord.md) y [Telegram](../channels/telegram.md) (etiquetas de respuesta y ajustes replyToMode)
-   [Configuración del asistente OpenClaw](./openclaw.md)
-   [Aplicación macOS](../platforms/macos.md) (ciclo de vida del gateway)

[Inmersión profunda en gestión de sesiones](../reference/session-management-compaction.md)[Flujo de trabajo de desarrollo Pi](../pi-dev.md)