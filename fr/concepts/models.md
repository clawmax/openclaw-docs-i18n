

  Concepts des modèles

  
# CLI Modèles

Voir [/concepts/model-failover](./model-failover.md) pour la rotation des profils d'authentification, les périodes de refroidissement et leur interaction avec les secours. Aperçu rapide des fournisseurs + exemples : [/concepts/model-providers](./model-providers.md).

## Fonctionnement de la sélection des modèles

OpenClaw sélectionne les modèles dans cet ordre :

1.  Modèle **principal** (`agents.defaults.model.primary` ou `agents.defaults.model`).
2.  **Secours** dans `agents.defaults.model.fallbacks` (dans l'ordre).
3.  Le **basculement d'authentification du fournisseur** se produit à l'intérieur d'un fournisseur avant de passer au modèle suivant.

Liés :

-   `agents.defaults.models` est la liste autorisée/le catalogue des modèles qu'OpenClaw peut utiliser (plus les alias).
-   `agents.defaults.imageModel` est utilisé **uniquement lorsque** le modèle principal ne peut pas accepter les images.
-   Les paramètres par défaut par agent peuvent remplacer `agents.defaults.model` via `agents.list[].model` plus les liaisons (voir [/concepts/multi-agent](./multi-agent.md)).

## Politique rapide des modèles

-   Définissez votre modèle principal sur le modèle de dernière génération le plus puissant disponible pour vous.
-   Utilisez les secours pour les tâches sensibles au coût/à la latence et pour les discussions à enjeux moindres.
-   Pour les agents avec outils ou pour les entrées non fiables, évitez les niveaux de modèles plus anciens/plus faibles.

## Assistant de configuration (recommandé)

Si vous ne souhaitez pas modifier manuellement la configuration, exécutez l'assistant d'intégration :

```bash
openclaw onboard
```

Il peut configurer le modèle et l'authentification pour les fournisseurs courants, y compris l'**abonnement OpenAI Code (Codex)** (OAuth) et **Anthropic** (clé API ou `claude setup-token`).

## Clés de configuration (aperçu)

-   `agents.defaults.model.primary` et `agents.defaults.model.fallbacks`
-   `agents.defaults.imageModel.primary` et `agents.defaults.imageModel.fallbacks`
-   `agents.defaults.models` (liste autorisée + alias + paramètres du fournisseur)
-   `models.providers` (fournisseurs personnalisés écrits dans `models.json`)

Les références de modèles sont normalisées en minuscules. Les alias de fournisseurs comme `z.ai/*` se normalisent en `zai/*`. Des exemples de configuration de fournisseurs (y compris OpenCode Zen) se trouvent dans [/gateway/configuration](../gateway/configuration.md#opencode-zen-multi-model-proxy).

## "Le modèle n'est pas autorisé" (et pourquoi les réponses s'arrêtent)

Si `agents.defaults.models` est défini, il devient la **liste autorisée** pour `/model` et pour les remplacements de session. Lorsqu'un utilisateur sélectionne un modèle qui n'est pas dans cette liste autorisée, OpenClaw renvoie :

```bash
Model "provider/model" is not allowed. Use /model to list available models.
```

Cela se produit **avant** qu'une réponse normale ne soit générée, donc le message peut donner l'impression qu'il "n'a pas répondu". La solution est soit de :

-   Ajouter le modèle à `agents.defaults.models`, ou
-   Effacer la liste autorisée (supprimer `agents.defaults.models`), ou
-   Choisir un modèle depuis `/model list`.

Exemple de configuration de liste autorisée :

```json
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-6": { alias: "Opus" },
    },
  },
}
```

## Changer de modèle dans le chat (/model)

Vous pouvez changer de modèle pour la session en cours sans redémarrer :

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Notes :

-   `/model` (et `/model list`) est un sélecteur compact et numéroté (famille de modèles + fournisseurs disponibles).
-   Sur Discord, `/model` et `/models` ouvrent un sélecteur interactif avec des menus déroulants pour le fournisseur et le modèle, plus une étape de soumission.
-   `/model <#>` sélectionne depuis ce sélecteur.
-   `/model status` est la vue détaillée (candidats d'authentification et, lorsqu'il est configuré, l'URL de base du point de terminaison du fournisseur `baseUrl` + mode `api`).
-   Les références de modèles sont analysées en divisant sur le **premier** `/`. Utilisez `provider/model` en tapant `/model `.
-   Si l'ID du modèle contient lui-même un `/` (style OpenRouter), vous devez inclure le préfixe du fournisseur (exemple : `/model openrouter/moonshotai/kimi-k2`).
-   Si vous omettez le fournisseur, OpenClaw traite l'entrée comme un alias ou un modèle pour le **fournisseur par défaut** (fonctionne uniquement lorsqu'il n'y a pas de `/` dans l'ID du modèle).

Comportement/config complet de la commande : [Commandes slash](../tools/slash-commands.md).

## Commandes CLI

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models` (sans sous-commande) est un raccourci pour `models status`.

### models list

Affiche les modèles configurés par défaut. Options utiles :

-   `--all` : catalogue complet
-   `--local` : fournisseurs locaux uniquement
-   `--provider ` : filtrer par fournisseur
-   `--plain` : un modèle par ligne
-   `--json` : sortie lisible par machine

### models status

Affiche le modèle principal résolu, les secours, le modèle d'image et un aperçu de l'authentification des fournisseurs configurés. Il indique également l'état d'expiration OAuth pour les profils trouvés dans le magasin d'authentification (avertit par défaut dans les 24h). `--plain` n'imprime que le modèle principal résolu. L'état OAuth est toujours affiché (et inclus dans la sortie `--json`). Si un fournisseur configuré n'a pas d'identifiants, `models status` imprime une section **Authentification manquante**. Le JSON inclut `auth.oauth` (fenêtre d'avertissement + profils) et `auth.providers` (authentification effective par fournisseur). Utilisez `--check` pour l'automatisation (sortie `1` lorsque manquant/expiré, `2` lorsque sur le point d'expirer). Le choix de l'authentification dépend du fournisseur/compte. Pour les hôtes de passerelle toujours actifs, les clés API sont généralement les plus prévisibles ; les flux de jetons d'abonnement sont également pris en charge. Exemple (Anthropic setup-token) :

```bash
claude setup-token
openclaw models status
```

## Analyse (modèles gratuits OpenRouter)

`openclaw models scan` inspecte le **catalogue des modèles gratuits** d'OpenRouter et peut optionnellement sonder les modèles pour la prise en charge des outils et des images. Options clés :

-   `--no-probe` : ignorer les sondes en direct (métadonnées uniquement)
-   `--min-params ` : taille minimale des paramètres (milliards)
-   `--max-age-days ` : ignorer les modèles plus anciens
-   `--provider ` : filtre par préfixe de fournisseur
-   `--max-candidates ` : taille de la liste de secours
-   `--set-default` : définir `agents.defaults.model.primary` sur la première sélection
-   `--set-image` : définir `agents.defaults.imageModel.primary` sur la première sélection d'image

Le sondage nécessite une clé API OpenRouter (depuis les profils d'authentification ou `OPENROUTER_API_KEY`). Sans clé, utilisez `--no-probe` pour lister uniquement les candidats. Les résultats de l'analyse sont classés par :

1.  Prise en charge des images
2.  Latence des outils
3.  Taille du contexte
4.  Nombre de paramètres

Entrée

-   Liste OpenRouter `/models` (filtre `:free`)
-   Nécessite une clé API OpenRouter depuis les profils d'authentification ou `OPENROUTER_API_KEY` (voir [/environment](../help/environment.md))
-   Filtres optionnels : `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
-   Contrôles de sonde : `--timeout`, `--concurrency`

Lorsqu'il est exécuté dans un TTY, vous pouvez sélectionner les secours de manière interactive. En mode non interactif, passez `--yes` pour accepter les valeurs par défaut.

## Registre des modèles (models.json)

Les fournisseurs personnalisés dans `models.providers` sont écrits dans `models.json` sous le répertoire de l'agent (par défaut `~/.openclaw/agents//models.json`). Ce fichier est fusionné par défaut sauf si `models.mode` est défini sur `replace`. Priorité du mode de fusion pour les ID de fournisseurs correspondants :

-   Une `baseUrl` non vide déjà présente dans le `models.json` de l'agent l'emporte.
-   Une `apiKey` non vide dans le `models.json` de l'agent l'emporte uniquement lorsque ce fournisseur n'est pas géré par SecretRef dans le contexte de configuration/profil d'authentification actuel.
-   Les valeurs `apiKey` des fournisseurs gérés par SecretRef sont rafraîchies à partir des marqueurs source (`ENV_VAR_NAME` pour les références d'environnement, `secretref-managed` pour les références de fichier/exécution) au lieu de persister les secrets résolus.
-   Les `apiKey`/`baseUrl` vides ou manquants dans l'agent reviennent à la configuration `models.providers`.
-   Les autres champs du fournisseur sont rafraîchis à partir de la configuration et des données normalisées du catalogue.

Cette persistance basée sur des marqueurs s'applique chaque fois qu'OpenClaw régénère `models.json`, y compris les chemins pilotés par des commandes comme `openclaw agent`.

[Démarrage rapide des fournisseurs de modèles](../providers/models.md)[Fournisseurs de modèles](./model-providers.md)