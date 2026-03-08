title: "Modèles et affirmations de vérification formelle de sécurité OpenClaw"
description: "Explorez les modèles de sécurité formels et les affirmations vérifiables d'OpenClaw pour l'autorisation, l'isolation des sessions et le contrôle des outils. Apprenez à reproduire les résultats et à comprendre les garanties de sécurité."
keywords: ["vérification formelle", "modèles de sécurité", "tla+", "sécurité openclaw", "vérification de modèle", "isolation de session", "autorisation", "exposition de la passerelle"]
---

  Sécurité

  
# Vérification Formelle (Modèles de Sécurité)

Cette page suit les **modèles de sécurité formels** d'OpenClaw (TLA+/TLC aujourd'hui ; plus si nécessaire).

> Note : certains liens plus anciens peuvent faire référence au nom précédent du projet.

**Objectif (étoile du nord) :** fournir un argument vérifié par machine qu'OpenClaw applique sa politique de sécurité prévue (autorisation, isolation des sessions, contrôle des outils et sécurité contre les mauvaises configurations), sous des hypothèses explicites. **Ce que c'est (aujourd'hui) :** une **suite de régression de sécurité** exécutable et pilotée par un attaquant :

-   Chaque affirmation a une vérification de modèle exécutable sur un espace d'états fini.
-   De nombreuses affirmations ont un **modèle négatif** associé qui produit une trace de contre-exemple pour une classe de bug réaliste.

**Ce que ce n'est pas (encore) :** une preuve qu'« OpenClaw est sécurisé sous tous les aspects » ou que l'implémentation TypeScript complète est correcte.

## Où se trouvent les modèles

Les modèles sont maintenus dans un dépôt séparé : [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

## Mises en garde importantes

-   Ce sont des **modèles**, pas l'implémentation TypeScript complète. Un écart entre le modèle et le code est possible.
-   Les résultats sont limités par l'espace d'états exploré par TLC ; un résultat « vert » n'implique pas la sécurité au-delà des hypothèses et limites modélisées.
-   Certaines affirmations reposent sur des hypothèses environnementales explicites (par exemple, un déploiement correct, des entrées de configuration correctes).

## Reproduire les résultats

Aujourd'hui, les résultats sont reproduits en clonant le dépôt des modèles localement et en exécutant TLC (voir ci-dessous). Une future itération pourrait offrir :

-   Des modèles exécutés en CI avec des artefacts publics (traces de contre-exemple, journaux d'exécution)
-   Un flux de travail hébergé « exécuter ce modèle » pour des vérifications petites et bornées

Pour commencer :

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11+ requis (TLC s'exécute sur la JVM).
# Le dépôt inclut une version figée de `tla2tools.jar` (outils TLA+) et fournit `bin/tlc` + des cibles Make.

make <target>
```

### Exposition de la passerelle et mauvaise configuration de passerelle ouverte

**Affirmation :** une liaison au-delà de localhost sans authentification peut rendre un compromis à distance possible / augmente l'exposition ; un jeton/mot de passe bloque les attaquants non authentifiés (selon les hypothèses du modèle).

-   Exécutions vertes :
    -   `make gateway-exposure-v2`
    -   `make gateway-exposure-v2-protected`
-   Rouge (attendu) :
    -   `make gateway-exposure-v2-negative`

Voir aussi : `docs/gateway-exposure-matrix.md` dans le dépôt des modèles.

### Pipeline nodes.run (capacité à risque le plus élevé)

**Affirmation :** `nodes.run` nécessite (a) une liste d'autorisation de commandes de nœud plus des commandes déclarées et (b) une approbation en direct lorsqu'elle est configurée ; les approbations sont tokenisées pour empêcher la relecture (dans le modèle).

-   Exécutions vertes :
    -   `make nodes-pipeline`
    -   `make approvals-token`
-   Rouge (attendu) :
    -   `make nodes-pipeline-negative`
    -   `make approvals-token-negative`

### Magasin d'appariement (contrôle DM)

**Affirmation :** les demandes d'appariement respectent le TTL et les limites de demandes en attente.

-   Exécutions vertes :
    -   `make pairing`
    -   `make pairing-cap`
-   Rouge (attendu) :
    -   `make pairing-negative`
    -   `make pairing-cap-negative`

### Contrôle d'ingress (mentions + contournement de commande de contrôle)

**Affirmation :** dans les contextes de groupe nécessitant une mention, une « commande de contrôle » non autorisée ne peut pas contourner le contrôle par mention.

-   Vert :
    -   `make ingress-gating`
-   Rouge (attendu) :
    -   `make ingress-gating-negative`

### Isolation du routage/clé de session

**Affirmation :** les messages privés de pairs distincts ne fusionnent pas dans la même session à moins d'être explicitement liés/configurés.

-   Vert :
    -   `make routing-isolation`
-   Rouge (attendu) :
    -   `make routing-isolation-negative`

## v1++ : modèles bornés supplémentaires (concurrence, nouvelles tentatives, exactitude des traces)

Il s'agit de modèles de suivi qui renforcent la fidélité autour des modes de défaillance du monde réel (mises à jour non atomiques, nouvelles tentatives et diffusion de messages).

### Concurrence / idempotence du magasin d'appariement

**Affirmation :** un magasin d'appariement doit appliquer `MaxPending` et l'idempotence même sous des entrelacements (c'est-à-dire que « vérifier puis écrire » doit être atomique / verrouillé ; un rafraîchissement ne doit pas créer de doublons). Ce que cela signifie :

-   Sous des requêtes concurrentes, vous ne pouvez pas dépasser `MaxPending` pour un canal.
-   Des requêtes/rafraîchissements répétés pour le même `(canal, expéditeur)` ne doivent pas créer de lignes en attente en double.
-   Exécutions vertes :
    -   `make pairing-race` (vérification de limite atomique/verrouillée)
    -   `make pairing-idempotency`
    -   `make pairing-refresh`
    -   `make pairing-refresh-race`
-   Rouge (attendu) :
    -   `make pairing-race-negative` (course à la limite non atomique début/validation)
    -   `make pairing-idempotency-negative`
    -   `make pairing-refresh-negative`
    -   `make pairing-refresh-race-negative`

### Corrélation de trace d'ingress / idempotence

**Affirmation :** l'ingestion doit préserver la corrélation des traces à travers la diffusion et être idempotente sous les nouvelles tentatives du fournisseur. Ce que cela signifie :

-   Lorsqu'un événement externe devient plusieurs messages internes, chaque partie conserve la même identité de trace/événement.
-   Les nouvelles tentatives n'entraînent pas de double traitement.
-   Si les identifiants d'événement du fournisseur sont manquants, la déduplication utilise une clé de secours sûre (par exemple, l'ID de trace) pour éviter de supprimer des événements distincts.
-   Vert :
    -   `make ingress-trace`
    -   `make ingress-trace2`
    -   `make ingress-idempotency`
    -   `make ingress-dedupe-fallback`
-   Rouge (attendu) :
    -   `make ingress-trace-negative`
    -   `make ingress-trace2-negative`
    -   `make ingress-idempotency-negative`
    -   `make ingress-dedupe-fallback-negative`

### Précedence dmScope du routage + identityLinks

**Affirmation :** le routage doit maintenir les sessions de messages privés isolées par défaut, et ne fusionner les sessions que lorsqu'elles sont explicitement configurées (précédence des canaux + liens d'identité). Ce que cela signifie :

-   Les remplacements dmScope spécifiques au canal doivent l'emporter sur les paramètres globaux par défaut.
-   Les identityLinks ne doivent fusionner qu'au sein de groupes liés explicitement, pas entre des pairs non liés.
-   Vert :
    -   `make routing-precedence`
    -   `make routing-identitylinks`
-   Rouge (attendu) :
    -   `make routing-precedence-negative`
    -   `make routing-identitylinks-negative`

[Tailscale](../gateway/tailscale.md)[README](./README.md)