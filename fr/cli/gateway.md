title: "Guide et référence des commandes CLI de la passerelle OpenClaw"
description: "Apprenez à exécuter, interroger, gérer et découvrir le serveur WebSocket de la passerelle OpenClaw à l'aide des commandes CLI. Inclut les options, la gestion des services et l'utilisation RPC."
keywords: ["passerelle openclaw", "serveur websocket", "commandes cli", "gestion de passerelle", "découverte de passerelle", "rpc de passerelle", "service de passerelle", "découverte bonjour"]
---

  Commandes CLI

  
# gateway

La Passerelle est le serveur WebSocket d'OpenClaw (canaux, nœuds, sessions, hooks). Les sous-commandes de cette page se trouvent sous `openclaw gateway …`. Documentation associée :

-   [/gateway/bonjour](../gateway/bonjour.md)
-   [/gateway/discovery](../gateway/discovery.md)
-   [/gateway/configuration](../gateway/configuration.md)

## Exécuter la Passerelle

Exécutez un processus de Passerelle local :

```bash
openclaw gateway
```

Alias en premier plan :

```bash
openclaw gateway run
```

Notes :

-   Par défaut, la Passerelle refuse de démarrer sauf si `gateway.mode=local` est défini dans `~/.openclaw/openclaw.json`. Utilisez `--allow-unconfigured` pour des exécutions ad-hoc/de développement.
-   La liaison au-delà de la boucle locale sans authentification est bloquée (mesure de sécurité).
-   `SIGUSR1` déclenche un redémarrage dans le processus lorsqu'il est autorisé (`commands.restart` est activé par défaut ; définissez `commands.restart: false` pour bloquer le redémarrage manuel, tandis que l'application/mise à jour de l'outil/de la configuration de la passerelle restent autorisées).
-   Les gestionnaires `SIGINT`/`SIGTERM` arrêtent le processus de la passerelle, mais ils ne restaurent aucun état personnalisé du terminal. Si vous encapsulez la CLI avec une TUI ou une entrée en mode brut, restaurez le terminal avant de quitter.

### Options

-   `--port ` : Port WebSocket (la valeur par défaut provient de la config/env ; généralement `18789`).
-   `--bind <loopback|lan|tailnet|auto|custom>` : mode de liaison de l'écouteur.
-   `--auth <token|password>` : remplacement du mode d'authentification.
-   `--token ` : remplacement du jeton (définit également `OPENCLAW_GATEWAY_TOKEN` pour le processus).
-   `--password ` : remplacement du mot de passe (définit également `OPENCLAW_GATEWAY_PASSWORD` pour le processus).
-   `--tailscale <off|serve|funnel>` : exposer la Passerelle via Tailscale.
-   `--tailscale-reset-on-exit` : réinitialiser la configuration Tailscale serve/funnel à l'arrêt.
-   `--allow-unconfigured` : autoriser le démarrage de la passerelle sans `gateway.mode=local` dans la configuration.
-   `--dev` : créer une configuration de développement + espace de travail s'ils sont manquants (ignore BOOTSTRAP.md).
-   `--reset` : réinitialiser la configuration de développement + les identifiants + les sessions + l'espace de travail (nécessite `--dev`).
-   `--force` : tuer tout écouteur existant sur le port sélectionné avant de démarrer.
-   `--verbose` : journaux détaillés.
-   `--claude-cli-logs` : n'afficher que les journaux claude-cli dans la console (et activer sa sortie stdout/stderr).
-   `--ws-log <auto|full|compact>` : style de journal WebSocket (par défaut `auto`).
-   `--compact` : alias pour `--ws-log compact`.
-   `--raw-stream` : journaliser les événements bruts du flux du modèle en jsonl.
-   `--raw-stream-path ` : chemin du fichier jsonl du flux brut.

## Interroger une Passerelle en cours d'exécution

Toutes les commandes d'interrogation utilisent le RPC WebSocket. Modes de sortie :

-   Par défaut : lisible par un humain (coloré dans un TTY).
-   `--json` : JSON lisible par une machine (pas de style/indicateur de progression).
-   `--no-color` (ou `NO_COLOR=1`) : désactiver ANSI tout en conservant la mise en page humaine.

