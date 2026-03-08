

  Expériences

  
# Exploration de Configuration de Modèles

Ce document capture des **idées** pour la future configuration des modèles. Ce n'est pas un cahier des charges définitif. Pour le comportement actuel, consultez :

-   [Modèles](../../concepts/models.md)
-   [Basculement de modèle](../../concepts/model-failover.md)
-   [OAuth + profils](../../concepts/oauth.md)

## Motivation

Les opérateurs souhaitent :

-   Plusieurs profils d'authentification par fournisseur (personnel vs professionnel).
-   Une sélection simple `/model` avec des secours prévisibles.
-   Une séparation claire entre les modèles de texte et les modèles compatibles avec les images.

## Direction possible (vue d'ensemble)

-   Garder la sélection de modèle simple : `fournisseur/modèle` avec des alias optionnels.
-   Permettre aux fournisseurs d'avoir plusieurs profils d'authentification, avec un ordre explicite.
-   Utiliser une liste globale de secours pour que toutes les sessions basculent de manière cohérente.
-   Ne remplacer le routage des images que lorsqu'il est explicitement configuré.

## Questions ouvertes

-   La rotation des profils doit-elle être par fournisseur ou par modèle ?
-   Comment l'interface utilisateur doit-elle présenter la sélection de profil pour une session ?
-   Quel est le chemin de migration le plus sûr depuis les anciennes clés de configuration ?

[Recherche sur la Mémoire de l'Espace de Travail](../research/memory.md)

---