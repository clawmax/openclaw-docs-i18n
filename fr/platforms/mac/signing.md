title: "Guide de signature d'application macOS pour les permissions stables d'OpenClaw"
description: "Apprenez à signer l'application macOS OpenClaw pour des permissions TCC stables. Configurez les identités de signature, gérez les builds ad-hoc et les identifiants de bundle."
keywords: ["signature macos", "codesign", "permissions tcc", "identifiant de bundle", "developer id", "signature ad-hoc", "runtime renforcé", "application macos"]
---

  Application compagnon macOS

  
# Signature macOS

Cette application est généralement construite depuis [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh), qui maintenant :

-   définit un identifiant de bundle de débogage stable : `ai.openclaw.mac.debug`
-   écrit le Info.plist avec cet identifiant de bundle (peut être écrasé via `BUNDLE_ID=...`)
-   appelle [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) pour signer le binaire principal et le bundle d'application afin que macOS traite chaque reconstruction comme le même bundle signé et conserve les permissions TCC (notifications, accessibilité, enregistrement d'écran, micro, parole). Pour des permissions stables, utilisez une véritable identité de signature ; la signature ad-hoc est optionnelle et fragile (voir [Permissions macOS](./permissions.md)).
-   utilise `CODESIGN_TIMESTAMP=auto` par défaut ; cela active les horodatages de confiance pour les signatures Developer ID. Définissez `CODESIGN_TIMESTAMP=off` pour ignorer l'horodatage (builds de débogage hors ligne).
-   injecte des métadonnées de build dans Info.plist : `OpenClawBuildTimestamp` (UTC) et `OpenClawGitCommit` (hash court) afin que le panneau À propos puisse afficher le build, git, et le canal debug/release.
-   **L'empaquetage nécessite Node 22+** : le script exécute les builds TS et le build de l'interface utilisateur de contrôle.
-   lit `SIGN_IDENTITY` depuis l'environnement. Ajoutez `export SIGN_IDENTITY="Apple Development: Votre Nom (TEAMID)"` (ou votre certificat Developer ID Application) à votre fichier rc du shell pour toujours signer avec votre certificat. La signature ad-hoc nécessite une activation explicite via `ALLOW_ADHOC_SIGNING=1` ou `SIGN_IDENTITY="-"` (non recommandé pour les tests de permissions).
-   exécute un audit d'ID d'équipe après la signature et échoue si un Mach-O à l'intérieur du bundle d'application est signé par un ID d'équipe différent. Définissez `SKIP_TEAM_ID_CHECK=1` pour contourner.

## Utilisation

```bash
# depuis la racine du dépôt
scripts/package-mac-app.sh               # sélectionne automatiquement l'identité ; erreur si aucune trouvée
SIGN_IDENTITY="Developer ID Application: Votre Nom" scripts/package-mac-app.sh   # vrai certificat
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (les permissions ne persisteront pas)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # ad-hoc explicite (même mise en garde)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # contournement de l'incompatibilité d'ID d'équipe Sparkle (développement uniquement)
```

### Note sur la signature Ad-hoc

Lors de la signature avec `SIGN_IDENTITY="-"` (ad-hoc), le script désactive automatiquement le **Runtime Renforcé** (`--options runtime`). Ceci est nécessaire pour éviter les plantages lorsque l'application tente de charger des frameworks intégrés (comme Sparkle) qui ne partagent pas le même ID d'équipe. Les signatures ad-hoc cassent également la persistance des permissions TCC ; consultez [Permissions macOS](./permissions.md) pour les étapes de récupération.

## Métadonnées de build pour À propos

`package-mac-app.sh` marque le bundle avec :

-   `OpenClawBuildTimestamp` : ISO8601 UTC au moment de l'empaquetage
-   `OpenClawGitCommit` : hash git court (ou `unknown` si indisponible)

L'onglet À propos lit ces clés pour afficher la version, la date de build, le commit git, et s'il s'agit d'un build de débogage (via `#if DEBUG`). Exécutez l'outil d'empaquetage pour rafraîchir ces valeurs après des modifications de code.

## Pourquoi

Les permissions TCC sont liées à l'identifiant de bundle *et* à la signature du code. Les builds de débogage non signés avec des UUID changeants faisaient que macOS oubliait les autorisations après chaque reconstruction. Signer les binaires (ad‑hoc par défaut) et conserver un identifiant/chemin de bundle fixe (`dist/OpenClaw.app`) préserve les autorisations entre les builds, suivant l'approche de VibeTunnel.

[Contrôle à distance](./remote.md)[Version macOS](./release.md)

---