

  Aperçu des plateformes

  
# Application Linux

La Passerelle est entièrement prise en charge sous Linux. **Node est l'environnement d'exécution recommandé**. Bun n'est pas recommandé pour la Passerelle (bugs WhatsApp/Telegram). Des applications compagnons natives pour Linux sont prévues. Les contributions sont les bienvenues si vous souhaitez aider à en construire une.

## Parcours rapide pour débutants (VPS)

1.  Installez Node 22+
2.  `npm i -g openclaw@latest`
3.  `openclaw onboard --install-daemon`
4.  Depuis votre ordinateur portable : `ssh -N -L 18789:127.0.0.1:18789 @<hôte>`
5.  Ouvrez `http://127.0.0.1:18789/` et collez votre jeton

Guide VPS étape par étape : [exe.dev](../install/exe-dev.md)

## Installation

-   [Premiers pas](../start/getting-started.md)
-   [Installation & mises à jour](../install/updating.md)
-   Flux optionnels : [Bun (expérimental)](../install/bun.md), [Nix](../install/nix.md), [Docker](../install/docker.md)

## Passerelle

-   [Runbook de la Passerelle](../gateway.md)
-   [Configuration](../gateway/configuration.md)

## Installation du service de la Passerelle (CLI)

Utilisez l'une de ces commandes :

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

Sélectionnez **Service de la Passerelle** lorsque vous y êtes invité. Réparer/migrer :

```bash
openclaw doctor
```

## Contrôle système (unité utilisateur systemd)

OpenClaw installe par défaut un service **utilisateur** systemd. Utilisez un service **système** pour les serveurs partagés ou toujours actifs. L'exemple complet de l'unité et les conseils se trouvent dans le [Runbook de la Passerelle](../gateway.md). Configuration minimale : Créez `~/.config/systemd/user/openclaw-gateway[-].service` :

```ini
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Activez-le :

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

[Application macOS](./macos.md)[Windows (WSL2)](./windows.md)