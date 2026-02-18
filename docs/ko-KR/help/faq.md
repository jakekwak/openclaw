---
summary: "OpenClaw 설정, 구성 및 사용에 관한 자주 묻는 질문"
title: "FAQ"
---

# FAQ

실제 환경 설정(로컬 개발, VPS, 멀티 에이전트, OAuth/API 키, 모델 페일오버)에 대한 빠른 답변과 심층 문제 해결입니다. 런타임 진단은 [문제 해결](/ko-KR/gateway/troubleshooting)을, 전체 구성 레퍼런스는 [구성](/ko-KR/gateway/configuration)을 참조하세요.

## 목차

- [Quick start and first-run setup]
  - [Im stuck whats the fastest way to get unstuck?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [What's the recommended way to install and set up OpenClaw?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [How do I open the dashboard after onboarding?](#how-do-i-open-the-dashboard-after-onboarding)
  - [How do I authenticate the dashboard (token) on localhost vs remote?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [What runtime do I need?](#what-runtime-do-i-need)
  - [Does it run on Raspberry Pi?](#does-it-run-on-raspberry-pi)
  - [Any tips for Raspberry Pi installs?](#any-tips-for-raspberry-pi-installs)
  - [It is stuck on "wake up my friend" / onboarding will not hatch. What now?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [Can I migrate my setup to a new machine (Mac mini) without redoing onboarding?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [Where do I see what is new in the latest version?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [I can't access docs.openclaw.ai (SSL error). What now?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [What's the difference between stable and beta?](#whats-the-difference-between-stable-and-beta)
  - [How do I install the beta version, and what's the difference between beta and dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [How do I try the latest bits?](#how-do-i-try-the-latest-bits)
  - [How long does install and onboarding usually take?](#how-long-does-install-and-onboarding-usually-take)
  - [Installer stuck? How do I get more feedback?](#installer-stuck-how-do-i-get-more-feedback)
  - [Windows install says git not found or openclaw not recognized](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [The docs didn't answer my question - how do I get a better answer?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [How do I install OpenClaw on Linux?](#how-do-i-install-openclaw-on-linux)
  - [How do I install OpenClaw on a VPS?](#how-do-i-install-openclaw-on-a-vps)
  - [Where are the cloud/VPS install guides?](#where-are-the-cloudvps-install-guides)
  - [Can I ask OpenClaw to update itself?](#can-i-ask-openclaw-to-update-itself)
  - [What does the onboarding wizard actually do?](#what-does-the-onboarding-wizard-actually-do)
  - [Do I need a Claude or OpenAI subscription to run this?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [Can I use Claude Max subscription without an API key](#can-i-use-claude-max-subscription-without-an-api-key)
  - [How does Anthropic "setup-token" auth work?](#how-does-anthropic-setuptoken-auth-work)
  - [Where do I find an Anthropic setup-token?](#where-do-i-find-an-anthropic-setuptoken)
  - [Do you support Claude subscription auth (Claude Pro or Max)?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Why am I seeing `HTTP 429: rate_limit_error` from Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [Is AWS Bedrock supported?](#is-aws-bedrock-supported)
  - [How does Codex auth work?](#how-does-codex-auth-work)
  - [Do you support OpenAI subscription auth (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
  - [How do I set up Gemini CLI OAuth](#how-do-i-set-up-gemini-cli-oauth)
  - [Is a local model OK for casual chats?](#is-a-local-model-ok-for-casual-chats)
  - [How do I keep hosted model traffic in a specific region?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [Do I have to buy a Mac Mini to install this?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [Do I need a Mac mini for iMessage support?](#do-i-need-a-mac-mini-for-imessage-support)
  - [If I buy a Mac mini to run OpenClaw, can I connect it to my MacBook Pro?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [Can I use Bun?](#can-i-use-bun)
  - [Telegram: what goes in `allowFrom`?](#telegram-what-goes-in-allowfrom)
  - [Can multiple people use one WhatsApp number with different OpenClaw instances?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - [Can I run a "fast chat" agent and an "Opus for coding" agent?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Does Homebrew work on Linux?](#does-homebrew-work-on-linux)
  - [What's the difference between the hackable (git) install and npm install?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [Can I switch between npm and git installs later?](#can-i-switch-between-npm-and-git-installs-later)
  - [Should I run the Gateway on my laptop or a VPS?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [How important is it to run OpenClaw on a dedicated machine?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [What are the minimum VPS requirements and recommended OS?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [Can I run OpenClaw in a VM and what are the requirements](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [What is OpenClaw?](#what-is-openclaw)
  - [What is OpenClaw, in one paragraph?](#what-is-openclaw-in-one-paragraph)
  - [What's the value proposition?](#whats-the-value-proposition)
  - [I just set it up what should I do first](#i-just-set-it-up-what-should-i-do-first)
  - [What are the top five everyday use cases for OpenClaw](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  - [Can OpenClaw help with lead gen outreach ads and blogs for a SaaS](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [What are the advantages vs Claude Code for web development?](#what-are-the-advantages-vs-claude-code-for-web-development)
- [Skills and automation](#skills-and-automation)
  - [How do I customize skills without keeping the repo dirty?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  - [Can I load skills from a custom folder?](#can-i-load-skills-from-a-custom-folder)
  - [How can I use different models for different tasks?](#how-can-i-use-different-models-for-different-tasks)
  - [The bot freezes while doing heavy work. How do I offload that?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  - [Cron or reminders do not fire. What should I check?](#cron-or-reminders-do-not-fire-what-should-i-check)
  - [How do I install skills on Linux?](#how-do-i-install-skills-on-linux)
  - [Can OpenClaw run tasks on a schedule or continuously in the background?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  - [Can I run Apple macOS-only skills from Linux?](#can-i-run-apple-macos-only-skills-from-linux)
  - [Do you have a Notion or HeyGen integration?](#do-you-have-a-notion-or-heygen-integration)
  - [How do I install the Chrome extension for browser takeover?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
- [Sandboxing and memory](#sandboxing-and-memory)
  - [Is there a dedicated sandboxing doc?](#is-there-a-dedicated-sandboxing-doc)
  - [How do I bind a host folder into the sandbox?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  - [How does memory work?](#how-does-memory-work)
  - [Memory keeps forgetting things. How do I make it stick?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  - [Does memory persist forever? What are the limits?](#does-memory-persist-forever-what-are-the-limits)
  - [Does semantic memory search require an OpenAI API key?](#does-semantic-memory-search-require-an-openai-api-key)
- [Where things live on disk](#where-things-live-on-disk)
  - [Is all data used with OpenClaw saved locally?](#is-all-data-used-with-openclaw-saved-locally)
  - [Where does OpenClaw store its data?](#where-does-openclaw-store-its-data)
  - [Where should AGENTS.md / SOUL.md / USER.md / MEMORY.md live?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  - [What's the recommended backup strategy?](#whats-the-recommended-backup-strategy)
  - [How do I completely uninstall OpenClaw?](#how-do-i-completely-uninstall-openclaw)
  - [Can agents work outside the workspace?](#can-agents-work-outside-the-workspace)
  - [I'm in remote mode - where is the session store?](#im-in-remote-mode-where-is-the-session-store)
- [Config basics](#config-basics)
  - [What format is the config? Where is it?](#what-format-is-the-config-where-is-it)
  - [I set `gateway.bind: "lan"` (or `"tailnet"`) and now nothing listens / the UI says unauthorized](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  - [Why do I need a token on localhost now?](#why-do-i-need-a-token-on-localhost-now)
  - [Do I have to restart after changing config?](#do-i-have-to-restart-after-changing-config)
  - [How do I enable web search (and web fetch)?](#how-do-i-enable-web-search-and-web-fetch)
  - [config.apply wiped my config. How do I recover and avoid this?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  - [How do I run a central Gateway with specialized workers across devices?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  - [Can the OpenClaw browser run headless?](#can-the-openclaw-browser-run-headless)
  - [How do I use Brave for browser control?](#how-do-i-use-brave-for-browser-control)
- [Remote gateways and nodes](#remote-gateways-and-nodes)
  - [How do commands propagate between Telegram, the gateway, and nodes?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  - [How can my agent access my computer if the Gateway is hosted remotely?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  - [Tailscale is connected but I get no replies. What now?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  - [Can two OpenClaw instances talk to each other (local + VPS)?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  - [Do I need separate VPSes for multiple agents](#do-i-need-separate-vpses-for-multiple-agents)
  - [Is there a benefit to using a node on my personal laptop instead of SSH from a VPS?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  - [Do nodes run a gateway service?](#do-nodes-run-a-gateway-service)
  - [Is there an API / RPC way to apply config?](#is-there-an-api-rpc-way-to-apply-config)
  - [What's a minimal "sane" config for a first install?](#whats-a-minimal-sane-config-for-a-first-install)
  - [How do I set up Tailscale on a VPS and connect from my Mac?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  - [How do I connect a Mac node to a remote Gateway (Tailscale Serve)?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  - [Should I install on a second laptop or just add a node?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
- [Env vars and .env loading](#env-vars-and-env-loading)
  - [How does OpenClaw load environment variables?](#how-does-openclaw-load-environment-variables)
  - ["I started the Gateway via the service and my env vars disappeared." What now?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  - [I set `COPILOT_GITHUB_TOKEN`, but models status shows "Shell env: off." Why?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
- [Sessions and multiple chats](#sessions-and-multiple-chats)
  - [How do I start a fresh conversation?](#how-do-i-start-a-fresh-conversation)
  - [Do sessions reset automatically if I never send `/new`?](#do-sessions-reset-automatically-if-i-never-send-new)
  - [Is there a way to make a team of OpenClaw instances one CEO and many agents](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  - [Why did context get truncated mid-task? How do I prevent it?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  - [How do I completely reset OpenClaw but keep it installed?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  - [I'm getting "context too large" errors - how do I reset or compact?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  - [Why am I seeing "LLM request rejected: messages.N.content.X.tool_use.input: Field required"?](#why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required)
  - [Why am I getting heartbeat messages every 30 minutes?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  - [Do I need to add a "bot account" to a WhatsApp group?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  - [How do I get the JID of a WhatsApp group?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  - [Why doesn't OpenClaw reply in a group?](#why-doesnt-openclaw-reply-in-a-group)
  - [Do groups/threads share context with DMs?](#do-groupsthreads-share-context-with-dms)
  - [How many workspaces and agents can I create?](#how-many-workspaces-and-agents-can-i-create)
  - [Can I run multiple bots or chats at the same time (Slack), and how should I set that up?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
- [Models: defaults, selection, aliases, switching](#models-defaults-selection-aliases-switching)
  - [What is the "default model"?](#what-is-the-default-model)
  - [What model do you recommend?](#what-model-do-you-recommend)
  - [How do I switch models without wiping my config?](#how-do-i-switch-models-without-wiping-my-config)
  - [Can I use self-hosted models (llama.cpp, vLLM, Ollama)?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  - [What do OpenClaw, Flawd, and Krill use for models?](#what-do-openclaw-flawd-and-krill-use-for-models)
  - [How do I switch models on the fly (without restarting)?](#how-do-i-switch-models-on-the-fly-without-restarting)
  - [Can I use GPT 5.2 for daily tasks and Codex 5.3 for coding](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
  - [Why do I see "Model … is not allowed" and then no reply?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  - [Why do I see "Unknown model: minimax/MiniMax-M2.1"?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  - [Can I use MiniMax as my default and OpenAI for complex tasks?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  - [Are opus / sonnet / gpt built-in shortcuts?](#are-opus-sonnet-gpt-builtin-shortcuts)
  - [How do I define/override model shortcuts (aliases)?](#how-do-i-defineoverride-model-shortcuts-aliases)
  - [How do I add models from other providers like OpenRouter or Z.AI?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
- [Model failover and "All models failed"](#model-failover-and-all-models-failed)
  - [How does failover work?](#how-does-failover-work)
  - [What does this error mean?](#what-does-this-error-mean)
  - [Fix checklist for `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  - [Why did it also try Google Gemini and fail?](#why-did-it-also-try-google-gemini-and-fail)
- [Auth profiles: what they are and how to manage them](#auth-profiles-what-they-are-and-how-to-manage-them)
  - [What is an auth profile?](#what-is-an-auth-profile)
  - [What are typical profile IDs?](#what-are-typical-profile-ids)
  - [Can I control which auth profile is tried first?](#can-i-control-which-auth-profile-is-tried-first)
  - [OAuth vs API key: what's the difference?](#oauth-vs-api-key-whats-the-difference)
- [Gateway: ports, "already running", and remote mode](#gateway-ports-already-running-and-remote-mode)
  - [What port does the Gateway use?](#what-port-does-the-gateway-use)
  - [Why does `openclaw gateway status` say `Runtime: running` but `RPC probe: failed`?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  - [Why does `openclaw gateway status` show `Config (cli)` and `Config (service)` different?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  - [What does "another gateway instance is already listening" mean?](#what-does-another-gateway-instance-is-already-listening-mean)
  - [How do I run OpenClaw in remote mode (client connects to a Gateway elsewhere)?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  - [The Control UI says "unauthorized" (or keeps reconnecting). What now?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  - [I set `gateway.bind: "tailnet"` but it can't bind / nothing listens](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  - [Can I run multiple Gateways on the same host?](#can-i-run-multiple-gateways-on-the-same-host)
  - [What does "invalid handshake" / code 1008 mean?](#what-does-invalid-handshake-code-1008-mean)
- [Logging and debugging](#logging-and-debugging)
  - [Where are logs?](#where-are-logs)
  - [How do I start/stop/restart the Gateway service?](#how-do-i-startstoprestart-the-gateway-service)
  - [I closed my terminal on Windows - how do I restart OpenClaw?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  - [The Gateway is up but replies never arrive. What should I check?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  - ["Disconnected from gateway: no reason" - what now?](#disconnected-from-gateway-no-reason-what-now)
  - [Telegram setMyCommands fails with network errors. What should I check?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  - [TUI shows no output. What should I check?](#tui-shows-no-output-what-should-i-check)
  - [How do I completely stop then start the Gateway?](#how-do-i-completely-stop-then-start-the-gateway)
  - [ELI5: `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  - [What's the fastest way to get more details when something fails?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
- [Media and attachments](#media-and-attachments)
  - [My skill generated an image/PDF, but nothing was sent](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
- [Security and access control](#security-and-access-control)
  - [Is it safe to expose OpenClaw to inbound DMs?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  - [Is prompt injection only a concern for public bots?](#is-prompt-injection-only-a-concern-for-public-bots)
  - [Should my bot have its own email GitHub account or phone number](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  - [Can I give it autonomy over my text messages and is that safe](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  - [Can I use cheaper models for personal assistant tasks?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  - [I ran `/start` in Telegram but didn't get a pairing code](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  - [WhatsApp: will it message my contacts? How does pairing work?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
- [Chat commands, aborting tasks, and "it won't stop"](#chat-commands-aborting-tasks-and-it-wont-stop)
  - [How do I stop internal system messages from showing in chat](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  - [How do I stop/cancel a running task?](#how-do-i-stopcancel-a-running-task)
  - [How do I send a Discord message from Telegram? ("Cross-context messaging denied")](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  - [Why does it feel like the bot "ignores" rapid-fire messages?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## 문제 발생 시 처음 60초

1. **빠른 상태 확인 (첫 번째 점검)**

   ```bash
   openclaw status
   ```

   빠른 로컬 요약: OS + 업데이트, 게이트웨이/서비스 접근성, 에이전트/세션, 프로바이더 구성 + 런타임 이슈 (게이트웨이에 접근 가능할 때).

2. **붙여넣기 가능한 리포트 (공유하기에 안전)**

   ```bash
   openclaw status --all
   ```

   읽기 전용 진단 및 로그 꼬리 (토큰은 마스킹됨).

3. **데몬 + 포트 상태**

   ```bash
   openclaw gateway status
   ```

   슈퍼바이저 런타임 vs RPC 접근성, 프로브 대상 URL, 서비스가 사용한 것으로 추정되는 구성을 표시합니다.

4. **심층 프로브**

   ```bash
   openclaw status --deep
   ```

   게이트웨이 상태 점검 + 프로바이더 프로브를 실행합니다 (접근 가능한 게이트웨이 필요). [Health](/ko-KR/gateway/health)를 참조하세요.

5. **최신 로그 추적**

   ```bash
   openclaw logs --follow
   ```

   RPC가 다운된 경우 다음으로 대체하세요:

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   파일 로그는 서비스 로그와 별개입니다. [Logging](/ko-KR/logging) 및 [문제 해결](/ko-KR/gateway/troubleshooting)을 참조하세요.

6. **닥터 실행 (복구)**

   ```bash
   openclaw doctor
   ```

   구성/상태를 복구/마이그레이션하고 상태 점검을 실행합니다. [Doctor](/ko-KR/gateway/doctor)를 참조하세요.

7. **게이트웨이 스냅샷**

   ```bash
   openclaw health --json
   openclaw health --verbose   # shows the target URL + config path on errors
   ```

   실행 중인 게이트웨이에 전체 스냅샷을 요청합니다 (WS 전용). [Health](/ko-KR/gateway/health)를 참조하세요.

## 빠른 시작 및 초기 설정

### 막혔어요 가장 빨리 해결하는 방법은

**자신의 컴퓨터를 볼 수 있는** 로컬 AI 에이전트를 사용하세요. Discord 에서 질문하는 것보다 훨씬 효과적입니다. 대부분의 "막혔어요" 사례는 원격 도우미가 검사할 수 없는 **로컬 구성 또는 환경 문제**이기 때문입니다.

- **Claude Code**: [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
- **OpenAI Codex**: [https://openai.com/codex/](https://openai.com/codex/)

이러한 도구는 저장소를 읽고, 명령을 실행하고, 로그를 검사하고, 머신 수준의 설정(PATH, 서비스, 권한, 인증 파일)을 수정하는 데 도움을 줄 수 있습니다. hackable (git) 설치를 통해 **전체 소스 체크아웃**을 제공하세요:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

이렇게 하면 OpenClaw 가 **git 체크아웃에서** 설치되므로 에이전트가 코드 + 문서를 읽고 실행 중인 정확한 버전에 대해 추론할 수 있습니다. `--install-method git` 없이 인스톨러를 다시 실행하면 언제든 stable 로 되돌릴 수 있습니다.

팁: 에이전트에게 수정 사항을 **계획하고 감독**하도록 요청한 후 (단계별로), 필요한 명령만 실행하세요. 변경 사항이 작고 감사하기 쉬워집니다.

실제 버그나 수정 사항을 발견하면 GitHub 이슈를 등록하거나 PR 을 보내주세요:
[https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues)
[https://github.com/openclaw/openclaw/pulls](https://github.com/openclaw/openclaw/pulls)

다음 명령어로 시작하세요 (도움을 요청할 때 출력을 공유하세요):

```bash
openclaw status
openclaw models status
openclaw doctor
```

각 명령의 기능:

- `openclaw status`: 게이트웨이/에이전트 상태 + 기본 구성의 빠른 스냅샷.
- `openclaw models status`: 프로바이더 인증 + 모델 가용성 확인.
- `openclaw doctor`: 일반적인 구성/상태 문제를 검증하고 복구.

기타 유용한 CLI 점검: `openclaw status --all`, `openclaw logs --follow`,
`openclaw gateway status`, `openclaw health --verbose`.

빠른 디버그 루프: [문제 발생 시 처음 60초](#문제-발생-시-처음-60초).
설치 문서: [Install](/ko-KR/install), [인스톨러 플래그](/ko-KR/install/installer), [업데이트](/ko-KR/install/updating).

### OpenClaw 을 설치하고 설정하는 권장 방법은

저장소에서는 소스에서 실행하고 온보딩 마법사를 사용하는 것을 권장합니다:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

마법사는 UI 자산을 자동으로 빌드할 수도 있습니다. 온보딩 후에는 일반적으로 포트 **18789** 에서 게이트웨이를 실행합니다.

소스에서 (기여자/개발):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # auto-installs UI deps on first run
openclaw onboard
```

아직 전역 설치가 없다면 `pnpm openclaw onboard` 로 실행하세요.

### 온보딩 후 대시보드를 여는 방법

마법사는 온보딩 직후 깨끗한 (토큰이 없는) 대시보드 URL 로 브라우저를 열고 요약에 링크를 출력합니다. 해당 탭을 유지하세요. 실행되지 않았다면 같은 머신에서 출력된 URL 을 복사/붙여넣기하세요.

### localhost vs 원격에서 대시보드 토큰을 인증하는 방법

**Localhost (같은 머신):**

- `http://127.0.0.1:18789/`를 여세요.
- 인증을 요청하면 `gateway.auth.token` (또는 `OPENCLAW_GATEWAY_TOKEN`)의 토큰을 Control UI 설정에 붙여넣으세요.
- 게이트웨이 호스트에서 가져오기: `openclaw config get gateway.auth.token` (또는 생성: `openclaw doctor --generate-gateway-token`).

**Localhost 가 아닌 경우:**

- **Tailscale Serve** (권장): bind 를 loopback 으로 유지하고, `openclaw gateway --tailscale serve` 를 실행한 후, `https://<magicdns>/`를 여세요. `gateway.auth.allowTailscale`이 `true`이면 identity 헤더가 인증을 충족합니다 (토큰 불필요).
- **Tailnet bind**: `openclaw gateway --bind tailnet --token "<token>"`을 실행하고, `http://<tailscale-ip>:18789/`를 열어 대시보드 설정에 토큰을 붙여넣으세요.
- **SSH tunnel**: `ssh -N -L 18789:127.0.0.1:18789 user@host` 후 `http://127.0.0.1:18789/`를 열고 Control UI 설정에 토큰을 붙여넣으세요.

[Dashboard](/ko-KR/web/dashboard) 및 [Web surfaces](/ko-KR/web)에서 bind 모드와 인증 세부 사항을 참조하세요.

### 어떤 런타임이 필요한가요

Node **>= 22** 가 필요합니다. `pnpm`이 권장됩니다. Bun 은 게이트웨이에 **권장되지 않습니다**.

### Raspberry Pi 에서 실행할 수 있나요

예. 게이트웨이는 가벼워서 개인 사용에는 **512MB-1GB RAM**, **1 코어**, 약 **500MB** 디스크면 충분하며, **Raspberry Pi 4 에서 실행**할 수 있습니다.

여유 공간이 필요하다면 (로그, 미디어, 기타 서비스), **2GB 가 권장**되지만 엄격한 최소 요구 사항은 아닙니다.

팁: 작은 Pi/VPS 가 게이트웨이를 호스팅하고, 로컬 화면/카메라/캔버스 또는 명령 실행을 위해 노트북/폰에 **노드**를 페어링할 수 있습니다. [Nodes](/ko-KR/nodes)를 참조하세요.

### Raspberry Pi 설치 팁이 있나요

요약: 작동하지만 거친 부분이 있을 수 있습니다.

- **64비트** OS 를 사용하고 Node >= 22 를 유지하세요.
- 로그를 확인하고 빠르게 업데이트할 수 있도록 **hackable (git) 설치**를 선호하세요.
- 채널/스킬 없이 시작한 후 하나씩 추가하세요.
- 이상한 바이너리 문제가 발생하면 보통 **ARM 호환성** 문제입니다.

문서: [Linux](/ko-KR/platforms/linux), [Install](/ko-KR/install).

### "wake up my friend"에서 멈춤 / 온보딩이 부화되지 않습니다. 어떻게 하나요

이 화면은 게이트웨이에 접근 가능하고 인증이 되어 있어야 합니다. TUI 는 첫 부화 시 자동으로 "Wake up, my friend!"를 보냅니다. 이 줄이 **응답 없이** 표시되고 토큰이 0에 머무르면 에이전트가 실행되지 않은 것입니다.

1. 게이트웨이를 재시작하세요:

```bash
openclaw gateway restart
```

2. 상태 + 인증을 확인하세요:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3. 여전히 멈추면 다음을 실행하세요:

```bash
openclaw doctor
```

게이트웨이가 원격인 경우 터널/Tailscale 연결이 활성화되어 있고 UI 가 올바른 게이트웨이를 가리키고 있는지 확인하세요. [Remote access](/ko-KR/gateway/remote)를 참조하세요.

### 온보딩을 다시 하지 않고 새 머신 (Mac mini)으로 설정을 마이그레이션할 수 있나요

예. **상태 디렉토리**와 **워크스페이스**를 복사한 후 Doctor 를 한 번 실행하세요. **두 위치 모두** 복사하면 봇이 "정확히 동일"하게 유지됩니다 (메모리, 세션 기록, 인증, 채널 상태):

1. 새 머신에 OpenClaw 를 설치하세요.
2. 이전 머신에서 `$OPENCLAW_STATE_DIR` (기본값: `~/.openclaw`)를 복사하세요.
3. 워크스페이스를 복사하세요 (기본값: `~/.openclaw/workspace`).
4. `openclaw doctor`를 실행하고 게이트웨이 서비스를 재시작하세요.

이렇게 하면 구성, 인증 프로필, WhatsApp 자격 증명, 세션, 메모리가 보존됩니다. 원격 모드인 경우 게이트웨이 호스트가 세션 저장소와 워크스페이스를 소유한다는 점을 기억하세요.

**중요:** 워크스페이스만 GitHub 에 커밋/푸시하면 **메모리 + 부트스트랩 파일**만 백업되고 세션 기록이나 인증은 **백업되지 않습니다**. 이들은 `~/.openclaw/` 아래에 있습니다 (예: `~/.openclaw/agents/<agentId>/sessions/`).

관련: [마이그레이션](/ko-KR/install/migrating), [디스크상의 데이터 위치](/ko-KR/help/faq#where-does-openclaw-store-its-data),
[에이전트 워크스페이스](/ko-KR/concepts/agent-workspace), [Doctor](/ko-KR/gateway/doctor),
[원격 모드](/ko-KR/gateway/remote).

### 최신 버전의 새로운 기능은 어디서 확인하나요

GitHub 변경 로그를 확인하세요:
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

최신 항목이 맨 위에 있습니다. 맨 위 섹션이 **Unreleased** 로 표시되면 다음 날짜가 있는 섹션이 최신 출시 버전입니다. 항목은 **Highlights**, **Changes**, **Fixes** (및 필요시 docs/기타 섹션)로 그룹화됩니다.

### docs.openclaw.ai 에 접근할 수 없습니다 (SSL 오류). 어떻게 하나요

일부 Comcast/Xfinity 연결이 Xfinity Advanced Security 를 통해 `docs.openclaw.ai`를 잘못 차단합니다. 이를 비활성화하거나 `docs.openclaw.ai`를 허용 목록에 추가한 후 다시 시도하세요. 자세한 내용: [문제 해결](/ko-KR/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity).
차단 해제를 도와주시려면 여기에 보고해 주세요: [https://spa.xfinity.com/check_url_status](https://spa.xfinity.com/check_url_status).

여전히 사이트에 접근할 수 없다면 문서는 GitHub 에 미러링되어 있습니다:
[https://github.com/openclaw/openclaw/tree/main/docs](https://github.com/openclaw/openclaw/tree/main/docs)

### stable 과 beta 의 차이점은 무엇인가요

**Stable** 과 **beta** 는 별도의 코드 라인이 아닌 **npm dist-tag** 입니다:

- `latest` = stable
- `beta` = 테스트용 초기 빌드

빌드를 **beta** 에 배포하고 테스트한 후 안정적이면 **같은 버전을 `latest`로 승격**합니다. 따라서 beta 와 stable 이 **같은 버전**을 가리킬 수 있습니다.

변경 사항 확인:
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

### beta 버전을 설치하는 방법은? beta 와 dev 의 차이점은

**Beta** 는 npm dist-tag `beta` 입니다 (`latest`와 일치할 수 있음).
**Dev** 는 `main`의 이동 헤드 (git)이며, 게시될 때 npm dist-tag `dev`를 사용합니다.

원라이너 (macOS/Linux):

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Windows 인스톨러 (PowerShell):
[https://openclaw.ai/install.ps1](https://openclaw.ai/install.ps1)

자세한 내용: [Development channels](/ko-KR/install/development-channels) 및 [인스톨러 플래그](/ko-KR/install/installer).

### 설치와 온보딩은 보통 얼마나 걸리나요

대략적인 가이드:

- **설치:** 2-5분
- **온보딩:** 구성하는 채널/모델 수에 따라 5-15분

멈추면 [인스톨러 멈춤](/ko-KR/help/faq#installer-stuck-how-do-i-get-more-feedback)과
[막혔어요](/ko-KR/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck)의 빠른 디버그 루프를 사용하세요.

### 최신 코드를 시도하는 방법은

두 가지 옵션:

1. **Dev 채널 (git checkout):**

```bash
openclaw update --channel dev
```

`main` 브랜치로 전환하고 소스에서 업데이트합니다.

2. **Hackable 설치 (인스톨러 사이트에서):**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

편집할 수 있는 로컬 저장소가 제공되며 git 을 통해 업데이트합니다.

수동으로 클린 클론을 선호하면 다음을 사용하세요:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

문서: [Update](/ko-KR/cli/update), [Development channels](/ko-KR/install/development-channels),
[Install](/ko-KR/install).

### 인스톨러가 멈췄나요? 더 많은 피드백을 받는 방법은

**verbose 출력**으로 인스톨러를 다시 실행하세요:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose
```

verbose 와 함께 beta 설치:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --beta --verbose
```

hackable (git) 설치의 경우:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --verbose
```

Windows (PowerShell) 동등 명령:

```powershell
# install.ps1 has no dedicated -Verbose flag yet.
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

추가 옵션: [인스톨러 플래그](/ko-KR/install/installer).

### Windows 설치에서 git not found 또는 openclaw not recognized 오류

두 가지 일반적인 Windows 문제:

**1) npm error spawn git / git not found**

- **Git for Windows** 를 설치하고 `git`이 PATH 에 있는지 확인하세요.
- PowerShell 을 닫고 다시 연 후 인스톨러를 다시 실행하세요.

**2) openclaw is not recognized after install**

- npm 전역 bin 폴더가 PATH 에 없습니다.
- 경로를 확인하세요:

  ```powershell
  npm config get prefix
  ```

- `<prefix>\\bin`이 PATH 에 있는지 확인하세요 (대부분의 시스템에서 `%AppData%\\npm`).
- PATH 업데이트 후 PowerShell 을 닫고 다시 여세요.

가장 원활한 Windows 설정을 원하면 기본 Windows 대신 **WSL2** 를 사용하세요.
문서: [Windows](/ko-KR/platforms/windows).

### 문서에서 답을 찾지 못했습니다 - 더 나은 답변을 얻는 방법은

**hackable (git) 설치**를 사용하여 전체 소스와 문서를 로컬에 두고, _해당 폴더에서_ 봇 (또는 Claude/Codex)에게 물어보세요. 저장소를 읽고 정확하게 답할 수 있습니다.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

More detail: [Install](/ko-KR/install) and [Installer flags](/ko-KR/install/installer).

### How do I install OpenClaw on Linux

Short answer: follow the Linux guide, then run the onboarding wizard.

- Linux quick path + service install: [Linux](/ko-KR/platforms/linux).
- Full walkthrough: [Getting Started](/ko-KR/start/getting-started).
- Installer + updates: [Install & updates](/ko-KR/install/updating).

### How do I install OpenClaw on a VPS

Any Linux VPS works. Install on the server, then use SSH/Tailscale to reach the Gateway.

Guides: [exe.dev](/ko-KR/install/exe-dev), [Hetzner](/ko-KR/install/hetzner), [Fly.io](/ko-KR/install/fly).
원격 접근: [게이트웨이 원격](/ko-KR/gateway/remote).

### 클라우드/VPS 설치 가이드는 어디에 있나요

일반적인 프로바이더가 포함된 **호스팅 허브**를 유지합니다. 하나를 선택하고 가이드를 따르세요:

- [VPS 호스팅](/ko-KR/vps) (모든 프로바이더를 한 곳에)
- [Fly.io](/ko-KR/install/fly)
- [Hetzner](/ko-KR/install/hetzner)
- [exe.dev](/ko-KR/install/exe-dev)

클라우드에서의 작동 방식: **게이트웨이가 서버에서 실행**되고, 노트북/폰에서 Control UI (또는 Tailscale/SSH)를 통해 접근합니다. 상태 + 워크스페이스는 서버에 있으므로 호스트를 신뢰할 수 있는 소스로 취급하고 백업하세요.

**노드** (Mac/iOS/Android/headless)를 해당 클라우드 게이트웨이에 페어링하여 게이트웨이를 클라우드에 유지하면서 노트북에서 로컬 화면/카메라/캔버스에 접근하거나 명령을 실행할 수 있습니다.

허브: [Platforms](/ko-KR/platforms). 원격 접근: [게이트웨이 원격](/ko-KR/gateway/remote).
노드: [Nodes](/ko-KR/nodes), [Nodes CLI](/ko-KR/cli/nodes).

### OpenClaw 에 자체 업데이트를 요청할 수 있나요

짧은 답변: **가능하지만 권장하지 않습니다**. 업데이트 흐름은 게이트웨이를 재시작할 수 있고 (활성 세션이 끊김), 클린 git checkout 이 필요할 수 있으며, 확인을 요청할 수 있습니다. 더 안전한 방법: 운영자로서 셸에서 업데이트를 실행하세요.

CLI 사용:

```bash
openclaw update
openclaw update status
openclaw update --channel stable|beta|dev
openclaw update --tag <dist-tag|version>
openclaw update --no-restart
```

에이전트에서 자동화해야 하는 경우:

```bash
openclaw update --yes --no-restart
openclaw gateway restart
```

문서: [Update](/ko-KR/cli/update), [업데이트](/ko-KR/install/updating).

### 온보딩 마법사는 실제로 무엇을 하나요

`openclaw onboard`는 권장 설정 경로입니다. **로컬 모드**에서 다음을 안내합니다:

- **모델/인증 설정** (Claude 구독에는 Anthropic **setup-token** 권장, OpenAI Codex OAuth 지원, API 키 선택 사항, LM Studio 로컬 모델 지원)
- **워크스페이스** 위치 + 부트스트랩 파일
- **게이트웨이 설정** (bind/port/auth/tailscale)
- **프로바이더** (WhatsApp, Telegram, Discord, Mattermost (plugin), Signal, iMessage)
- **데몬 설치** (macOS 에서 LaunchAgent; Linux/WSL2 에서 systemd user unit)
- **상태 점검** 및 **스킬** 선택

구성된 모델이 알 수 없거나 인증이 누락된 경우 경고도 표시합니다.

### 이것을 실행하려면 Claude 또는 OpenAI 구독이 필요한가요

아니요. **API 키** (Anthropic/OpenAI/기타) 또는 **로컬 전용 모델**로 OpenClaw 를 실행할 수 있어 데이터가 기기에 머무릅니다. 구독 (Claude Pro/Max 또는 OpenAI Codex)은 해당 프로바이더를 인증하는 선택적 방법입니다.

문서: [Anthropic](/ko-KR/providers/anthropic), [OpenAI](/ko-KR/providers/openai),
[로컬 모델](/ko-KR/gateway/local-models), [모델](/ko-KR/concepts/models).

### API 키 없이 Claude Max 구독을 사용할 수 있나요

예. API 키 대신 **setup-token** 으로 인증할 수 있습니다. 이것이 구독 경로입니다.

Claude Pro/Max 구독에는 **API 키가 포함되지 않으므로** 구독 계정에는 이 방법이 올바른 접근 방식입니다. 중요: Anthropic 의 구독 정책 및 약관에서 이 사용이 허용되는지 확인해야 합니다. 가장 명시적이고 지원되는 경로를 원한다면 Anthropic API 키를 사용하세요.

### Anthropic setup-token 인증은 어떻게 작동하나요

`claude setup-token`은 Claude Code CLI 를 통해 **토큰 문자열**을 생성합니다 (웹 콘솔에서는 사용할 수 없음). **어떤 머신에서든** 실행할 수 있습니다. 마법사에서 **Anthropic token (paste setup-token)** 을 선택하거나 `openclaw models auth paste-token --provider anthropic`으로 붙여넣으세요. 토큰은 **anthropic** 프로바이더의 인증 프로필로 저장되며 API 키처럼 사용됩니다 (자동 갱신 없음). 자세한 내용: [OAuth](/ko-KR/concepts/oauth).

### Anthropic setup-token 은 어디서 찾나요

Anthropic Console 에는 **없습니다**. setup-token 은 **어떤 머신에서든** **Claude Code CLI** 로 생성됩니다:

```bash
claude setup-token
```

출력된 토큰을 복사한 후 마법사에서 **Anthropic token (paste setup-token)** 을 선택하세요. 게이트웨이 호스트에서 실행하려면 `openclaw models auth setup-token --provider anthropic`을 사용하세요. 다른 곳에서 `claude setup-token`을 실행한 경우 게이트웨이 호스트에서 `openclaw models auth paste-token --provider anthropic`으로 붙여넣으세요. [Anthropic](/ko-KR/providers/anthropic)을 참조하세요.

### Claude 구독 인증 (Claude Pro 또는 Max)을 지원하나요

예 - **setup-token** 을 통해. OpenClaw 는 더 이상 Claude Code CLI OAuth 토큰을 재사용하지 않습니다. setup-token 또는 Anthropic API 키를 사용하세요. 어디서든 토큰을 생성하고 게이트웨이 호스트에 붙여넣으세요. [Anthropic](/ko-KR/providers/anthropic) 및 [OAuth](/ko-KR/concepts/oauth)를 참조하세요.

참고: Claude 구독 접근은 Anthropic 의 약관에 의해 관리됩니다. 프로덕션 또는 다중 사용자 워크로드의 경우 API 키가 보통 더 안전한 선택입니다.

### Anthropic 에서 HTTP 429 rate_limit_error 가 표시되는 이유는

현재 기간의 **Anthropic 쿼터/속도 제한**이 소진되었음을 의미합니다. **Claude 구독** (setup-token 또는 Claude Code OAuth)을 사용하는 경우 기간이 재설정될 때까지 기다리거나 플랜을 업그레이드하세요. **Anthropic API 키**를 사용하는 경우 Anthropic Console 에서 사용량/결제를 확인하고 필요에 따라 한도를 올리세요.

팁: **폴백 모델**을 설정하면 프로바이더가 속도 제한되어 있는 동안 OpenClaw 가 계속 응답할 수 있습니다.
[Models](/ko-KR/cli/models) 및 [OAuth](/ko-KR/concepts/oauth)를 참조하세요.

### AWS Bedrock 이 지원되나요

예 - pi-ai 의 **Amazon Bedrock (Converse)** 프로바이더와 **수동 구성**을 통해. 게이트웨이 호스트에서 AWS 자격 증명/리전을 제공하고 모델 구성에 Bedrock 프로바이더 항목을 추가해야 합니다. [Amazon Bedrock](/ko-KR/providers/bedrock) 및 [모델 프로바이더](/ko-KR/providers/models)를 참조하세요. 관리형 키 흐름을 선호하면 Bedrock 앞에 OpenAI 호환 프록시를 사용하는 것도 유효한 옵션입니다.

### Codex 인증은 어떻게 작동하나요

OpenClaw 는 OAuth (ChatGPT 로그인)를 통해 **OpenAI Code (Codex)** 를 지원합니다. 마법사가 OAuth 흐름을 실행할 수 있으며 적절한 경우 기본 모델을 `openai-codex/gpt-5.3-codex`로 설정합니다. [모델 프로바이더](/ko-KR/concepts/model-providers) 및 [마법사](/ko-KR/start/wizard)를 참조하세요.

### OpenAI 구독 인증 (Codex OAuth)을 지원하나요

예. OpenClaw 는 **OpenAI Code (Codex) 구독 OAuth** 를 완전히 지원합니다. 온보딩 마법사가 OAuth 흐름을 실행할 수 있습니다.

[OAuth](/ko-KR/concepts/oauth), [모델 프로바이더](/ko-KR/concepts/model-providers), [마법사](/ko-KR/start/wizard)를 참조하세요.

### Gemini CLI OAuth 설정 방법

Gemini CLI 는 `openclaw.json`의 클라이언트 ID 나 시크릿이 아닌 **플러그인 인증 흐름**을 사용합니다.

단계:

1. 플러그인 활성화: `openclaw plugins enable google-gemini-cli-auth`
2. 로그인: `openclaw models auth login --provider google-gemini-cli --set-default`

게이트웨이 호스트의 인증 프로필에 OAuth 토큰을 저장합니다. 자세한 내용: [모델 프로바이더](/ko-KR/concepts/model-providers).

### 일상적인 대화에 로컬 모델을 사용해도 되나요

보통 아닙니다. OpenClaw 는 대용량 컨텍스트 + 강력한 안전성이 필요합니다. 작은 카드는 잘라내고 누출됩니다. 꼭 사용해야 한다면 로컬에서 (LM Studio) 실행할 수 있는 **가장 큰** MiniMax M2.1 빌드를 실행하고 [/gateway/local-models](/ko-KR/gateway/local-models)를 참조하세요. 더 작은/양자화된 모델은 프롬프트 인젝션 위험을 증가시킵니다 - [Security](/ko-KR/gateway/security)를 참조하세요.

### 호스팅된 모델 트래픽을 특정 지역에 유지하는 방법

리전 고정 엔드포인트를 선택하세요. OpenRouter 는 MiniMax, Kimi, GLM 에 대해 미국 호스팅 옵션을 제공합니다. 데이터를 리전 내에 유지하려면 미국 호스팅 변형을 선택하세요. `models.mode: "merge"`를 사용하면 선택한 리전 프로바이더를 존중하면서 폴백을 사용할 수 있도록 Anthropic/OpenAI 를 함께 나열할 수 있습니다.

### 이것을 설치하려면 Mac Mini 를 구입해야 하나요

아니요. OpenClaw 는 macOS 또는 Linux (Windows 는 WSL2 통해)에서 실행됩니다. Mac mini 는 선택 사항입니다 - 일부 사람들은 항상 켜져 있는 호스트로 구매하지만 작은 VPS, 홈 서버, 또는 Raspberry Pi 급 장치도 작동합니다.

**macOS 전용 도구**에만 Mac 이 필요합니다. iMessage 의 경우 [BlueBubbles](/ko-KR/channels/bluebubbles) (권장)를 사용하세요 - BlueBubbles 서버는 모든 Mac 에서 실행되며 게이트웨이는 Linux 나 다른 곳에서 실행할 수 있습니다. 다른 macOS 전용 도구를 원하면 Mac 에서 게이트웨이를 실행하거나 macOS 노드를 페어링하세요.

문서: [BlueBubbles](/ko-KR/channels/bluebubbles), [Nodes](/ko-KR/nodes), [Mac 원격 모드](/ko-KR/platforms/mac/remote).

### iMessage 지원을 위해 Mac mini 가 필요한가요

Messages 에 로그인한 **macOS 기기**가 필요합니다. Mac mini 일 필요는 **없습니다** - 어떤 Mac 이든 됩니다. iMessage 에는 **[BlueBubbles](/ko-KR/channels/bluebubbles)** (권장)를 사용하세요 - BlueBubbles 서버는 macOS 에서 실행되고, 게이트웨이는 Linux 나 다른 곳에서 실행할 수 있습니다.

일반적인 설정:

- Linux/VPS 에서 게이트웨이를 실행하고, Messages 에 로그인한 모든 Mac 에서 BlueBubbles 서버를 실행합니다.
- 가장 간단한 단일 머신 설정을 원하면 Mac 에서 모든 것을 실행합니다.

문서: [BlueBubbles](/ko-KR/channels/bluebubbles), [Nodes](/ko-KR/nodes),
[Mac 원격 모드](/ko-KR/platforms/mac/remote).

### Mac mini 를 구매해서 OpenClaw 를 실행하면 MacBook Pro 에 연결할 수 있나요

예. **Mac mini 가 게이트웨이를 실행**하고 MacBook Pro 가 **노드** (컴패니언 기기)로 연결할 수 있습니다. 노드는 게이트웨이를 실행하지 않습니다 - 해당 기기에서 화면/카메라/캔버스 및 `system.run` 같은 추가 기능을 제공합니다.

일반적인 패턴:

- Mac mini 에서 게이트웨이 (항상 켜짐).
- MacBook Pro 에서 macOS 앱 또는 노드 호스트를 실행하고 게이트웨이에 페어링.
- `openclaw nodes status` / `openclaw nodes list`로 확인하세요.

문서: [Nodes](/ko-KR/nodes), [Nodes CLI](/ko-KR/cli/nodes).

### Bun 을 사용할 수 있나요

Bun 은 **권장되지 않습니다**. 특히 WhatsApp 과 Telegram 에서 런타임 버그가 발생합니다.
안정적인 게이트웨이에는 **Node** 를 사용하세요.

여전히 Bun 을 실험하고 싶다면 WhatsApp/Telegram 없이 비프로덕션 게이트웨이에서 하세요.

### Telegram: allowFrom 에 무엇을 넣나요

`channels.telegram.allowFrom`은 **사람 발신자의 Telegram 사용자 ID** (숫자)입니다. 봇 사용자 이름이 아닙니다.

온보딩 마법사는 `@username` 입력을 받아 숫자 ID 로 변환하지만, OpenClaw 인증은 숫자 ID 만 사용합니다.

더 안전한 방법 (서드파티 봇 없이):

- 봇에 DM 을 보낸 후 `openclaw logs --follow`를 실행하고 `from.id`를 읽으세요.

공식 Bot API:

- 봇에 DM 을 보낸 후 `https://api.telegram.org/bot<bot_token>/getUpdates`를 호출하고 `message.from.id`를 읽으세요.

Third-party (less private):

- DM `@userinfobot` or `@getidsbot`.

See [/channels/telegram](/ko-KR/channels/telegram#access-control-dms--groups).

### Can multiple people use one WhatsApp number with different OpenClaw instances

Yes, via **multi-agent routing**. Bind each sender's WhatsApp **DM** (peer `kind: "direct"`, sender E.164 like `+15551234567`) to a different `agentId`, so each person gets their own workspace and session store. Replies still come from the **same WhatsApp account**, and DM access control (`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`) is global per WhatsApp account. See [Multi-Agent Routing](/ko-KR/concepts/multi-agent) and [WhatsApp](/ko-KR/channels/whatsapp).

### Can I run a fast chat agent and an Opus for coding agent

Yes. Use multi-agent routing: give each agent its own default model, then bind inbound routes (provider account or specific peers) to each agent. Example config lives in [Multi-Agent Routing](/ko-KR/concepts/multi-agent). See also [Models](/ko-KR/concepts/models) and [Configuration](/ko-KR/gateway/configuration).

### Does Homebrew work on Linux

Yes. Homebrew supports Linux (Linuxbrew). Quick setup:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>
```

systemd 를 통해 OpenClaw 를 실행하는 경우 서비스 PATH 에 `/home/linuxbrew/.linuxbrew/bin` (또는 brew prefix)이 포함되어 `brew`로 설치된 도구가 비로그인 셸에서 해결되는지 확인하세요.
최신 빌드는 Linux systemd 서비스에서 일반적인 사용자 bin 디렉토리도 앞에 추가합니다 (예: `~/.local/bin`, `~/.npm-global/bin`, `~/.local/share/pnpm`, `~/.bun/bin`) 그리고 설정된 경우 `PNPM_HOME`, `NPM_CONFIG_PREFIX`, `BUN_INSTALL`, `VOLTA_HOME`, `ASDF_DATA_DIR`, `NVM_DIR`, `FNM_DIR`을 존중합니다.

### hackable (git) 설치와 npm 설치의 차이점은

- **Hackable (git) 설치:** 전체 소스 체크아웃, 편집 가능, 기여자에게 최적.
  로컬에서 빌드를 실행하고 코드/문서를 패치할 수 있습니다.
- **npm 설치:** 전역 CLI 설치, 저장소 없음, "그냥 실행"에 최적.
  업데이트는 npm dist-tag 에서 제공됩니다.

문서: [시작하기](/ko-KR/start/getting-started), [업데이트](/ko-KR/install/updating).

### npm 과 git 설치 간에 나중에 전환할 수 있나요

예. 다른 방식을 설치한 후 Doctor 를 실행하여 게이트웨이 서비스가 새 엔트리포인트를 가리키도록 하세요.
이렇게 하면 **데이터가 삭제되지 않습니다** - OpenClaw 코드 설치만 변경됩니다. 상태
(`~/.openclaw`)와 워크스페이스 (`~/.openclaw/workspace`)는 그대로 유지됩니다.

npm 에서 git 으로:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

git 에서 npm 으로:

```bash
npm install -g openclaw@latest
openclaw doctor
openclaw gateway restart
```

Doctor 는 게이트웨이 서비스 엔트리포인트 불일치를 감지하고 현재 설치에 맞게 서비스 구성을 다시 작성하도록 제안합니다 (자동화에서 `--repair` 사용).

백업 팁: [백업 전략](/ko-KR/help/faq#whats-the-recommended-backup-strategy)을 참조하세요.

### 게이트웨이를 노트북에서 실행해야 하나요 아니면 VPS 에서

짧은 답변: **24/7 안정성을 원하면 VPS 를 사용하세요**. 가장 낮은 마찰을 원하고 절전/재시작에 문제가 없다면 로컬에서 실행하세요.

**노트북 (로컬 게이트웨이)**

- **장점:** 서버 비용 없음, 로컬 파일에 직접 접근, 라이브 브라우저 창.
- **단점:** 절전/네트워크 끊김 = 연결 해제, OS 업데이트/재부팅 중단, 깨어 있어야 함.

**VPS / 클라우드**

- **장점:** 항상 켜짐, 안정적인 네트워크, 노트북 절전 문제 없음, 계속 실행하기 쉬움.
- **단점:** 보통 헤드리스로 실행 (스크린샷 사용), 원격 파일 접근만 가능, 업데이트에 SSH 필요.

**OpenClaw 관련 참고:** WhatsApp/Telegram/Slack/Mattermost (plugin)/Discord 는 모두 VPS 에서 잘 작동합니다. 유일한 실질적 트레이드오프는 **헤드리스 브라우저** vs 보이는 창입니다. [Browser](/ko-KR/tools/browser)를 참조하세요.

**권장 기본값:** 이전에 게이트웨이 연결 끊김이 있었다면 VPS. Mac 을 적극적으로 사용하고 로컬 파일 접근이나 보이는 브라우저로 UI 자동화를 원할 때는 로컬이 좋습니다.

### 전용 머신에서 OpenClaw 를 실행하는 것이 얼마나 중요한가요

필수는 아니지만 **안정성과 격리를 위해 권장**됩니다.

- **전용 호스트 (VPS/Mac mini/Pi):** 항상 켜짐, 절전/재부팅 중단이 적음, 더 깨끗한 권한, 계속 실행하기 쉬움.
- **공유 노트북/데스크톱:** 테스트와 적극적 사용에 전혀 문제없지만 머신이 절전 모드이거나 업데이트할 때 일시 중지를 예상하세요.

두 가지의 장점을 모두 원하면 게이트웨이를 전용 호스트에 두고 노트북을 로컬 화면/카메라/exec 도구용 **노드**로 페어링하세요. [Nodes](/ko-KR/nodes)를 참조하세요.
보안 가이드는 [Security](/ko-KR/gateway/security)를 읽으세요.

### 최소 VPS 요구 사항과 권장 OS 는

OpenClaw 는 가볍습니다. 기본 게이트웨이 + 채팅 채널 하나의 경우:

- **절대 최소:** 1 vCPU, 1GB RAM, ~500MB 디스크.
- **권장:** 1-2 vCPU, 2GB RAM 이상으로 여유 (로그, 미디어, 다중 채널). 노드 도구와 브라우저 자동화는 리소스를 많이 사용할 수 있습니다.

OS: **Ubuntu LTS** (또는 최신 Debian/Ubuntu)를 사용하세요. Linux 설치 경로가 가장 잘 테스트되어 있습니다.

문서: [Linux](/ko-KR/platforms/linux), [VPS 호스팅](/ko-KR/vps).

### VM 에서 OpenClaw 를 실행할 수 있나요? 요구 사항은

예. VM 을 VPS 와 동일하게 취급하세요: 항상 켜져 있고, 접근 가능하며, 게이트웨이와 활성화한 채널에 충분한 RAM 이 있어야 합니다.

기준 가이드:

- **절대 최소:** 1 vCPU, 1GB RAM.
- **권장:** 다중 채널, 브라우저 자동화 또는 미디어 도구를 실행하는 경우 2GB RAM 이상.
- **OS:** Ubuntu LTS 또는 다른 최신 Debian/Ubuntu.

Windows 를 사용하는 경우 **WSL2 가 가장 쉬운 VM 스타일 설정**이며 최고의 도구 호환성을 제공합니다. [Windows](/ko-KR/platforms/windows), [VPS 호스팅](/ko-KR/vps)을 참조하세요.
macOS 를 VM 에서 실행하는 경우 [macOS VM](/ko-KR/install/macos-vm)을 참조하세요.

## What is OpenClaw?

### What is OpenClaw in one paragraph

OpenClaw is a personal AI assistant you run on your own devices. It replies on the messaging surfaces you already use (WhatsApp, Telegram, Slack, Mattermost (plugin), Discord, Google Chat, Signal, iMessage, WebChat) and can also do voice + a live Canvas on supported platforms. The **Gateway** is the always-on control plane; the assistant is the product.

### What's the value proposition

OpenClaw is not "just a Claude wrapper." It's a **local-first control plane** that lets you run a
capable assistant on **your own hardware**, reachable from the chat apps you already use, with
stateful sessions, memory, and tools - without handing control of your workflows to a hosted
SaaS.

Highlights:

- **Your devices, your data:** run the Gateway wherever you want (Mac, Linux, VPS) and keep the
  workspace + session history local.
- **Real channels, not a web sandbox:** WhatsApp/Telegram/Slack/Discord/Signal/iMessage/etc,
  plus mobile voice and Canvas on supported platforms.
- **Model-agnostic:** use Anthropic, OpenAI, MiniMax, OpenRouter, etc., with per-agent routing
  and failover.
- **Local-only option:** run local models so **all data can stay on your device** if you want.
- **Multi-agent routing:** separate agents per channel, account, or task, each with its own
  workspace and defaults.
- **Open source and hackable:** inspect, extend, and self-host without vendor lock-in.

Docs: [Gateway](/ko-KR/gateway), [Channels](/ko-KR/channels), [Multi-agent](/ko-KR/concepts/multi-agent),
[Memory](/ko-KR/concepts/memory).

### I just set it up what should I do first

Good first projects:

- Build a website (WordPress, Shopify, or a simple static site).
- Prototype a mobile app (outline, screens, API plan).
- Organize files and folders (cleanup, naming, tagging).
- Connect Gmail and automate summaries or follow ups.

It can handle large tasks, but it works best when you split them into phases and
use sub agents for parallel work.

### What are the top five everyday use cases for OpenClaw

Everyday wins usually look like:

- **Personal briefings:** summaries of inbox, calendar, and news you care about.
- **Research and drafting:** quick research, summaries, and first drafts for emails or docs.
- **Reminders and follow ups:** cron or heartbeat driven nudges and checklists.
- **Browser automation:** filling forms, collecting data, and repeating web tasks.
- **Cross device coordination:** send a task from your phone, let the Gateway run it on a server, and get the result back in chat.

### Can OpenClaw help with lead gen outreach ads and blogs for a SaaS

Yes for **research, qualification, and drafting**. It can scan sites, build shortlists,
summarize prospects, and write outreach or ad copy drafts.

For **outreach or ad runs**, keep a human in the loop. Avoid spam, follow local laws and
platform policies, and review anything before it is sent. The safest pattern is to let
OpenClaw draft and you approve.

Docs: [Security](/ko-KR/gateway/security).

### What are the advantages vs Claude Code for web development

OpenClaw is a **personal assistant** and coordination layer, not an IDE replacement. Use
Claude Code or Codex for the fastest direct coding loop inside a repo. Use OpenClaw when you
want durable memory, cross-device access, and tool orchestration.

Advantages:

- **Persistent memory + workspace** across sessions
- **Multi-platform access** (WhatsApp, Telegram, TUI, WebChat)
- **Tool orchestration** (browser, files, scheduling, hooks)
- **Always-on Gateway** (run on a VPS, interact from anywhere)
- **Nodes** for local browser/screen/camera/exec

Showcase: [https://openclaw.ai/showcase](https://openclaw.ai/showcase)

## Skills and automation

### How do I customize skills without keeping the repo dirty

Use managed overrides instead of editing the repo copy. Put your changes in `~/.openclaw/skills/<name>/SKILL.md` (or add a folder via `skills.load.extraDirs` in `~/.openclaw/openclaw.json`). Precedence is `<workspace>/skills` > `~/.openclaw/skills` > bundled, so managed overrides win without touching git. Only upstream-worthy edits should live in the repo and go out as PRs.

### Can I load skills from a custom folder

Yes. Add extra directories via `skills.load.extraDirs` in `~/.openclaw/openclaw.json` (lowest precedence). Default precedence remains: `<workspace>/skills` → `~/.openclaw/skills` → bundled → `skills.load.extraDirs`. `clawhub` installs into `./skills` by default, which OpenClaw treats as `<workspace>/skills`.

### How can I use different models for different tasks

Today the supported patterns are:

- **Cron jobs**: isolated jobs can set a `model` override per job.
- **Sub-agents**: route tasks to separate agents with different default models.
- **On-demand switch**: use `/model` to switch the current session model at any time.

See [Cron jobs](/ko-KR/automation/cron-jobs), [Multi-Agent Routing](/ko-KR/concepts/multi-agent), and [Slash commands](/ko-KR/tools/slash-commands).

### The bot freezes while doing heavy work How do I offload that

Use **sub-agents** for long or parallel tasks. Sub-agents run in their own session,
return a summary, and keep your main chat responsive.

Ask your bot to "spawn a sub-agent for this task" or use `/subagents`.
Use `/status` in chat to see what the Gateway is doing right now (and whether it is busy).

Token tip: long tasks and sub-agents both consume tokens. If cost is a concern, set a
cheaper model for sub-agents via `agents.defaults.subagents.model`.

Docs: [Sub-agents](/ko-KR/tools/subagents).

### Cron or reminders do not fire What should I check

Cron runs inside the Gateway process. If the Gateway is not running continuously,
scheduled jobs will not run.

Checklist:

- Confirm cron is enabled (`cron.enabled`) and `OPENCLAW_SKIP_CRON` is not set.
- Check the Gateway is running 24/7 (no sleep/restarts).
- Verify timezone settings for the job (`--tz` vs host timezone).

Debug:

```bash
openclaw cron run <jobId> --force
openclaw cron runs --id <jobId> --limit 50
```

Docs: [Cron jobs](/ko-KR/automation/cron-jobs), [Cron vs Heartbeat](/ko-KR/automation/cron-vs-heartbeat).

### How do I install skills on Linux

Use **ClawHub** (CLI) or drop skills into your workspace. The macOS Skills UI isn't available on Linux.
Browse skills at [https://clawhub.com](https://clawhub.com).

Install the ClawHub CLI (pick one package manager):

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

### Can OpenClaw run tasks on a schedule or continuously in the background

Yes. Use the Gateway scheduler:

- **Cron jobs** for scheduled or recurring tasks (persist across restarts).
- **Heartbeat** for "main session" periodic checks.
- **Isolated jobs** for autonomous agents that post summaries or deliver to chats.

Docs: [Cron jobs](/ko-KR/automation/cron-jobs), [Cron vs Heartbeat](/ko-KR/automation/cron-vs-heartbeat),
[Heartbeat](/ko-KR/gateway/heartbeat).

### Can I run Apple macOS-only skills from Linux?

Not directly. macOS skills are gated by `metadata.openclaw.os` plus required binaries, and skills only appear in the system prompt when they are eligible on the **Gateway host**. On Linux, `darwin`-only skills (like `apple-notes`, `apple-reminders`, `things-mac`) will not load unless you override the gating.

You have three supported patterns:

**Option A - run the Gateway on a Mac (simplest).**
Run the Gateway where the macOS binaries exist, then connect from Linux in [remote mode](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere) or over Tailscale. The skills load normally because the Gateway host is macOS.

**Option B - use a macOS node (no SSH).**
Run the Gateway on Linux, pair a macOS node (menubar app), and set **Node Run Commands** to "Always Ask" or "Always Allow" on the Mac. OpenClaw can treat macOS-only skills as eligible when the required binaries exist on the node. The agent runs those skills via the `nodes` tool. If you choose "Always Ask", approving "Always Allow" in the prompt adds that command to the allowlist.

**Option C - proxy macOS binaries over SSH (advanced).**
Keep the Gateway on Linux, but make the required CLI binaries resolve to SSH wrappers that run on a Mac. Then override the skill to allow Linux so it stays eligible.

1. Create an SSH wrapper for the binary (example: `memo` for Apple Notes):

   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/memo "$@"
   ```

2. Put the wrapper on `PATH` on the Linux host (for example `~/bin/memo`).
3. Override the skill metadata (workspace or `~/.openclaw/skills`) to allow Linux:

   ```markdown
   ---
   name: apple-notes
   description: Manage Apple Notes via the memo CLI on macOS.
   metadata: { "openclaw": { "os": ["darwin", "linux"], "requires": { "bins": ["memo"] } } }
   ---
   ```

4. Start a new session so the skills snapshot refreshes.

### Do you have a Notion or HeyGen integration

Not built-in today.

Options:

- **Custom skill / plugin:** best for reliable API access (Notion/HeyGen both have APIs).
- **Browser automation:** works without code but is slower and more fragile.

If you want to keep context per client (agency workflows), a simple pattern is:

- One Notion page per client (context + preferences + active work).
- Ask the agent to fetch that page at the start of a session.

If you want a native integration, open a feature request or build a skill
targeting those APIs.

Install skills:

```bash
clawhub install <skill-slug>
clawhub update --all
```

ClawHub installs into `./skills` under your current directory (or falls back to your configured OpenClaw workspace); OpenClaw treats that as `<workspace>/skills` on the next session. For shared skills across agents, place them in `~/.openclaw/skills/<name>/SKILL.md`. Some skills expect binaries installed via Homebrew; on Linux that means Linuxbrew (see the Homebrew Linux FAQ entry above). See [Skills](/ko-KR/tools/skills) and [ClawHub](/ko-KR/tools/clawhub).

### How do I install the Chrome extension for browser takeover

Use the built-in installer, then load the unpacked extension in Chrome:

```bash
openclaw browser extension install
openclaw browser extension path
```

Then Chrome → `chrome://extensions` → enable "Developer mode" → "Load unpacked" → pick that folder.

Full guide (including remote Gateway + security notes): [Chrome extension](/ko-KR/tools/chrome-extension)

If the Gateway runs on the same machine as Chrome (default setup), you usually **do not** need anything extra.
If the Gateway runs elsewhere, run a node host on the browser machine so the Gateway can proxy browser actions.
You still need to click the extension button on the tab you want to control (it doesn't auto-attach).

## Sandboxing and memory

### Is there a dedicated sandboxing doc

Yes. See [Sandboxing](/ko-KR/gateway/sandboxing). For Docker-specific setup (full gateway in Docker or sandbox images), see [Docker](/ko-KR/install/docker).

### Docker feels limited How do I enable full features

The default image is security-first and runs as the `node` user, so it does not
include system packages, Homebrew, or bundled browsers. For a fuller setup:

- Persist `/home/node` with `OPENCLAW_HOME_VOLUME` so caches survive.
- Bake system deps into the image with `OPENCLAW_DOCKER_APT_PACKAGES`.
- Install Playwright browsers via the bundled CLI:
  `node /app/node_modules/playwright-core/cli.js install chromium`
- Set `PLAYWRIGHT_BROWSERS_PATH` and ensure the path is persisted.

Docs: [Docker](/ko-KR/install/docker), [Browser](/ko-KR/tools/browser).

**Can I keep DMs personal but make groups public sandboxed with one agent**

Yes - if your private traffic is **DMs** and your public traffic is **groups**.

Use `agents.defaults.sandbox.mode: "non-main"` so group/channel sessions (non-main keys) run in Docker, while the main DM session stays on-host. Then restrict what tools are available in sandboxed sessions via `tools.sandbox.tools`.

Setup walkthrough + example config: [Groups: personal DMs + public groups](/ko-KR/channels/groups#pattern-personal-dms-public-groups-single-agent)

Key config reference: [Gateway configuration](/ko-KR/gateway/configuration#agentsdefaultssandbox)

### How do I bind a host folder into the sandbox

Set `agents.defaults.sandbox.docker.binds` to `["host:path:mode"]` (e.g., `"/home/user/src:/src:ro"`). Global + per-agent binds merge; per-agent binds are ignored when `scope: "shared"`. Use `:ro` for anything sensitive and remember binds bypass the sandbox filesystem walls. See [Sandboxing](/ko-KR/gateway/sandboxing#custom-bind-mounts) and [Sandbox vs Tool Policy vs Elevated](/ko-KR/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check) for examples and safety notes.

### How does memory work

OpenClaw memory is just Markdown files in the agent workspace:

- Daily notes in `memory/YYYY-MM-DD.md`
- Curated long-term notes in `MEMORY.md` (main/private sessions only)

OpenClaw also runs a **silent pre-compaction memory flush** to remind the model
to write durable notes before auto-compaction. This only runs when the workspace
is writable (read-only sandboxes skip it). See [Memory](/ko-KR/concepts/memory).

### Memory keeps forgetting things How do I make it stick

Ask the bot to **write the fact to memory**. Long-term notes belong in `MEMORY.md`,
short-term context goes into `memory/YYYY-MM-DD.md`.

This is still an area we are improving. It helps to remind the model to store memories;
it will know what to do. If it keeps forgetting, verify the Gateway is using the same
workspace on every run.

Docs: [Memory](/ko-KR/concepts/memory), [Agent workspace](/ko-KR/concepts/agent-workspace).

### Does semantic memory search require an OpenAI API key

Only if you use **OpenAI embeddings**. Codex OAuth covers chat/completions and
does **not** grant embeddings access, so **signing in with Codex (OAuth or the
Codex CLI login)** does not help for semantic memory search. OpenAI embeddings
still need a real API key (`OPENAI_API_KEY` or `models.providers.openai.apiKey`).

If you don't set a provider explicitly, OpenClaw auto-selects a provider when it
can resolve an API key (auth profiles, `models.providers.*.apiKey`, or env vars).
It prefers OpenAI if an OpenAI key resolves, otherwise Gemini if a Gemini key
resolves. If neither key is available, memory search stays disabled until you
configure it. If you have a local model path configured and present, OpenClaw
prefers `local`.

If you'd rather stay local, set `memorySearch.provider = "local"` (and optionally
`memorySearch.fallback = "none"`). If you want Gemini embeddings, set
`memorySearch.provider = "gemini"` and provide `GEMINI_API_KEY` (or
`memorySearch.remote.apiKey`). We support **OpenAI, Gemini, or local** embedding
models - see [Memory](/ko-KR/concepts/memory) for the setup details.

### Does memory persist forever What are the limits

Memory files live on disk and persist until you delete them. The limit is your
storage, not the model. The **session context** is still limited by the model
context window, so long conversations can compact or truncate. That is why
memory search exists - it pulls only the relevant parts back into context.

Docs: [Memory](/ko-KR/concepts/memory), [Context](/ko-KR/concepts/context).

## Where things live on disk

### Is all data used with OpenClaw saved locally

No - **OpenClaw's state is local**, but **external services still see what you send them**.

- **Local by default:** sessions, memory files, config, and workspace live on the Gateway host
  (`~/.openclaw` + your workspace directory).
- **Remote by necessity:** messages you send to model providers (Anthropic/OpenAI/etc.) go to
  their APIs, and chat platforms (WhatsApp/Telegram/Slack/etc.) store message data on their
  servers.
- **You control the footprint:** using local models keeps prompts on your machine, but channel
  traffic still goes through the channel's servers.

Related: [Agent workspace](/ko-KR/concepts/agent-workspace), [Memory](/ko-KR/concepts/memory).

### Where does OpenClaw store its data

Everything lives under `$OPENCLAW_STATE_DIR` (default: `~/.openclaw`):

| Path                                                            | Purpose                                                      |
| --------------------------------------------------------------- | ------------------------------------------------------------ |
| `$OPENCLAW_STATE_DIR/openclaw.json`                             | Main config (JSON5)                                          |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json`                    | Legacy OAuth import (copied into auth profiles on first use) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | Auth profiles (OAuth + API keys)                             |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth.json`          | Runtime auth cache (managed automatically)                   |
| `$OPENCLAW_STATE_DIR/credentials/`                              | Provider state (e.g. `whatsapp/<accountId>/creds.json`)      |
| `$OPENCLAW_STATE_DIR/agents/`                                   | Per-agent state (agentDir + sessions)                        |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/`                | Conversation history & state (per agent)                     |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/sessions.json`   | Session metadata (per agent)                                 |

Legacy single-agent path: `~/.openclaw/agent/*` (migrated by `openclaw doctor`).

Your **workspace** (AGENTS.md, memory files, skills, etc.) is separate and configured via `agents.defaults.workspace` (default: `~/.openclaw/workspace`).

### Where should AGENTSmd SOULmd USERmd MEMORYmd live

These files live in the **agent workspace**, not `~/.openclaw`.

- **Workspace (per agent)**: `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
  `MEMORY.md` (or `memory.md`), `memory/YYYY-MM-DD.md`, optional `HEARTBEAT.md`.
- **State dir (`~/.openclaw`)**: config, credentials, auth profiles, sessions, logs,
  and shared skills (`~/.openclaw/skills`).

Default workspace is `~/.openclaw/workspace`, configurable via:

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

If the bot "forgets" after a restart, confirm the Gateway is using the same
workspace on every launch (and remember: remote mode uses the **gateway host's**
workspace, not your local laptop).

Tip: if you want a durable behavior or preference, ask the bot to **write it into
AGENTS.md or MEMORY.md** rather than relying on chat history.

See [Agent workspace](/ko-KR/concepts/agent-workspace) and [Memory](/ko-KR/concepts/memory).

### What's the recommended backup strategy

Put your **agent workspace** in a **private** git repo and back it up somewhere
private (for example GitHub private). This captures memory + AGENTS/SOUL/USER
files, and lets you restore the assistant's "mind" later.

Do **not** commit anything under `~/.openclaw` (credentials, sessions, tokens).
If you need a full restore, back up both the workspace and the state directory
separately (see the migration question above).

Docs: [Agent workspace](/ko-KR/concepts/agent-workspace).

### How do I completely uninstall OpenClaw

See the dedicated guide: [Uninstall](/ko-KR/install/uninstall).

### Can agents work outside the workspace

Yes. The workspace is the **default cwd** and memory anchor, not a hard sandbox.
Relative paths resolve inside the workspace, but absolute paths can access other
host locations unless sandboxing is enabled. If you need isolation, use
[`agents.defaults.sandbox`](/ko-KR/gateway/sandboxing) or per-agent sandbox settings. If you
want a repo to be the default working directory, point that agent's
`workspace` to the repo root. The OpenClaw repo is just source code; keep the
workspace separate unless you intentionally want the agent to work inside it.

Example (repo as default cwd):

```json5
{
  agents: {
    defaults: {
      workspace: "~/Projects/my-repo",
    },
  },
}
```

### Im in remote mode where is the session store

Session state is owned by the **gateway host**. If you're in remote mode, the session store you care about is on the remote machine, not your local laptop. See [Session management](/ko-KR/concepts/session).

## Config basics

### What format is the config Where is it

OpenClaw reads an optional **JSON5** config from `$OPENCLAW_CONFIG_PATH` (default: `~/.openclaw/openclaw.json`):

```
$OPENCLAW_CONFIG_PATH
```

If the file is missing, it uses safe-ish defaults (including a default workspace of `~/.openclaw/workspace`).

### I set gatewaybind lan or tailnet and now nothing listens the UI says unauthorized

Non-loopback binds **require auth**. Configure `gateway.auth.mode` + `gateway.auth.token` (or use `OPENCLAW_GATEWAY_TOKEN`).

```json5
{
  gateway: {
    bind: "lan",
    auth: {
      mode: "token",
      token: "replace-me",
    },
  },
}
```

Notes:

- `gateway.remote.token` is for **remote CLI calls** only; it does not enable local gateway auth.
- The Control UI authenticates via `connect.params.auth.token` (stored in app/UI settings). Avoid putting tokens in URLs.

### Why do I need a token on localhost now

The wizard generates a gateway token by default (even on loopback) so **local WS clients must authenticate**. This blocks other local processes from calling the Gateway. Paste the token into the Control UI settings (or your client config) to connect.

If you **really** want open loopback, remove `gateway.auth` from your config. Doctor can generate a token for you any time: `openclaw doctor --generate-gateway-token`.

### Do I have to restart after changing config

The Gateway watches the config and supports hot-reload:

- `gateway.reload.mode: "hybrid"` (default): hot-apply safe changes, restart for critical ones
- `hot`, `restart`, `off` are also supported

### How do I enable web search and web fetch

`web_fetch` works without an API key. `web_search` requires a Brave Search API
key. **Recommended:** run `openclaw configure --section web` to store it in
`tools.web.search.apiKey`. Environment alternative: set `BRAVE_API_KEY` for the
Gateway process.

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
      },
      fetch: {
        enabled: true,
      },
    },
  },
}
```

Notes:

- If you use allowlists, add `web_search`/`web_fetch` or `group:web`.
- `web_fetch` is enabled by default (unless explicitly disabled).
- Daemons read env vars from `~/.openclaw/.env` (or the service environment).

Docs: [Web tools](/ko-KR/tools/web).

### How do I run a central Gateway with specialized workers across devices

The common pattern is **one Gateway** (e.g. Raspberry Pi) plus **nodes** and **agents**:

- **Gateway (central):** owns channels (Signal/WhatsApp), routing, and sessions.
- **Nodes (devices):** Macs/iOS/Android connect as peripherals and expose local tools (`system.run`, `canvas`, `camera`).
- **Agents (workers):** separate brains/workspaces for special roles (e.g. "Hetzner ops", "Personal data").
- **Sub-agents:** spawn background work from a main agent when you want parallelism.
- **TUI:** connect to the Gateway and switch agents/sessions.

Docs: [Nodes](/ko-KR/nodes), [Remote access](/ko-KR/gateway/remote), [Multi-Agent Routing](/ko-KR/concepts/multi-agent), [Sub-agents](/ko-KR/tools/subagents), [TUI](/ko-KR/web/tui).

### Can the OpenClaw browser run headless

Yes. It's a config option:

```json5
{
  browser: { headless: true },
  agents: {
    defaults: {
      sandbox: { browser: { headless: true } },
    },
  },
}
```

Default is `false` (headful). Headless is more likely to trigger anti-bot checks on some sites. See [Browser](/ko-KR/tools/browser).

Headless uses the **same Chromium engine** and works for most automation (forms, clicks, scraping, logins). The main differences:

- No visible browser window (use screenshots if you need visuals).
- Some sites are stricter about automation in headless mode (CAPTCHAs, anti-bot).
  For example, X/Twitter often blocks headless sessions.

### How do I use Brave for browser control

Set `browser.executablePath` to your Brave binary (or any Chromium-based browser) and restart the Gateway.
See the full config examples in [Browser](/ko-KR/tools/browser#use-brave-or-another-chromium-based-browser).

## Remote gateways and nodes

### How do commands propagate between Telegram the gateway and nodes

Telegram messages are handled by the **gateway**. The gateway runs the agent and
only then calls nodes over the **Gateway WebSocket** when a node tool is needed:

Telegram → Gateway → Agent → `node.*` → Node → Gateway → Telegram

Nodes don't see inbound provider traffic; they only receive node RPC calls.

### How can my agent access my computer if the Gateway is hosted remotely

Short answer: **pair your computer as a node**. The Gateway runs elsewhere, but it can
call `node.*` tools (screen, camera, system) on your local machine over the Gateway WebSocket.

Typical setup:

1. Run the Gateway on the always-on host (VPS/home server).
2. Put the Gateway host + your computer on the same tailnet.
3. Ensure the Gateway WS is reachable (tailnet bind or SSH tunnel).
4. Open the macOS app locally and connect in **Remote over SSH** mode (or direct tailnet)
   so it can register as a node.
5. Approve the node on the Gateway:

   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

No separate TCP bridge is required; nodes connect over the Gateway WebSocket.

Security reminder: pairing a macOS node allows `system.run` on that machine. Only
pair devices you trust, and review [Security](/ko-KR/gateway/security).

Docs: [Nodes](/ko-KR/nodes), [Gateway protocol](/ko-KR/gateway/protocol), [macOS remote mode](/ko-KR/platforms/mac/remote), [Security](/ko-KR/gateway/security).

### Tailscale is connected but I get no replies What now

Check the basics:

- Gateway is running: `openclaw gateway status`
- Gateway health: `openclaw status`
- Channel health: `openclaw channels status`

Then verify auth and routing:

- If you use Tailscale Serve, make sure `gateway.auth.allowTailscale` is set correctly.
- If you connect via SSH tunnel, confirm the local tunnel is up and points at the right port.
- Confirm your allowlists (DM or group) include your account.

Docs: [Tailscale](/ko-KR/gateway/tailscale), [Remote access](/ko-KR/gateway/remote), [Channels](/ko-KR/channels).

### Can two OpenClaw instances talk to each other local VPS

Yes. There is no built-in "bot-to-bot" bridge, but you can wire it up in a few
reliable ways:

**Simplest:** use a normal chat channel both bots can access (Telegram/Slack/WhatsApp).
Have Bot A send a message to Bot B, then let Bot B reply as usual.

**CLI bridge (generic):** run a script that calls the other Gateway with
`openclaw agent --message ... --deliver`, targeting a chat where the other bot
listens. If one bot is on a remote VPS, point your CLI at that remote Gateway
via SSH/Tailscale (see [Remote access](/ko-KR/gateway/remote)).

Example pattern (run from a machine that can reach the target Gateway):

```bash
openclaw agent --message "Hello from local bot" --deliver --channel telegram --reply-to <chat-id>
```

Tip: add a guardrail so the two bots do not loop endlessly (mention-only, channel
allowlists, or a "do not reply to bot messages" rule).

Docs: [Remote access](/ko-KR/gateway/remote), [Agent CLI](/ko-KR/cli/agent), [Agent send](/ko-KR/tools/agent-send).

### Do I need separate VPSes for multiple agents

No. One Gateway can host multiple agents, each with its own workspace, model defaults,
and routing. That is the normal setup and it is much cheaper and simpler than running
one VPS per agent.

Use separate VPSes only when you need hard isolation (security boundaries) or very
different configs that you do not want to share. Otherwise, keep one Gateway and
use multiple agents or sub-agents.

### Is there a benefit to using a node on my personal laptop instead of SSH from a VPS

Yes - nodes are the first-class way to reach your laptop from a remote Gateway, and they
unlock more than shell access. The Gateway runs on macOS/Linux (Windows via WSL2) and is
lightweight (a small VPS or Raspberry Pi-class box is fine; 4 GB RAM is plenty), so a common
setup is an always-on host plus your laptop as a node.

- **No inbound SSH required.** Nodes connect out to the Gateway WebSocket and use device pairing.
- **Safer execution controls.** `system.run` is gated by node allowlists/approvals on that laptop.
- **More device tools.** Nodes expose `canvas`, `camera`, and `screen` in addition to `system.run`.
- **Local browser automation.** Keep the Gateway on a VPS, but run Chrome locally and relay control
  with the Chrome extension + a node host on the laptop.

SSH is fine for ad-hoc shell access, but nodes are simpler for ongoing agent workflows and
device automation.

Docs: [Nodes](/ko-KR/nodes), [Nodes CLI](/ko-KR/cli/nodes), [Chrome extension](/ko-KR/tools/chrome-extension).

### Should I install on a second laptop or just add a node

If you only need **local tools** (screen/camera/exec) on the second laptop, add it as a
**node**. That keeps a single Gateway and avoids duplicated config. Local node tools are
currently macOS-only, but we plan to extend them to other OSes.

Install a second Gateway only when you need **hard isolation** or two fully separate bots.

Docs: [Nodes](/ko-KR/nodes), [Nodes CLI](/ko-KR/cli/nodes), [Multiple gateways](/ko-KR/gateway/multiple-gateways).

### Do nodes run a gateway service

No. Only **one gateway** should run per host unless you intentionally run isolated profiles (see [Multiple gateways](/ko-KR/gateway/multiple-gateways)). Nodes are peripherals that connect
to the gateway (iOS/Android nodes, or macOS "node mode" in the menubar app). For headless node
hosts and CLI control, see [Node host CLI](/ko-KR/cli/node).

A full restart is required for `gateway`, `discovery`, and `canvasHost` changes.

### Is there an API RPC way to apply config

Yes. `config.apply` validates + writes the full config and restarts the Gateway as part of the operation.

### configapply wiped my config How do I recover and avoid this

`config.apply` replaces the **entire config**. If you send a partial object, everything
else is removed.

Recover:

- Restore from backup (git or a copied `~/.openclaw/openclaw.json`).
- If you have no backup, re-run `openclaw doctor` and reconfigure channels/models.
- If this was unexpected, file a bug and include your last known config or any backup.
- A local coding agent can often reconstruct a working config from logs or history.

Avoid it:

- Use `openclaw config set` for small changes.
- Use `openclaw configure` for interactive edits.

Docs: [Config](/ko-KR/cli/config), [Configure](/ko-KR/cli/configure), [Doctor](/ko-KR/gateway/doctor).

### What's a minimal sane config for a first install

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

This sets your workspace and restricts who can trigger the bot.

### How do I set up Tailscale on a VPS and connect from my Mac

Minimal steps:

1. **Install + login on the VPS**

   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```

2. **Install + login on your Mac**
   - Use the Tailscale app and sign in to the same tailnet.
3. **Enable MagicDNS (recommended)**
   - In the Tailscale admin console, enable MagicDNS so the VPS has a stable name.
4. **Use the tailnet hostname**
   - SSH: `ssh user@your-vps.tailnet-xxxx.ts.net`
   - Gateway WS: `ws://your-vps.tailnet-xxxx.ts.net:18789`

If you want the Control UI without SSH, use Tailscale Serve on the VPS:

```bash
openclaw gateway --tailscale serve
```

This keeps the gateway bound to loopback and exposes HTTPS via Tailscale. See [Tailscale](/ko-KR/gateway/tailscale).

### How do I connect a Mac node to a remote Gateway Tailscale Serve

Serve exposes the **Gateway Control UI + WS**. Nodes connect over the same Gateway WS endpoint.

Recommended setup:

1. **Make sure the VPS + Mac are on the same tailnet**.
2. **Use the macOS app in Remote mode** (SSH target can be the tailnet hostname).
   The app will tunnel the Gateway port and connect as a node.
3. **Approve the node** on the gateway:

   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Docs: [Gateway protocol](/ko-KR/gateway/protocol), [Discovery](/ko-KR/gateway/discovery), [macOS remote mode](/ko-KR/platforms/mac/remote).

## Env vars and .env loading

### How does OpenClaw load environment variables

OpenClaw reads env vars from the parent process (shell, launchd/systemd, CI, etc.) and additionally loads:

- `.env` from the current working directory
- a global fallback `.env` from `~/.openclaw/.env` (aka `$OPENCLAW_STATE_DIR/.env`)

Neither `.env` file overrides existing env vars.

You can also define inline env vars in config (applied only if missing from the process env):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

See [/environment](/ko-KR/help/environment) for full precedence and sources.

### I started the Gateway via the service and my env vars disappeared What now

Two common fixes:

1. Put the missing keys in `~/.openclaw/.env` so they're picked up even when the service doesn't inherit your shell env.
2. Enable shell import (opt-in convenience):

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

This runs your login shell and imports only missing expected keys (never overrides). Env var equivalents:
`OPENCLAW_LOAD_SHELL_ENV=1`, `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`.

### I set COPILOTGITHUBTOKEN but models status shows Shell env off Why

`openclaw models status` reports whether **shell env import** is enabled. "Shell env: off"
does **not** mean your env vars are missing - it just means OpenClaw won't load
your login shell automatically.

If the Gateway runs as a service (launchd/systemd), it won't inherit your shell
environment. Fix by doing one of these:

1. Put the token in `~/.openclaw/.env`:

   ```
   COPILOT_GITHUB_TOKEN=...
   ```

2. Or enable shell import (`env.shellEnv.enabled: true`).
3. Or add it to your config `env` block (applies only if missing).

Then restart the gateway and recheck:

```bash
openclaw models status
```

Copilot tokens are read from `COPILOT_GITHUB_TOKEN` (also `GH_TOKEN` / `GITHUB_TOKEN`).
See [/concepts/model-providers](/ko-KR/concepts/model-providers) and [/environment](/ko-KR/help/environment).

## Sessions and multiple chats

### How do I start a fresh conversation

Send `/new` or `/reset` as a standalone message. See [Session management](/ko-KR/concepts/session).

### Do sessions reset automatically if I never send new

Yes. Sessions expire after `session.idleMinutes` (default **60**). The **next**
message starts a fresh session id for that chat key. This does not delete
transcripts - it just starts a new session.

```json5
{
  session: {
    idleMinutes: 240,
  },
}
```

### Is there a way to make a team of OpenClaw instances one CEO and many agents

Yes, via **multi-agent routing** and **sub-agents**. You can create one coordinator
agent and several worker agents with their own workspaces and models.

That said, this is best seen as a **fun experiment**. It is token heavy and often
less efficient than using one bot with separate sessions. The typical model we
envision is one bot you talk to, with different sessions for parallel work. That
bot can also spawn sub-agents when needed.

Docs: [Multi-agent routing](/ko-KR/concepts/multi-agent), [Sub-agents](/ko-KR/tools/subagents), [Agents CLI](/ko-KR/cli/agents).

### Why did context get truncated midtask How do I prevent it

Session context is limited by the model window. Long chats, large tool outputs, or many
files can trigger compaction or truncation.

What helps:

- Ask the bot to summarize the current state and write it to a file.
- Use `/compact` before long tasks, and `/new` when switching topics.
- Keep important context in the workspace and ask the bot to read it back.
- Use sub-agents for long or parallel work so the main chat stays smaller.
- Pick a model with a larger context window if this happens often.

### How do I completely reset OpenClaw but keep it installed

Use the reset command:

```bash
openclaw reset
```

Non-interactive full reset:

```bash
openclaw reset --scope full --yes --non-interactive
```

Then re-run onboarding:

```bash
openclaw onboard --install-daemon
```

Notes:

- The onboarding wizard also offers **Reset** if it sees an existing config. See [Wizard](/ko-KR/start/wizard).
- If you used profiles (`--profile` / `OPENCLAW_PROFILE`), reset each state dir (defaults are `~/.openclaw-<profile>`).
- Dev reset: `openclaw gateway --dev --reset` (dev-only; wipes dev config + credentials + sessions + workspace).

### Im getting context too large errors how do I reset or compact

Use one of these:

- **Compact** (keeps the conversation but summarizes older turns):

  ```
  /compact
  ```

  or `/compact <instructions>` to guide the summary.

- **Reset** (fresh session ID for the same chat key):

  ```
  /new
  /reset
  ```

If it keeps happening:

- Enable or tune **session pruning** (`agents.defaults.contextPruning`) to trim old tool output.
- Use a model with a larger context window.

Docs: [Compaction](/ko-KR/concepts/compaction), [Session pruning](/ko-KR/concepts/session-pruning), [Session management](/ko-KR/concepts/session).

### Why am I seeing LLM request rejected messagesNcontentXtooluseinput Field required

This is a provider validation error: the model emitted a `tool_use` block without the required
`input`. It usually means the session history is stale or corrupted (often after long threads
or a tool/schema change).

Fix: start a fresh session with `/new` (standalone message).

### Why am I getting heartbeat messages every 30 minutes

Heartbeats run every **30m** by default. Tune or disable them:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h", // or "0m" to disable
      },
    },
  },
}
```

If `HEARTBEAT.md` exists but is effectively empty (only blank lines and markdown
headers like `# Heading`), OpenClaw skips the heartbeat run to save API calls.
If the file is missing, the heartbeat still runs and the model decides what to do.

Per-agent overrides use `agents.list[].heartbeat`. Docs: [Heartbeat](/ko-KR/gateway/heartbeat).

### Do I need to add a bot account to a WhatsApp group

No. OpenClaw runs on **your own account**, so if you're in the group, OpenClaw can see it.
By default, group replies are blocked until you allow senders (`groupPolicy: "allowlist"`).

If you want only **you** to be able to trigger group replies:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

### How do I get the JID of a WhatsApp group

Option 1 (fastest): tail logs and send a test message in the group:

```bash
openclaw logs --follow --json
```

Look for `chatId` (or `from`) ending in `@g.us`, like:
`1234567890-1234567890@g.us`.

Option 2 (if already configured/allowlisted): list groups from config:

```bash
openclaw directory groups list --channel whatsapp
```

Docs: [WhatsApp](/ko-KR/channels/whatsapp), [Directory](/ko-KR/cli/directory), [Logs](/ko-KR/cli/logs).

### Why doesnt OpenClaw reply in a group

Two common causes:

- Mention gating is on (default). You must @mention the bot (or match `mentionPatterns`).
- You configured `channels.whatsapp.groups` without `"*"` and the group isn't allowlisted.

See [Groups](/ko-KR/channels/groups) and [Group messages](/ko-KR/channels/group-messages).

### Do groupsthreads share context with DMs

Direct chats collapse to the main session by default. Groups/channels have their own session keys, and Telegram topics / Discord threads are separate sessions. See [Groups](/ko-KR/channels/groups) and [Group messages](/ko-KR/channels/group-messages).

### How many workspaces and agents can I create

No hard limits. Dozens (even hundreds) are fine, but watch for:

- **Disk growth:** sessions + transcripts live under `~/.openclaw/agents/<agentId>/sessions/`.
- **Token cost:** more agents means more concurrent model usage.
- **Ops overhead:** per-agent auth profiles, workspaces, and channel routing.

Tips:

- Keep one **active** workspace per agent (`agents.defaults.workspace`).
- Prune old sessions (delete JSONL or store entries) if disk grows.
- Use `openclaw doctor` to spot stray workspaces and profile mismatches.

### Can I run multiple bots or chats at the same time Slack and how should I set that up

Yes. Use **Multi-Agent Routing** to run multiple isolated agents and route inbound messages by
channel/account/peer. Slack is supported as a channel and can be bound to specific agents.

Browser access is powerful but not "do anything a human can" - anti-bot, CAPTCHAs, and MFA can
still block automation. For the most reliable browser control, use the Chrome extension relay
on the machine that runs the browser (and keep the Gateway anywhere).

Best-practice setup:

- Always-on Gateway host (VPS/Mac mini).
- One agent per role (bindings).
- Slack channel(s) bound to those agents.
- Local browser via extension relay (or a node) when needed.

Docs: [Multi-Agent Routing](/ko-KR/concepts/multi-agent), [Slack](/ko-KR/channels/slack),
[Browser](/ko-KR/tools/browser), [Chrome extension](/ko-KR/tools/chrome-extension), [Nodes](/ko-KR/nodes).

## Models: defaults, selection, aliases, switching

### What is the default model

OpenClaw's default model is whatever you set as:

```
agents.defaults.model.primary
```

Models are referenced as `provider/model` (example: `anthropic/claude-opus-4-6`). If you omit the provider, OpenClaw currently assumes `anthropic` as a temporary deprecation fallback - but you should still **explicitly** set `provider/model`.

### What model do you recommend

**Recommended default:** `anthropic/claude-opus-4-6`.
**Good alternative:** `anthropic/claude-sonnet-4-5`.
**Reliable (less character):** `openai/gpt-5.2` - nearly as good as Opus, just less personality.
**Budget:** `zai/glm-4.7`.

MiniMax M2.1 has its own docs: [MiniMax](/ko-KR/providers/minimax) and
[Local models](/ko-KR/gateway/local-models).

Rule of thumb: use the **best model you can afford** for high-stakes work, and a cheaper
model for routine chat or summaries. You can route models per agent and use sub-agents to
parallelize long tasks (each sub-agent consumes tokens). See [Models](/ko-KR/concepts/models) and
[Sub-agents](/ko-KR/tools/subagents).

Strong warning: weaker/over-quantized models are more vulnerable to prompt
injection and unsafe behavior. See [Security](/ko-KR/gateway/security).

More context: [Models](/ko-KR/concepts/models).

### Can I use selfhosted models llamacpp vLLM Ollama

Yes. If your local server exposes an OpenAI-compatible API, you can point a
custom provider at it. Ollama is supported directly and is the easiest path.

Security note: smaller or heavily quantized models are more vulnerable to prompt
injection. We strongly recommend **large models** for any bot that can use tools.
If you still want small models, enable sandboxing and strict tool allowlists.

Docs: [Ollama](/ko-KR/providers/ollama), [Local models](/ko-KR/gateway/local-models),
[Model providers](/ko-KR/concepts/model-providers), [Security](/ko-KR/gateway/security),
[Sandboxing](/ko-KR/gateway/sandboxing).

### How do I switch models without wiping my config

Use **model commands** or edit only the **model** fields. Avoid full config replaces.

Safe options:

- `/model` in chat (quick, per-session)
- `openclaw models set ...` (updates just model config)
- `openclaw configure --section model` (interactive)
- edit `agents.defaults.model` in `~/.openclaw/openclaw.json`

Avoid `config.apply` with a partial object unless you intend to replace the whole config.
If you did overwrite config, restore from backup or re-run `openclaw doctor` to repair.

Docs: [Models](/ko-KR/concepts/models), [Configure](/ko-KR/cli/configure), [Config](/ko-KR/cli/config), [Doctor](/ko-KR/gateway/doctor).

### What do OpenClaw, Flawd, and Krill use for models

- **OpenClaw + Flawd:** Anthropic Opus (`anthropic/claude-opus-4-6`) - see [Anthropic](/ko-KR/providers/anthropic).
- **Krill:** MiniMax M2.1 (`minimax/MiniMax-M2.1`) - see [MiniMax](/ko-KR/providers/minimax).

### How do I switch models on the fly without restarting

Use the `/model` command as a standalone message:

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

You can list available models with `/model`, `/model list`, or `/model status`.

`/model` (and `/model list`) shows a compact, numbered picker. Select by number:

```
/model 3
```

You can also force a specific auth profile for the provider (per session):

```
/model opus@anthropic:default
/model opus@anthropic:work
```

Tip: `/model status` shows which agent is active, which `auth-profiles.json` file is being used, and which auth profile will be tried next.
It also shows the configured provider endpoint (`baseUrl`) and API mode (`api`) when available.

**How do I unpin a profile I set with profile**

Re-run `/model` **without** the `@profile` suffix:

```
/model anthropic/claude-opus-4-6
```

If you want to return to the default, pick it from `/model` (or send `/model <default provider/model>`).
Use `/model status` to confirm which auth profile is active.

### Can I use GPT 5.2 for daily tasks and Codex 5.3 for coding

Yes. Set one as default and switch as needed:

- **Quick switch (per session):** `/model gpt-5.2` for daily tasks, `/model gpt-5.3-codex` for coding.
- **Default + switch:** set `agents.defaults.model.primary` to `openai/gpt-5.2`, then switch to `openai-codex/gpt-5.3-codex` when coding (or the other way around).
- **Sub-agents:** route coding tasks to sub-agents with a different default model.

See [Models](/ko-KR/concepts/models) and [Slash commands](/ko-KR/tools/slash-commands).

### Why do I see Model is not allowed and then no reply

If `agents.defaults.models` is set, it becomes the **allowlist** for `/model` and any
session overrides. Choosing a model that isn't in that list returns:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

That error is returned **instead of** a normal reply. Fix: add the model to
`agents.defaults.models`, remove the allowlist, or pick a model from `/model list`.

### Why do I see Unknown model minimaxMiniMaxM21

This means the **provider isn't configured** (no MiniMax provider config or auth
profile was found), so the model can't be resolved. A fix for this detection is
in **2026.1.12** (unreleased at the time of writing).

Fix checklist:

1. Upgrade to **2026.1.12** (or run from source `main`), then restart the gateway.
2. Make sure MiniMax is configured (wizard or JSON), or that a MiniMax API key
   exists in env/auth profiles so the provider can be injected.
3. Use the exact model id (case-sensitive): `minimax/MiniMax-M2.1` or
   `minimax/MiniMax-M2.1-lightning`.
4. Run:

   ```bash
   openclaw models list
   ```

   and pick from the list (or `/model list` in chat).

See [MiniMax](/ko-KR/providers/minimax) and [Models](/ko-KR/concepts/models).

### Can I use MiniMax as my default and OpenAI for complex tasks

Yes. Use **MiniMax as the default** and switch models **per session** when needed.
Fallbacks are for **errors**, not "hard tasks," so use `/model` or a separate agent.

**Option A: switch per session**

```json5
{
  env: { MINIMAX_API_KEY: "sk-...", OPENAI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "minimax" },
        "openai/gpt-5.2": { alias: "gpt" },
      },
    },
  },
}
```

Then:

```
/model gpt
```

**Option B: separate agents**

- Agent A default: MiniMax
- Agent B default: OpenAI
- Route by agent or use `/agent` to switch

Docs: [Models](/ko-KR/concepts/models), [Multi-Agent Routing](/ko-KR/concepts/multi-agent), [MiniMax](/ko-KR/providers/minimax), [OpenAI](/ko-KR/providers/openai).

### Are opus sonnet gpt builtin shortcuts

Yes. OpenClaw ships a few default shorthands (only applied when the model exists in `agents.defaults.models`):

- `opus` → `anthropic/claude-opus-4-6`
- `sonnet` → `anthropic/claude-sonnet-4-5`
- `gpt` → `openai/gpt-5.2`
- `gpt-mini` → `openai/gpt-5-mini`
- `gemini` → `google/gemini-3-pro-preview`
- `gemini-flash` → `google/gemini-3-flash-preview`

If you set your own alias with the same name, your value wins.

### How do I defineoverride model shortcuts aliases

Aliases come from `agents.defaults.models.<modelId>.alias`. Example:

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "anthropic/claude-haiku-4-5": { alias: "haiku" },
      },
    },
  },
}
```

Then `/model sonnet` (or `/<alias>` when supported) resolves to that model ID.

### How do I add models from other providers like OpenRouter or ZAI

OpenRouter (pay-per-token; many models):

```json5
{
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
      models: { "openrouter/anthropic/claude-sonnet-4-5": {} },
    },
  },
  env: { OPENROUTER_API_KEY: "sk-or-..." },
}
```

Z.AI (GLM models):

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
  env: { ZAI_API_KEY: "..." },
}
```

If you reference a provider/model but the required provider key is missing, you'll get a runtime auth error (e.g. `No API key found for provider "zai"`).

**No API key found for provider after adding a new agent**

This usually means the **new agent** has an empty auth store. Auth is per-agent and
stored in:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Fix options:

- Run `openclaw agents add <id>` and configure auth during the wizard.
- Or copy `auth-profiles.json` from the main agent's `agentDir` into the new agent's `agentDir`.

Do **not** reuse `agentDir` across agents; it causes auth/session collisions.

## Model failover and "All models failed"

### How does failover work

Failover happens in two stages:

1. **Auth profile rotation** within the same provider.
2. **Model fallback** to the next model in `agents.defaults.model.fallbacks`.

Cooldowns apply to failing profiles (exponential backoff), so OpenClaw can keep responding even when a provider is rate-limited or temporarily failing.

### What does this error mean

```
No credentials found for profile "anthropic:default"
```

It means the system attempted to use the auth profile ID `anthropic:default`, but could not find credentials for it in the expected auth store.

### Fix checklist for No credentials found for profile anthropicdefault

- **Confirm where auth profiles live** (new vs legacy paths)
  - Current: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  - Legacy: `~/.openclaw/agent/*` (migrated by `openclaw doctor`)
- **Confirm your env var is loaded by the Gateway**
  - If you set `ANTHROPIC_API_KEY` in your shell but run the Gateway via systemd/launchd, it may not inherit it. Put it in `~/.openclaw/.env` or enable `env.shellEnv`.
- **Make sure you're editing the correct agent**
  - Multi-agent setups mean there can be multiple `auth-profiles.json` files.
- **Sanity-check model/auth status**
  - Use `openclaw models status` to see configured models and whether providers are authenticated.

**Fix checklist for No credentials found for profile anthropic**

This means the run is pinned to an Anthropic auth profile, but the Gateway
can't find it in its auth store.

- **Use a setup-token**
  - Run `claude setup-token`, then paste it with `openclaw models auth setup-token --provider anthropic`.
  - If the token was created on another machine, use `openclaw models auth paste-token --provider anthropic`.
- **If you want to use an API key instead**
  - Put `ANTHROPIC_API_KEY` in `~/.openclaw/.env` on the **gateway host**.
  - Clear any pinned order that forces a missing profile:

    ```bash
    openclaw models auth order clear --provider anthropic
    ```

- **Confirm you're running commands on the gateway host**
  - In remote mode, auth profiles live on the gateway machine, not your laptop.

### Why did it also try Google Gemini and fail

If your model config includes Google Gemini as a fallback (or you switched to a Gemini shorthand), OpenClaw will try it during model fallback. If you haven't configured Google credentials, you'll see `No API key found for provider "google"`.

Fix: either provide Google auth, or remove/avoid Google models in `agents.defaults.model.fallbacks` / aliases so fallback doesn't route there.

**LLM request rejected message thinking signature required google antigravity**

Cause: the session history contains **thinking blocks without signatures** (often from
an aborted/partial stream). Google Antigravity requires signatures for thinking blocks.

Fix: OpenClaw now strips unsigned thinking blocks for Google Antigravity Claude. If it still appears, start a **new session** or set `/thinking off` for that agent.

## Auth profiles: what they are and how to manage them

Related: [/concepts/oauth](/ko-KR/concepts/oauth) (OAuth flows, token storage, multi-account patterns)

### What is an auth profile

An auth profile is a named credential record (OAuth or API key) tied to a provider. Profiles live in:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

### What are typical profile IDs

OpenClaw uses provider-prefixed IDs like:

- `anthropic:default` (common when no email identity exists)
- `anthropic:<email>` for OAuth identities
- custom IDs you choose (e.g. `anthropic:work`)

### Can I control which auth profile is tried first

Yes. Config supports optional metadata for profiles and an ordering per provider (`auth.order.<provider>`). This does **not** store secrets; it maps IDs to provider/mode and sets rotation order.

OpenClaw may temporarily skip a profile if it's in a short **cooldown** (rate limits/timeouts/auth failures) or a longer **disabled** state (billing/insufficient credits). To inspect this, run `openclaw models status --json` and check `auth.unusableProfiles`. Tuning: `auth.cooldowns.billingBackoffHours*`.

You can also set a **per-agent** order override (stored in that agent's `auth-profiles.json`) via the CLI:

```bash
# Defaults to the configured default agent (omit --agent)
openclaw models auth order get --provider anthropic

# Lock rotation to a single profile (only try this one)
openclaw models auth order set --provider anthropic anthropic:default

# Or set an explicit order (fallback within provider)
openclaw models auth order set --provider anthropic anthropic:work anthropic:default

# Clear override (fall back to config auth.order / round-robin)
openclaw models auth order clear --provider anthropic
```

To target a specific agent:

```bash
openclaw models auth order set --provider anthropic --agent main anthropic:default
```

### OAuth vs API key whats the difference

OpenClaw supports both:

- **OAuth** often leverages subscription access (where applicable).
- **API keys** use pay-per-token billing.

The wizard explicitly supports Anthropic setup-token and OpenAI Codex OAuth and can store API keys for you.

## Gateway: ports, "already running", and remote mode

### What port does the Gateway use

`gateway.port` controls the single multiplexed port for WebSocket + HTTP (Control UI, hooks, etc.).

Precedence:

```
--port > OPENCLAW_GATEWAY_PORT > gateway.port > default 18789
```

### Why does openclaw gateway status say Runtime running but RPC probe failed

Because "running" is the **supervisor's** view (launchd/systemd/schtasks). The RPC probe is the CLI actually connecting to the gateway WebSocket and calling `status`.

Use `openclaw gateway status` and trust these lines:

- `Probe target:` (the URL the probe actually used)
- `Listening:` (what's actually bound on the port)
- `Last gateway error:` (common root cause when the process is alive but the port isn't listening)

### Why does openclaw gateway status show Config cli and Config service different

You're editing one config file while the service is running another (often a `--profile` / `OPENCLAW_STATE_DIR` mismatch).

Fix:

```bash
openclaw gateway install --force
```

Run that from the same `--profile` / environment you want the service to use.

### What does another gateway instance is already listening mean

OpenClaw enforces a runtime lock by binding the WebSocket listener immediately on startup (default `ws://127.0.0.1:18789`). If the bind fails with `EADDRINUSE`, it throws `GatewayLockError` indicating another instance is already listening.

Fix: stop the other instance, free the port, or run with `openclaw gateway --port <port>`.

### How do I run OpenClaw in remote mode client connects to a Gateway elsewhere

Set `gateway.mode: "remote"` and point to a remote WebSocket URL, optionally with a token/password:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

Notes:

- `openclaw gateway` only starts when `gateway.mode` is `local` (or you pass the override flag).
- The macOS app watches the config file and switches modes live when these values change.

### The Control UI says unauthorized or keeps reconnecting What now

Your gateway is running with auth enabled (`gateway.auth.*`), but the UI is not sending the matching token/password.

Facts (from code):

- The Control UI stores the token in browser localStorage key `openclaw.control.settings.v1`.

Fix:

- Fastest: `openclaw dashboard` (prints + copies the dashboard URL, tries to open; shows SSH hint if headless).
- If you don't have a token yet: `openclaw doctor --generate-gateway-token`.
- If remote, tunnel first: `ssh -N -L 18789:127.0.0.1:18789 user@host` then open `http://127.0.0.1:18789/`.
- Set `gateway.auth.token` (or `OPENCLAW_GATEWAY_TOKEN`) on the gateway host.
- In the Control UI settings, paste the same token.
- Still stuck? Run `openclaw status --all` and follow [Troubleshooting](/ko-KR/gateway/troubleshooting). See [Dashboard](/ko-KR/web/dashboard) for auth details.

### I set gatewaybind tailnet but it cant bind nothing listens

`tailnet` bind picks a Tailscale IP from your network interfaces (100.64.0.0/10). If the machine isn't on Tailscale (or the interface is down), there's nothing to bind to.

Fix:

- Start Tailscale on that host (so it has a 100.x address), or
- Switch to `gateway.bind: "loopback"` / `"lan"`.

Note: `tailnet` is explicit. `auto` prefers loopback; use `gateway.bind: "tailnet"` when you want a tailnet-only bind.

### Can I run multiple Gateways on the same host

Usually no - one Gateway can run multiple messaging channels and agents. Use multiple Gateways only when you need redundancy (ex: rescue bot) or hard isolation.

Yes, but you must isolate:

- `OPENCLAW_CONFIG_PATH` (per-instance config)
- `OPENCLAW_STATE_DIR` (per-instance state)
- `agents.defaults.workspace` (workspace isolation)
- `gateway.port` (unique ports)

Quick setup (recommended):

- Use `openclaw --profile <name> …` per instance (auto-creates `~/.openclaw-<name>`).
- Set a unique `gateway.port` in each profile config (or pass `--port` for manual runs).
- Install a per-profile service: `openclaw --profile <name> gateway install`.

Profiles also suffix service names (`bot.molt.<profile>`; legacy `com.openclaw.*`, `openclaw-gateway-<profile>.service`, `OpenClaw Gateway (<profile>)`).
Full guide: [Multiple gateways](/ko-KR/gateway/multiple-gateways).

### What does invalid handshake code 1008 mean

The Gateway is a **WebSocket server**, and it expects the very first message to
be a `connect` frame. If it receives anything else, it closes the connection
with **code 1008** (policy violation).

Common causes:

- You opened the **HTTP** URL in a browser (`http://...`) instead of a WS client.
- You used the wrong port or path.
- A proxy or tunnel stripped auth headers or sent a non-Gateway request.

Quick fixes:

1. Use the WS URL: `ws://<host>:18789` (or `wss://...` if HTTPS).
2. Don't open the WS port in a normal browser tab.
3. If auth is on, include the token/password in the `connect` frame.

If you're using the CLI or TUI, the URL should look like:

```
openclaw tui --url ws://<host>:18789 --token <token>
```

Protocol details: [Gateway protocol](/ko-KR/gateway/protocol).

## Logging and debugging

### Where are logs

File logs (structured):

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

You can set a stable path via `logging.file`. File log level is controlled by `logging.level`. Console verbosity is controlled by `--verbose` and `logging.consoleLevel`.

Fastest log tail:

```bash
openclaw logs --follow
```

Service/supervisor logs (when the gateway runs via launchd/systemd):

- macOS: `$OPENCLAW_STATE_DIR/logs/gateway.log` and `gateway.err.log` (default: `~/.openclaw/logs/...`; profiles use `~/.openclaw-<profile>/logs/...`)
- Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
- Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

See [Troubleshooting](/ko-KR/gateway/troubleshooting#log-locations) for more.

### How do I startstoprestart the Gateway service

Use the gateway helpers:

```bash
openclaw gateway status
openclaw gateway restart
```

If you run the gateway manually, `openclaw gateway --force` can reclaim the port. See [Gateway](/ko-KR/gateway).

### I closed my terminal on Windows how do I restart OpenClaw

There are **two Windows install modes**:

**1) WSL2 (recommended):** the Gateway runs inside Linux.

Open PowerShell, enter WSL, then restart:

```powershell
wsl
openclaw gateway status
openclaw gateway restart
```

If you never installed the service, start it in the foreground:

```bash
openclaw gateway run
```

**2) Native Windows (not recommended):** the Gateway runs directly in Windows.

Open PowerShell and run:

```powershell
openclaw gateway status
openclaw gateway restart
```

If you run it manually (no service), use:

```powershell
openclaw gateway run
```

Docs: [Windows (WSL2)](/ko-KR/platforms/windows), [Gateway service runbook](/ko-KR/gateway).

### The Gateway is up but replies never arrive What should I check

Start with a quick health sweep:

```bash
openclaw status
openclaw models status
openclaw channels status
openclaw logs --follow
```

Common causes:

- Model auth not loaded on the **gateway host** (check `models status`).
- Channel pairing/allowlist blocking replies (check channel config + logs).
- WebChat/Dashboard is open without the right token.

If you are remote, confirm the tunnel/Tailscale connection is up and that the
Gateway WebSocket is reachable.

Docs: [Channels](/ko-KR/channels), [Troubleshooting](/ko-KR/gateway/troubleshooting), [Remote access](/ko-KR/gateway/remote).

### Disconnected from gateway no reason what now

This usually means the UI lost the WebSocket connection. Check:

1. Is the Gateway running? `openclaw gateway status`
2. Is the Gateway healthy? `openclaw status`
3. Does the UI have the right token? `openclaw dashboard`
4. If remote, is the tunnel/Tailscale link up?

Then tail logs:

```bash
openclaw logs --follow
```

Docs: [Dashboard](/ko-KR/web/dashboard), [Remote access](/ko-KR/gateway/remote), [Troubleshooting](/ko-KR/gateway/troubleshooting).

### Telegram setMyCommands fails with network errors What should I check

Start with logs and channel status:

```bash
openclaw channels status
openclaw channels logs --channel telegram
```

If you are on a VPS or behind a proxy, confirm outbound HTTPS is allowed and DNS works.
If the Gateway is remote, make sure you are looking at logs on the Gateway host.

Docs: [Telegram](/ko-KR/channels/telegram), [Channel troubleshooting](/ko-KR/channels/troubleshooting).

### TUI shows no output What should I check

First confirm the Gateway is reachable and the agent can run:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

In the TUI, use `/status` to see the current state. If you expect replies in a chat
channel, make sure delivery is enabled (`/deliver on`).

Docs: [TUI](/ko-KR/web/tui), [Slash commands](/ko-KR/tools/slash-commands).

### How do I completely stop then start the Gateway

If you installed the service:

```bash
openclaw gateway stop
openclaw gateway start
```

This stops/starts the **supervised service** (launchd on macOS, systemd on Linux).
Use this when the Gateway runs in the background as a daemon.

If you're running in the foreground, stop with Ctrl-C, then:

```bash
openclaw gateway run
```

Docs: [Gateway service runbook](/ko-KR/gateway).

### ELI5 openclaw gateway restart vs openclaw gateway

- `openclaw gateway restart`: restarts the **background service** (launchd/systemd).
- `openclaw gateway`: runs the gateway **in the foreground** for this terminal session.

If you installed the service, use the gateway commands. Use `openclaw gateway` when
you want a one-off, foreground run.

### What's the fastest way to get more details when something fails

Start the Gateway with `--verbose` to get more console detail. Then inspect the log file for channel auth, model routing, and RPC errors.

## Media and attachments

### My skill generated an imagePDF but nothing was sent

Outbound attachments from the agent must include a `MEDIA:<path-or-url>` line (on its own line). See [OpenClaw assistant setup](/ko-KR/start/openclaw) and [Agent send](/ko-KR/tools/agent-send).

CLI sending:

```bash
openclaw message send --target +15555550123 --message "Here you go" --media /path/to/file.png
```

Also check:

- The target channel supports outbound media and isn't blocked by allowlists.
- The file is within the provider's size limits (images are resized to max 2048px).

See [Images](/ko-KR/nodes/images).

## Security and access control

### Is it safe to expose OpenClaw to inbound DMs

Treat inbound DMs as untrusted input. Defaults are designed to reduce risk:

- Default behavior on DM-capable channels is **pairing**:
  - Unknown senders receive a pairing code; the bot does not process their message.
  - Approve with: `openclaw pairing approve <channel> <code>`
  - Pending requests are capped at **3 per channel**; check `openclaw pairing list <channel>` if a code didn't arrive.
- Opening DMs publicly requires explicit opt-in (`dmPolicy: "open"` and allowlist `"*"`).

Run `openclaw doctor` to surface risky DM policies.

### Is prompt injection only a concern for public bots

No. Prompt injection is about **untrusted content**, not just who can DM the bot.
If your assistant reads external content (web search/fetch, browser pages, emails,
docs, attachments, pasted logs), that content can include instructions that try
to hijack the model. This can happen even if **you are the only sender**.

The biggest risk is when tools are enabled: the model can be tricked into
exfiltrating context or calling tools on your behalf. Reduce the blast radius by:

- using a read-only or tool-disabled "reader" agent to summarize untrusted content
- keeping `web_search` / `web_fetch` / `browser` off for tool-enabled agents
- sandboxing and strict tool allowlists

Details: [Security](/ko-KR/gateway/security).

### Should my bot have its own email GitHub account or phone number

Yes, for most setups. Isolating the bot with separate accounts and phone numbers
reduces the blast radius if something goes wrong. This also makes it easier to rotate
credentials or revoke access without impacting your personal accounts.

Start small. Give access only to the tools and accounts you actually need, and expand
later if required.

Docs: [Security](/ko-KR/gateway/security), [Pairing](/ko-KR/channels/pairing).

### Can I give it autonomy over my text messages and is that safe

We do **not** recommend full autonomy over your personal messages. The safest pattern is:

- Keep DMs in **pairing mode** or a tight allowlist.
- Use a **separate number or account** if you want it to message on your behalf.
- Let it draft, then **approve before sending**.

If you want to experiment, do it on a dedicated account and keep it isolated. See
[Security](/ko-KR/gateway/security).

### Can I use cheaper models for personal assistant tasks

Yes, **if** the agent is chat-only and the input is trusted. Smaller tiers are
more susceptible to instruction hijacking, so avoid them for tool-enabled agents
or when reading untrusted content. If you must use a smaller model, lock down
tools and run inside a sandbox. See [Security](/ko-KR/gateway/security).

### I ran start in Telegram but didnt get a pairing code

Pairing codes are sent **only** when an unknown sender messages the bot and
`dmPolicy: "pairing"` is enabled. `/start` by itself doesn't generate a code.

Check pending requests:

```bash
openclaw pairing list telegram
```

If you want immediate access, allowlist your sender id or set `dmPolicy: "open"`
for that account.

### WhatsApp will it message my contacts How does pairing work

No. Default WhatsApp DM policy is **pairing**. Unknown senders only get a pairing code and their message is **not processed**. OpenClaw only replies to chats it receives or to explicit sends you trigger.

Approve pairing with:

```bash
openclaw pairing approve whatsapp <code>
```

List pending requests:

```bash
openclaw pairing list whatsapp
```

Wizard phone number prompt: it's used to set your **allowlist/owner** so your own DMs are permitted. It's not used for auto-sending. If you run on your personal WhatsApp number, use that number and enable `channels.whatsapp.selfChatMode`.

## Chat commands, aborting tasks, and "it won't stop"

### How do I stop internal system messages from showing in chat

Most internal or tool messages only appear when **verbose** or **reasoning** is enabled
for that session.

Fix in the chat where you see it:

```
/verbose off
/reasoning off
```

If it is still noisy, check the session settings in the Control UI and set verbose
to **inherit**. Also confirm you are not using a bot profile with `verboseDefault` set
to `on` in config.

Docs: [Thinking and verbose](/ko-KR/tools/thinking), [Security](/ko-KR/gateway/security#reasoning--verbose-output-in-groups).

### How do I stopcancel a running task

Send any of these **as a standalone message** (no slash):

```
stop
abort
esc
wait
exit
interrupt
```

These are abort triggers (not slash commands).

For background processes (from the exec tool), you can ask the agent to run:

```
process action:kill sessionId:XXX
```

Slash commands overview: see [Slash commands](/ko-KR/tools/slash-commands).

Most commands must be sent as a **standalone** message that starts with `/`, but a few shortcuts (like `/status`) also work inline for allowlisted senders.

### How do I send a Discord message from Telegram Crosscontext messaging denied

OpenClaw blocks **cross-provider** messaging by default. If a tool call is bound
to Telegram, it won't send to Discord unless you explicitly allow it.

Enable cross-provider messaging for the agent:

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[from {channel}] " },
          },
        },
      },
    },
  },
}
```

Restart the gateway after editing config. If you only want this for a single
agent, set it under `agents.list[].tools.message` instead.

### Why does it feel like the bot ignores rapidfire messages

Queue mode controls how new messages interact with an in-flight run. Use `/queue` to change modes:

- `steer` - new messages redirect the current task
- `followup` - run messages one at a time
- `collect` - batch messages and reply once (default)
- `steer-backlog` - steer now, then process backlog
- `interrupt` - abort current run and start fresh

You can add options like `debounce:2s cap:25 drop:summarize` for followup modes.

## Answer the exact question from the screenshot/chat log

**Q: "What's the default model for Anthropic with an API key?"**

**A:** In OpenClaw, credentials and model selection are separate. Setting `ANTHROPIC_API_KEY` (or storing an Anthropic API key in auth profiles) enables authentication, but the actual default model is whatever you configure in `agents.defaults.model.primary` (for example, `anthropic/claude-sonnet-4-5` or `anthropic/claude-opus-4-6`). If you see `No credentials found for profile "anthropic:default"`, it means the Gateway couldn't find Anthropic credentials in the expected `auth-profiles.json` for the agent that's running.

---

Still stuck? Ask in [Discord](https://discord.com/invite/clawd) or open a [GitHub discussion](https://github.com/openclaw/openclaw/discussions).
