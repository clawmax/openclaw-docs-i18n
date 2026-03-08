

  Référence technique

  
# Date et Heure

OpenClaw utilise par défaut **l'heure locale de l'hôte pour les horodatages de transport** et **uniquement le fuseau horaire de l'utilisateur dans l'invite système**. Les horodatages des fournisseurs sont préservés afin que les outils conservent leur sémantique native (l'heure actuelle est disponible via `session_status`).

## Enveloppes de messages (locale par défaut)

Les messages entrants sont encapsulés avec un horodatage (précision à la minute) :

```
[Provider ... 2026-01-05 16:26 PST] texte du message
```

Cet horodatage d'enveloppe est **local à l'hôte par défaut**, quel que soit le fuseau horaire du fournisseur. Vous pouvez modifier ce comportement :

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
-   `envelopeTimezone: "local"` utilise le fuseau horaire de l'hôte.
-   `envelopeTimezone: "user"` utilise `agents.defaults.userTimezone` (reste sur le fuseau horaire de l'hôte en cas d'absence).
-   Utilisez un fuseau horaire IANA explicite (par exemple, `"America/Chicago"`) pour une zone fixe.
-   `envelopeTimestamp: "off"` supprime les horodatages absolus des en-têtes d'enveloppe.
-   `envelopeElapsed: "off"` supprime les suffixes de temps écoulé (le style `+2m`).

### Exemples

**Local (par défaut) :**

```
[WhatsApp +1555 2026-01-18 00:19 PST] bonjour
```

**Fuseau horaire utilisateur :**

```
[WhatsApp +1555 2026-01-18 00:19 CST] bonjour
```

**Temps écoulé activé :**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] suivi
```

## Invite système : Date et Heure actuelles

Si le fuseau horaire de l'utilisateur est connu, l'invite système inclut une section dédiée **Date et Heure actuelles** avec **uniquement le fuseau horaire** (pas de format d'horloge/heure) pour maintenir la stabilité du cache des invites :

```bash
Fuseau horaire : America/Chicago
```

Lorsque l'agent a besoin de l'heure actuelle, utilisez l'outil `session_status` ; la carte d'état inclut une ligne d'horodatage.

## Lignes d'événements système (locales par défaut)

Les événements système mis en file d'attente et insérés dans le contexte de l'agent sont préfixés d'un horodatage utilisant la même sélection de fuseau horaire que les enveloppes de messages (par défaut : local à l'hôte).

```yaml
Système : [2026-01-12 12:19:17 PST] Modèle changé.
```

### Configurer le fuseau horaire utilisateur + format

```json
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto", // auto | 12 | 24
    },
  },
}
```

-   `userTimezone` définit le **fuseau horaire local de l'utilisateur** pour le contexte de l'invite.
-   `timeFormat` contrôle l'affichage **12h/24h** dans l'invite. `auto` suit les préférences du système d'exploitation.

## Détection du format d'heure (auto)

Lorsque `timeFormat: "auto"`, OpenClaw inspecte la préférence du système d'exploitation (macOS/Windows) et utilise par défaut le formatage de la locale. La valeur détectée est **mise en cache par processus** pour éviter des appels système répétés.

## Charges utiles des outils + connecteurs (heure brute du fournisseur + champs normalisés)

Les outils de canal renvoient **les horodatages natifs du fournisseur** et ajoutent des champs normalisés pour la cohérence :

-   `timestampMs` : millisecondes epoch (UTC)
-   `timestampUtc` : chaîne ISO 8601 UTC

Les champs bruts du fournisseur sont préservés pour ne rien perdre.

-   Slack : chaînes de type epoch provenant de l'API
-   Discord : horodatages ISO UTC
-   Telegram/WhatsApp : horodatages numériques/ISO spécifiques au fournisseur

Si vous avez besoin de l'heure locale, convertissez-la en aval en utilisant le fuseau horaire connu.

## Documentation associée

-   [Invite Système](./concepts/system-prompt.md)
-   [Fuseaux horaires](./concepts/timezone.md)
-   [Messages](./concepts/messages.md)

[Hygène des Transcripts](./reference/transcript-hygiene.md)[TypeBox](./concepts/typebox.md)