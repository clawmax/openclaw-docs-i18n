

  Hébergement et déploiement

  
# VMs macOS

## Configuration par défaut recommandée (la plupart des utilisateurs)

-   **Petit VPS Linux** pour une passerelle (Gateway) toujours active et un faible coût. Voir [Hébergement VPS](../vps.md).
-   **Matériel dédié** (Mac mini ou machine Linux) si vous voulez un contrôle total et une **IP résidentielle** pour l'automatisation du navigateur. De nombreux sites bloquent les IP de centres de données, donc la navigation locale fonctionne souvent mieux.
-   **Hybride :** gardez la passerelle sur un VPS bon marché, et connectez votre Mac en tant que **nœud** lorsque vous avez besoin de l'automatisation navigateur/interface utilisateur. Voir [Nœuds](../nodes.md) et [Passerelle distante](../gateway/remote.md).

Utilisez une VM macOS lorsque vous avez spécifiquement besoin de capacités exclusives à macOS (iMessage/BlueBubbles) ou que vous voulez une isolation stricte par rapport à votre Mac quotidien.

## Options de VM macOS

### VM locale sur votre Mac Apple Silicon (Lume)

Exécutez OpenClaw dans une VM macOS isolée sur votre Mac Apple Silicon existant en utilisant [Lume](https://cua.ai/docs/lume). Cela vous donne :

-   Un environnement macOS complet en isolation (votre hôte reste propre)
-   Prise en charge d'iMessage via BlueBubbles (impossible sous Linux/Windows)
-   Réinitialisation instantanée en clonant les VMs
-   Pas de coûts matériels ou cloud supplémentaires

### Fournisseurs de Mac hébergés (cloud)

Si vous voulez macOS dans le cloud, les fournisseurs de Mac hébergés fonctionnent aussi :

-   [MacStadium](https://www.macstadium.com/) (Macs hébergés)
-   D'autres vendeurs de Mac hébergés fonctionnent également ; suivez leur documentation VM + SSH

Une fois que vous avez un accès SSH à une VM macOS, continuez à l'étape 6 ci-dessous.

* * *

## Chemin rapide (Lume, utilisateurs expérimentés)

1.  Installez Lume
2.  `lume create openclaw --os macos --ipsw latest`
3.  Complétez l'Assistant de configuration, activez la Connexion à distance (SSH)
4.  `lume run openclaw --no-display`
5.  Connectez-vous en SSH, installez OpenClaw, configurez les canaux
6.  Terminé

* * *

## Ce dont vous avez besoin (Lume)

-   Mac Apple Silicon (M1/M2/M3/M4)
-   macOS Sequoia ou ultérieur sur l'hôte
-   ~60 Go d'espace disque libre par VM
-   ~20 minutes

* * *

## 1) Installer Lume

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

Si `~/.local/bin` n'est pas dans votre PATH :

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

Vérifiez :

```bash
lume --version
```

Docs : [Installation de Lume](https://cua.ai/docs/lume/guide/getting-started/installation)

* * *

## 2) Créer la VM macOS

```bash
lume create openclaw --os macos --ipsw latest
```

Ceci télécharge macOS et crée la VM. Une fenêtre VNC s'ouvre automatiquement. Note : Le téléchargement peut prendre un certain temps selon votre connexion.

* * *

## 3) Compléter l'Assistant de configuration

Dans la fenêtre VNC :

1.  Sélectionnez la langue et la région
2.  Ignorez l'identifiant Apple (ou connectez-vous si vous voulez iMessage plus tard)
3.  Créez un compte utilisateur (retenez le nom d'utilisateur et le mot de passe)
4.  Ignorez toutes les fonctionnalités optionnelles

Une fois la configuration terminée, activez SSH :

1.  Ouvrez Réglages Système → Général → Partage
2.  Activez "Connexion à distance"

* * *

## 4) Obtenir l'adresse IP de la VM

```bash
lume get openclaw
```

Cherchez l'adresse IP (généralement `192.168.64.x`).

* * *

## 5) Se connecter en SSH à la VM

```bash
ssh youruser@192.168.64.X
```

Remplacez `youruser` par le compte que vous avez créé, et l'IP par celle de votre VM.

* * *

## 6) Installer OpenClaw

À l'intérieur de la VM :

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Suivez les invites de l'intégration pour configurer votre fournisseur de modèle (Anthropic, OpenAI, etc.).

* * *

## 7) Configurer les canaux

Éditez le fichier de configuration :

```bash
nano ~/.openclaw/openclaw.json
```

Ajoutez vos canaux :

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

Puis connectez-vous à WhatsApp (scannez le QR) :

```bash
openclaw channels login
```

* * *

## 8) Exécuter la VM en mode headless

Arrêtez la VM et redémarrez sans affichage :

```bash
lume stop openclaw
lume run openclaw --no-display
```

La VM s'exécute en arrière-plan. Le démon d'OpenClaw maintient la passerelle en fonctionnement. Pour vérifier l'état :

```bash
ssh youruser@192.168.64.X "openclaw status"
```

* * *

## Bonus : Intégration iMessage

C'est la fonctionnalité phare de l'exécution sur macOS. Utilisez [BlueBubbles](https://bluebubbles.app) pour ajouter iMessage à OpenClaw. À l'intérieur de la VM :

1.  Téléchargez BlueBubbles depuis bluebubbles.app
2.  Connectez-vous avec votre identifiant Apple
3.  Activez l'API Web et définissez un mot de passe
4.  Pointez les webhooks de BlueBubbles vers votre passerelle (exemple : `https://your-gateway-host:3000/bluebubbles-webhook?password=`)

Ajoutez à votre configuration OpenClaw :

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

Redémarrez la passerelle. Maintenant, votre agent peut envoyer et recevoir des iMessages. Détails complets de la configuration : [Canal BlueBubbles](../channels/bluebubbles.md)

* * *

## Sauvegarder une image de référence

Avant de personnaliser davantage, capturez un instantané de votre état propre :

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

Réinitialisez à tout moment :

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

* * *

## Exécution 24h/24 et 7j/7

Gardez la VM en fonctionnement en :

-   Gardant votre Mac branché
-   Désactivant la mise en veille dans Réglages Système → Économiseur d'énergie
-   Utilisant `caffeinate` si nécessaire

Pour une disponibilité permanente réelle, envisagez un Mac mini dédié ou un petit VPS. Voir [Hébergement VPS](../vps.md).

* * *

## Dépannage

| Problème | Solution |
| --- | --- |
| Impossible de se connecter en SSH à la VM | Vérifiez que "Connexion à distance" est activée dans les Réglages Système de la VM |
| L'IP de la VM n'apparaît pas | Attendez que la VM démarre complètement, exécutez `lume get openclaw` à nouveau |
| Commande Lume introuvable | Ajoutez `~/.local/bin` à votre PATH |
| Le QR WhatsApp ne scanne pas | Assurez-vous d'être connecté à la VM (pas à l'hôte) lors de l'exécution de `openclaw channels login` |

* * *

## Documentation associée

-   [Hébergement VPS](../vps.md)
-   [Nœuds](../nodes.md)
-   [Passerelle distante](../gateway/remote.md)
-   [Canal BlueBubbles](../channels/bluebubbles.md)
-   [Démarrage rapide de Lume](https://cua.ai/docs/lume/guide/getting-started/quickstart)
-   [Référence CLI de Lume](https://cua.ai/docs/lume/reference/cli-reference)
-   [Configuration de VM sans surveillance](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (avancé)
-   [Isolation Docker](./docker.md) (approche alternative d'isolation)

[GCP](./gcp.md)[exe.dev](./exe-dev.md)