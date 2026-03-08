title: "Guía de Pruebas de OpenClaw: Suites de Pruebas Unitarias, E2E y en Vivo"
description: "Aprende cómo probar OpenClaw con suites de pruebas unitarias, de integración, E2E y en vivo. Ejecuta comandos para depuración, cobertura y validación de proveedores/modelos reales."
keywords: ["pruebas openclaw", "suites vitest", "pruebas e2e", "pruebas en vivo", "prueba de humo del gateway", "pruebas de nodo android", "integración de modelos", "cobertura de pruebas"]
---

  Entorno y depuración

  
# Pruebas

OpenClaw tiene tres suites de Vitest (unidad/integración, e2e, en vivo) y un pequeño conjunto de ejecutores Docker. Este documento es una guía de "cómo probamos":

-   Qué cubre cada suite (y qué *no* cubre deliberadamente)
-   Qué comandos ejecutar para flujos de trabajo comunes (local, pre-push, depuración)
-   Cómo las pruebas en vivo descubren credenciales y seleccionan modelos/proveedores
-   Cómo agregar regresiones para problemas reales de modelos/proveedores

## Inicio rápido

La mayoría de los días:

-   Puerta completa (esperada antes del push): `pnpm build && pnpm check && pnpm test`

Cuando tocas pruebas o quieres confianza extra:

-   Puerta de cobertura: `pnpm test:coverage`
-   Suite E2E: `pnpm test:e2e`

Cuando depuras proveedores/modelos reales (requiere credenciales reales):

-   Suite en vivo (modelos + sondeos de herramienta/imagen del gateway): `pnpm test:live`

Consejo: cuando solo necesitas un caso fallido, prefiere reducir las pruebas en vivo mediante las variables de entorno de lista permitida descritas a continuación.

## Suites de pruebas (qué se ejecuta dónde)

Piensa en las suites como "realismo creciente" (y creciente fragilidad/costo):

### Unidad / integración (predeterminada)

-   Comando: `pnpm test`
-   Configuración: `scripts/test-parallel.mjs` (ejecuta `vitest.unit.config.ts`, `vitest.extensions.config.ts`, `vitest.gateway.config.ts`)
-   Archivos: `src/**/*.test.ts`, `extensions/**/*.test.ts`
-   Alcance:
    -   Pruebas unitarias puras
    -   Pruebas de integración en proceso (autenticación del gateway, enrutamiento, herramientas, análisis, configuración)
    -   Regresiones deterministas para errores conocidos
-   Expectativas:
    -   Se ejecuta en CI
    -   No se requieren claves reales
    -   Debe ser rápida y estable
-   Nota sobre el pool:
    -   OpenClaw usa `vmForks` de Vitest en Node 22/23 para fragmentos unitarios más rápidos.
    -   En Node 24+, OpenClaw vuelve automáticamente a `forks` regular para evitar errores de enlace de VM de Node (`ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`).
    -   Anula manualmente con `OPENCLAW_TEST_VM_FORKS=0` (forzar `forks`) o `OPENCLAW_TEST_VM_FORKS=1` (forzar `vmForks`).

### E2E (prueba de humo del gateway)

-   Comando: `pnpm test:e2e`
-   Configuración: `vitest.e2e.config.ts`
-   Archivos: `src/**/*.e2e.test.ts`
-   Valores predeterminados de tiempo de ejecución:
    -   Usa `vmForks` de Vitest para un inicio de archivo más rápido.
    -   Usa trabajadores adaptativos (CI: 2-4, local: 4-8).
    -   Se ejecuta en modo silencioso por defecto para reducir la sobrecarga de E/S de la consola.
-   Anulaciones útiles:
    -   `OPENCLAW_E2E_WORKERS=` para forzar el número de trabajadores (limitado a 16).
    -   `OPENCLAW_E2E_VERBOSE=1` para volver a habilitar la salida detallada de la consola.
-   Alcance:
    -   Comportamiento de extremo a extremo del gateway multi-instancia
    -   Superficies WebSocket/HTTP, emparejamiento de nodos y redes más pesadas
-   Expectativas:
    -   Se ejecuta en CI (cuando está habilitado en el pipeline)
    -   No se requieren claves reales
    -   Más partes móviles que las pruebas unitarias (puede ser más lento)

