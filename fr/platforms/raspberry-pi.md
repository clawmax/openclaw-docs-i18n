

  Aperçu des plateformes

  
# Raspberry Pi

## Objectif

Exécuter une passerelle OpenClaw persistante, toujours allumée, sur un Raspberry Pi pour un coût unique d'environ **35-80 $** (pas de frais mensuels). Parfait pour :

-   Un assistant IA personnel 24h/24 et 7j/7
-   Un hub de domotique
-   Un bot Telegram/WhatsApp à faible consommation, toujours disponible

## Exigences matérielles

| Modèle de Pi | RAM | Fonctionne ? | Notes |
| --- | --- | --- | --- |
| **Pi 5** | 4GB/8GB | ✅ Meilleur | Le plus rapide, recommandé |
| **Pi 4** | 4GB | ✅ Bon | Le point idéal pour la plupart des utilisateurs |
| **Pi 4** | 2GB | ✅ OK | Fonctionne, ajouter un swap |
| **Pi 4** | 1GB | ⚠️ Limité | Possible avec swap, configuration minimale |
| **Pi 3B+** | 1GB | ⚠️ Lent | Fonctionne mais lent |
| **Pi Zero 2 W** | 512MB | ❌ | Non recommandé |

**Spécifications minimales :** 1 Go de RAM, 1 cœur, 500 Mo d'espace disque  
**Recommandé :** 2 Go+ de RAM, OS 64 bits, carte SD 16 Go+ (ou SSD USB)

## Ce dont vous aurez besoin

-   Raspberry Pi 4 ou 5 (2 Go+ recommandé)
-   Carte MicroSD (16 Go+) ou SSD USB (meilleures performances)
-   Alimentation (PSU officiel Pi recommandé)
-   Connexion réseau (Ethernet ou WiFi)
-   ~30 minutes

## 1) Installer le système d'exploitation

Utilisez **Raspberry Pi OS Lite (64 bits)** — pas besoin de bureau pour un serveur headless.

