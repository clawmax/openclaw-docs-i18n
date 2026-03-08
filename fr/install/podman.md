

  Autres méthodes d'installation

  
# Podman

Exécutez la passerelle OpenClaw dans un conteneur Podman **rootless**. Utilise la même image que Docker (construite à partir du [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) du dépôt).

## Prérequis

-   Podman (rootless)
-   Sudo pour la configuration unique (création d'utilisateur, construction de l'image)

## Démarrage rapide

**1\. Configuration unique** (depuis la racine du dépôt ; crée l'utilisateur, construit l'image, installe le script de lancement) :

```
./setup-podman.sh
```

Cela crée également un fichier minimal `~openclaw/.openclaw/openclaw.json` (définit `gateway.mode="local"`) afin que la passerelle puisse démarrer sans exécuter l'assistant. Par défaut, le conteneur **n'est pas** installé comme un service systemd, vous le démarrez manuellement (voir ci-dessous). Pour une configuration de type production avec démarrage automatique et redémarrages, installez-le plutôt comme un service utilisateur systemd Quadlet :

```bash
./setup-podman.sh --quadlet
```

(Ou définissez `OPENCLAW_PODMAN_QUADLET=1` ; utilisez `--container` pour installer uniquement le conteneur et le script de lancement.) Variables d'environnement optionnelles au moment de la construction (à définir avant d'exécuter `setup-podman.sh`) :

