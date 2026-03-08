

  Premières étapes

  
# Assistant d'intégration (CLI)

L'assistant d'intégration est la méthode **recommandée** pour configurer OpenClaw sur macOS, Linux ou Windows (via WSL2 ; fortement recommandé). Il configure une passerelle locale ou une connexion à une passerelle distante, ainsi que les canaux, les compétences et les paramètres par défaut de l'espace de travail en un seul flux guidé.

```bash
openclaw onboard
```

> **ℹ️** Premier chat le plus rapide : ouvrez l'interface de contrôle (aucune configuration de canal nécessaire). Exécutez `openclaw dashboard` et discutez dans le navigateur. Docs : [Tableau de bord](../web/dashboard.md).

 Pour reconfigurer plus tard :

```bash
openclaw configure
openclaw agents add <name>
```

> **ℹ️** `--json` n'implique pas le mode non interactif. Pour les scripts, utilisez `--non-interactive`.

 

> **💡** L'assistant d'intégration inclut une étape de recherche web où vous pouvez choisir un fournisseur (Perplexity, Brave, Gemini, Grok, ou Kimi) et coller votre clé API pour que l'agent puisse utiliser `web_search`. Vous pouvez également configurer cela plus tard avec `openclaw configure --section web`. Docs : [Outils web](../tools/web.md).

## Démarrage rapide vs Avancé

L'assistant commence par **Démarrage rapide** (valeurs par défaut) vs **Avancé** (contrôle total). 

-   Passerelle locale (loopback)
-   Espace de travail par défaut (ou espace de travail existant)
-   Port de la passerelle **18789**
-   Authentification de la passerelle **Token** (auto‑généré, même en loopback)
-   Politique d'outils par défaut pour les nouvelles installations locales : `tools.profile: "coding"` (un profil explicite existant est préservé)
-   Isolation des messages directs par défaut : l'intégration locale écrit `session.dmScope: "per-channel-peer"` lorsqu'il n'est pas défini. Détails : [Référence de l'intégration CLI](./wizard-cli-reference.md#outputs-and-internals)
-   Exposition Tailscale **Désactivée**
-   Les messages directs Telegram et WhatsApp passent par défaut en **liste autorisée** (vous serez invité à saisir votre numéro de téléphone)

-   Expose chaque étape (mode, espace de travail, passerelle, canaux, démon, compétences).

## Ce que l'assistant configure

**Le mode local (par défaut)** vous guide à travers ces étapes :

1.  **Modèle/Authentification** — choisissez n'importe quel fournisseur/flux d'authentification pris en charge (clé API, OAuth, ou jeton de configuration), y compris Fournisseur personnalisé (compatible OpenAI, compatible Anthropic, ou Détection automatique inconnue). Choisissez un modèle par défaut. Note de sécurité : si cet agent exécutera des outils ou traitera du contenu webhook/hooks, préférez le modèle de dernière génération le plus puissant disponible et gardez la politique d'outils stricte. Les modèles plus faibles/anciens sont plus faciles à injecter par prompt. Pour les exécutions non interactives, `--secret-input-mode ref` stocke des références basées sur des variables d'environnement dans les profils d'authentification au lieu des valeurs de clé API en texte clair. En mode non interactif `ref`, la variable d'environnement du fournisseur doit être définie ; passer des drapeaux de clé en ligne sans cette variable d'environnement échoue rapidement. Dans les exécutions interactives, choisir le mode de référence de secret vous permet de pointer vers une variable d'environnement ou une référence de fournisseur configurée (`file` ou `exec`), avec une validation préalable rapide avant l'enregistrement.
2.  **Espace de travail** — Emplacement des fichiers de l'agent (par défaut `~/.openclaw/workspace`). Initialise les fichiers d'amorçage.
3.  **Passerelle** — Port, adresse de liaison, mode d'authentification, exposition Tailscale. En mode interactif avec jeton, choisissez le stockage par défaut du jeton en texte clair ou optez pour SecretRef. Chemin SecretRef du jeton en mode non interactif : `--gateway-token-ref-env <ENV_VAR>`.
4.  **Canaux** — WhatsApp, Telegram, Discord, Google Chat, Mattermost, Signal, BlueBubbles, ou iMessage.
5.  **Démon** — Installe un LaunchAgent (macOS) ou une unité utilisateur systemd (Linux/WSL2). Si l'authentification par jeton nécessite un jeton et que `gateway.auth.token` est géré par SecretRef, l'installation du démon le valide mais ne persiste pas le jeton résolu dans les métadonnées d'environnement du service superviseur. Si l'authentification par jeton nécessite un jeton et que le SecretRef configuré n'est pas résolu, l'installation du démon est bloquée avec des instructions actionnables. Si `gateway.auth.token` et `gateway.auth.password` sont tous deux configurés et que `gateway.auth.mode` n'est pas défini, l'installation du démon est bloquée jusqu'à ce que le mode soit défini explicitement.
6.  **Vérification de santé** — Démarre la Passerelle et vérifie qu'elle fonctionne.
7.  **Compétences** — Installe les compétences recommandées et les dépendances optionnelles.

> **ℹ️** Réexécuter l'assistant **ne** supprime rien sauf si vous choisissez explicitement **Réinitialiser** (ou passez `--reset`). L'option CLI `--reset` concerne par défaut la configuration, les identifiants et les sessions ; utilisez `--reset-scope full` pour inclure l'espace de travail. Si la configuration est invalide ou contient des clés obsolètes, l'assistant vous demande d'exécuter d'abord `openclaw doctor`.

 **Le mode distant** configure uniquement le client local pour se connecter à une Passerelle ailleurs. Il n'installe **pas** et ne modifie rien sur l'hôte distant.

## Ajouter un autre agent

Utilisez `openclaw agents add ` pour créer un agent séparé avec son propre espace de travail, ses propres sessions et ses profils d'authentification. L'exécution sans `--workspace` lance l'assistant. Ce qu'il définit :

-   `agents.list[].name`
-   `agents.list[].workspace`
-   `agents.list[].agentDir`

Notes :

-   Les espaces de travail par défaut suivent `~/.openclaw/workspace-`.
-   Ajoutez `bindings` pour router les messages entrants (l'assistant peut le faire).
-   Drapeaux non interactifs : `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## Référence complète

Pour des descriptions détaillées étape par étape, des scripts non interactifs, la configuration de Signal, l'API RPC et une liste complète des champs de configuration que l'assistant écrit, consultez la [Référence de l'assistant](../reference/wizard.md).

## Documentation associée

-   Référence des commandes CLI : [`openclaw onboard`](../cli/onboard.md)
-   Vue d'ensemble de l'intégration : [Vue d'ensemble de l'intégration](./onboarding-overview.md)
-   Intégration de l'application macOS : [Intégration](./onboarding.md)
-   Rituel de premier démarrage de l'agent : [Amorçage de l'agent](./bootstrapping.md)

[Vue d'ensemble de l'intégration](./onboarding-overview.md)[Intégration : Application macOS](./onboarding.md)

---