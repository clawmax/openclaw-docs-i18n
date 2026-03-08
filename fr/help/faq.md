title: "Guide d'aide FAQ Installation Configuration Dépannage OpenClaw"
description: "Trouvez des réponses rapides et un dépannage approfondi pour la configuration, l'installation, VPS, multi-agent, OAuth, clés API, basculement de modèle et Raspberry Pi d'OpenClaw."
keywords: ["faq openclaw", "installation openclaw", "dépannage openclaw", "configuration openclaw", "vps openclaw", "raspberry pi openclaw", "configuration openclaw", "authentification openclaw"]
---

  Aide

  
# FAQ

Réponses rapides plus un dépannage approfondi pour des configurations réelles (développement local, VPS, multi-agent, OAuth/clés API, basculement de modèle). Pour les diagnostics d'exécution, consultez [Dépannage](../gateway/troubleshooting.md). Pour la référence complète de configuration, consultez [Configuration](../gateway/configuration.md).

## Table des matières

-   [Démarrage rapide et configuration initiale]
    -   [Je suis bloqué, quel est le moyen le plus rapide de me débloquer ?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
    -   [Quelle est la méthode recommandée pour installer et configurer OpenClaw ?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
    -   [Comment ouvrir le tableau de bord après l'intégration ?](#how-do-i-open-the-dashboard-after-onboarding)
    -   [Comment authentifier le tableau de bord (jeton) sur localhost vs distant ?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
    -   [De quel environnement d'exécution ai-je besoin ?](#what-runtime-do-i-need)
    -   [Fonctionne-t-il sur Raspberry Pi ?](#does-it-run-on-raspberry-pi)
    -   [Des conseils pour les installations sur Raspberry Pi ?](#any-tips-for-raspberry-pi-installs)
    -   [C'est bloqué sur "wake up my friend" / l'intégration n'éclot pas. Que faire ?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
    -   [Puis-je migrer ma configuration vers une nouvelle machine (Mac mini) sans refaire l'intégration ?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
    -   [Où puis-je voir les nouveautés de la dernière version ?](#where-do-i-see-what-is-new-in-the-latest-version)
    -   [Je ne peux pas accéder à docs.openclaw.ai (erreur SSL). Que faire ?](#i-cant-access-docsopenclawai-ssl-error-what-now)
    -   [Quelle est la différence entre stable et bêta ?](#whats-the-difference-between-stable-and-beta)
    -   [Comment installer la version bêta, et quelle est la différence entre bêta et dev ?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
    -   [Comment essayer les dernières versions ?](#how-do-i-try-the-latest-bits)
    -   [Combien de temps durent généralement l'installation et l'intégration ?](#how-long-does-install-and-onboarding-usually-take)
    -   [L'installateur est bloqué ? Comment obtenir plus de retours ?](#installer-stuck-how-do-i-get-more-feedback)
    -   [L'installation Windows indique git introuvable ou openclaw non reconnu](#windows-install-says-git-not-found-or-openclaw-not-recognized)
    -   [La sortie exec Windows affiche du texte chinois illisible, que dois-je faire](#windows-exec-output-shows-garbled-chinese-text-what-should-i-do)
    -   [La documentation n'a pas répondu à ma question - comment obtenir une meilleure réponse ?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
    -   [Comment installer OpenClaw sur Linux ?](#how-do-i-install-openclaw-on-linux)
    -   [Comment installer OpenClaw sur un VPS ?](#how-do-i-install-openclaw-on-a-vps)
    -   [Où sont les guides d'installation cloud/VPS ?](#where-are-the-cloudvps-install-guides)
    -   [Puis-je demander à OpenClaw de se mettre à jour lui-même ?](#can-i-ask-openclaw-to-update-itself)
    -   [Que fait réellement l'assistant d'intégration ?](#what-does-the-onboarding-wizard-actually-do)
    -   [Ai-je besoin d'un abonnement Claude ou OpenAI pour exécuter cela ?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
    -   [Puis-je utiliser un abonnement Claude Max sans clé API](#can-i-use-claude-max-subscription-without-an-api-key)
    -   [Comment fonctionne l'authentification Anthropic "setup-token" ?](#how-does-anthropic-setuptoken-auth-work)
    -   [Où trouver un setup-token Anthropic ?](#where-do-i-find-an-anthropic-setuptoken)
    -   [Prenez-vous en charge l'authentification par abonnement Claude (Claude Pro ou Max) ?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
    -   [Pourquoi vois-je `HTTP 429: rate_limit_error` d'Anthropic ?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
    -   [AWS Bedrock est-il pris en charge ?](#is-aws-bedrock-supported)
    -   [Comment fonctionne l'authentification Codex ?](#how-does-codex-auth-work)
    -   [Prenez-vous en charge l'authentification par abonnement OpenAI (Codex OAuth) ?](#do-you-support-openai-subscription-auth-codex-oauth)
    -   [Comment configurer Gemini CLI OAuth](#how-do-i-set-up-gemini-cli-oauth)
    -   [Un modèle local est-il acceptable pour des discussions occasionnelles ?](#is-a-local-model-ok-for-casual-chats)
    -   [Comment garder le trafic des modèles hébergés dans une région spécifique ?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
    -   [Dois-je acheter un Mac Mini pour installer cela ?](#do-i-have-to-buy-a-mac-mini-to-install-this)
    -   [Ai-je besoin d'un Mac mini pour la prise en charge d'iMessage ?](#do-i-need-a-mac-mini-for-imessage-support)
    -   [Si j'achète un Mac mini pour exécuter OpenClaw, puis-je le connecter à mon MacBook Pro ?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
    -   [Puis-je utiliser Bun ?](#can-i-use-bun)
    -   [Telegram : que mettre dans `allowFrom` ?](#telegram-what-goes-in-allowfrom)
    -   [Plusieurs personnes peuvent-elles utiliser un numéro WhatsApp avec différentes instances OpenClaw ?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
    -   [Puis-je exécuter un agent "chat rapide" et un agent "Opus pour le codage" ?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
    -   [Homebrew fonctionne-t-il sur Linux ?](#does-homebrew-work-on-linux)
    -   [Quelle est la différence entre l'installation hackable (git) et l'installation npm ?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
    -   [Puis-je basculer entre les installations npm et git plus tard ?](#can-i-switch-between-npm-and-git-installs-later)
    -   [Dois-je exécuter la Gateway sur mon ordinateur portable ou sur un VPS ?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
    -   [Quelle est l'importance d'exécuter OpenClaw sur une machine dédiée ?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
    -   [Quelles sont les exigences minimales VPS et l'OS recommandé ?](#what-are-the-minimum-vps-requirements-and-recommended-os)
    -   [Puis-je exécuter OpenClaw dans une VM et quelles sont les exigences](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
-   [Qu'est-ce qu'OpenClaw ?](#what-is-openclaw)
    -   [Qu'est-ce qu'OpenClaw, en un paragraphe ?](#what-is-openclaw-in-one-paragraph)
    -   [Quelle est la proposition de valeur ?](#whats-the-value-proposition)
    -   [Je viens de le configurer, que dois-je faire en premier](#i-just-set-it-up-what-should-i-do-first)
    -   [Quels sont les cinq principaux cas d'utilisation quotidiens pour OpenClaw](#what-are-the-top-five-everyday-use-cases-for-openclaw)
    -   [OpenClaw peut-il aider avec la génération de leads, le démarchage, les publicités et les blogs pour un SaaS](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
    -   [Quels sont les avantages par rapport à Claude Code pour le développement web ?](#what-are-the-advantages-vs-claude-code-for-web-development)
-   [Compétences et automatisation](#skills-and-automation)
    -   [Comment personnaliser les compétences sans salir le dépôt ?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
    -   [Puis-je charger des compétences depuis un dossier personnalisé ?](#can-i-load-skills-from-a-custom-folder)
    -   [Comment utiliser différents modèles pour différentes tâches ?](#how-can-i-use-different-models-for-different-tasks)
    -   [Le bot se fige pendant un travail intensif. Comment décharger cela ?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
    -   [Les cron ou rappels ne se déclenchent pas. Que dois-je vérifier ?](#cron-or-reminders-do-not-fire-what-should-i-check)
    -   [Comment installer des compétences sur Linux ?](#how-do-i-install-skills-on-linux)
    -   [OpenClaw peut-il exécuter des tâches selon un planning ou en continu en arrière-plan ?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
    -   [Puis-je exécuter des compétences exclusives à Apple macOS depuis Linux ?](#can-i-run-apple-macos-only-skills-from-linux)
    -   [Avez-vous une intégration Notion ou HeyGen ?](#do-you-have-a-notion-or-heygen-integration)
    -   [Comment installer l'extension Chrome pour la prise de contrôle du navigateur ?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
-   [Sandboxing et mémoire](#sandboxing-and-memory)
    -   [Existe-t-il une documentation dédiée au sandboxing ?](#is-there-a-dedicated-sandboxing-doc)
    -   [Comment lier un dossier hôte dans le sandbox ?](#how-do-i-bind-a-host-folder-into-the-sandbox)
    -   [Comment fonctionne la mémoire ?](#how-does-memory-work)
    -   [La mémoire oublie constamment des choses. Comment les rendre persistantes ?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
    -   [La mémoire persiste-t-elle pour toujours ? Quelles sont les limites ?](#does-memory-persist-forever-what-are-the-limits)
    -   [La recherche sémantique en mémoire nécessite-t-elle une clé API OpenAI ?](#does-semantic-memory-search-require-an-openai-api-key)
-   [Où les choses vivent sur le disque](#where-things-live-on-disk)
    -   [Toutes les données utilisées avec OpenClaw sont-elles sauvegardées localement ?](#is-all-data-used-with-openclaw-saved-locally)
    -   [Où OpenClaw stocke-t-il ses données ?](#where-does-openclaw-store-its-data)
    -   [Où doivent vivre AGENTS.md / SOUL.md / USER.md / MEMORY.md ?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
    -   [Quelle est la stratégie de sauvegarde recommandée ?](#whats-the-recommended-backup-strategy)
    -   [Comment désinstaller complètement OpenClaw ?](#how-do-i-completely-uninstall-openclaw)
    -   [Les agents peuvent-ils travailler en dehors de l'espace de travail ?](#can-agents-work-outside-the-workspace)
    -   [Je suis en mode distant - où se trouve le magasin de sessions ?](#im-in-remote-mode-where-is-the-session-store)
-   [Bases de la configuration](#config-basics)
    -   [Quel est le format de la configuration ? Où se trouve-t-elle ?](#what-format-is-the-config-where-is-it)
    -   [J'ai défini `gateway.bind: "lan"` (ou `"tailnet"`) et maintenant rien n'écoute / l'interface utilisateur indique non autorisé](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
    -   [Pourquoi ai-je besoin d'un jeton sur localhost maintenant ?](#why-do-i-need-a-token-on-localhost-now)
    -   [Dois-je redémarrer après avoir modifié la configuration ?](#do-i-have-to-restart-after-changing-config)
    -   [Comment désactiver les slogans amusants de l'interface CLI ?](#how-do-i-disable-funny-cli-taglines)
    -   [Comment activer la recherche web (et la récupération web) ?](#how-do-i-enable-web-search-and-web-fetch)
    -   [config.apply a effacé ma configuration. Comment récupérer et éviter cela ?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
    -   [Comment exécuter une Gateway centrale avec des travailleurs spécialisés sur plusieurs appareils ?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
    -   [Le navigateur OpenClaw peut-il fonctionner sans interface graphique ?](#can-the-openclaw-browser-run-headless)
    -   [Comment utiliser Brave pour le contrôle du navigateur ?](#how-do-i-use-brave-for-browser-control)
-   [Gateways distantes et nœuds](#remote-gateways-and-nodes)
    -   [Comment les commandes se propagent-elles entre Telegram, la gateway et les nœuds ?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
    -   [Comment mon agent peut-il accéder à mon ordinateur si la Gateway est hébergée à distance ?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
    -   [Tailscale est connecté mais je ne reçois aucune réponse. Que faire maintenant ?](#tailscale-is-connected-but-i-get-no-replies-what-now)
    -   [Deux instances OpenClaw peuvent-elles communiquer entre elles (local + VPS) ?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
    -   [Ai-je besoin de VPS séparés pour plusieurs agents](#do-i-need-separate-vpses-for-multiple-agents)
    -   [Y a-t-il un avantage à utiliser un nœud sur mon ordinateur portable personnel au lieu de SSH depuis un VPS ?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
    -   [Les nœuds exécutent-ils un service de gateway ?](#do-nodes-run-a-gateway-service)
    -   [Existe-t-il un moyen API / RPC pour appliquer la configuration ?](#is-there-an-api-rpc-way-to-apply-config)
    -   [Quelle est une configuration minimale "saine" pour une première installation ?](#whats-a-minimal-sane-config-for-a-first-install)
    -   [Comment configurer Tailscale sur un VPS et se connecter depuis mon Mac ?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
    -   [Comment connecter un nœud Mac à une Gateway distante (Tailscale Serve) ?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
    -   [Dois-je installer sur un deuxième ordinateur portable ou simplement ajouter un nœud ?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
-   [Variables d'environnement et chargement .env](#env-vars-and-env-loading)
    -   [Comment OpenClaw charge-t-il les variables d'environnement ?](#how-does-openclaw-load-environment-variables)
    -   ["J'ai démarré la Gateway via le service et mes variables d'environnement ont disparu." Que faire maintenant ?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
    -   [J'ai défini `COPILOT_GITHUB_TOKEN`, mais le statut des modèles indique "Shell env: off." Pourquoi ?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
-   [Sessions et discussions multiples](#sessions-and-multiple-chats)
    -   [Comment démarrer une nouvelle conversation ?](#how-do-i-start-a-fresh-conversation)
    -   [Les sessions se réinitialisent-elles automatiquement si je n'envoie jamais `/new` ?](#do-sessions-reset-automatically-if-i-never-send-new)
    -   [Existe-t-il un moyen de créer une équipe d'instances OpenClaw, un PDG et de nombreux agents](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
    -   [Pourquoi le contexte a-t-il été tronqué en plein milieu d'une tâche ? Comment l'empêcher ?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
    -   [Comment réinitialiser complètement OpenClaw mais le garder installé ?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
    -   [Je reçois des erreurs "contexte trop large" - comment réinitialiser ou compacter ?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
    -   [Pourquoi vois-je "LLM request rejected: messages.content.tool\_use.input field required" ?](#why-am-i-seeing-llm-request-rejected-messagescontenttool_useinput-field-required)
    -   [Pourquoi reçois-je des messages de pulsation toutes les 30 minutes ?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
    -   [Dois-je ajouter un "compte bot" à un groupe WhatsApp ?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
    -   [Comment obtenir le JID d'un groupe WhatsApp ?](#how-do-i-get-the-jid-of-a-whatsapp-group)
    -   [Pourquoi OpenClaw ne répond-il pas dans un groupe ?](#why-doesnt-openclaw-reply-in-a-group)
    -   [Les groupes/fils partagent-ils le contexte avec les messages privés ?](#do-groupsthreads-share-context-with-dms)
    -   [Combien d'espaces de travail et d'agents puis-je créer ?](#how-many-workspaces-and-agents-can-i-create)
    -   [Puis-je exécuter plusieurs bots ou discussions en même temps (Slack), et comment devrais-je configurer cela ?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
-   [Modèles : valeurs par défaut, sélection, alias, basculement](#models-defaults-selection-aliases-switching)
    -   [Qu'est-ce que le "modèle par défaut" ?](#what-is-the-default-model)
    -   [Quel modèle recommandez-vous ?](#what-model-do-you-recommend)
    -   [Comment changer de modèle sans effacer ma configuration ?](#how-do-i-switch-models-without-wiping-my-config)
    -   [Puis-je utiliser des modèles auto-hébergés (llama.cpp, vLLM, Ollama) ?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
    -   [Quels modèles utilisent OpenClaw, Flawd et Krill ?](#what-do-openclaw-flawd-and-krill-use-for-models)
    -   [Comment changer de modèle à la volée (sans redémarrer) ?](#how-do-i-switch-models-on-the-fly-without-restarting)
    -   [Puis-je utiliser GPT 5.2 pour les tâches quotidiennes et Codex 5.3 pour le codage](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
    -   [Pourquoi vois-je "Model … is not allowed" et ensuite aucune réponse ?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
    -   [Pourquoi vois-je "Unknown model: minimax/MiniMax-M2.5" ?](#why-do-i-see-unknown-model-minimaxminimaxm25)
    -   [Puis-je utiliser MiniMax comme valeur par défaut et OpenAI pour les tâches complexes ?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
    -   [opus / sonnet / gpt sont-ils des raccourcis intégrés ?](#are-opus-sonnet-gpt-builtin-shortcuts)
    -   [Comment définir/surcharger les raccourcis de modèle (alias) ?](#how-do-i-defineoverride-model-shortcuts-aliases)
    -   [Comment ajouter des modèles d'autres fournisseurs comme OpenRouter ou Z.AI ?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
-   [Basculement de modèle et "Tous les modèles ont échoué"](#model-failover-and-all-models-failed)
    -   [Comment fonctionne le basculement ?](#how-does-failover-work)
    -   [Que signifie cette erreur ?](#what-does-this-error-mean)
    -   [Liste de vérification pour corriger `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
    -   [Pourquoi a-t-il aussi essayé Google Gemini et échoué ?](#why-did-it-also-try-google-gemini-and-fail)
-   [Profils d'authentification : ce qu'ils sont et comment les gérer](#auth-profiles-what-they-are-and-how-to-manage-them)
    -   [Qu'est-ce qu'un profil d'authentification ?](#what-is-an-auth-profile)
    -   [Quels sont les identifiants de profil typiques ?](#what-are-typical-profile-ids)
    -   [Puis-je contrôler quel profil d'authentification est essayé en premier ?](#can-i-control-which-auth-profile-is-tried-first)
    -   [OAuth vs clé API : quelle est la différence ?](#oauth-vs-api-key-whats-the-difference)
-   [Gateway : ports, "déjà en cours d'exécution", et mode distant](#gateway-ports-already-running-and-remote-mode)
    -   [Quel port utilise la Gateway ?](#what-port-does-the-gateway-use)
    -   [Pourquoi `openclaw gateway status` indique-t-il `Runtime: running` mais `RPC probe: failed` ?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
    -   [Pourquoi `openclaw gateway status` affiche-t-il `Config (cli)` et `Config (service)` différents ?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
    -   [Que signifie "another gateway instance is already listening" ?](#what-does-another-gateway-instance-is-already-listening-mean)
    -   [Comment exécuter OpenClaw en mode distant (le client se connecte à une Gateway ailleurs) ?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
    -   [L'interface de contrôle indique "non autorisé" (ou se reconnecte constamment). Que faire maintenant ?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
    -   [J'ai défini `gateway.bind: "tailnet"` mais il ne peut pas se lier / rien n'écoute](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
    -   [Puis-je exécuter plusieurs Gateways sur le même hôte ?](#can-i-run-multiple-gateways-on-the-same-host)
    -   [Que signifie "invalid handshake" / code 1008 ?](#what-does-invalid-handshake-code-1008-mean)
-   [Journalisation et débogage](#logging-and-debugging)
    -   [Où sont les journaux ?](#where-are-logs)
    -   [Comment démarrer/arrêter/redémarrer le service Gateway ?](#how-do-i-startstoprestart-the-gateway-service)
    -   [J'ai fermé mon terminal sur Windows - comment redémarrer OpenClaw ?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
    -   [La Gateway est active mais les réponses n'arrivent jamais. Que dois-je vérifier ?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
    -   ["Déconnecté de la gateway : aucune raison" - que faire maintenant ?](#disconnected-from-gateway-no-reason-what-now)
    -   [Telegram setMyCommands échoue avec des erreurs réseau. Que dois-je vérifier ?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
    -   [L'interface TUI n'affiche aucune sortie. Que dois-je vérifier ?](#tui-shows-no-output-what-should-i-check)
    -   [Comment arrêter complètement puis démarrer la Gateway ?](#how-do-i-completely-stop-then-start-the-gateway)
    -   [ELI5 : `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
    -   [Quel est le moyen le plus rapide d'obtenir plus de détails quand quelque chose échoue ?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
-   [Médias et pièces jointes](#media-and-attachments)
    -   [Ma compétence a généré une image/PDF, mais rien n'a été envoyé](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
-   [Sécurité et contrôle d'accès](#security-and-access-control)
    -   [Est-il sûr d'exposer OpenClaw aux messages privés entrants ?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
    -   [L'injection de prompt n'est-elle qu'un problème pour les bots publics ?](#is-prompt-injection-only-a-concern-for-public-bots)
    -   [Mon bot doit-il avoir son propre e-mail, compte GitHub ou numéro de téléphone](#should-my-bot-have-its-own-email-github-account-or-phone-number)
    -   [Puis-je lui donner de l'autonomie sur mes messages texte et est-ce sûr](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
    -   [Puis-je utiliser des modèles moins chers pour les tâches d'assistant personnel ?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
    -   [J'ai exécuté `/start` dans Telegram mais je n'ai pas reçu de code d'appairage](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
    -   [WhatsApp : va-t-il envoyer des messages à mes contacts ? Comment fonctionne l'appairage ?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
-   [Commandes de discussion, interruption de tâches, et "ça ne s'arrête pas"](#chat-commands-aborting-tasks-and-it-wont-stop)
    -   [Comment empêcher l'affichage des messages système internes dans le chat](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
    -   [Comment arrêter/annuler une tâche en cours d'exécution ?](#how-do-i-stopcancel-a-running-task)
    -   [Comment envoyer un message Discord depuis Telegram ? ("Cross-context messaging denied")](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
    -   [Pourquoi a-t-on l'impression que le bot "ignore" les messages rapides ?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## Premières 60 secondes si quelque chose est cassé

1.  **Statut rapide (première vérification)**
    
    Copier
    
    ```bash
    openclaw status
    ```
    
    Résumé local rapide : OS + mise à jour, accessibilité du service/gateway, agents/sessions, configuration du fournisseur + problèmes d'exécution (quand la gateway est accessible).
2.  **Rapport collable (sûr à partager)**
    
    Copier
    
    ```bash
    openclaw status --all
    ```
    
    Diagnostic en lecture seule avec queue de journal (jetons masqués).
3.  **État du démon + du port**
    
    Copier
    
    ```bash
    openclaw gateway status
    ```
    
    Affiche l'exécution du superviseur vs l'accessibilité RPC, l'URL cible de la sonde, et quelle configuration le service a probablement utilisé.
4.  **Sondes approfondies**
    
    Copier
    
    ```bash
    openclaw status --deep
    ```
    
    Exécute les vérifications de santé de la gateway + les sondes des fournisseurs (nécessite une gateway accessible). Voir [Santé](../gateway/health.md).
5.  **Suivre le dernier journal**
    
    Copier
    
    ```bash
    openclaw logs --follow
    ```
    
    Si le RPC est hors service, revenez à :
    
    Copier
    
    ```bash
    tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
    ```
    
    Les journaux de fichiers sont séparés des journaux du service ; voir [Journalisation](../logging.md) et [Dépannage](../gateway/troubleshooting.md).
6.  **Exécuter le docteur (réparations)**
    
    Copier
    
    ```bash
    openclaw doctor
    ```
    
    Répare/migre la configuration/état + exécute les vérifications de santé. Voir [Docteur](../gateway/doctor.md).
7.  **Instantané de la Gateway**
    
    Copier
    
    ```bash
    openclaw health --json
    openclaw health --verbose   # montre l'URL cible + le chemin de configuration en cas d'erreurs
    ```
    
    Demande à la gateway en cours d'exécution un instantané complet (WS uniquement). Voir [Santé](../gateway/health.md).

## Démarrage rapide et configuration initiale

### Je suis bloqué, quel est le moyen le plus rapide de me débloquer

Utilisez un agent d'IA local qui peut **voir votre machine**. C'est bien plus efficace que de demander sur Discord, car la plupart des cas "Je suis bloqué" sont des **problèmes de configuration ou d'environnement locaux** que les assistants à distance ne peuvent pas inspecter.

-   **Claude Code** : [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
-   **OpenAI Codex** : [https://openai.com/codex/](https://openai.com/codex/)

Ces outils peuvent lire le dépôt, exécuter des commandes, inspecter les journaux et aider à résoudre votre configuration au niveau de la machine (PATH, services, permissions, fichiers d'authentification). Donnez-leur le **dépôt source complet** via l'installation hackable (git) :

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Cela installe OpenClaw **à partir d'un dépôt git**, afin que l'agent puisse lire le code + la documentation et raisonner sur la version exacte que vous exécutez. Vous pouvez toujours revenir à la version stable plus tard en réexécutant l'installateur sans `--install-method git`. Astuce : demandez à l'agent de **planifier et superviser** la réparation (étape par étape), puis exécutez uniquement les commandes nécessaires. Cela garde les modifications petites et plus faciles à auditer. Si vous découvrez un vrai bug ou une correction, veuillez ouvrir un problème GitHub ou envoyer une PR : [https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues) [https://github.com/openclaw/openclaw/pulls](https://github.com/openclaw/openclaw/pulls) Commencez par ces commandes (partagez les sorties lorsque vous demandez de l'aide) :

```bash
openclaw status
openclaw models status
openclaw doctor
```

Ce qu'elles font :

-   `openclaw status` : instantané rapide de la santé de la gateway/agent + configuration de base.
-   `openclaw models status` : vérifie l'authentification du fournisseur + la disponibilité des modèles.
-   `openclaw doctor` : valide et répare les problèmes courants de configuration/état.

Autres vérifications CLI utiles : `openclaw status --all`, `openclaw logs --follow`, `openclaw gateway status`, `openclaw health --verbose`. Boucle de débogage rapide : [Premières 60 secondes si quelque chose est cassé](#first-