

  Autres méthodes d'installation

  
# Ansible

La méthode recommandée pour déployer OpenClaw sur des serveurs de production est via **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — un installateur automatisé avec une architecture sécurité-first.

## Démarrage rapide

Installation en une commande :

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Guide complet : [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** Le dépôt openclaw-ansible est la source de référence pour le déploiement Ansible. Cette page est un aperçu rapide.

## Ce que vous obtenez

-   🔒 **Sécurité pare-feu d'abord** : UFW + isolation Docker (seul SSH + Tailscale accessibles)
-   🔐 **VPN Tailscale** : Accès distant sécurisé sans exposer les services publiquement
-   🐳 **Docker** : Conteneurs sandbox isolés, liaisons localhost uniquement
-   🛡️ **Défense en profondeur** : Architecture de sécurité à 4 couches
-   🚀 **Configuration en une commande** : Déploiement complet en quelques minutes
-   🔧 **Intégration Systemd** : Démarrage automatique au boot avec durcissement

## Prérequis

-   **OS** : Debian 11+ ou Ubuntu 20.04+
-   **Accès** : Privilèges root ou sudo
-   **Réseau** : Connexion Internet pour l'installation des paquets
-   **Ansible** : 2.14+ (installé automatiquement par le script de démarrage rapide)

## Ce qui est installé

Le playbook Ansible installe et configure :

1.  **Tailscale** (VPN maillé pour un accès distant sécurisé)
2.  **Pare-feu UFW** (ports SSH + Tailscale uniquement)
3.  **Docker CE + Compose V2** (pour les sandbox des agents)
4.  **Node.js 22.x + pnpm** (dépendances d'exécution)
5.  **OpenClaw** (hébergé sur l'hôte, pas conteneurisé)
6.  **Service Systemd** (démarrage automatique avec durcissement de sécurité)

Note : La passerelle s'exécute **directement sur l'hôte** (pas dans Docker), mais les sandbox des agents utilisent Docker pour l'isolation. Voir [Sandboxing](../gateway/sandboxing.md) pour plus de détails.

## Configuration post-installation

Une fois l'installation terminée, basculez vers l'utilisateur openclaw :

```bash
sudo -i -u openclaw
```

Le script post-installation vous guidera à travers :

1.  **Assistant d'intégration** : Configurer les paramètres d'OpenClaw
2.  **Connexion du fournisseur** : Connecter WhatsApp/Telegram/Discord/Signal
3.  **Test de la passerelle** : Vérifier l'installation
4.  **Configuration Tailscale** : Se connecter à votre maillage VPN

### Commandes rapides

```bash
# Vérifier l'état du service
sudo systemctl status openclaw

# Voir les logs en direct
sudo journalctl -u openclaw -f

# Redémarrer la passerelle
sudo systemctl restart openclaw

# Connexion du fournisseur (exécuter en tant qu'utilisateur openclaw)
sudo -i -u openclaw
openclaw channels login
```

## Architecture de sécurité

### Défense à 4 couches

1.  **Pare-feu (UFW)** : Seul SSH (22) + Tailscale (41641/udp) exposés publiquement
2.  **VPN (Tailscale)** : Passerelle accessible uniquement via le maillage VPN
3.  **Isolation Docker** : La chaîne iptables DOCKER-USER empêche l'exposition des ports externes
4.  **Durcissement Systemd** : NoNewPrivileges, PrivateTmp, utilisateur non privilégié

### Vérification

Testez la surface d'attaque externe :

```bash
nmap -p- VOTRE_IP_SERVEUR
```

Devrait montrer **seulement le port 22** (SSH) ouvert. Tous les autres services (passerelle, Docker) sont verrouillés.

### Disponibilité de Docker

Docker est installé pour les **sandbox des agents** (exécution isolée des outils), pas pour exécuter la passerelle elle-même. La passerelle se lie uniquement au localhost et est accessible via le VPN Tailscale. Voir [Multi-Agent Sandbox & Tools](../tools/multi-agent-sandbox-tools.md) pour la configuration du sandbox.

## Installation manuelle

Si vous préférez un contrôle manuel plutôt que l'automatisation :

```bash
# 1. Installer les prérequis
sudo apt update && sudo apt install -y ansible git

# 2. Cloner le dépôt
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Installer les collections Ansible
ansible-galaxy collection install -r requirements.yml

# 4. Exécuter le playbook
./run-playbook.sh

# Ou exécuter directement (puis exécuter manuellement /tmp/openclaw-setup.sh après)
# ansible-playbook playbook.yml --ask-become-pass
```

## Mise à jour d'OpenClaw

L'installateur Ansible configure OpenClaw pour des mises à jour manuelles. Voir [Mise à jour](./updating.md) pour le flux de mise à jour standard. Pour ré-exécuter le playbook Ansible (par exemple, pour des changements de configuration) :

```bash
cd openclaw-ansible
./run-playbook.sh
```

Note : Ce processus est idempotent et sûr à exécuter plusieurs fois.

## Dépannage

### Le pare-feu bloque ma connexion

Si vous êtes bloqué à l'extérieur :

-   Assurez-vous d'abord de pouvoir accéder via le VPN Tailscale
-   L'accès SSH (port 22) est toujours autorisé
-   La passerelle est **uniquement** accessible via Tailscale par conception

### Le service ne démarre pas

```bash
# Vérifier les logs
sudo journalctl -u openclaw -n 100

# Vérifier les permissions
sudo ls -la /opt/openclaw

# Tester le démarrage manuel
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Problèmes de sandbox Docker

```bash
# Vérifier que Docker fonctionne
sudo systemctl status docker

# Vérifier l'image du sandbox
sudo docker images | grep openclaw-sandbox

# Construire l'image du sandbox si elle manque
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### Échec de connexion du fournisseur

Assurez-vous d'exécuter en tant qu'utilisateur `openclaw` :

```bash
sudo -i -u openclaw
openclaw channels login
```

## Configuration avancée

Pour l'architecture de sécurité détaillée et le dépannage :

-   [Architecture de sécurité](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
-   [Détails techniques](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
-   [Guide de dépannage](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## Liens connexes

-   [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — guide de déploiement complet
-   [Docker](./docker.md) — configuration de la passerelle conteneurisée
-   [Sandboxing](../gateway/sandboxing.md) — configuration du sandbox des agents
-   [Multi-Agent Sandbox & Tools](../tools/multi-agent-sandbox-tools.md) — isolation par agent

[Nix](./nix.md)[Bun (Expérimental)](./bun.md)