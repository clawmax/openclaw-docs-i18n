

  Environnement et débogage

  
# Indicateurs de diagnostic

Les indicateurs de diagnostic vous permettent d'activer des journaux de débogage ciblés sans activer la journalisation verbeuse partout. Les indicateurs sont optionnels et n'ont aucun effet à moins qu'un sous-système ne les vérifie.

## Fonctionnement

-   Les indicateurs sont des chaînes de caractères (insensibles à la casse).
-   Vous pouvez activer les indicateurs dans la configuration ou via une surcharge d'environnement.
-   Les caractères génériques sont pris en charge :
    -   `telegram.*` correspond à `telegram.http`
    -   `*` active tous les indicateurs

## Activation via la configuration

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Plusieurs indicateurs :

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

Redémarrez la passerelle après avoir modifié les indicateurs.

## Surcharge par variable d'environnement (ponctuelle)

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Désactiver tous les indicateurs :

```
OPENCLAW_DIAGNOSTICS=0
```

## Destination des journaux

Les indicateurs émettent des journaux dans le fichier de journal de diagnostic standard. Par défaut :

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Si vous définissez `logging.file`, utilisez ce chemin à la place. Les journaux sont au format JSONL (un objet JSON par ligne). La rédaction s'applique toujours en fonction de `logging.redactSensitive`.

## Extraire les journaux

Sélectionnez le fichier journal le plus récent :

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Filtrer pour les diagnostics HTTP Telegram :

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

Ou suivez les journaux en temps réel pendant la reproduction :

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

Pour les passerelles distantes, vous pouvez également utiliser `openclaw logs --follow` (voir [/cli/logs](../cli/logs.md)).

## Notes

-   Si `logging.level` est défini à un niveau supérieur à `warn`, ces journaux peuvent être supprimés. La valeur par défaut `info` convient.
-   Il est sûr de laisser les indicateurs activés ; ils n'affectent que le volume des journaux pour le sous-système spécifique.
-   Utilisez [/logging](../logging.md) pour modifier les destinations, les niveaux et la rédaction des journaux.

[Problème de crash Node + tsx](../debug/node-issue.md)[Node.js](../install/node.md)

---