### En vivo (proveedores reales + modelos reales)

-   Comando: `pnpm test:live`
-   Configuración: `vitest.live.config.ts`
-   Archivos: `src/**/*.live.test.ts`
-   Predeterminado: **habilitado** por `pnpm test:live` (establece `OPENCLAW_LIVE_TEST=1`)
-   Alcance:
    -   "¿Funciona realmente este proveedor/modelo *hoy* con credenciales reales?"
    -   Detectar cambios de formato del proveedor, peculiaridades de llamadas a herramientas, problemas de autenticación y comportamiento de límites de tasa
-   Expectativas:
    -   No es estable en CI por diseño (redes reales, políticas reales de proveedores, cuotas, interrupciones)
    -   Cuesta dinero / usa límites de tasa
    -   Prefiere ejecutar subconjuntos reducidos en lugar de "todo"
    -   Las ejecuciones en vivo obtendrán `~/.profile` para recoger claves API faltantes
-   Rotación de claves API (específica del proveedor): establece `*_API_KEYS` con formato de coma/punto y coma o `*_API_KEY_1`, `*_API_KEY_2` (por ejemplo `OPENAI_API_KEYS`, `ANTHROPIC_API_KEYS`, `GEMINI_API_KEYS`) o anulación por prueba en vivo mediante `OPENCLAW_LIVE_*_KEY`; las pruebas reintentan en respuestas de límite de tasa.

## ¿Qué suite debo ejecutar?

Usa esta tabla de decisión:

-   Editando lógica/pruebas: ejecuta `pnpm test` (y `pnpm test:coverage` si cambiaste mucho)
-   Tocando redes del gateway / protocolo WS / emparejamiento: agrega `pnpm test:e2e`
-   Depurando "mi bot está caído" / fallos específicos del proveedor / llamadas a herramientas: ejecuta un `pnpm test:live` reducido

## En vivo: barrido de capacidades del nodo Android

-   Prueba: `src/gateway/android-node.capabilities.live.test.ts`
-   Script: `pnpm android:test:integration`
-   Objetivo: invocar **cada comando actualmente anunciado** por un nodo Android conectado y afirmar el comportamiento del contrato del comando.
-   Alcance:
    -   Configuración precondicionada/manual (la suite no instala/ejecuta/empareja la aplicación).
    -   Validación de `node.invoke` del gateway comando por comando para el nodo Android seleccionado.
-   Preconfiguración requerida:
    -   Aplicación Android ya conectada + emparejada al gateway.
    -   Aplicación mantenida en primer plano.
    -   Permisos/consentimiento de captura otorgados para las capacidades que esperas que pasen.
-   Anulaciones de destino opcionales:
    -   `OPENCLAW_ANDROID_NODE_ID` o `OPENCLAW_ANDROID_NODE_NAME`.
    -   `OPENCLAW_ANDROID_GATEWAY_URL` / `OPENCLAW_ANDROID_GATEWAY_TOKEN` / `OPENCLAW_ANDROID_GATEWAY_PASSWORD`.
-   Detalles completos de configuración de Android: [Aplicación Android](../platforms/android.md)

## En vivo: prueba de humo de modelos (claves de perfil)

Las pruebas en vivo se dividen en dos capas para poder aislar fallos:

-   "Modelo directo" nos dice si el proveedor/modelo puede responder al menos con la clave dada.
-   "Prueba de humo del gateway" nos dice si el pipeline completo gateway+agente funciona para ese modelo (sesiones, historial, herramientas, política del sandbox, etc.).

### Capa 1: Completado de modelo directo (sin gateway)

-   Prueba: `src/agents/models.profiles.live.test.ts`
-   Objetivo:
    -   Enumerar modelos descubiertos
    -   Usar `getApiKeyForModel` para seleccionar modelos para los que tienes credenciales
    -   Ejecutar un pequeño completado por modelo (y regresiones específicas donde sea necesario)
-   Cómo habilitar:
    -   `pnpm test:live` (o `OPENCLAW_LIVE_TEST=1` si invocas Vitest directamente)
