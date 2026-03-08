

  Fondamentaux

  
# OAuth

OpenClaw prend en charge l'« authentification par abonnement » via OAuth pour les fournisseurs qui l'offrent (notamment **OpenAI Codex (ChatGPT OAuth)**). Pour les abonnements Anthropic, utilisez le flux **setup-token**. L'utilisation d'abonnements Anthropic en dehors de Claude Code a été restreinte par le passé pour certains utilisateurs, considérez-la donc comme un risque lié au choix de l'utilisateur et vérifiez vous-même la politique actuelle d'Anthropic. L'OAuth OpenAI Codex est explicitement pris en charge pour une utilisation dans des outils externes comme OpenClaw. Cette page explique : Pour Anthropic en production, l'authentification par clé API est le chemin recommandé plus sûr par rapport à l'authentification par setup-token d'abonnement.

-   comment fonctionne l'**échange de jetons** OAuth (PKCE)
-   où les jetons sont **stockés** (et pourquoi)
-   comment gérer **plusieurs comptes** (profils + remplacements par session)

OpenClaw prend également en charge les **plugins de fournisseurs** qui embarquent leurs propres flux OAuth ou par clé API. Exécutez-les via :

```bash
openclaw models auth login --provider <id>
```

## Le puits à jetons (pourquoi il existe)

Les fournisseurs OAuth génèrent couramment un **nouveau jeton de rafraîchissement** lors des flux de connexion/rafraîchissement. Certains fournisseurs (ou clients OAuth) peuvent invalider les anciens jetons de rafraîchissement lorsqu'un nouveau est émis pour le même utilisateur/application. Symptôme pratique :

-   vous vous connectez via OpenClaw *et* via Claude Code / Codex CLI → l'un d'eux se retrouve aléatoirement « déconnecté » plus tard

Pour réduire cela, OpenClaw traite `auth-profiles.json` comme un **puits à jetons** :

-   l'exécution lit les identifiants depuis **un seul endroit**
-   nous pouvons conserver plusieurs profils et les router de manière déterministe

## Stockage (où vivent les jetons)

Les secrets sont stockés **par agent** :

-   Profils d'authentification (OAuth + clés API + références optionnelles au niveau valeur) : `~/.openclaw/agents//agent/auth-profiles.json`
-   Fichier de compatibilité hérité : `~/.openclaw/agents//agent/auth.json` (les entrées statiques `api_key` sont nettoyées lorsqu'elles sont découvertes)

Fichier d'importation hérité uniquement (toujours pris en charge, mais pas le stockage principal) :

-   `~/.openclaw/credentials/oauth.json` (importé dans `auth-profiles.json` lors de la première utilisation)

Tous les éléments ci-dessus respectent également `$OPENCLAW_STATE_DIR` (remplacement du répertoire d'état). Référence complète : [/gateway/configuration](../gateway/configuration.md#auth-storage-oauth--api-keys) Pour les références statiques de secrets et le comportement d'activation des instantanés d'exécution, consultez [Gestion des secrets](../gateway/secrets.md).

## Setup-token Anthropic (authentification par abonnement)

> **⚠️** La prise en charge du setup-token Anthropic est une compatibilité technique, pas une garantie de politique. Anthropic a bloqué par le passé certaines utilisations d'abonnement en dehors de Claude Code. Décidez vous-même d'utiliser ou non l'authentification par abonnement, et vérifiez les conditions actuelles d'Anthropic.

 Exécutez `claude setup-token` sur n'importe quelle machine, puis collez-le dans OpenClaw :

```bash
openclaw models auth setup-token --provider anthropic
```

Si vous avez généré le jeton ailleurs, collez-le manuellement :

```bash
openclaw models auth paste-token --provider anthropic
```

Vérifiez :

```bash
openclaw models status
```

## Échange OAuth (comment fonctionne la connexion)

Les flux de connexion interactifs d'OpenClaw sont implémentés dans `@mariozechner/pi-ai` et intégrés aux assistants/commandes.

### Setup-token Anthropic

Forme du flux :

1.  exécutez `claude setup-token`
2.  collez le jeton dans OpenClaw
3.  stockez-le comme un profil d'authentification par jeton (pas de rafraîchissement)

Le chemin de l'assistant est `openclaw onboard` → choix d'authentification `setup-token` (Anthropic).

### OpenAI Codex (ChatGPT OAuth)

L'OAuth OpenAI Codex est explicitement pris en charge pour une utilisation en dehors de la CLI Codex, y compris les flux de travail OpenClaw. Forme du flux (PKCE) :

1.  générer le vérifiant/défi PKCE + un `state` aléatoire
2.  ouvrir `https://auth.openai.com/oauth/authorize?...`
3.  tenter de capturer le rappel sur `http://127.0.0.1:1455/auth/callback`
4.  si le rappel ne peut pas se lier (ou vous êtes à distance/sans interface), collez l'URL/le code de redirection
5.  échanger sur `https://auth.openai.com/oauth/token`
6.  extraire `accountId` du jeton d'accès et stocker `{ access, refresh, expires, accountId }`

Le chemin de l'assistant est `openclaw onboard` → choix d'authentification `openai-codex`.

## Rafraîchissement + expiration

Les profils stockent un horodatage `expires`. À l'exécution :

-   si `expires` est dans le futur → utilisez le jeton d'accès stocké
-   si expiré → rafraîchissez (sous un verrou de fichier) et écrasez les identifiants stockés

Le flux de rafraîchissement est automatique ; vous n'avez généralement pas besoin de gérer les jetons manuellement.

## Comptes multiples (profils) + routage

Deux modèles :

### 1) Préféré : agents séparés

Si vous voulez que « personnel » et « travail » n'interagissent jamais, utilisez des agents isolés (sessions + identifiants + espace de travail séparés) :

```bash
openclaw agents add work
openclaw agents add personal
```

Configurez ensuite l'authentification par agent (assistant) et routez les conversations vers le bon agent.

### 2) Avancé : plusieurs profils dans un seul agent

`auth-profiles.json` prend en charge plusieurs identifiants de profil pour le même fournisseur. Choisissez quel profil est utilisé :

-   globalement via l'ordre de configuration (`auth.order`)
-   par session via `/model ...@`

Exemple (remplacement de session) :

-   `/model Opus@anthropic:work`

Comment voir quels identifiants de profil existent :

-   `openclaw channels list --json` (affiche `auth[]`)

Documentation associée :

-   [/concepts/model-failover](./model-failover.md) (règles de rotation et de refroidissement)
-   [/tools/slash-commands](../tools/slash-commands.md) (surface de commande)

[Espace de travail de l'agent](./agent-workspace.md)[Amorçage](../start/bootstrapping.md)