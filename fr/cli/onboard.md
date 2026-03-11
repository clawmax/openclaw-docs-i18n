

  Commandes CLI

  
# onboard

Assistant d'intégration interactif (configuration de passerelle locale ou distante).

## Guides associés

-   Centre d'intégration CLI : [Assistant d'intégration (CLI)](../start/wizard.md)
-   Vue d'ensemble de l'intégration : [Vue d'ensemble de l'intégration](../start/onboarding-overview.md)
-   Référence d'intégration CLI : [Référence d'intégration CLI](../start/wizard-cli-reference.md)
-   Automatisation CLI : [Automatisation CLI](../start/wizard-cli-automation.md)
-   Intégration macOS : [Intégration (Application macOS)](../start/onboarding.md)

## Exemples

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url wss://gateway-host:18789
```

Pour les cibles en texte clair sur réseau privé `ws://` (réseaux de confiance uniquement), définissez `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` dans l'environnement du processus d'intégration. Fournisseur personnalisé non interactif :

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --secret-input-mode plaintext \
  --custom-compatibility openai
```

`--custom-api-key` est facultatif en mode non interactif. S'il est omis, l'intégration vérifie `CUSTOM_API_KEY`. Stockez les clés des fournisseurs sous forme de références au lieu de texte clair :

```bash
openclaw onboard --non-interactive \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

Avec `--secret-input-mode ref`, l'intégration écrit des références basées sur des variables d'environnement au lieu des valeurs de clé en texte clair. Pour les fournisseurs basés sur un profil d'authentification, cela écrit des entrées `keyRef` ; pour les fournisseurs personnalisés, cela écrit `models.providers..apiKey` comme une référence d'environnement (par exemple `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`). Contrat du mode `ref` non interactif :

-   Définissez la variable d'environnement du fournisseur dans l'environnement du processus d'intégration (par exemple `OPENAI_API_KEY`).
-   Ne passez pas d'options de clé en ligne (par exemple `--openai-api-key`) à moins que cette variable d'environnement ne soit également définie.
-   Si une option de clé en ligne est passée sans la variable d'environnement requise, l'intégration échoue rapidement avec des instructions.

Options de jeton de passerelle en mode non interactif :

-   `--gateway-auth token --gateway-token ` stocke un jeton en texte clair.
-   `--gateway-auth token --gateway-token-ref-env ` stocke `gateway.auth.token` comme une SecretRef d'environnement.
-   `--gateway-token` et `--gateway-token-ref-env` s'excluent mutuellement.
-   `--gateway-token-ref-env` nécessite une variable d'environnement non vide dans l'environnement du processus d'intégration.
-   Avec `--install-daemon`, lorsque l'authentification par jeton nécessite un jeton, les jetons de passerelle gérés par SecretRef sont validés mais ne sont pas persistés sous forme de texte clair résolu dans les métadonnées d'environnement du service superviseur.
-   Avec `--install-daemon`, si le mode jeton nécessite un jeton et que la SecretRef configurée n'est pas résolue, l'intégration échoue en mode fermé avec des instructions de correction.
-   Avec `--install-daemon`, si `gateway.auth.token` et `gateway.auth.password` sont tous deux configurés et que `gateway.auth.mode` n'est pas défini, l'intégration bloque l'installation jusqu'à ce que le mode soit défini explicitement.

Exemple :

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN \
  --accept-risk
```

Comportement de l'intégration interactive avec le mode référence :

-   Choisissez **Utiliser une référence de secret** lorsque vous y êtes invité.
-   Ensuite, choisissez soit :
    -   Variable d'environnement
    -   Fournisseur de secret configuré (`file` ou `exec`)
-   L'intégration effectue une validation préalable rapide avant d'enregistrer la référence.
    -   Si la validation échoue, l'intégration affiche l'erreur et vous permet de réessayer.

Choix de point de terminaison Z.AI non interactif : Remarque : `--auth-choice zai-api-key` détecte automatiquement le meilleur point de terminaison Z.AI pour votre clé (préfère l'API générale avec `zai/glm-5`). Si vous voulez spécifiquement les points de terminaison du plan de codage GLM, choisissez `zai-coding-global` ou `zai-coding-cn`.

```bash
# Sélection de point de terminaison sans invite
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Autres choix de points de terminaison Z.AI :
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Exemple non interactif Mistral :

```bash
openclaw onboard --non-interactive \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY"
```

Notes sur les flux :

-   `quickstart` : invites minimales, génère automatiquement un jeton de passerelle.
-   `manual` : invites complètes pour le port/la liaison/l'authentification (alias de `advanced`).
-   Comportement de la portée DM de l'intégration locale : [Référence d'intégration CLI](../start/wizard-cli-reference.md#outputs-and-internals).
-   Premier chat le plus rapide : `openclaw dashboard` (Interface de contrôle, pas de configuration de canal).
-   Fournisseur personnalisé : connectez n'importe quel point de terminaison compatible OpenAI ou Anthropic, y compris les fournisseurs hébergés non listés. Utilisez Inconnu pour une détection automatique.

## Commandes de suivi courantes

```bash
openclaw configure
openclaw agents add <name>
```

> **ℹ️** `--json` n'implique pas le mode non interactif. Utilisez `--non-interactive` pour les scripts.

[nodes](./nodes.md)[pairing](./pairing.md)