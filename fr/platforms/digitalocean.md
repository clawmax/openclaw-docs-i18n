

  Aperçu des plateformes

  
# DigitalOcean

## Objectif

Exécuter une passerelle OpenClaw persistante sur DigitalOcean pour **6$/mois** (ou 4$/mois avec tarification réservée). Si vous voulez une option à 0$/mois et que l'architecture ARM et une configuration spécifique au fournisseur ne vous dérangent pas, consultez le [guide Oracle Cloud](./oracle.md).

## Comparaison des coûts (2026)

| Fournisseur | Offre | Spécifications | Prix/mois | Notes |
| --- | --- | --- | --- | --- |
| Oracle Cloud | Always Free ARM | jusqu'à 4 OCPU, 24 Go RAM | 0$ | ARM, capacité limitée / particularités d'inscription |
| Hetzner | CX22 | 2 vCPU, 4 Go RAM | €3,79 (~4$) | Option payante la moins chère |
| DigitalOcean | Basic | 1 vCPU, 1 Go RAM | 6$ | Interface simple, bonne documentation |
| Vultr | Cloud Compute | 1 vCPU, 1 Go RAM | 6$ | Nombreux emplacements |
| Linode | Nanode | 1 vCPU, 1 Go RAM | 5$ | Fait maintenant partie d'Akamai |

**Choisir un fournisseur :**

-   DigitalOcean : UX la plus simple + configuration prévisible (ce guide)
-   Hetzner : bon rapport prix/performances (voir [guide Hetzner](../install/hetzner.md))
-   Oracle Cloud : peut être à 0$/mois, mais plus capricieux et ARM uniquement (voir [guide Oracle](./oracle.md))

* * *

## Prérequis

-   Compte DigitalOcean ([inscription avec 200$ de crédit gratuit](https://m.do.co/c/signup))
-   Paire de clés SSH (ou volonté d'utiliser l'authentification par mot de passe)
-   ~20 minutes

## 1) Créer un Droplet

> **⚠️** Utilisez une image de base propre (Ubuntu 24.04 LTS). Évitez les images 1-click du Marketplace de tiers, sauf si vous avez examiné leurs scripts de démarrage et leurs règles de pare-feu par défaut.

1.  Connectez-vous à [DigitalOcean](https://cloud.digitalocean.com/)
2.  Cliquez sur **Créer → Droplets**
3.  Choisissez :
    -   **Région :** La plus proche de vous (ou de vos utilisateurs)
    -   **Image :** Ubuntu 24.04 LTS
    -   **Taille :** Basic → Regular → **6$/mois** (1 vCPU, 1 Go RAM, 25 Go SSD)
    -   **Authentification :** Clé SSH (recommandé) ou mot de passe
4.  Cliquez sur **Créer un Droplet**
5.  Notez l'adresse IP

## 2) Se connecter via SSH

```bash
ssh root@VOTRE_IP_DROPLET
```

## 3) Installer OpenClaw

```bash
# Mettre à jour le système
apt update && apt upgrade -y

# Installer Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Installer OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# Vérifier
openclaw --version
```

## 4) Exécuter l'intégration

```bash
openclaw onboard --install-daemon
```

L'assistant vous guidera à travers :

-   L'authentification des modèles (clés API ou OAuth)
-   La configuration des canaux (Telegram, WhatsApp, Discord, etc.)
-   Le jeton de la passerelle (généré automatiquement)
-   L'installation du démon (systemd)

## 5) Vérifier la passerelle

```bash
# Vérifier l'état
openclaw status

# Vérifier le service
systemctl --user status openclaw-gateway.service

# Voir les logs
journalctl --user -u openclaw-gateway.service -f
```

## 6) Accéder au tableau de bord

La passerelle se lie à l'interface de bouclage (loopback) par défaut. Pour accéder à l'interface de contrôle : **Option A : Tunnel SSH (recommandé)**

```bash
# Depuis votre machine locale
ssh -L 18789:localhost:18789 root@VOTRE_IP_DROPLET

# Puis ouvrez : http://localhost:18789
```

**Option B : Tailscale Serve (HTTPS, bouclage uniquement)**

```bash
# Sur le droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configurer la passerelle pour utiliser Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

Ouvrez : `https:///` Notes :

-   Serve maintient la passerelle en bouclage uniquement et authentifie le trafic de l'interface de contrôle/WebSocket via les en-têtes d'identité Tailscale (l'authentification sans jeton suppose un hôte de passerelle de confiance ; les API HTTP nécessitent toujours un jeton/mot de passe).
-   Pour exiger un jeton/mot de passe à la place, définissez `gateway.auth.allowTailscale: false` ou utilisez `gateway.auth.mode: "password"`.

**Option C : Liaison Tailnet (sans Serve)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Ouvrez : `http://<tailscale-ip>:18789` (jeton requis).

## 7) Connecter vos canaux

### Telegram

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

### WhatsApp

```bash
openclaw channels login whatsapp
# Scanner le code QR
```

Voir [Canaux](../channels.md) pour les autres fournisseurs.

* * *

## Optimisations pour 1 Go de RAM

Le droplet à 6$ n'a que 1 Go de RAM. Pour que tout fonctionne correctement :

### Ajouter un swap (recommandé)

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### Utiliser un modèle plus léger

Si vous rencontrez des erreurs de mémoire insuffisante (OOM), envisagez :

-   D'utiliser des modèles basés sur une API (Claude, GPT) au lieu de modèles locaux
-   De définir `agents.defaults.model.primary` sur un modèle plus petit

### Surveiller la mémoire

```
free -h
htop
```

* * *

## Persistance

Tous les états sont stockés dans :

-   `~/.openclaw/` — configuration, identifiants, données de session
-   `~/.openclaw/workspace/` — espace de travail (SOUL.md, mémoire, etc.)

Ces données survivent aux redémarrages. Sauvegardez-les périodiquement :

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## Alternative gratuite Oracle Cloud

Oracle Cloud propose des instances **Always Free** ARM qui sont nettement plus puissantes que toute option payante ici — pour 0$/mois.

| Ce que vous obtenez | Spécifications |
| --- | --- |
| **4 OCPU** | ARM Ampere A1 |
| **24 Go RAM** | Plus que suffisant |
| **200 Go de stockage** | Volume de bloc |
| **Gratuit à vie** | Aucun frais de carte de crédit |

**Mises en garde :**

-   L'inscription peut être capricieuse (réessayez en cas d'échec)
-   Architecture ARM — la plupart des choses fonctionnent, mais certains binaires nécessitent des builds ARM

Pour le guide d'installation complet, voir [Oracle Cloud](./oracle.md). Pour des conseils d'inscription et le dépannage du processus d'enrôlement, consultez ce [guide communautaire](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd).

* * *

## Dépannage

### La passerelle ne démarre pas

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

### Port déjà utilisé

```bash
lsof -i :18789
kill <PID>
```

### Mémoire insuffisante

```bash
# Vérifier la mémoire
free -h

# Ajouter plus de swap
# Ou passer à un droplet à 12$/mois (2 Go RAM)
```

* * *

## Voir aussi

-   [Guide Hetzner](../install/hetzner.md) — moins cher, plus puissant
-   [Installation Docker](../install/docker.md) — configuration conteneurisée
-   [Tailscale](../gateway/tailscale.md) — accès distant sécurisé
-   [Configuration](../gateway/configuration.md) — référence complète de configuration

[iOS App](./ios.md)[Oracle Cloud](./oracle.md)

---