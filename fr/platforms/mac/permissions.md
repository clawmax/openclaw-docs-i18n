title: "Guide des autorisations macOS pour les octrois stables de l'application OpenClaw"
description: "Apprenez à maintenir des autorisations macOS stables pour OpenClaw. Corrigez les invites qui disparaissent et réinitialisez TCC pour l'Accessibilité, la Capture d'écran et l'accès aux fichiers."
keywords: ["autorisations macos", "réinitialisation tcc", "octroi accessibilité", "capture d'écran", "identifiant de bundle", "signature de code", "openclaw mac", "confidentialité sécurité"]
---

  Application compagnon macOS

  
# Autorisations macOS

Les octrois d'autorisations macOS sont fragiles. TCC associe un octroi d'autorisation à la signature de code de l'application, à son identifiant de bundle et à son chemin sur le disque. Si l'un de ces éléments change, macOS considère l'application comme nouvelle et peut supprimer ou masquer les invites.

## Conditions requises pour des autorisations stables

-   Même chemin : exécutez l'application depuis un emplacement fixe (pour OpenClaw, `dist/OpenClaw.app`).
-   Même identifiant de bundle : changer l'ID de bundle crée une nouvelle identité d'autorisation.
-   Application signée : les builds non signés ou signés ad hoc ne conservent pas les autorisations.
-   Signature cohérente : utilisez un vrai certificat Apple Development ou Developer ID pour que la signature reste stable entre les reconstructions.

Les signatures ad hoc génèrent une nouvelle identité à chaque build. macOS oubliera les octrois précédents, et les invites peuvent disparaître complètement jusqu'à ce que les entrées obsolètes soient effacées.

## Liste de vérification de récupération lorsque les invites disparaissent

1.  Quittez l'application.
2.  Supprimez l'entrée de l'application dans Paramètres Système -> Confidentialité et sécurité.
3.  Relancez l'application depuis le même chemin et ré-accordez les autorisations.
4.  Si l'invite n'apparaît toujours pas, réinitialisez les entrées TCC avec `tccutil` et réessayez.
5.  Certaines autorisations ne réapparaissent qu'après un redémarrage complet de macOS.

Exemples de réinitialisations (remplacez l'identifiant de bundle si nécessaire) :

```bash
sudo tccutil reset Accessibility ai.openclaw.mac
sudo tccutil reset ScreenCapture ai.openclaw.mac
sudo tccutil reset AppleEvents
```

## Autorisations des fichiers et dossiers (Bureau/Documents/Téléchargements)

macOS peut aussi restreindre l'accès au Bureau, aux Documents et aux Téléchargements pour les processus de terminal/arrière-plan. Si les lectures de fichiers ou les listages de répertoires se bloquent, accordez l'accès au même contexte de processus qui effectue les opérations sur les fichiers (par exemple Terminal/iTerm, l'application lancée par un LaunchAgent, ou le processus SSH). Solution de contournement : déplacez les fichiers dans l'espace de travail OpenClaw (`~/.openclaw/workspace`) si vous souhaitez éviter les octrois par dossier. Si vous testez les autorisations, signez toujours avec un vrai certificat. Les builds ad hoc ne sont acceptables que pour des exécutions locales rapides où les autorisations n'ont pas d'importance.

[Journalisation macOS](./logging.md)[Contrôle à distance](./remote.md)