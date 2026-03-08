title: "Checklist de publication et guide de versionnement d'OpenClaw"
description: "Guide étape par étape pour publier et distribuer les versions d'OpenClaw. Apprenez les vérifications préalables, le processus de build, la publication sur npm et la distribution de l'application macOS."
keywords: ["publication openclaw", "checklist de publication", "publication npm", "versionnement", "flux sparkle appcast", "distribution application macos", "journal des modifications", "publication de plugin"]
---

  Notes de version

  
# Checklist de publication

Utilisez `pnpm` (Node 22+) depuis la racine du dépôt. Gardez l'arbre de travail propre avant le tagging/la publication.

## Déclenchement par l'opérateur

Quand l'opérateur dit "release", effectuez immédiatement cette vérification préalable (pas de questions supplémentaires sauf blocage) :

-   Lisez ce document et `docs/platforms/mac/release.md`.
-   Chargez les variables d'environnement depuis `~/.profile` et confirmez que `SPARKLE_PRIVATE_KEY_FILE` + les variables App Store Connect sont définies (SPARKLE\_PRIVATE\_KEY\_FILE doit se trouver dans `~/.profile`).
-   Utilisez les clés Sparkle depuis `~/Library/CloudStorage/Dropbox/Backup/Sparkle` si nécessaire.

1.  **Version & métadonnées**

