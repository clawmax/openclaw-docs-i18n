

  Configuration et opérations

  
# Authentification

OpenClaw prend en charge OAuth et les clés API pour les fournisseurs de modèles. Pour les hôtes de passerelle toujours actifs, les clés API sont généralement l'option la plus prévisible. Les flux d'abonnement/OAuth sont également pris en charge lorsqu'ils correspondent au modèle de compte de votre fournisseur. Voir [/concepts/oauth](../concepts/oauth.md) pour le flux OAuth complet et la disposition du stockage. Pour l'authentification basée sur SecretRef (fournisseurs `env`/`file`/`exec`), voir [Gestion des secrets](./secrets.md). Pour les règles d'éligibilité/code de raison des identifiants utilisées par `models status --probe`, voir [Sémantique des identifiants d'authentification](../auth-credential-semantics.md).

## Configuration recommandée (clé API, tout fournisseur)

Si vous exécutez une passerelle de longue durée, commencez par une clé API pour votre fournisseur choisi. Pour Anthropic spécifiquement, l'authentification par clé API est la voie sûre et est recommandée par rapport à l'authentification par jeton de configuration d'abonnement.

1.  Créez une clé API dans la console de votre fournisseur.
2.  Placez-la sur **l'hôte de la passerelle** (la machine exécutant `openclaw gateway`).

```bash
export <PROVIDER>_API_KEY="..."
openclaw models status
```

3.  Si la Passerelle s'exécute sous systemd/launchd, préférez placer la clé dans `~/.openclaw/.env` pour que le démon puisse la lire :

```bash
cat >> ~/.openclaw/.env <<'EOF'
<PROVIDER>_API_KEY=...
EOF
```

Puis redémarrez le démon (ou redémarrez votre processus Gateway) et vérifiez à nouveau :

```bash
openclaw models status
openclaw doctor
```

Si vous préférez ne pas gérer vous-même les variables d'environnement, l'assistant d'intégration peut stocker les clés API pour une utilisation par le démon : `openclaw onboard`. Voir [Aide](../help.md) pour les détails sur l'héritage des variables d'environnement (`env.shellEnv`, `~/.openclaw/.env`, systemd/launchd).

## Anthropic : jeton de configuration (authentification par abonnement)

Si vous utilisez un abonnement Claude, le flux de jeton de configuration est pris en charge. Exécutez-le sur **l'hôte de la passerelle** :

```bash
claude setup-token
```

Puis collez-le dans OpenClaw :

```bash
openclaw models auth setup-token --provider anthropic
```

Si le jeton a été créé sur une autre machine, collez-le manuellement :

```bash
openclaw models auth paste-token --provider anthropic
```

Si vous voyez une erreur Anthropic comme :

```bash
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

…utilisez plutôt une clé API Anthropic.

> **⚠️** La prise en charge du jeton de configuration Anthropic est uniquement une compatibilité technique. Anthropic a bloqué certaines utilisations par abonnement en dehors de Claude Code par le passé. Utilisez-le uniquement si vous décidez que le risque politique est acceptable, et vérifiez vous-même les conditions actuelles d'Anthropic.

Saisie manuelle du jeton (tout fournisseur ; écrit `auth-profiles.json` + met à jour la configuration) :

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

Les références de profil d'authentification sont également prises en charge pour les identifiants statiques :

-   Les identifiants `api_key` peuvent utiliser `keyRef: { source, provider, id }`
-   Les identifiants `token` peuvent utiliser `tokenRef: { source, provider, id }`

Vérification adaptée à l'automatisation (sortie `1` en cas d'expiration/absence, `2` en cas d'expiration imminente) :

```bash
openclaw models status --check
```

Les scripts opérationnels facultatifs (systemd/Termux) sont documentés ici : [/automation/auth-monitoring](../automation/auth-monitoring.md)

> `claude setup-token` nécessite un TTY interactif.

## Vérification de l'état d'authentification des modèles

```bash
openclaw models status
openclaw doctor
```

## Comportement de rotation des clés API (passerelle)

Certains fournisseurs prennent en charge la nouvelle tentative d'une requête avec des clés alternatives lorsqu'un appel API atteint une limite de débit du fournisseur.

-   Ordre de priorité :
    -   `OPENCLAW_LIVE__KEY` (remplacement unique)
    -   `_API_KEYS`
    -   `_API_KEY`
    -   `_API_KEY_*`
-   Les fournisseurs Google incluent également `GOOGLE_API_KEY` comme repli supplémentaire.
-   La même liste de clés est dédupliquée avant utilisation.
-   OpenClaw réessaie avec la clé suivante uniquement pour les erreurs de limite de débit (par exemple `429`, `rate_limit`, `quota`, `resource exhausted`).
-   Les erreurs non liées à la limite de débit ne sont pas réessayées avec des clés alternatives.
-   Si toutes les clés échouent, l'erreur finale de la dernière tentative est renvoyée.

## Contrôle de l'identifiant utilisé

### Par session (commande de chat)

Utilisez `/model <alias-or-id>@` pour épingler un identifiant de fournisseur spécifique pour la session en cours (exemples d'identifiants de profil : `anthropic:default`, `anthropic:work`). Utilisez `/model` (ou `/model list`) pour un sélecteur compact ; utilisez `/model status` pour la vue complète (candidats + prochain profil d'authentification, ainsi que les détails du point de terminaison du fournisseur lorsqu'ils sont configurés).

### Par agent (remplacement CLI)

Définissez un ordre de remplacement explicite des profils d'authentification pour un agent (stocké dans le `auth-profiles.json` de cet agent) :

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

Utilisez `--agent ` pour cibler un agent spécifique ; omettez-le pour utiliser l'agent par défaut configuré.

## Dépannage

### "Aucun identifiant trouvé"

Si le profil de jeton Anthropic est manquant, exécutez `claude setup-token` sur **l'hôte de la passerelle**, puis vérifiez à nouveau :

```bash
openclaw models status
```

### Jeton en expiration/expiré

Exécutez `openclaw models status` pour confirmer quel profil est en train d'expirer. Si le profil est manquant, réexécutez `claude setup-token` et collez à nouveau le jeton.

## Prérequis

-   Compte d'abonnement Anthropic (pour `claude setup-token`)
-   CLI Claude Code installé (commande `claude` disponible)

[Exemples de configuration](./configuration-examples.md)[Sémantique des identifiants d'authentification](../auth-credential-semantics.md)