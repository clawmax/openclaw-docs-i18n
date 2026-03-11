

  Ayuda

  
# Preguntas Frecuentes

Respuestas rápidas más solución de problemas profunda para configuraciones del mundo real (desarrollo local, VPS, multi-agente, OAuth/claves API, conmutación por error de modelos). Para diagnósticos en tiempo de ejecución, consulta [Solución de Problemas](../gateway/troubleshooting.md). Para la referencia completa de configuración, consulta [Configuración](../gateway/configuration.md).

## Tabla de contenidos

-   [Inicio rápido y configuración inicial]
    -   [Estoy atascado, ¿cuál es la forma más rápida de desatascarme?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
    -   [¿Cuál es la forma recomendada de instalar y configurar OpenClaw?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
    -   [¿Cómo abro el panel de control después de la incorporación?](#how-do-i-open-the-dashboard-after-onboarding)
    -   [¿Cómo autentico el panel de control (token) en localhost vs remoto?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
    -   [¿Qué entorno de ejecución necesito?](#what-runtime-do-i-need)
    -   [¿Funciona en Raspberry Pi?](#does-it-run-on-raspberry-pi)
    -   [¿Algún consejo para instalaciones en Raspberry Pi?](#any-tips-for-raspberry-pi-installs)
    -   [Está atascado en "despierta amigo mío" / la incorporación no eclosionará. ¿Qué hago ahora?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
    -   [¿Puedo migrar mi configuración a una máquina nueva (Mac mini) sin repetir la incorporación?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
    -   [¿Dónde veo qué hay de nuevo en la última versión?](#where-do-i-see-what-is-new-in-the-latest-version)
    -   [No puedo acceder a docs.openclaw.ai (error SSL). ¿Qué hago ahora?](#i-cant-access-docsopenclawai-ssl-error-what-now)
    -   [¿Cuál es la diferencia entre estable y beta?](#whats-the-difference-between-stable-and-beta)
    -   [¿Cómo instalo la versión beta y cuál es la diferencia entre beta y dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
    -   [¿Cómo pruebo las últimas novedades?](#how-do-i-try-the-latest-bits)
    -   [¿Cuánto tiempo suele tomar la instalación y la incorporación?](#how-long-does-install-and-onboarding-usually-take)
    -   [¿Instalador atascado? ¿Cómo obtengo más información?](#installer-stuck-how-do-i-get-more-feedback)
    -   [La instalación en Windows dice que git no se encuentra o openclaw no se reconoce](#windows-install-says-git-not-found-or-openclaw-not-recognized)
    -   [La salida de exec en Windows muestra texto chino ilegible, ¿qué debo hacer?](#windows-exec-output-shows-garbled-chinese-text-what-should-i-do)
    -   [La documentación no respondió mi pregunta, ¿cómo obtengo una mejor respuesta?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
    -   [¿Cómo instalo OpenClaw en Linux?](#how-do-i-install-openclaw-on-linux)
    -   [¿Cómo instalo OpenClaw en un VPS?](#how-do-i-install-openclaw-on-a-vps)
    -   [¿Dónde están las guías de instalación en la nube/VPS?](#where-are-the-cloudvps-install-guides)
    -   [¿Puedo pedirle a OpenClaw que se actualice a sí mismo?](#can-i-ask-openclaw-to-update-itself)
    -   [¿Qué hace realmente el asistente de incorporación?](#what-does-the-onboarding-wizard-actually-do)
    -   [¿Necesito una suscripción a Claude o OpenAI para ejecutar esto?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
    -   [¿Puedo usar la suscripción Claude Max sin una clave API?](#can-i-use-claude-max-subscription-without-an-api-key)
    -   [¿Cómo funciona la autenticación "setup-token" de Anthropic?](#how-does-anthropic-setuptoken-auth-work)
    -   [¿Dónde encuentro un setup-token de Anthropic?](#where-do-i-find-an-anthropic-setuptoken)
    -   [¿Soportan la autenticación por suscripción de Claude (Claude Pro o Max)?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
    -   [¿Por qué veo `HTTP 429: rate_limit_error` de Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
    -   [¿Se soporta AWS Bedrock?](#is-aws-bedrock-supported)
    -   [¿Cómo funciona la autenticación de Codex?](#how-does-codex-auth-work)
    -   [¿Soportan la autenticación por suscripción de OpenAI (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
    -   [¿Cómo configuro Gemini CLI OAuth?](#how-do-i-set-up-gemini-cli-oauth)
    -   [¿Es un modelo local adecuado para chats casuales?](#is-a-local-model-ok-for-casual-chats)
    -   [¿Cómo mantengo el tráfico de modelos alojados en una región específica?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
    -   [¿Tengo que comprar un Mac Mini para instalar esto?](#do-i-have-to-buy-a-mac-mini-to-install-this)
    -   [¿Necesito un Mac mini para soporte de iMessage?](#do-i-need-a-mac-mini-for-imessage-support)
    -   [Si compro un Mac mini para ejecutar OpenClaw, ¿puedo conectarlo a mi MacBook Pro?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
    -   [¿Puedo usar Bun?](#can-i-use-bun)
    -   [Telegram: ¿qué va en `allowFrom`?](#telegram-what-goes-in-allowfrom)
    -   [¿Pueden varias personas usar un número de WhatsApp con diferentes instancias de OpenClaw?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
    -   [¿Puedo ejecutar un agente de "chat rápido" y un agente de "Opus para programación"?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
    -   [¿Funciona Homebrew en Linux?](#does-homebrew-work-on-linux)
    -   [¿Cuál es la diferencia entre la instalación hackeable (git) y la instalación npm?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
    -   [¿Puedo cambiar entre instalaciones npm y git más tarde?](#can-i-switch-between-npm-and-git-installs-later)
    -   [¿Debo ejecutar el Gateway en mi portátil o en un VPS?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
    -   [¿Qué tan importante es ejecutar OpenClaw en una máquina dedicada?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
    -   [¿Cuáles son los requisitos mínimos de VPS y el sistema operativo recomendado?](#what-are-the-minimum-vps-requirements-and-recommended-os)
    -   [¿Puedo ejecutar OpenClaw en una VM y cuáles son los requisitos?](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
-   [¿Qué es OpenClaw?](#what-is-openclaw)
    -   [¿Qué es OpenClaw, en un párrafo?](#what-is-openclaw-in-one-paragraph)
    -   [¿Cuál es la propuesta de valor?](#whats-the-value-proposition)
    -   [Acabo de configurarlo, ¿qué debo hacer primero?](#i-just-set-it-up-what-should-i-do-first)
    -   [¿Cuáles son los cinco principales casos de uso cotidianos para OpenClaw?](#what-are-the-top-five-everyday-use-cases-for-openclaw)
    -   [¿Puede OpenClaw ayudar con la generación de leads, alcance, anuncios y blogs para un SaaS?](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
    -   [¿Cuáles son las ventajas frente a Claude Code para desarrollo web?](#what-are-the-advantages-vs-claude-code-for-web-development)
-   [Habilidades y automatización](#skills-and-automation)
    -   [¿Cómo personalizo habilidades sin ensuciar el repositorio?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
    -   [¿Puedo cargar habilidades desde una carpeta personalizada?](#can-i-load-skills-from-a-custom-folder)
    -   [¿Cómo puedo usar diferentes modelos para diferentes tareas?](#how-can-i-use-different-models-for-different-tasks)
    -   [El bot se congela mientras hace trabajo pesado. ¿Cómo descargo eso?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
    -   [Cron o recordatorios no se activan. ¿Qué debo verificar?](#cron-or-reminders-do-not-fire-what-should-i-check)
    -   [¿Cómo instalo habilidades en Linux?](#how-do-i-install-skills-on-linux)
    -   [¿Puede OpenClaw ejecutar tareas en un horario o continuamente en segundo plano?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
    -   [¿Puedo ejecutar habilidades exclusivas de Apple macOS desde Linux?](#can-i-run-apple-macos-only-skills-from-linux)
    -   [¿Tienen una integración con Notion o HeyGen?](#do-you-have-a-notion-or-heygen-integration)
    -   [¿Cómo instalo la extensión de Chrome para la toma de control del navegador?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
-   [Sandboxing y memoria](#sandboxing-and-memory)
    -   [¿Hay un documento dedicado sobre sandboxing?](#is-there-a-dedicated-sandboxing-doc)
    -   [¿Cómo vinculo una carpeta del host al sandbox?](#how-do-i-bind-a-host-folder-into-the-sandbox)
    -   [¿Cómo funciona la memoria?](#how-does-memory-work)
    -   [La memoria sigue olvidando cosas. ¿Cómo hago que se quede?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
    -   [¿La memoria persiste para siempre? ¿Cuáles son los límites?](#does-memory-persist-forever-what-are-the-limits)
    -   [¿La búsqueda semántica de memoria requiere una clave API de OpenAI?](#does-semantic-memory-search-require-an-openai-api-key)
-   [Dónde viven las cosas en el disco](#where-things-live-on-disk)
    -   [¿Todos los datos usados con OpenClaw se guardan localmente?](#is-all-data-used-with-openclaw-saved-locally)
    -   [¿Dónde almacena OpenClaw sus datos?](#where-does-openclaw-store-its-data)
    -   [¿Dónde deben vivir AGENTS.md / SOUL.md / USER.md / MEMORY.md?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
    -   [¿Cuál es la estrategia de respaldo recomendada?](#whats-the-recommended-backup-strategy)
    -   [¿Cómo desinstalo completamente OpenClaw?](#how-do-i-completely-uninstall-openclaw)
    -   [¿Pueden los agentes trabajar fuera del espacio de trabajo?](#can-agents-work-outside-the-workspace)
    -   [Estoy en modo remoto, ¿dónde está el almacén de sesiones?](#im-in-remote-mode-where-is-the-session-store)
-   [Conceptos básicos de configuración](#config-basics)
    -   [¿Qué formato tiene la configuración? ¿Dónde está?](#what-format-is-the-config-where-is-it)
    -   [Establecí `gateway.bind: "lan"` (o `"tailnet"`) y ahora nada escucha / la UI dice no autorizado](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
    -   [¿Por qué necesito un token en localhost ahora?](#why-do-i-need-a-token-on-localhost-now)
    -   [¿Tengo que reiniciar después de cambiar la configuración?](#do-i-have-to-restart-after-changing-config)
    -   [¿Cómo desactivo los lemas graciosos de la CLI?](#how-do-i-disable-funny-cli-taglines)
    -   [¿Cómo habilito la búsqueda web (y la obtención web)?](#how-do-i-enable-web-search-and-web-fetch)
    -   [config.apply borró mi configuración. ¿Cómo la recupero y evito esto?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
    -   [¿Cómo ejecuto un Gateway central con trabajadores especializados en varios dispositivos?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
    -   [¿Puede el navegador de OpenClaw ejecutarse sin cabeza?](#can-the-openclaw-browser-run-headless)
    -   [¿Cómo uso Brave para el control del navegador?](#how-do-i-use-brave-for-browser-control)
-   [Gateways remotos y nodos](#remote-gateways-and-nodes)
    -   [¿Cómo se propagan los comandos entre Telegram, el gateway y los nodos?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
    -   [¿Cómo puede mi agente acceder a mi computadora si el Gateway está alojado remotamente?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
    -   [Tailscale está conectado pero no obtengo respuestas. ¿Qué hago ahora?](#tailscale-is-connected-but-i-get-no-replies-what-now)
    -   [¿Pueden dos instancias de OpenClaw hablar entre sí (local + VPS)?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
    -   [¿Necesito VPS separados para múltiples agentes?](#do-i-need-separate-vpses-for-multiple-agents)
    -   [¿Hay un beneficio en usar un nodo en mi portátil personal en lugar de SSH desde un VPS?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
    -   [¿Los nodos ejecutan un servicio de gateway?](#do-nodes-run-a-gateway-service)
    -   [¿Hay una forma API / RPC de aplicar configuración?](#is-there-an-api-rpc-way-to-apply-config)
    -   [¿Cuál es una configuración mínima "sana" para una primera instalación?](#whats-a-minimal-sane-config-for-a-first-install)
    -   [¿Cómo configuro Tailscale en un VPS y me conecto desde mi Mac?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
    -   [¿Cómo conecto un nodo Mac a un Gateway remoto (Tailscale Serve)?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
    -   [¿Debo instalar en un segundo portátil o simplemente agregar un nodo?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
-   [Variables de entorno y carga de .env](#env-vars-and-env-loading)
    -   [¿Cómo carga OpenClaw las variables de entorno?](#how-does-openclaw-load-environment-variables)
    -   ["Inicié el Gateway a través del servicio y mis variables de entorno desaparecieron." ¿Qué hago ahora?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
    -   [Establecí `COPILOT_GITHUB_TOKEN`, pero el estado de modelos muestra "Shell env: off." ¿Por qué?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
-   [Sesiones y múltiples chats](#sessions-and-multiple-chats)
    -   [¿Cómo inicio una conversación nueva?](#how-do-i-start-a-fresh-conversation)
    -   [¿Las sesiones se reinician automáticamente si nunca envío `/new`?](#do-sessions-reset-automatically-if-i-never-send-new)
    -   [¿Hay una forma de hacer un equipo de instancias de OpenClaw, un CEO y muchos agentes?](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
    -   [¿Por qué se truncó el contexto a mitad de la tarea? ¿Cómo lo evito?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
    -   [¿Cómo reinicio completamente OpenClaw pero lo mantengo instalado?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
    -   [Recibo errores "contexto demasiado grande", ¿cómo reinicio o compacto?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
    -   [¿Por qué veo "LLM request rejected: messages.content.tool\_use.input field required"?](#why-am-i-seeing-llm-request-rejected-messagescontenttool_useinput-field-required)
    -   [¿Por qué recibo mensajes de latido cada 30 minutos?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
    -   [¿Necesito agregar una "cuenta de bot" a un grupo de WhatsApp?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
    -   [¿Cómo obtengo el JID de un grupo de WhatsApp?](#how-do-i-get-the-jid-of-a-whatsapp-group)
    -   [¿Por qué OpenClaw no responde en un grupo?](#why-doesnt-openclaw-reply-in-a-group)
    -   [¿Los grupos/hilos comparten contexto con los mensajes directos?](#do-groupsthreads-share-context-with-dms)
    -   [¿Cuántos espacios de trabajo y agentes puedo crear?](#how-many-workspaces-and-agents-can-i-create)
    -   [¿Puedo ejecutar múltiples bots o chats al mismo tiempo (Slack) y cómo debo configurar eso?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
-   [Modelos: valores predeterminados, selección, alias, cambio](#models-defaults-selection-aliases-switching)
    -   [¿Qué es el "modelo predeterminado"?](#what-is-the-default-model)
    -   [¿Qué modelo recomiendan?](#what-model-do-you-recommend)
    -   [¿Cómo cambio de modelos sin borrar mi configuración?](#how-do-i-switch-models-without-wiping-my-config)
    -   [¿Puedo usar modelos autoalojados (llama.cpp, vLLM, Ollama)?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
    -   [¿Qué usan OpenClaw, Flawd y Krill para modelos?](#what-do-openclaw-flawd-and-krill-use-for-models)
    -   [¿Cómo cambio de modelos sobre la marcha (sin reiniciar)?](#how-do-i-switch-models-on-the-fly-without-restarting)
    -   [¿Puedo usar GPT 5.2 para tareas diarias y Codex 5.3 para programación?](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
    -   [¿Por qué veo "Modelo ... no permitido" y luego no hay respuesta?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
    -   [¿Por qué veo "Unknown model: minimax/MiniMax-M2.5"?](#why-do-i-see-unknown-model-minimaxminimaxm25)
    -   [¿Puedo usar MiniMax como mi predeterminado y OpenAI para tareas complejas?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
    -   [¿Son opus / sonnet / gpt atajos incorporados?](#are-opus-sonnet-gpt-builtin-shortcuts)
    -   [¿Cómo defino/sobrescribo atajos de modelos (alias)?](#how-do-i-defineoverride-model-shortcuts-aliases)
    -   [¿Cómo agrego modelos de otros proveedores como OpenRouter o Z.AI?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
-   [Conmutación por error de modelos y "Todos los modelos fallaron"](#model-failover-and-all-models-failed)
    -   [¿Cómo funciona la conmutación por error?](#how-does-failover-work)
    -   [¿Qué significa este error?](#what-does-this-error-mean)
    -   [Lista de verificación para arreglar `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
    -   [¿Por qué también intentó Google Gemini y falló?](#why-did-it-also-try-google-gemini-and-fail)
-   [Perfiles de autenticación: qué son y cómo gestionarlos](#auth-profiles-what-they-are-and-how-to-manage-them)
    -   [¿Qué es un perfil de autenticación?](#what-is-an-auth-profile)
    -   [¿Cuáles son los IDs de perfil típicos?](#what-are-typical-profile-ids)
    -   [¿Puedo controlar qué perfil de autenticación se intenta primero?](#can-i-control-which-auth-profile-is-tried-first)
    -   [OAuth vs clave API: ¿cuál es la diferencia?](#oauth-vs-api-key-whats-the-difference)
-   [Gateway: puertos, "ya se está ejecutando" y modo remoto](#gateway-ports-already-running-and-remote-mode)
    -   [¿Qué puerto usa el Gateway?](#what-port-does-the-gateway-use)
    -   [¿Por qué `openclaw gateway status` dice `Runtime: running` pero `RPC probe: failed`?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
    -   [¿Por qué `openclaw gateway status` muestra `Config (cli)` y `Config (service)` diferentes?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
    -   [¿Qué significa "otra instancia del gateway ya está escuchando"?](#what-does-another-gateway-instance-is-already-listening-mean)
    -   [¿Cómo ejecuto OpenClaw en modo remoto (el cliente se conecta a un Gateway en otro lugar)?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
    -   [La UI de Control dice "no autorizado" (o sigue reconectando). ¿Qué hago ahora?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
    -   [Establecí `gateway.bind: "tailnet"` pero no puede vincular / nada escucha](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
    -   [¿Puedo ejecutar múltiples Gateways en el mismo host?](#can-i-run-multiple-gateways-on-the-same-host)
    -   [¿Qué significa "invalid handshake" / código 1008?](#what-does-invalid-handshake-code-1008-mean)
-   [Registro y depuración](#logging-and-debugging)
    -   [¿Dónde están los registros?](#where-are-logs)
    -   [¿Cómo inicio/detengo/reinicio el servicio del Gateway?](#how-do-i-startstoprestart-the-gateway-service)
    -   [Cerré mi terminal en Windows, ¿cómo reinicio OpenClaw?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
    -   [El Gateway está activo pero las respuestas nunca llegan. ¿Qué debo verificar?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
    -   ["Desconectado del gateway: sin razón", ¿qué hago ahora?](#disconnected-from-gateway-no-reason-what-now)
    -   [Telegram setMyCommands falla con errores de red. ¿Qué debo verificar?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
    -   [La TUI no muestra salida. ¿Qué debo verificar?](#tui-shows-no-output-what-should-i-check)
    -   [¿Cómo detengo completamente y luego inicio el Gateway?](#how-do-i-completely-stop-then-start-the-gateway)
    -   [Explicado para un niño de 5 años: `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
    -   [¿Cuál es la forma más rápida de obtener más detalles cuando algo falla?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
-   [Medios y archivos adjuntos](#media-and-attachments)
    -   [Mi habilidad generó una imagen/PDF, pero no se envió nada](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
-   [Seguridad y control de acceso](#security-and-access-control)
    -   [¿Es seguro exponer OpenClaw a mensajes directos entrantes?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
    -   [¿La inyección de prompts es solo una preocupación para bots públicos?](#is-prompt-injection-only-a-concern-for-public-bots)
    -   [¿Mi bot debería tener su propia cuenta de correo electrónico, GitHub o número de teléfono?](#should-my-bot-have-its-own-email-github-account-or-phone-number)
    -   [¿Puedo darle autonomía sobre mis mensajes de texto y es eso seguro?](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
    -   [¿Puedo usar modelos más baratos para tareas de asistente personal?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
    -   [Ejecuté `/start` en Telegram pero no obtuve un código de emparejamiento](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
    -   [WhatsApp: ¿enviará mensajes a mis contactos? ¿Cómo funciona el emparejamiento?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
-   [Comandos de chat, abortar tareas y "no se detiene"](#chat-commands-aborting-tasks-and-it-wont-stop)
    -   [¿Cómo evito que los mensajes internos del sistema se muestren en el chat?](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
    -   [¿Cómo detengo/cancelo una tarea en ejecución?](#how-do-i-stopcancel-a-running-task)
    -   [¿Cómo envío un mensaje de Discord desde Telegram? ("Cross-context messaging denied")](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
    -   [¿Por qué parece que el bot "ignora" mensajes rápidos?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## Primeros 60 segundos si algo está roto

1.  **Estado rápido (primera verificación)**
    
    Copiar
    
    ```bash
    openclaw status
    ```
    
    Resumen local rápido: SO + actualización, accesibilidad del gateway/servicio, agentes/sesiones, configuración del proveedor + problemas de tiempo de ejecución (cuando el gateway es accesible).
2.  **Informe para pegar (seguro para compartir)**
    
    Copiar
    
    ```bash
    openclaw status --all
    ```
    
    Diagnóstico de solo lectura con cola de registros (tokens redactados).
3.  **Estado del daemon + puerto**
    
    Copiar
    
    ```bash
    openclaw gateway status
    ```
    
    Muestra el tiempo de ejecución del supervisor vs la accesibilidad RPC, la URL objetivo de la sonda y qué configuración probablemente usó el servicio.
4.  **Sondas profundas**
    
    Copiar
    
    ```bash
    openclaw status --deep
    ```
    
    Ejecuta comprobaciones de salud del gateway + sondas de proveedores (requiere un gateway accesible). Ver [Salud](../gateway/health.md).
5.  **Seguir el último registro**
    
    Copiar
    
    ```bash
    openclaw logs --follow
    ```
    
    Si RPC está caído, recurre a:
    
    Copiar
    
    ```bash
    tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
    ```
    
    Los registros de archivos están separados de los registros del servicio; ver [Registro](../logging.md) y [Solución de Problemas](../gateway/troubleshooting.md).
6.  **Ejecutar el doctor (reparaciones)**
    
    Copiar
    
    ```bash
    openclaw doctor
    ```
    
    Repara/migra configuración/estado + ejecuta comprobaciones de salud. Ver [Doctor](../gateway/doctor.md).
7.  **Instantánea del Gateway**
    
    Copiar
    
    ```bash
    openclaw health --json
    openclaw health --verbose   # muestra la URL objetivo + ruta de configuración en errores
    ```
    
    Pide al gateway en ejecución una instantánea completa (solo WS). Ver [Salud](../gateway/health.md).

## Inicio rápido y configuración inicial

### Estoy atascado, ¿cuál es la forma más rápida de desatascarme

Usa un agente de IA local que pueda **ver tu máquina**. Eso es mucho más efectivo que preguntar en Discord, porque la mayoría de los casos de "Estoy atascado" son **problemas de configuración o entorno local** que los ayudantes remotos no pueden inspeccionar.

-   **Claude Code**: [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
-   **OpenAI Codex**: [https://openai.com/codex/](https://openai.com/codex/)

Estas herramientas pueden leer el repositorio, ejecutar comandos, inspeccionar registros y ayudar a arreglar la configuración a nivel de máquina (PATH, servicios, permisos, archivos de autenticación). Dale la **copia completa del código fuente** a través de la instalación hackeable (git):

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Esto instala OpenClaw **desde una copia de git**, para que el agente pueda leer el código + la documentación y razonar sobre la versión exacta que estás ejecutando. Siempre puedes volver a la versión estable más tarde volviendo a ejecutar el instalador sin `--install-method git`. Consejo: pídele al agente que **planifique y supervise** la reparación (paso a paso), luego ejecuta solo los comandos necesarios. Eso mantiene los cambios pequeños y más fáciles de auditar. Si descubres un error real o una solución, por favor abre un issue en GitHub o envía un PR: [https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues) [https://github.com/openclaw/openclaw/pulls](https://github.com/openclaw/openclaw/pulls) Comienza con estos comandos (comparte las salidas cuando pidas ayuda):

```bash
openclaw status
openclaw models status
openclaw doctor
```

Qué hacen:

-   `openclaw status`: instantánea rápida de la salud del gateway/agente + configuración básica.
-   `openclaw models status`: verifica la autenticación del proveedor + disponibilidad de modelos.
-   `openclaw doctor`: valida y repara problemas comunes de configuración/estado.

Otras comprobaciones útiles de la CLI: `openclaw status --all`, `openclaw logs --follow`, `openclaw gateway status`, `openclaw health --verbose`. Bucle de depuración