

  Configuration et opérations

  
# Gestion des secrets

OpenClaw prend en charge les SecretRefs additifs, ce qui permet de ne pas stocker les identifiants pris en charge en texte clair dans la configuration. Le texte clair fonctionne toujours. Les SecretRefs sont activés au cas par cas pour chaque identifiant.

## Objectifs et modèle d'exécution

Les secrets sont résolus dans un instantané en mémoire au moment de l'exécution.

-   La résolution est effectuée de manière anticipée lors de l'activation, et non de manière différée sur les chemins de requête.
-   Le démarrage échoue rapidement lorsqu'une SecretRef effectivement active ne peut pas être résolue.
-   Le rechargement utilise un échange atomique : succès complet, ou conservation du dernier instantané valide connu.
-   Les requêtes d'exécution lisent uniquement à partir de l'instantané actif en mémoire.

Cela permet de maintenir les pannes des fournisseurs de secrets en dehors des chemins de requête critiques.

## Filtrage des surfaces actives

Les SecretRefs ne sont validées que sur les surfaces effectivement actives.

-   Surfaces activées : les références non résolues bloquent le démarrage/le rechargement.
-   Surfaces inactives : les références non résolues ne bloquent pas le démarrage/le rechargement.
-   Les références inactives émettent des diagnostics non fatals avec le code `SECRETS_REF_IGNORED_INACTIVE_SURFACE`.

Exemples de surfaces inactives :

