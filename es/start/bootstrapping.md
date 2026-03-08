

  Arranque inicial

  
# Arranque inicial del agente

El arranque inicial es el ritual de **primera ejecución** que prepara un espacio de trabajo del agente y recopila detalles de identidad. Ocurre después de la incorporación, cuando el agente se inicia por primera vez.

## Qué hace el arranque inicial

En la primera ejecución del agente, OpenClaw prepara el espacio de trabajo (por defecto `~/.openclaw/workspace`):

-   Inicializa `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`.
-   Ejecuta un breve ritual de preguntas y respuestas (una pregunta a la vez).
-   Escribe la identidad + preferencias en `IDENTITY.md`, `USER.md`, `SOUL.md`.
-   Elimina `BOOTSTRAP.md` cuando termina, para que solo se ejecute una vez.

## Dónde se ejecuta

El arranque inicial siempre se ejecuta en el **host de puerta de enlace**. Si la aplicación de macOS se conecta a una Puerta de enlace remota, el espacio de trabajo y los archivos de arranque residen en esa máquina remota.

> **ℹ️** Cuando la Puerta de enlace se ejecuta en otra máquina, edita los archivos del espacio de trabajo en el host de la puerta de enlace (por ejemplo, `user@gateway-host:~/.openclaw/workspace`).

## Documentación relacionada

-   Incorporación de la aplicación macOS: [Incorporación](./onboarding.md)
-   Diseño del espacio de trabajo: [Espacio de trabajo del agente](../concepts/agent-workspace.md)

[OAuth](../concepts/oauth.md)[Gestión de sesiones](../concepts/session.md)

---