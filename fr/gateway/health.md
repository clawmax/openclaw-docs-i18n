

  Configuration et opérations

  
# Vérifications de santé

Guide rapide pour vérifier la connectivité des canaux sans deviner.

## Vérifications rapides

-   `openclaw status` — résumé local : accessibilité/mode de la passerelle, indication de mise à jour, ancienneté de l'authentification du canal lié, sessions + activité récente.
-   `openclaw status --all` — diagnostic local complet (lecture seule, couleur, sûr à coller pour le débogage).
-   `openclaw status --deep` — sonde également la passerelle en cours d'exécution (sondes par canal lorsque pris en charge).
-   `openclaw health --json` — demande à la passerelle en cours d'exécution un instantané complet de santé (WS uniquement ; pas de socket Baileys direct).
-   Envoyez `/status` comme message autonome dans WhatsApp/WebChat pour obtenir une réponse de statut sans invoquer l'agent.
-   Journaux : suivez `/tmp/openclaw/openclaw-*.log` et filtrez pour `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound`.

## Diagnostics approfondis

-   Identifiants sur disque : `ls -l ~/.openclaw/credentials/whatsapp//creds.json` (l'heure de modification devrait être récente).
-   Stockage des sessions : `ls -l ~/.openclaw/agents//sessions/sessions.json` (le chemin peut être remplacé dans la configuration). Le nombre et les destinataires récents sont affichés via `status`.
-   Flux de reliaison : `openclaw channels logout && openclaw channels login --verbose` lorsque les codes de statut 409–515 ou `loggedOut` apparaissent dans les journaux. (Note : le flux de connexion QR redémarre automatiquement une fois pour le statut 515 après l'appairage.)

## En cas d'échec

-   `logged out` ou statut 409–515 → reliez avec `openclaw channels logout` puis `openclaw channels login`.
-   Passerelle inaccessible → démarrez-la : `openclaw gateway --port 18789` (utilisez `--force` si le port est occupé).
-   Aucun message entrant → confirmez que le téléphone lié est en ligne et que l'expéditeur est autorisé (`channels.whatsapp.allowFrom`) ; pour les discussions de groupe, assurez-vous que la liste d'autorisation et les règles de mention correspondent (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

## Commande dédiée « health »

`openclaw health --json` demande à la passerelle en cours d'exécution son instantané de santé (pas de sockets de canal directs depuis la CLI). Elle rapporte l'ancienneté des identifiants/auth liés lorsqu'ils sont disponibles, les résumés des sondes par canal, le résumé du stockage des sessions et une durée de sonde. Elle se termine avec un code non nul si la passerelle est inaccessible ou si la sonde échoue/expire. Utilisez `--timeout ` pour remplacer la valeur par défaut de 10s.

[Authentification proxy de confiance](./trusted-proxy-auth.md)[Pulsation](./heartbeat.md)

---