-   Establece `OPENCLAW_LIVE_MODELS=modern` (o `all`, alias para modern) para ejecutar realmente esta suite; de lo contrario, se omite para mantener `pnpm test:live` enfocado en la prueba de humo del gateway
-   Cómo seleccionar modelos:
    -   `OPENCLAW_LIVE_MODELS=modern` para ejecutar la lista permitida moderna (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.5, Grok 4)
    -   `OPENCLAW_LIVE_MODELS=all` es un alias para la lista permitida moderna
    -   o `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,..."` (lista permitida separada por comas)
-   Cómo seleccionar proveedores:
    -   `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (lista permitida separada por comas)
-   De dónde vienen las claves:
    -   Por defecto: almacén de perfiles y respaldos de entorno
    -   Establece `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` para exigir **solo el almacén de perfiles**
-   Por qué existe esto:
    -   Separa "la API del proveedor está rota / la clave es inválida" de "el pipeline del agente del gateway está roto"
    -   Contiene regresiones pequeñas y aisladas (ejemplo: reproducción de razonamiento de Respuestas OpenAI/Respuestas Codex + flujos de llamadas a herramientas)

### Capa 2: Prueba de humo del Gateway + agente de desarrollo (lo que "@openclaw" realmente hace)

-   Prueba: `src/gateway/gateway-models.profiles.live.test.ts`
-   Objetivo:
    -   Iniciar un gateway en proceso
    -   Crear/parchear una sesión `agent:dev:*` (anulación de modelo por ejecución)
    -   Iterar modelos-con-claves y afirmar:
        -   respuesta "significativa" (sin herramientas)
        -   una invocación real de herramienta funciona (sonda de lectura)
        -   sondas de herramienta adicionales opcionales (sonda de ejecución+lectura)
        -   las rutas de regresión de OpenAI (solo llamada a herramienta → seguimiento) siguen funcionando
-   Detalles de la sonda (para que puedas explicar fallos rápidamente):
    -   Sonda `read`: la prueba escribe un archivo nonce en el espacio de trabajo y le pide al agente que lo `lea` y devuelva el nonce.
    -   Sonda `exec+read`: la prueba le pide al agente que `ejecute`-escriba un nonce en un archivo temporal, luego lo `lea`.
    -   Sonda de imagen: la prueba adjunta un PNG generado (gato + código aleatorio) y espera que el modelo devuelva `cat <CÓDIGO>`.
    -   Referencia de implementación: `src/gateway/gateway-models.profiles.live.test.ts` y `src/gateway/live-image-probe.ts`.
-   Cómo habilitar:
    -   `pnpm test:live` (o `OPENCLAW_LIVE_TEST=1` si invocas Vitest directamente)
-   Cómo seleccionar modelos:
    -   Predeterminado: lista permitida moderna (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.5, Grok 4)
    -   `OPENCLAW_LIVE_GATEWAY_MODELS=all` es un alias para la lista permitida moderna
    -   O establece `OPENCLAW_LIVE_GATEWAY_MODELS="proveedor/modelo"` (o lista separada por comas) para reducir
-   Cómo seleccionar proveedores (evitar "todo OpenRouter"):
    -   `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (lista permitida separada por comas)
