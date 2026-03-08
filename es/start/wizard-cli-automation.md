

  Guías

  
# Automatización CLI

Usa `--non-interactive` para automatizar `openclaw onboard`.

> **ℹ️** `--json` no implica modo no interactivo. Usa `--non-interactive` (y `--workspace`) para scripts.

## Ejemplo base no interactivo

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --secret-input-mode plaintext \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Añade `--json` para un resumen legible por máquina. Usa `--secret-input-mode ref` para almacenar referencias respaldadas por variables de entorno en los perfiles de autenticación en lugar de valores en texto plano. La selección interactiva entre referencias de entorno y referencias de proveedor configuradas (`file` o `exec`) está disponible en el flujo del asistente de incorporación. En el modo `ref` no interactivo, las variables de entorno del proveedor deben estar configuradas en el entorno del proceso. Pasar banderas de clave en línea sin la variable de entorno correspondiente ahora falla rápidamente. Ejemplo:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

## Ejemplos específicos por proveedor

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-provider-id "my-custom" \
  --custom-compatibility anthropic \
  --gateway-port 18789 \
  --gateway-bind loopback
```

`--custom-api-key` es opcional. Si se omite, la incorporación verifica `CUSTOM_API_KEY`.Variante en modo ref:

```bash
export CUSTOM_API_KEY="your-key"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --secret-input-mode ref \
  --custom-provider-id "my-custom" \
  --custom-compatibility anthropic \
  --gateway-port 18789 \
  --gateway-bind loopback
```

En este modo, la incorporación almacena `apiKey` como `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`.

## Añadir otro agente

Usa `openclaw agents add ` para crear un agente separado con su propio espacio de trabajo, sesiones y perfiles de autenticación. Ejecutar sin `--workspace` inicia el asistente.

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

Lo que configura:

-   `agents.list[].name`
-   `agents.list[].workspace`
-   `agents.list[].agentDir`

Notas:

-   Los espacios de trabajo predeterminados siguen `~/.openclaw/workspace-`.
-   Añade `bindings` para enrutar mensajes entrantes (el asistente puede hacer esto).
-   Banderas no interactivas: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## Documentación relacionada

-   Centro de incorporación: [Asistente de Incorporación (CLI)](./wizard.md)
-   Referencia completa: [Referencia de Incorporación CLI](./wizard-cli-reference.md)
-   Referencia de comandos: [`openclaw onboard`](../cli/onboard.md)

[Referencia CLI](./wizard-cli-reference.md)