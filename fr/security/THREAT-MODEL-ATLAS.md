title: "Modèle de Menace de Sécurité OpenClaw AI Utilisant le Cadre MITRE ATLAS"
description: "Analysez les menaces adverses pour les plateformes d'agents IA avec notre modèle de menace MITRE ATLAS. Découvrez l'injection de prompt, les attaques de chaîne d'approvisionnement des compétences et les mesures d'atténuation."
keywords: ["mitre atlas", "sécurité ia", "modèle de menace", "injection de prompt", "sécurité llm", "menaces adverses", "compromission de la chaîne d'approvisionnement", "sécurité openclaw"]
---

  Sécurité

  
# MODÈLE DE MENACE ATLAS

## Cadre MITRE ATLAS

**Version :** 1.0-brouillon **Dernière mise à jour :** 2026-02-04 **Méthodologie :** MITRE ATLAS + Diagrammes de Flux de Données **Cadre :** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Attribution du Cadre

Ce modèle de menace est construit sur [MITRE ATLAS](https://atlas.mitre.org/), le cadre standard de l'industrie pour documenter les menaces adverses contre les systèmes IA/ML. ATLAS est maintenu par [MITRE](https://www.mitre.org/) en collaboration avec la communauté de la sécurité IA. **Ressources ATLAS clés :**

-   [Techniques ATLAS](https://atlas.mitre.org/techniques/)
-   [Tactiques ATLAS](https://atlas.mitre.org/tactics/)
-   [Études de cas ATLAS](https://atlas.mitre.org/studies/)
-   [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
-   [Contribuer à ATLAS](https://atlas.mitre.org/resources/contribute)

### Contribuer à ce Modèle de Menace

Ceci est un document vivant maintenu par la communauté OpenClaw. Voir [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) pour les directives de contribution :

-   Signaler de nouvelles menaces
-   Mettre à jour les menaces existantes
-   Proposer des chaînes d'attaque
-   Suggérer des mesures d'atténuation

* * *

## 1. Introduction

### 1.1 Objectif

Ce modèle de menace documente les menaces adverses contre la plateforme d'agent IA OpenClaw et le marketplace de compétences ClawHub, en utilisant le cadre MITRE ATLAS conçu spécifiquement pour les systèmes IA/ML.

### 1.2 Périmètre

| Composant | Inclus | Notes |
| --- | --- | --- |
| Runtime d'Agent OpenClaw | Oui | Exécution principale de l'agent, appels d'outils, sessions |
| Passerelle | Oui | Authentification, routage, intégration des canaux |
| Intégrations de Canaux | Oui | WhatsApp, Telegram, Discord, Signal, Slack, etc. |
| Marketplace ClawHub | Oui | Publication de compétences, modération, distribution |
| Serveurs MCP | Oui | Fournisseurs d'outils externes |
| Appareils Utilisateurs | Partiel | Applications mobiles, clients de bureau |

### 1.3 Hors Périmètre

Rien n'est explicitement hors périmètre pour ce modèle de menace.

* * *

## 2. Architecture du Système

### 2.1 Limites de Confiance

```
┌─────────────────────────────────────────────────────────────────┐
│                    ZONE NON FIABLE                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LIMITE DE CONFIANCE 1 : Accès Canal              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      PASSERELLE                           │   │
│  │  • Appairage d'appareil (période de grâce 30s)            │   │
│  │  • Validation AllowFrom / AllowList                       │   │
│  │  • Auth Token/Mot de passe/Tailscale                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LIMITE DE CONFIANCE 2 : Isolation de Session     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   SESSIONS AGENT                          │   │
│  │  • Clé de session = agent:canal:pair                      │   │
│  │  • Politiques d'outils par agent                          │   │
│  │  • Journalisation des transcriptions                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LIMITE DE CONFIANCE 3 : Exécution d'Outil        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  SANDBOX D'EXÉCUTION                      │   │
│  │  • Sandbox Docker OU Hôte (exec-approvals)                │   │
│  │  • Exécution distante Node                                │   │
│  │  • Protection SSRF (épinglage DNS + blocage IP)           │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LIMITE DE CONFIANCE 4 : Contenu Externe          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              URLs RÉCUPÉRÉES / EMAILS / WEBHOOKS          │   │
│  │  • Encapsulation du contenu externe (balises XML)         │   │
│  │  • Injection d'avis de sécurité                           │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LIMITE DE CONFIANCE 5 : Chaîne d'Approvisionnement │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                             │   │
│  │  • Publication de compétences (semver, SKILL.md requis)   │   │
│  │  • Drapeaux de modération basés sur des motifs            │   │
│  │  • Analyse VirusTotal (à venir)                           │   │
│  │  • Vérification de l'ancienneté du compte GitHub          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Flux de Données

| Flux | Source | Destination | Données | Protection |
| --- | --- | --- | --- | --- |
| F1 | Canal | Passerelle | Messages utilisateur | TLS, AllowFrom |
| F2 | Passerelle | Agent | Messages routés | Isolation de session |
| F3 | Agent | Outils | Invocations d'outils | Application des politiques |
| F4 | Agent | Externe | requêtes web\_fetch | Blocage SSRF |
| F5 | ClawHub | Agent | Code de compétence | Modération, analyse |
| F6 | Agent | Canal | Réponses | Filtrage de sortie |

* * *

## 3. Analyse des Menaces par Tactic ATLAS

### 3.1 Reconnaissance (AML.TA0002)

#### T-RECON-001 : Découverte des Points de Terminaison d'Agent

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0006 - Balayage Actif |
| **Description** | L'attaquant scanne les points de terminaison exposés de la passerelle OpenClaw |
| **Vecteur d'Attaque** | Balayage réseau, requêtes shodan, énumération DNS |
| **Composants Affectés** | Passerelle, points de terminaison API exposés |
| **Atténuations Actuelles** | Option d'authentification Tailscale, liaison à loopback par défaut |
| **Risque Résiduel** | Moyen - Les passerelles publiques sont découvrables |
| **Recommandations** | Documenter le déploiement sécurisé, ajouter une limitation de débit sur les points de terminaison de découverte |

#### T-RECON-002 : Sondage d'Intégration de Canal

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0006 - Balayage Actif |
| **Description** | L'attaquant sonde les canaux de messagerie pour identifier les comptes gérés par IA |
| **Vecteur d'Attaque** | Envoi de messages tests, observation des modèles de réponse |
| **Composants Affectés** | Toutes les intégrations de canaux |
| **Atténuations Actuelles** | Aucune spécifique |
| **Risque Résiduel** | Faible - Valeur limitée de la découverte seule |
| **Recommandations** | Envisager une randomisation du timing des réponses |

* * *

### 3.2 Accès Initial (AML.TA0004)

#### T-ACCESS-001 : Interception du Code d'Appairage

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0040 - Accès API d'Inférence de Modèle IA |
| **Description** | L'attaquant intercepte le code d'appairage pendant la période de grâce de 30s |
| **Vecteur d'Attaque** | Shoulder surfing, reniflage réseau, ingénierie sociale |
| **Composants Affectés** | Système d'appairage d'appareil |
| **Atténuations Actuelles** | Expiration 30s, codes envoyés via le canal existant |
| **Risque Résiduel** | Moyen - Période de grâce exploitable |
| **Recommandations** | Réduire la période de grâce, ajouter une étape de confirmation |

#### T-ACCESS-002 : Usurpation AllowFrom

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0040 - Accès API d'Inférence de Modèle IA |
| **Description** | L'attaquant usurpe l'identité de l'expéditeur autorisé dans le canal |
| **Vecteur d'Attaque** | Dépend du canal - usurpation de numéro de téléphone, usurpation de nom d'utilisateur |
| **Composants Affectés** | Validation AllowFrom par canal |
| **Atténuations Actuelles** | Vérification d'identité spécifique au canal |
| **Risque Résiduel** | Moyen - Certains canaux vulnérables à l'usurpation |
| **Recommandations** | Documenter les risques spécifiques aux canaux, ajouter une vérification cryptographique si possible |

#### T-ACCESS-003 : Vol de Token

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0040 - Accès API d'Inférence de Modèle IA |
| **Description** | L'attaquant vole les jetons d'authentification des fichiers de configuration |
| **Vecteur d'Attaque** | Logiciel malveillant, accès non autorisé à l'appareil, exposition de sauvegarde de configuration |
| **Composants Affectés** | ~/.openclaw/credentials/, stockage de configuration |
| **Atténuations Actuelles** | Permissions de fichiers |
| **Risque Résiduel** | Élevé - Les jetons sont stockés en clair |
| **Recommandations** | Implémenter le chiffrement des jetons au repos, ajouter la rotation des jetons |

* * *

### 3.3 Exécution (AML.TA0005)

#### T-EXEC-001 : Injection de Prompt Directe

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0051.000 - Injection de Prompt LLM : Directe |
| **Description** | L'attaquant envoie des prompts conçus pour manipuler le comportement de l'agent |
| **Vecteur d'Attaque** | Messages de canal contenant des instructions adverses |
| **Composants Affectés** | LLM de l'agent, toutes les surfaces d'entrée |
| **Atténuations Actuelles** | Détection de motifs, encapsulation du contenu externe |
| **Risque Résiduel** | Critique - Détection seulement, pas de blocage ; les attaques sophistiquées contournent |
| **Recommandations** | Implémenter une défense multicouche, validation de sortie, confirmation utilisateur pour les actions sensibles |

#### T-EXEC-002 : Injection de Prompt Indirecte

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0051.001 - Injection de Prompt LLM : Indirecte |
| **Description** | L'attaquant intègre des instructions malveillantes dans le contenu récupéré |
| **Vecteur d'Attaque** | URLs malveillantes, emails empoisonnés, webhooks compromis |
| **Composants Affectés** | web\_fetch, ingestion d'emails, sources de données externes |
| **Atténuations Actuelles** | Encapsulation du contenu avec balises XML et avis de sécurité |
| **Risque Résiduel** | Élevé - Le LLM peut ignorer les instructions d'encapsulation |
| **Recommandations** | Implémenter l'assainissement du contenu, contextes d'exécution séparés |

#### T-EXEC-003 : Injection d'Arguments d'Outil

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0051.000 - Injection de Prompt LLM : Directe |
| **Description** | L'attaquant manipule les arguments d'outil via l'injection de prompt |
| **Vecteur d'Attaque** | Prompts conçus pour influencer les valeurs des paramètres d'outil |
| **Composants Affectés** | Toutes les invocations d'outils |
| **Atténuations Actuelles** | Approbations d'exécution pour les commandes dangereuses |
| **Risque Résiduel** | Élevé - Repose sur le jugement de l'utilisateur |
| **Recommandations** | Implémenter la validation des arguments, appels d'outils paramétrés |

#### T-EXEC-004 : Contournement de l'Approbation d'Exécution

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0043 - Créer des Données Adverses |
| **Description** | L'attaquant conçoit des commandes qui contournent la liste d'autorisation d'approbation |
| **Vecteur d'Attaque** | Obfuscation de commande, exploitation d'alias, manipulation de chemin |
| **Composants Affectés** | exec-approvals.ts, liste d'autorisation de commandes |
| **Atténuations Actuelles** | Liste d'autorisation + mode demande |
| **Risque Résiduel** | Élevé - Pas d'assainissement des commandes |
| **Recommandations** | Implémenter la normalisation des commandes, étendre la liste de blocage |

* * *

### 3.4 Persistance (AML.TA0006)

#### T-PERSIST-001 : Installation de Compétence Malveillante

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0010.001 - Compromission de la Chaîne d'Approvisionnement : Logiciel IA |
| **Description** | L'attaquant publie une compétence malveillante sur ClawHub |
| **Vecteur d'Attaque** | Créer un compte, publier une compétence avec du code malveillant caché |
| **Composants Affectés** | ClawHub, chargement de compétences, exécution d'agent |
| **Atténuations Actuelles** | Vérification de l'ancienneté du compte GitHub, drapeaux de modération basés sur des motifs |
| **Risque Résiduel** | Critique - Pas de sandboxing, examen limité |
| **Recommandations** | Intégration VirusTotal (en cours), sandboxing des compétences, examen communautaire |

#### T-PERSIST-002 : Empoisonnement des Mises à Jour de Compétences

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0010.001 - Compromission de la Chaîne d'Approvisionnement : Logiciel IA |
| **Description** | L'attaquant compromet une compétence populaire et pousse une mise à jour malveillante |
| **Vecteur d'Attaque** | Compromission de compte, ingénierie sociale du propriétaire de la compétence |
| **Composants Affectés** | Gestion de version ClawHub, flux de mise à jour automatique |
| **Atténuations Actuelles** | Empreinte de version |
| **Risque Résiduel** | Élevé - Les mises à jour automatiques peuvent récupérer des versions malveillantes |
| **Recommandations** | Implémenter la signature des mises à jour, capacité de retour arrière, épinglage de version |

#### T-PERSIST-003 : Altération de la Configuration de l'Agent

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0010.002 - Compromission de la Chaîne d'Approvisionnement : Données |
| **Description** | L'attaquant modifie la configuration de l'agent pour persister l'accès |
| **Vecteur d'Attaque** | Modification du fichier de configuration, injection de paramètres |
| **Composants Affectés** | Configuration de l'agent, politiques d'outils |
| **Atténuations Actuelles** | Permissions de fichiers |
| **Risque Résiduel** | Moyen - Nécessite un accès local |
| **Recommandations** | Vérification de l'intégrité de la configuration, journalisation d'audit pour les changements de configuration |

* * *

### 3.5 Évasion de Défense (AML.TA0007)

#### T-EVADE-001 : Contournement des Motifs de Modération

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0043 - Créer des Données Adverses |
| **Description** | L'attaquant conçoit le contenu de la compétence pour contourner les motifs de modération |
| **Vecteur d'Attaque** | Homoglyphes Unicode, astuces d'encodage, chargement dynamique |
| **Composants Affectés** | ClawHub moderation.ts |
| **Atténuations Actuelles** | Règles FLAG\_RULES basées sur des motifs |
| **Risque Résiduel** | Élevé - Les regex simples sont facilement contournées |
| **Recommandations** | Ajouter une analyse comportementale (VirusTotal Code Insight), détection basée sur AST |

#### T-EVADE-002 : Échappement de l'Encapsulation de Contenu

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0043 - Créer des Données Adverses |
| **Description** | L'attaquant conçoit un contenu qui échappe au contexte d'encapsulation XML |
| **Vecteur d'Attaque** | Manipulation de balises, confusion de contexte, remplacement d'instructions |
| **Composants Affectés** | Encapsulation du contenu externe |
| **Atténuations Actuelles** | Balises XML + avis de sécurité |
| **Risque Résiduel** | Moyen - De nouveaux échappements sont découverts régulièrement |
| **Recommandations** | Plusieurs couches d'encapsulation, validation côté sortie |

* * *

### 3.6 Découverte (AML.TA0008)

#### T-DISC-001 : Énumération d'Outils

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0040 - Accès API d'Inférence de Modèle IA |
| **Description** | L'attaquant énumère les outils disponibles via des prompts |
| **Vecteur d'Attaque** | Requêtes du style "Quels outils avez-vous ?" |
| **Composants Affectés** | Registre d'outils de l'agent |
| **Atténuations Actuelles** | Aucune spécifique |
| **Risque Résiduel** | Faible - Les outils sont généralement documentés |
| **Recommandations** | Envisager des contrôles de visibilité des outils |

#### T-DISC-002 : Extraction de Données de Session

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0040 - Accès API d'Inférence de Modèle IA |
| **Description** | L'attaquant extrait des données sensibles du contexte de session |
| **Vecteur d'Attaque** | Requêtes "De quoi avons-nous discuté ?", sondage de contexte |
| **Composants Affectés** | Transcripts de session, fenêtre de contexte |
| **Atténuations Actuelles** | Isolation de session par expéditeur |
| **Risque Résiduel** | Moyen - Les données au sein de la session sont accessibles |
| **Recommandations** | Implémenter la rédaction des données sensibles dans le contexte |

* * *

### 3.7 Collecte & Exfiltration (AML.TA0009, AML.TA0010)

#### T-EXFIL-001 : Vol de Données via web\_fetch

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0009 - Collecte |
| **Description** | L'attaquant exfiltre des données en ordonnant à l'agent d'envoyer vers une URL externe |
| **Vecteur d'Attaque** | Injection de prompt amenant l'agent à POSTer des données vers le serveur de l'attaquant |
| **Composants Affectés** | Outil web\_fetch |
| **Atténuations Actuelles** | Blocage SSRF pour les réseaux internes |
| **Risque Résiduel** | Élevé - Les URLs externes sont autorisées |
| **Recommandations** | Implémenter une liste d'autorisation d'URLs, sensibilisation à la classification des données |

#### T-EXFIL-002 : Envoi Non Autorisé de Messages

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0009 - Collecte |
| **Description** | L'attaquant amène l'agent à envoyer des messages contenant des données sensibles |
| **Vecteur d'Attaque** | Injection de prompt amenant l'agent à envoyer un message à l'attaquant |
| **Composants Affectés** | Outil de message, intégrations de canaux |
| **Atténuations Actuelles** | Contrôle des messages sortants |
| **Risque Résiduel** | Moyen - Le contrôle peut être contourné |
| **Recommandations** | Exiger une confirmation explicite pour les nouveaux destinataires |

#### T-EXFIL-003 : Récolte d'Identifiants

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0009 - Collecte |
| **Description** | Une compétence malveillante récolte des identifiants depuis le contexte de l'agent |
| **Vecteur d'Attaque** | Le code de la compétence lit les variables d'environnement, les fichiers de configuration |
| **Composants Affectés** | Environnement d'exécution des compétences |
| **Atténuations Actuelles** | Aucune spécifique aux compétences |
| **Risque Résiduel** | Critique - Les compétences s'exécutent avec les privilèges de l'agent |
| **Recommandations** | Sandboxing des compétences, isolation des identifiants |

* * *

### 3.8 Impact (AML.TA0011)

#### T-IMPACT-001 : Exécution Non Autorisée de Commande

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0031 - Éroder l'Intégrité du Modèle IA |
| **Description** | L'attaquant exécute des commandes arbitraires sur le système de l'utilisateur |
| **Vecteur d'Attaque** | Injection de prompt combinée avec un contournement d'approbation d'exécution |
| **Composants Affectés** | Outil Bash, exécution de commande |
| **Atténuations Actuelles** | Approbations d'exécution, option sandbox Docker |
| **Risque Résiduel** | Critique - Exécution sur l'hôte sans sandbox |
| **Recommandations** | Par défaut en sandbox, améliorer l'UX d'approbation |

#### T-IMPACT-002 : Épuisement des Ressources (DoS)

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0031 - Éroder l'Intégrité du Modèle IA |
| **Description** | L'attaquant épuise les crédits API ou les ressources de calcul |
| **Vecteur d'Attaque** | Inondation automatisée de messages, appels d'outils coûteux |
| **Composants Affectés** | Passerelle, sessions d'agent, fournisseur d'API |
| **Atténuations Actuelles** | Aucune |
| **Risque Résiduel** | Élevé - Pas de limitation de débit |
| **Recommandations** | Implémenter des limites de débit par expéditeur, budgets de coût |

#### T-IMPACT-003 : Atteinte à la Réputation

| Attribut | Valeur |
| --- | --- |
| **ID ATLAS** | AML.T0031 - Éroder l'Intégrité du Modèle IA |
| **Description** | L'attaquant amène l'agent à envoyer un contenu nuisible/offensant |
| **Vecteur d'Attaque** | Injection de prompt amenant des réponses inappropriées |
| **Composants Affectés** | Génération de sortie, messagerie de canal |
| **Atténuations Actuelles** | Politiques de contenu du fournisseur LLM |
| **Risque Résiduel** | Moyen - Les filtres des fournisseurs sont imparfaits |
| **Recommandations** | Couche de filtrage de sortie, contrôles utilisateur |

* * *

## 4. Analyse de la Chaîne d'Approvisionnement ClawHub

### 4.1 Contrôles de Sécurité Actuels

| Contrôle | Implémentation | Efficacité |
| --- | --- | --- |
| Ancienneté du Compte GitHub | `requireGitHubAccountAge()` | Moyenne - Augmente la barrière pour les nouveaux attaquants |
| Assainissement des Chemins | `sanitizePath()` | Élevée - Empêche la traversée de chemin |
| Validation du Type de Fichier | `isTextFile()` | Moyenne - Seulement les fichiers texte, mais peuvent toujours être malveillants |
| Limites de Taille | 50MB total du bundle | Élevée - Empêche l'épuisement des ressources |
| SKILL.md Requis | Readme obligatoire | Faible valeur de sécurité - Informationnel seulement |
| Modération par Motifs | FLAG\_RULES dans moderation.ts | Faible - Facilement contournée |
| Statut de Modération | Champ `moderationStatus` | Moyenne - Examen manuel possible |

### 4.2 Motifs de Drapeaux de Modération

Motifs actuels dans `moderation.ts` :

```
// Identifiants connus comme mauvais
/(keepcold131\/ClawdAuthenticatorTool|ClawdAuthenticatorTool)/i

// Mots-clés suspects
/(malware|stealer|phish|phishing|keylogger)/i
/(api[-_ ]?key|token|password|private key|secret)/i
/(wallet|seed phrase|mnemonic|crypto)/i
/(discord\.gg|webhook|hooks\.slack)/i
/(curl[^\n]+\|\s*(sh|bash))/i
/(bit\.ly|tinyurl\.com|t\.co|goo\.gl|is\.gd)/i
```

**Limitations :**

-   Vérifie seulement le slug, displayName, summary, frontmatter, metadata, chemins de fichiers
-   N'analyse pas le contenu réel du code de la compétence
-   Regex simple facilement contournée par obfuscation
-   Pas d'analyse comportementale

### 4.3 Améliorations Prévues

| Amélioration | Statut | Impact |
| --- | --- | --- |
| Intégration VirusTotal | En Cours | Élevé - Analyse comportementale Code Insight |
| Signalement Communautaire | Partiel (table `skillReports` existe) | Moyen |
| Journalisation d'Audit | Partiel (table `auditLogs` existe) | Moyen |
| Système de Badges | Implémenté | Moyen - `highlighted`, `official`, `deprecated`, `redactionApproved` |

* * *

## 5. Matrice de Risque

### 5.1 Probabilité vs Impact

| ID Menace | Probabilité | Impact | Niveau de Risque | Priorité |
| --- | --- | --- | --- | --- |
| T-EXEC-001 | Élevée | Critique | **Critique** | P0 |
| T-PERSIST-001 | Élevée | Critique | **Critique** | P0 |
| T-EXFIL-003 | Moyenne | Critique | **Critique** | P0 |
| T-IMPACT-001 | Moyenne | Critique | **Élevé** | P1 |
| T-EXEC-002 | Élevée | Élevé | **Élevé** | P1 |
| T-EXEC-004 | Moyenne | Élevé | **Élevé** | P1 |
| T-ACCESS-003 | Moyenne | Élevé | **Élevé** | P1 |
| T-EXFIL-001 | Moyenne | Élevé | **Élevé** | P1 |
| T-IMPACT-002 | Élevée | Moyen | **Élevé** | P1 |
| T-EVADE-001 | Élevée | Moyen | **Moyen** | P2 |
| T-ACCESS-001 | Faible | Élevé | **Moyen** | P2 |
| T-ACCESS-002 | Faible | Élevé | **Moyen** | P2 |
| T-PERSIST-002 | Faible | Élevé | **Moyen** | P2 |

### 5.2 Chaînes d'Attaque Critiques

**Chaîne d'Attaque 1 : Vol de Données Basé sur une Compétence**

```
T-PERSIST-001 → T-EVADE-001 → T-EXFIL-003
(Publier une compétence malveillante) → (Contourner la modération) → (Récolter les identifiants)
```

**Chaîne d'Attaque 2 : Injection de Prompt vers Exécution de Code à Distance**

```
T-EXEC-001 → T-EXEC-004 → T-IMPACT-001
(Injecter un prompt) → (Contourner l'approbation d'exécution) → (Exécuter des commandes)
```

**Chaîne d'Attaque 3 : Injection Indirecte via Contenu Récupéré**

```
T-EXEC-002 → T-EXFIL-001 → Exfiltration externe
(Empoisonner le contenu d'une URL) → (L'agent récupère et suit les instructions) → (Données envoyées à l'attaquant)
```

* * *

## 6. Résumé des Recommandations

### 6.1 Immédiates (P0)

| ID | Recommandation | Adresse |
| --- | --- | --- |
| R-001 | Compléter l'intégration VirusTotal | T-PERSIST-001, T-EVADE-001 |
| R-002 | Implémenter le sandboxing des compétences | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Ajouter une validation de sortie pour les actions sensibles | T-EXEC-001, T-EXEC-002 |

### 6.2 Court Terme (P1)

| ID | Recommandation | Adresse |
| --- | --- | --- |
| R-004 | Implémenter la limitation de débit | T-IMPACT-002 |
| R-005 | Ajouter le chiffrement des jetons au repos | T-ACCESS-003 |
| R-006 | Améliorer l'UX et la validation des approbations d'exécution | T-EXEC-004 |
| R-007 | Implémenter une liste d'autorisation d'URLs pour web\_fetch | T-EXFIL-001 |

### 6.3 Moyen Terme (P2)

| ID | Recommandation | Adresse |
| --- | --- | --- |
| R-008 | Ajouter une vérification cryptographique des canaux si possible | T-ACCESS-002 |
| R-009 | Implémenter la vérification de l'intégrité de la configuration | T-PERSIST-003 |
| R-010 | Ajouter la signature des mises à jour et l'épinglage de version | T-PERSIST-002 |

* * *

## 7. Annexes

### 7.1 Correspondance des Techniques ATLAS

| ID ATLAS | Nom de la Technique | Menaces OpenClaw |
| --- | --- | --- |
| AML.T0006 | Balayage Actif | T