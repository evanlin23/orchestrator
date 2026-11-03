# Open-source and third-party software for orchestrating multiple AI coding agents and subscriptions (as of 3 Nov 2026)

Scope: the software layer only (supervisors, consultation bridges, routers/harnesses, proxies, mobile front-ends, protocols, personal-assistant frameworks). Vendor quotas, ToS depth, hosted vendor agents and hosting are covered by other researchers. ToS facts appear here only where they decide whether a tool still works.

**How the maturity data was collected.** github.com and api.github.com were blocked from this research environment. Star counts, created dates, last-push dates and archived flags come from the ecosyste.ms GitHub index (`https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/OWNER%2FREPO`), synced between 2026-10-19 and 2026-11-03. Package versions and release dates come from registry.npmjs.org and pypi.org. READMEs were read from raw.githubusercontent.com on 2026-11-03. Treat star counts as roughly ±2 weeks old. ecosyste.ms returned no GitHub release tags for any repo, so "latest release" below means the npm/PyPI publish date unless stated otherwise.

**Auth legend** (this decides ToS risk):
- **[W] Wraps the official CLI.** It spawns `claude`, `codex`, `agy`/`gemini` and so on as a subprocess, PTY or tmux pane, and the vendor's own CLI owns the login. Lowest risk.
- **[S] Vendor SDK on a subscription login.** For example the Claude Agent SDK, the Codex SDK, or `claude -p` driven by a third-party app. This is a grey zone that depends on the vendor.
- **[O] OAuth token reuse.** The tool extracts or obtains subscription OAuth tokens and calls vendor backends itself, often presenting itself as the official client. High ToS risk; Anthropic and Google actively enforce against it in 2026.
- **[K] API keys.** Pay-as-you-go; not subscription-backed.

---

## Q0. What 2026 platform changes reshape this software layer? (read first)

### Takeaway
Two 2026 changes make most 2025-era guides wrong for this user.
- **Google, 18 June 2026:** "Login with Google" stopped working in Gemini CLI for Google AI Pro/Ultra (and free) users. Consumer Gemini now runs through the closed-source **Antigravity CLI (`agy`)**.
- **Anthropic, January–April 2026:** server-side blocks and ToS changes progressively stopped Claude subscription OAuth from working in third-party harnesses.

Together these push a subscription-only user toward tools that wrap the official CLIs ([W]) and away from [O] tools.

