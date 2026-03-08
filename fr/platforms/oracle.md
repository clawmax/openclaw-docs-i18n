title: "Exécuter OpenClaw Gateway sur le niveau gratuit ARM d'Oracle Cloud Always Free"
description: "Apprenez à déployer et exécuter une passerelle OpenClaw persistante sur le niveau gratuit ARM d'Oracle Cloud Always Free, incluant la configuration, la sécurité et les étapes de dépannage."
keywords: ["openclaw oracle", "oracle cloud niveau gratuit", "passerelle openclaw", "déploiement serveur arm", "tailscale oracle", "installation openclaw", "oci toujours gratuit", "sécurité openclaw"]
---

  Aperçu des plateformes

  
# Oracle Cloud

## Objectif

Exécuter une passerelle OpenClaw persistante sur le niveau **Always Free** ARM d'Oracle Cloud. Le niveau gratuit d'Oracle peut être un excellent choix pour OpenClaw (surtout si vous avez déjà un compte OCI), mais il comporte des compromis :

-   Architecture ARM (la plupart des choses fonctionnent, mais certains binaires peuvent être uniquement x86)
-   La capacité et l'inscription peuvent être capricieuses

## Comparaison des coûts (2026)

| Fournisseur | Plan | Spécifications | Prix/mois | Notes |
| --- | --- | --- | --- | --- |
| Oracle Cloud | Always Free ARM | jusqu'à 4 OCPU, 24 Go RAM | 0 $ | ARM, capacité limitée |
| Hetzner | CX22 | 2 vCPU, 4 Go RAM | ~ 4 $ | Option payante la moins chère |
| DigitalOcean | Basic | 1 vCPU, 1 Go RAM | 6 $ | Interface facile, bonne documentation |
| Vultr | Cloud Compute | 1 vCPU, 1 Go RAM | 6 $ | Nombreux emplacements |
| Linode | Nanode | 1 vCPU, 1 Go RAM | 5 $ | Fait maintenant partie d'Akamai |

* * *

## Prérequis