-   Las sondas de herramienta e imagen están siempre activas en esta prueba en vivo:
    -   Sonda `read` + sonda `exec+read` (estrés de herramienta)
    -   Sonda de imagen se ejecuta cuando el modelo anuncia soporte de entrada de imagen
    -   Flujo (alto nivel):
        -   La prueba genera un PNG pequeño con "CAT" + código aleatorio (`src/gateway/live-image-probe.ts`)
        -   Lo envía mediante `agent` `attachments: [{ mimeType: "image/png", content: "" }]`
        -   El gateway analiza los adjuntos en `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
        -   El agente integrado reenvía un mensaje de usuario multimodal al modelo
        -   Afirmación: la respuesta contiene `cat` + el código (tolerancia OCR: se permiten errores menores)

Consejo: para ver qué puedes probar en tu máquina (y los ids exactos de `proveedor/modelo`), ejecuta:

```bash
openclaw models list
openclaw models list --json
```

## En vivo: Prueba de humo del token de configuración de Anthropic

-   Prueba: `src/agents/anthropic.setup-token.live.test.ts`
-   Objetivo: verificar que el token de configuración de Claude Code CLI (o un perfil de token de configuración pegado) pueda completar un prompt de Anthropic.
-   Habilitar:
    -   `pnpm test:live` (o `OPENCLAW_LIVE_TEST=1` si invocas Vitest directamente)
    -   `OPENCLAW_LIVE_SETUP_TOKEN=1`
-   Fuentes del token (elige una):
    -   Perfil: `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
    -   Token crudo: `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
-   Anulación de modelo (opcional):
    -   `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-6`

Ejemplo de configuración:

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

## En vivo: Prueba de humo del backend CLI (Claude Code CLI u otros CLIs locales)

-   Prueba: `src/gateway/gateway-cli-backend.live.test.ts`
-   Objetivo: validar el pipeline Gateway + agente usando un backend CLI local, sin tocar tu configuración predeterminada.
-   Habilitar:
    -   `pnpm test:live` (o `OPENCLAW_LIVE_TEST=1` si invocas Vitest directamente)
    -   `OPENCLAW_LIVE_CLI_BACKEND=1`
-   Valores predeterminados:
    -   Modelo: `claude-cli/claude-sonnet-4-6`
    -   Comando: `claude`
    -   Argumentos: `["-p","--output-format","json","--permission-mode","bypassPermissions"]`
-   Anulaciones (opcionales):
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-6"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.4"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` para enviar un adjunto de imagen real (las rutas se inyectan en el prompt).
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` para pasar rutas de archivos de imagen como argumentos CLI en lugar de inyección en el prompt.
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (o `"list"`) para controlar cómo se pasan los argumentos de imagen cuando `IMAGE_ARG` está establecido.
    -   `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` para enviar un segundo turno y validar el flujo de reanudación.
-   `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` para mantener habilitada la configuración MCP de Claude Code CLI (por defecto deshabilita la configuración MCP con un archivo vacío temporal).

Ejemplo:

```
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-6" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

### Recetas en vivo recomendadas

Las listas permitidas explícitas y reducidas son las más rápidas y menos frágiles:

-   Modelo único, directo (sin gateway):
    -   `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`
-   Modelo único, prueba de humo del gateway:
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   Llamadas a herramientas en varios proveedores:
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   Enfoque en Google (clave API de Gemini + Antigravity):
    -   Gemini (clave API): `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
    -   Antigravity (OAuth): `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

Notas:

-   `google/...` usa la API de Gemini (clave API).
-   `google-antigravity/...` usa el puente OAuth de Antigravity (endpoint de agente estilo Cloud Code Assist).
-   `google-gemini-cli/...` usa el CLI de Gemini local en tu máquina (autenticación separada + peculiaridades de herramientas).
-   API de Gemini vs CLI de Gemini:
    -   API: OpenClaw llama a la API alojada de Gemini de Google sobre HTTP (autenticación por clave API / perfil); esto es a lo que la mayoría de los usuarios se refieren con "Gemini".
    -   CLI: OpenClaw ejecuta un binario `gemini` local; tiene su propia autenticación y puede comportarse de manera diferente (soporte de streaming/herramientas/desfase de versión).

## En vivo: matriz de modelos (lo que cubrimos)

No hay una "lista de modelos de CI" fija (en vivo es opcional), pero estos son los modelos **recomendados** para cubrir regularmente en una máquina de desarrollo con claves.

### Conjunto de humo moderno (llamadas a herramientas + imagen)

Esta es la ejecución de "modelos comunes" que esperamos que siga funcionando:

-   OpenAI (no Codex): `openai/gpt-5.2` (opcional: `openai/gpt-5.1`)
-   OpenAI Codex: `openai-codex/gpt-5.4`
-   Anthropic: `anthropic/claude-opus-4-6` (o `anthropic/claude-sonnet-4-5`)
-   Google (API Gemini): `google/gemini-3-pro-preview` y `google/gemini-3-flash-preview` (evitar modelos Gemini 2.x más antiguos)
-   Google (Antigravity): `google-antigravity/claude-opus-4-6-thinking` y `google-antigravity/gemini-3-flash`
-   Z.AI (GLM): `zai/glm-4.7`
-   MiniMax: `minimax/minimax-m2.5`

