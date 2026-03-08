title: "Support des plateformes OpenClaw pour macOS iOS Android Windows Linux"
description: "Découvrez les plateformes prises en charge par OpenClaw, notamment macOS, Windows, Linux, iOS et Android. Trouvez des guides d'installation pour l'hébergement VPS et le service Gateway."
keywords: ["plateformes openclaw", "installation de la passerelle", "hébergement vps", "application compagnon macos", "windows wsl2", "service systemd linux", "nœuds mobiles", "aperçu des plateformes"]
---

  Aperçu des plateformes

  
# Plateformes

Le cœur d'OpenClaw est écrit en TypeScript. **Node est l'environnement d'exécution recommandé**. Bun n'est pas recommandé pour la Gateway (bugs WhatsApp/Telegram). Des applications compagnons existent pour macOS (application de barre de menus) et les nœuds mobiles (iOS/Android). Les applications compagnons pour Windows et Linux sont prévues, mais la Gateway est entièrement prise en charge aujourd'hui. Des applications compagnons natives pour Windows sont également prévues ; la Gateway est recommandée via WSL2.

## Choisissez votre OS

-   macOS : [macOS](./platforms/macos.md)
-   iOS : [iOS](./platforms/ios.md)
-   Android : [Android](./platforms/android.md)
-   Windows : [Windows](./platforms/windows.md)
-   Linux : [Linux](./platforms/linux.md)

## VPS & hébergement

-   Hub VPS : [Hébergement VPS](./vps.md)
-   Fly.io : [Fly.io](./install/fly.md)
-   Hetzner (Docker) : [Hetzner](./install/hetzner.md)
-   GCP (Compute Engine) : [GCP](./install/gcp.md)
-   exe.dev (VM + proxy HTTPS) : [exe.dev](./install/exe-dev.md)

## Liens courants

-   Guide d'installation : [Premiers pas](./start/getting-started.md)
-   Runbook de la Gateway : [Gateway](./gateway.md)
-   Configuration de la Gateway : [Configuration](./gateway/configuration.md)
-   État du service : `openclaw gateway status`

## Installation du service Gateway (CLI)

Utilisez l'une de ces méthodes (toutes prises en charge) :

-   Assistant (recommandé) : `openclaw onboard --install-daemon`
-   Direct : `openclaw gateway install`
-   Flux de configuration : `openclaw configure` → sélectionnez **Service Gateway**
-   Réparation/migration : `openclaw doctor` (propose d'installer ou de réparer le service)

La cible du service dépend de l'OS :

-   macOS : LaunchAgent (`ai.openclaw.gateway` ou `ai.openclaw.` ; ancienne version `com.openclaw.*`)
-   Linux/WSL2 : service utilisateur systemd (`openclaw-gateway[-].service`)

[Application macOS](./platforms/macos.md)

---