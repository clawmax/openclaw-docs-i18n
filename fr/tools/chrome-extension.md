

  Navigateur

  
# Extension Chrome

L'extension Chrome OpenClaw permet à l'agent de contrôler **vos onglets Chrome existants** (votre fenêtre Chrome normale) au lieu de lancer un profil Chrome distinct géré par openclaw. L'attachement/détachement se fait via **un seul bouton de la barre d'outils Chrome**.

## Ce que c'est (concept)

Il y a trois parties :

-   **Service de contrôle du navigateur** (Passerelle ou nœud) : l'API que l'agent/l'outil appelle (via la Passerelle)
-   **Serveur relais local** (CDP en boucle locale) : fait le pont entre le serveur de contrôle et l'extension (`http://127.0.0.1:18792` par défaut)
-   **Extension Chrome MV3** : s'attache à l'onglet actif en utilisant `chrome.debugger` et achemine les messages CDP vers le relais

OpenClaw contrôle ensuite l'onglet attaché via la surface normale de l'outil `browser` (en sélectionnant le bon profil).

## Installer / charger (non empaquetée)

1.  Installez l'extension dans un chemin local stable :

```bash
openclaw browser extension install
```

2.  Affichez le chemin du répertoire d'installation de l'extension :

```bash
openclaw browser extension path
```

3.  Chrome → `chrome://extensions`

-   Activez le "Mode développeur"
-   "Charger l'extension non empaquetée" → sélectionnez le répertoire affiché ci-dessus

4.  Épinglez l'extension.

## Mises à jour (pas d'étape de compilation)

L'extension est fournie dans la version d'OpenClaw (package npm) sous forme de fichiers statiques. Il n'y a pas d'étape de "compilation" séparée. Après une mise à niveau d'OpenClaw :

-   Ré-exécutez `openclaw browser extension install` pour rafraîchir les fichiers installés dans votre répertoire d'état OpenClaw.
-   Chrome → `chrome://extensions` → cliquez sur "Recharger" pour l'extension.

## Utilisation (définir le jeton de passerelle une fois)

OpenClaw est livré avec un profil de navigateur intégré nommé `chrome` qui cible le relais de l'extension sur le port par défaut. Avant le premier attachement, ouvrez les Options de l'extension et définissez :

-   `Port` (par défaut `18792`)
-   `Jeton de passerelle` (doit correspondre à `gateway.auth.token` / `OPENCLAW_GATEWAY_TOKEN`)

Utilisez-le :

-   CLI : `openclaw browser --browser-profile chrome tabs`
-   Outil agent : `browser` avec `profile="chrome"`

Si vous voulez un nom différent ou un port de relais différent, créez votre propre profil :

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

### Ports de Passerelle personnalisés

Si vous utilisez un port de passerelle personnalisé, le port du relais de l'extension est automatiquement dérivé : **Port du Relais d'Extension = Port de la Passerelle + 3** Exemple : si `gateway.port: 19001`, alors :

-   Port du relais d'extension : `19004` (passerelle + 3)

Configurez l'extension pour utiliser le port de relais dérivé dans la page Options de l'extension.

## Attacher / détacher (bouton de barre d'outils)

-   Ouvrez l'onglet que vous voulez qu'OpenClaw contrôle.
-   Cliquez sur l'icône de l'extension.
    -   Le badge affiche `ON` lorsqu'il est attaché.
-   Cliquez à nouveau pour détacher.

## Quel onglet contrôle-t-il ?

-   Il ne contrôle **pas** automatiquement "n'importe quel onglet que vous regardez".
-   Il contrôle **uniquement l'onglet (ou les onglets) que vous avez explicitement attaché(s)** en cliquant sur le bouton de la barre d'outils.
-   Pour changer : ouvrez l'autre onglet et cliquez sur l'icône de l'extension là-bas.

## Badge + erreurs courantes

-   `ON` : attaché ; OpenClaw peut piloter cet onglet.
-   `…` : connexion au relais local.
-   `!` : relais inaccessible/non authentifié (le plus souvent : serveur relais non démarré, ou jeton de passerelle manquant/incorrect).

Si vous voyez `!` :

-   Assurez-vous que la Passerelle fonctionne localement (configuration par défaut), ou exécutez un hôte nœud sur cette machine si la Passerelle fonctionne ailleurs.
-   Ouvrez la page Options de l'extension ; elle valide l'accessibilité du relais + l'authentification par jeton de passerelle.

