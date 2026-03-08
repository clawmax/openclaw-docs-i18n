

  Mise en réseau et découverte

  
# Modèle de réseau

La plupart des opérations transitent par la Passerelle (`openclaw gateway`), un processus unique de longue durée qui possède les connexions aux canaux et le plan de contrôle WebSocket.

## Règles fondamentales

-   Une Passerelle par hôte est recommandée. C'est le seul processus autorisé à posséder la session WhatsApp Web. Pour les bots de secours ou une isolation stricte, exécutez plusieurs passerelles avec des profils et des ports isolés. Voir [Passerelles multiples](./multiple-gateways.md).
-   Boucle locale en priorité : le WebSocket de la Passerelle utilise par défaut `ws://127.0.0.1:18789`. L'assistant génère un jeton de passerelle par défaut, même pour la boucle locale. Pour un accès via tailnet, exécutez `openclaw gateway --bind tailnet --token ...` car les jetons sont requis pour les liaisons non locales.
-   Les nœuds se connectent au WebSocket de la Passerelle via LAN, tailnet ou SSH selon les besoins. L'ancien pont TCP est obsolète.
-   L'hôte Canvas est servi par le serveur HTTP de la Passerelle sur **le même port** que la Passerelle (par défaut `18789`) :
    -   `/__openclaw__/canvas/`
    -   `/__openclaw__/a2ui/` Lorsque `gateway.auth` est configuré et que la Passerelle est liée au-delà de la boucle locale, ces routes sont protégées par l'authentification de la Passerelle. Les clients nœuds utilisent des URL de capacité limitées au nœud, liées à leur session WebSocket active. Voir [Configuration de la passerelle](./configuration.md) (`canvasHost`, `gateway`).
-   L'utilisation à distance se fait typiquement par tunnel SSH ou VPN tailnet. Voir [Accès à distance](./remote.md) et [Découverte](./discovery.md).

[Modèles locaux](./local-models.md)[Appariement géré par la passerelle](./pairing.md)

---