Ejecuta prueba de humo del gateway con herramientas + imagen: `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.4,anthropic/claude-opus-4-6,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

### Línea base: llamadas a herramientas (Lectura + Ejecución opcional)

Elige al menos uno por familia de proveedores:

-   OpenAI: `openai/gpt-5.2` (o `openai/gpt-5-mini`)
-   Anthropic: `anthropic/claude-opus-4-6` (o `anthropic/claude-sonnet-4-5`)
-   Google: `google/gemini-3-flash-preview` (o `google/gemini-3-pro-preview`)
-   Z.AI (GLM): `zai/glm-4.7`
-   MiniMax: `minimax/minimax-m2.5`

Cobertura adicional opcional (agradable de tener):

-   xAI: `xai/grok-4` (o el último disponible)
-   Mistral: `mistral/`… (elige un modelo capaz de "herramientas" que tengas habilitado)
-   Cerebras: `cerebras/`… (si tienes acceso)
-   LM Studio: `lmstudio/`… (local; las llamadas a herramientas dependen del modo API)

### Visión: envío de imagen (adjunto → mensaje multimodal)

Incluye al menos un modelo capaz de imágenes en `OPENCLAW_LIVE_GATEWAY_MODELS` (variantes de Claude/Gemini/OpenAI con capacidad de visión, etc.) para ejercitar la sonda de imagen.

### Agregadores / gateways alternativos

Si tienes claves habilitadas, también admitimos pruebas mediante:

-   OpenRouter: `openrouter/...` (cientos de modelos; usa `openclaw models scan` para encontrar candidatos capaces de herramientas+imagen)
-   OpenCode Zen: `opencode/...` (autenticación mediante `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

Más proveedores que puedes incluir en la matriz en vivo (si tienes credenciales/configuración):

-   Integrados: `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
-   Vía `models.providers` (endpoints personalizados): `minimax` (cloud/API), más cualquier proxy compatible con OpenAI/Anthropic (LM Studio, vLLM, LiteLLM, etc.)

Consejo: no intentes codificar "todos los modelos" en la documentación. La lista autoritativa es lo que `discoverModels(...)` devuelve en tu máquina + las claves que estén disponibles.

## Credenciales (nunca las comprometas)

Las pruebas en vivo descubren credenciales de la misma manera que la CLI. Implicaciones prácticas:

-   Si la CLI funciona, las pruebas en vivo deberían encontrar las mismas claves.
-   Si una prueba en vivo dice "sin credenciales", depura de la misma manera que depurarías `openclaw models list` / selección de modelos.
-   Almacén de perfiles: `~/.openclaw/credentials/` (preferido; lo que significa "claves de perfil" en las pruebas)
-   Configuración: `~/.openclaw/openclaw.json` (o `OPENCLAW_CONFIG_PATH`)

Si quieres confiar en claves de entorno (por ejemplo, exportadas en tu `~/.profile`), ejecuta pruebas locales después de `source ~/.profile`, o usa los ejecutores Docker a continuación (pueden montar `~/.profile` en el contenedor).

## Deepgram en vivo (transcripción de audio)

-   Prueba: `src/media-understanding/providers/deepgram/audio.live.test.ts`
-   Habilitar: `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

## BytePlus plan de codificación en vivo

-   Prueba: `src/agents/byteplus.live.test.ts`
-   Habilitar: `BYTEPLUS_API_KEY=... BYTEPLUS_LIVE_TEST=1 pnpm test:live src/agents/byteplus.live.test.ts`
-   Anulación de modelo opcional: `BYTEPLUS_CODING_MODEL=ark-code-latest`

## Ejecutores Docker (comprobaciones opcionales de "funciona en Linux")

Estos ejecutan `pnpm test:live` dentro de la imagen Docker del repositorio, montando tu directorio de configuración local y espacio de trabajo (y obteniendo `~/.profile` si está montado):

-   Modelos directos: `pnpm test:docker:live-models` (script: `scripts/test-live-models-docker.sh`)
-   Gateway + agente de desarrollo: `pnpm test:docker:live-gateway` (script: `scripts/test-live-gateway-models-docker.sh`)
-   Asistente de incorporación (TTY, scaffolding completo): `pnpm test:docker:onboard` (script: `scripts/e2e/onboard-docker.sh`)
-   Redes del gateway (dos contenedores, autenticación WS + salud): `pnpm test:docker:gateway-network` (script: `scripts/e2e/gateway-network-docker.sh`)
-   Plugins (carga de extensión personalizada + prueba de humo del registro): `pnpm test:docker:plugins` (script: `scripts/e2e/plugins-docker.sh`)

