title: "Guide d'automatisation CLI OpenClaw pour l'intégration non interactive"
description: "Apprenez à automatiser l'assistant d'intrégration OpenClaw en utilisant le drapeau --non-interactive. Obtenez des exemples pour Anthropic, OpenAI, Gemini et des fournisseurs personnalisés."
keywords: ["openclaw cli", "intégration non interactive", "automatiser openclaw", "automatisation cli", "commande onboard", "configuration clé api", "automatisation agent", "interface en ligne de commande"]
---

  Guides

  
# Automatisation CLI

Utilisez `--non-interactive` pour automatiser `openclaw onboard`.

> **ℹ️** `--json` n'implique pas le mode non interactif. Utilisez `--non-interactive` (et `--workspace`) pour les scripts.

## Exemple de base non interactif

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

Ajoutez `--json` pour un résumé lisible par machine. Utilisez `--secret-input-mode ref` pour stocker des références basées sur des variables d'environnement dans les profils d'authentification au lieu de valeurs en texte clair. La sélection interactive entre les références d'environnement et les références de fournisseur configurées (`file` ou `exec`) est disponible dans le flux de l'assistant d'intégration. En mode `ref` non interactif, les variables d'environnement du fournisseur doivent être définies dans l'environnement du processus. Passer des drapeaux de clé en ligne sans la variable d'environnement correspondante échoue maintenant rapidement. Exemple :

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

## Exemples spécifiques aux fournisseurs

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

`--custom-api-key` est optionnel. S'il est omis, l'intégration vérifie `CUSTOM_API_KEY`. Variante en mode ref :

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

Dans ce mode, l'intégration stocke `apiKey` comme `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`.

## Ajouter un autre agent

Utilisez `openclaw agents add ` pour créer un agent séparé avec son propre espace de travail, ses sessions et ses profils d'authentification. L'exécution sans `--workspace` lance l'assistant.

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

Ce qu'il configure :

-   `agents.list[].name`
-   `agents.list[].workspace`
-   `agents.list[].agentDir`

Notes :

-   Les espaces de travail par défaut suivent `~/.openclaw/workspace-`.
-   Ajoutez `bindings` pour router les messages entrants (l'assistant peut le faire).
-   Drapeaux non interactifs : `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## Documentation associée

-   Hub d'intégration : [Assistant d'intégration (CLI)](./wizard.md)
-   Référence complète : [Référence CLI d'intégration](./wizard-cli-reference.md)
-   Référence des commandes : [`openclaw onboard`](../cli/onboard.md)

[Référence CLI](./wizard-cli-reference.md)

---