### Cited Findings
- Gemini CLI's own README now says: "No free tier or Google AI plans: Free individual access and Google AI Pro / Ultra (Google One) moved to Antigravity CLI on June 18, 2026". It positions Gemini CLI (`gemini`) for "Gemini Code Assist Standard / Enterprise or Code Assist for GitHub ... (plus legacy paid API / Vertex keys)" and Antigravity CLI (`agy`) for "free / Google AI Pro & Ultra accounts" — [gemini-cli README](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/README.md)
- Google's deprecation page: starting June 18, 2026 the consumer tiers (Gemini Code Assist for individuals, Google AI Pro, Google AI Ultra) "can no longer use the Login with Google option to access the IDE extensions or Gemini CLI", and those users "can migrate to the Antigravity family of products". Code Assist Standard/Enterprise follow a separate schedule. **Google AI Plus is not mentioned** — [Google developers: Code Assist individuals deprecation](https://developers.google.com/gemini-code-assist/docs/deprecations/code-assist-individuals)
- Antigravity CLI is a closed-source Go binary invoked as `agy`. Google says there "won't be 1:1 feature parity right out of the gate" but keeps Agent Skills, Hooks, Subagents and Extensions. Extensions migrate with `agy plugin import gemini` — [The New Stack (via search summary)](https://thenewstack.io/gemini-cli-antigravity-replacement/); [Medium migration guide](https://medium.com/@hsinghungwang/migrating-from-gemini-cli-to-antigravity-cli-agy-af324c10c781)
- `agy -p` headless mode sends one prompt and exits. It needs cached credentials from a prior interactive login, takes `--output-format text|json|stream-json`, and `--print-timeout` defaults to 5m. `agy mcp` manages MCP server configs — [Antigravity CLI headless docs](https://antigravity.google/docs/cli/headless); [computingforgeeks cheat sheet](https://computingforgeeks.com/antigravity-cli-cheat-sheet/)
- A GitHub issue reports that `agy -p "prompt"` hangs indefinitely when run without a TTY (from a subprocess or with piped output), while the equivalent `gemini` command worked — [google-antigravity/antigravity-cli#318](https://github.com/google-antigravity/antigravity-cli/issues/318) (fix status unknown)
- gemini-mcp-tool (v1.1.8, 2026-06-18) switched its default backend to `agy`. It notes that the agy backend is "experimental: print mode is Gemini 3.5 Flash-only ... and tool execution isn't sandboxed in `-p`" — [jamubc/gemini-mcp-tool README](https://raw.githubusercontent.com/jamubc/gemini-mcp-tool/main/README.md)
- Anthropic timeline (secondary sources; no Anthropic primary source was found in this pass):
  - 9 Jan 2026: server-side checks stopped OpenCode, Cline and Roo Code from using Pro/Max OAuth.
  - 19 Feb 2026: ToS/docs gained an "Authentication and credential use" section banning subscription OAuth in third-party tools.
  - 19 Mar 2026: legal requests led OpenCode to remove Anthropic OAuth.
  - About 4 Apr 2026: the block extended to all third-party harnesses, starting with OpenClaw, and such usage was moved to pay-as-you-go "extra usage".
  - Sources: [VentureBeat](https://venturebeat.com/ai/anthropic-cracks-down-on-unauthorized-claude-usage-by-third-party-harnesses/); [WinBuzzer](https://winbuzzer.com/2026/02/19/anthropic-bans-claude-subscription-oauth-in-third-party-apps-xcxwbn/); [GIGAZINE](https://gigazine.net/gsc_news/en/20260220-anthropic-third-party-block); [falcao.org](https://falcao.org/posts/anthropic-claude-access-crackdown-ecosystem-fallout/); [implicator.ai](https://www.implicator.ai/anthropic-cuts-openclaw-from-claude-subscriptions-citing-unsustainable-compute-costs/). The sources disagree on the date: 3 vs 4 April.
- OpenClaw's Anthropic docs record a dated notice from 15 June 2026: Anthropic "paused the previously announced separate Agent SDK credit plan", and "Subscription-plan Agent SDK, `claude -p`, and third-party app usage still count against subscription limits" — [docs.openclaw.ai/providers/anthropic](https://docs.openclaw.ai/providers/anthropic)

### Inferences
- Any tool that still defaults to the `gemini` binary for consumer accounts will fail for this user's Google AI Pro account unless it is reconfigured to `agy`. This includes PAL `clink` (default `cli_name: gemini`), Claude Squad `-p gemini`, ccmanager and Agent Deck. Tools that already moved to `agy` include oh-my-claudecode, NTM, Happy, HAPI and gemini-mcp-tool.
- Because `agy -p` is reported to hang without a TTY, supervisors that drive agents inside tmux/PTY panes are currently more reliable for Gemini than those that call headless print mode from a subprocess. Examples: Claude Squad, oh-my-claudecode `omc team`, NTM, Agent of Empires.
- For Claude, only [W]-style tools are safe for a subscription-only user. These run the real `claude` binary (interactive, tmux, or Anthropic's own Remote Control). `claude -p` driven by third-party software is a grey zone; the ToS researcher should confirm.

### Gaps
- Whether **Google AI Plus** (as opposed to Pro/Ultra) is included in the Gemini CLI retirement is not stated on Google's page. Gemini CLI's README says "Google AI plans" in general.
- Whether `agy` supports ACP, or an MCP-server mode (exposing itself as a tool), was not found.
- No Anthropic primary-source text on the April 2026 third-party harness change was retrieved.

---

## Q1. Multi-agent supervisors that drive official CLIs as subprocesses — which can mix Claude Code + Codex + Gemini/Antigravity?

### Takeaway
The healthiest options in November 2026 that can mix vendors while wrapping the official CLIs ([W]) are:
- **Agent of Empires** — active, tmux, web dashboard usable on a phone.
- **ccmanager** — active, eight CLIs, no tmux needed.
- **Agent Deck** — has a Telegram "conductor".
- **oh-my-claudecode `omc team`** — Claude spawns Codex, Antigravity or Grok tmux workers.
- **Maestro** — moderator "group chat", cross-agent @mentions, quota-aware retry and phone remote. It does not support Gemini.
- **Gas Town** — Mayor/worker hierarchy.
- **HAPI** — wrapper with a phone app.

Several big 2025 names are dead or pivoted:
- Vibe Kanban: sunset in April 2026.
- Terragon: shut down January 2026.
- Uzi: abandoned June 2025.
- Omnara: pivoted to an API-based managed-agent platform.
- Roo Code: archived May 2026 (it is an IDE harness rather than a supervisor; see Q3).

### Cited Findings
Metadata: [ecosyste.ms index](https://repos.ecosyste.ms/) and [npm registry](https://registry.npmjs.org/), as described above. Agents are listed as the README states them.

| Tool | What it does | Agents supported (per README) | Auth | Maturity (★ / last push / latest pkg) | Phone | Source |
|---|---|---|---|---|---|---|
| **Claude Squad** (smtg-ai) | Go TUI that runs each agent in tmux plus its own git worktree; background/yolo mode | Claude Code (default), Codex, Gemini, Aider, OpenCode, Amp via `-p "<program>"` | [W] — README still says "For Codex: Set your API key with `export OPENAI_API_KEY`" | ★8,676, AGPL-3.0, created 2025-03, last push 2026-08-20 | None built in (SSH+tmux only) | [README](https://raw.githubusercontent.com/smtg-ai/claude-squad/main/README.md) |
| **ccmanager** (kbwo) | TUI session manager across worktrees/projects, no tmux; busy/waiting/idle status; status-change hooks; experimental AI auto-approval; devcontainers | Claude Code, Gemini CLI, Codex CLI, Cursor Agent, Copilot CLI, Cline CLI, OpenCode, Kimi CLI | [W] | ★1,274, MIT, npm 4.4.11 on 2026-10-27 | Hooks only (could fire notifications) | [README](https://raw.githubusercontent.com/kbwo/ccmanager/main/README.md) |
| **Agent of Empires (aoe)** | tmux session manager with TUI, web, CLI and HTTP API; worktrees; Docker/Podman/Apple-container sandboxing; diff review; notifications. Supported by Mozilla.ai | Claude Code, OpenCode, Mistral Vibe, Codex CLI, Gemini CLI, Pi (`aoe agents` lists the rest) | [W] | ★3,398, MIT, created 2026-01-09, last push 2026-11-01 | **Yes**: `aoe serve` web dashboard with a "Phone: structured view" | [README](https://raw.githubusercontent.com/agent-of-empires/agent-of-empires/main/README.md) |
| **Agent Deck** (asheshgoplani) | Go TUI and web UI (`agent-deck web` on :8420); MCP manager per session; "Conductor" plus a Telegram bot bridge that "watches every Claude session"; checks `claude auth status` and `codex login status` | Claude Code, Codex, Gemini CLI, Pi, OpenCode | [W] | ★1,125, MIT, last push 2026-10-19 | **Yes**: Telegram conductor and web UI | [README](https://raw.githubusercontent.com/asheshgoplani/agent-deck/main/README.md) |
| **oh-my-claudecode (OMC)** (Yeachan-Heo) | Claude Code plugin and CLI. `omc team N:<cli> "task"` spawns tmux worker panes that die when done; `/ask codex` and `/ask antigravity` "advisor" synthesis. v4.4.0 removed its Codex/Gemini MCP servers in favour of CLI-first workers | claude, codex, gemini ("enterprise/API-key"), antigravity (`agy`), grok, cursor | [W] — "Requires the selected CLI ... installed/authenticated" | ★40,500, MIT, created 2026-01-09; npm `oh-my-claude-sisyphus` 5.7.30 on 2026-11-02. Sibling: oh-my-codex | No | [README](https://raw.githubusercontent.com/Yeachan-Heo/oh-my-claudecode/main/README.md) |
| **Maestro** (RunMaestro) | Desktop "agent orchestration command center". **Group Chat**: one agent is appointed moderator and routes questions to other agents. **Cross-Agent Mentions**: `@agent` consults another agent in the background. **Agent Resilience**: resends the prompt after "529 Overloaded or a spent plan quota". Also Auto Run playbooks, `maestro-cli` headless/cron, and a usage dashboard | Claude Code, Codex, OpenCode, Factory Droid, Copilot-CLI (beta), Qwen3 Coder (beta), Oh My Pi (beta). **Not Gemini CLI** ("may be added") | [W] — "a pass-through to your AI provider ... authentication you have configured ... works identically" | ★3,528, created 2025-11, last push 2026-11-02 | **Yes**: built-in web server with QR code, plus a Cloudflare tunnel for remote access | [README](https://raw.githubusercontent.com/RunMaestro/Maestro/main/README.md) |
| **Gas Town** (gastownhall; Steve Yegge) | "Multi-agent workspace manager". The **Mayor** (a Claude Code instance) coordinates **Polecats** (worker agents). Work state lives in a git-backed Beads ledger; mailboxes and handoffs; Witness/Deacon watchdogs; Refinery merge queue; escalation tiers. Claims it "scale[s] comfortably to 20-30 agents" | Claude Code, GitHub Copilot, Codex, Gemini "and others" | [W] | ★18,620, created 2025-12-16, last push 2026-10-28 | No | [README](https://raw.githubusercontent.com/gastownhall/gastown/main/README.md) |
| **NTM – Named Tmux Manager** (Dicklesworthstone) | Go control plane that spawns and tiles agents in tmux; Agent Mail coordination, file reservations; REST/SSE/WebSocket/OpenAPI "robot" surfaces | Claude Code, Codex, Antigravity CLI, Oh My Pi, Grok Build ("Gemini CLI is supported as legacy") | [W] | ★464, created 2025-12, last push 2026-10-31 | Via its REST/SSE API only | [README](https://raw.githubusercontent.com/Dicklesworthstone/ntm/main/README.md) |
| **HAPI** (tiann) | "Local-first alternative to Happy": runs official CLIs and controls them remotely; seamless local/remote handoff; shared Codex sessions (Codex 0.154.0+) | Claude Code, Codex, Cursor Agent, Grok Build, OpenCode, Kimi, Copilot, Antigravity, Pi, DeepSeek Harness | [W] — "HAPI wraps your AI agent instead of replacing it" | ★5,344, AGPL-3.0, created 2025-12-24; npm `@twsxtd/hapi` 0.34.0 on 2026-11-02 | **Yes**: free web/PWA with push; paid native iOS/Android apps; Telegram Mini App; "Approve AI requests from your phone with one tap" | [README](https://raw.githubusercontent.com/tiann/hapi/main/README.md) |
| **Happy** (slopus) | Mobile, web and desktop client plus relay. The original `happy` CLI wraps Claude Code and Codex (`happy claude`, `happy codex`). The new desktop app runs **Happy Agent**, its own runtime that "picks up the Claude, Codex, and Grok sign-ins already on your machine" and can "hand the agent loop to ... Claude Code or Codex CLI themselves, Gemini through Google Antigravity, OpenCode, or any ACP agent". Subagents can run on any model; sessions are multiplayer | Claude, Codex, Grok, Gemini (via Antigravity), OpenCode, ACP agents | Original CLI = [W]. **Happy Agent runtime = [S]/[O]-adjacent** for Claude, because it uses the Claude sign-in inside a non-Anthropic harness. README: it "never pools or resells provider access, and leaves provider terms and limits in force" | ★24,458, MIT, last push 2026-11-02; npm `happy` 1.2.6 on 2026-10-27 (moved from `happy-coder`, last 1.1.9 on 2026-05-14) | **Yes**: E2E-encrypted iOS/Android/web; APNs/FCM push via its own push server | [README](https://raw.githubusercontent.com/slopus/happy/main/README.md); [happy-agent README](https://raw.githubusercontent.com/slopus/happy-agent/main/README.md) |
| **Nimbalyst** (formerly Crystal; stravu) | Visual desktop workspace: parallel worktree sessions, WYSIWYG editing of agent output, red/green diff review, task trackers | Codex and Claude Code; OpenCode and Copilot in alpha; "a Gemini provider" | [W] (inferred from the "sessions" model; not explicit) | Nimbalyst ★1,997, MIT, created 2025-10-30, last push 2026-10-27. Crystal ★3,130, last push 2026-02-26 ("Crystal is now Nimbalyst") | **Yes**: iOS companion with push, text/voice replies, swipe-through diffs, task queueing | [README](https://raw.githubusercontent.com/nimbalyst/nimbalyst/main/README.md) |
| **Emdash** (generalaction, YC W26) | Desktop ADE: parallel agents in worktrees; remote machines over SSH/SFTP; ingests issues from Linear/GitHub/Jira and others; installs lifecycle hooks for status and notifications | Claude Code, Codex, Cursor, OpenCode, Amp, Devin, Qwen Code, Droid, Copilot (Gemini not listed) | [W] — "detects installed provider CLIs" | ★6,104, Apache-2.0, last push 2026-10-26 | No | [README](https://raw.githubusercontent.com/generalaction/emdash/main/README.md) |
| **Sculptor** (Imbue) | Desktop app for parallel agents in isolated workspaces; bundled workflow skills; experimental Docker/remote backend; "experimental research preview" | Claude Code and the Pi harness integrated; "any terminal-based agents" | [W] | ★242, MIT, last push 2026-10-29 | No | [README](https://raw.githubusercontent.com/imbue-ai/sculptor/main/README.md) |
| **Ruflo** (formerly claude-flow; ruvnet) | "Agent meta-harness for Claude Code and Codex": swarms, memory, routing hooks, MCP server (`npx ruflo mcp start`), TUI console. Claims 314 MCP tools. Requires Claude Code 2.1.287+. "Mods run with your account's permissions and are not sandboxed" | Claude Code, Codex, any MCP client | [W] plus MCP | ★80,523, MIT; npm `claude-flow`/`ruflo` 3.71.0 on 2026-11-01 | No (TUI) | [README](https://raw.githubusercontent.com/ruvnet/ruflo/main/README.md) |
| **Every Code** (just-every/code) | Fork of Codex CLI. `/plan` (Claude, Gemini and GPT consensus), `/solve` (race) and `/code` coordinate multiple CLI agents; Auto Drive | Codex core plus installed `claude`, `gemini`, `qwen` CLIs | Core: "Sign in with ChatGPT" or API key. Others: [W] | ★4,039, Apache-2.0; npm `@just-every/code` 0.6.196 on 2026-09-29 | No | [README](https://raw.githubusercontent.com/just-every/code/main/README.md) |
| **HumanLayer** | Runs coding agents on your machines and lets a team steer tasks, sessions, plans and reviews from desktop, web and a daemon; session-mirroring plugins | Claude Code, Codex, OpenCode, Pi | [W] plus plugins | ★12,762, Apache-2.0, last push 2026-11-02 | Web app | [README](https://raw.githubusercontent.com/humanlayer/humanlayer/main/README.md) |
| **cmux** (manaflow-ai) | Ghostty-based **macOS** terminal with vertical tabs and agent "notification rings"; SSH | Any CLI agent | [W] | ★28,635, created 2026-01-28, last push 2026-11-01 | No (macOS only) | [README](https://raw.githubusercontent.com/manaflow-ai/cmux/main/README.md) |
| **workmux** / **dmux** | Lightweight git worktree plus tmux pane launchers | Any CLI agent | [W] (dmux can also configure an API-key provider) | workmux ★2,939 (last push 2026-10-25); dmux ★1,813 (last push 2026-08-16) | No | [workmux](https://raw.githubusercontent.com/raine/workmux/main/README.md); [dmux](https://raw.githubusercontent.com/standardagents/dmux/main/README.md) |
| **opcode** (formerly Claudia; winfunc) | Tauri GUI for Claude Code sessions, custom agents, usage tracking | Claude Code only | [W] | ★22,421, last push 2026-09-18 | No | [README](https://raw.githubusercontent.com/winfunc/opcode/main/README.md) |
| **Vibe Kanban** (BloopAI) — **SUNSET** | Kanban board to workspaces (branch, terminal, dev server); diff review | Claude Code, Codex, Gemini CLI, Copilot, Amp, Cursor, OpenCode, Droid, CCR, Qwen Code | [W] | ★28,474, Apache-2.0; npm 0.1.44 on 2026-04-24; README says "Vibe Kanban is sunsetting" | Was cloud-backed | [README](https://raw.githubusercontent.com/BloopAI/vibe-kanban/main/README.md); [shutdown post](https://vibekanban.com/blog/shutdown) |
| **Terragon** — **DEAD** | Cloud background-agent orchestrator ("BYO Subscription & API Keys") | Claude Code, Codex, Amp, Gemini | Mixed | Open-source snapshot "at the time of shutdown", 16 Jan 2026; ★259, last push 2026-02-10 | Was web | [README](https://raw.githubusercontent.com/terragon-labs/terragon-oss/main/README.md) |
| **Omnara** — **PIVOTED** | Now an "open-source managed agent platform ... self-hostable alternative to Claude Managed Agents and OpenAI's Agents API". The quickstart uses a provider config `omnara-openrouter` and managed sandboxes | Any model via providers | [K] (OpenRouter/API) | ★3,332, Apache-2.0, last push 2026-10-31; npm `omnara` 1.0.41 on 2026-11-03; the PyPI package is a "Deprecated ... redirect" | Dashboard | [README](https://raw.githubusercontent.com/omnara-ai/omnara/main/README.md); [PyPI](https://pypi.org/pypi/omnara/json) |
| **Uzi** (devflowinc) — **ABANDONED** | Parallel agents in worktrees | — | [W] | ★582, last push 2025-06-04 | — | [ecosyste.ms](https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/devflowinc%2Fuzi) |
| **Tmux-Orchestrator** — **ABANDONED** | tmux orchestrator agent | Claude | [W] | ★1,809, last push 2025-07-14 | — | [ecosyste.ms](https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/Jedward23%2FTmux-Orchestrator) |
| **coder/agentapi** — **DEPRECATED** | HTTP API wrapper over the terminal UIs of Claude Code, Goose, Aider, Gemini, Amp and Codex | (as listed) | [W] | Archived (last push 2026-09-13); README: "AgentAPI is deprecated and no longer maintained" | — | [README](https://raw.githubusercontent.com/coder/agentapi/main/README.md) |

- Vibe Kanban details:
  - Bloop announced the shutdown on 10 April 2026.
  - Remote services ran for 30 days, then moved to a fully local architecture.
  - Removed: kanban issues, comments, projects and organisations.
  - [vibekanban.com/blog/shutdown](https://vibekanban.com/blog/shutdown); [vibecoding.app review](https://vibecoding.app/blog/vibe-kanban-review)
  - One third-party page says v0.1.44 shows "Project functionality has been retired". This is unverified — [agentsroom.dev](https://agentsroom.dev/hi/alternatives/vibe-kanban)
- Discovery list: awesome-claude-code has ★57,099 and was pushed 2026-11-02 — [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code)
- Cross-harness plugin catalogue: wshobson/agents (★42,076) ships one set of plugins/agents/skills for Claude Code, Codex CLI, Cursor, OpenCode, Antigravity CLI, Copilot and Pi — [README](https://raw.githubusercontent.com/wshobson/agents/main/README.md)
- Dagger container-use (★4,080, last push 2026-09-21) gives each agent its own containerised dev environment — [README](https://raw.githubusercontent.com/dagger/container-use/main/README.md)

### Inferences
- **Closest match to "supervisor routes tasks to the cheapest adequate subscription":**
  - **Maestro's** moderator group chat plus quota-aware resend.
  - **oh-my-claudecode's** `omc team N:codex|antigravity` and `/ask`, driven from a Claude Max session.
  - **Gas Town's** Mayor model.

  None of them does automatic cost- or quota-aware routing across vendors. Routing is either chosen by the human or decided by an LLM moderator. A true "cheapest adequate subscription" router would still need custom glue: a small supervisor that reads each CLI's usage status and dispatches through tmux/ACP.
- **For phone steering of a mixed fleet without [O] risk:** HAPI, Agent of Empires (web) and Maestro (web + Cloudflare) all wrap official CLIs and expose phone UIs. Happy's original CLI does too. Happy's newer desktop "Happy Agent" runtime reuses Claude sign-ins outside Claude Code, which, given Anthropic's 2026 stance, is the riskier mode for Max accounts.
- Gas Town and Ruflo are powerful but heavy and opinionated. Maestro, Agent of Empires, ccmanager and HAPI are easier to adopt incrementally.

### Gaps
- **Conductor** (conductor.build, closed-source Mac app) and **Superset** (superset.sh) were not researched in this pass. Agent support and auth are unverified.
- Wandb **Catnip**, **Auto-Claude** and **NanoClaw** could not be located in the index under the names tried.
- GitHub release tags and open-issue counts were unavailable because the GitHub API was blocked.

---

## Q2. Cross-model consultation: how well can Claude delegate cheap work to Codex / Gemini quotas (MCP servers, plugins, skills, subagents)?

### Takeaway
The best-supported path is **OpenAI's own `codex-plugin-cc`**. It installs inside Claude Code and delegates reviews and tasks to the locally logged-in Codex CLI, so usage counts against the ChatGPT plan's Codex limits. It is [W] and vendor-official on the Codex side.

For Gemini, every bridge now has to go through **Antigravity CLI (`agy`)**, whose headless mode is experimental: Flash-only in print mode, and reported to hang without a TTY.

**PAL MCP's `clink`** still works as a generic CLI-to-CLI bridge, but its GitHub repo has not been pushed since December 2025 and it defaults to `gemini` with permission-bypass flags.

### Cited Findings
- **openai/codex-plugin-cc** (★34,767, Apache-2.0, created 2026-03-30, last push 2026-07-08):
  - Commands: `/codex:review`, `/codex:adversarial-review`, and `/codex:rescue`, `/codex:transfer`, `/codex:status`, `/codex:result`, `/codex:cancel` to "delegate work, hand off sessions, and manage background jobs".
  - It adds a `codex:codex-rescue` subagent to `/agents`.
  - Requirements: "ChatGPT subscription (incl. Free) or OpenAI API key. Usage will contribute to your Codex usage limits."
  - Setup: `/plugin marketplace add openai/codex-plugin-cc`, then `/codex:setup`; it uses the local `codex login`.
  - [README](https://raw.githubusercontent.com/openai/codex-plugin-cc/main/README.md)
- **PAL MCP server** (formerly Zen MCP; BeehiveInnovations; ★11,813; last GitHub push 2025-12-15 per ecosyste.ms):
  - `clink` "connect[s] external CLIs like Gemini CLI, Codex CLI, and Claude Code" and supports "CLI Subagents — Claude Code can spawn Codex subagents, Codex can spawn Gemini CLI subagents".
  - Roles such as `planner` and `codereviewer`; custom roles live in `conf/cli_clients/`; `cli_name` defaults to `gemini`.
  - Its other tools (chat/consensus) call model APIs and need API keys.
  - [README](https://raw.githubusercontent.com/BeehiveInnovations/pal-mcp-server/main/README.md); [clink.md](https://raw.githubusercontent.com/BeehiveInnovations/pal-mcp-server/main/docs/tools/clink.md)
- PAL clink caution: "Clink launches real CLI agents with relaxed permission flags (Gemini ships with `--yolo`, Codex with `--dangerously-bypass-approvals-and-sandbox`, Claude with `--permission-mode acceptEdits`)". Its Codex preset runs `codex exec --json --dangerously-bypass-approvals-and-sandbox` — [clink.md](https://raw.githubusercontent.com/BeehiveInnovations/pal-mcp-server/main/docs/tools/clink.md)
- Conflicting maturity signal: PyPI `pal-mcp-server` shows version 11.8.2 uploaded 2026-10-26, while the GitHub repo's last push was 2025-12-15. It is unclear whether the PyPI package is the same project — [PyPI](https://pypi.org/pypi/pal-mcp-server/json); [ecosyste.ms](https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/BeehiveInnovations%2Fpal-mcp-server)
- **gemini-mcp-tool** (★2,293; npm 1.1.8 on 2026-06-18): an MCP server for Claude that shells out to Gemini. It "now defaults to Antigravity CLI (`agy`)". Enterprise and paid-API-key users can set one env var to keep `gemini` — [README](https://raw.githubusercontent.com/jamubc/gemini-mcp-tool/main/README.md)
- **antigravity-intern** (PyPI) is an "MCP server bridging Antigravity CLI (agy) so Claude Code can use it as a sub-agent". Third-party agy bridges note that interactive login cannot be done from print mode, and one reports that headless mode ignores `permissions.allow` — [PyPI antigravity-intern](https://pypi.org/project/antigravity-intern/); [glama listing](https://glama.ai/mcp/servers/LoliLin/memoh-antigravity-cli-mcp)
- Third-party Codex MCP wrappers (for example `@cexll/codex-mcp-server`, `codex-cli-mcp-tool`, `mcp-codex-dev`) expose `codex exec` to Claude Code. They authenticate with an API key or "login with ChatGPT account via `codex login`" — [playbooks.com](https://playbooks.com/mcp/cexll/codex-mcp-server); [glama](https://glama.ai/mcp/servers/@cexll/codex-mcp-server/blob/19fbc1b6441f4f8888cf5b45fcb427a379e05632/docs/concepts/how-it-works.md)
- oh-my-claudecode provides `/ask codex "..."` and `/ask antigravity "..."` as "advisor" calls that Claude synthesises. It removed its own Codex/Gemini MCP servers in v4.4.0 in favour of tmux CLI workers — [README](https://raw.githubusercontent.com/Yeachan-Heo/oh-my-claudecode/main/README.md)
- Maestro's "Cross-Agent Mentions": "Type `@another-agent` in any chat to consult it inline. Maestro forwards the relevant slice of your conversation, runs the other agent in the background" — [README](https://raw.githubusercontent.com/RunMaestro/Maestro/main/README.md)
- steipete/claude-code-mcp ("Claude Code as one-shot MCP server") is **archived** (★1,315, last push 2026-05-15) — [ecosyste.ms](https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/steipete%2Fclaude-code-mcp)
- Codex docs describe Codex as an MCP **client** (config in `config.toml`). The docs navigation lists "App Server", "Remote connections" and "ChatGPT mobile app" pages, but the MCP page did not mention a `codex mcp-server` mode — [learn.chatgpt.com Codex MCP](https://learn.chatgpt.com/docs/extend/mcp?surface=cli)

### Inferences
- **Recommended pattern for this user:** Claude Code on a Max account is the "brain". Codex work (reviews, rescues, bulk edits) goes through `codex-plugin-cc` against the ChatGPT Plus quota. Gemini work goes through an agy bridge (gemini-mcp-tool in agy mode, or an OMC `antigravity` tmux worker) against Google AI Pro. All of these are [W]: each vendor's CLI holds its own login.
- ChatGPT Plus has the smallest quota of the three paid vendors (the quota researcher should confirm). Codex delegation therefore fits targeted reviews better than bulk work.
- Gemini via `agy -p` is currently "Flash-only" in print mode according to gemini-mcp-tool. Delegating to Gemini Pro-class models may require interactive or tmux-driven agy sessions rather than print mode.
- Cross-vendor **reverse** delegation (Codex calling Claude) is possible with PAL clink's `claude` preset or a custom MCP wrapper. That would drive `claude` in non-interactive mode, which falls into the [S] grey zone discussed in Q0.

### Gaps
- `codex mcp-server` (Codex as an MCP server) and `claude mcp serve` (Claude Code as an MCP server) could not be confirmed in the current official docs. The Claude Code MCP reference page retrieved did not mention `mcp serve`. Treat both as unverified in their November 2026 form.
- No benchmark or user study was found on how much quota is actually saved by Claude-to-Codex/Gemini delegation.

---

## Q3. Model routers and multi-provider harnesses: which can use consumer subscriptions, and how?

### Takeaway
The multi-provider harnesses fall into three groups:
- **Use subscriptions only by driving official CLIs over ACP ([W]/[S]):** Goose. It explicitly lets you "use ... your existing Claude, ChatGPT, or Gemini subscriptions via ACP".
- **Have their own subscription OAuth flows ([O] for Anthropic and Google):** OpenCode's community auth plugins, oh-my-openagent's `/login`, Happy Agent.
- **Are API-key gateways ([K]):** Claude Code Router v3, LiteLLM, Crush (plus Charm's own "Hyper" subscription), Cline, Kilo, Aider.

Anthropic subscription OAuth no longer works (and is prohibited) in third-party harnesses. ChatGPT-subscription OAuth in OpenCode is described by its plugin as using "OpenAI's official authentication method". Google's Antigravity OAuth in third-party clients is flagged by the plugins themselves as a ban risk.

### Cited Findings
| Tool | Role | Subscription support / auth | Maturity | Source |
|---|---|---|---|---|
| **OpenCode** (anomalyco/opencode; formerly sst/opencode) | Open-source TUI coding agent, multi-provider | Anthropic OAuth removed after Anthropic legal requests (19 Mar 2026, secondary sources). ChatGPT via the `opencode-openai-codex-auth` plugin ("ChatGPT OAuth → Codex backend"; ★2,207, last push 2026-01-09; mentions a "Legacy OpenCode (v1.0.209 and below)" path). Gemini via `opencode-gemini-auth` (★1,770, last push 2026-09-23). Antigravity via `opencode-antigravity-auth` (★10,951): "Unofficial plugin. Google may restrict accounts using third-party clients." | ★224,160, MIT; npm `opencode-ai` 1.19.5 on 2026-10-28 | [ecosyste.ms](https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/anomalyco%2Fopencode); [codex-auth README](https://raw.githubusercontent.com/numman-ali/opencode-openai-codex-auth/main/README.md); [antigravity-auth README](https://raw.githubusercontent.com/NoeFabris/opencode-antigravity-auth/main/README.md); [falcao.org](https://falcao.org/posts/anthropic-claude-access-crackdown-ecosystem-fallout/) |
| **oh-my-openagent / OmO** (formerly oh-my-opencode; code-yeongyu) | Multi-model harness: "mass ulw" agent graphs, memory | "`/login` covers Claude, ChatGPT, Kimi and GLM subscriptions" — **[O] for Claude** | ★70,435; npm 5.1.59 on 2026-11-03 | [README](https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/main/README.md) |
| **Goose** (aaif-goose/goose; formerly block/goose) | Extensible open-source agent | "15+ providers ... Use API keys or your existing Claude, ChatGPT, or Gemini subscriptions via ACP", i.e. Goose drives the vendors' own agents through ACP adapters | ★56,155, last push 2026-11-03 | [README](https://raw.githubusercontent.com/aaif-goose/goose/main/README.md) |
| **Crush** (Charm) | TUI coding agent | "[Hyper], from Charm, is the official Crush provider. It's subscription-based". Otherwise API keys (Anthropic, OpenAI, Gemini, OpenRouter...) | ★28,849; npm `@charmland/crush` 0.99.0 on 2026-10-29 | [README](https://raw.githubusercontent.com/charmbracelet/crush/main/README.md) |
| **Claude Code Router (CCR)** (musistudio) | v3 is now "a local model gateway and control plane for coding agents": desktop app and `ccr ui`; gateway on 127.0.0.1:3456; routing rules, retries, "credential pools, key rotation, and ordered fallback models"; logs and cost estimates | Clients: Claude Code, Claude Design, Codex, Grok CLI, Kimi CLI, Kilo Code, OpenCode, Pi, ZCode, WorkBuddy. Providers are added by **API key** (OpenAI, Anthropic Messages, Gemini, OpenRouter, DeepSeek, Moonshot, Mistral, Z.AI...) — [K] | ★37,868, MIT; npm 3.1.3 on 2026-10-09 | [README](https://raw.githubusercontent.com/musistudio/claude-code-router/main/README.md) |
| **LiteLLM** (BerriAI) | "AI Gateway. Rust core with Python SDK. Call 100+ LLM APIs"; load balancing and cost tracking | [K] | ★62,137; PyPI 1.106.6 on 2026-10-28 | [ecosyste.ms](https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/BerriAI%2Flitellm); [PyPI](https://pypi.org/pypi/litellm/json) |
| **Cline** | IDE extension, SDK and CLI | Providers include Google (Gemini), OpenRouter, Vercel AI Gateway and others. Blocked from Claude subscription OAuth on 9 Jan 2026 (secondary) | ★71,069; npm `cline` 3.0.74 on 2026-10-26 | [README](https://raw.githubusercontent.com/cline/cline/main/README.md); [VentureBeat](https://venturebeat.com/ai/anthropic-cracks-down-on-unauthorized-claude-usage-by-third-party-harnesses/) |
| **Kilo Code** | "All-in-one agentic engineering platform" (IDE and CLI) | Auth specifics not extracted | ★27,765, MIT; npm `@kilocode/cli` 7.8.18 on 2026-10-26 | [ecosyste.ms](https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/Kilo-Org%2Fkilocode) |
| **Roo Code** — **ARCHIVED** | VS Code agent | Repo archived; last push 2026-05-15. The team moved to Roomote, a Slack-based cloud agent; Roo Code Cloud and Router shut down 15 May 2026; Cline is the recommended migration path | ★24,252 | [ecosyste.ms](https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/RooCodeInc%2FRoo-Code); [kilo.ai](https://kilo.ai/compare/roo-code-shutdown-roomote); [rywalker.com](https://rywalker.com/research/roo-code) |
| **Aider** | Terminal pair programmer | [K] | ★49,771; PyPI 0.86.2 on 2026-02-12; last push 2026-05-22 (slowing) | [PyPI](https://pypi.org/pypi/aider-chat/json) |
| **OpenHands** | Autonomous dev agent platform | Not examined | ★92,572, last push 2026-11-03 | [ecosyste.ms](https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/OpenHands%2FOpenHands) |
| **CC Switch** (farion1231) | Desktop "All-in-One assistant" that switches the provider config of Claude Code, Codex, OpenCode, OpenClaw, Grok Build and Hermes Agent in one click; manages MCP, skills and prompts | Edits each CLI's provider config (API endpoints/keys). README is sponsored by relay resellers advertising "Claude Code / Codex / Gemini official channels at 38% / 2% / 9% of original price" | ★149,866, MIT, last push 2026-10-29 | [README](https://raw.githubusercontent.com/farion1231/cc-switch/main/README.md) |

- Pi (`@mariozechner/pi-coding-agent`, npm 0.73.1 on 2026-05-07) appears as a supported harness in Sculptor, HAPI, HumanLayer, CCR, Agent Deck and others — [npm](https://registry.npmjs.org/@mariozechner%2fpi-coding-agent)

### Inferences
- For a user who wants subscription-backed official CLIs, these "routers" are mostly the wrong tool:
  - CCR and LiteLLM route **API-key** traffic. Pointing Claude Code at CCR means those requests no longer use the Max plan.
  - OpenCode and OmO use **[O]** OAuth for Claude, which Anthropic now blocks and prohibits.
- **Goose-over-ACP** is the one multi-provider harness that reaches all three subscriptions through vendor-owned agents. It is the cleanest "single harness" option, at the cost of Goose being the outer loop.
- CC Switch's popularity is mostly a China-market phenomenon of switching between relay resellers (inferred from sponsor content). It is not an orchestration plane.

### Gaps
- OpenRouter was not examined. It is API-credit based, so it is out of scope for subscription use.
- Kilo Code's and Crush's current subscription/OAuth options were not fully extracted.
- Whether OpenCode now ships native ChatGPT-subscription login (rather than via the plugin) is implied by the plugin's "Legacy OpenCode (v1.0.209 and below)" note but not confirmed.

---

## Q4. Subscription-to-API proxies and multi-account load balancers

### Takeaway
These tools turn subscription logins into an OpenAI/Anthropic/Gemini-compatible API, often with multi-account round-robin. Examples: CLIProxyAPI, better-ccflare, claude-relay-service, antigravity-claude-proxy. They are all [O] by construction, and several READMEs warn of account bans themselves. For this user's two Max accounts, a "load-balance across OAuth accounts" proxy is the single highest-risk pattern in this catalogue. CLIProxyAPI is the only tool found that claims support for **Meta Muse Code** accounts.

### Cited Findings
- **CLIProxyAPI** (router-for-me):
  - Maturity: ★57,273, MIT, last push 2026-11-02.
  - What it does: "Wrap Antigravity, ChatGPT Codex, Claude Code, Grok Build, Muse Code, Devin as an OpenAI/Gemini/Claude/Codex compatible API service".
  - Features: "Claude Code support via OAuth login", "OpenAI Codex support ... via OAuth login", "Multiple accounts with round-robin load balancing (Gemini, OpenAI, Claude, Grok)", "Claude Code multi-account load balancing".
  - Meta: "Muse Code accounts through Meta login and Meta Model API keys".
  - Its sponsors include relay resellers and residential-proxy vendors "built for automation and multi-account operations".
  - [README](https://raw.githubusercontent.com/router-for-me/CLIProxyAPI/main/README.md)
- **better-ccflare** (tombii):
  - Maturity: ★274; npm 3.6.0 on 2026-11-03.
  - "Load balancer proxy for Claude API with intelligent distribution across multiple OAuth accounts". Claims "Zero Rate Limit Errors — Automatically distribute requests across multiple accounts". Supports Claude OAuth and Codex (OpenAI OAuth), among others.
  - The original snipeship/ccflare (★1,053) was last pushed 2026-04-19.
  - [README](https://raw.githubusercontent.com/tombii/better-ccflare/main/README.md); [npm](https://registry.npmjs.org/better-ccflare)
- **claude-relay-service** (Wei-Shaw):
  - Maturity: ★12,768, last push 2026-10-31.
  - Self-hosted relay with multi-account Claude OAuth rotation and "拼车共享" (carpool sharing).
  - Its README warns: "使用本项目可能违反Anthropic的服务条款" ("using this project may violate Anthropic's ToS") and disclaims "账户封禁" (account bans).
  - [README](https://raw.githubusercontent.com/Wei-Shaw/claude-relay-service/main/README.md)
- **antigravity-claude-proxy** (badrisnarayanan):
  - Maturity: ★4,032, last push 2026-11-02.
  - Exposes Antigravity's Claude and Gemini models to Claude Code and OpenClaw.
  - Its README warns: "Google has been issuing ToS violation bans on accounts connected to this proxy" and recommends "Do not use your main account. Use a burner account".
  - [README](https://raw.githubusercontent.com/badrisnarayanan/antigravity-claude-proxy/main/README.md)
- **claude-code-proxy** (1rgs; ★3,764, last push 2026-06-23) and fuergaosi233/claude-code-proxy (★2,798, last push 2026-03-12) go the other direction: they run Claude Code against OpenAI-format APIs using API keys — [1rgs README](https://raw.githubusercontent.com/1rgs/claude-code-proxy/main/README.md)

### Inferences
- These proxies would technically let one supervisor treat all subscriptions as interchangeable API endpoints, which is the user's "route to the cheapest adequate subscription" dream. Every major vendor in scope is now enforcing against exactly this, so the proxies are a poor foundation for a durable personal setup. The ToS researcher should give the definitive risk read.
- Pooling two Max 20x accounts behind one endpoint (better-ccflare, CLIProxyAPI, claude-relay-service) is a distinct, higher-risk pattern than running two official `claude` CLIs side by side under separate logins.

### Gaps
- No official Meta "Muse Code" CLI or its auth model was researched. Whether the user's Muse free tier has any sanctioned programmatic or CLI path is unknown from this pass.

---

## Q5. Mobile front-ends: steering agents from a phone (push notifications, approve/deny)

### Takeaway
For Claude, **Anthropic's own Remote Control** is now the strongest option. It is official, works only with subscription logins, offers server mode with worktree spawning, sends push notifications for permission prompts, and holds permission prompts open until they are answered. For a mixed Claude/Codex/Gemini fleet on one phone, the leading open-source options are:
- **HAPI** — wrapper, PWA/native apps, one-tap approvals, Telegram.
- **Happy** — E2E encrypted, push.
- **Nimbalyst iOS** — push, voice replies.
- **Agent of Empires** and **Maestro** — web dashboards.
- **Agent Deck** — Telegram.

### Cited Findings
- **Claude Code Remote Control (official)**:
  - Connects claude.ai/code or the Claude iOS/Android app to a Claude Code session "running on your machine".
  - "Available on Pro, Max, Ultra, Team, and Enterprise plans. API keys are not supported."
  - Three ways to start it: `claude remote-control` (server mode), `claude --remote-control`/`--rc` (interactive), and `/remote-control` (inside a session).
  - Server-mode flags include `--spawn worktree` (each on-demand session gets its own git worktree), `--capacity <N>` (default 32 concurrent sessions) and `--permission-mode`.
  - [code.claude.com/docs/en/remote-control](https://code.claude.com/docs/en/remote-control)
- Remote Control push and approvals: "When Remote Control is active, Claude can send push notifications to your phone". `/config` → "Push when actions required" covers permission prompts and questions. "Claude Code keeps permission prompts and `AskUserQuestion` questions open until you answer them" — [code.claude.com/docs/en/remote-control](https://code.claude.com/docs/en/remote-control)
- Remote Control networking and limits:
  - "Your local Claude Code session makes outbound HTTPS requests only and never opens inbound ports".
  - The transcript is stored on Anthropic servers while connected.
  - Server mode "gives up after roughly 10 minutes" of network outage and exits.
  - [code.claude.com/docs/en/remote-control](https://code.claude.com/docs/en/remote-control)
- A reported (now closed) bug: approving a permission prompt from the phone did not release the prompt on the host (April 2026) — [claudeissues.com #52084](https://claudeissues.com/issue/52084-remote-control-approving-permission-from-phone-hangs-claude-code-on-host)
- Claude Code "channels": an MCP server "can also act as a channel that pushes messages into your session, so Claude reacts to Telegram messages, Discord chats" — [Claude Code MCP docs](https://code.claude.com/docs/en/mcp.md)
- **HAPI**:
  - Free web/PWA "for a home-screen app and push notifications"; paid native iOS/Android apps with "chat, approvals, session creation, files, dictation, and native push notifications"; Telegram Mini App.
  - "The relay uses WireGuard + TLS for end-to-end encryption". Self-hosting via Cloudflare Tunnel or Tailscale.
  - [README](https://raw.githubusercontent.com/tiann/hapi/main/README.md)
- **Happy**: an "End-to-end encrypted mobile app" (iOS, Android, web); a push server delivers via APNs/FCM; multiplayer steering — [README](https://raw.githubusercontent.com/slopus/happy/main/README.md)
- **Nimbalyst iOS companion**: "See which agents need you, reply by text or voice, swipe through diffs, queue the next task, and get a push when an agent is waiting" — [README](https://raw.githubusercontent.com/nimbalyst/nimbalyst/main/README.md)
- **Maestro**: "Built-in web server with QR code access. Monitor and control all your agents from your phone ... remote tunneling via Cloudflare" — [README](https://raw.githubusercontent.com/RunMaestro/Maestro/main/README.md)
- **Agent of Empires**: web dashboard with a "Phone: structured view" — [README](https://raw.githubusercontent.com/agent-of-empires/agent-of-empires/main/README.md)
- **Agent Deck**: Telegram bot conductor "that watches every Claude session you have running" — [README](https://raw.githubusercontent.com/asheshgoplani/agent-deck/main/README.md)
- **CloudCLI (formerly Claude Code UI)**: siteboon/claudecodeui (★14,305, AGPL-3.0) is a "desktop and mobile UI for Claude Code, Cursor CLI, and Codex". The npm package moved to `@cloudcli-ai/cloudcli` — [README](https://raw.githubusercontent.com/siteboon/claudecodeui/main/README.md); [npm](https://registry.npmjs.org/@siteboon%2fclaude-code-ui)
- **Claude-Code-Remote** (JessyTsui; ★1,287; last push 2025-12-06, going stale) controls Claude Code via email, Telegram, LINE and Discord through hooks — [README](https://raw.githubusercontent.com/JessyTsui/Claude-Code-Remote/main/README.md)
- **VibeTunnel** (★4,701; last push 2026-08-05; npm still 1.0.0-beta.15.1 from 2025-08-02) turns a browser into a terminal for agents — [README](https://raw.githubusercontent.com/amantus-ai/vibetunnel/main/README.md)
- Codex: the docs navigation includes "Remote connections" and "ChatGPT mobile app" pages for Codex. Their content was not retrieved — [learn.chatgpt.com](https://learn.chatgpt.com/docs/extend/mcp?surface=cli)

### Inferences
- **Suggested layering for this user:**
  - Claude Max sessions: native Remote Control, the most ToS-safe option, with push and approvals.
  - Codex and Antigravity sessions: one wrapper (HAPI or Agent of Empires) on the same VPS, reached through its PWA/web over Tailscale or a Cloudflare tunnel.
  - SSH (Termius/Blink) plus tmux/mosh stays as the universal fallback; every tmux-based supervisor in Q1 can be attached directly.
- Remote Control's `--spawn worktree --capacity N` server mode overlaps with many third-party Claude-only supervisors for phone use. It is Claude-only, though, and cannot steer Codex or Gemini sessions.

### Gaps
- Termius, Blink, mosh and Tailscale specifics (push support and so on) were not researched in this pass.
- Codex's own remote/mobile control features, and whether they can approve local Codex CLI prompts from the ChatGPT app, were not retrieved.
- Whether Remote Control works headless on a Linux VPS was not explicitly confirmed. The docs describe outbound-only HTTPS, which suggests it does, but this was not verified.

---

## Q6. Agent-to-agent protocols — A2A, MCP, ACP: relevance for a personal setup

### Takeaway
**ACP (Agent Client Protocol)** is the most practically useful protocol for this user. Official or semi-official ACP adapters exist for Claude (`claude-agent-acp`) and Codex (`codex-acp`, which supports ChatGPT subscriptions). Clients such as Zed, JetBrains, Toad, Goose and Happy can drive them. So ACP is a standard way for one front-end or supervisor to talk to all vendors' agents.

**MCP** remains the tool/consultation layer (Q2). **A2A** targets networked, opaque agent services and offers little for a single-user, CLI-centric setup.

### Cited Findings
- ACP "standardizes communication between code editors ... and coding agents". Maturity: ★4,543, Apache-2.0, last push 2026-10-30. It has Rust crates and a Kotlin SDK, and the site lists agents and clients — [agent-client-protocol README](https://raw.githubusercontent.com/agentclientprotocol/agent-client-protocol/main/README.md); [agentclientprotocol.com](https://agentclientprotocol.com/)
- **claude-agent-acp**: "Use Claude Agent SDK from any ACP-compatible client". Maturity: ★2,719; npm 0.100.0 on 2026-11-02. It supports client MCP servers, and a client that declares `clientCapabilities.subagents` "gets each Agent or Task subagent as its own child session". It replaced `@zed-industries/claude-code-acp`, whose last version was 0.16.2 on 2026-02-17 — [README](https://raw.githubusercontent.com/agentclientprotocol/claude-agent-acp/main/README.md); [npm](https://registry.npmjs.org/@agentclientprotocol%2fclaude-agent-acp)
- **codex-acp**: "ACP server implementation that exposes Codex CLI functionality". Development moved from zed-industries (now archived) to agentclientprotocol/codex-acp (★496, last push 2026-10-29). It supports "ChatGPT subscription (requires paid subscription and doesn't work in remote projects)" — [zed codex-acp README](https://raw.githubusercontent.com/zed-industries/codex-acp/main/README.md); [ecosyste.ms](https://repos.ecosyste.ms/api/v1/hosts/GitHub/repositories/agentclientprotocol%2Fcodex-acp)
- ACP clients and consumers:
  - Goose uses ACP to reach Claude, ChatGPT and Gemini subscriptions — [Goose README](https://raw.githubusercontent.com/aaif-goose/goose/main/README.md)
  - Happy plugs in "any Agent Client Protocol agent" as a harness — [Happy README](https://raw.githubusercontent.com/slopus/happy/main/README.md)
  - Toad (Will McGugan's TUI; ★3,498, AGPL-3.0, last push 2026-05-26) runs "dozens of agents" through ACP — [Toad README](https://raw.githubusercontent.com/batrachianai/toad/main/README.md)
  - JetBrains AIR is named as an ACP client in the claude-agent-acp README.
- **A2A** is "an open protocol enabling communication and interoperability between opaque agentic applications". It uses JSON-RPC 2.0 over HTTP(S), "Agent Cards" for discovery, and Python/Go/JS SDKs, and is positioned as complementing MCP. Maturity: ★26,348, Apache-2.0, last push 2026-10-30 — [A2A README](https://raw.githubusercontent.com/a2aproject/A2A/main/README.md)

### Inferences
- **Auth nuance:** claude-agent-acp runs on the **Claude Agent SDK**, not the interactive `claude` binary. Using it with a Max login inside a third-party client is the [S] grey zone; OpenClaw's docs note that Agent SDK usage "draw[s] from the signed-in subscription's limits". codex-acp wraps Codex CLI functionality and explicitly supports ChatGPT subscriptions.
- A custom "orchestration plane" could speak ACP southbound to each vendor agent, getting structured events, permission requests and subagent sessions instead of scraping tmux. Codex is the cleanest fit; Claude is a grey zone (see above). Gemini/agy has no confirmed ACP adapter.
- A2A only becomes relevant if the user exposes agents as network services to other agents, for example hosted vendor agents. It is overkill for a phone-steered personal fleet.

### Gaps
- An ACP adapter for Antigravity CLI (`agy`) was not found or confirmed.
- A2A governance (for example a Linux Foundation home) and version status were not extracted in this pass.

---

## Q7. Personal-assistant agent frameworks (OpenClaw and peers) on a VPS via chat apps — how they handle multiple model subscriptions

### Takeaway
OpenClaw (★393k) is the dominant self-hosted, chat-app-fronted assistant. It offers three Anthropic paths:
- **API key** — recommended by OpenClaw.
- **`setup-token`** — the subscription OAuth token, which advertises itself as a Claude Code client; [O].
- **Claude CLI backend** — runs `claude -p` against the existing login; [S]/[W].

OpenAI Codex is handled through OpenClaw-managed OAuth. Since April 2026, Anthropic has blocked subscription OAuth in OpenClaw (secondary sources). The peers (Hermes Agent, nanobot, ZeroClaw, PicoClaw) are API-key/provider-centric.

### Cited Findings
- **OpenClaw**:
  - Maturity: ★393,543, MIT, created 2025-11-24, last push 2026-10-31; npm `openclaw` 2026.10.10 on 2026-11-03.
  - Channels: "Discord, iMessage, Slack, Teams, Telegram, WhatsApp, and 20+ more, plus native apps for macOS, iOS, Android".
  - "Models and agent harnesses (Claude, Codex, local models) are plugins you can swap".
  - [README](https://raw.githubusercontent.com/openclaw/openclaw/main/README.md); [npm](https://registry.npmjs.org/openclaw)
- OpenClaw's three Anthropic auth options — [docs.openclaw.ai/providers/anthropic](https://docs.openclaw.ai/providers/anthropic):
  - **API key** (pay-as-you-go).
  - **setup-token**: `claude setup-token` prints a long-lived `sk-ant-oat01-` token. "Setup-token requests advertise a maintained Claude Code client version", and Anthropic uses that identity to gate newer models.
  - **Claude CLI backend (`claude -p`)**: reuses an existing Claude Code login on the same host; OpenClaw "does not read, persist, or refresh the native login tokens".
  - The docs recommend an API key for production and avoiding copying native OAuth tokens.
- OpenClaw lists OpenAI Codex as a subscription-style provider with OpenClaw-managed OAuth, recommended for containers where Claude CLI reuse is not available — [docs.openclaw.ai/providers/anthropic](https://docs.openclaw.ai/providers/anthropic)
- Anthropic blocked subscription OAuth in third-party harnesses "starting with OpenClaw" around 4 April 2026, with users pushed to pay-as-you-go "extra usage" (secondary sources) — [lilting.ch](https://lilting.ch/en/articles/anthropic-claude-code-openclaw-third-party-paygo); [implicator.ai](https://www.implicator.ai/anthropic-cuts-openclaw-from-claude-subscriptions-citing-unsustainable-compute-costs/)
- antigravity-claude-proxy explicitly targets "Claude Code and OpenClaw (Clawdbot)" as consumers of Antigravity-provided models, with Google ban warnings — [README](https://raw.githubusercontent.com/badrisnarayanan/antigravity-claude-proxy/main/README.md)
- **Hermes Agent** (NousResearch):
  - Maturity: ★259,770, MIT, last push 2026-11-02; PyPI 0.19.0 on 2026-07-20.
  - Telegram, Discord, Slack, WhatsApp, Signal and CLI from one gateway.
  - "Spawn isolated subagents ... choose their provider, model, and limits".
  - Providers: Nous Portal, OpenRouter, OpenAI, custom endpoints.
  - [README](https://raw.githubusercontent.com/NousResearch/hermes-agent/main/README.md)
- **nanobot** (HKUDS): ★49,507, MIT, created 2026-02-01. A Python personal agent with Telegram, Discord, Slack, WeChat, Email and other channels; providers and fallback models are configured; "nanobot does not supply" model access — [README](https://raw.githubusercontent.com/HKUDS/nanobot/main/README.md)
- **ZeroClaw**: ★33,095, created 2026-02-13. A single Rust binary with about 20 providers (Anthropic, OpenAI, Ollama...), 30+ channels, and "fallback chains and routing" — [README](https://raw.githubusercontent.com/zeroclaw-labs/zeroclaw/main/README.md)
- **PicoClaw** (Sipeed): ★30,077, created 2026-02-04. Written in Go; "30+ LLM providers" via API keys in `model_list`; v0.2.4 on 2026-03-25 added model routing and MCP — [README](https://raw.githubusercontent.com/sipeed/picoclaw/main/README.md)

### Inferences
- For a subscription-only user, OpenClaw's **Claude CLI backend** (`claude -p` on the same VPS) and **Codex OAuth** are its only subscription paths, and the Claude one sits in the [S] grey zone. Its `setup-token` path is [O]. OpenClaw makes a good **chat-app front door** ("text my VPS from WhatsApp") but a weak quota router.
- A safer composition: keep the official CLIs as executors (Q1 supervisors and Remote Control) and use a chat-app bridge only for notifications and commands. Examples are Agent Deck's Telegram conductor, HAPI's Telegram Mini App, or Claude Code "channels". This avoids handing subscription credentials to an assistant framework.

### Gaps
- OpenClaw's current Google/Gemini/Antigravity provider support was not covered by the page retrieved.
- Whether Hermes or nanobot have added sanctioned subscription logins (for example ChatGPT/Codex OAuth or Copilot) was not extracted.
- NanoClaw (a Claude-Agent-SDK-in-containers OpenClaw alternative) could not be located in the index under the names tried.
