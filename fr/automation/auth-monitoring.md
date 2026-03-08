title: "Surveillance Automatisée de l'Authentification pour l'Expiration OAuth d'OpenClaw"
description: "Apprenez à automatiser la surveillance et l'alerte des identifiants OAuth pour OpenClaw en utilisant des vérifications CLI et des scripts optionnels pour les workflows systemd et mobiles."
keywords: ["openclaw", "surveillance oauth", "automatisation", "expiration auth", "vérification cli", "minuterie systemd", "alerte", "scripts"]
---

  Automatisation

  
# Surveillance de l'Authentification

OpenClaw expose l'état de santé d'expiration OAuth via `openclaw models status`. Utilisez-le pour l'automatisation et les alertes ; les scripts sont des extras optionnels pour les workflows téléphone.

## Préféré : Vérification CLI (portable)

```bash
openclaw models status --check
```

Codes de sortie :

-   `0` : OK
-   `1` : identifiants expirés ou manquants
-   `2` : expiration prochaine (dans les 24h)

Cela fonctionne dans cron/systemd et ne nécessite aucun script supplémentaire.

## Scripts optionnels (workflows ops / téléphone)

Ils se trouvent dans `scripts/` et sont **optionnels**. Ils supposent un accès SSH à l'hôte passerelle et sont ajustés pour systemd + Termux.

-   `scripts/claude-auth-status.sh` utilise désormais `openclaw models status --json` comme source de vérité (avec repli sur la lecture directe des fichiers si la CLI est indisponible), gardez donc `openclaw` dans le `PATH` pour les minuteries.
-   `scripts/auth-monitor.sh` : cible de minuterie cron/systemd ; envoie des alertes (ntfy ou téléphone).
-   `scripts/systemd/openclaw-auth-monitor.{service,timer}` : minuterie utilisateur systemd.
-   `scripts/claude-auth-status.sh` : vérificateur d'authentification Claude Code + OpenClaw (complet/json/simple).
-   `scripts/mobile-reauth.sh` : flux de ré-authentification guidé via SSH.
-   `scripts/termux-quick-auth.sh` : widget à un clic pour le statut + ouverture de l'URL d'authentification.
-   `scripts/termux-auth-widget.sh` : flux widget guidé complet.
-   `scripts/termux-sync-widget.sh` : synchronisation des identifiants Claude Code → OpenClaw.

Si vous n'avez pas besoin d'automatisation téléphone ou de minuteries systemd, ignorez ces scripts.

[Sondages](./poll.md)[Nœuds](../nodes.md)