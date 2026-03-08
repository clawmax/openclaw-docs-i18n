title: "Configuration des fuseaux horaires d'OpenClaw pour les horodatages et les invites système"
description: "Apprenez comment OpenClaw standardise les horodatages, configure les fuseaux horaires des enveloppes de messages et définit le fuseau horaire de l'utilisateur pour les invites système. Contrôlez l'affichage du temps pour les modèles d'IA."
keywords: ["fuseau horaire openclaw", "standardisation des horodatages", "enveloppe de message", "fuseau horaire iana", "fuseau horaire utilisateur", "invite système", "configuration des horodatages", "normalisation utc"]
---

  Détails internes

  
# Fuseaux horaires

OpenClaw standardise les horodatages pour que le modèle voie une **seule heure de référence**.

## Enveloppes de messages (locale par défaut)

Les messages entrants sont encapsulés dans une enveloppe comme :

```
[Provider ... 2026-01-05 16:26 PST] texte du message
```

L'horodatage dans l'enveloppe est **local à l'hôte par défaut**, avec une précision à la minute. Vous pouvez le remplacer avec :

```json
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | fuseau horaire IANA
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on", // "on" | "off"
    },
  },
}
```

-   `envelopeTimezone: "utc"` utilise UTC.
-   `envelopeTimezone: "user"` utilise `agents.defaults.userTimezone` (reste sur le fuseau horaire de l'hôte par défaut).
-   Utilisez un fuseau horaire IANA explicite (par exemple, `"Europe/Vienna"`) pour un décalage fixe.
-   `envelopeTimestamp: "off"` supprime les horodatages absolus des en-têtes d'enveloppe.
-   `envelopeElapsed: "off"` supprime les suffixes de temps écoulé (le style `+2m`).

### Exemples

**Local (par défaut) :**

```
[Signal Alice +1555 2026-01-18 00:19 PST] bonjour
```

**Fuseau horaire fixe :**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] bonjour
```

**Temps écoulé :**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] suivi
```

## Charges utiles des outils (données brutes du fournisseur + champs normalisés)

Les appels d'outils (`channels.discord.readMessages`, `channels.slack.readMessages`, etc.) renvoient les **horodatages bruts du fournisseur**. Nous ajoutons également des champs normalisés pour la cohérence :

-   `timestampMs` (millisecondes d'époque UTC)
-   `timestampUtc` (chaîne ISO 8601 UTC)

Les champs bruts du fournisseur sont conservés.

## Fuseau horaire de l'utilisateur pour l'invite système

Définissez `agents.defaults.userTimezone` pour indiquer au modèle le fuseau horaire local de l'utilisateur. S'il n'est pas défini, OpenClaw résout le **fuseau horaire de l'hôte au moment de l'exécution** (aucune écriture de configuration).

```json
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

L'invite système inclut :

-   Une section `Date et heure actuelles` avec l'heure locale et le fuseau horaire
-   `Format de l'heure : 12 heures` ou `24 heures`

Vous pouvez contrôler le format de l'invite avec `agents.defaults.timeFormat` (`auto` | `12` | `24`). Voir [Date & Heure](../date-time.md) pour le comportement complet et des exemples.

[Suivi de l'utilisation](./usage-tracking.md)[Crédits](../reference/credits.md)