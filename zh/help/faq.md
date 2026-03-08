

  帮助

  
# 常见问题

针对实际设置（本地开发、VPS、多智能体、OAuth/API 密钥、模型故障转移）的快速答案和深度故障排除。关于运行时诊断，请参阅 [故障排除](../gateway/troubleshooting.md)。关于完整的配置参考，请参阅 [配置](../gateway/configuration.md)。

## 目录

-   \[快速入门和首次运行设置\]
    -   [我被卡住了，最快摆脱困境的方法是什么？](#im-stuck-whats-the-fastest-way-to-get-unstuck)
    -   [推荐安装和设置 OpenClaw 的方式是什么？](#whats-the-recommended-way-to-install-and-set-up-openclaw)
    -   [完成引导后如何打开仪表板？](#how-do-i-open-the-dashboard-after-onboarding)
    -   [如何在本地主机与远程环境下验证仪表板（令牌）？](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
    -   [我需要什么运行时？](#what-runtime-do-i-need)
    -   [它能在树莓派上运行吗？](#does-it-run-on-raspberry-pi)
    -   [树莓派安装有什么技巧吗？](#any-tips-for-raspberry-pi-installs)
    -   [它卡在“醒来吧，我的朋友”/引导无法孵化。现在怎么办？](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
    -   [我能将我的设置迁移到新机器（Mac mini）而无需重新引导吗？](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
    -   [我在哪里能看到最新版本有什么新内容？](#where-do-i-see-what-is-new-in-the-latest-version)
    -   [我无法访问 docs.openclaw.ai（SSL 错误）。现在怎么办？](#i-cant-access-docsopenclawai-ssl-error-what-now)
    -   [稳定版和测试版有什么区别？](#whats-the-difference-between-stable-and-beta)
    -   [如何安装测试版，测试版和开发版有什么区别？](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
    -   [如何尝试最新的构建？](#how-do-i-try-the-latest-bits)
    -   [安装和引导通常需要多长时间？](#how-long-does-install-and-onboarding-usually-take)
    -   [安装程序卡住了？如何获取更多反馈？](#installer-stuck-how-do-i-get-more-feedback)
    -   [Windows 安装提示 git 未找到或 openclaw 无法识别](#windows-install-says-git-not-found-or-openclaw-not-recognized)
    -   [Windows exec 输出显示乱码中文文本，我该怎么办](#windows-exec-output-shows-garbled-chinese-text-what-should-i-do)
    -   [文档没有回答我的问题 - 如何获得更好的答案？](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
    -   [如何在 Linux 上安装 OpenClaw？](#how-do-i-install-openclaw-on-linux)
    -   [如何在 VPS 上安装 OpenClaw？](#how-do-i-install-openclaw-on-a-vps)
    -   [云/VPS 安装指南在哪里？](#where-are-the-cloudvps-install-guides)
    -   [我能让 OpenClaw 自己更新吗？](#can-i-ask-openclaw-to-update-itself)
    -   [引导向导实际上做了什么？](#what-does-the-onboarding-wizard-actually-do)
    -   [运行这个需要 Claude 或 OpenAI 订阅吗？](#do-i-need-a-claude-or-openai-subscription-to-run-this)
    -   [我可以在没有 API 密钥的情况下使用 Claude Max 订阅吗](#can-i-use-claude-max-subscription-without-an-api-key)
    -   [Anthropic 的“setup-token”认证是如何工作的？](#how-does-anthropic-setuptoken-auth-work)
    -   [我在哪里可以找到 Anthropic 的 setup-token？](#where-do-i-find-an-anthropic-setuptoken)
    -   [你们支持 Claude 订阅认证（Claude Pro 或 Max）吗？](#do-you-support-claude-subscription-auth-claude-pro-or-max)
    -   [为什么我看到来自 Anthropic 的 `HTTP 429: rate_limit_error`？](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
    -   [支持 AWS Bedrock 吗？](#is-aws-bedrock-supported)
    -   [Codex 认证是如何工作的？](#how-does-codex-auth-work)
    -   [你们支持 OpenAI 订阅认证（Codex OAuth）吗？](#do-you-support-openai-subscription-auth-codex-oauth)
    -   [如何设置 Gemini CLI OAuth](#how-do-i-set-up-gemini-cli-oauth)
    -   [本地模型适合日常聊天吗？](#is-a-local-model-ok-for-casual-chats)
    -   [如何将托管模型流量保持在特定区域？](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
    -   [我必须买一台 Mac Mini 来安装这个吗？](#do-i-have-to-buy-a-mac-mini-to-install-this)
    -   [我需要 Mac mini 来支持 iMessage 吗？](#do-i-need-a-mac-mini-for-imessage-support)
    -   [如果我买一台 Mac mini 来运行 OpenClaw，能把它连接到我的 MacBook Pro 吗？](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
    -   [我能用 Bun 吗？](#can-i-use-bun)
    -   [Telegram：`allowFrom` 里填什么？](#telegram-what-goes-in-allowfrom)
    -   [多个人能用一个 WhatsApp 号码配合不同的 OpenClaw 实例吗？](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
    -   [我能运行一个“快速聊天”智能体和一个“用 Opus 编程”的智能体吗？](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
    -   [Homebrew 能在 Linux 上工作吗？](#does-homebrew-work-on-linux)
    -   [可破解（git）安装和 npm 安装有什么区别？](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
    -   [我以后能在 npm 和 git 安装之间切换吗？](#can-i-switch-between-npm-and-git-installs-later)
    -   [我应该在我的笔记本电脑上运行网关还是在 VPS 上运行？](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
    -   [在专用机器上运行 OpenClaw 有多重要？](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
    -   [VPS 的最低要求和推荐操作系统是什么？](#what-are-the-minimum-vps-requirements-and-recommended-os)
    -   [我能在虚拟机里运行 OpenClaw 吗，有什么要求](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
-   [什么是 OpenClaw？](#what-is-openclaw)
    -   [用一段话描述，什么是 OpenClaw？](#what-is-openclaw-in-one-paragraph)
    -   [它的价值主张是什么？](#whats-the-value-proposition)
    -   [我刚设置好，应该先做什么](#i-just-set-it-up-what-should-i-do-first)
    -   [OpenClaw 的五大日常用例是什么](#what-are-the-top-five-everyday-use-cases-for-openclaw)
    -   [OpenClaw 能帮助 SaaS 进行潜在客户生成、外联、广告和博客吗？](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
    -   [与 Claude Code 相比，在 Web 开发方面有什么优势？](#what-are-the-advantages-vs-claude-code-for-web-development)
-   [技能和自动化](#skills-and-automation)
    -   [如何在不弄脏仓库的情况下自定义技能？](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
    -   [我能从自定义文件夹加载技能吗？](#can-i-load-skills-from-a-custom-folder)
    -   [如何为不同的任务使用不同的模型？](#how-can-i-use-different-models-for-different-tasks)
    -   [机器人在执行繁重工作时卡住了。如何卸载？](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
    -   [Cron 或提醒不触发。我应该检查什么？](#cron-or-reminders-do-not-fire-what-should-i-check)
    -   [如何在 Linux 上安装技能？](#how-do-i-install-skills-on-linux)
    -   [OpenClaw 能按计划或在后台持续运行任务吗？](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
    -   [我能从 Linux 运行仅限 Apple macOS 的技能吗？](#can-i-run-apple-macos-only-skills-from-linux)
    -   [你们有 Notion 或 HeyGen 集成吗？](#do-you-have-a-notion-or-heygen-integration)
    -   [如何安装用于浏览器接管的 Chrome 扩展？](#how-do-i-install-the-chrome-extension-for-browser-takeover)
-   [沙盒化和内存](#sandboxing-and-memory)
    -   [有专门的沙盒化文档吗？](#is-there-a-dedicated-sandboxing-doc)
    -   [如何将主机文件夹绑定到沙盒中？](#how-do-i-bind-a-host-folder-into-the-sandbox)
    -   [内存是如何工作的？](#how-does-memory-work)
    -   [内存老是忘记东西。如何让它记住？](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
    -   [内存会永久保存吗？有什么限制？](#does-memory-persist-forever-what-are-the-limits)
    -   [语义内存搜索需要 OpenAI API 密钥吗？](#does-semantic-memory-search-require-an-openai-api-key)
-   [文件在磁盘上的位置](#where-things-live-on-disk)
    -   [OpenClaw 使用的所有数据都保存在本地吗？](#is-all-data-used-with-openclaw-saved-locally)
    -   [OpenClaw 将其数据存储在哪里？](#where-does-openclaw-store-its-data)
    -   [AGENTS.md / SOUL.md / USER.md / MEMORY.md 应该放在哪里？](#where-should-agentsmd-soulmd-usermd-memorymd-live)
    -   [推荐的备份策略是什么？](#whats-the-recommended-backup-strategy)
    -   [如何完全卸载 OpenClaw？](#how-do-i-completely-uninstall-openclaw)
    -   [智能体能在工作区外工作吗？](#can-agents-work-outside-the-workspace)
    -   [我处于远程模式 - 会话存储在哪里？](#im-in-remote-mode-where-is-the-session-store)
-   [配置基础](#config-basics)
    -   [配置是什么格式？在哪里？](#what-format-is-the-config-where-is-it)
    -   [我设置了 `gateway.bind: "lan"`（或 `"tailnet"`），现在没有监听 / UI 显示未授权](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
    -   [为什么我现在在本地主机上需要令牌？](#why-do-i-need-a-token-on-localhost-now)
    -   [更改配置后必须重启吗？](#do-i-have-to-restart-after-changing-config)
    -   [如何禁用有趣的 CLI 标语？](#how-do-i-disable-funny-cli-taglines)
    -   [如何启用网络搜索（和网页抓取）？](#how-do-i-enable-web-search-and-web-fetch)
    -   [config.apply 清除了我的配置。如何恢复并避免这种情况？](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
    -   [如何运行一个中心网关，并在多台设备上部署专门的 worker？](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
    -   [OpenClaw 浏览器能无头运行吗？](#can-the-openclaw-browser-run-headless)
    -   [如何使用 Brave 进行浏览器控制？](#how-do-i-use-brave-for-browser-control)
-   [远程网关和节点](#remote-gateways-and-nodes)
    -   [命令如何在 Telegram、网关和节点之间传播？](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
    -   [如果网关托管在远程，我的智能体如何访问我的电脑？](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-removely)
    -   [Tailscale 已连接但我收不到回复。现在怎么办？](#tailscale-is-connected-but-i-get-no-replies-what-now)
    -   [两个 OpenClaw 实例能相互通信吗（本地 + VPS）？](#can-two-openclaw-instances-talk-to-each-other-local-vps)
    -   [多个智能体需要单独的 VPS 吗](#do-i-need-separate-vpses-for-multiple-agents)
    -   [在我的个人笔记本电脑上使用节点而不是从 VPS SSH 有什么好处？](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
    -   [节点运行网关服务吗？](#do-nodes-run-a-gateway-service)
    -   [有 API / RPC 方式来应用配置吗？](#is-there-an-api-rpc-way-to-apply-config)
    -   [首次安装的“合理”最小配置是什么？](#whats-a-minimal-sane-config-for-a-first-install)
    -   [如何在 VPS 上设置 Tailscale 并从我的 Mac 连接？](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
    -   [如何将 Mac 节点连接到远程网关（Tailscale Serve）？](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
    -   [我应该安装在第二台笔记本电脑上还是只添加一个节点？](#should-i-install-on-a-second-laptop-or-just-add-a-node)
-   [环境变量和 .env 加载](#env-vars-and-env-loading)
    -   [OpenClaw 如何加载环境变量？](#how-does-openclaw-load-environment-variables)
    -   [“我通过服务启动了网关，我的环境变量消失了。”现在怎么办？](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
    -   [我设置了 `COPILOT_GITHUB_TOKEN`，但模型状态显示“Shell env: off。”为什么？](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
-   [会话和多个聊天](#sessions-and-multiple-chats)
    -   [如何开始一个新的对话？](#how-do-i-start-a-fresh-conversation)
    -   [如果我从不发送 `/new`，会话会自动重置吗？](#do-sessions-reset-automatically-if-i-never-send-new)
    -   [有没有办法组建一个 OpenClaw 实例团队，一个 CEO 和多个智能体](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
    -   [为什么上下文在任务中途被截断？如何防止？](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
    -   [如何完全重置 OpenClaw 但保留安装？](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
    -   [我收到“上下文太大”错误 - 如何重置或压缩？](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
    -   [为什么我看到“LLM request rejected: messages.content.tool\_use.input field required”？](#why-am-i-seeing-llm-request-rejected-messagescontenttool_useinput-field-required)
    -   [为什么我每 30 分钟收到一次心跳消息？](#why-am-i-getting-heartbeat-messages-every-30-minutes)
    -   [我需要将“机器人账户”添加到 WhatsApp 群组吗？](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
    -   [如何获取 WhatsApp 群组的 JID？](#how-do-i-get-the-jid-of-a-whatsapp-group)
    -   [为什么 OpenClaw 不在群里回复？](#why-doesnt-openclaw-reply-in-a-group)
    -   [群组/线程与私聊共享上下文吗？](#do-groupsthreads-share-context-with-dms)
    -   [我能创建多少个工作区和智能体？](#how-many-workspaces-and-agents-can-i-create)
    -   [我能同时运行多个机器人或聊天吗（Slack），应该如何设置？](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
-   [模型：默认值、选择、别名、切换](#models-defaults-selection-aliases-switching)
    -   [什么是“默认模型”？](#what-is-the-default-model)
    -   [你推荐什么模型？](#what-model-do-you-recommend)
    -   [如何在不清除配置的情况下切换模型？](#how-do-i-switch-models-without-wiping-my-config)
    -   [我能使用自托管模型（llama.cpp, vLLM, Ollama）吗？](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
    -   [OpenClaw、Flawd 和 Krill 使用什么模型？](#what-do-openclaw-flawd-and-krill-use-for-models)
    -   [如何动态切换模型（无需重启）？](#how-do-i-switch-models-on-the-fly-without-restarting)
    -   [我能用 GPT 5.2 处理日常任务，用 Codex 5.3 编程吗](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
    -   [为什么我看到“Model … is not allowed”然后没有回复？](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
    -   [为什么我看到“Unknown model: minimax/MiniMax-M2.5”？](#why-do-i-see-unknown-model-minimaxminimaxm25)
    -   [我能用 MiniMax 作为默认，用 OpenAI 处理复杂任务吗？](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
    -   [opus / sonnet / gpt 是内置快捷方式吗？](#are-opus-sonnet-gpt-builtin-shortcuts)
    -   [如何定义/覆盖模型快捷方式（别名）？](#how-do-i-defineoverride-model-shortcuts-aliases)
    -   [如何添加来自其他提供商（如 OpenRouter 或 Z.AI）的模型？](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
-   [模型故障转移和“所有模型都失败了”](#model-failover-and-all-models-failed)
    -   [故障转移是如何工作的？](#how-does-failover-work)
    -   [这个错误是什么意思？](#what-does-this-error-mean)
    -   [修复 `No credentials found for profile "anthropic:default"` 的检查清单](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
    -   [为什么它也尝试了 Google Gemini 并失败了？](#why-did-it-also-try-google-gemini-and-fail)
-   [认证配置文件：它们是什么以及如何管理](#auth-profiles-what-they-are-and-how-to-manage-them)
    -   [什么是认证配置文件？](#what-is-an-auth-profile)
    -   [典型的配置文件 ID 是什么？](#what-are-typical-profile-ids)
    -   [我能控制哪个认证配置文件被首先尝试吗？](#can-i-control-which-auth-profile-is-tried-first)
    -   [OAuth 与 API 密钥：有什么区别？](#oauth-vs-api-key-whats-the-difference)
-   [网关：端口、“已在运行”和远程模式](#gateway-ports-already-running-and-remote-mode)
    -   [网关使用什么端口？](#what-port-does-the-gateway-use)
    -   [为什么 `openclaw gateway status` 显示 `Runtime: running` 但 `RPC probe: failed`？](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
    -   [为什么 `openclaw gateway status` 显示 `Config (cli)` 和 `Config (service)` 不同？](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
    -   [“另一个网关实例已在监听”是什么意思？](#what-does-another-gateway-instance-is-already-listening-mean)
    -   [如何在远程模式下运行 OpenClaw（客户端连接到其他地方的网关）？](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
    -   [控制 UI 显示“未授权”（或不断重连）。现在怎么办？](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
    -   [我设置了 `gateway.bind: "tailnet"` 但它无法绑定 / 没有监听](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
    -   [我能在同一台主机上运行多个网关吗？](#can-i-run-multiple-gateways-on-the-same-host)
    -   [“invalid handshake” / code 1008 是什么意思？](#what-does-invalid-handshake-code-1008-mean)
-   [日志记录和调试](#logging-and-debugging)
    -   [日志在哪里？](#where-are-logs)
    -   [如何启动/停止/重启网关服务？](#how-do-i-startstoprestart-the-gateway-service)
    -   [我在 Windows 上关闭了终端 - 如何重启 OpenClaw？](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
    -   [网关已启动但回复从未到达。我应该检查什么？](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
    -   [“Disconnected from gateway: no reason” - 现在怎么办？](#disconnected-from-gateway-no-reason-what-now)
    -   [Telegram setMyCommands 因网络错误失败。我应该检查什么？](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
    -   [TUI 没有输出。我应该检查什么？](#tui-shows-no-output-what-should-i-check)
    -   [如何完全停止然后启动网关？](#how-do-i-completely-stop-then-start-the-gateway)
    -   [通俗解释：`openclaw gateway restart` 与 `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
    -   [当某些东西失败时，获取更多细节的最快方法是什么？](#whats-the-fastest-way-to-get-more-details-when-something-fails)
-   [媒体和附件](#media-and-attachments)
    -   [我的技能生成了图片/PDF，但什么都没发送](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
-   [安全和访问控制](#security-and-access-control)
    -   [将 OpenClaw 暴露给入站私聊安全吗？](#is-it-safe-to-expose-openclaw-to-inbound-dms)
    -   [提示注入只是公开机器人的问题吗？](#is-prompt-injection-only-a-concern-for-public-bots)
    -   [我的机器人应该有自己的电子邮件 GitHub 账户或电话号码吗](#should-my-bot-have-its-own-email-github-account-or-phone-number)
    -   [我能让它自主处理我的短信吗，这安全吗](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
    -   [我能用更便宜的模型处理个人助理任务吗？](#can-i-use-cheaper-models-for-personal-assistant-tasks)
    -   [我在 Telegram 中运行了 `/start` 但没有收到配对码](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
    -   [WhatsApp：它会给我的联系人发消息吗？配对是如何工作的？](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
-   [聊天命令、中止任务和“它停不下来”](#chat-commands-aborting-tasks-and-it-wont-stop)
    -   [如何阻止内部系统消息显示在聊天中？](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
    -   [如何停止/取消正在运行的任务？](#how-do-i-stopcancel-a-running-task)
    -   [如何从 Telegram 发送 Discord 消息？（“跨上下文消息发送被拒绝”）](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
    -   [为什么感觉机器人“忽略”了快速连续的消息？](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## 如果出现问题，前 60 秒

1.  **快速状态（首先检查）**
    
    复制
    
    ```bash
    openclaw status
    ```
    
    快速本地摘要：操作系统 + 更新、网关/服务可达性、智能体/会话、提供商配置 + 运行时问题（当网关可达时）。
2.  **可粘贴的报告（可安全分享）**
    
    复制
    
    ```bash
    openclaw status --all
    ```
    
    只读诊断，包含日志尾部（令牌已脱敏）。
3.  **守护进程 + 端口状态**
    
    复制
    
    ```bash
    openclaw gateway status
    ```
    
    显示监督器运行时与 RPC 可达性、探测目标 URL，以及服务可能使用的配置。
4.  **深度探测**
    
    复制
    
    ```bash
    openclaw status --deep
    ```
    
    运行网关健康检查 + 提供商探测（需要可达的网关）。参见 [健康状态](../gateway/health.md)。
5.  **跟踪最新日志**
    
    复制
    
    ```bash
    openclaw logs --follow
    ```
    
    如果 RPC 宕机，回退到：
    
    复制
    
    ```bash
    tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
    ```
    
    文件日志与服务日志是分开的；参见 [日志记录](../logging.md) 和 [故障排除](../gateway/troubleshooting.md)。
6.  **运行医生（修复）**
    
    复制
    
    ```bash
    openclaw doctor
    ```
    
    修复/迁移配置/状态 + 运行健康检查。参见 [医生](../gateway/doctor.md)。
7.  **网关快照**
    
    复制
    
    ```bash
    openclaw health --json
    openclaw health --verbose   # 显示目标 URL + 错误时的配置路径
    ```
    
    向正在运行的网关请求完整快照（仅限 WebSocket）。参见 [健康状态](../gateway/health.md)。

## 快速入门和首次运行设置

### 我被卡住了，最快摆脱困境的方法是什么

使用一个能**看到你机器**的本地 AI 智能体。这比在 Discord 上提问有效得多，因为大多数“我被卡住了”的情况是**本地配置或环境问题**，远程助手无法检查。

-   **Claude Code**: [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
-   **OpenAI Codex**: [https://openai.com/codex/](https://openai.com/codex/)

这些工具可以读取仓库、运行命令、检查日志，并帮助修复机器级别的设置（PATH、服务、权限、认证文件）。通过可破解（git）安装方式，给它们**完整的源代码检出**：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

这会从 git 检出安装 OpenClaw，这样智能体就能读取代码 + 文档，并推理你正在运行的确切版本。你以后随时可以通过重新运行不带 `--install-method git` 的安装程序切换回稳定版。提示：让智能体**规划并监督**修复（逐步进行），然后只执行必要的命令。这样可以使更改更小，更容易审核。如果你发现了真正的错误或修复，请提交 GitHub issue 或发送 PR：[https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues) [https://github.com/openclaw/openclaw/pulls](https://github.com/openclaw/openclaw/pulls) 从这些命令开始（寻求帮助时分享输出）：

```bash
openclaw status
openclaw models status
openclaw doctor
```

它们的作用：

-   `openclaw status`：网关/智能体健康状态 + 基本配置的快速快照。
-   `openclaw models status`：检查提供商认证 + 模型可用性。
-   `openclaw doctor`：验证并修复常见的配置/状态问题。

其他有用的 CLI 检查：`openclaw status --all`, `openclaw logs --follow`, `openclaw gateway status`, `openclaw health --verbose`。快速调试循环：[如果出现问题，前 60 秒](#first-60-seconds-if-somethings-broken)。安装文档：[安装](../install.md), [安装程序标志](../install/installer.md), [更新](../install/updating.md)。

### 推荐安装和设置 OpenClaw 的方式是什么

仓库推荐从源代码运行并使用引导向导：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

向导也可以自动构建 UI 资源。引导完成后，你通常在端口 **18789** 上运行网关。从源代码（贡献者/开发者）：

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # 首次运行时自动安装 UI 依赖
openclaw onboard
```

如果你还没有全局安装，可以通过 `pnpm openclaw onboard` 运行。

### 完成引导后如何打开仪表板

向导会在引导后立即用干净的（非令牌化的）仪表板 URL 打开你的浏览器，并在摘要中打印链接。保持该标签页打开；如果它没有启动，在同一台机器上复制/粘贴打印的 URL。

### 如何在本地主机与远程环境下验证仪表板（令牌）

**本地主机（同一台机器）：**

-   打开 `http://127.0.0.1:18789/`。
-   如果要求认证，将 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）中的令牌粘贴到控制 UI 设置中。
-   从网关主机检索：`openclaw config get gateway.auth.token`（或生成一个：`openclaw doctor --generate-gateway-token`）。

**不在本地主机上：**

-   **Tailscale Serve**（推荐）：保持绑定环回，运行 `openclaw gateway --tailscale serve`，打开 `https:///`。如果 `gateway.auth.allowTailscale` 是 `true`，身份标头满足控制 UI/WebSocket 认证（无需令牌，假设网关主机可信）；HTTP API 仍需要令牌/密码。
-   **Tailnet 绑定**：运行 `openclaw gateway --bind tailnet --token ""`，打开 `http://<tailscale-ip>:18789/`，在仪表板设置中粘贴令牌。
-   **SSH 隧道**：`ssh -N -L 18789:127.0.0.1:18789 user@host` 然后打开 `http://127.0.0.1:18789/` 并在控制 UI 设置中粘贴令牌。

参见 [仪表板](../web/dashboard.md) 和 [Web 界面](../web.md) 了解绑定模式和认证详情。

### 我需要什么运行时

需要 Node **\>= 22**。推荐使用 `pnpm`。**不推荐**将 Bun 用于网关。

### 它能在树莓派上运行吗

是的。网关是轻量级的 - 文档列出 **512MB-1GB 内存**、**1 核**和大约 **500MB** 磁盘空间就足够个人使用，并指出**树莓派 4 可以运行它**。如果你想要额外的余量（日志、媒体、其他服务），**推荐 2GB**，但这不是硬性最低要求。提示：一个小型 Pi/VPS 可以托管网关，你可以在你的笔记本电脑/手机上配对**节点**以访问本地屏幕/摄像头/画布或执行命令。参见 [节点](../nodes.md)。

### 树莓派安装有什么技巧吗

简短版本：它能工作，但预计会有粗糙的边缘。

-   使用**64 位**操作系统并保持 Node >= 22。
-   优先使用**可破解（git）安装**，以便查看日志和快速更新。
-   开始时不要启用频道/技能，然后逐个添加。
-   如果遇到奇怪的二进制问题，通常是**ARM 兼容性**问题。

文档：[Linux](../platforms/linux.md), [安装](../install.md)。

### 它卡在“醒来吧，我的朋友”/引导无法孵化。现在怎么办

该屏幕依赖于网关可达且已认证。TUI 也会在首次孵化时自动发送“Wake up, my friend!”。如果你看到那一行**没有回复**且令牌保持为 0，说明智能体从未运行。

1.  重启网关：

```bash
openclaw gateway restart
```

2.  检查状态 + 认证：

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3.  如果仍然挂起，运行：

```bash
openclaw doctor
```

如果网关是远程的，请确保隧道/Tailscale 连接正常，并且 UI 指向正确的网关。参见 [远程访问](../gateway/remote.md)。

### 我能将我的设置迁移到新机器（Mac mini）而无需重新引导吗

是的。复制**状态目录**和**工作区**，然后运行一次 Doctor。这能让你的机器人“完全一样”（内存、会话历史、认证、频道状态），只要你复制**两个**位置：

1.  在新机器上安装 OpenClaw。
2.  从旧机器复制 `$OPENCLAW_STATE_DIR`（默认：`~/.openclaw`）。
3.  复制你的工作区（默认：`~/.openclaw/workspace`）。
4.  运行 `openclaw doctor` 并重启网关服务。

这将保留配置、认证配置文件、WhatsApp 凭据、会话和内存。如果你处于远程模式，请记住网关主机拥有会话存储和工作区。**重要：**如果你只将工作区提交/推送到 GitHub，你备份的是**内存 + 引导文件**，但**不是**会话