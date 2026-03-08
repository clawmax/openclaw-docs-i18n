title: "Guide d'hébergement et de déploiement OpenClaw sur VPS"
description: "Apprenez à déployer OpenClaw sur des fournisseurs VPS comme Railway, Oracle Cloud et Fly.io. Guides d'installation, conseils de sécurité et optimisation des performances pour les agents cloud."
keywords: ["hébergement vps", "déploiement openclaw", "passerelle cloud", "railway", "oracle cloud", "réglage systemd", "nœuds", "optimisation arm"]
---

  Hébergement et déploiement

  
# Hébergement VPS

Ce hub contient des liens vers les guides d'hébergement VPS pris en charge et explique le fonctionnement des déploiements cloud à un haut niveau.

## Choisir un fournisseur

-   **Railway** (installation en un clic + navigateur) : [Railway](./install/railway.md)
-   **Northflank** (installation en un clic + navigateur) : [Northflank](./install/northflank.md)
-   **Oracle Cloud (Toujours Gratuit)** : [Oracle](./platforms/oracle.md) — 0 $/mois (Toujours Gratuit, ARM ; la capacité/l'inscription peut être capricieuse)
-   **Fly.io** : [Fly.io](./install/fly.md)
-   **Hetzner (Docker)** : [Hetzner](./install/hetzner.md)
-   **GCP (Compute Engine)** : [GCP](./install/gcp.md)
-   **exe.dev** (VM + proxy HTTPS) : [exe.dev](./install/exe-dev.md)
-   **AWS (EC2/Lightsail/free tier)** : fonctionne également bien. Guide vidéo : [https://x.com/techfrenAJ/status/2014934471095812547](https://x.com/techfrenAJ/status/2014934471095812547)

## Fonctionnement des configurations cloud

-   La **Passerelle (Gateway) s'exécute sur le VPS** et possède l'état et l'espace de travail.
-   Vous vous connectez depuis votre ordinateur portable/téléphone via l'**Interface de Contrôle** ou **Tailscale/SSH**.
-   Considérez le VPS comme la source de vérité et **sauvegardez** l'état et l'espace de travail.
-   Sécurité par défaut : gardez la Passerelle en boucle locale et accédez-y via un tunnel SSH ou Tailscale Serve. Si vous la liez à `lan`/`tailnet`, exigez `gateway.auth.token` ou `gateway.auth.password`.

Accès à distance : [Passerelle distante](./gateway/remote.md)  
Hub des plateformes : [Plateformes](./platforms.md)

## Agent d'entreprise partagé sur un VPS

C'est une configuration valide lorsque les utilisateurs sont dans une même limite de confiance (par exemple une équipe d'entreprise), et que l'agent est réservé aux activités professionnelles.

-   Gardez-le sur un environnement d'exécution dédié (VPS/VM/conteneur + utilisateur/comptes OS dédiés).
-   Ne connectez pas cet environnement à des comptes Apple/Google personnels ou à des profils de navigateur/gestionnaire de mots de passe personnels.
-   Si les utilisateurs sont antagonistes les uns envers les autres, séparez-les par passerelle/hôte/utilisateur OS.

Détails du modèle de sécurité : [Sécurité](./gateway/security.md)

## Utiliser des nœuds avec un VPS

Vous pouvez garder la Passerelle dans le cloud et appairer des **nœuds** sur vos appareils locaux (Mac/iOS/Android/sans interface). Les nœuds fournissent des capacités d'écran/caméra/canvas local et `system.run` tandis que la Passerelle reste dans le cloud. Docs : [Nœuds](./nodes.md), [CLI des nœuds](./cli/nodes.md)

## Réglage du démarrage pour les petites VM et les hôtes ARM

Si les commandes CLI semblent lentes sur des VM peu puissantes (ou des hôtes ARM), activez le cache de compilation des modules de Node :

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF'
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

-   `NODE_COMPILE_CACHE` améliore les temps de démarrage des commandes répétées.
-   `OPENCLAW_NO_RESPAWN=1` évite la surcharge de démarrage supplémentaire due à un chemin de redémarrage automatique.
-   La première exécution de commande charge le cache ; les exécutions suivantes sont plus rapides.
-   Pour les spécificités du Raspberry Pi, voir [Raspberry Pi](./platforms/raspberry-pi.md).

### Liste de réglages systemd (optionnel)

Pour les hôtes VM utilisant `systemd`, considérez :

-   Ajoutez des variables d'environnement au service pour un chemin de démarrage stable :
    -   `OPENCLAW_NO_RESPAWN=1`
    -   `NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache`
-   Gardez un comportement de redémarrage explicite :
    -   `Restart=always`
    -   `RestartSec=2`
    -   `TimeoutStartSec=90`
-   Préférez des disques SSD pour les chemins d'état/cache afin de réduire les pénalités de démarrage à froid dues aux E/S aléatoires.

Exemple :

```bash
sudo systemctl edit openclaw
```

```ini
[Service]
Environment=OPENCLAW_NO_RESPAWN=1
Environment=NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
Restart=always
RestartSec=2
TimeoutStartSec=90
```

Comment les politiques `Restart=` aident à la récupération automatisée : [systemd peut automatiser la récupération des services](https://www.redhat.com/en/blog/systemd-automate-recovery).

[Désinstaller](./install/uninstall.md)[Fly.io](./install/fly.md)