-   `OPENCLAW_DOCKER_APT_PACKAGES` — installe des paquets apt supplémentaires pendant la construction de l'image
-   `OPENCLAW_EXTENSIONS` — pré-installe les dépendances des extensions (noms d'extensions séparés par des espaces, p. ex. `diagnostics-otel matrix`)

**2\. Démarrer la passerelle** (manuel, pour un test rapide) :

```bash
./scripts/run-openclaw-podman.sh launch
```

**3\. Assistant d'intégration** (p. ex. pour ajouter des canaux ou des fournisseurs) :

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Ouvrez ensuite `http://127.0.0.1:18789/` et utilisez le jeton provenant de `~openclaw/.openclaw/.env` (ou la valeur affichée par l'assistant).

## Systemd (Quadlet, optionnel)

Si vous avez exécuté `./setup-podman.sh --quadlet` (ou `OPENCLAW_PODMAN_QUADLET=1`), une unité [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) est installée afin que la passerelle s'exécute comme un service utilisateur systemd pour l'utilisateur openclaw. Le service est activé et démarré à la fin de la configuration.

-   **Démarrer :** `sudo systemctl --machine openclaw@ --user start openclaw.service`
-   **Arrêter :** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
-   **Statut :** `sudo systemctl --machine openclaw@ --user status openclaw.service`
-   **Journaux :** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Le fichier quadlet se trouve à `~openclaw/.config/containers/systemd/openclaw.container`. Pour modifier les ports ou les variables d'environnement, éditez ce fichier (ou le `.env` qu'il source), puis exécutez `sudo systemctl --machine openclaw@ --user daemon-reload` et redémarrez le service. Au démarrage, le service démarre automatiquement si le lingering est activé pour openclaw (la configuration le fait si loginctl est disponible). Pour ajouter quadlet **après** une configuration initiale qui ne l'utilisait pas, ré-exécutez : `./setup-podman.sh --quadlet`.

## L'utilisateur openclaw (non-login)

`setup-podman.sh` crée un utilisateur système dédié `openclaw` :

-   **Shell :** `nologin` — pas de connexion interactive ; réduit la surface d'attaque.
-   **Répertoire personnel :** p. ex. `/home/openclaw` — contient `~/.openclaw` (configuration, espace de travail) et le script de lancement `run-openclaw-podman.sh`.
-   **Podman Rootless :** L'utilisateur doit avoir une plage **subuid** et **subgid**. De nombreuses distributions les attribuent automatiquement lors de la création de l'utilisateur. Si la configuration affiche un avertissement, ajoutez des lignes dans `/etc/subuid` et `/etc/subgid` :
    
    Copier
    
    ```
    openclaw:100000:65536
    ```
    
    Puis démarrez la passerelle en tant que cet utilisateur (p. ex. depuis cron ou systemd) :
    
    Copier
    
    ```bash
    sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
    sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
    ```
    
-   **Configuration :** Seuls `openclaw` et root peuvent accéder à `/home/openclaw/.openclaw`. Pour éditer la configuration : utilisez l'interface de contrôle une fois la passerelle en cours d'exécution, ou `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Environnement et configuration

-   **Jeton :** Stocké dans `~openclaw/.openclaw/.env` sous `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` et `run-openclaw-podman.sh` le génèrent s'il est manquant (utilise `openssl`, `python3`, ou `od`).
-   **Optionnel :** Dans ce `.env`, vous pouvez définir les clés des fournisseurs (p. ex. `GROQ_API_KEY`, `OLLAMA_API_KEY`) et d'autres variables d'environnement OpenClaw.
-   **Ports hôte :** Par défaut, le script mappe `18789` (passerelle) et `18790` (pont). Remplacez le mappage de port **hôte** avec `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` et `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` lors du lancement.
-   **Liaison de la passerelle :** Par défaut, `run-openclaw-podman.sh` démarre la passerelle avec `--bind loopback` pour un accès local sécurisé. Pour exposer sur le réseau local, définissez `OPENCLAW_GATEWAY_BIND=lan` et configurez `gateway.controlUi.allowedOrigins` (ou activez explicitement le fallback de l'en-tête hôte) dans `openclaw.json`.
-   **Chemins :** La configuration hôte et l'espace de travail sont par défaut `~openclaw/.openclaw` et `~openclaw/.openclaw/workspace`. Remplacez les chemins hôte utilisés par le script de lancement avec `OPENCLAW_CONFIG_DIR` et `OPENCLAW_WORKSPACE_DIR`.

## Modèle de stockage

-   **Données hôte persistantes :** `OPENCLAW_CONFIG_DIR` et `OPENCLAW_WORKSPACE_DIR` sont montés en bind dans le conteneur et conservent l'état sur l'hôte.
-   **Sandbox tmpfs éphémère :** si vous activez `agents.defaults.sandbox`, les conteneurs de sandbox d'outils montent `tmpfs` sur `/tmp`, `/var/tmp`, et `/run`. Ces chemins sont en mémoire et disparaissent avec le conteneur de sandbox ; la configuration du conteneur Podman de niveau supérieur n'ajoute pas ses propres montages tmpfs.
-   **Points chauds de croissance du disque :** les principaux chemins à surveiller sont `media/`, `agents//sessions/sessions.json`, les fichiers JSONL de transcription, `cron/runs/*.jsonl`, et les journaux de fichiers rotatifs sous `/tmp/openclaw/` (ou votre `logging.file` configuré).

`setup-podman.sh` met maintenant en attente l'image tar dans un répertoire temporaire privé et affiche le répertoire de base choisi pendant la configuration. Pour les exécutions non-root, il n'accepte `TMPDIR` que lorsque cette base est sûre à utiliser ; sinon, il revient à `/var/tmp`, puis `/tmp`. Le tar sauvegardé reste propriétaire uniquement et est transmis en flux dans le `podman load` de l'utilisateur cible, de sorte que les répertoires temporaires privés de l'appelant ne bloquent pas la configuration.

## Commandes utiles

-   **Journaux :** Avec quadlet : `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Avec le script : `sudo -u openclaw podman logs -f openclaw`
-   **Arrêter :** Avec quadlet : `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Avec le script : `sudo -u openclaw podman stop openclaw`
-   **Redémarrer :** Avec quadlet : `sudo systemctl --machine openclaw@ --user start openclaw.service`. Avec le script : ré-exécutez le script de lancement ou `podman start openclaw`
-   **Supprimer le conteneur :** `sudo -u openclaw podman rm -f openclaw` — la configuration et l'espace de travail sur l'hôte sont conservés

## Dépannage

-   **Permission refusée (EACCES) sur la configuration ou les profils d'authentification :** Le conteneur utilise par défaut `--userns=keep-id` et s'exécute avec le même uid/gid que l'utilisateur hôte exécutant le script. Assurez-vous que vos `OPENCLAW_CONFIG_DIR` et `OPENCLAW_WORKSPACE_DIR` hôtes appartiennent à cet utilisateur.
-   **Démarrage de la passerelle bloqué (manque `gateway.mode=local`) :** Assurez-vous que `~openclaw/.openclaw/openclaw.json` existe et définit `gateway.mode="local"`. `setup-podman.sh` crée ce fichier s'il est manquant.
-   **Podman Rootless échoue pour l'utilisateur openclaw :** Vérifiez que `/etc/subuid` et `/etc/subgid` contiennent une ligne pour `openclaw` (p. ex. `openclaw:100000:65536`). Ajoutez-la si elle est manquante et redémarrez.
-   **Nom de conteneur déjà utilisé :** Le script de lancement utilise `podman run --replace`, donc le conteneur existant est remplacé lorsque vous redémarrez. Pour nettoyer manuellement : `podman rm -f openclaw`.
-   **Script introuvable lors de l'exécution en tant qu'openclaw :** Assurez-vous que `setup-podman.sh` a été exécuté afin que `run-openclaw-podman.sh` soit copié dans le répertoire personnel d'openclaw (p. ex. `/home/openclaw/run-openclaw-podman.sh`).
-   **Service Quadlet introuvable ou échoue à démarrer :** Exécutez `sudo systemctl --machine openclaw@ --user daemon-reload` après avoir modifié le fichier `.container`. Quadlet nécessite cgroups v2 : `podman info --format '{{.Host.CgroupsVersion}}'` devrait afficher `2`.

## Optionnel : exécuter en tant que votre propre utilisateur

Pour exécuter la passerelle en tant que votre utilisateur normal (pas d'utilisateur openclaw dédié) : construisez l'image, créez `~/.openclaw/.env` avec `OPENCLAW_GATEWAY_TOKEN`, et exécutez le conteneur avec `--userns=keep-id` et des montages vers votre `~/.openclaw`. Le script de lancement est conçu pour le flux utilisateur openclaw ; pour une configuration mono-utilisateur, vous pouvez plutôt exécuter la commande `podman run` du script manuellement, en pointant la configuration et l'espace de travail vers votre répertoire personnel. Recommandé pour la plupart des utilisateurs : utilisez `setup-podman.sh` et exécutez en tant qu'utilisateur openclaw afin que la configuration et le processus soient isolés.

[Docker](./docker.md)[Nix](./nix.md)