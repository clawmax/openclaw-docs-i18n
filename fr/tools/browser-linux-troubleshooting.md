

  Navigateur

  
# Dépannage du Navigateur

## Problème : « Échec du démarrage de Chrome CDP sur le port 18800 »

Le serveur de contrôle du navigateur d'OpenClaw échoue à lancer Chrome/Brave/Edge/Chromium avec l'erreur :

```json
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```

### Cause Racine

Sur Ubuntu (et de nombreuses distributions Linux), l'installation par défaut de Chromium est un **paquet snap**. Le confinement AppArmor de Snap interfère avec la façon dont OpenClaw lance et surveille le processus du navigateur. La commande `apt install chromium` installe un paquet factice qui redirige vers snap :

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

Ce n'est PAS un vrai navigateur — ce n'est qu'un wrapper.

### Solution 1 : Installer Google Chrome (Recommandé)

Installez le paquet officiel `.deb` de Google Chrome, qui n'est pas sandboxé par snap :

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # si des erreurs de dépendance surviennent
```

Puis mettez à jour votre configuration OpenClaw (`~/.openclaw/openclaw.json`) :

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### Solution 2 : Utiliser Chromium Snap en Mode Attach-Only

Si vous devez utiliser Chromium snap, configurez OpenClaw pour qu'il se connecte à un navigateur démarré manuellement :

1.  Mettez à jour la configuration :

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2.  Démarrez Chromium manuellement :

```
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3.  Optionnellement, créez un service utilisateur systemd pour démarrer Chrome automatiquement :

```bash
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Activez avec : `systemctl --user enable --now openclaw-browser.service`

### Vérifier que le Navigateur Fonctionne

Vérifiez le statut :

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Testez la navigation :

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

### Référence de Configuration

| Option | Description | Par défaut |
| --- | --- | --- |
| `browser.enabled` | Activer le contrôle du navigateur | `true` |
| `browser.executablePath` | Chemin vers un binaire de navigateur basé sur Chromium (Chrome/Brave/Edge/Chromium) | détection automatique (préfère le navigateur par défaut s'il est basé sur Chromium) |
| `browser.headless` | Exécuter sans interface graphique | `false` |
| `browser.noSandbox` | Ajouter le flag `--no-sandbox` (nécessaire pour certaines configurations Linux) | `false` |
| `browser.attachOnly` | Ne pas lancer le navigateur, seulement s'y connecter | `false` |
| `browser.cdpPort` | Port du protocole Chrome DevTools | `18800` |

### Problème : « Le relais de l'extension Chrome est en cours d'exécution, mais aucun onglet n'est connecté »

Vous utilisez le profil `chrome` (relais d'extension). Il s'attend à ce que l'extension de navigateur OpenClaw soit attachée à un onglet actif. Options de correction :

1.  **Utiliser le navigateur géré :** `openclaw browser start --browser-profile openclaw` (ou définir `browser.defaultProfile: "openclaw"`).
2.  **Utiliser le relais d'extension :** installez l'extension, ouvrez un onglet et cliquez sur l'icône de l'extension OpenClaw pour l'attacher.

Notes :

-   Le profil `chrome` utilise votre **navigateur Chromium par défaut du système** lorsque c'est possible.
-   Les profils locaux `openclaw` attribuent automatiquement `cdpPort`/`cdpUrl` ; ne définissez ceux-ci que pour un CDP distant.

[Extension Chrome](./chrome-extension.md)[Agent Send](./agent-send.md)