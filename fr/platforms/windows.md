

  Aperçu des plateformes

  
# Windows (WSL2)

OpenClaw sur Windows est recommandé **via WSL2** (Ubuntu recommandé). Le CLI + la Passerelle s'exécutent à l'intérieur de Linux, ce qui maintient l'environnement d'exécution cohérent et rend les outils bien plus compatibles (Node/Bun/pnpm, binaires Linux, compétences). Le mode natif Windows peut être plus délicat. WSL2 vous offre l'expérience Linux complète — une seule commande pour installer : `wsl --install`. Des applications compagnes natives Windows sont prévues.

## Installation (WSL2)

-   [Premiers pas](../start/getting-started.md) (à utiliser dans WSL)
-   [Installation & mises à jour](../install/updating.md)
-   Guide officiel WSL2 (Microsoft) : [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)

## Passerelle

-   [Runbook de la passerelle](../gateway.md)
-   [Configuration](../gateway/configuration.md)

## Installation du service de passerelle (CLI)

Dans WSL2 :

```bash
openclaw onboard --install-daemon
```

Ou :

```bash
openclaw gateway install
```

Ou :

```bash
openclaw configure
```

Sélectionnez **Service de passerelle** lorsque vous y êtes invité. Réparer/migrer :

```bash
openclaw doctor
```

## Démarrage automatique de la passerelle avant la connexion Windows

Pour les configurations sans écran, assurez-vous que la chaîne de démarrage complète s'exécute même lorsque personne ne se connecte à Windows.

### 1) Maintenir les services utilisateur en cours d'exécution sans connexion

Dans WSL :

```bash
sudo loginctl enable-linger "$(whoami)"
```

### 2) Installer le service utilisateur de la passerelle OpenClaw

Dans WSL :

```bash
openclaw gateway install
```

### 3) Démarrer WSL automatiquement au démarrage de Windows

Dans PowerShell en tant qu'Administrateur :

```bash
schtasks /create /tn "WSL Boot" /tr "wsl.exe -d Ubuntu --exec /bin/true" /sc onstart /ru SYSTEM
```

Remplacez `Ubuntu` par le nom de votre distribution depuis :

```bash
wsl --list --verbose
```

### Vérifier la chaîne de démarrage

Après un redémarrage (avant la connexion Windows), vérifiez depuis WSL :

```bash
systemctl --user is-enabled openclaw-gateway
systemctl --user status openclaw-gateway --no-pager
```

## Avancé : exposer les services WSL sur le LAN (portproxy)

WSL a son propre réseau virtuel. Si une autre machine doit atteindre un service exécuté **à l'intérieur de WSL** (SSH, un serveur TTS local, ou la Passerelle), vous devez rediriger un port Windows vers l'IP actuelle de WSL. L'IP de WSL change après les redémarrages, vous devrez donc peut-être rafraîchir la règle de redirection. Exemple (PowerShell **en tant qu'Administrateur**) :

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Autorisez le port via le Pare-feu Windows (une fois) :

```
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

Rafraîchissez le portproxy après les redémarrages de WSL :

```
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

Notes :

-   Le SSH depuis une autre machine cible **l'IP de l'hôte Windows** (exemple : `ssh user@windows-host -p 2222`).
-   Les nœuds distants doivent pointer vers une URL de Passerelle **accessible** (pas `127.0.0.1`) ; utilisez `openclaw status --all` pour confirmer.
-   Utilisez `listenaddress=0.0.0.0` pour l'accès LAN ; `127.0.0.1` le garde local uniquement.
-   Si vous voulez que ce soit automatique, enregistrez une tâche planifiée pour exécuter l'étape de rafraîchissement à la connexion.

## Installation étape par étape de WSL2

### 1) Installer WSL2 + Ubuntu

Ouvrez PowerShell (Admin) :

```bash
wsl --install
# Ou choisissez une distribution explicitement :
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Redémarrez si Windows le demande.

### 2) Activer systemd (requis pour l'installation de la passerelle)

Dans votre terminal WSL :

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Puis depuis PowerShell :

```bash
wsl --shutdown
```

Rouvrez Ubuntu, puis vérifiez :

```bash
systemctl --user status
```

### 3) Installer OpenClaw (dans WSL)

Suivez le flux Premiers pas pour Linux dans WSL :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installe automatiquement les dépendances de l'interface utilisateur au premier lancement
pnpm build
openclaw onboard
```

Guide complet : [Premiers pas](../start/getting-started.md)

## Application compagnon Windows

Nous n'avons pas encore d'application compagnon Windows. Les contributions sont les bienvenues si vous souhaitez contribuer à sa réalisation.

[Application Linux](./linux.md)[Application Android](./android.md)