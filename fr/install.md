

  Aperçu de l'installation

  
# Installation

Vous avez déjà suivi le [Guide de démarrage](./start/getting-started.md) ? Tout est prêt — cette page est dédiée aux méthodes d'installation alternatives, aux instructions spécifiques aux plateformes et à la maintenance.

## Configuration système requise

-   **[Node 22+](./install/node.md)** (le [script d'installation](#install-methods) l'installera s'il est manquant)
-   macOS, Linux ou Windows
-   `pnpm` uniquement si vous compilez depuis les sources

> **ℹ️** Sur Windows, nous recommandons fortement d'exécuter OpenClaw sous [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install).

## Méthodes d'installation

> **💡** Le **script d'installation** est la méthode recommandée pour installer OpenClaw. Il gère la détection de Node, l'installation et l'intégration en une seule étape.

 

> **⚠️** Pour les hébergements VPS/cloud, évitez autant que possible les images "1-clic" des marketplaces tiers. Préférez une image de base propre du système d'exploitation (par exemple Ubuntu LTS), puis installez OpenClaw vous-même avec le script d'installation.

 

Télécharge le CLI, l'installe globalement via npm et lance l'assistant d'intégration.

C'est tout — le script gère la détection de Node, l'installation et l'intégration.Pour ignorer l'intégration et simplement installer le binaire :

Pour tous les drapeaux, variables d'environnement et options d'automatisation/CI, consultez [Fonctionnement interne de l'installateur](./install/installer.md).

Si vous avez déjà Node 22+ et préférez gérer l'installation vous-même :

Pour les contributeurs ou toute personne souhaitant exécuter à partir d'un dépôt local.

1

[

](#)

Cloner et compiler

Clonez le [dépôt OpenClaw](https://github.com/openclaw/openclaw) et compilez :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build
pnpm build
```

2

[

](#)

Lier le CLI

Rendez la commande `openclaw` disponible globalement :

```bash
pnpm link --global
```

Alternativement, ignorez le lien et exécutez les commandes via `pnpm openclaw ...` depuis l'intérieur du dépôt.

3

[

](#)

Lancer l'intégration

```bash
openclaw onboard --install-daemon
```

Pour des workflows de développement plus avancés, consultez [Configuration](./start/setup.md).

## Autres méthodes d'installation

## Après l'installation

Vérifiez que tout fonctionne :

```bash
openclaw doctor         # vérifier les problèmes de configuration
openclaw status         # état de la passerelle
openclaw dashboard      # ouvrir l'interface utilisateur dans le navigateur
```

Si vous avez besoin de chemins d'exécution personnalisés, utilisez :

-   `OPENCLAW_HOME` pour les chemins internes basés sur le répertoire personnel
-   `OPENCLAW_STATE_DIR` pour l'emplacement des données modifiables
-   `OPENCLAW_CONFIG_PATH` pour l'emplacement du fichier de configuration

Consultez [Variables d'environnement](./help/environment.md) pour l'ordre de priorité et les détails complets.

## Dépannage : openclaw introuvable

Diagnostic rapide :

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Si `$(npm prefix -g)/bin` (macOS/Linux) ou `$(npm prefix -g)` (Windows) **n'est pas** dans votre `$PATH`, votre shell ne peut pas trouver les binaires npm globaux (y compris `openclaw`).Correction — ajoutez-le à votre fichier de démarrage du shell (`~/.zshrc` ou `~/.bashrc`) :

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

Sur Windows, ajoutez la sortie de `npm prefix -g` à votre PATH.Ensuite, ouvrez un nouveau terminal (ou `rehash` dans zsh / `hash -r` dans bash).

## Mise à jour / Désinstallation

[Fonctionnement interne de l'installateur](./install/installer.md)

---