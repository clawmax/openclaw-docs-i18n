

  Commandes CLI

  
# models

Découverte, analyse et configuration des modèles (modèle par défaut, modèles de secours, profils d'authentification). Liens utiles :

-   Fournisseurs + modèles : [Models](../providers/models.md)
-   Configuration de l'authentification des fournisseurs : [Getting started](../start/getting-started.md)

## Commandes courantes

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` affiche le modèle par défaut et les modèles de secours résolus ainsi qu'un aperçu de l'authentification. Lorsque des instantanés d'utilisation des fournisseurs sont disponibles, la section d'état OAuth/jeton inclut des en-têtes d'utilisation des fournisseurs. Ajoutez `--probe` pour exécuter des sondages d'authentification en direct sur chaque profil de fournisseur configuré. Les sondages sont de vraies requêtes (peuvent consommer des jetons et déclencher des limites de débit). Utilisez `--agent ` pour inspecter l'état du modèle et de l'authentification d'un agent configuré. Lorsqu'il est omis, la commande utilise `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` s'il est défini, sinon l'agent par défaut configuré. Notes :

-   `models set <model-or-alias>` accepte `provider/model` ou un alias.
-   Les références de modèle sont analysées en divisant sur le **premier** `/`. Si l'ID du modèle contient un `/` (style OpenRouter), incluez le préfixe du fournisseur (exemple : `openrouter/moonshotai/kimi-k2`).
-   Si vous omettez le fournisseur, OpenClaw traite l'entrée comme un alias ou un modèle pour le **fournisseur par défaut** (fonctionne uniquement lorsqu'il n'y a pas de `/` dans l'ID du modèle).
-   `models status` peut afficher `marker()` dans la sortie d'authentification pour les espaces réservés non secrets (par exemple `OPENAI_API_KEY`, `secretref-managed`, `minimax-oauth`, `qwen-oauth`, `ollama-local`) au lieu de les masquer comme des secrets.

### models status

Options :

-   `--json`
-   `--plain`
-   `--check` (sortie 1=expiré/absent, 2=expirant)
-   `--probe` (sondage en direct des profils d'authentification configurés)
-   `--probe-provider ` (sonder un fournisseur)
-   `--probe-profile ` (répéter ou liste d'IDs de profils séparés par des virgules)
-   `--probe-timeout `
-   `--probe-concurrency `
-   `--probe-max-tokens `
-   `--agent ` (ID de l'agent configuré ; remplace `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`)

## Alias + modèles de secours

```bash
openclaw models aliases list
openclaw models fallbacks list
```

## Profils d'authentification

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` exécute le flux d'authentification d'un plugin de fournisseur (OAuth/clé API). Utilisez `openclaw plugins list` pour voir quels fournisseurs sont installés. Notes :

-   `setup-token` demande une valeur de jeton de configuration (générez-le avec `claude setup-token` sur n'importe quelle machine).
-   `paste-token` accepte une chaîne de jeton générée ailleurs ou via une automatisation.
-   Note sur la politique d'Anthropic : la prise en charge du jeton de configuration est une compatibilité technique. Anthropic a bloqué certaines utilisations d'abonnement en dehors de Claude Code par le passé, vérifiez donc les conditions actuelles avant de l'utiliser largement.

[message](./message.md)[node](./node.md)