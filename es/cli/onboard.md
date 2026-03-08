

  Comandos CLI

  
# onboard

Asistente de incorporación interactivo (configuración de puerta de enlace local o remota).

## Guías relacionadas

-   Centro de incorporación CLI: [Asistente de incorporación (CLI)](../start/wizard.md)
-   Visión general de incorporación: [Visión general de incorporación](../start/onboarding-overview.md)
-   Referencia de incorporación CLI: [Referencia de incorporación CLI](../start/wizard-cli-reference.md)
-   Automatización CLI: [Automatización CLI](../start/wizard-cli-automation.md)
-   Incorporación en macOS: [Incorporación (App macOS)](../start/onboarding.md)

## Ejemplos

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url wss://gateway-host:18789
```

Para objetivos de red privada en texto plano `ws://` (solo redes confiables), establece `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` en el entorno del proceso de incorporación. Proveedor personalizado no interactivo:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --secret-input-mode plaintext \
  --custom-compatibility openai
```

`--custom-api-key` es opcional en modo no interactivo. Si se omite, la incorporación verifica `CUSTOM_API_KEY`. Almacena las claves del proveedor como referencias en lugar de texto plano:

```bash
openclaw onboard --non-interactive \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

Con `--secret-input-mode ref`, la incorporación escribe referencias respaldadas por entorno en lugar de valores de clave en texto plano. Para proveedores respaldados por perfil de autenticación, esto escribe entradas `keyRef`; para proveedores personalizados, esto escribe `models.providers..apiKey` como una referencia de entorno (por ejemplo `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`). Contrato del modo `ref` no interactivo:

-   Establece la variable de entorno del proveedor en el entorno del proceso de incorporación (por ejemplo `OPENAI_API_KEY`).
-   No pases banderas de clave en línea (por ejemplo `--openai-api-key`) a menos que esa variable de entorno también esté establecida.
-   Si se pasa una bandera de clave en línea sin la variable de entorno requerida, la incorporación falla rápidamente con orientación.

Opciones de token de puerta de enlace en modo no interactivo:

-   `--gateway-auth token --gateway-token ` almacena un token en texto plano.
-   `--gateway-auth token --gateway-token-ref-env ` almacena `gateway.auth.token` como un SecretRef de entorno.
-   `--gateway-token` y `--gateway-token-ref-env` son mutuamente excluyentes.
-   `--gateway-token-ref-env` requiere una variable de entorno no vacía en el entorno del proceso de incorporación.
-   Con `--install-daemon`, cuando la autenticación por token requiere un token, los tokens de puerta de enlace gestionados por SecretRef se validan pero no se persisten como texto plano resuelto en los metadatos del entorno del servicio supervisor.
-   Con `--install-daemon`, si el modo de token requiere un token y el SecretRef de token configurado no está resuelto, la incorporación falla cerrada con orientación de remediación.
-   Con `--install-daemon`, si tanto `gateway.auth.token` como `gateway.auth.password` están configurados y `gateway.auth.mode` no está establecido, la incorporación bloquea la instalación hasta que el modo se establece explícitamente.

Ejemplo:

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN \
  --accept-risk
```

Comportamiento de incorporación interactiva con modo de referencia:

-   Elige **Usar referencia de secreto** cuando se te solicite.
-   Luego elige:
    -   Variable de entorno
    -   Proveedor de secretos configurado (`file` o `exec`)
-   La incorporación realiza una validación de pre-vuelo rápida antes de guardar la referencia.
    -   Si la validación falla, la incorporación muestra el error y te permite reintentar.

Opciones de endpoint Z.AI no interactivas: Nota: `--auth-choice zai-api-key` ahora detecta automáticamente el mejor endpoint Z.AI para tu clave (prefiere la API general con `zai/glm-5`). Si específicamente quieres los endpoints del Plan de Codificación GLM, elige `zai-coding-global` o `zai-coding-cn`.

```bash
# Selección de endpoint sin solicitudes
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Otras opciones de endpoint Z.AI:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Ejemplo no interactivo de Mistral:

```bash
openclaw onboard --non-interactive \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY"
```

Notas de flujo:

-   `quickstart`: solicitudes mínimas, genera automáticamente un token de puerta de enlace.
-   `manual`: solicitudes completas para puerto/enlace/autenticación (alias de `advanced`).
-   Comportamiento del alcance DM en incorporación local: [Referencia de incorporación CLI](../start/wizard-cli-reference.md#outputs-and-internals).
-   Primer chat más rápido: `openclaw dashboard` (UI de Control, sin configuración de canal).
-   Proveedor personalizado: conecta cualquier endpoint compatible con OpenAI o Anthropic, incluidos proveedores alojados no listados. Usa Desconocido para auto-detectar.

## Comandos de seguimiento comunes

```bash
openclaw configure
openclaw agents add <nombre>
```

> **ℹ️** `--json` no implica modo no interactivo. Usa `--non-interactive` para scripts.

[nodes](./nodes.md)[pairing](./pairing.md)