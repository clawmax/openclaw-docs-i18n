

  Commandes CLI

  
# node

Exécutez un **hôte de nœud headless** qui se connecte au WebSocket de la passerelle et expose `system.run` / `system.which` sur cette machine.

## Pourquoi utiliser un hôte de nœud ?

Utilisez un hôte de nœud lorsque vous souhaitez que des agents **exécutent des commandes sur d'autres machines** de votre réseau sans y installer l'application compagnon macOS complète. Cas d'usage courants :

-   Exécuter des commandes sur des machines Linux/Windows distantes (serveurs de build, machines de labo, NAS).
-   Garder l'exécution **sandboxée** sur la passerelle, mais déléguer les exécutions approuvées à d'autres hôtes.
-   Fournir une cible d'exécution légère et headless pour l'automatisation ou les nœuds d'intégration continue.

L'exécution reste protégée par les **approbations d'exécution** et les listes d'autorisation par agent sur l'hôte de nœud, vous permettant ainsi de maintenir un accès aux commandes délimité et explicite.

## Proxy navigateur (zéro configuration)

Les hôtes de nœud annoncent automatiquement un proxy navigateur si `browser.enabled` n'est pas désactivé sur le nœud. Cela permet à l'agent d'utiliser l'automatisation du navigateur sur ce nœud sans configuration supplémentaire. Désactivez-le sur le nœud si nécessaire :

```json
{
  nodeHost: {
    browserProxy: {
      enabled: false,
    },
  },
}
```

## Exécution (premier plan)

```bash
openclaw node run --host <gateway-host> --port 18789
```

Options :

-   `--host ` : Hôte WebSocket de la passerelle (par défaut : `127.0.0.1`)
-   `--port ` : Port WebSocket de la passerelle (par défaut : `18789`)
-   `--tls` : Utiliser TLS pour la connexion à la passerelle
-   `--tls-fingerprint ` : Empreinte de certificat TLS attendue (sha256)
-   `--node-id ` : Remplacer l'identifiant du nœud (efface le jeton d'appairage)
-   `--display-name ` : Remplacer le nom d'affichage du nœud

## Service (arrière-plan)

Installez un hôte de nœud headless en tant que service utilisateur.

```bash
openclaw node install --host <gateway-host> --port 18789
```

Options :

-   `--host ` : Hôte WebSocket de la passerelle (par défaut : `127.0.0.1`)
-   `--port ` : Port WebSocket de la passerelle (par défaut : `18789`)
-   `--tls` : Utiliser TLS pour la connexion à la passerelle
-   `--tls-fingerprint ` : Empreinte de certificat TLS attendue (sha256)
-   `--node-id ` : Remplacer l'identifiant du nœud (efface le jeton d'appairage)
-   `--display-name ` : Remplacer le nom d'affichage du nœud
-   `--runtime ` : Runtime du service (`node` ou `bun`)
-   `--force` : Réinstaller/écraser si déjà installé

Gérez le service :

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

Utilisez `openclaw node run` pour un hôte de nœud en premier plan (sans service). Les commandes de service acceptent `--json` pour une sortie lisible par machine.

## Appairage

La première connexion crée une demande d'appairage d'appareil en attente (`role: node`) sur la passerelle. Approuvez-la via :

```bash
openclaw devices list
openclaw devices approve <requestId>
```

L'hôte de nœud stocke son identifiant de nœud, son jeton, son nom d'affichage et les informations de connexion à la passerelle dans `~/.openclaw/node.json`.

## Approbations d'exécution

`system.run` est contrôlé par des approbations d'exécution locales :

-   `~/.openclaw/exec-approvals.json`
-   [Approbations d'exécution](../tools/exec-approvals.md)
-   `openclaw approvals --node <id|name|ip>` (éditer depuis la passerelle)

[models](./models.md)[nodes](./nodes.md)

---