

  Sécurité

  
# CONTRIBUER AU MODÈLE DE MENACES

Merci de contribuer à rendre OpenClaw plus sécurisé. Ce modèle de menaces est un document vivant et nous accueillons les contributions de tous - vous n'avez pas besoin d'être un expert en sécurité.

## Façons de contribuer

### Ajouter une menace

Vous avez repéré un vecteur d'attaque ou un risque que nous n'avons pas couvert ? Ouvrez un ticket sur [openclaw/trust](https://github.com/openclaw/trust/issues) et décrivez-le avec vos propres mots. Vous n'avez pas besoin de connaître de cadres de travail ou de remplir tous les champs - décrivez simplement le scénario. **Il est utile d'inclure (mais pas obligatoire) :**

-   Le scénario d'attaque et comment il pourrait être exploité
-   Quelles parties d'OpenClaw sont affectées (CLI, passerelle, canaux, ClawHub, serveurs MCP, etc.)
-   Quelle est sa gravité selon vous (faible / moyenne / élevée / critique)
-   Tous liens vers des recherches, CVE ou exemples réels connexes

Nous nous chargerons du mapping ATLAS, des identifiants de menace et de l'évaluation des risques lors de la revue. Si vous souhaitez inclure ces détails, c'est parfait - mais ce n'est pas attendu.

> **Ceci est pour ajouter au modèle de menaces, pas pour signaler des vulnérabilités actives.** Si vous avez trouvé une vulnérabilité exploitable, consultez notre [page Trust](https://trust.openclaw.ai) pour les instructions de divulgation responsable.

### Suggérer une mesure d'atténuation

Vous avez une idée pour traiter une menace existante ? Ouvrez un ticket ou une PR en référençant la menace. Les mesures d'atténuation utiles sont spécifiques et actionnables - par exemple, "une limitation de débit par expéditeur de 10 messages/minute au niveau de la passerelle" est mieux que "mettre en place une limitation de débit".

### Proposer une chaîne d'attaque

Les chaînes d'attaque montrent comment plusieurs menaces se combinent pour former un scénario d'attaque réaliste. Si vous voyez une combinaison dangereuse, décrivez les étapes et comment un attaquant les enchaînerait. Un court récit de la façon dont l'attaque se déroule en pratique est plus précieux qu'un modèle formel.

### Corriger ou améliorer le contenu existant

Fautes de frappe, clarifications, informations obsolètes, meilleurs exemples - les PR sont les bienvenues, pas besoin de ticket.

## Ce que nous utilisons

### MITRE ATLAS

Ce modèle de menaces est construit sur [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), un cadre conçu spécifiquement pour les menaces IA/ML comme l'injection de prompt, l'utilisation abusive d'outils et l'exploitation d'agents. Vous n'avez pas besoin de connaître ATLAS pour contribuer - nous mappons les soumissions au cadre lors de la revue.

### Identifiants de menace

Chaque menace reçoit un identifiant comme `T-EXEC-003`. Les catégories sont :

| Code | Catégorie |
| --- | --- |
| RECON | Reconnaissance - collecte d'informations |
| ACCESS | Accès initial - obtenir un accès |
| EXEC | Exécution - exécution d'actions malveillantes |
| PERSIST | Persistance - maintien de l'accès |
| EVADE | Évasion de défense - éviter la détection |
| DISC | Découverte - apprendre l'environnement |
| EXFIL | Exfiltration - vol de données |
| IMPACT | Impact - dommage ou perturbation |

Les identifiants sont attribués par les mainteneurs lors de la revue. Vous n'avez pas besoin d'en choisir un.

### Niveaux de risque

| Niveau | Signification |
| --- | --- |
| **Critique** | Compromission totale du système, ou probabilité élevée + impact critique |
| **Élevé** | Dommages significatifs probables, ou probabilité moyenne + impact critique |
| **Moyen** | Risque modéré, ou faible probabilité + impact élevé |
| **Faible** | Peu probable et impact limité |

Si vous n'êtes pas sûr du niveau de risque, décrivez simplement l'impact et nous l'évaluerons.

## Processus de revue

1.  **Tri** - Nous examinons les nouvelles soumissions sous 48 heures
2.  **Évaluation** - Nous vérifions la faisabilité, attribuons le mapping ATLAS et l'identifiant de menace, validons le niveau de risque
3.  **Documentation** - Nous nous assurons que tout est formaté et complet
4.  **Fusion** - Ajouté au modèle de menaces et à la visualisation

## Ressources

-   [Site web ATLAS](https://atlas.mitre.org/)
-   [Techniques ATLAS](https://atlas.mitre.org/techniques/)
-   [Études de cas ATLAS](https://atlas.mitre.org/studies/)
-   [Modèle de menaces OpenClaw](./THREAT-MODEL-ATLAS.md)

## Contact

-   **Vulnérabilités de sécurité :** Consultez notre [page Trust](https://trust.openclaw.ai) pour les instructions de signalement
-   **Questions sur le modèle de menaces :** Ouvrez un ticket sur [openclaw/trust](https://github.com/openclaw/trust/issues)
-   **Discussion générale :** Canal Discord #security

## Reconnaissance

Les contributeurs au modèle de menaces sont reconnus dans les remerciements du modèle de menaces, les notes de version et le hall of fame de la sécurité OpenClaw pour les contributions significatives.

[MODÈLE DE MENACES ATLAS](./THREAT-MODEL-ATLAS.md)[Web](../web.md)

---