-   [ ]  Incrémentez la version dans `package.json` (ex : `2026.1.29`).
-   [ ]  Exécutez `pnpm plugins:sync` pour aligner les versions des packages d'extensions + les journaux des modifications.
-   [ ]  Mettez à jour les chaînes CLI/version dans [`src/version.ts`](https://github.com/openclaw/openclaw/blob/main/src/version.ts) et l'agent utilisateur Baileys dans [`src/web/session.ts`](https://github.com/openclaw/openclaw/blob/main/src/web/session.ts).
-   [ ]  Confirmez les métadonnées du package (nom, description, repository, keywords, license) et que la map `bin` pointe vers [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) pour `openclaw`.
-   [ ]  Si les dépendances ont changé, exécutez `pnpm install` pour que `pnpm-lock.yaml` soit à jour.

2.  **Build & artefacts**

-   [ ]  Si les entrées A2UI ont changé, exécutez `pnpm canvas:a2ui:bundle` et committez toute mise à jour de [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js).
-   [ ]  `pnpm run build` (régénère `dist/`).
-   [ ]  Vérifiez que le package npm `files` inclut tous les dossiers `dist/*` requis (notamment `dist/node-host/**` et `dist/acp/**` pour le mode headless node + CLI ACP).
-   [ ]  Confirmez que `dist/build-info.json` existe et inclut le hash `commit` attendu (la bannière CLI l'utilise pour les installations npm).
-   [ ]  Optionnel : `npm pack --pack-destination /tmp` après le build ; inspectez le contenu du tarball et gardez-le à portée de main pour la release GitHub (ne **pas** le committer).

3.  **Journal des modifications & documentation**

-   [ ]  Mettez à jour `CHANGELOG.md` avec les points forts pour l'utilisateur (créez le fichier s'il manque) ; gardez les entrées strictement décroissantes par version.
-   [ ]  Assurez-vous que les exemples/options du README correspondent au comportement actuel du CLI (notamment les nouvelles commandes ou options).

4.  **Validation**

-   [ ]  `pnpm build`
-   [ ]  `pnpm check`
-   [ ]  `pnpm test` (ou `pnpm test:coverage` si vous avez besoin d'un rapport de couverture)
-   [ ]  `pnpm release:check` (vérifie le contenu du npm pack)
-   [ ]  `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (test de fumée d'installation Docker, chemin rapide ; requis avant la publication)
    -   Si la version npm précédente immédiate est connue comme cassée, définissez `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` ou `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` pour l'étape de pré-installation.
-   [ ]  (Optionnel) Test de fumée complet de l'installateur (ajoute la couverture non-root + CLI) : `pnpm test:install:smoke`
-   [ ]  (Optionnel) E2E de l'installateur (Docker, exécute `curl -fsSL https://openclaw.ai/install.sh | bash`, procède à l'onboarding, puis exécute des appels réels de l'outil) :
    -   `pnpm test:install:e2e:openai` (requiert `OPENAI_API_KEY`)
    -   `pnpm test:install:e2e:anthropic` (requiert `ANTHROPIC_API_KEY`)
    -   `pnpm test:install:e2e` (requiert les deux clés ; exécute les deux fournisseurs)
-   [ ]  (Optionnel) Vérification ponctuelle de la passerelle web si vos changements affectent les chemins d'envoi/réception.

5.  **Application macOS (Sparkle)**

-   [ ]  Build + signez l'application macOS, puis ziprez-la pour la distribution.
-   [ ]  Générez le flux Sparkle appcast (notes HTML via [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) et mettez à jour `appcast.xml`.
-   [ ]  Gardez le zip de l'application (et le zip dSYM optionnel) prêt à être attaché à la release GitHub.
-   [ ]  Suivez [macOS release](../platforms/mac/release.md) pour les commandes exactes et les variables d'environnement requises.
    -   `APP_BUILD` doit être numérique + monotone (pas de `-beta`) pour que Sparkle compare correctement les versions.
    -   Si vous notarisez, utilisez le profil de trousseau `openclaw-notary` créé à partir des variables d'environnement de l'API App Store Connect (voir [macOS release](../platforms/mac/release.md)).

6.  **Publication (npm)**

-   [ ]  Confirmez que le statut git est propre ; committez et poussez si nécessaire.
-   [ ]  `npm login` (vérifiez la 2FA) si nécessaire.
-   [ ]  `npm publish --access public` (utilisez `--tag beta` pour les pré-versions).
-   [ ]  Vérifiez le registre : `npm view openclaw version`, `npm view openclaw dist-tags`, et `npx -y openclaw@X.Y.Z --version` (ou `--help`).

### Dépannage (notes de la version 2.0.0-beta2)

-   **npm pack/publish se bloque ou produit un tarball énorme** : le bundle de l'application macOS dans `dist/OpenClaw.app` (et les zips de release) sont inclus dans le package. Corrigez en listant explicitement le contenu à publier via `package.json` `files` (inclure les sous-répertoires dist, docs, skills ; exclure les bundles d'applications). Confirmez avec `npm pack --dry-run` que `dist/OpenClaw.app` n'est pas listé.
-   **Boucle d'authentification web npm pour les dist-tags** : utilisez l'authentification legacy pour obtenir une invite OTP :
    -   `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
-   **La vérification `npx` échoue avec `ECOMPROMISED: Lock compromised`** : réessayez avec un cache frais :
    -   `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
-   **Le tag doit être redirigé après une correction tardive** : forcez la mise à jour et poussez le tag, puis assurez-vous que les assets de la release GitHub correspondent toujours :
    -   `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7.  **Release GitHub + appcast**

-   [ ]  Taggez et poussez : `git tag vX.Y.Z && git push origin vX.Y.Z` (ou `git push --tags`).
-   [ ]  Créez/rafraîchissez la release GitHub pour `vX.Y.Z` avec le **titre `openclaw X.Y.Z`** (pas seulement le tag) ; le corps doit inclure la section **complète** du journal des modifications pour cette version (Points forts + Changements + Corrections), en ligne (pas de liens nus), et **ne doit pas répéter le titre dans le corps**.
-   [ ]  Attachez les artefacts : tarball `npm pack` (optionnel), `OpenClaw-X.Y.Z.zip`, et `OpenClaw-X.Y.Z.dSYM.zip` (si généré).
-   [ ]  Committez le `appcast.xml` mis à jour et poussez-le (Sparkle se nourrit depuis main).
-   [ ]  Depuis un répertoire temporaire propre (sans `package.json`), exécutez `npx -y openclaw@X.Y.Z send --help` pour confirmer que les points d'entrée d'installation/CLI fonctionnent.
-   [ ]  Annoncez/partagez les notes de version.

## Périmètre de publication des plugins (npm)

Nous ne publions que **les plugins npm existants** sous le scope `@openclaw/*`. Les plugins intégrés qui ne sont pas sur npm restent **uniquement sur l'arborescence disque** (toujours livrés dans `extensions/**`). Processus pour obtenir la liste :

1.  `npm search @openclaw --json` et capturez les noms des packages.
2.  Comparez avec les noms dans `extensions/*/package.json`.
3.  Publiez uniquement **l'intersection** (déjà sur npm).

Liste actuelle des plugins npm (à mettre à jour si nécessaire) :

-   @openclaw/bluebubbles
-   @openclaw/diagnostics-otel
-   @openclaw/discord
-   @openclaw/feishu
-   @openclaw/lobster
-   @openclaw/matrix
-   @openclaw/msteams
-   @openclaw/nextcloud-talk
-   @openclaw/nostr
-   @openclaw/voice-call
-   @openclaw/zalo
-   @openclaw/zalouser

Les notes de version doivent également mentionner **les nouveaux plugins optionnels intégrés** qui sont **désactivés par défaut** (exemple : `tlon`).

[Crédits](./credits.md)[Tests](./test.md)