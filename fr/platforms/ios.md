

  Aperçu des plateformes

  
# Application iOS

Disponibilité : prévisualisation interne. L'application iOS n'est pas encore distribuée publiquement.

## Fonctionnalités

-   Se connecte à une passerelle via WebSocket (LAN ou tailnet).
-   Expose les capacités du nœud : Canvas, Capture d'écran, Capture de caméra, Localisation, Mode conversation, Réveil vocal.
-   Reçoit les commandes `node.invoke` et rapporte les événements d'état du nœud.

## Prérequis

-   Une passerelle en cours d'exécution sur un autre appareil (macOS, Linux ou Windows via WSL2).
-   Un chemin réseau :
    -   Même LAN via Bonjour, **ou**
    -   Tailnet via DNS-SD unicast (exemple de domaine : `openclaw.internal.`), **ou**
    -   Hôte/port manuel (solution de secours).

## Démarrage rapide (appairage + connexion)

1.  Démarrez la passerelle :

```bash
openclaw gateway --port 18789
```

2.  Dans l'application iOS, ouvrez les Paramètres et choisissez une passerelle découverte (ou activez l'hôte manuel et saisissez l'hôte/le port).
3.  Approuvez la demande d'appairage sur l'hôte de la passerelle :

```bash
openclaw devices list
openclaw devices approve <requestId>
```

4.  Vérifiez la connexion :

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

## Chemins de découverte

### Bonjour (LAN)

La passerelle publie `_openclaw-gw._tcp` sur `local.`. L'application iOS les liste automatiquement.

### Tailnet (inter-réseaux)

Si mDNS est bloqué, utilisez une zone DNS-SD unicast (choisissez un domaine ; exemple : `openclaw.internal.`) et le DNS fractionné Tailscale. Voir [Bonjour](../gateway/bonjour.md) pour l'exemple CoreDNS.

### Hôte/port manuel

Dans les Paramètres, activez **Hôte manuel** et saisissez l'hôte et le port de la passerelle (par défaut `18789`).

## Canvas + A2UI

Le nœud iOS affiche un canevas WKWebView. Utilisez `node.invoke` pour le piloter :

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18789/__openclaw__/canvas/"}'
```

Notes :

-   L'hôte du canevas de la passerelle sert `/__openclaw__/canvas/` et `/__openclaw__/a2ui/`.
-   Il est servi par le serveur HTTP de la passerelle (même port que `gateway.port`, par défaut `18789`).
-   Le nœud iOS navigue automatiquement vers A2UI lors de la connexion lorsqu'une URL d'hôte de canevas est publiée.
-   Revenez à l'échafaudage intégré avec `canvas.navigate` et `{"url":""}`.

### Évaluation / capture du canevas

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

## Réveil vocal + mode conversation

-   Le réveil vocal et le mode conversation sont disponibles dans les Paramètres.
-   iOS peut suspendre l'audio en arrière-plan ; considérez les fonctionnalités vocales comme étant au mieux de leurs capacités lorsque l'application n'est pas active.

## Erreurs courantes

-   `NODE_BACKGROUND_UNAVAILABLE` : ramenez l'application iOS au premier plan (les commandes de canevas/caméra/écran le nécessitent).
-   `A2UI_HOST_NOT_CONFIGURED` : la passerelle n'a pas publié d'URL d'hôte de canevas ; vérifiez `canvasHost` dans la [configuration de la passerelle](../gateway/configuration.md).
-   L'invite d'appairage n'apparaît jamais : exécutez `openclaw devices list` et approuvez manuellement.
-   La reconnexion échoue après une réinstallation : le jeton d'appairage du trousseau a été effacé ; réappairez le nœud.

## Documentation associée

-   [Appairage](../channels/pairing.md)
-   [Découverte](../gateway/discovery.md)
-   [Bonjour](../gateway/bonjour.md)

[Application Android](./android.md)[DigitalOcean](./digitalocean.md)