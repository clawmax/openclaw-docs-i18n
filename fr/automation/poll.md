

  Automatisation

  
# Sondages

## Canaux pris en charge

-   Telegram
-   WhatsApp (canal web)
-   Discord
-   MS Teams (Cartes adaptatives)

## CLI

```bash
# Telegram
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "On le livre ?" --poll-option "Oui" --poll-option "Non"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Choisissez une heure" --poll-option "10h" --poll-option "14h" \
  --poll-duration-seconds 300

# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Déjeuner aujourd'hui ?" --poll-option "Oui" --poll-option "Non" --poll-option "Peut-être"
openclaw message poll --target 123456789@g.us \
  --poll-question "Heure de la réunion ?" --poll-option "10h" --poll-option "14h" --poll-option "16h" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Collation ?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan ?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Déjeuner ?" --poll-option "Pizza" --poll-option "Sushi"
```

Options :

-   `--channel` : `whatsapp` (par défaut), `telegram`, `discord`, ou `msteams`
-   `--poll-multi` : permet de sélectionner plusieurs options
-   `--poll-duration-hours` : Discord uniquement (par défaut 24 si omis)
-   `--poll-duration-seconds` : Telegram uniquement (5-600 secondes)
-   `--poll-anonymous` / `--poll-public` : visibilité du sondage (Telegram uniquement)

## Gateway RPC

Méthode : `poll` Paramètres :

-   `to` (string, requis)
-   `question` (string, requis)
-   `options` (string\[\], requis)
-   `maxSelections` (number, optionnel)
-   `durationHours` (number, optionnel)
-   `durationSeconds` (number, optionnel, Telegram uniquement)
-   `isAnonymous` (boolean, optionnel, Telegram uniquement)
-   `channel` (string, optionnel, défaut : `whatsapp`)
-   `idempotencyKey` (string, requis)

## Différences entre canaux

-   Telegram : 2-10 options. Prend en charge les sujets de forum via `threadId` ou les cibles `:topic:`. Utilise `durationSeconds` au lieu de `durationHours`, limité à 5-600 secondes. Prend en charge les sondages anonymes et publics.
-   WhatsApp : 2-12 options, `maxSelections` doit être dans la limite du nombre d'options, ignore `durationHours`.
-   Discord : 2-10 options, `durationHours` limité à 1-768 heures (défaut 24). `maxSelections > 1` active la sélection multiple ; Discord ne prend pas en charge un nombre de sélections strict.
-   MS Teams : Sondages en Carte adaptative (gérés par OpenClaw). Pas d'API de sondage native ; `durationHours` est ignoré.

## Outil Agent (Message)

Utilisez l'outil `message` avec l'action `poll` (`to`, `pollQuestion`, `pollOption`, optionnel `pollMulti`, `pollDurationHours`, `channel`). Pour Telegram, l'outil accepte aussi `pollDurationSeconds`, `pollAnonymous`, et `pollPublic`. Utilisez `action: "poll"` pour la création de sondage. Les champs de sondage passés avec `action: "send"` sont rejetés. Note : Discord n'a pas de mode "choisir exactement N" ; `pollMulti` correspond à la sélection multiple. Les sondages Teams sont rendus sous forme de Cartes adaptatives et nécessitent que la passerelle reste en ligne pour enregistrer les votes dans `~/.openclaw/msteams-polls.json`.

[Gmail PubSub](./gmail-pubsub.md)[Surveillance de l'authentification](./auth-monitoring.md)

---