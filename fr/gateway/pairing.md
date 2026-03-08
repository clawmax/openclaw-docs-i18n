

  Réseautage et découverte

  
# Appairage Propriétaire de la Passerelle

Dans l'appairage propriétaire de la passerelle, la **Passerelle** est la source de vérité pour déterminer quels nœuds peuvent rejoindre le réseau. Les interfaces utilisateur (application macOS, futurs clients) sont simplement des interfaces frontales qui approuvent ou rejettent les demandes en attente. **Important :** Les nœuds WS utilisent l'**appairage d'appareil** (rôle `node`) pendant `connect`. `node.pair.*` est un magasin d'appairage séparé et ne **conditionne pas** la poignée de main WS. Seuls les clients qui appellent explicitement `node.pair.*` utilisent ce flux.

## Concepts

-   **Demande en attente** : un nœud a demandé à rejoindre ; nécessite une approbation.
-   **Nœud appairé** : nœud approuvé avec un jeton d'authentification émis.
-   **Transport** : le point de terminaison WS de la passerelle transmet les demandes mais ne décide pas de l'appartenance. (La prise en charge de l'ancien pont TCP est dépréciée/supprimée.)

## Fonctionnement de l'appairage

1.  Un nœud se connecte au WS de la passerelle et demande un appairage.
2.  La passerelle stocke une **demande en attente** et émet `node.pair.requested`.
3.  Vous approuvez ou rejetez la demande (CLI ou interface utilisateur).
4.  Lors de l'approbation, la passerelle émet un **nouveau jeton** (les jetons sont renouvelés lors d'un ré-appairage).
5.  Le nœud se reconnecte en utilisant le jeton et est désormais « appairé ».

Les demandes en attente expirent automatiquement après **5 minutes**.

## Flux de travail CLI (adapté au mode sans interface)

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "iPad Salon"
```

`nodes status` affiche les nœuds appairés/connectés et leurs capacités.

## Surface API (protocole de la passerelle)

Événements :

-   `node.pair.requested` — émis lorsqu'une nouvelle demande en attente est créée.
-   `node.pair.resolved` — émis lorsqu'une demande est approuvée/rejetée/expirée.

Méthodes :

-   `node.pair.request` — créer ou réutiliser une demande en attente.
-   `node.pair.list` — lister les nœuds en attente et appairés.
-   `node.pair.approve` — approuver une demande en attente (émet un jeton).
-   `node.pair.reject` — rejeter une demande en attente.
-   `node.pair.verify` — vérifier `{ nodeId, token }`.

Notes :

-   `node.pair.request` est idempotent par nœud : des appels répétés renvoient la même demande en attente.
-   L'approbation **génère toujours** un nouveau jeton ; aucun jeton n'est jamais renvoyé par `node.pair.request`.
-   Les demandes peuvent inclure `silent: true` comme indication pour les flux d'approbation automatique.

## Approbation automatique (application macOS)

L'application macOS peut éventuellement tenter une **approbation silencieuse** lorsque :

-   la demande est marquée `silent`, et
-   l'application peut vérifier une connexion SSH à l'hôte de la passerelle en utilisant le même utilisateur.

Si l'approbation silencieuse échoue, elle revient à l'invite normale « Approuver/Rejeter ».

## Stockage (local, privé)

L'état d'appairage est stocké sous le répertoire d'état de la passerelle (par défaut `~/.openclaw`) :

-   `~/.openclaw/nodes/paired.json`
-   `~/.openclaw/nodes/pending.json`

Si vous remplacez `OPENCLAW_STATE_DIR`, le dossier `nodes/` se déplace avec lui. Notes de sécurité :

-   Les jetons sont des secrets ; traitez `paired.json` comme sensible.
-   Le renouvellement d'un jeton nécessite une ré-approbation (ou la suppression de l'entrée du nœud).

## Comportement du transport

-   Le transport est **sans état** ; il ne stocke pas l'appartenance.
-   Si la passerelle est hors ligne ou si l'appairage est désactivé, les nœuds ne peuvent pas s'appairer.
-   Si la passerelle est en mode distant, l'appairage se fait toujours contre le magasin de la passerelle distante.

[Modèle de réseau](./network-model.md)[Découverte et Transports](./discovery.md)