-   Compte Oracle Cloud ([inscription](https://www.oracle.com/cloud/free/)) — voir le [guide d'inscription communautaire](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) en cas de problème
-   Compte Tailscale (gratuit sur [tailscale.com](https://tailscale.com))
-   ~30 minutes

## 1) Créer une instance OCI

1.  Connectez-vous à la [Console Oracle Cloud](https://cloud.oracle.com/)
2.  Naviguez vers **Compute → Instances → Créer une instance**
3.  Configurez :
    -   **Nom :** `openclaw`
    -   **Image :** Ubuntu 24.04 (aarch64)
    -   **Forme :** `VM.Standard.A1.Flex` (Ampere ARM)
    -   **OCPUs :** 2 (ou jusqu'à 4)
    -   **Mémoire :** 12 Go (ou jusqu'à 24 Go)
    -   **Volume de démarrage :** 50 Go (jusqu'à 200 Go gratuit)
    -   **Clé SSH :** Ajoutez votre clé publique
4.  Cliquez sur **Créer**
5.  Notez l'adresse IP publique

**Astuce :** Si la création de l'instance échoue avec "Out of capacity", essayez un autre domaine de disponibilité ou réessayez plus tard. La capacité du niveau gratuit est limitée.

## 2) Se connecter et mettre à jour

```bash
# Connexion via l'IP publique
ssh ubuntu@VOTRE_IP_PUBLIQUE

# Mettre à jour le système
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**Note :** `build-essential` est requis pour la compilation ARM de certaines dépendances.

## 3) Configurer l'utilisateur et le nom d'hôte

```bash
# Définir le nom d'hôte
sudo hostnamectl set-hostname openclaw

# Définir le mot de passe pour l'utilisateur ubuntu
sudo passwd ubuntu

# Activer le lingering (maintient les services utilisateur en fonctionnement après la déconnexion)
sudo loginctl enable-linger ubuntu
```

## 4) Installer Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

Cela active Tailscale SSH, vous permettant de vous connecter via `ssh openclaw` depuis n'importe quel appareil de votre tailnet — aucune IP publique nécessaire. Vérifiez :

```bash
tailscale status
```

**À partir de maintenant, connectez-vous via Tailscale :** `ssh ubuntu@openclaw` (ou utilisez l'IP Tailscale).

## 5) Installer OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
source ~/.bashrc
```

Lorsque vous êtes invité à choisir "How do you want to hatch your bot?", sélectionnez **"Do this later"**.

> Note : Si vous rencontrez des problèmes de compilation native ARM, commencez par les paquets système (par exemple `sudo apt install -y build-essential`) avant de vous tourner vers Homebrew.

## 6) Configurer la passerelle (loopback + authentification par jeton) et activer Tailscale Serve

Utilisez l'authentification par jeton par défaut. Elle est prévisible et évite d'avoir besoin de drapeaux "insecure auth" dans l'interface de contrôle.

```bash
# Gardez la passerelle privée sur la VM
openclaw config set gateway.bind loopback

# Exiger une authentification pour la passerelle + l'interface de contrôle
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Exposer via Tailscale Serve (HTTPS + accès tailnet)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

## 7) Vérifier

```bash
# Vérifier la version
openclaw --version

# Vérifier le statut du démon
systemctl --user status openclaw-gateway

# Vérifier Tailscale Serve
tailscale serve status

# Tester la réponse locale
curl http://localhost:18789
```

## 8) Verrouiller la sécurité du VCN

Maintenant que tout fonctionne, verrouillez le VCN pour bloquer tout le trafic sauf Tailscale. Le Virtual Cloud Network d'OCI agit comme un pare-feu au niveau du réseau — le trafic est bloqué avant d'atteindre votre instance.

1.  Allez dans **Networking → Virtual Cloud Networks** dans la Console OCI
2.  Cliquez sur votre VCN → **Security Lists** → Default Security List
3.  **Supprimez** toutes les règles d'entrée sauf :
    -   `0.0.0.0/0 UDP 41641` (Tailscale)
4.  Conservez les règles de sortie par défaut (autoriser tout le trafic sortant)

Cela bloque SSH sur le port 22, HTTP, HTTPS et tout le reste au niveau du réseau. Désormais, vous ne pouvez vous connecter que via Tailscale.

* * *

## Accéder à l'interface de contrôle

Depuis n'importe quel appareil sur votre réseau Tailscale :

```
https://openclaw.<nom-du-tailnet>.ts.net/
```

Remplacez `<nom-du-tailnet>` par le nom de votre tailnet (visible dans `tailscale status`). Aucun tunnel SSH nécessaire. Tailscale fournit :

-   Chiffrement HTTPS (certificats automatiques)
-   Authentification via l'identité Tailscale
-   Accès depuis n'importe quel appareil de votre tailnet (ordinateur portable, téléphone, etc.)

* * *

## Sécurité : VCN + Tailscale (recommandation de base)

Avec le VCN verrouillé (seul le port UDP 41641 ouvert) et la passerelle liée à loopback, vous obtenez une défense en profondeur solide : le trafic public est bloqué au niveau du réseau, et l'accès administrateur se fait via votre tailnet. Cette configuration supprime souvent le *besoin* de règles de pare-feu supplémentaires au niveau de l'hôte pour stopper les attaques par force brute SSH à l'échelle d'Internet — mais vous devez tout de même maintenir le système d'exploitation à jour, exécuter `openclaw security audit` et vérifier que vous n'écoutez pas accidentellement sur des interfaces publiques.

### Ce qui est déjà protégé

| Étape traditionnelle | Nécessaire ? | Pourquoi |
| --- | --- | --- |
| Pare-feu UFW | Non | Le VCN bloque avant que le trafic n'atteigne l'instance |
| fail2ban | Non | Pas de force brute si le port 22 est bloqué au niveau du VCN |
| Durcissement sshd | Non | Tailscale SSH n'utilise pas sshd |
| Désactiver la connexion root | Non | Tailscale utilise l'identité Tailscale, pas les utilisateurs système |
| Authentification par clé SSH uniquement | Non | Tailscale s'authentifie via votre tailnet |
| Durcissement IPv6 | Généralement non | Dépend des paramètres de votre VCN/sous-réseau ; vérifiez ce qui est réellement attribué/exposé |

### Toujours recommandé

-   **Permissions des identifiants :** `chmod 700 ~/.openclaw`
-   **Audit de sécurité :** `openclaw security audit`
-   **Mises à jour système :** `sudo apt update && sudo apt upgrade` régulièrement
-   **Surveiller Tailscale :** Examinez les appareils dans la [console d'administration Tailscale](https://login.tailscale.com/admin)

### Vérifier la posture de sécurité

```bash
# Confirmer qu'aucun port public n'écoute
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Vérifier que Tailscale SSH est actif
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH actif"

# Optionnel : désactiver sshd complètement
sudo systemctl disable --now ssh
```

* * *

## Solution de secours : Tunnel SSH

Si Tailscale Serve ne fonctionne pas, utilisez un tunnel SSH :

```bash
# Depuis votre machine locale (via Tailscale)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

Puis ouvrez `http://localhost:18789`.

* * *

## Dépannage

### La création de l'instance échoue ("Out of capacity")

Les instances ARM du niveau gratuit sont populaires. Essayez :

-   Un autre domaine de disponibilité
-   Réessayez pendant les heures creuses (tôt le matin)
-   Utilisez le filtre "Always Free" lors de la sélection de la forme

### Tailscale ne se connecte pas

```bash
# Vérifier le statut
sudo tailscale status

# Se ré-authentifier
sudo tailscale up --ssh --hostname=openclaw --reset
```

### La passerelle ne démarre pas

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

### Impossible d'atteindre l'interface de contrôle

```bash
# Vérifier que Tailscale Serve fonctionne
tailscale serve status

# Vérifier que la passerelle écoute
curl http://localhost:18789

# Redémarrer si nécessaire
systemctl --user restart openclaw-gateway
```

### Problèmes de binaires ARM

Certains outils peuvent ne pas avoir de versions ARM. Vérifiez :

```bash
uname -m  # Devrait afficher aarch64
```

La plupart des paquets npm fonctionnent bien. Pour les binaires, recherchez les versions `linux-arm64` ou `aarch64`.

* * *

## Persistance

Tous les états sont stockés dans :

-   `~/.openclaw/` — configuration, identifiants, données de session
-   `~/.openclaw/workspace/` — espace de travail (SOUL.md, mémoire, artefacts)

Sauvegardez périodiquement :

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## Voir aussi

-   [Accès distant à la passerelle](../gateway/remote.md) — autres modèles d'accès distant
-   [Intégration Tailscale](../gateway/tailscale.md) — documentation complète de Tailscale
-   [Configuration de la passerelle](../gateway/configuration.md) — toutes les options de configuration
-   [Guide DigitalOcean](./digitalocean.md) — si vous voulez une option payante + une inscription plus facile
-   [Guide Hetzner](../install/hetzner.md) — alternative basée sur Docker

[DigitalOcean](./digitalocean.md)[Raspberry Pi](./raspberry-pi.md)