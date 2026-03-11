

  Avancé

  
# Canaux de développement

Dernière mise à jour : 2026-01-21 OpenClaw propose trois canaux de mise à jour :

-   **stable** : dist-tag npm `latest`.
-   **beta** : dist-tag npm `beta` (builds en test).
-   **dev** : tête courante de `main` (git). dist-tag npm : `dev` (lorsqu'il est publié).

Nous publions des builds vers **beta**, les testons, puis **promouvons un build validé vers `latest`** sans changer le numéro de version — les dist-tags sont la source de vérité pour les installations npm.

## Changer de canal

Git checkout :

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

-   `stable`/`beta` récupèrent le dernier tag correspondant (souvent le même tag).
-   `dev` bascule sur `main` et rebase sur l'upstream.

Installation globale npm/pnpm :

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

Cela met à jour via le dist-tag npm correspondant (`latest`, `beta`, `dev`). Lorsque vous **changez explicitement** de canal avec `--channel`, OpenClaw aligne également la méthode d'installation :

-   `dev` s'assure d'un git checkout (par défaut `~/openclaw`, à remplacer avec `OPENCLAW_GIT_DIR`), le met à jour, et installe le CLI global depuis ce checkout.
-   `stable`/`beta` installe depuis npm en utilisant le dist-tag correspondant.

Astuce : si vous voulez stable + dev en parallèle, gardez deux clones et pointez votre passerelle vers celui stable.

## Plugins et canaux

Lorsque vous changez de canal avec `openclaw update`, OpenClaw synchronise également les sources des plugins :

-   `dev` préfère les plugins fournis depuis le checkout git.
-   `stable` et `beta` restaurent les paquets de plugins installés via npm.

## Bonnes pratiques de tagging

-   Taguez les releases sur lesquelles vous voulez que les git checkouts atterrissent (`vYYYY.M.D` pour stable, `vYYYY.M.D-beta.N` pour beta).
-   `vYYYY.M.D.beta.N` est également reconnu pour la compatibilité, mais préférez `-beta.N`.
-   Les anciens tags `vYYYY.M.D-` sont toujours reconnus comme stable (non-beta).
-   Gardez les tags immuables : ne déplacez ou ne réutilisez jamais un tag.
-   Les dist-tags npm restent la source de vérité pour les installations npm :
    -   `latest` → stable
    -   `beta` → build candidat
    -   `dev` → snapshot de main (optionnel)

## Disponibilité de l'application macOS

Les builds beta et dev peuvent **ne pas** inclure une release d'application macOS. C'est normal :

-   Le tag git et le dist-tag npm peuvent toujours être publiés.
-   Indiquez "pas de build macOS pour cette beta" dans les notes de release ou le changelog.

[Déployer sur Northflank](./northflank.md)

---