Prueba de humo manual de hilo de lenguaje natural ACP (no CI):

-   `bun scripts/dev/discord-acp-plain-language-smoke.ts --channel <discord-channel-id> ...`
-   Mantén este script para flujos de trabajo de regresión/depuración. Puede ser necesario nuevamente para la validación del enrutamiento de hilos ACP, así que no lo elimines.

Variables de entorno útiles:

-   `OPENCLAW_CONFIG_DIR=...` (predeterminado: `~/.openclaw`) montado en `/home/node/.openclaw`
-   `OPENCLAW_WORKSPACE_DIR=...` (predeterminado: `~/.openclaw/workspace`) montado en `/home/node/.openclaw/workspace`
-   `OPENCLAW_PROFILE_FILE=...` (predeterminado: `~/.profile`) montado en `/home/node/.profile` y obtenido antes de ejecutar pruebas
-   `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` para reducir la ejecución
-   `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` para asegurar que las credenciales provengan del almacén de perfiles (no del entorno)

## Sanidad de la documentación

Ejecuta comprobaciones de documentación después de editar documentos: `pnpm docs:list`.

## Regresión sin conexión (seguro para CI)

Estas son regresiones de "pipeline real" sin proveedores reales:

-   Llamadas a herramientas del gateway (OpenAI simulado, gateway real + bucle de agente): `src/gateway/gateway.test.ts` (caso: "ejecuta una llamada a herramienta de OpenAI simulada de extremo a extremo a través del bucle de agente del gateway")
-   Asistente del gateway (WS `wizard.start`/`wizard.next`, escribe configuración + autenticación forzada): `src/gateway/gateway.test.ts` (caso: "ejecuta el asistente sobre ws y escribe la configuración del token de autenticación")

## Evaluaciones de confiabilidad del agente (habilidades)

Ya tenemos algunas pruebas seguras para CI que se comportan como "evaluaciones de confiabilidad del agente":

-   Llamadas a herramientas simuladas a través del gateway real + bucle de agente (`src/gateway/gateway.test.ts`).
-   Flujos de asistente de extremo a extremo que validan el cableado de sesión y los efectos de configuración (`src/gateway/gateway.test.ts`).

Lo que aún falta para habilidades (ver [Habilidades](../tools/skills.md)):

-   **Toma de decisiones:** cuando las habilidades se enumeran en el prompt, ¿el agente elige la habilidad correcta (o evita las irrelevantes)?
-   **Cumplimiento:** ¿el agente lee `SKILL.md` antes de usar y sigue los pasos/argumentos requeridos?
-   **Contratos de flujo de trabajo:** escenarios de múltiples turnos que afirman el orden de las herramientas, la persistencia del historial de sesiones y los límites del sandbox.

Las evaluaciones futuras deben mantenerse deterministas primero:

-   Un ejecutor de escenarios que use proveedores simulados para afirmar llamadas a herramientas + orden, lecturas de archivos de habilidades y cableado de sesión.
-   Una pequeña suite de escenarios centrados en habilidades (usar vs evitar, control de acceso, inyección de prompt).
-   Evaluaciones en vivo opcionales (opcionales, controladas por entorno) solo después de que la suite segura para CI esté en su lugar.

## Agregar regresiones (guía)

Cuando arreglas un problema de proveedor/modelo descubierto en vivo:

-   Agrega una regresión segura para CI si es posible (proveedor simulado/stub, o captura la transformación exacta de la forma de solicitud)
-   Si es inherentemente solo en vivo (límites de tasa, políticas de autenticación), mantén la prueba en vivo reducida y opcional mediante variables de entorno
-   Prefiere apuntar a la capa más pequeña que detecte el error:
    -   error de conversión/reproducción de solicitud del proveedor → prueba de modelos directos
    -   error del pipeline de sesión/historial/herramientas del gateway → prueba de humo en vivo del gateway o prueba simulada del gateway segura para CI

[Depuración](./debugging.md)[Scripts](./scripts.md)