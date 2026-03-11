

  Compétences

  
# Création de Compétences

OpenClaw est conçu pour être facilement extensible. Les "Compétences" sont le principal moyen d'ajouter de nouvelles capacités à votre assistant.

## Qu'est-ce qu'une Compétence ?

Une compétence est un répertoire contenant un fichier `SKILL.md` (qui fournit des instructions et des définitions d'outils au LLM) et, optionnellement, quelques scripts ou ressources.

## Étape par Étape : Votre Première Compétence

### 1\. Créer le Répertoire

Les compétences résident dans votre espace de travail, généralement `~/.openclaw/workspace/skills/`. Créez un nouveau dossier pour votre compétence :

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2\. Définir le SKILL.md

Créez un fichier `SKILL.md` dans ce répertoire. Ce fichier utilise un frontmatter YAML pour les métadonnées et du Markdown pour les instructions.

```
---
name: hello_world
description: Une compétence simple qui dit bonjour.
---

# Compétence Hello World

Lorsque l'utilisateur demande une salutation, utilisez l'outil `echo` pour dire "Hello from your custom skill!".
```

### 3\. Ajouter des Outils (Optionnel)

Vous pouvez définir des outils personnalisés dans le frontmatter ou indiquer à l'agent d'utiliser des outils système existants (comme `bash` ou `browser`).

### 4\. Actualiser OpenClaw

Demandez à votre agent d'“actualiser les compétences” ou redémarrez la passerelle. OpenClaw découvrira le nouveau répertoire et indexera le fichier `SKILL.md`.

## Bonnes Pratiques

-   **Soyez Concis** : Donnez des instructions au modèle sur *quoi* faire, pas sur comment être une IA.
-   **Sécurité d'abord** : Si votre compétence utilise `bash`, assurez-vous que les invites n'autorisent pas l'injection de commandes arbitraires à partir d'une entrée utilisateur non fiable.
-   **Testez Localement** : Utilisez `openclaw agent --message "use my new skill"` pour tester.

## Compétences Partagées

Vous pouvez également parcourir et contribuer à des compétences sur [ClawHub](https://clawhub.com).

[Bac à Sable et Outils Multi-Agents](./multi-agent-sandbox-tools.md)[Commandes Slash](./slash-commands.md)