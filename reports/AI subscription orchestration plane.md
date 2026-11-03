# Wire six AI subscriptions through GitHub

You can build a personal orchestration plane today with no server and without breaking any vendor's terms. Make Claude Ultra account A the dispatcher, make a private GitHub repository the message bus, and have every other agent pick up work from GitHub labels and comments. The evidence forces this shape. GitHub is the one channel all of the user's coding agents already watch, and Claude is the only subscription that accepts work over an inbound HTTP trigger. In the zero-server version:

- A Claude Project on account A, run from the Claude phone app, files labeled issues.
- A GitHub Actions workflow forwards `agent:claude-b` issues to account B's scheduled-task `/fire` endpoint.
- Codex is called in with `@codex` comments, and its PR reviews come from a separate usage bucket.
- Gemini work runs through Google's GitHub Action on a free AI Studio key.
- The user merges from GitHub Mobile.

None of this costs anything beyond the subscriptions. A $15–24/month always-on box adds things the vendor clouds can't do. It runs `codex exec` locally on cheaper allowances, OpenAI's official Claude-to-Codex plugin, Claude Remote Control for both accounts, and one phone dashboard for the non-Claude agents. It is also the only place the Google AI Pro subscription can be scripted at all. Google's coding value fell apart in 2026. Gemini CLI stopped serving consumer accounts on 18 June. Jules goes read-only after 11 November and shuts down on 31 December, according to its own changelog, which outranks the undated limits page one note relied on. Both Google plans therefore become research and media seats. Meta's free Muse has no sanctioned programmatic path and stays a manual errand-runner. The safety line is the same at every vendor: the vendor's own unmodified client, signed in by the subscriber, doing the subscriber's own work. The 2026 ban waves hit proxies, OAuth reuse in third-party tools, pooled accounts and scripted UIs.

## Only Claude accepts work from another program

What an orchestrator can do with each subscription depends on which surfaces accept work from outside. The vendors differ far more than their prices suggest.

**Claude has by far the most options.** Each Ultra account can run:

- headless `claude -p` with JSON streaming, and the Agent SDK, on the subscription login ([Claude Code headless docs](https://code.claude.com/docs/en/headless));
- cloud sessions on roughly 4 vCPU / 16 GB VMs, with "no separate charge for the cloud VM" ([cloud environments](https://code.claude.com/docs/en/cloud-environments); [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web));
- Projects, which coordinate parallel cloud-session threads and can be used from the iOS and Android app ([Projects](https://code.claude.com/docs/en/claude-projects));
- scheduled tasks that fire on cron, on GitHub PR, issue and release events, or on an authenticated `POST` to a per-task `/fire` URL that returns the new session's URL ([scheduled tasks](https://code.claude.com/docs/en/routines)).

The cloud session that compiled this report can create, message, interrupt and schedule sibling sessions. Its scheduler rejects target sessions the account does not own (seen directly in the session's tool list on 3 Nov 2026). Native agent-to-agent messaging therefore stops at the account boundary. The per-task `/fire` token is the only documented bridge to the second account.

Since 30 Oct 2026 the old Max 20x has been called **Ultra ($200/mo)**:

- no five-hour limit once the account is verified;
- a weekly cap listed as "10x Pro";
- Fable, the top model, capped at 50% of the weekly limit;
- $200 a month of API credits included ([claude.com/pricing](https://claude.com/pricing)).

TNW notes that 10x is lower than the "20x" the old name implied. Newer Ultra accounts also keep a temporary five-hour limit until Anthropic verifies them ([TNW](https://thenextweb.com/news/claude-ultra-max-plan-rename-five-hour-limit)), which may still apply to the user's second account.

**Codex only accepts work that arrives as an event from another service.** On ChatGPT Plus it offers:

- `codex exec` with JSONL output ([non-interactive docs](https://learn.chatgpt.com/docs/non-interactive-mode.md));
- cloud tasks from the ChatGPT phone app;
- `@codex` commands on GitHub PRs ([Codex GitHub](https://learn.chatgpt.com/docs/third-party/github));
- Linear delegation, including triage rules that auto-delegate ([Codex Linear](https://learn.chatgpt.com/docs/third-party/linear));
- scheduled tasks that fire on GitHub PR activity filtered by label, on Slack channel messages or on Gmail messages ([Codex scheduled tasks](https://learn.chatgpt.com/docs/automations)).

The notes found no public HTTP endpoint for starting a Codex cloud task. Plus has had a rolling five-hour window again since 25 Aug 2026, plus a weekly cap whose size is not published ([9to5Mac](https://9to5mac.com/2026/08/24/openai-restores-5-hour-codex-and-work-limits-for-chatgpt-plus-users/)).

OpenAI's model table captures the user's "expensive model for cheap work" problem. It estimates **5–45 local messages per five hours on GPT-6 Astra versus 350–3,000 on GPT-6 Luna**, a spread of more than 60x. Cloud tasks "may use more of your allowance than local messages", and PR reviews run through GitHub **count against a separate code-review category** ([Codex pricing](https://learn.chatgpt.com/docs/pricing)).

**Google's subscriptions lost their programmable surfaces this year.**

- Gemini CLI stopped serving free, AI Pro and AI Ultra individual accounts on 18 June 2026. Consumers were moved to the closed-source Antigravity CLI, `agy` ([Gemini CLI discussion #27274](https://github.com/google-gemini/gemini-cli/discussions/27274)).
- Antigravity's AI Pro quota is "refreshed every five hours until the weekly limit is reached", with no numbers published ([Antigravity plans](https://antigravity.google/docs/plans)). Commenters on the transition thread report weekly quota running out in minutes.
- AI Plus does not appear on Antigravity's pricing page ([Antigravity pricing](https://antigravity.google/pricing)). It was already an unsupported tier for Gemini CLI ([geminicli.com](https://geminicli.com/docs/resources/quota-and-pricing/)), and its plan page lists no developer tools ([Google AI plans](https://one.google.com/about/google-ai-plans/)).
- Google's one truly hosted agent is the Antigravity managed agent in the Gemini API. It runs in a Google-hosted sandbox with background and cron triggers, but it bills an API key with a free-tier quota, not a subscription ([AI Studio docs](https://aistudio.google.com/docs/antigravity-agent)).
- Firebase Studio stopped accepting new workspaces on 22 June 2026 ([release notes](https://firebase.google.com/support/release-notes/firebase-studio)).

**Meta's free Muse has no programmatic way in at all.** It runs on its own per-user cloud computer with a browser you can watch, and has connectors including GitHub, Notion and Granola ([TechCrunch](https://techcrunch.com/2026/09/23/everything-new-coming-to-metas-ai-agent-muse/)). Its free allowance is reported at about 100 million tokens a week ([The Neuron](https://theneuron.ai/explainer-articles/how-to-get-started-with-meta-muse/)). The notes found no API, webhook, CLI or scheduler for it. Meta's terms forbid accessing its products "using automated means (without our prior permission)" ([Meta Terms](https://www.facebook.com/terms.php)). Meta's developer route, the Muse Spark API, is billed per token, from $1.25 input and $4.25 output per million tokens ([Meta dev blog](https://dev.meta.ai/resources/blog/build-with-muse-spark/)).

| Subscription | Sanctioned programmatic surfaces | Phone surface | Quota shape | Role in the plane |
|---|---|---|---|---|
| Claude Ultra A | `claude -p`, Agent SDK, cloud sessions, Projects, scheduled tasks (cron, GitHub, `/fire`), GitHub Action via `setup-token` | Claude app (Code tab, Projects) | Weekly only once verified; Fable ≤50% of weekly; +$200/mo API credits | Hub: dispatcher, planner, final reviewer |
| Claude Ultra B | Same, independent pool | Claude app, separate login | Same; may keep a 5-hour cap until verified | Implementation worker fired over `/fire` |
| ChatGPT Plus | `codex exec`, Codex SDK, cloud tasks, `@codex` on PRs, Linear, event-triggered tasks | ChatGPT app | 5-hour window plus weekly cap; GitHub reviews in a separate bucket | Reviewer and cheap mechanical worker |
| Google AI Pro | Antigravity app and CLI (`agy -p` documented for scripts and CI); Jules until 11 Nov | Gemini app; no mobile Antigravity surface found | Antigravity: 5-hour refresh plus weekly cap, unpublished; Colab 200 CU/mo; $10/mo Google Cloud credit | Research seat; optional Antigravity worker on a box |
| Google AI Plus | Chat apps only | Gemini app | Free-tier Antigravity at best | Deep Research and media seat |
| Meta Muse (free) | None | Muse app, web, WhatsApp | About 100M tokens/week (reported) | Manual browsing and errand worker |

This report reads the user's setup as one Google account on AI Plus and a second on AI Pro. Two of the notes reasoned about two AI Pro accounts. The doubled Google capacity they describe does not apply.

### Jules: both notes are right, but only until 11 November

**The disagreement.** The quota note treats Jules as the AI Pro plan's strongest asset: 100 tasks per rolling 24 hours with 15 running at once on AI Pro, against 15 tasks and 3 at once on the base tier ([Jules usage limits](https://jules.google/docs/usage-limits)). It recommends `jules remote new` as Google's scriptable path. The cloud-agents note cites a changelog entry dated 29 Oct 2026 saying **"Jules will wind down on December 31, 2026"**. Jules becomes read-only after 11 November, and Google points users to Antigravity, where Jules' background functions "will gradually be migrated" with no promise of matching features ([Jules changelog](https://jules.google/docs/changelog/); [ProPakistani, 2 Nov 2026](https://propakistani.pk/2026/11/02/google-is-shutting-down-its-ai-coding-tool/); [CryptoBriefing](https://cryptobriefing.com/google-winds-down-jules-ai-antigravity/)).

**Which source wins.** The evidence clearly favours the shutdown. The changelog is dated, comes from Google itself, and three outlets report the same thing. The limits page has no update date and still names Gemini 2.5 Pro and Gemini 3 Pro as its models. The CLI details come from October 2025 launch posts ([Google Developers Blog](https://developers.googleblog.com/meet-jules-tools-a-command-line-companion-for-googles-async-coding-agent/)). The two notes are written at different times rather than truly contradicting each other: the limits page describes a product that is technically still running.

**What it means.** As of 3 November, Jules has **eight days of usable life, then seven weeks read-only, then nothing**. The changelog says sessions can be finished until 11 November. It is not clear whether that allows starting new sessions or only completing open ones. Design nothing around Jules, use the remaining window only to finish or export work through the Jules CLI, and leave it out of the routing rubric.

The user doesn't use Google's cloud environments anyway, so the bigger consequence is strategic. Without Jules, the AI Pro plan has no hosted coding agent reachable from a phone. Google drops out of the coding side of the plane until Antigravity offers a background-agent surface.

## Zero-server plan: a private repo carries work between accounts

Every vendor's coding agent listens to GitHub, and only Claude listens to HTTP. The bus is therefore a private repository's issues, labels and pull requests, with Claude account A deciding where each task goes.

```
Phone → Claude app: Project on account A (dispatcher + routing rubric)
           │ files issues with full context, acceptance criteria and a label
           ▼
   private GitHub repo (issues · labels · PRs · AGENTS.md)
     ├─ agent:claude-a → A's own Project threads (cloud sessions, A's quota)
     ├─ agent:claude-b → Actions workflow ─curl→ B's scheduled-task /fire (B's quota)
     ├─ agent:codex    → A comments "@codex …" on a draft PR (ChatGPT Plus quota)
     └─ agent:gemini   → run-gemini-cli workflow (free AI Studio API key)
           │ every worker returns a PR
           ▼
   @codex review (separate review bucket) + A's review → user merges in GitHub Mobile
```

### Account A dispatches from a Project

From the phone, the user describes work in one long-running Project conversation. Claude starts a parallel cloud-session thread for each task, routes follow-ups and tracks PRs. The Overview pane has buttons for "Fix CI", "Address comments" and "Merge it" ([Projects](https://code.claude.com/docs/en/claude-projects)).

A few setup constraints apply:

- Projects are still a gradually rolling-out public beta.
- Project threads don't load a repo's `.mcp.json` or `.claude/skills/`, so the routing rubric has to live in an account-level skill.
- Every repo needs the Claude GitHub App installed ([same](https://code.claude.com/docs/en/claude-projects)).

The dispatcher's job is to apply the rubric. It does `agent:claude-a` work in its own threads and files everything else as GitHub issues with full context and acceptance criteria. The full context matters because each GitHub-triggered scheduled task starts a new session that does not reuse earlier ones ([scheduled tasks](https://code.claude.com/docs/en/routines)).

If Projects are not yet enabled on the account, a plain dispatcher cloud session works. Its VM pauses when idle and can be reclaimed, which loses running shell commands and session-scoped scheduled tasks ([cloud environments](https://code.claude.com/docs/en/cloud-environments)). Durable triggers should therefore be account-level scheduled tasks.

### Account B is reached through its own scheduled task

1. **On account B**, create a scheduled task with an API trigger. Its saved prompt must explicitly opt in to treating the fire payload as a task from the user's own dispatcher. The payload arrives wrapped as untrusted `<routine-fire-payload>` data, so the task won't act on it without that opt-in.
2. **In the shared repo**, store B's per-task bearer token, which is shown only once, as a GitHub Actions secret.
3. **Add a workflow** on `issues: labeled`. When the label is `agent:claude-b` and the sender is the user's own login, it POSTs the issue to `https://api.anthropic.com/v1/claude_code/routines/<trig_id>/fire` with the `experimental-cc-routine-2026-04-01` beta header ([scheduled tasks](https://code.claude.com/docs/en/routines)).
4. **B does the work**, pushes a `claude/`-prefixed branch and opens a PR.

There is a fully native alternative: a B scheduled task on a GitHub issue trigger. It works, but filters are documented only for PR events, so it may spend a run on every label event.

Design around these caps, none of which allows overage ([same](https://code.claude.com/docs/en/routines)):

- 30 combined fires per hour per task;
- 100 API fires per hour per account;
- GitHub events beyond the hourly caps are dropped.

### Codex is called in by comment

Once the repo is connected to Codex, any `@codex` comment other than `review` "starts a legacy cloud chat using the PR as context" and can push fixes ([Codex GitHub](https://learn.chatgpt.com/docs/third-party/github)). So the dispatcher opens a draft PR that references the issue and comments `@codex implement #N per the acceptance criteria`.

This should work because Claude cloud sessions post to GitHub using the user's own GitHub account. Anthropic warns those posts can trigger `issue_comment` workflows ([Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web)). To GitHub, and presumably to Codex, they look like the user.

Two alternatives exist:

- A Codex scheduled task triggered by PR activity filtered on a `codex` label. This works only on "eligible plans", which OpenAI doesn't name.
- A Linear issue created through Claude's Linear connector and handed to Codex by a triage rule ([Codex scheduled tasks](https://learn.chatgpt.com/docs/automations); [Codex Linear](https://learn.chatgpt.com/docs/third-party/linear)).

Turn on Codex's automatic review for every PR. It flags only P0 and P1 issues and is billed to the separate review bucket, which makes it the cheapest second opinion in the plane.

### Gemini runs on an API key, not the subscription

Google's `run-gemini-cli` GitHub Action is documented with a free Google AI Studio key ([run-gemini-cli](https://github.com/google-github-actions/run-gemini-cli)). An `agent:gemini` label fires it on GitHub-hosted runners. That spends Google's free API quota, which through Gemini CLI is 250 requests a day on Flash models only ([geminicli.com](https://geminicli.com/docs/resources/quota-and-pricing/)), not either Google subscription. That is deliberate: Google itself names an API key as the supported way to use Gemini from tools other than its own ([Gemini CLI FAQ](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/docs/resources/faq.md)).

### Every worker returns a PR

The dispatcher watches incoming PRs, either through a thread subscribed to PR activity or a scheduled task with a PR-opened trigger. It asks for changes with another `@codex` comment or label, and stops after a fixed number of rounds. Practitioners have settled on one agent implementing, another reviewing and a human merging, with at most three rounds ([knightli](https://knightli.com/en/2026/07/10/claude-code-codex-code-review-workflow/); [skills.sh](https://skills.sh/verneagent/tiny-skills/claude-review)). The user merges from GitHub Mobile.

### Loop and abuse guards

These guards are part of the design, not extras:

- **Keep the repo private.** Subscription tokens and run logs must never be public, and Codex's docs say not to run it in public environments ([Codex auth](https://learn.chatgpt.com/docs/auth.md)).
- **Check who applied the label.** The forwarding workflow should act only when the user's own login applied the label, so nobody else can spend B's quota.
- **Cap the rounds.** Use a `round:N` label and stop at three.
- **Use separate labels for each direction** of hand-off.
- **Set an explicit `allowed_bots` list** in any `claude-code-action` workflow. The action documents no loop prevention ([security.md](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md)).
- **Use GitHub's built-in circuit breaker.** Events created with a workflow's `GITHUB_TOKEN` don't start new workflow runs ([GitHub Docs](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow)), so a label applied by a workflow can't set off a cascade.

Expect minutes of delay per hop from VM start-up, webhook delivery and Codex batching events that arrive close together.

### It costs nothing, but it can't route by live quota

A forwarding job takes seconds against the $12 of Actions compute GitHub Free includes each month ([Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)). What it can't do is route by remaining quota:

- The router is account A's judgment, guided by the rubric.
- Codex's cheaper local allowance goes unused, because every Codex run is a cloud task.
- Both Google subscriptions sit outside the bus.
- Muse stays manual. Its output enters the bus when the user pastes it into an issue. Whether its GitHub connector can open issues itself is undocumented.

GitHub's Agent HQ offers a neater single phone screen for assigning issues to Claude, Codex or Copilot from GitHub Mobile. But those sessions are billed to Copilot AI credits and Actions minutes, not the user's Claude or ChatGPT plans ([GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents)).

## A $15 always-on box adds local quotas, Google and one dashboard

The box only has to hold CLI logins and long-running tmux sessions. It adds venues and keeps the zero-server bus. Cloud sessions and Codex cloud keep doing what they do.

### Choosing a host

| Host | Size | Price | Notes |
|---|---|---|---|
| exe.dev Personal | 2 vCPU / 4 GiB pool, 100 GiB disk, persistent VMs over SSH and HTTPS | $15 flat ([exe.dev pricing](https://exe.dev/pricing)) | Whether its VMs suspend when idle is unverified |
| Hetzner CX33 / CAX21 (EU) | 4 vCPU / 8 GB | €8.99 / €10.99 | Cheaper per gigabyte, but every Cost-Optimized plan showed "currently unavailable" on 3 Nov ([Hetzner](https://www.hetzner.com/cloud/cost-optimized/)) |
| DigitalOcean or Lightsail (US) | 4 GB | $24 ([DigitalOcean](https://www.digitalocean.com/pricing/droplets); [Lightsail](https://aws.amazon.com/lightsail/pricing/)) | — |
| Oracle Always Free (ARM) | 2 OCPU / 12 GB | $0 | Can reclaim instances whose CPU, network and memory all stay under 20% for seven days ([Oracle](https://docs.oracle.com/en-us/iaas/Content/FreeTier/resourceref.htm)). That is exactly how a mostly-waiting supervisor behaves. |

Claude Code alone needs 4 GB+ of RAM ([setup](https://code.claude.com/docs/en/setup)), so 8 GB is a comfortable target for three CLIs. Codespaces and Cloud Shell can't host a supervisor. A codespace has a 12-hour maximum lifetime, and Cloud Shell caps sessions at 12 hours and 50 hours a week ([Codespaces lifecycle](https://docs.github.com/en/codespaces/about-codespaces/understanding-the-codespace-lifecycle); [Cloud Shell limits](https://docs.cloud.google.com/shell/docs/quotas-limits)).

### Network and phone access

Join the box to Tailscale's free Personal plan ([Tailscale](https://tailscale.com/pricing)) and expose no public ports. Run OpenSSH with mosh and tmux; mosh does not work through Tailscale's own SSH server ([OpenReplay](https://blog.openreplay.com/remote-box-agentic-coding-setup/)). Connect from Termius on Android, or Blink or Termius on iOS.

### Keep each account separate

Give each subscription its own Unix user:

- `CLAUDE_CONFIG_DIR` separates the two Claude logins; each directory "has its own settings, session history, and claude.ai login" ([authentication](https://code.claude.com/docs/en/authentication)).
- Codex gets its own `CODEX_HOME`.
- The Antigravity login gets a separate user.

Two billing traps sit in Claude's credential order. Any `ANTHROPIC_API_KEY` in the environment silently moves `-p` runs to API billing. And `--bare` never reads the subscription login, yet it "will become the default for `-p` in a future release" ([headless docs](https://code.claude.com/docs/en/headless)).

Codex refreshes its plaintext `auth.json` while running. Run one Codex job at a time per login to avoid refresh races ([Codex auth](https://learn.chatgpt.com/docs/auth.md); [danielvaughan](https://codex.danielvaughan.com/2026/04/01/codex-cli-authentication-flows-credential-management/)). Codex supports `--device-auth` for signing in on a machine with no browser. Do the first Antigravity sign-in on a device that has a browser. Antigravity reportedly keeps credentials in the operating system's keyring ([Antigravity migration](https://antigravity.google/docs/cli/gcli-migration)), which a headless server may lack.

### Steer Claude with Remote Control

Run `claude remote-control --spawn worktree` for each account, inside tmux and under systemd. The Claude app then controls sessions on the box and sends push notifications for permission prompts, which stay open until answered. The box only makes outbound HTTPS connections. Server mode exits after about ten minutes without network, which is why systemd should restart it. Turn on Trusted Devices too ([Remote Control](https://code.claude.com/docs/en/remote-control)). The box can also queue messages into running cloud sessions with `claude -p "…" --cloud <session-id>` ([Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web)).

### Delegate from Claude to the other CLIs

Install OpenAI's own `codex-plugin-cc` in Claude Code. Its `/codex:review`, `/codex:adversarial-review` and `/codex:rescue` commands hand work to the locally logged-in Codex CLI, and the usage counts against the ChatGPT plan ([codex-plugin-cc](https://raw.githubusercontent.com/openai/codex-plugin-cc/main/README.md)). This is the most official cross-vendor delegation route available.

For Antigravity, use an interactive tmux worker rather than print mode:

- `agy -p` reportedly hangs when run without a terminal ([issue #318](https://github.com/google-antigravity/antigravity-cli/issues/318));
- gemini-mcp-tool reports that print mode is Gemini 3.5 Flash-only ([gemini-mcp-tool](https://raw.githubusercontent.com/jamubc/gemini-mcp-tool/main/README.md)).

oh-my-claudecode's `omc team N:antigravity` starts exactly these tmux workers from inside a Claude session ([oh-my-claudecode](https://raw.githubusercontent.com/Yeachan-Heo/oh-my-claudecode/main/README.md)).

### One phone dashboard for everything else

- **HAPI** wraps the official CLIs, Antigravity included, rather than replacing them. It adds a free installable web app with push notifications, one-tap approvals and a Telegram Mini App ([HAPI](https://raw.githubusercontent.com/tiann/hapi/main/README.md)).
- **Agent of Empires** is a tmux session manager with a web dashboard that has a "Phone: structured view" ([Agent of Empires](https://raw.githubusercontent.com/agent-of-empires/agent-of-empires/main/README.md)).
- **Maestro** has the richest cross-agent features: a moderator "group chat" and automatic resending when a plan's quota runs out. It doesn't support Gemini ([Maestro](https://raw.githubusercontent.com/RunMaestro/Maestro/main/README.md)).

### The router is a small script you write

None of the supervisors surveyed routes work across vendors by cost or remaining quota; a human or an LLM moderator makes the choice. A small script of your own fills the gap:

1. **Take in tasks** from GitHub issues or an authenticated ntfy topic.
2. **Classify each task** with a free API tier, or with account A's included API credits through a Console API key. API-key automation is the sanctioned route at every vendor and doesn't touch subscription limits ([Agent SDK help](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan)).
3. **Dispatch** through one of:
   - `claude -p --output-format stream-json` under the right `CLAUDE_CONFIG_DIR`, whose `system/api_retry` events flag rate limits;
   - `codex exec --json` under `CODEX_HOME`;
   - an `agy` tmux pane.
4. **Post results** as PRs.

Anyone who knows a public ntfy topic name can read and post to it. Use authenticated or self-hosted topics for anything that carries approvals ([claude-remote-approver](https://github.com/yuuichieguchi/claude-remote-approver)).

Goose is the one off-the-shelf harness that reaches all three paid vendors' subscriptions, by driving each vendor's own agent over ACP ([Goose](https://raw.githubusercontent.com/aaif-goose/goose/main/README.md)). It is an alternative for a user who wants one outer loop. Its Claude path, though, runs Anthropic's Agent SDK inside a third-party client, which is the grey zone discussed below.

## Routing rubric: start cheap, escalate once, spend Fable last

The rule: give each task the cheapest tier that can plausibly do it, escalate once if it fails, and reach the scarce resources only by explicit escalation. The scarce resources are Fable, GPT-6 Astra and account A's judgment. Two quota facts drive this. Codex's Luna-to-Astra spread is more than 60x ([Codex pricing](https://learn.chatgpt.com/docs/pricing)). Fable can use at most half of Ultra's weekly limit ([claude.com/pricing](https://claude.com/pricing)). Third-party guides report that claude.ai chat and Claude Code share one usage pool ([ai-toolbox](https://www.ai-toolbox.co/claude-management-and-productivity/claude-usage-limits-2026)), so casual chat belongs on the other apps.

| Task class | First choice | Escalate to | Why |
|---|---|---|---|
| Triage, classification, summarizing issues or logs, routing decisions | Free API tiers (Gemini API key, Groq, OpenRouter `:free`) or account A's included API credits on Haiku | Claude Sonnet in A's thread | API keys are the sanctioned automation path and don't touch subscription pools |
| PR review, bug-hunting | Codex automatic `@codex review` | Claude A on Opus, for risky or architectural PRs | GitHub reviews use Codex's separate review bucket |
| Mechanical edits: CI fixes, lint, dependency bumps, test scaffolding | Codex Luna via local `codex exec` (box), or `@codex fix the CI failures` (zero-server) | Codex Terra or Sol; Claude Sonnet | Luna allows 350–3,000 local messages per 5 hours |
| Well-specified implementation from an issue | Claude B (Sonnet, Opus when needed) via `/fire` | Codex cloud on Sol; Claude A | Keeps A's weekly budget for judgment; B is a separate pool |
| Planning, architecture, breaking work into tasks, cross-repo refactors, hard debugging | Claude A on Opus | Fable on A, by explicit escalation only | No 5-hour cap once verified; Fable capped at 50% of weekly |
| "Is this approach sound?" second opinion | `/codex:adversarial-review` from Claude (box) | Gemini in Antigravity (box) | A different model family catches different mistakes |
| Tasks that want Google models in CI | `run-gemini-cli` with a free AI Studio key | Interactive Antigravity on AI Pro (box) | Google subscriptions don't authorize Google's CI Action |
| Deep web research and long reports | Gemini Deep Research on AI Pro ("expanded") or AI Plus | ChatGPT | App features that spend no coding quota |
| Browser errands, bookings, form-filling, reading sites | Muse (free) | ChatGPT | Muse's own cloud computer and browser; hand-off is manual |
| Notebooks and GPU experiments | Colab on AI Pro (200 compute units/month) | — | Included with AI Pro |
| Images, video, media | AI Pro (1,000 Flow credits/month) or AI Plus | ChatGPT | Keeps Claude and Codex quotas for code |
| Casual questions and drafting | Gemini app, ChatGPT, Muse | Claude | Claude chat reportedly shares the coding pool |

The Colab and Flow allowances come from Google's plan page ([Google AI plans](https://one.google.com/about/google-ai-plans/)).

Four rules sit behind the table.

**Split the two Claude accounts by role, not by remaining quota.** A plans and judges, B builds. This uses both pools fully while avoiding the pattern the safety section warns against: automatic failover from an exhausted account to a fresh one.

**Escalate only once.** When two tiers fail on the same task, the task description is usually the problem. Send it back to A for re-scoping rather than further up the price ladder.

**Never use Codex's speed modes for background work.** Fast mode costs 2.5x included usage, and Ultrafast costs 8x on Astra and 6.1 Sol ([Codex pricing](https://learn.chatgpt.com/docs/pricing)).

**Treat free API tiers as disposable.** None is reliable enough to plan a workload around. Mistral's free Experiment plan may use prompts to train models ([awesome-free-llm-apis](https://github.com/mnfst/awesome-free-llm-apis/wiki)), so keep private code off any free tier whose terms allow training. The steadier sanctioned API pool is the one Anthropic already bundles: $200 a month of API credits on each Ultra account, $400 in total, which don't roll over ([codersera](https://codersera.com/blog/anthropic-june-2026-billing-change-claude-code/)). That is also the correct way to run Claude inside any third-party harness the user wants to try.

## Account safety means official clients doing your own work

All four vendors' rules point to the same sanctioned pattern. What varies is how strictly each one enforces it.

| Avoid | Examples | Why it's dangerous |
|---|---|---|
| Tools that turn subscriptions into an API, and multi-account load balancers | CLIProxyAPI, better-ccflare, claude-relay-service, antigravity-claude-proxy, sub2api | Prohibited in Google's and Anthropic's own terms and disowned by OpenAI's Codex lead; the most documented bans; one user lost Antigravity on 5 of 6 pooled accounts |
| Claude subscription login inside third-party harnesses | OpenCode auth plugins, oh-my-openagent `/login`, OpenClaw `setup-token`, the Happy Agent runtime | Anthropic forbids routing plan credentials through other apps; spoofing safeguards since Jan 2026; billed as separate extra usage since 4 Apr 2026 |
| Google login in anything other than Google's own clients | OpenClaw, opencode-antigravity-auth, Antigravity proxies | Antigravity's terms call it a breach; suspension waves in Feb and Sep 2026 hit paying AI Pro and Ultra subscribers |
| Automatic failover or pooling across the two Claude accounts | "When A hits its limit, switch to B" | Limits assume "ordinary, individual usage"; the terms forbid bypassing protective measures |
| Scripting consumer chat UIs | Browser automation of Muse, ChatGPT web or the Gemini app | Meta forbids automated access, OpenAI forbids programmatic extraction, Anthropic §3.7 forbids bots and scripts |
| Round-the-clock saturation loops | Unattended agents running all week | Anthropic introduced weekly limits specifically to stop this |
| Sharing, publishing or pasting credentials | Tokens in public repos, shared boxes or chat logs | All four vendors forbid credential sharing; Codex warns against public environments |

### Anthropic

Anthropic's rules are explicit. It "does not permit third-party developers to offer Claude.ai login into their own applications, or to route requests through Free, Pro, Max, or Ultra plan credentials on behalf of their users". Developers "may not collect, store, or intermediate Claude.ai credentials". The same document says nothing prevents "an end user from signing in to the unmodified Claude Code binary with their own Claude subscription" ([Claude Code legal and compliance](https://code.claude.com/docs/en/legal-and-compliance)).

Enforcement followed that line:

- January 2026: safeguards against spoofing the Claude Code client ([VentureBeat](https://venturebeat.com/technology/anthropic-cracks-down-on-unauthorized-claude-usage-by-third-party-harnesses)).
- 19 March 2026: OpenCode removed its Anthropic login after legal requests ([OpenCode PR #18186](https://github.com/anomalyco/opencode/pull/18186)).
- 4 April 2026: third-party harness usage moved off plan limits to separately billed extra usage ([TechCrunch](https://techcrunch.com/2026/04/04/anthropic-says-claude-code-subscribers-will-need-to-pay-extra-for-openclaw-support/)).

On holding two accounts, an Anthropic staffer said "It's not against terms of service to have multiple MAX accounts" ([PiunikaWeb](https://piunikaweb.com/2026/02/19/anthropic-claude-max-ban-agent-sdk-clarification/)). The Consumer Terms themselves say nothing on multiple accounts. They do forbid sharing an account and bypassing "protective measures" ([Consumer Terms](https://www.anthropic.com/legal/consumer-terms)). Both Ultra accounts should therefore belong only to the user, be used by no one else, and have distinct roles.

### OpenAI

OpenAI is the most permissive. Its Codex lead reminded users "you can use your ChatGPT account in a flourishing set of other tools" ([X, @thsottiaux](https://x.com/thsottiaux/status/2058071172361998482)). Codex docs say access tokens suit "trusted scripts, schedulers, and private CI runners", while still recommending API keys for automation ([Codex auth](https://learn.chatgpt.com/docs/auth.md)). The same lead said "We don't support sub2api" about tools that turn a subscription into a shared API ([X PSA](https://x.com/hqmank/status/2091541490959081977?lang=en)). One unexplained Codex ban, whose owner suspected a datacenter IP, is a single anecdote but argues for keeping VPS-driven Codex use modest ([OpenAI community](https://community.openai.com/t/codex-chatgpt-pro-account-banned-with-no-warning-no-explanation-18-month-subscriber/1381906)).

### Google

Google is the strictest. Its Antigravity terms call "using OpenClaw with Antigravity OAuth" a breach and forbid "using the Service in connection with products not provided by us" ([Antigravity terms](https://antigravity.google/terms?hl=en)). Enforcement has come in waves: in February 2026, paying subscribers were cut off, with most commenters saying only Gemini CLI/Antigravity access was lost and the Gemini app still worked ([OpenClaw #14203](https://github.com/openclaw/openclaw/issues/14203)); again in September ([Enterprise DNA](https://enterprisedna.co/resources/ai-pulse/ai-pulse-2026-09-03-google-suspends-paying-antigravity-subscribers-for-using-thi/)). Appeals restored access after days or weeks ([Hacker News](https://news.ycombinator.com/item?id=49548452)).

Developers have asked twice whether another program may launch the official `agy --print` binary, and Google has not answered ([Google AI forum](https://discuss.ai.google.dev/t/is-invoking-the-official-antigravity-cli-agy-print-from-a-third-party-developer-tool-an-acceptable-use/175462)). Its 28 Oct changelog added "a notice about third party developer tools" ([Antigravity changelog](https://antigravity.google/docs/changelog)). Keep any Antigravity scripting interactive-first and human-paced. If either Google account is the user's main Gmail identity, keep that account on Google's own apps only.

### Meta

Meta's terms require "Create only one account" and forbid automated access ([Meta Terms](https://www.facebook.com/terms.php)). Muse's shared chats leaked through passive DNS, and Meta suspended sharing in October 2026 ([Ars Technica](https://arstechnica.com/security/2026/10/meta-muse-exposes-users-personal-and-business-secrets-when-they-share-conversations/)). Queries train Meta's models by default ([Wikipedia](https://en.wikipedia.org/wiki/Muse_(AI_agent))). Keep secrets and private repo code away from it.

### The grey zones are narrower than they look

- **Your own script spawning `claude -p` for your own work** is documented and fine. Anthropic says the Agent SDK and `claude -p` on a plan "still draw from your subscription limits" ([Agent SDK help](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan)).
- **Anthropic's Agent SDK inside a third-party ACP client**, such as `claude-agent-acp` in Goose, is genuinely grey.
- **Codex in GitHub Actions on ChatGPT login** is labeled "advanced" and should only ever run in private repos.
- **PAL MCP's `clink` bridge** launches CLIs with `--yolo` and `--dangerously-bypass-approvals-and-sandbox` ([clink.md](https://raw.githubusercontent.com/BeehiveInnovations/pal-mcp-server/main/docs/tools/clink.md)). That is a security risk rather than a terms risk.

Following the rules still doesn't guarantee immunity. Anthropic reversed wrongful automated bans in January ([VentureBeat](https://venturebeat.com/technology/anthropic-cracks-down-on-unauthorized-claude-usage-by-third-party-harnesses)), and a Google user was flagged without ever using OpenClaw ([Hacker News](https://news.ycombinator.com/item?id=49548452)). Keep usage in human-paced bursts, and keep a recovery plan: log out, log back in, and regenerate tokens.

## Questions that could change this design

**Google's future decides whether it rejoins the coding plane.** Open points:

- whether Antigravity inherits Jules' label trigger, API and scheduled tasks, and gets a mobile surface;
- whether Google will say that a user's own program may launch `agy -p`;
- whether issue #318, the hang without a terminal, gets fixed;
- what Antigravity's quotas are in actual numbers;
- whether AI Plus gets any developer entitlement;
- whether `agy` can sign in on a headless Linux box without a desktop keyring;
- whether AI Pro's $10 monthly Google Cloud credit can fund a Gemini API key. That would give the Google subscription a sanctioned automation route; the notes infer it but don't confirm it.

**The Codex bridge rests on two unconfirmed points.** First, that Codex honours `@codex` comments posted through Claude's GitHub proxy under the user's identity. Second, that Plus counts as an "eligible plan" for Codex scheduled tasks triggered by events. If the first fails, Linear delegation is the fallback.

**Several Claude account mechanics are undocumented:**

- the maximum number of cloud sessions an Ultra account can run at once;
- how long verification of the second account takes before its five-hour cap lifts;
- whether a cloud session can call account B's `/fire` endpoint directly through a stored network secret, which would remove the GitHub Actions hop.

**Policy drift is the largest outside risk.** Anthropic announced, then paused, a plan to move Agent SDK and `claude -p` usage onto separate metered credit, and it could resume ([Agent SDK help](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan)). `--bare` is slated to become the default for `-p`. The notes couldn't reconcile Anthropic's 4 April move of third-party harnesses to extra usage with a June help-center line saying "third-party app usage still draw[s] from your subscription limits". Ultra's real weekly allowance is published only as "10x Pro".

**Muse's GitHub connector is undocumented.** It is unknown what the connector can do, when the promised agent email address arrives, and whether Muse's cloud computer runs code. These decide whether Muse can ever join the bus without hand-copying.

**Nobody has published measured reliability data** for a hosted Claude ↔ Codex ↔ Gemini loop over GitHub. Run a two-week pilot on one repo before trusting it with real deadlines.

## Conclusion

The binding constraint is not tokens but **sanctioned interfaces**. The user has plenty of quota spread across six seats. What is scarce is surfaces that let one agent hand work to another without touching credentials. Claude's per-task `/fire` endpoint is the only inbound door any subscription offers, which makes account A the hub by necessity rather than by preference. GitHub is the only neutral meeting point, because every vendor already ships an integration that listens to it. Two consequences follow. First, the cross-vendor glue should stay deliberately thin, as labels, comments and prompts rather than code. In 2026 alone, every vendor in the user's stack renamed, restricted or retired something, from Max becoming Ultra to Gemini CLI becoming `agy`, Jules shutting down and Codex's five-hour window being lifted and then restored. With thin glue, swapping a worker means changing a label. Second, the most overlooked asset is the $400 a month of API credits bundled with the two Ultra plans. That is the one pool that legitimately powers classifiers, routers and third-party harnesses with no terms ambiguity.

Google's position deserves a fresh look rather than more engineering. With Jules ending and Gemini CLI gone, the AI Pro and AI Plus plans now buy research, media, Colab and storage, not a programmable coding agent for a phone-first, server-free user. Treat them that way until Antigravity ships a background-agent surface, and check again after 31 December, when it will be clear what Antigravity actually inherited from Jules.