1.  Téléchargez [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2.  Choisissez l'OS : **Raspberry Pi OS Lite (64 bits)**
3.  Cliquez sur l'icône d'engrenage (⚙️) pour pré-configurer :
    -   Définir le nom d'hôte : `gateway-host`
    -   Activer SSH
    -   Définir nom d'utilisateur/mot de passe
    -   Configurer le WiFi (si vous n'utilisez pas Ethernet)
4.  Flashez sur votre carte SD / lecteur USB
5.  Insérez et démarrez le Pi

## 2) Se connecter via SSH

```bash
ssh user@gateway-host
# ou utilisez l'adresse IP
ssh user@192.168.x.x
```

## 3) Configuration du système

```bash
# Mettre à jour le système
sudo apt update && sudo apt upgrade -y

# Installer les paquets essentiels
sudo apt install -y git curl build-essential

# Définir le fuseau horaire (important pour cron/les rappels)
sudo timedatectl set-timezone America/Chicago  # Changez pour votre fuseau horaire
```

## 4) Installer Node.js 22 (ARM64)

```bash
# Installer Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Vérifier
node --version  # Devrait afficher v22.x.x
npm --version
```

## 5) Ajouter un Swap (Important pour 2 Go ou moins)

Le swap empêche les plantages par manque de mémoire :

```bash
# Créer un fichier swap de 2 Go
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Rendre permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Optimiser pour faible RAM (réduire la swappiness)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## 6) Installer OpenClaw

### Option A : Installation standard (Recommandée)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Option B : Installation modifiable (Pour le bidouillage)

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

L'installation modifiable vous donne un accès direct aux logs et au code — utile pour déboguer les problèmes spécifiques à ARM.

## 7) Exécuter l'intégration

```bash
openclaw onboard --install-daemon
```

Suivez l'assistant :

1.  **Mode passerelle :** Local
2.  **Authentification :** Clés API recommandées (OAuth peut être capricieux sur un Pi headless)
3.  **Canaux :** Telegram est le plus simple pour commencer
4.  **Démon :** Oui (systemd)

## 8) Vérifier l'installation

```bash
# Vérifier le statut
openclaw status

# Vérifier le service
sudo systemctl status openclaw

# Voir les logs
journalctl -u openclaw -f
```

## 9) Accéder au tableau de bord

Puisque le Pi est headless, utilisez un tunnel SSH :

```bash
# Depuis votre ordinateur portable/bureau
ssh -L 18789:localhost:18789 user@gateway-host

# Puis ouvrez dans le navigateur
open http://localhost:18789
```

Ou utilisez Tailscale pour un accès toujours actif :

```bash
# Sur le Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Mettre à jour la configuration
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

* * *

## Optimisations des performances

### Utiliser un SSD USB (Amélioration considérable)

Les cartes SD sont lentes et s'usent. Un SSD USB améliore considérablement les performances :

```bash
# Vérifier si le démarrage se fait depuis USB
lsblk
```

Voir le [guide de démarrage USB Pi](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) pour la configuration.

### Accélérer le démarrage de la CLI (cache de compilation des modules)

Sur les hôtes Pi moins puissants, activez le cache de compilation des modules de Node pour que les exécutions répétées de la CLI soient plus rapides :

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF' # pragma: allowlist secret
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

Notes :

-   `NODE_COMPILE_CACHE` accélère les exécutions ultérieures (`status`, `health`, `--help`).
-   `/var/tmp` survit mieux aux redémarrages que `/tmp`.
-   `OPENCLAW_NO_RESPAWN=1` évite le coût de démarrage supplémentaire dû au redémarrage automatique de la CLI.
-   La première exécution charge le cache ; les exécutions suivantes en bénéficient le plus.

### Réglage du démarrage systemd (optionnel)

Si ce Pi exécute principalement OpenClaw, ajoutez un drop-in de service pour réduire les saccades de redémarrage et maintenir un environnement de démarrage stable :

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

Puis appliquez :

```bash
sudo systemctl daemon-reload
sudo systemctl restart openclaw
```

Si possible, conservez l'état/le cache d'OpenClaw sur un stockage basé sur SSD pour éviter les goulots d'étranglement d'E/S aléatoires des cartes SD lors des démarrages à froid. Comment les politiques `Restart=` aident à la récupération automatique : [systemd peut automatiser la récupération des services](https://www.redhat.com/en/blog/systemd-automate-recovery).

### Réduire l'utilisation de la mémoire

```bash
# Désactiver l'allocation de mémoire GPU (headless)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Désactiver le Bluetooth si non nécessaire
sudo systemctl disable bluetooth
```

### Surveiller les ressources

```bash
# Vérifier la mémoire
free -h

# Vérifier la température du CPU
vcgencmd measure_temp

# Surveillance en direct
htop
```

* * *

## Notes spécifiques à ARM

### Compatibilité binaire

La plupart des fonctionnalités d'OpenClaw fonctionnent sur ARM64, mais certains binaires externes peuvent nécessiter des builds ARM :

| Outil | Statut ARM64 | Notes |
| --- | --- | --- |
| Node.js | ✅ | Fonctionne parfaitement |
| WhatsApp (Baileys) | ✅ | Pur JS, aucun problème |
| Telegram | ✅ | Pur JS, aucun problème |
| gog (CLI Gmail) | ⚠️ | Vérifier la version ARM |
| Chromium (navigateur) | ✅ | `sudo apt install chromium-browser` |

Si une compétence échoue, vérifiez si son binaire a une version ARM. De nombreux outils Go/Rust en ont ; d'autres non.

### 32 bits vs 64 bits

**Utilisez toujours un OS 64 bits.** Node.js et de nombreux outils modernes le nécessitent. Vérifiez avec :

```bash
uname -m
# Devrait afficher : aarch64 (64 bits) et non armv7l (32 bits)
```

* * *

## Configuration de modèle recommandée

Puisque le Pi n'est que la passerelle (les modèles s'exécutent dans le cloud), utilisez des modèles basés sur API :

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**N'essayez pas d'exécuter des LLM locaux sur un Pi** — même les petits modèles sont trop lents. Laissez Claude/GPT faire le gros du travail.

* * *

## Démarrage automatique au boot

L'assistant d'intégration configure cela, mais pour vérifier :

```bash
# Vérifier que le service est activé
sudo systemctl is-enabled openclaw

# Activer si ce n'est pas le cas
sudo systemctl enable openclaw

# Démarrer au boot
sudo systemctl start openclaw
```

* * *

## Dépannage

### Manque de mémoire (OOM)

```bash
# Vérifier la mémoire
free -h

# Ajouter plus de swap (voir Étape 5)
# Ou réduire les services en cours d'exécution sur le Pi
```

### Performances lentes

-   Utilisez un SSD USB au lieu d'une carte SD
-   Désactivez les services inutilisés : `sudo systemctl disable cups bluetooth avahi-daemon`
-   Vérifiez l'étranglement du CPU : `vcgencmd get_throttled` (devrait retourner `0x0`)

### Le service ne démarre pas

```bash
# Vérifier les logs
journalctl -u openclaw --no-pager -n 100

# Correction courante : reconstruire
cd ~/openclaw  # si vous utilisez l'installation modifiable
npm run build
sudo systemctl restart openclaw
```

### Problèmes de binaires ARM

Si une compétence échoue avec "exec format error" :

1.  Vérifiez si le binaire a une version ARM64
2.  Essayez de le compiler à partir des sources
3.  Ou utilisez un conteneur Docker avec support ARM

### Coupures WiFi

Pour les Pis headless sur WiFi :

```bash
# Désactiver la gestion de l'alimentation WiFi
sudo iwconfig wlan0 power off

# Rendre permanent
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

* * *

## Comparaison des coûts

| Configuration | Coût unique | Coût mensuel | Notes |
| --- | --- | --- | --- |
| **Pi 4 (2 Go)** | ~45 $ | 0 $ | \+ électricité (~5 $/an) |
| **Pi 4 (4 Go)** | ~55 $ | 0 $ | Recommandé |
| **Pi 5 (4 Go)** | ~60 $ | 0 $ | Meilleures performances |
| **Pi 5 (8 Go)** | ~80 $ | 0 $ | Excessif mais pérenne |
| DigitalOcean | 0 $ | 6 $/mois | 72 $/an |
| Hetzner | 0 $ | 3,79 €/mois | ~50 $/an |

**Point d'équilibre :** Un Pi s'amortit en ~6-12 mois par rapport à un VPS cloud.

* * *

## Voir aussi

-   [Guide Linux](./linux.md) — configuration Linux générale
-   [Guide DigitalOcean](./digitalocean.md) — alternative cloud
-   [Guide Hetzner](../install/hetzner.md) — configuration Docker
-   [Tailscale](../gateway/tailscale.md) — accès à distance
-   [Nœuds](../nodes.md) — associez votre ordinateur portable/téléphone à la passerelle Pi

[Oracle Cloud](./oracle.md)[Configuration Dev macOS](./mac/dev-setup.md)