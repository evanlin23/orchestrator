# Vendor-hosted cloud coding agents and cross-vendor handoff (as of 2026-11-03)

Scope: what each vendor's own hosted agent can do for orchestration (triggers, schedules, inter-session messaging, GitHub, MCP, webhooks, mobile), and how to wire them together with hosted services only (no home server). Quotas, ToS, open-source tools and self-hosting are covered by other researchers.

Provenance note: items marked "[First-hand]" come from the tool schemas exposed to this research agent, which itself runs as a Claude Code cloud session on claude.ai (observed 2026-11-03). They are direct observation of the current product surface, not a web page, and should be described that way in the report.

**Breaking items the report writer must not miss:**
1. **Google Jules is being shut down.** It was announced Oct 29, 2026. Jules goes read-only after **Nov 11, 2026** and shuts down on **Dec 31, 2026**. Google points users to Antigravity ([Jules changelog](https://jules.google/docs/changelog/)).
2. **Gemini CLI no longer serves consumer subscriptions.** Free individual access and Google AI Pro/Ultra moved to the **Antigravity CLI on June 18, 2026** ([gemini-cli README](https://github.com/google-gemini/gemini-cli)).
3. **GitHub Agent HQ bills through Copilot, not your vendor subscriptions.** It runs Claude and Codex as agents, but they are "powered by your existing Copilot subscription" and use AI credits plus Actions minutes. The docs say nothing about bringing your own Claude Max or ChatGPT Plus ([GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents)).

---

## Q1. Claude Code on the web / cloud sessions: spawning, messaging, scheduled tasks, GitHub, network, MCP, mobile, cross-account

### Takeaway
Claude Code cloud is the most orchestration-capable hosted agent of the group:
- **Inside a session:** it can spawn and message sibling sessions, schedule itself, subscribe to PR events, and change its own network allowlist (each change needs user approval).
- **From outside:** scheduled tasks (formerly "routines") can be fired by cron, by a per-task HTTP `/fire` endpoint with a bearer token, or by GitHub PR, issue and release events.

All native session-to-session features are confined to one claude.ai account. The per-task `/fire` bearer token is the only documented native bridge to a second account.

### Cited Findings

**Surfaces, mobile and plans**
- Cloud sessions are available on Pro, Max, Ultra and Team, and for Enterprise users with premium or Chat + Claude Code seats. They count toward the plan's usage limits, with "no separate charge for the cloud VM" ([Claude Code docs: Use Claude Code in the cloud](https://code.claude.com/docs/en/claude-code-on-the-web)).
- Sessions can start from:
  - the browser (claude.ai/code)
  - the **Code tab in the Claude mobile app**
  - the Desktop app
  - the terminal (`claude --cloud`)
  - cloud scheduled tasks, where each run is a cloud session

  Sessions keep running after the laptop closes and can be steered from any device ([same](https://code.claude.com/docs/en/claude-code-on-the-web)).
- `claude -p "msg" --cloud <session-id>` queues a message into an existing cloud session from any machine logged in with `claude auth login`. It returns JSON `{ok, session_id, url}` with `--output-format json`, and the docs mention use from CI scripts ([same](https://code.claude.com/docs/en/claude-code-on-the-web)).
- **Teleport** (`claude --teleport`) requires "Same account: You must be authenticated to the same claude.ai account used in the cloud session" ([same](https://code.claude.com/docs/en/claude-code-on-the-web)).
- **Remote Control is not a server-free option.** It lets the phone steer a session running locally on the user's own machine, so it needs a machine that is on ([same](https://code.claude.com/docs/en/claude-code-on-the-web); [Projects docs](https://code.claude.com/docs/en/claude-projects)).
- Subagents work in cloud sessions. Experimental "agent teams" can be enabled with `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` as an environment variable ([same](https://code.claude.com/docs/en/claude-code-on-the-web)).

**Projects (built-in orchestrator)**
- Projects are in **public beta** on Pro, Max and Ultra, rolling out gradually, and not on Team or Enterprise. A project is "one ongoing conversation where Claude coordinates a stream of related work": Claude starts a parallel cloud-session "thread" per task, routes follow-ups, and tracks PRs ([Projects docs](https://code.claude.com/docs/en/claude-projects)).
- Projects can be created and used in the **Claude mobile app for iOS and Android** ([same](https://code.claude.com/docs/en/claude-projects)).
- The Overview pane has PR buttons ("Resolve conflicts", "Fix CI", "Address comments", "Merge it") and a Scheduled tasks tab ([same](https://code.claude.com/docs/en/claude-projects)).
- A thread limit you state in conversation is "not a hard cap" ([same](https://code.claude.com/docs/en/claude-projects)).
- Project threads need the Claude GitHub App installed on each repo; a `/web-setup` token is not enough ([same](https://code.claude.com/docs/en/claude-projects)).
- In project threads, repo `.mcp.json` and `.claude/skills/` are **not loaded**. Use account-level claude.ai connectors and skills instead ([same](https://code.claude.com/docs/en/claude-projects)).

**Inter-session tools available inside a cloud session** [First-hand: this session's tool schema, 2026-11-03]
- `create_session`: spawn a new cloud session with a prompt, repo, branch and environment.
- `send_message`: message another session. It arrives "marked as sent by this session", and the target treats it as peer data.
- `list_sessions`, `get_session` and `list_events`: read another session's transcript and status.
- `interrupt_session` and `archive_session`.
- `create_trigger`, which has three modes: fire into this session, fire into a named other session, or start a fresh session each time. The minimum interval is normally hourly.
- `fire_trigger`, `send_later` (one-shot reminder back into the same session) and `subscribe_pr_activity` (PR comments, reviews and CI results delivered into the conversation; one session per PR at a time).
- `watch_url`: an inbound webhook, but its signing secret is sealed to the *artifact service* only, so it is **not** a general-purpose webhook for third parties.
- `request_credential`: store an API key as a per-host network secret that the proxy injects, so the value never enters the VM.
- `get_environment`, `update_environment`, `create_environment`: change the allowlist or setup script. Every change shows the user an approval prompt.
- `request_computer_folder`: use a folder on the user's computer through Remote Control.

**Account scoping** [First-hand: tool schema]
- Persistent-session targets for triggers "must belong to the same account — the server rejects sessions you don't own".
- `list_sessions` covers sessions "visible to the authenticated account".
- No tool reaches sessions in a different claude.ai account.

**Scheduled tasks (formerly routines)**
- They are in **research preview**. Each one is a saved prompt plus repos, connectors and an environment, and runs on Anthropic infrastructure "when your laptop is closed" ([Scheduled tasks docs](https://code.claude.com/docs/en/routines)).
- Triggers are **Schedule** (cron, minimum hourly, or one-off), **API**, and **GitHub event**. One task can combine several triggers. Available on Pro, Max, Ultra, Team and Enterprise. Create them at claude.ai/code/routines, in the Desktop app, or with `/schedule` in the CLI ([same](https://code.claude.com/docs/en/routines)).
- **API trigger:**
  - Request: `POST https://api.anthropic.com/v1/claude_code/routines/<trig_id>/fire` with `Authorization: Bearer <per-task token>`, header `anthropic-beta: experimental-cc-routine-2026-04-01`, and optional body `{"text": "..."}`.
  - Response: `claude_code_session_id` and `claude_code_session_url`.
  - The token is scoped to triggering that one task and is shown once. API triggers can only be managed on the web, not the CLI.
  - Fire text arrives wrapped as untrusted `<routine-fire-payload>` data, so the saved prompt must explicitly opt in to acting on it.
  - Source: [same](https://code.claude.com/docs/en/routines).
- **GitHub trigger:**
  - Events: Pull request (opened, closed, assigned, labeled, synchronized and others), Release, and **Issue** (opened, edited, closed, reopened, labeled, assigned and others).
  - PR filters: author, title, body, base and head branch, labels, draft state.
  - Needs the Claude GitHub App on the repo. `/web-setup` "does not enable webhook delivery".
  - Every event starts a new, independent session ("doesn't reuse sessions across events").
  - "Per-task and per-account hourly caps", and "events beyond the limit are dropped".
  - Source: [same](https://code.claude.com/docs/en/routines).
- **Hourly limits** (none has overage):

  | Action | Limit | Counted per |
  | :- | :- | :- |
  | Scheduled runs | 100/hour | account |
  | Run now + API fires + re-arm, combined | 30/hour | task |
  | API fires | 100/hour | account |
  | Run now | 100/hour | account |

  Source: [same](https://code.claude.com/docs/en/routines).
- **Run behavior:**
  - Runs start in **auto mode**, where a classifier approves actions, and may stop for permission prompts on repeated blocks.
  - Runs push to `claude/`-prefixed branches by default.
  - Scheduled tasks belong to the individual account, and everything appears as the user's GitHub and connector identity.
  - Source: [same](https://code.claude.com/docs/en/routines).

**Network, environment and MCP**
- The Default environment uses **Trusted** network access: a default allowlist of package registries, cloud provider APIs, container registries and common dev domains. Other hosts get `403` with `x-deny-reason: host_not_allowed`. Change it to **Custom** (your own allowed domains, optionally plus the defaults) or **Full** (unrestricted) ([Scheduled tasks docs](https://code.claude.com/docs/en/routines); [cloud docs](https://code.claude.com/docs/en/claude-code-on-the-web)).
- MCP connectors are routed through Anthropic's servers and need no allowlist entries ([routines docs](https://code.claude.com/docs/en/routines)).
- GitHub credentials never enter the VM; a GitHub proxy attaches them server-side ([cloud docs](https://code.claude.com/docs/en/claude-code-on-the-web)).
- Network secrets stay outside the sandbox and are attached to matching requests ([same](https://code.claude.com/docs/en/claude-code-on-the-web)).
- The connectors available are the claude.ai integrations on the account (claude.ai/customize/connectors, for example Slack, Linear, Google Drive). MCP servers added locally with `claude mcp add` don't carry over, but a committed `.mcp.json` works for single-repo tasks ([routines docs](https://code.claude.com/docs/en/routines)).

**PR auto-fix**
- Claude subscribes to PR activity and reacts to CI failures and review comments. This needs the Claude GitHub App.
- It can be turned on from the mobile app by saying "watch this PR and fix any CI failures or review comments".
- It cannot react to merge conflicts, because GitHub emits no webhook for them.
- Source: [cloud docs](https://code.claude.com/docs/en/claude-code-on-the-web).
- **Important for cross-vendor triggering:** Claude's replies on GitHub "are posted using your GitHub account… labeled as coming from Claude Code". The docs warn they "can trigger" `issue_comment` workflows ([same](https://code.claude.com/docs/en/claude-code-on-the-web)).

**Claude GitHub Action (`anthropics/claude-code-action`)**
- It runs on your own GitHub runner and responds to @claude mentions, issue assignments, or explicit prompts ([repo README](https://github.com/anthropics/claude-code-action)).
- Authentication can be `anthropic_api_key` **or** `claude_code_oauth_token` ([security.md](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md)).
- Only users with write access can trigger it.
- GitHub Apps and bots cannot trigger it by default. `allowed_bots` enables named bots or `*`, and allowed bots skip permission checks.
- `exclude_comments_by_actor` filters bot comments. **No loop-prevention mechanism is documented** ([same](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md)).

### Inferences
- **Native fan-out within one account is fully hosted.** A "dispatcher" session or Project on account A can `create_session` and `send_message` to workers and use `subscribe_pr_activity` to wait for results. This needs no server and works from the phone.
- **Cross-account (Max account A to Max account B).**
  - *Simplest route:* give B a scheduled task with an **API trigger**, and store B's `/fire` token as a GitHub Actions secret in a shared repo. A workflow on `issues: labeled` (label `claude-b`) curls B's `/fire` with the issue body as `text`.
  - *Fully native alternative:* give B's scheduled task a **GitHub Issue-labeled trigger** on the same repo, with the Claude GitHub App installed. However, only PR triggers have documented filters, so an issue-labeled task may fire on every label and must exit early in its prompt when the label doesn't match. That spends a run, and runs count against the caps.
  - *From inside A's session:* calling B's `/fire` directly would mean exposing B's token to A's VM. It is unclear whether `request_credential` can inject a header for `api.anthropic.com` without clashing with the session's own API traffic. Untested.
- **Running other vendors' CLIs inside a Claude cloud VM is technically plausible.**
  - Install the Codex CLI, Gemini CLI or Antigravity CLI with npm; registries are on the default allowlist.
  - Add the vendor API hosts under Custom network access (each change needs an approval prompt).
  - Inject an API key as a network secret, for example `x-goog-api-key` for a free AI Studio key. Per the tool schema, header names must end in key, token or auth.
  - Subscription-login flows (ChatGPT login for Codex CLI, Google login for Antigravity CLI) would need OAuth tokens to persist in an ephemeral VM; not verified. ToS questions belong to another researcher.
- **Comments and labels made by a Claude cloud session can probably trigger other vendors' GitHub integrations** (`@codex`, `@claude` in Agent HQ, Copilot assignment, Actions workflows). They go out under the user's own GitHub identity, not a bot, so they are not subject to the GITHUB_TOKEN non-triggering rule. This makes Claude a good "router" that posts `@codex …` or applies labels.

### Gaps
- The Projects page doesn't state concurrent-thread or concurrent-session caps. The routines troubleshooting mentions only an organization "limit on cloud sessions running at once"; the number for Max 20x was not found.
- The exact contents of the Default "Trusted" allowlist were not fetched, so it is unknown whether api.openai.com, chatgpt.com or generativelanguage.googleapis.com are included (see the [cloud-environments page](https://code.claude.com/docs/en/cloud-environments)).
- The `allowed_bots` loop behavior, and the exact `trigger_phrase`/`label_trigger` inputs, live in `docs/usage.md` and `action.yml`, which were not fetched.

---

## Q2. OpenAI Codex cloud: mobile, @codex on GitHub, automations, Slack/Linear, network, API/SDK

### Takeaway
Codex cloud runs in ChatGPT, now framed as "cloud chats" under **ChatGPT Work** on web, mobile and desktop. It has strong *inbound* hooks:
- `@codex` on PRs
- Linear assignment or `@Codex` mentions, plus Linear triage rules that auto-delegate
- Slack
- scheduled tasks that fire on **GitHub PR activity, Slack channel messages, or Gmail messages**

No public webhook or HTTP API for starting cloud tasks was found. The Codex GitHub Action runs `codex exec` on the runner with an **API key only**.

### Cited Findings

**Surfaces and environments**
- Start from "ChatGPT on the web, mobile, or in the desktop app". On mobile you open Codex; on web or desktop it's Work in > Cloud. The CLI can start tasks and list cloud chats ([Codex Cloud docs](https://learn.chatgpt.com/docs/cloud); developers.openai.com/codex/* now 308-redirects to learn.chatgpt.com/docs/*).
- Each task gets its own workspace and continues while the computer sleeps ([same](https://learn.chatgpt.com/docs/cloud)).
- Environment settings: repository, secrets, outbound identity, network, install script, a "Start skill", access ("Only me" or workspace), and OIDC connections ([same](https://learn.chatgpt.com/docs/cloud)).
- Network options: "Enable all public internet access", "Enable package manager domains", or additional allowed domains. "Saving an HTTP secret also allows access to its domains" ([same](https://learn.chatgpt.com/docs/cloud)).
- Codex Cloud is available to eligible Plus, Pro, Business, Enterprise and Edu accounts "subject to rollout and workspace settings" ([OpenAI Help, via search snippet](https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan)). The ChatGPT Work/Codex help article returned 403 and could not be read ([help.openai.com 20001275](https://help.openai.com/en/articles/20001275-chatgpt-work-and-codex)).

**GitHub**
- PR comment commands:
  - `@codex review` posts a review flagging only P0/P1 issues.
  - `@codex security review` is a research preview.
  - Any other `@codex …` comment "starts a legacy cloud chat using the PR as context", for example `@codex fix the CI failures`, and can push fixes.
- Automatic review on all PRs can be enabled.
- `AGENTS.md` sections titled `## Code Review Rules` steer reviews.
- Source: [Codex GitHub docs](https://learn.chatgpt.com/docs/third-party/github).
- That page covers PRs only. It doesn't document issue assignment or whether bot-authored comments trigger Codex ([same](https://learn.chatgpt.com/docs/third-party/github)).

**Linear**
- Assign an issue to Codex, or `@Codex` in a comment. Codex "creates a cloud chat and replies with progress and results", posting a summary and a chat link "so you can create a pull request". A repo can be pinned with `@Codex fix this in owner/repo`.
- **Triage rules can auto-delegate new issues to Codex** (Delegate > Codex), which then run under the issue creator's account.
- Available on paid ChatGPT plans.
- Source: [Codex Linear docs](https://learn.chatgpt.com/docs/third-party/linear).

**Scheduled tasks (automations)**
- *Where they run:* in the desktop app, against local projects, which needs the computer on; or on ChatGPT web, which uses uploaded context and connected tools but no local folder.
- *Triggers:* time schedules (RRULE, minute intervals for chat tasks), or **app events on ChatGPT web and mobile only**:
  - **Gmail** new messages, filterable by sender or subject
  - **Slack** new messages in selected channels, filterable by author (add `@ChatGPT` to the channel)
  - **GitHub** PR activity, filterable by PR, author, title or **label**
- A task can have several event triggers but "can't combine event triggers with a time-based schedule". Matching events arriving close together "may be combined into one run".
- Event triggers are "on eligible plans", which are not named. Admins have an "Allow event-triggered scheduled tasks" permission. No webhooks are mentioned.
- Source: [Codex scheduled tasks docs](https://learn.chatgpt.com/docs/automations).
- The same page notes that GPT-5.5 retired from Codex with ChatGPT sign-in on Oct 14, 2026 ([same](https://learn.chatgpt.com/docs/automations)).
- ChatGPT (non-Codex) scheduled tasks give Plus accounts 5 active task slots ([OpenAI Help, via search snippet](https://help.openai.com/en/articles/10291617-scheduled-tasks-in-chatgpt)).

**GitHub Action**
- `openai/codex-action` installs the Codex CLI and runs `codex exec` inside the Actions job.
- Auth is via the `openai-api-key` input; the docs don't mention ChatGPT login.
- Inputs include `allow-users` and `allow-bots` (default: write-access users only) and a `sandbox` setting. There is no statement that it can start Codex *cloud* tasks.
- Source: [Codex GitHub Action docs](https://learn.chatgpt.com/docs/github-action).
- The navigation also lists Codex SDK, app-server and mobile pages, which were not fetched ([Codex Cloud docs nav](https://learn.chatgpt.com/docs/cloud)).

**Agent HQ:** Codex is also available as a GitHub Agent HQ agent, billed to Copilot (see Q5) ([GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents)).

### Inferences
- The most server-free ways to *push* work into Codex from another vendor's agent:
  - **(a)** Have Claude open or label a PR, with a Codex scheduled task whose GitHub PR-activity trigger is filtered to a label such as `codex`.
  - **(b)** Have Claude comment `@codex <instruction>` on a PR via gh or GitHub MCP. It posts as the user, so Codex likely accepts it.
  - **(c)** Have Claude create a Linear issue through the claude.ai Linear connector, with a triage rule that auto-delegates to Codex.
  - **(d)** Have Claude post in a Slack channel watched by a Codex Slack-triggered task, or send a Gmail message matching a subject filter.
- Return path: Codex's PR, comment or Linear summary can then trigger a Claude scheduled task (GitHub PR trigger) or a `subscribe_pr_activity` session.
- Because PR-mention cloud chats are labeled "legacy", OpenAI appears to be moving Codex cloud under "ChatGPT Work". Expect naming and behavior churn.

### Gaps
- No documented public HTTP API or webhook to start a Codex cloud task was found. Whether the Codex SDK can target cloud tasks is unverified (page not fetched).
- Plus-tier Codex cloud task quotas, and whether Plus counts as an "eligible plan" for event-triggered tasks, are not confirmed.
- The Slack integration page (learn.chatgpt.com/docs/third-party/slack) was not read.
- Whether `@codex` comments authored by a GitHub App or bot are honored is undocumented.

---

## Q3. Google: Jules, Gemini CLI GitHub Action, Code Assist, Antigravity: what Plus/Pro unlock, phone usability

### Takeaway
Google's lineup changed under the user's feet:
- **Jules** is effectively dead for new work after Nov 11, 2026, and shuts down Dec 31.
- **Gemini CLI** moved consumer and AI Pro/Ultra users to the **Antigravity CLI** on June 18, 2026.
- **Antigravity 2.0** (May 2026) is desktop and CLI-centric.

The genuinely hosted, server-free Google option is the **Antigravity managed agent in the Gemini API** (preview). It runs in a Google-hosted Linux sandbox, with `background=True` and cron triggers. It is billed per token with a free-tier quota, on an API key rather than a Google AI subscription. For GitHub-native use, `run-gemini-cli` with a free AI Studio key remains the documented path.

### Cited Findings

**Jules shutdown**
- "Jules will wind down on December 31, 2026." Sessions can be finished until Nov 11, after which Jules is read-only (changelog entry Oct 29, 2026) ([Jules changelog](https://jules.google/docs/changelog/)).
- Google directs users to Antigravity, where Jules' background functions "will gradually be migrated", with no guarantee of feature parity. Pro/Ultra lose Jules after Dec 31; export is via the Jules CLI ([CryptoBriefing](https://cryptobriefing.com/google-winds-down-jules-ai-antigravity/); [CodeSprint](https://www.codesprintconsulting.com/blog/google-jules-shutdown); [ProPakistani, Nov 2 2026](https://propakistani.pk/2026/11/02/google-is-shutting-down-its-ai-coding-tool/)). These are secondary sources; the changelog is primary.

**Jules features for the record** (useful if Antigravity inherits them) ([Jules changelog](https://jules.google/docs/changelog/)):
- the `jules` issue label as a trigger (Jun 2025)
- PR comment responses, with "Reactive Mode" responding only to `@Jules` (Sep 2025)
- the Jules API (Oct 2025), now v1alpha with repoless sessions and patch output (Jan 2026)
- the Jules Tools CLI with `--parallel` (max 5)
- Scheduled Tasks (Dec 2025), with edit/pause in Jan 2026
- Suggested Tasks for AI Pro and Ultra
- MCP (Feb 2026: Linear, Stitch, Neon, Tinybird, Context7, Supabase)
- CI Fixer (Feb 2026)

**Jules tier limits** (third-party, consistent across sources): Free 15 tasks/day with 3 concurrent; Pro 100/day with 15; Ultra 300/day with 60 ([hackup.ai](https://hackup.ai/ai-plans/jules/); [morphllm](https://www.morphllm.com/comparisons/jules-google-coding-agent)). These are moot after Nov 11.

**Gemini CLI**
- "Free individual access and Google AI Pro / Ultra (Google One) moved to Antigravity CLI on June 18, 2026."
- Gemini CLI is now "designed for Gemini Code Assist Standard / Enterprise licenses" plus API-key and Vertex users.
- Source: [gemini-cli README](https://github.com/google-gemini/gemini-cli).

**`google-github-actions/run-gemini-cli`**
- The documented key is a Google AI Studio API key, with "generous free-of-charge quotas". `use_gemini_code_assist` is an alternative.
- Source: [run-gemini-cli repo](https://github.com/google-github-actions/run-gemini-cli); [Google AI Developers on X](https://x.com/googleaidevs/status/1967665922807902400); both via search summary.
- A Google AI Pro subscription likely does **not** authorize the action ([inference from the README and gemini-cli notice](https://github.com/google-gemini/gemini-cli)).

**Antigravity 2.0**
- Launched at I/O on May 19, 2026, with a desktop app, CLI and SDK.
- Supports custom subagent workflows and "schedule tasks that can automatically run in the background", plus headless mode in the CLI.
- Plans: Individual ($0), Google AI Pro, Ultra.
- Sources: [TechCrunch](https://techcrunch.com/2026/05/19/google-launches-antigravity-2-0-with-an-updated-desktop-app-and-cli-tool-at-io-2026/); [The Next Web](https://thenextweb.com/news/google-antigravity-2-desktop-cli-sdk-io-2026); [Google I/O dev highlights](https://blog.google/innovation-and-ai/technology/developers-tools/google-io-2026-developer-highlights/).
- One review says Antigravity works only with GitHub, not GitLab or Bitbucket ([emergent.sh review](https://emergent.sh/learn/google-antigravity-review)); single source.

**Antigravity managed agent (Gemini API)**
- "inside your own secure Linux sandbox, hosted by Google". It reasons, runs code, manages files and browses.
- Called through the Interactions API with `agent="antigravity-preview-09-2026"` and `environment="remote"`, at `https://generativelanguage.googleapis.com/v1beta/interactions` or the Python SDK.
- Supports `background=True`, polled by interaction ID.
- **Triggers on a cron schedule** with an IANA time zone that reuse the same environment, with pause/resume/run-now.
- Network rules can allowlist `api.github.com` with a stored credential. There is no native GitHub app and no webhooks.
- Pay-as-you-go; "Free tier projects include a free rate limit and usage quota". Status: preview.
- Source: [AI Studio docs: Antigravity agent](https://aistudio.google.com/docs/antigravity-agent).

### Inferences
- For the user's Google AI Plus/Pro subscriptions, the hosted, phone-usable, server-free value for *orchestration* is now thin.
  - The subscriptions mainly lift Antigravity (desktop/CLI) limits, which need a machine.
  - The Antigravity managed agent and `run-gemini-cli` are keyed to AI Studio API projects, which have their own free tier, not to the subscriptions.
- Practical Google worker paths without a server:
  - **(a)** A GitHub Actions workflow with `run-gemini-cli` and a free AI Studio key, triggered by a label or `@gemini-cli` comment, running on GitHub-hosted runners.
  - **(b)** A Claude cloud session or GitHub Action calls the Antigravity agent REST API with `background=True`, and the agent pushes to GitHub using a credential stored in its network rules.
- Jules' `jules` label trigger should not be designed into anything new.

### Gaps
- No source states what **Google AI Plus** (vs Pro) unlocks for Antigravity, Jules or Code Assist.
- **Gemini Code Assist GitHub app** (PR review) status in 2026 was not researched.
- **Firebase Studio** and **Gemini in Colab** agent features were not researched.
- Whether Antigravity has a mobile or web surface for background agents was not found; search results described only desktop, CLI and voice.
- Whether Jules' label trigger or API will be ported to Antigravity is unconfirmed.

---

## Q4. Meta AI agent (Muse): capabilities and external integrations

### Takeaway
Muse is a consumer personal agent:
- a per-user cloud VM with a watchable browser
- connectors including **GitHub**, Notion and Granola, with no published details
- a developer connector platform
- a dedicated agent **email address**, announced as "soon"

It's on iOS, Android and the web, US-only, with a free tier. It's not a coding agent with repo or PR workflows. Meta's coding product, Muse Code, is a local terminal harness. No API, webhook or scheduled-trigger surface for chaining Muse was found.

### Cited Findings
- **Release and pricing:** released Sept 8, 2026 in the US on iOS, Android and web. Free tier plus $20 and $100/month subscriptions ([Wikipedia: Muse (AI agent)](https://en.wikipedia.org/wiki/Muse_(AI_agent)); [Axios](https://www.axios.com/2026/09/08/meta-debuts-muse-personal-ai-agent)).
- **Execution:** a separate VM per user in Meta's cloud, with a built-in browser the user can watch. A permissions system called "Sentinel" separates read from write access per service ([Wikipedia](https://en.wikipedia.org/wiki/Muse_(AI_agent)); [CNN](https://www.cnn.com/2026/09/23/tech/meta-muse-ai-agent)).
- **Sept 23, 2026 announcements:**
  - background work that "check[s] back in when it's done"
  - Mac app control
  - its own email address "soon" (you can also add Muse to a thread or forward emails to it)
  - named connectors **GitHub, Granola, Notion**
  - a developer connector platform (1,500+ applications in a week)
  - free for "a huge number of tokens"; no published tier limits

  Source: [TechCrunch](https://techcrunch.com/2026/09/23/everything-new-coming-to-metas-ai-agent-muse/).
- **Muse Code** is a terminal coding agent on Muse Spark models, with subagents, MCP, hooks, skills and sandboxed execution, running on macOS and Linux. This is local, not hosted ([innfactory](https://innfactory.ai/en/ai-harness/muse/); [superqode docs](https://superagenticai.github.io/superqode/providers/muse-code/)); third-party sources.
- **Risks:**
  - Amazon blocked Muse on Sept 21, 2026 ([GeekWire](https://www.geekwire.com/2026/amazon-blocks-metas-muse-ai-assistant-in-new-standoff-over-agentic-shopping/)).
  - Shared chats were discoverable via passive DNS, and Meta suspended sharing in Oct 2026 ([Ars Technica](https://arstechnica.com/security/2026/10/meta-muse-exposes-users-personal-and-business-secrets-when-they-share-conversations/)).
  - Queries train models by default ([Wikipedia](https://en.wikipedia.org/wiki/Muse_(AI_agent))).

### Inferences
- Muse's realistic role in an orchestration plane is a **side worker for browsing and research** rather than a code agent.
- A plausible no-server handoff is **email**: forward or CC Muse's agent address. Its GitHub connector might also let it read or open issues, but that is unverified.
- Given the shared-chat exposure, avoid routing secrets or private repo content through Muse.

### Gaps
- What the Muse GitHub connector can actually do (read repos? create issues or PRs?) is undocumented in the sources found.
- Whether Muse's VM supports code execution for users (vs browsing only) is unconfirmed; one source claims the browser sub-agent can't run JavaScript ([i10x](https://i10x.ai/news/meta-muse-ai-agent), single source).
- No public API, webhook or scheduling surface was found.
- Free-tier limits are unpublished.
- ai.meta.com/muse returned no body text.

---

## Q5. GitHub Copilot coding agent / Agent HQ, and GitHub as a cross-vendor message bus

### Takeaway
GitHub Agent HQ lets you assign issues and PRs to **Copilot, Claude or Codex**, or several at once, from github.com, **GitHub Mobile** and VS Code, and `@mention` them in PR comments. Claude and Codex there are **billed to a Copilot plan** (Pro, Pro+, Max, Business, Enterprise; not Free) via AI credits and Actions minutes, **not** to the user's Claude Max or ChatGPT Plus. Jules is not listed. So Agent HQ is a convenient phone UI and shared bus, but it doesn't consume the subscriptions the user already pays for.

### Cited Findings
- **Launch:** Claude and Codex entered public preview on Feb 4, 2026 for Copilot Pro+ and Enterprise, using one premium request per session during preview ([GitHub Changelog](https://github.blog/changelog/2026-02-04-claude-and-codex-are-now-available-in-public-preview-on-github/)).
- **Assignment:** from the issue Assignees dropdown (Claude, Codex, Copilot, or all three), producing a draft PR. You can also assign an open PR. `@copilot`, `@claude` and `@codex` work in PR review comments ([same](https://github.blog/changelog/2026-02-04-claude-and-codex-are-now-available-in-public-preview-on-github/); [GitHub Blog](https://github.blog/news-insights/company-news/pick-your-agent-use-claude-and-codex-on-agent-hq/)).
- **Current docs:**
  - Plans: Copilot Pro, Pro+, Max, Business, Enterprise.
  - Billing: "Each agent session consumes AI credits… and GitHub Actions minutes". Agents "can be powered by your existing Copilot subscription"; the docs say nothing about using your own Anthropic or OpenAI subscription or key.
  - Surfaces: Agents tab at `github.com/OWNER/REPO/agents`, issue assignment, `@AGENT_NAME` in PR comments, **GitHub Mobile** Home > + to start a session, and VS Code.
  - Only Claude and Codex are listed as third-party agents. Claude models include Sonnet 4.6, Sonnet 5, Opus 4.8, Opus 5 and Fable 5.1; Codex models include GPT-5.3-Codex and GPT-5.4 nano.
  - Source: [GitHub Docs: About third-party agents](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents).
- **Billing change:** GitHub moved Copilot to usage-based billing with AI Credits on June 1, 2026. Legacy premium-request docs apply only to some annual Pro/Pro+ subscribers ([GitHub Docs: Copilot requests (legacy)](https://docs.github.com/en/copilot/reference/copilot-billing/request-based-billing-legacy/copilot-requests); [morphllm](https://www.morphllm.com/comparisons/claude-code-vs-copilot)).
- **Earlier announcements:** the Dec 2025 coverage said Agent HQ would add Jules, Cognition and xAI agents. There is no confirmation that Jules shipped, and Jules is now shutting down ([The New Stack](https://thenewstack.io/github-embraces-the-coding-agent-competition-with-agent-hq/); [i-programmer](https://www.i-programmer.info/news/105-artificial-intelligence/18651-github-adds-claude-and-codex-to-agent-hq.html)).
- **Copilot coding agent** works asynchronously in a GitHub Actions environment when you assign an issue to `@copilot`, and opens a draft PR ([GitHub Blog](https://github.blog/news-insights/company-news/pick-your-agent-use-claude-and-codex-on-agent-hq/)).
- **Vendor-native GitHub entry points that bill to the user's own subscriptions:**
  - Claude: GitHub App, scheduled-task GitHub triggers, PR auto-fix ([Claude docs](https://code.claude.com/docs/en/routines)).
  - Codex: `@codex` on PRs and PR-activity scheduled tasks ([Codex docs](https://learn.chatgpt.com/docs/third-party/github); [automations](https://learn.chatgpt.com/docs/automations)).

### Inferences
- **"GitHub as the bus" works best with each vendor's own GitHub integration, not Agent HQ**, if the goal is to burn the user's existing Max and Plus quotas:
  - **Task queue:** issues, with labels for routing (`agent:claude-a`, `agent:claude-b`, `agent:codex`, `agent:gemini`).
  - **Claude-A:** a scheduled task with a GitHub Issue-labeled trigger, or a dispatcher session.
  - **Claude-B:** its own scheduled task on the same repo, or an Actions workflow that curls B's `/fire`.
  - **Codex:** `@codex` comment on a PR, a PR-label event task, or Linear delegation.
  - **Gemini:** a `run-gemini-cli` workflow.
  - **Handoff artifacts:** PRs, plus `AGENTS.md`/`CLAUDE.md` for shared conventions.
- Agent HQ is still worth considering as a **single phone UI** (GitHub Mobile) if the user adds a Copilot plan, accepting that runs consume Copilot credits.

### Gaps
- The current Copilot Pro and Max price and AI-credit allowance were not verified.
- Whether Agent HQ's Claude/Codex sessions can ever use a linked Anthropic or OpenAI account is not stated anywhere found.
- Whether Copilot Free includes the coding agent was not checked; the third-party agents page omits Free.

---

## Q6. Practitioner cross-vendor handoff recipes and their limitations

### Takeaway
Reported practice settles on **role separation**: one agent implements, another reviews, a human merges. Loops are bounded, typically at most 3 rounds. Most published recipes are local plugins or skills rather than hosted GitHub pipelines. Hard data on outcomes is absent, and there is no systematic evidence that hosted multi-vendor GitHub loops work reliably.

### Cited Findings
- **Hamel Husain's `review-loop` plugin:** Claude makes a change, Codex independently reviews it, and Claude addresses the findings (updated Mar 15, 2026) ([sourcepulse listing](https://www.sourcepulse.org/projects/25108003); [skillselion](https://skillselion.com/skills/hamelsmu/claude-review-loop/plugins/review-loop)).
- **Role guidance:** "one agent should own implementation; the other should review", with the human making the final call (July 2026) ([knightli](https://knightli.com/en/2026/07/10/claude-code-codex-code-review-workflow/)).
- **Bounded loops:** a `claude-review` skill defaults to max 3 rounds, and a Codex review skill escalates after 3 cycles ([skills.sh](https://skills.sh/verneagent/tiny-skills/claude-review); [skillselion](https://skillselion.com/skills/nickcrew/claude-cortex/codex-code-review)).
- **Addy Osmani's "Code Agent Orchestra"** frames the shift from conducting one agent to orchestrating many, citing tools like Conductor (parallel Claude Code and Codex in worktrees). That is local, not hosted ([addyosmani.com](https://addyosmani.com/blog/code-agent-orchestra/)).
- **Community issue-driven wrappers:** VibeGithub (command Jules, Copilot and OpenCode from issues, auto-assign) and `jules-agent` on PyPI (routing: Claude plans, Codex codes, Copilot reviews) ([VibeGithub](https://github.com/friuns/VibeGithub); [PyPI jules-agent](https://pypi.org/project/jules-agent/)). Hobby projects; Jules parts are now obsolete.
- **Agent HQ "assign to all three and compare"** is GitHub's own recommended multi-agent pattern ([GitHub Blog](https://github.blog/news-insights/company-news/pick-your-agent-use-claude-and-codex-on-agent-hq/)).
- **Limitations documented by the vendors:**
  - Claude GitHub triggers drop events beyond hourly caps and start a new session per event, so there is no shared context across events ([Claude routines](https://code.claude.com/docs/en/routines)).
  - Claude auto-fix cannot see merge conflicts ([Claude cloud docs](https://code.claude.com/docs/en/claude-code-on-the-web)).
  - Codex batches close-together events into one run ([Codex automations](https://learn.chatgpt.com/docs/automations)).
  - claude-code-action blocks bots by default and documents no loop prevention ([security.md](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md)).
  - Claude's GitHub replies can trigger `issue_comment` automation ([Claude cloud docs](https://code.claude.com/docs/en/claude-code-on-the-web)).
- **GitHub Actions rule:** events created with the workflow `GITHUB_TOKEN` don't start new workflow runs, except `workflow_dispatch` and `repository_dispatch`. A bot-applied label therefore won't fire a `labeled` workflow unless a PAT or GitHub App token is used ([GitHub Docs: Triggering a workflow from a workflow](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow); caveat also noted in community results).

### Inferences
**Concrete hosted-only recipe (phone-operable):**

1. From the Claude mobile app, a **Claude-A Project** (or a dispatcher session) plans the work. It files GitHub issues with routing labels and acceptance criteria, and commits `AGENTS.md`/`CLAUDE.md` conventions.
2. Workers are triggered as follows:

   | Label | Worker | How it fires |
   | :- | :- | :- |
   | `agent:claude-b` | Claude account B | Actions workflow curls B's scheduled-task `/fire` (token in repo secrets), or B's own Issue-labeled trigger |
   | `agent:codex` | Codex | Claude comments `@codex implement #N` on a stub PR, or creates a Linear issue auto-delegated to Codex |
   | `agent:gemini` | Gemini | `run-gemini-cli` workflow with an AI Studio key |

3. Workers open PRs.
4. Claude-A reviews, via `subscribe_pr_activity` or a scheduled task with a PR-opened trigger filtered by head branch or label. It asks for changes with `@codex …` or a label, capped at N rounds.
5. The human merges from GitHub Mobile.

**Loop and latency guards:**
- Require a "round:N" label and stop at 3.
- Use distinct labels per direction.
- Keep `allowed_bots` explicit.
- Use the GITHUB_TOKEN non-triggering behavior deliberately as a circuit breaker.
- Expect minutes of latency per hop: VM provisioning, webhook delivery, and event batching.

### Gaps
- No practitioner write-up was found that demonstrates a fully hosted **Claude ↔ Codex ↔ Gemini GitHub loop** with measured latency or failure rates.
- No Hacker News or Reddit threads on this specific pattern were retrieved within the search budget.
- No data on duplicate-run or bot-loop incidents.

---

## Q7. Server-free glue: GitHub Actions, Slack, Linear, Gmail, Zapier/Make/n8n cloud

### Takeaway
Enough hosted glue exists that no home server is needed:

| Glue | What it is | What it bridges |
| :- | :- | :- |
| GitHub Actions | GitHub-hosted runners | label/comment events, curl to Claude `/fire`, `run-gemini-cli`, `codex-action` with an API key, `claude-code-action` with an OAuth token |
| Codex event-triggered tasks | runs inside Codex | Slack, Gmail and GitHub PR events |
| Linear | delegation and triage rules | Codex (and Jules MCP historically) |
| Claude connectors | MCP via Anthropic servers | Slack, Linear, Gmail and Drive, usable inside scheduled tasks |

The cleanest cross-vendor bridges are *message-in-a-shared-inbox* patterns: GitHub, Slack, Linear or Gmail.

### Cited Findings
- **Claude connectors** reach services through Anthropic's servers, act as the user, and are included in scheduled tasks by default. The docs' example: a triage task reads Slack and creates Linear issues, then posts a summary to Slack ([Claude routines](https://code.claude.com/docs/en/routines)).
- **Claude's `/fire` endpoint** can be called from "alerting systems, deploy pipelines, internal tools, or anywhere you can make an authenticated HTTP request" ([same](https://code.claude.com/docs/en/routines)).
- **Codex scheduled tasks** can trigger on Slack channel messages (with `@ChatGPT` added to the channel), Gmail new messages (filter by sender or subject), and GitHub PR activity (filter by label) ([Codex automations](https://learn.chatgpt.com/docs/automations)).
- **Linear triage rules** can auto-delegate to Codex ([Codex Linear docs](https://learn.chatgpt.com/docs/third-party/linear)).
- **Slack integrations:**
  - Claude: Claude in Slack sessions (auto-shared on Team) and "Claude Tag", an organization-managed @Claude in Slack ([Claude cloud docs](https://code.claude.com/docs/en/claude-code-on-the-web); [Claude Tag](https://claude.com/docs/claude-tag/overview)).
  - Codex: a Slack integration page exists ([Codex docs nav](https://learn.chatgpt.com/docs/cloud)).
  - Jules: the API launched with Slack "ChatOps" examples ([Jules changelog](https://jules.google/docs/changelog/)).
- **Antigravity managed agent** can be run on demand or on a cron trigger through REST with an API key, with no server needed ([AI Studio docs](https://aistudio.google.com/docs/antigravity-agent)).
- **`openai/codex-action`** requires `openai-api-key` ([Codex GitHub Action docs](https://learn.chatgpt.com/docs/github-action)). **`claude-code-action`** accepts `claude_code_oauth_token` ([security.md](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md)).

### Inferences
- **Slack or Gmail as a cross-vendor bus.**
  - *Claude to Codex:* a Claude scheduled task or session uses its Slack or Gmail connector to post a message, for example a Slack message in `#codex-queue`, or an email with subject `[codex] …`. That fires a Codex event-triggered task.
  - *Codex to Claude:* Codex's results land in GitHub as PRs, which trigger a Claude GitHub-event task.
  - *Caveats:* this relies on messages posted as the user being accepted by Codex's author filters, and on Plus being an "eligible plan". Neither is confirmed.
- **Zapier, Make and n8n cloud** could stand in as glue (for example, a Gmail or GitHub trigger calling Claude's `/fire`). They aren't required, since GitHub Actions covers the same HTTP-call role for free on public repos and within free minutes on private repos.
- **The Claude GitHub Action on a GitHub-hosted runner** is a second Claude execution venue, separate from cloud sessions. With a subscription OAuth token it may draw on a Max plan; ToS and quota implications belong to other researchers.

### Gaps
- Zapier, Make and n8n cloud specifics (free-tier task limits, native Claude/Codex/Gemini steps) were not researched.
- GitHub Actions free-minute allowances for private repos in 2026 were not verified.
- Whether `claude_code_oauth_token` is officially tied to Pro/Max (`claude setup-token`) is not confirmed on the page fetched.
- Codex Slack-trigger behavior with messages posted via another app or connector is unverified.
