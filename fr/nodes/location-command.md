

  Médias et appareils

  
# Commande de localisation

## En bref

-   `location.get` est une commande de nœud (via `node.invoke`).
-   Désactivée par défaut.
-   Les paramètres utilisent un sélecteur : Désactivé / Pendant l'utilisation / Toujours.
-   Bascule séparée : Localisation précise.

## Pourquoi un sélecteur (et pas juste un interrupteur)

Les permissions du système d'exploitation sont à plusieurs niveaux. Nous pouvons exposer un sélecteur dans l'application, mais le SE décide toujours de l'autorisation réelle.

-   iOS/macOS : l'utilisateur peut choisir **Pendant l'utilisation** ou **Toujours** dans les invites système/Paramètres. L'application peut demander une mise à niveau, mais le SE peut exiger un passage par les Paramètres.
-   Android : la localisation en arrière-plan est une permission séparée ; sur Android 10+, elle nécessite souvent un flux via les Paramètres.
-   La localisation précise est une autorisation séparée (iOS 14+ « Précise », Android « fine » vs « coarse »).

Le sélecteur dans l'interface utilisateur pilote le mode que nous demandons ; l'autorisation réelle réside dans les paramètres du SE.

## Modèle de paramètres

Par appareil nœud :

-   `location.enabledMode`: `off | whileUsing | always`
-   `location.preciseEnabled`: booléen

Comportement de l'interface utilisateur :

-   Sélectionner `whileUsing` demande la permission au premier plan.
-   Sélectionner `always` s'assure d'abord d'avoir `whileUsing`, puis demande la permission en arrière-plan (ou envoie l'utilisateur aux Paramètres si nécessaire).
-   Si le SE refuse le niveau demandé, revenir au niveau le plus élevé accordé et afficher le statut.

## Mappage des permissions (node.permissions)

Optionnel. Le nœud macOS rapporte `location` via la carte des permissions ; iOS/Android peut l'omettre.

## Commande : location.get

Appelée via `node.invoke`. Paramètres (suggérés) :

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

Charge utile de réponse :

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

Erreurs (codes stables) :

-   `LOCATION_DISABLED` : le sélecteur est désactivé.
-   `LOCATION_PERMISSION_REQUIRED` : permission manquante pour le mode demandé.
-   `LOCATION_BACKGROUND_UNAVAILABLE` : l'application est en arrière-plan mais seul Pendant l'utilisation est autorisé.
-   `LOCATION_TIMEOUT` : pas de positionnement dans le temps imparti.
-   `LOCATION_UNAVAILABLE` : échec système / aucun fournisseur.

## Comportement en arrière-plan (futur)

Objectif : le modèle peut demander la localisation même lorsque le nœud est en arrière-plan, mais seulement lorsque :

-   L'utilisateur a sélectionné **Toujours**.
-   Le SE accorde la localisation en arrière-plan.
-   L'application est autorisée à s'exécuter en arrière-plan pour la localisation (mode arrière-plan iOS / service au premier plan Android ou autorisation spéciale).

Flux déclenché par push (futur) :

1.  La passerelle envoie un push au nœud (push silencieux ou données FCM).
2.  Le nœud se réveille brièvement et demande la localisation à l'appareil.
3.  Le nœud transmet la charge utile à la passerelle.

Notes :

-   iOS : Permission Toujours + mode de localisation en arrière-plan requis. Le push silencieux peut être limité ; prévoir des échecs intermittents.
-   Android : la localisation en arrière-plan peut nécessiter un service au premier plan ; sinon, prévoir un refus.

## Intégration modèle/outillage

-   Surface outil : l'outil `nodes` ajoute l'action `location_get` (nœud requis).
-   CLI : `openclaw nodes location get --node `.
-   Lignes directrices pour l'agent : n'appeler que lorsque l'utilisateur a activé la localisation et comprend la portée.

## Texte d'interface utilisateur (suggéré)

-   Désactivé : « Le partage de localisation est désactivé. »
-   Pendant l'utilisation : « Uniquement lorsque OpenClaw est ouvert. »
-   Toujours : « Autoriser la localisation en arrière-plan. Nécessite une permission système. »
-   Précise : « Utiliser la localisation GPS précise. Désactivez pour partager une localisation approximative. »

[Voice Wake](./voicewake.md)[Synthèse vocale](../tts.md)