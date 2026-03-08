

  Environnement et débogage

  
# Plantage Node + tsx

Cette page décrit le dépannage des plantages **Node.js et tsx** lors de l'exécution de la passerelle ou du CLI OpenClaw (par ex. `node --import tsx`, ou des scripts utilisant tsx).

## Causes courantes

- **Version de Node** : OpenClaw nécessite Node 22+. Une version plus ancienne peut provoquer des plantages à l'exécution ou des modules natifs.
- **tsx / chargeur** : Les plantages au démarrage ou au chargement TypeScript peuvent venir de la version de tsx, de l'ordre de `--import`, ou de chargeurs en conflit.
- **Mémoire / natif** : Des charges importantes ou des modules natifs peuvent provoquer des OOM ou des segfaults ; voir [Node.js](../install/node.md) pour la version et l'environnement.

## Vérifications rapides

1. **Version de Node** : Exécuter `node -v` (attendre v22 ou plus).
2. **Reproduire sans tsx** : Tenter le même flux avec `node` seul et du JS précompilé pour voir si le plantage est spécifique à tsx.
3. **Diagnostics** : Utiliser les [indicateurs de diagnostic](../diagnostics/flags.md) (ex. `OPENCLAW_DEBUG_*`) pour obtenir plus de logs avant un plantage.

## Voir aussi

- [Débogage](../help/debugging.md) — surcharges runtime, mode watch de la passerelle, profil dev
- [Scripts](../help/scripts.md) — scripts utilitaires et conventions
- [Node.js](../install/node.md) — version de Node et PATH
- [Indicateurs de diagnostic](../diagnostics/flags.md) — indicateurs de débogage et de trace
