

  Commandes CLI

  
# CLI du Bac à Sable

Gérez les conteneurs de bac à sable basés sur Docker pour l'exécution isolée des agents.

## Vue d'ensemble

OpenClaw peut exécuter des agents dans des conteneurs Docker isolés pour la sécurité. Les commandes `sandbox` vous aident à gérer ces conteneurs, notamment après des mises à jour ou des changements de configuration.

## Commandes

### openclaw sandbox explain

Inspectez le mode/portée/accès au workspace **effectif** du bac à sable, la politique des outils du bac à sable et les portes élevées (avec les chemins de clés de configuration de correction).

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

### openclaw sandbox list

Listez tous les conteneurs de bac à sable avec leur statut et leur configuration.

```bash
openclaw sandbox list
openclaw sandbox list --browser  # Lister uniquement les conteneurs navigateur
openclaw sandbox list --json     # Sortie JSON
```

**La sortie inclut :**

-   Nom du conteneur et statut (en cours/arrêté)
-   Image Docker et correspondance avec la configuration
-   Âge (temps depuis la création)
-   Temps d'inactivité (temps depuis la dernière utilisation)
-   Session/agent associé

### openclaw sandbox recreate

Supprimez les conteneurs de bac à sable pour forcer leur recréation avec les images/configurations mises à jour.

```bash
openclaw sandbox recreate --all                # Recréer tous les conteneurs
openclaw sandbox recreate --session main       # Session spécifique
openclaw sandbox recreate --agent mybot        # Agent spécifique
openclaw sandbox recreate --browser            # Seulement les conteneurs navigateur
openclaw sandbox recreate --all --force        # Ignorer la confirmation
```

**Options :**

-   `--all` : Recréer tous les conteneurs de bac à sable
-   `--session <clé>` : Recréer le conteneur pour une session spécifique
-   `--agent ` : Recréer les conteneurs pour un agent spécifique
-   `--browser` : Recréer uniquement les conteneurs navigateur
-   `--force` : Ignorer l'invite de confirmation

**Important :** Les conteneurs sont automatiquement recréés lors de la prochaine utilisation de l'agent.

## Cas d'utilisation

### Après la mise à jour des images Docker

```bash
# Télécharger la nouvelle image
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# Mettre à jour la configuration pour utiliser la nouvelle image
# Éditer config: agents.defaults.sandbox.docker.image (ou agents.list[].sandbox.docker.image)

# Recréer les conteneurs
openclaw sandbox recreate --all
```

### Après la modification de la configuration du bac à sable

```bash
# Éditer config: agents.defaults.sandbox.* (ou agents.list[].sandbox.*)

# Recréer pour appliquer la nouvelle configuration
openclaw sandbox recreate --all
```

### Après la modification de setupCommand

```bash
openclaw sandbox recreate --all
# ou juste un agent :
openclaw sandbox recreate --agent family
```

### Pour un agent spécifique uniquement

```bash
# Mettre à jour uniquement les conteneurs d'un agent
openclaw sandbox recreate --agent alfred
```

## Pourquoi est-ce nécessaire ?

**Problème :** Lorsque vous mettez à jour les images Docker ou la configuration du bac à sable :

-   Les conteneurs existants continuent de fonctionner avec les anciens paramètres
-   Les conteneurs ne sont nettoyés qu'après 24h d'inactivité
-   Les agents régulièrement utilisés maintiennent les anciens conteneurs en fonctionnement indéfiniment

**Solution :** Utilisez `openclaw sandbox recreate` pour forcer la suppression des anciens conteneurs. Ils seront automatiquement recréés avec les paramètres actuels lors de la prochaine utilisation. Astuce : préférez `openclaw sandbox recreate` à la commande manuelle `docker rm`. Cela utilise la nomenclature des conteneurs de la Gateway et évite les incohérences lorsque la portée/les clés de session changent.

## Configuration

Les paramètres du bac à sable se trouvent dans `~/.openclaw/openclaw.json` sous `agents.defaults.sandbox` (les surcharges par agent vont dans `agents.list[].sandbox`) :

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all", // off, non-main, all
        "scope": "agent", // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-",
          // ... plus d'options Docker
        },
        "prune": {
          "idleHours": 24, // Nettoyage auto après 24h d'inactivité
          "maxAgeDays": 7, // Nettoyage auto après 7 jours
        },
      },
    },
  },
}
```

## Voir aussi

-   [Documentation du Bac à Sable](../gateway/sandboxing.md)
-   [Configuration des Agents](../concepts/agent-workspace.md)
-   [Commande Doctor](../gateway/doctor.md) - Vérifier la configuration du bac à sable

[reset](./reset.md)[secrets](./secrets.md)

---