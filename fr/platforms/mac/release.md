

  Application compagnon macOS

  
# Release macOS

Cette application embarque désormais les mises à jour automatiques Sparkle. Les builds de release doivent être signés avec un Developer ID, zippés et publiés avec une entrée d'appcast signée.

## Prérequis

-   Certificat Developer ID Application installé (exemple : `Developer ID Application:  ()`).
-   Chemin de la clé privée Sparkle défini dans l'environnement comme `SPARKLE_PRIVATE_KEY_FILE` (chemin vers votre clé privée ed25519 Sparkle ; la clé publique est intégrée dans Info.plist). Si elle est manquante, vérifiez `~/.profile`.
-   Identifiants de notarisation (profil keychain ou clé API) pour `xcrun notarytool` si vous souhaitez une distribution DMG/zip sécurisée par Gatekeeper.
    -   Nous utilisons un profil Keychain nommé `openclaw-notary`, créé à partir des variables d'environnement de clé API App Store Connect dans votre profil shell :
        -   `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
        -   `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
        -   `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
-   Dépendances `pnpm` installées (`pnpm install --config.node-linker=hoisted`).
-   Les outils Sparkle sont récupérés automatiquement via SwiftPM dans `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` (`sign_update`, `generate_appcast`, etc.).

## Construction & packaging

Notes :

-   `APP_BUILD` correspond à `CFBundleVersion`/`sparkle:version` ; conservez-le numérique + monotone (pas de `-beta`), sinon Sparkle le compare comme égal.
-   Si `APP_BUILD` est omis, `scripts/package-mac-app.sh` dérive une valeur par défaut compatible Sparkle à partir de `APP_VERSION` (`YYYYMMDDNN` : les versions stables utilisent par défaut `90`, les pré-releases utilisent un canal dérivé du suffixe) et utilise la valeur la plus élevée entre celle-ci et le nombre de commits git.
-   Vous pouvez toujours écraser `APP_BUILD` explicitement lorsque l'ingénierie de release nécessite une valeur monotone spécifique.
-   Par défaut, utilise l'architecture actuelle (`$(uname -m)`). Pour les builds universels/de release, définissez `BUILD_ARCHS="arm64 x86_64"` (ou `BUILD_ARCHS=all`).
-   Utilisez `scripts/package-mac-dist.sh` pour les artefacts de release (zip + DMG + notarisation). Utilisez `scripts/package-mac-app.sh` pour le packaging local/de développement.

```bash
# Depuis la racine du dépôt ; définissez les identifiants de release pour activer le flux Sparkle.
# APP_BUILD doit être numérique + monotone pour la comparaison Sparkle.
# La valeur par défaut est auto-dérivée de APP_VERSION lorsqu'elle est omise.
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# Zip pour distribution (inclut les resource forks pour la prise en charge des deltas Sparkle)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.3.7.zip

# Optionnel : construisez également un DMG stylisé pour les humains (glisser-déposer vers /Applications)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.3.7.dmg

# Recommandé : construire + notariser/agrapher zip + DMG
# D'abord, créez un profil keychain une fois :
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# Optionnel : livrez le dSYM avec la release
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.3.7.dSYM.zip
```

## Entrée d'appcast

Utilisez le générateur de notes de release pour que Sparkle affiche des notes HTML formatées :

```
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.3.7.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

Génère les notes de release HTML à partir de `CHANGELOG.md` (via [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)) et les intègre dans l'entrée de l'appcast. Commitez le fichier `appcast.xml` mis à jour aux côtés des artefacts de release (zip + dSYM) lors de la publication.

## Publier & vérifier

-   Téléversez `OpenClaw-2026.3.7.zip` (et `OpenClaw-2026.3.7.dSYM.zip`) dans la release GitHub pour le tag `v2026.3.7`.
-   Assurez-vous que l'URL brute de l'appcast correspond au flux intégré : `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
-   Vérifications de bon sens :
    -   `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` retourne 200.
    -   `curl -I ` retourne 200 après le téléversement des artefacts.
    -   Sur une build publique précédente, exécutez "Vérifier les mises à jour…" depuis l'onglet À propos et vérifiez que Sparkle installe la nouvelle build proprement.

Définition de terminé : l'application signée et l'appcast sont publiés, le flux de mise à jour fonctionne depuis une version installée plus ancienne, et les artefacts de release sont attachés à la release GitHub.

[Signature macOS](./signing.md)[Gateway sur macOS](./bundled-gateway.md)