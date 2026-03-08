

  Premiers pas

  
# Premiers pas

Objectif : passer de zéro à un premier chat fonctionnel avec une configuration minimale.

> **ℹ️** Chat le plus rapide : ouvrez l'Interface de Contrôle (aucune configuration de canal nécessaire). Exécutez `openclaw dashboard` et discutez dans le navigateur, ou ouvrez `http://127.0.0.1:18789/` sur l'hôte de la passerelle. Docs : [Tableau de bord](../web/dashboard.md) et [Interface de Contrôle](../web/control-ui.md).

## Prérequis

-   Node 22 ou plus récent

> **💡** Vérifiez votre version de Node avec `node --version` si vous n'êtes pas sûr.

## Configuration rapide (CLI)

### Étape 1 : Installer OpenClaw (recommandé)

> **ℹ️** Autres méthodes d'installation et prérequis : [Installation](../install.md).

### Étape 2 : Exécuter l'assistant de configuration

```bash
openclaw onboard --install-daemon
```

L'assistant configure l'authentification, les paramètres de la passerelle et les canaux optionnels. Voir [Assistant de configuration](./wizard.md) pour plus de détails.

### Étape 3 : Vérifier la Passerelle

Si vous avez installé le service, il devrait déjà être en cours d'exécution :

```bash
openclaw gateway status
```

### Étape 4 : Ouvrir l'Interface de Contrôle

```bash
openclaw dashboard
```

 

> **✅** Si l'Interface de Contrôle se charge, votre Passerelle est prête à l'emploi.

## Vérifications et extras optionnels

Utile pour des tests rapides ou du dépannage.

```bash
openclaw gateway --port 18789
```

Nécessite un canal configuré.

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

## Variables d'environnement utiles

Si vous exécutez OpenClaw avec un compte de service ou souhaitez des emplacements personnalisés pour la configuration/état :

-   `OPENCLAW_HOME` définit le répertoire personnel utilisé pour la résolution des chemins internes.
-   `OPENCLAW_STATE_DIR` remplace le répertoire d'état.
-   `OPENCLAW_CONFIG_PATH` remplace le chemin du fichier de configuration.

Référence complète des variables d'environnement : [Variables d'environnement](../help/environment.md).

## Approfondir

## Ce que vous aurez

-   Une Passerelle en cours d'exécution
-   L'authentification configurée
-   L'accès à l'Interface de Contrôle ou un canal connecté

## Prochaines étapes

-   Sécurité et approbations des messages directs : [Appairage](../channels/pairing.md)
-   Connecter plus de canaux : [Canaux](../channels.md)
-   Flux de travail avancés et depuis les sources : [Configuration](./setup.md)

[Fonctionnalités](../concepts/features.md)[Vue d'ensemble de la configuration initiale](./onboarding-overview.md)

---