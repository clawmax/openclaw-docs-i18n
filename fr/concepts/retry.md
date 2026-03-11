

  Messages et livraison

  
# Politique de nouvelle tentative

## Objectifs

-   Nouvelle tentative par requête HTTP, et non par flux multi-étapes.
-   Préserver l'ordre en réessayant uniquement l'étape en cours.
-   Éviter la duplication des opérations non idempotentes.

## Valeurs par défaut

-   Tentatives : 3
-   Délai maximum : 30000 ms
-   Jitter : 0,1 (10 pour cent)
-   Valeurs par défaut des fournisseurs :
    -   Délai minimum Telegram : 400 ms
    -   Délai minimum Discord : 500 ms

## Comportement

### Discord

-   Nouvelles tentatives uniquement sur les erreurs de limite de débit (HTTP 429).
-   Utilise `retry_after` de Discord lorsqu'il est disponible, sinon backoff exponentiel.

### Telegram

-   Nouvelles tentatives sur les erreurs transitoires (429, timeout, connexion/réinitialisation/fermeture, temporairement indisponible).
-   Utilise `retry_after` lorsqu'il est disponible, sinon backoff exponentiel.
-   Les erreurs d'analyse Markdown ne sont pas réessayées ; elles basculent en texte brut.

## Configuration

Définissez la politique de nouvelle tentative par fournisseur dans `~/.openclaw/openclaw.json` :

```json
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

## Notes

-   Les nouvelles tentatives s'appliquent par requête (envoi de message, téléchargement de média, réaction, sondage, autocollant).
-   Les flux composites ne réessaient pas les étapes terminées.

[Diffusion en continu et découpage](./streaming.md)[File d'attente des commandes](./queue.md)

---