-   Entrées de canal/compte désactivées.
-   Identifiants de canal de haut niveau qu'aucun compte activé n'hérite.
-   Surfaces d'outil/fonctionnalité désactivées.
-   Clés spécifiques au fournisseur de recherche web qui ne sont pas sélectionnées par `tools.web.search.provider`. En mode auto (fournisseur non défini), les clés spécifiques au fournisseur sont également actives pour la détection automatique du fournisseur.
-   Les SecretRefs `gateway.remote.token` / `gateway.remote.password` sont actives (lorsque `gateway.remote.enabled` n'est pas `false`) si l'une de ces conditions est vraie :
    -   `gateway.mode=remote`
    -   `gateway.remote.url` est configuré
    -   `gateway.tailscale.mode` est `serve` ou `funnel` En mode local sans ces surfaces distantes :
    -   `gateway.remote.token` est active lorsque l'authentification par jeton peut l'emporter et qu'aucun jeton d'environnement/d'authentification n'est configuré.
    -   `gateway.remote.password` est active uniquement lorsque l'authentification par mot de passe peut l'emporter et qu'aucun mot de passe d'environnement/d'authentification n'est configuré.
-   La SecretRef `gateway.auth.token` est inactive pour la résolution de l'authentification au démarrage lorsque `OPENCLAW_GATEWAY_TOKEN` (ou `CLAWDBOT_GATEWAY_TOKEN`) est défini, car le jeton d'environnement l'emporte pour cette exécution.

## Diagnostics de surface d'authentification de la passerelle

Lorsqu'une SecretRef est configurée sur `gateway.auth.token`, `gateway.auth.password`, `gateway.remote.token` ou `gateway.remote.password`, le démarrage/le rechargement de la passerelle enregistre explicitement l'état de la surface :

-   `active` : la SecretRef fait partie de la surface d'authentification effective et doit être résolue.
-   `inactive` : la SecretRef est ignorée pour cette exécution car une autre surface d'authentification l'emporte, ou parce que l'authentification distante est désactivée/non active.

Ces entrées sont enregistrées avec `SECRETS_GATEWAY_AUTH_SURFACE` et incluent la raison utilisée par la politique des surfaces actives, afin que vous puissiez voir pourquoi un identifiant a été traité comme actif ou inactif.

## Pré-vérification de référence d'intégration

Lorsque l'intégration s'exécute en mode interactif et que vous choisissez le stockage SecretRef, OpenClaw exécute une validation préalable avant l'enregistrement :

-   Références d'environnement : valide le nom de la variable d'environnement et confirme qu'une valeur non vide est visible pendant l'intégration.
-   Références de fournisseur (`file` ou `exec`) : valide la sélection du fournisseur, résout l'`id` et vérifie le type de valeur résolue.
-   Chemin de réutilisation du démarrage rapide : lorsque `gateway.auth.token` est déjà une SecretRef, l'intégration la résout avant le sondage/l'amorçage du tableau de bord (pour les références `env`, `file` et `exec`) en utilisant la même porte d'échec rapide.

Si la validation échoue, l'intégration affiche l'erreur et vous permet de réessayer.

## Contrat SecretRef

Utilisez une seule forme d'objet partout :

```json
{ source: "env" | "file" | "exec", provider: "default", id: "..." }
```

### source: "env"

```json
{ source: "env", provider: "default", id: "OPENAI_API_KEY" }
```

Validation :

-   `provider` doit correspondre à `^[a-z][a-z0-9_-]{0,63}$`
-   `id` doit correspondre à `^[A-Z][A-Z0-9_]{0,127}$`

### source: "file"

```json
{ source: "file", provider: "filemain", id: "/providers/openai/apiKey" }
```

Validation :

-   `provider` doit correspondre à `^[a-z][a-z0-9_-]{0,63}$`
-   `id` doit être un pointeur JSON absolu (`/...`)
-   Échappement RFC6901 dans les segments : `~` => `~0`, `/` => `~1`

### source: "exec"

```json
{ source: "exec", provider: "vault", id: "providers/openai/apiKey" }
```

Validation :

-   `provider` doit correspondre à `^[a-z][a-z0-9_-]{0,63}$`
-   `id` doit correspondre à `^[A-Za-z0-9][A-Za-z0-9._:/-]{0,255}$`

## Configuration des fournisseurs

Définissez les fournisseurs sous `secrets.providers` :

```json
{
  secrets: {
    providers: {
      default: { source: "env" },
      filemain: {
        source: "file",
        path: "~/.openclaw/secrets.json",
        mode: "json", // ou "singleValue"
      },
      vault: {
        source: "exec",
        command: "/usr/local/bin/openclaw-vault-resolver",
        args: ["--profile", "prod"],
        passEnv: ["PATH", "VAULT_ADDR"],
        jsonOnly: true,
      },
    },
    defaults: {
      env: "default",
      file: "filemain",
      exec: "vault",
    },
    resolution: {
      maxProviderConcurrency: 4,
      maxRefsPerProvider: 512,
      maxBatchBytes: 262144,
    },
  },
}
```

### Fournisseur d'environnement

-   Liste d'autorisation facultative via `allowlist`.
-   Les valeurs d'environnement manquantes/vides échouent à la résolution.

### Fournisseur de fichier

-   Lit un fichier local depuis `path`.
-   `mode: "json"` attend une charge utile d'objet JSON et résout `id` comme pointeur.
-   `mode: "singleValue"` attend l'id de référence `"value"` et retourne le contenu du fichier.
-   Le chemin doit passer les vérifications de propriété/autorisation.
-   Note de sécurité Windows : si la vérification des ACL n'est pas disponible pour un chemin, la résolution échoue. Pour les chemins de confiance uniquement, définissez `allowInsecurePath: true` sur ce fournisseur pour contourner les vérifications de sécurité du chemin.

### Fournisseur d'exécution

-   Exécute le chemin binaire absolu configuré, pas d'interpréteur de commandes.
-   Par défaut, `command` doit pointer vers un fichier régulier (pas un lien symbolique).
-   Définissez `allowSymlinkCommand: true` pour autoriser les chemins de commande de lien symbolique (par exemple les shims Homebrew). OpenClaw valide le chemin cible résolu.
-   Associez `allowSymlinkCommand` avec `trustedDirs` pour les chemins des gestionnaires de paquets (par exemple `["/opt/homebrew"]`).
-   Prend en charge le délai d'attente, le délai d'attente sans sortie, les limites d'octets de sortie, la liste d'autorisation d'environnement et les répertoires de confiance.
-   Note de sécurité Windows : si la vérification des ACL n'est pas disponible pour le chemin de la commande, la résolution échoue. Pour les chemins de confiance uniquement, définissez `allowInsecurePath: true` sur ce fournisseur pour contourner les vérifications de sécurité du chemin.

Charge utile de requête (stdin) :

```json
{ "protocolVersion": 1, "provider": "vault", "ids": ["providers/openai/apiKey"] }
```

Charge utile de réponse (stdout) :

```json
{ "protocolVersion": 1, "values": { "providers/openai/apiKey": "<openai-api-key>" } } // pragma: allowlist secret
```

Erreurs par id facultatives :

```json
{
  "protocolVersion": 1,
  "values": {},
  "errors": { "providers/openai/apiKey": { "message": "not found" } }
}
```

## Exemples d'intégration d'exécution

### CLI 1Password

```json
{
  secrets: {
    providers: {
      onepassword_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/op",
        allowSymlinkCommand: true, // requis pour les binaires liés symboliquement par Homebrew
        trustedDirs: ["/opt/homebrew"],
        args: ["read", "op://Personal/OpenClaw QA API Key/password"],
        passEnv: ["HOME"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "onepassword_openai", id: "value" },
      },
    },
  },
}
```

### CLI HashiCorp Vault

```json
{
  secrets: {
    providers: {
      vault_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/vault",
        allowSymlinkCommand: true, // requis pour les binaires liés symboliquement par Homebrew
        trustedDirs: ["/opt/homebrew"],
        args: ["kv", "get", "-field=OPENAI_API_KEY", "secret/openclaw"],
        passEnv: ["VAULT_ADDR", "VAULT_TOKEN"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "vault_openai", id: "value" },
      },
    },
  },
}
```

### sops

```json
{
  secrets: {
    providers: {
      sops_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/sops",
        allowSymlinkCommand: true, // requis pour les binaires liés symboliquement par Homebrew
        trustedDirs: ["/opt/homebrew"],
        args: ["-d", "--extract", '["providers"]["openai"]["apiKey"]', "/path/to/secrets.enc.json"],
        passEnv: ["SOPS_AGE_KEY_FILE"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "sops_openai", id: "value" },
      },
    },
  },
}
```

## Surface d'identifiants prise en charge

Les identifiants pris en charge et non pris en charge canoniques sont listés dans :

-   [Surface d'identifiants SecretRef](../reference/secretref-credential-surface.md)

Les identifiants générés à l'exécution ou rotatifs et le matériel d'actualisation OAuth sont intentionnellement exclus de la résolution en lecture seule des SecretRefs.

## Comportement requis et précédence

-   Champ sans référence : inchangé.
-   Champ avec une référence : requis sur les surfaces actives pendant l'activation.
-   Si le texte clair et la référence sont tous deux présents, la référence prend la précédence sur les chemins de précédence pris en charge.

Signaux d'avertissement et d'audit :

-   `SECRETS_REF_OVERRIDES_PLAINTEXT` (avertissement d'exécution)
-   `REF_SHADOWED` (constat d'audit lorsque les identifiants de `auth-profiles.json` prennent la précédence sur les références de `openclaw.json`)

Comportement de compatibilité Google Chat :

-   `serviceAccountRef` prend la précédence sur le texte clair `serviceAccount`.
-   La valeur en texte clair est ignorée lorsqu'une référence sœur est définie.

## Déclencheurs d'activation

L'activation des secrets s'exécute sur :

-   Le démarrage (pré-vérification plus activation finale)
-   Le chemin d'application à chaud du rechargement de configuration
-   Le chemin de vérification de redémarrage du rechargement de configuration
-   Le rechargement manuel via `secrets.reload`

Contrat d'activation :

-   Le succès échange l'instantané de manière atomique.
-   L'échec au démarrage interrompt le démarrage de la passerelle.
-   L'échec du rechargement à l'exécution conserve le dernier instantané valide connu.

## Signaux de dégradation et de récupération

Lorsque l'activation au moment du rechargement échoue après un état sain, OpenClaw entre dans un état de secrets dégradé. Événement système et codes de journalisation ponctuels :

-   `SECRETS_RELOADER_DEGRADED`
-   `SECRETS_RELOADER_RECOVERED`

Comportement :

-   Dégradé : l'exécution conserve le dernier instantané valide connu.
-   Récupéré : émis une fois après la prochaine activation réussie.
-   Les échecs répétés alors qu'il est déjà dégradé enregistrent des avertissements mais ne génèrent pas de spam d'événements.
-   L'échec rapide au démarrage n'émet pas d'événements de dégradation car l'exécution n'est jamais devenue active.

## Résolution des chemins de commande

Les chemins de commande peuvent opter pour la résolution SecretRef prise en charge via le RPC d'instantané de la passerelle. Il existe deux comportements généraux :

-   Les chemins de commande stricts (par exemple les chemins de mémoire distante `openclaw memory` et `openclaw qr --remote`) lisent à partir de l'instantané actif et échouent rapidement lorsqu'une SecretRef requise n'est pas disponible.
-   Les chemins de commande en lecture seule (par exemple `openclaw status`, `openclaw status --all`, `openclaw channels status`, `openclaw channels resolve`, et les flux de réparation de médecin/configuration en lecture seule) préfèrent également l'instantané actif, mais se dégradent au lieu d'abandonner lorsqu'une SecretRef ciblée n'est pas disponible dans ce chemin de commande.

Comportement en lecture seule :

-   Lorsque la passerelle est en cours d'exécution, ces commandes lisent d'abord à partir de l'instantané actif.
-   Si la résolution de la passerelle est incomplète ou si la passerelle n'est pas disponible, elles tentent un repli local ciblé pour la surface de commande spécifique.
-   Si une SecretRef ciblée est toujours indisponible, la commande continue avec une sortie en lecture seule dégradée et des diagnostics explicites tels que "configuré mais indisponible dans ce chemin de commande".
-   Ce comportement dégradé est local à la commande uniquement. Il n'affaiblit pas le démarrage, le rechargement ou les chemins d'envoi/d'authentification de l'exécution.

Autres notes :

-   L'actualisation de l'instantané après la rotation des secrets backend est gérée par `openclaw secrets reload`.
-   Méthode RPC de la passerelle utilisée par ces chemins de commande : `secrets.resolve`.

## Flux de travail d'audit et de configuration

Flux opérateur par défaut :

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

### secrets audit

Les constatations incluent :

-   valeurs en texte clair au repos (`openclaw.json`, `auth-profiles.json`, `.env`, et les `agents/*/agent/models.json` générés)
-   résidus d'en-têtes de fournisseur sensibles en texte clair dans les entrées `models.json` générées
-   références non résolues
-   masquage de précédence (`auth-profiles.json` prenant la priorité sur les références de `openclaw.json`)
-   résidus hérités (`auth.json`, rappels OAuth)

Note sur les résidus d'en-tête :

-   La détection des en-têtes de fournisseur sensibles est basée sur des heuristiques de noms (noms d'en-têtes et fragments d'authentification/identifiants courants tels que `authorization`, `x-api-key`, `token`, `secret`, `password` et `credential`).

### secrets configure

Assistant interactif qui :

-   configure d'abord `secrets.providers` (`env`/`file`/`exec`, ajouter/modifier/supprimer)
-   vous permet de sélectionner les champs porteurs de secrets pris en charge dans `openclaw.json` plus `auth-profiles.json` pour une portée d'agent
-   peut créer un nouveau mappage `auth-profiles.json` directement dans le sélecteur cible
-   capture les détails de SecretRef (`source`, `provider`, `id`)
-   exécute une résolution préalable
-   peut appliquer immédiatement

Modes utiles :

-   `openclaw secrets configure --providers-only`
-   `openclaw secrets configure --skip-provider-setup`
-   `openclaw secrets configure --agent `

Application par défaut de `configure` :

-   nettoie les identifiants statiques correspondants de `auth-profiles.json` pour les fournisseurs ciblés
-   nettoie les entrées statiques héritées `api_key` de `auth.json`
-   nettoie les lignes de secrets connues correspondantes de `<config-dir>/.env`

### secrets apply

Applique un plan sauvegardé :

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
```

Pour les détails stricts du contrat cible/chemin et les règles de rejet exactes, voir :

-   [Contrat de plan d'application des secrets](./secrets-plan-contract.md)

## Politique de sécurité unidirectionnelle

OpenClaw n'écrit intentionnellement pas de sauvegardes de retour en arrière contenant des valeurs de secrets en texte clair historiques. Modèle de sécurité :

-   la pré-vérification doit réussir avant le mode écriture
-   l'activation d'exécution est validée avant la validation
-   l'application met à jour les fichiers en utilisant le remplacement atomique de fichiers et la restauration au mieux en cas d'échec

## Notes de compatibilité d'authentification héritée

Pour les identifiants statiques, l'exécution ne dépend plus du stockage d'authentification hérité en texte clair.

-   La source d'identifiants d'exécution est l'instantané en mémoire résolu.
-   Les entrées statiques héritées `api_key` sont nettoyées lorsqu'elles sont découvertes.
-   Le comportement de compatibilité lié à OAuth reste séparé.

## Note sur l'interface web

Certaines unions SecretInput sont plus faciles à configurer en mode éditeur brut qu'en mode formulaire.

## Documents connexes

-   Commandes CLI : [secrets](../cli/secrets.md)
-   Détails du contrat de plan : [Contrat de plan d'application des secrets](./secrets-plan-contract.md)
-   Surface d'identifiants : [Surface d'identifiants SecretRef](../reference/secretref-credential-surface.md)
-   Configuration de l'authentification : [Authentification](./authentication.md)
-   Posture de sécurité : [Sécurité](./security.md)
-   Précédence de l'environnement : [Variables d'environnement](../help/environment.md)

[Sémantique des identifiants d'authentification](../auth-credential-semantics.md)[Contrat de plan d'application des secrets](./secrets-plan-contract.md)