## Passerelle distante (utiliser un hôte nœud)

### Passerelle locale (même machine que Chrome) — généralement aucune étape supplémentaire

Si la Passerelle fonctionne sur la même machine que Chrome, elle démarre le service de contrôle du navigateur en boucle locale et démarre automatiquement le serveur relais. L'extension communique avec le relais local ; les appels CLI/outils vont vers la Passerelle.

### Passerelle distante (Passerelle fonctionne ailleurs) — exécutez un hôte nœud

Si votre Passerelle fonctionne sur une autre machine, démarrez un hôte nœud sur la machine qui exécute Chrome. La Passerelle proxyfera les actions du navigateur vers ce nœud ; l'extension + le relais restent locaux à la machine du navigateur. Si plusieurs nœuds sont connectés, épinglez-en un avec `gateway.nodes.browser.node` ou définissez `gateway.nodes.browser.mode`.

## Sandboxing (conteneurs d'outils)

Si votre session d'agent est sandboxée (`agents.defaults.sandbox.mode != "off"`), l'outil `browser` peut être restreint :

-   Par défaut, les sessions sandboxées ciblent souvent le **navigateur sandbox** (`target="sandbox"`), et non votre Chrome hôte.
-   La prise de contrôle par le relais d'extension Chrome nécessite de contrôler le serveur de contrôle du navigateur **hôte**.

Options :

-   Plus simple : utilisez l'extension depuis une session/agent **non sandboxée**.
-   Ou autorisez le contrôle du navigateur hôte pour les sessions sandboxées :

```json
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Assurez-vous ensuite que l'outil n'est pas refusé par la politique des outils, et (si nécessaire) appelez `browser` avec `target="host"`. Débogage : `openclaw sandbox explain`

## Conseils pour l'accès à distance

-   Gardez la Passerelle et l'hôte nœud sur le même tailnet ; évitez d'exposer les ports du relais au LAN ou à l'Internet public.
-   Appariez les nœuds intentionnellement ; désactivez le routage proxy du navigateur si vous ne voulez pas de contrôle à distance (`gateway.nodes.browser.mode="off"`).

## Fonctionnement du "chemin de l'extension"

`openclaw browser extension path` affiche le répertoire **installé** sur le disque contenant les fichiers de l'extension. Le CLI n'affiche **pas** intentionnellement un chemin `node_modules`. Exécutez toujours d'abord `openclaw browser extension install` pour copier l'extension vers un emplacement stable sous votre répertoire d'état OpenClaw. Si vous déplacez ou supprimez ce répertoire d'installation, Chrome marquera l'extension comme cassée jusqu'à ce que vous la rechargiez depuis un chemin valide.

## Implications de sécurité (lisez ceci)

C'est puissant et risqué. Considérez cela comme donner au modèle "les mains sur votre navigateur".

-   L'extension utilise l'API de débogage de Chrome (`chrome.debugger`). Lorsqu'elle est attachée, le modèle peut :
    -   cliquer/taper/naviguer dans cet onglet
    -   lire le contenu de la page
    -   accéder à tout ce à quoi la session connectée de l'onglet peut accéder
-   **Ce n'est pas isolé** comme le profil dédié géré par openclaw.
    -   Si vous vous attachez à votre profil/onglet quotidien, vous accordez l'accès à cet état de compte.

Recommandations :

-   Préférez un profil Chrome dédié (séparé de votre navigation personnelle) pour l'utilisation du relais d'extension.
-   Gardez la Passerelle et tout hôte nœud uniquement sur le tailnet ; comptez sur l'authentification de la Passerelle + l'appariement des nœuds.
-   Évitez d'exposer les ports du relais sur le LAN (`0.0.0.0`) et évitez Funnel (public).
-   Le relais bloque les origines non-extension et nécessite une authentification par jeton de passerelle pour `/cdp` et `/extension`.

Liens connexes :

-   Vue d'ensemble de l'outil navigateur : [Navigateur](./browser.md)
-   Audit de sécurité : [Sécurité](../gateway/security.md)
-   Configuration Tailscale : [Tailscale](../gateway/tailscale.md)

[Connexion au navigateur](./browser-login.md)[Dépannage du navigateur](./browser-linux-troubleshooting.md)