Options partagées (lorsqu'elles sont prises en charge) :

-   `--url ` : URL WebSocket de la Passerelle.
-   `--token ` : Jeton de la Passerelle.
-   `--password ` : Mot de passe de la Passerelle.
-   `--timeout ` : délai d'attente/budget (varie selon la commande).
-   `--expect-final` : attendre une réponse "finale" (appels d'agent).

Note : lorsque vous définissez `--url`, la CLI ne revient pas aux identifiants de configuration ou d'environnement. Passez `--token` ou `--password` explicitement. L'absence d'identifiants explicites est une erreur.

### gateway health

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### gateway status

`gateway status` affiche le service de la Passerelle (launchd/systemd/schtasks) ainsi qu'une sonde RPC optionnelle.

```bash
openclaw gateway status
openclaw gateway status --json
```

Options :

-   `--url ` : remplacer l'URL de la sonde.
-   `--token ` : authentification par jeton pour la sonde.
-   `--password ` : authentification par mot de passe pour la sonde.
-   `--timeout ` : délai d'attente de la sonde (par défaut `10000`).
-   `--no-probe` : ignorer la sonde RPC (vue service uniquement).
-   `--deep` : scanner également les services au niveau du système.

Notes :

-   `gateway status` résout les SecretRefs d'authentification configurés pour l'authentification de la sonde lorsque c'est possible.
-   Si un SecretRef d'authentification requis n'est pas résolu dans ce chemin de commande, l'authentification de la sonde peut échouer ; passez `--token`/`--password` explicitement ou résolvez d'abord la source du secret.

### gateway probe

`gateway probe` est la commande "déboguer tout". Elle sonde toujours :

-   votre passerelle distante configurée (si définie), et
-   localhost (boucle locale) **même si une passerelle distante est configurée**.

Si plusieurs passerelles sont accessibles, elle les affiche toutes. Plusieurs passerelles sont prises en charge lorsque vous utilisez des profils/ports isolés (par exemple, un bot de secours), mais la plupart des installations exécutent toujours une seule passerelle.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

#### Distant via SSH (parité avec l'application Mac)

Le mode "Distant via SSH" de l'application macOS utilise un transfert de port local pour que la passerelle distante (qui peut être liée uniquement à la boucle locale) devienne accessible à `ws://127.0.0.1:`. Équivalent CLI :

```bash
openclaw gateway probe --ssh user@gateway-host
```

Options :

-   `--ssh ` : `user@host` ou `user@host:port` (le port par défaut est `22`).
-   `--ssh-identity ` : fichier d'identité.
-   `--ssh-auto` : choisir le premier hôte de passerelle découvert comme cible SSH (LAN/WAB uniquement).

Configuration (optionnelle, utilisée comme valeurs par défaut) :

-   `gateway.remote.sshTarget`
-   `gateway.remote.sshIdentity`

### gateway call &lt;method&gt;

Assistant RPC de bas niveau.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

## Gérer le service de la Passerelle

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Notes :

-   `gateway install` prend en charge `--port`, `--runtime`, `--token`, `--force`, `--json`.
-   Lorsque l'authentification par jeton nécessite un jeton et que `gateway.auth.token` est géré par SecretRef, `gateway install` valide que le SecretRef est résoluble mais ne persiste pas le jeton résolu dans les métadonnées d'environnement du service.
-   Si l'authentification par jeton nécessite un jeton et que le SecretRef de jeton configuré n'est pas résolu, l'installation échoue en mode fermé au lieu de persister un texte en clair de secours.
-   En mode d'authentification inféré, les variables d'environnement shell-only `OPENCLAW_GATEWAY_PASSWORD`/`CLAWDBOT_GATEWAY_PASSWORD` n'assouplissent pas les exigences de jeton d'installation ; utilisez une configuration durable (`gateway.auth.password` ou config `env`) lors de l'installation d'un service géré.
-   Si `gateway.auth.token` et `gateway.auth.password` sont tous deux configurés et que `gateway.auth.mode` n'est pas défini, l'installation est bloquée jusqu'à ce que le mode soit défini explicitement.
-   Les commandes de cycle de vie acceptent `--json` pour le scriptage.

## Découvrir des passerelles (Bonjour)

`gateway discover` scanne à la recherche de balises de Passerelle (`_openclaw-gw._tcp`).

-   Multicast DNS-SD : `local.`
-   Unicast DNS-SD (Bonjour étendu) : choisissez un domaine (exemple : `openclaw.internal.`) et configurez un DNS fractionné + un serveur DNS ; voir [/gateway/bonjour](../gateway/bonjour.md)

Seules les passerelles avec la découverte Bonjour activée (par défaut) diffusent la balise. Les enregistrements de découverte étendue incluent (TXT) :

-   `role` (indice de rôle de la passerelle)
-   `transport` (indice de transport, par exemple `gateway`)
-   `gatewayPort` (port WebSocket, généralement `18789`)
-   `sshPort` (port SSH ; par défaut `22` s'il n'est pas présent)
-   `tailnetDns` (nom d'hôte MagicDNS, lorsqu'il est disponible)
-   `gatewayTls` / `gatewayTlsSha256` (TLS activé + empreinte de certificat)
-   `cliPath` (indice optionnel pour les installations distantes)

### gateway discover

```bash
openclaw gateway discover
```

Options :

-   `--timeout ` : délai d'attente par commande (parcourir/résoudre) ; par défaut `2000`.
-   `--json` : sortie lisible par machine (désactive également le style/l'indicateur de progression).

Exemples :

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```

[doctor](./doctor.md)[health](./health.md)

---