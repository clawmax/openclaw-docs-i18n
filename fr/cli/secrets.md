

  Commandes CLI

  
# secrets

Utilisez `openclaw secrets` pour gérer les SecretRefs et maintenir l'instantané d'exécution actif en bonne santé. Rôles des commandes :

-   `reload` : RPC de la passerelle (`secrets.reload`) qui réévalue les références et échange l'instantané d'exécution uniquement en cas de succès complet (aucune écriture de configuration).
-   `audit` : analyse en lecture seule des magasins de configuration/d'authentification/de modèles générés et des résidus hérités pour détecter le texte en clair, les références non résolues et les dérives de priorité.
-   `configure` : planificateur interactif pour la configuration des fournisseurs, le mappage des cibles et la prévérification (TTY requis).
-   `apply` : exécute un plan sauvegardé (`--dry-run` pour validation uniquement), puis nettoie les résidus de texte en clair ciblés.

Boucle opérateur recommandée :

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets audit --check
openclaw secrets reload
```

Note sur le code de sortie pour CI/portes :

-   `audit --check` retourne `1` en cas de découvertes.
-   les références non résolues retournent `2`.

Liens connexes :

-   Guide des secrets : [Gestion des secrets](../gateway/secrets.md)
-   Surface d'identification : [Surface d'identification SecretRef](../reference/secretref-credential-surface.md)
-   Guide de sécurité : [Sécurité](../gateway/security.md)

## Recharger l'instantané d'exécution

Réévalue les références de secrets et échange atomiquement l'instantané d'exécution.

```bash
openclaw secrets reload
openclaw secrets reload --json
```

Notes :

-   Utilise la méthode RPC de la passerelle `secrets.reload`.
-   Si l'évaluation échoue, la passerelle conserve le dernier instantané valide connu et retourne une erreur (pas d'activation partielle).
-   La réponse JSON inclut `warningCount`.

## Audit

Analyse l'état d'OpenClaw pour détecter :

-   le stockage de secrets en texte clair
-   les références non résolues
-   les dérives de priorité (les identifiants de `auth-profiles.json` éclipsant les références de `openclaw.json`)
-   les résidus générés `agents/*/agent/models.json` (valeurs `apiKey` des fournisseurs et en-têtes sensibles des fournisseurs)
-   les résidus hérités (entrées de l'ancien magasin d'authentification, rappels OAuth)

Note sur les résidus d'en-têtes :

-   La détection des en-têtes sensibles des fournisseurs est basée sur une heuristique de noms (noms d'en-têtes d'authentification/d'identification courants et fragments tels que `authorization`, `x-api-key`, `token`, `secret`, `password` et `credential`).

```bash
openclaw secrets audit
openclaw secrets audit --check
openclaw secrets audit --json
```

Comportement de sortie :

-   `--check` quitte avec un code non nul en cas de découvertes.
-   les références non résolues quittent avec un code non nul de priorité plus élevée.

Aperçu de la structure du rapport :

-   `status` : `clean | findings | unresolved`
-   `summary` : `plaintextCount`, `unresolvedRefCount`, `shadowedRefCount`, `legacyResidueCount`
-   codes de découverte :
    -   `PLAINTEXT_FOUND`
    -   `REF_UNRESOLVED`
    -   `REF_SHADOWED`
    -   `LEGACY_RESIDUE`

## Configurer (assistant interactif)

Construisez interactivement les modifications des fournisseurs et des SecretRefs, exécutez une prévérification et appliquez éventuellement :

```bash
openclaw secrets configure
openclaw secrets configure --plan-out /tmp/openclaw-secrets-plan.json
openclaw secrets configure --apply --yes
openclaw secrets configure --providers-only
openclaw secrets configure --skip-provider-setup
openclaw secrets configure --agent ops
openclaw secrets configure --json
```

Déroulement :

-   Configuration des fournisseurs d'abord (`add/edit/remove` pour les alias `secrets.providers`).
-   Mappage des identifiants ensuite (sélectionnez les champs et attribuez les références `{source, provider, id}`).
-   Pré-vérification et application optionnelle à la fin.

Drapeaux :

-   `--providers-only` : configure uniquement `secrets.providers`, ignore le mappage des identifiants.
-   `--skip-provider-setup` : ignore la configuration des fournisseurs et mappe les identifiants vers les fournisseurs existants.
-   `--agent ` : limite la découverte des cibles et les écritures `auth-profiles.json` à un seul magasin d'agent.

Notes :

-   Nécessite un TTY interactif.
-   Vous ne pouvez pas combiner `--providers-only` avec `--skip-provider-setup`.
-   `configure` cible les champs porteurs de secrets dans `openclaw.json` ainsi que `auth-profiles.json` pour la portée d'agent sélectionnée.
-   `configure` prend en charge la création de nouveaux mappages `auth-profiles.json` directement dans le flux du sélecteur.
-   Surface prise en charge canonique : [Surface d'identification SecretRef](../reference/secretref-credential-surface.md).
-   Il effectue une pré-vérification de résolution avant l'application.
-   Les plans générés activent par défaut les options de nettoyage (`scrubEnv`, `scrubAuthProfilesForProviderTargets`, `scrubLegacyAuthJson` tous activés).
-   Le chemin d'application est unidirectionnel pour les valeurs en texte clair nettoyées.
-   Sans `--apply`, la CLI demande toujours `Appliquer ce plan maintenant ?` après la pré-vérification.
-   Avec `--apply` (et sans `--yes`), la CLI demande une confirmation supplémentaire irréversible.

Note de sécurité sur le fournisseur exec :

-   Les installations Homebrew exposent souvent des binaires liés symboliquement sous `/opt/homebrew/bin/*`.
-   Définissez `allowSymlinkCommand: true` uniquement lorsque nécessaire pour les chemins de gestionnaires de paquets de confiance, et associez-le à `trustedDirs` (par exemple `["/opt/homebrew"]`).
-   Sous Windows, si la vérification ACL n'est pas disponible pour un chemin de fournisseur, OpenClaw échoue en mode fermé. Pour les chemins de confiance uniquement, définissez `allowInsecurePath: true` sur ce fournisseur pour contourner les vérifications de sécurité des chemins.

## Appliquer un plan sauvegardé

Appliquez ou pré-vérifiez un plan généré précédemment :

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --json
```

Détails du contrat de plan (chemins cibles autorisés, règles de validation et sémantique d'échec) :

-   [Contrat de plan d'application des secrets](../gateway/secrets-plan-contract.md)

Ce que `apply` peut mettre à jour :

-   `openclaw.json` (cibles SecretRef + ajouts/suppressions de fournisseurs)
-   `auth-profiles.json` (nettoyage des cibles des fournisseurs)
-   résidus de l'ancien `auth.json`
-   `~/.openclaw/.env` clés secrètes connues dont les valeurs ont été migrées

## Pourquoi pas de sauvegardes de restauration

`secrets apply` n'écrit intentionnellement pas de sauvegardes de restauration contenant les anciennes valeurs en texte clair. La sécurité provient d'une pré-vérification stricte + d'une application atomique avec une restauration en mémoire de meilleur effort en cas d'échec.

## Exemple

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

Si `audit --check` signale encore des découvertes de texte clair, mettez à jour les chemins cibles restants signalés et relancez l'audit.

[CLI Bac à sable](./sandbox.md)[sécurité](./security.md)