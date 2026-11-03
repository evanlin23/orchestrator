# Vendor programmatic access and quotas for consumer AI subscriptions (as of 3 Nov 2026)

Scope: every way to drive each subscription non-interactively or from another program, plus published quotas. Policy/ToS only in passing. "Fetched" means the page itself was read; "(snippet)" means the claim came from a search-result summary and the page was not opened. Several plans were renamed or restructured in 2026; the dates matter.

**Headline changes the reader must know first**
- **Claude "Max 20x" no longer exists under that name.** On 30 Oct 2026 Anthropic renamed Max 5x to "Max" ($100) and Max 20x to "Ultra" ($200), and removed the 5-hour limit for verified Ultra accounts. The user's two "Max 20x" subscriptions are now two Ultra subscriptions.
- **Gemini CLI stopped serving consumer Google accounts (free, AI Pro, AI Ultra) on 18 Jun 2026.** Its consumer replacement is the closed-source Antigravity CLI. The well-known "1,000/1,500 requests per day" Gemini CLI quotas are historical for consumer accounts.
- **Codex on ChatGPT Plus has a 5-hour window again** (restored 25 Aug 2026, after being lifted on 12 Jul 2026), plus a weekly cap whose size is not published.
- **Meta "Muse" is a consumer agent launched 8 Sep 2026** that runs on its own cloud computer. No API or CLI for driving that consumer agent was found. Meta's developer API (Meta Model API, Muse Spark 1.1) is separate and pay-per-token.

---

## 1. Anthropic: Claude Max 20x (now "Ultra"), programmatic access and limits

### Takeaway
Claude has the richest subscription-backed programmatic surface of any vendor here, and all of it is documented:
- `claude -p` headless mode with JSON and stream-JSON output.
- The Agent SDK in Python and TypeScript.
- A one-year `claude setup-token` OAuth token for CI.
- The GitHub Action, which accepts a subscription OAuth token.
- Cloud sessions.
- Cloud "scheduled tasks" (formerly "routines") with an HTTP `/fire` endpoint that an external program can call to start a cloud session.

When you are signed in with the plan, all of these draw on the subscription's limits. Since 30 Oct 2026 the Ultra plan (formerly Max 20x) has no 5-hour cap for verified accounts. Its weekly cap is described as "10x Pro", and Fable, the top model, is capped at 50% of the weekly limit.

### Cited Findings

**Plan structure and limits (current)**
- On 30 Oct 2026, Max 5x was renamed "Max" ($100/mo, €90) and Max 20x was renamed "Ultra" ($200/mo, €180). The five-hour limit is removed on Ultra for verified accounts, and weekly limits are "unchanged". Newer Ultra accounts keep a temporary five-hour limit until Anthropic verifies them (anti-fraud) and are emailed when it is lifted. Anthropic: "If you're a subscriber, there's nothing you need to do." — [TNW, 1 Nov 2026](https://thenextweb.com/news/claude-ultra-max-plan-rename-five-hour-limit) (fetched)
- The pricing page lists:
  - Pro: $20/mo ($17/mo billed annually).
  - Max: $100, "5x Pro usage" every five hours and every week, "$100 a month in API credits", "Fable included" up to half your weekly limit.
  - Ultra: $200, "No 5-hour limit" (verified accounts), "10x Pro usage" weekly, "$200 a month in API credits", Fable at "50% of weekly limits".
  - Opus, Sonnet and Haiku are on all paid plans. Context is "up to 1M". Max and Ultra get priority access at high-traffic times. — [claude.com/pricing](https://claude.com/pricing) (fetched)
- TNW notes Ultra's weekly allowance is listed as 10x Pro, "down from the 20x the old 'Max 20x' name implied", and that Anthropic's post did not mention this. — [TNW](https://thenextweb.com/news/claude-ultra-max-plan-rename-five-hour-limit)
- Pricing-page caveat: usage limits depend on the model used, chat length and file size. The page gives no numeric message or hour counts. — [claude.com/pricing](https://claude.com/pricing)
- Pro and Max share one usage pool across claude.ai and Claude Code. — [search summary of third-party guides](https://www.ai-toolbox.co/claude-management-and-productivity/claude-usage-limits-2026) (snippet)
- Widely repeated figures of "240–480 Sonnet hours / 24–40 Opus hours per week" for Max 20x trace back to Anthropic's July 2025 subscriber emails. They are not a current published guarantee, and current docs give no fixed per-model hour counts. — [explainx.ai timeline](https://www.explainx.ai/blog/claude-usage-limits-2026-timeline-explained) and [morphllm](https://www.morphllm.com/claude-code-usage-limits) (snippets). The only per-model cap on the current pricing page is the Fable share. Treat older Opus/Sonnet bucket numbers as stale.

**Limit history in 2026 (context for any hard-coded numbers)**
- May 2026: Anthropic doubled Claude Code's five-hour limits for Pro, Max, Team and seat-based Enterprise, citing new SpaceX compute. — [Anthropic, "Higher usage limits and a SpaceX compute deal"](https://www.anthropic.com/news/higher-limits-spacex) (snippet); corroborated by [TNW](https://thenextweb.com/news/claude-ultra-max-plan-rename-five-hour-limit)
- Promotion: "From May 13, 2026 through August 31, 2026, your weekly usage limit in Claude Code is 50% higher. 5-hour usage limits are not affected." — [support.claude.com 15910845](https://support.claude.com/en/articles/15910845-claude-code-may-july-2026-weekly-limits-promotion) (snippet). A page with the same slug reportedly says weekly limits returned to standard after 19 Jul 2026, so there is a conflict on the end date (snippet).
- A single unverified source claims weekly limits rose 25% on 14 Sep 2026 and five-hour limits rose on 22 Sep 2026. — [search summary](https://www.morphllm.com/claude-code-usage-limits) (snippet; not corroborated)
- April 2026: Anthropic blocked OpenClaw and other third-party agent tools from using Claude subscriptions. June 2026: a lawsuit alleged both Max plans oversold usage. — [TNW](https://thenextweb.com/news/claude-ultra-max-plan-rename-five-hour-limit) (policy note only)

**Authentication with a subscription**
- Claude Code accepts "Claude Pro, Max, or Ultra subscription: log in with your claude.ai account" (browser OAuth). `/login` subscription OAuth is "the default for Claude Pro, Max, Ultra, Team, and Enterprise users". — [code.claude.com/docs/en/authentication](https://code.claude.com/docs/en/authentication) (fetched)
- **Multiple accounts on one machine:** set `CLAUDE_CONFIG_DIR` per account (e.g. `alias claude-work='CLAUDE_CONFIG_DIR=~/.claude-work claude'`). "Each directory has its own settings, session history, and claude.ai login or API key." On Linux, credentials are in `$CLAUDE_CONFIG_DIR/.credentials.json` (mode 0600). — [authentication docs](https://code.claude.com/docs/en/authentication)
- **`claude setup-token`** "generate[s] a one-year OAuth token". It prints the token and does not save it; you set it as `CLAUDE_CODE_OAUTH_TOKEN`. The token "requires a Pro, Max, Ultra, Team, or Enterprise plan" and "can only make model requests, so it can't establish Remote Control sessions or fetch claude.ai connectors. MCP servers you configure locally still work." — [authentication docs](https://code.claude.com/docs/en/authentication)
- **Credential precedence**, highest first:
  1. Cloud-provider env vars
  2. `ANTHROPIC_AUTH_TOKEN`
  3. `ANTHROPIC_API_KEY` ("In non-interactive mode (`-p`), the key is always used when present")
  4. `apiKeyHelper`
  5. `CLAUDE_CODE_OAUTH_TOKEN`
  6. Anthropic profile or federation credentials
  7. Subscription OAuth from `/login`

  A stray `ANTHROPIC_API_KEY` therefore silently switches `-p` runs to API billing. — [authentication docs](https://code.claude.com/docs/en/authentication)
- An unattended login can expire. Claude Code warns 3 days before, and `/status` shows `Login: Expired`. "A background session in agent view or a Remote Control session that outlives the login stops making progress once the credential expires." — [authentication docs](https://code.claude.com/docs/en/authentication)
- "Cloud sessions always use your subscription credentials. If you set `ANTHROPIC_API_KEY` or `ANTHROPIC_AUTH_TOKEN` in the cloud environment, it doesn't override your subscription credentials." — [authentication docs](https://code.claude.com/docs/en/authentication)

**Headless mode (`claude -p`)**
- `claude -p "<prompt>"` (or `--print`) runs non-interactively and reads stdin (piped input capped at 10 MB). It exits 0 on success and non-zero on failure. — [code.claude.com/docs/en/headless](https://code.claude.com/docs/en/headless) (fetched)
- Output and session flags:
  - `--output-format text|json|stream-json`.
  - `json` includes `result`, `session_id`, `total_cost_usd` and a per-model cost breakdown (client-side estimates).
  - `--json-schema` returns a `structured_output` field.
  - Stream-JSON needs `--verbose`; add `--include-partial-messages` for token deltas. The stream emits `system/init`, `system/api_retry` (error categories include `rate_limit`) and a final `result` message.
  - `--continue` and `--resume <session_id>` support multi-turn scripted conversations. — [headless docs](https://code.claude.com/docs/en/headless)
- Permissions:
  - `--allowedTools`.
  - `--permission-mode auto|dontAsk|acceptEdits`.
  - `--permission-prompts none` for unattended runs (v2.1.259+).
  - From v2.1.302, `-p` runs start in Manual mode unless a mode is set. — [headless docs](https://code.claude.com/docs/en/headless)
- **`--bare` mode does NOT use subscription login.** "In bare mode, Claude Code never reads OAuth credentials or the system keychain", and it does not read `CLAUDE_CODE_OAUTH_TOKEN`. It "will become the default for `-p` in a future release". An orchestrator on subscription auth must avoid `--bare`, and should watch for that default change. — [headless docs](https://code.claude.com/docs/en/headless); [authentication docs](https://code.claude.com/docs/en/authentication)
- `--cloud <session-id>` with `-p` "queues a message into that cloud session and exits". A local script can message a running cloud session. — [headless docs](https://code.claude.com/docs/en/headless)

**Agent SDK with a subscription; billing changes in 2026**
- The Agent SDK (Python and TypeScript) is "the same tools, agent loop, and context management that power Claude Code"; `claude -p` is its CLI form. — [headless docs](https://code.claude.com/docs/en/headless)
- Signed in with your Claude plan, the Agent SDK and `claude -p` "still draw from your subscription limits". On 15 Jun 2026 Anthropic "paused the previously-announced changes to Claude Agent SDK usage". Third-party apps built on the Agent SDK also still draw from subscription limits. — [support.claude.com 15036540](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan) (fetched; "updated this week")
- Since 7 Oct 2026, Max, Ultra and Team include monthly API credits. They cover the Claude API, Claude Managed Agents, and the Agent SDK and `claude -p` *when run with an API key from a linked Claude Console organization*. — [support.claude.com 15036540](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan). Amounts: $100/mo on Max, $200/mo on Ultra — [claude.com/pricing](https://claude.com/pricing). The credits don't roll over and don't raise plan limits. — [codersera summary](https://codersera.com/blog/anthropic-june-2026-billing-change-claude-code/) (fetched, third-party)
- Background: on 13 May 2026 Anthropic announced that Agent SDK, `claude -p`, GitHub Actions and third-party SDK apps would "move off subscription limits onto a separate monthly dollar credit, metered at API prices" from 15 Jun. That change was paused and has not taken effect as of the 26 Oct 2026 Help Center state. — [codersera](https://codersera.com/blog/anthropic-june-2026-billing-change-claude-code/); [support.claude.com 15036540](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan)

**GitHub Action (`anthropics/claude-code-action@v1`)**
- Auth inputs are `anthropic_api_key` or `claude_code_oauth_token`. The OAuth token comes from `claude setup-token` and works on "Pro, Max, Ultra, Team, and Enterprise plans". "If you authenticate with an OAuth token, runs use your Claude subscription instead of API billing." `/install-github-app` can mint and store the subscription token as the repo secret `CLAUDE_CODE_OAUTH_TOKEN`. — [code.claude.com/docs/en/github-actions](https://code.claude.com/docs/en/github-actions) (fetched)
- Modes:
  - Interactive (default): responds to `@claude` mentions in issues and PRs.
  - Automation: set a `prompt` input to run on any event, including cron.
- Caveat: "an OAuth token is tied to the subscription of the person who ran `claude setup-token`". The docs recommend a Console API key for org-shared secrets. — [github-actions docs](https://code.claude.com/docs/en/github-actions)

**Cloud scheduled tasks ("routines"): an HTTP-triggerable subscription worker**
- A scheduled task is a saved prompt plus repos, connectors and an environment. It runs as a full Claude Code cloud session on Anthropic infrastructure.
- Triggers: Schedule (cron, minimum hourly), API (HTTP POST), GitHub events. Available on Pro, Max, Ultra, Team and Enterprise. Manage at claude.ai/code/routines or with `/schedule` in the CLI. — [code.claude.com/docs/en/routines](https://code.claude.com/docs/en/routines) (fetched; "research preview")
- **API trigger:** `POST https://api.anthropic.com/v1/claude_code/routines/<trig_id>/fire` with `Authorization: Bearer <per-task token>` and an optional `text` body.
  - It returns `claude_code_session_id` and `claude_code_session_url`.
  - The beta header is `experimental-cc-routine-2026-04-01`.
  - The `text` arrives wrapped in `<routine-fire-payload>` as untrusted data. The saved prompt must opt in to acting on it.
  - "The `/fire` endpoint is available to claude.ai users only and is not part of the Claude Platform API surface." — [routines docs](https://code.claude.com/docs/en/routines)
- Limits:
  - Scheduled runs: 100 per hour per account.
  - "Run now", API fires and one-off re-runs: 30 per hour per task, combined.
  - GitHub webhook events have per-task and per-account hourly caps.
  - There is no overage on these caps.
  - Runs "draw down subscription usage the same way interactive sessions do" and "count against your account's usage and limits".
  - At the subscription limit, runs are rejected unless usage credits are turned on. — [routines docs](https://code.claude.com/docs/en/routines)
- `/schedule` requires a claude.ai subscription login. It is unavailable with a Console API key, a profile or cloud-provider auth, or inside a cloud session (use the web UI there). — [routines docs](https://code.claude.com/docs/en/routines)
- "Projects" coordinate "ongoing work Claude coordinates across parallel cloud sessions". Scheduled tasks created from a project can continue a thread rather than start a new session. — [routines docs](https://code.claude.com/docs/en/routines)

### Inferences
- With two Ultra accounts, keep two separate `CLAUDE_CONFIG_DIR`s (or two `setup-token` tokens). Limits, scheduled tasks and routine tokens all belong to the individual account, so each account is an independent quota pool. This follows from per-account statements in the auth and routines docs; no doc speaks to multiple paid accounts held by one person.
- Ultra is now the best "bulk heavy-reasoning worker" in this set: no 5-hour throttle once the account is verified, and a weekly-only cap. Fable is the scarce resource (≤50% of weekly), so route Fable-worthy tasks deliberately and default other work to Opus or Sonnet.
- Three subscription-backed ways for an external orchestrator to dispatch Claude work:
  - Locally: `claude -p --output-format stream-json` or the Agent SDK, on OAuth.
  - In the cloud: routines `/fire`, which returns a session URL.
  - Follow-ups: `claude -p --cloud <session-id>` to queue messages.
- Observed first-hand (no public doc URL): this research session's own Claude Code cloud environment exposes tools such as `create_session`, `send_message`, `create_trigger` and `send_later`. One cloud session can programmatically spawn and message sibling cloud sessions and schedule tasks. This supports "session starts/messages another session" as an existing capability.
- The October API credits ($200/mo on Ultra) are a separate small API-key pool (Console-linked key). They can serve `--bare`, SDK or API workloads without touching subscription limits.

### Gaps
- No official numeric weekly limit (tokens, messages or hours) for Ultra or Max. Anthropic publishes only multipliers versus Pro. Exact Opus vs Sonnet vs Haiku allowances (other than the Fable ≤50% cap) are unpublished.
- No primary Anthropic post on the 30 Oct rename was opened. TNW cites Anthropic's X post, and claude.com/pricing confirms the plan table.
- How long "verification" takes for Ultra accounts, and whether a second Ultra account held by the same person is treated differently, is unknown.
- Maximum concurrent cloud sessions per individual account was not found. The routines doc only mentions org-level concurrent-session limits.

---

## 2. OpenAI: ChatGPT Plus and Codex

### Takeaway
On ChatGPT Plus, Codex can be driven non-interactively in several ways:
- `codex exec` (JSONL events, output schemas, resume).
- The Codex SDK (TypeScript and Python, which drives the local app-server).
- Codex cloud tasks.
- `@codex` on GitHub PRs.

Usage comes from a shared allowance with a rolling 5-hour window (restored for Plus on 25 Aug 2026) plus an unpublished weekly cap. Cloud tasks cost more of the allowance than local messages. GitHub reviews have their own "code review" usage category. The official Codex GitHub Action is documented with an API key only. ChatGPT-login auth in CI is "advanced" and discouraged for public repos.

### Cited Findings

**Plus usage limits (official Codex pricing page; `developers.openai.com/codex/pricing` now redirects to `learn.chatgpt.com/docs/pricing`)**
- Local messages per 5-hour window, Plus and Standard Business (estimates, "not fixed message limits"):

  | Model | Messages per 5 h |
  |---|---|
  | GPT-6 Astra | 5–45 |
  | GPT-6.1 Sol | 15–160 |
  | GPT-6.1 Terra | 35–285 |
  | GPT-6 Sol | 15–150 |
  | GPT-6 Luna | 350–3,000 |

  — [learn.chatgpt.com/docs/pricing](https://learn.chatgpt.com/docs/pricing) (fetched)
- "Weekly limits may apply", with no numbers. Cloud tasks have no separate number: they "may use more of your allowance than local messages", and local messages and cloud chats share the same allowance. Code reviews run through GitHub count toward a *separate* code-review usage category; local reviews count toward general limits. — [pricing](https://learn.chatgpt.com/docs/pricing)
- Credits:
  - Plus and Pro can buy extra credits after hitting limits.
  - Credit rates per 1M tokens, e.g. GPT-6.1 Sol: 50 input / 2.5 cached / 250 output; GPT-6 Luna: 2.5 / 0.25 / 12.5. "A typical GPT-5.6 Sol task may use 2–15 credits."
  - Fast mode costs 2.5x included usage (2x credits). Ultrafast costs 8x included usage (6x credits) on Astra and 6.1 Sol. — [pricing](https://learn.chatgpt.com/docs/pricing)
- Pro ($100 / $200 / $500 per month): "Pro plans currently have no five-hour limit". Enterprise and Edu flexible plans scale with credits. GPT-5.5 retired from Codex with ChatGPT sign-in on 14 Oct 2026; API-key sessions are unaffected. — [pricing](https://learn.chatgpt.com/docs/pricing)
- The pricing page lists "Codex SDK, codex exec, and scriptable workflows" as a feature, with no separate limits for SDK, exec or GitHub. — [pricing](https://learn.chatgpt.com/docs/pricing)
- Timeline:
  - OpenAI lifted the 5-hour limit on Codex and ChatGPT Work around 12 Jul 2026.
  - It restored the limit for Plus from 25 Aug 2026. Thibault Sottiaux: it lets OpenAI "smoothen the load on our compute, allowing to keep the plan generous in terms of weekly usage".
  - The 5-hour limit stays off for $100 and $200 Pro "for the coming months". — [9to5Mac, 24 Aug 2026](https://9to5mac.com/2026/08/24/openai-restores-5-hour-codex-and-work-limits-for-chatgpt-plus-users/); [TechRepublic](https://www.techrepublic.com/article/news-openai-five-hour-codex-limit-chatgpt-plus/) (snippets; consistent across outlets)
- Check usage with `/status` in the CLI or under Settings → Usage, which shows reset times. — [jdhodges guide](https://www.jdhodges.com/blog/how-to-check-codex-usage-chatgpt-plus/) (snippet)
- One source says Pro 20x ($200) new sign-ups have been paused since 10 Sep 2026. — [developersdigest](https://www.developersdigest.tech/blog/codex-usage-limits-pricing-2026) (snippet; unverified)

**Authentication**
- Three sign-in methods:
  - "Sign in with ChatGPT for subscription access" (browser).
  - API key (`codex login --with-api-key`).
  - Device code (beta: `codex login --device-auth`; enable it first in ChatGPT security settings).
- "Codex cloud requires signing in with ChatGPT." — [learn.chatgpt.com/docs/auth.md](https://learn.chatgpt.com/docs/auth.md) (fetched)
- Headless machines: log in where a browser exists, then copy `~/.codex/auth.json` (under `CODEX_HOME`, default `~/.codex`). Codex "refreshes tokens automatically during use before they expire". Treat the file like a password. "API keys are still the recommended default for automation." Codex access tokens (`CODEX_ACCESS_TOKEN`) are an Enterprise-workspace feature only. — [auth docs](https://learn.chatgpt.com/docs/auth.md)
- Concurrent CI jobs that share one `auth.json` can race during token refresh. — [danielvaughan.com guide](https://codex.danielvaughan.com/2026/04/01/codex-cli-authentication-flows-credential-management/) (snippet, third-party)

**`codex exec` (non-interactive)**
- `codex exec "<task>"` sends progress to stderr and only the final message to stdout. Flags:
  - `--json`: JSONL events such as `thread.started`, `turn.completed`, `error`.
  - `--output-schema <file>`: structured final answer.
  - `-o/--output-last-message`: write the final message to a file.
  - `codex exec resume --last` or `resume <SESSION_ID>`.
  - `--sandbox read-only` (default), `workspace-write`, or `danger-full-access`.
  - `--full-auto` is removed; use `--sandbox workspace-write` or `--approve-for-me`.
  - `--ephemeral`, `--skip-git-repo-check`, `--ignore-user-config`. — [learn.chatgpt.com/docs/non-interactive-mode.md](https://learn.chatgpt.com/docs/non-interactive-mode.md) (fetched)
- "By default, `codex exec` reuses saved CLI authentication", so it runs on ChatGPT login if that is what is saved. `CODEX_API_KEY=<key>` can be set inline per run. ChatGPT-managed auth in CI is an "advanced" option "for users who need ChatGPT/Codex rate limits instead of API key usage". "Don't use ChatGPT-managed auth in public or open-source repositories." — [non-interactive docs](https://learn.chatgpt.com/docs/non-interactive-mode.md)
- **GitHub Action `openai/codex-action`:** the docs show it with the `openai-api-key` input from `secrets.OPENAI_API_KEY`. It starts a Responses API proxy to protect the key. No ChatGPT-plan auth is shown for the Action. — [non-interactive docs](https://learn.chatgpt.com/docs/non-interactive-mode.md)

**Codex SDK**
- TypeScript `@openai/codex-sdk` (Node 18+, server-side) and Python `openai-codex` (3.10+). The Python SDK "controls the local Codex app-server over JSON-RPC". Threads support `startThread` / `run` / `resumeThread`, and in Python `thread_start` / `thread_resume` / `thread_fork`. Use the app server "to build custom clients that handle authentication, conversation history, approvals, and streamed agent events". — [learn.chatgpt.com/docs/codex-sdk.md](https://learn.chatgpt.com/docs/codex-sdk.md) (fetched)
- The SDK page does not say whether ChatGPT-plan auth applies. It says models "must be available to your signed-in account". — [codex-sdk docs](https://learn.chatgpt.com/docs/codex-sdk.md)

**GitHub `@codex` and cloud**
- `@codex review` on a PR gives a review that "flags only P0 and P1 issues". `@codex <anything else>` (e.g. "fix the CI failures") "starts a legacy cloud chat" with the PR as context and can push fixes. Automatic reviews can be turned on per repo. Setup: connect the repo to Codex and enable code review. The page covers PRs only, not issues. — [learn.chatgpt.com/docs/third-party/github.md](https://learn.chatgpt.com/docs/third-party/github.md) (fetched)
- Plus ($20) includes Codex web, CLI, IDE extension, iOS, cloud code review and Slack integration. API-key auth covers CLI, SDK and IDE only, with no cloud features such as GitHub review or Slack. — [developersdigest](https://www.developersdigest.tech/blog/codex-usage-limits-pricing-2026) and [morphllm codex pricing](https://www.morphllm.com/codex-pricing) (snippets)
- Plus quotas are not usable through the OpenAI API. API-key Codex usage is billed "based on API pricing". — [pricing](https://learn.chatgpt.com/docs/pricing)

### Inferences
- Plus is best treated as a mid-volume worker with a hard 5-hour throttle. Route cheap, high-volume work to the Luna or Terra tiers (hundreds to thousands of local messages per 5 h). Reserve Astra or Sol for hard tasks (5–160 per 5 h). Avoid cloud tasks for bulk work, because they drain the shared allowance faster.
- For an orchestrator, `codex exec --json` plus saved ChatGPT `auth.json` (one `CODEX_HOME` per account) is the subscription-backed path. Don't run parallel jobs against one `auth.json`, because of refresh races. `@codex` on PRs is a second, GitHub-driven path with its own review-usage bucket.
- The official Codex GitHub Action appears to be API-key only. To run Codex in Actions on a Plus quota you would install the CLI and seed `auth.json` yourself, which OpenAI calls advanced and warns against in public repos.

### Gaps
- No published weekly cap and no per-5h numbers for Plus cloud tasks or code reviews; only the local-message ranges above.
- ChatGPT "agent mode" (ChatGPT agent) message limits on Plus were not found in this pass.
- Whether the Codex SDK officially supports ChatGPT-login auth is not stated. It likely inherits the CLI/app-server's saved auth, but that is unconfirmed.

---

## 3. Google: AI Plus vs AI Pro (Gemini CLI → Antigravity CLI, Jules, Antigravity, Colab, free tiers)

### Takeaway
As of 18 Jun 2026, Gemini CLI no longer serves consumer Google accounts (free, AI Pro, AI Ultra). The subscription-backed terminal agent is now Antigravity CLI (`agy`, closed source).

Antigravity quotas are unpublished:
- AI Pro: a "high" quota that refreshes every 5 h until a weekly cap.
- Free/other plans (which in practice appears to include AI Plus): a "meaningful" quota that refreshes weekly. Community reports describe it as very low.

Jules is the clearest numeric quota:
- Free: 15 tasks per 24 h, 3 concurrent.
- AI Pro: 100 per 24 h, 15 concurrent.
- Ultra: 300 per 24 h, 60 concurrent.
- AI Plus is not listed.

Jules has a CLI (`jules remote new`) and a REST API. AI Plus adds little or nothing for developer tools. AI Pro is the useful tier, and holding two AI Pro accounts doubles Jules and Antigravity capacity.

### Cited Findings

**Gemini CLI → Antigravity CLI transition**
- Google announcement (posted 19 May 2026): "Today, June 18, 2026, marks the official transition of our terminal experience from Gemini CLI to the new Antigravity CLI. Starting today, Gemini CLI will stop serving requests for Google AI Pro, Google AI Ultra, and free tier individual accounts."
  - Also affected: Gemini Code Assist IDE extensions and Gemini Code Assist on GitHub on those tiers.
  - Still served: Code Assist Standard and Enterprise, Google Cloud, and paid Gemini / Gemini Enterprise Agent Platform API keys.
  - Gemini CLI "remains available to the community as an Apache 2.0 licensed repository". Antigravity CLI auto-imports Gemini CLI config. — [GitHub discussion #27274](https://github.com/google-gemini/gemini-cli/discussions/27274) (fetched)
- User comments on that thread:
  - "antigravity has low super low quota and 7 days.. of refresh quota".
  - Strict limits are shared across all Gemini models, so hours of work now last 10–20 minutes.
  - Weekly quota ran out "in under 10 minutes".
  - One user cancelled.
  - One account showed "Antigravity Starter Quota". — [discussion #27274](https://github.com/google-gemini/gemini-cli/discussions/27274) (anecdotal)
- There was no grace period. — [The New Stack](https://thenewstack.io/gemini-cli-antigravity-replacement/) (snippet)
- Antigravity CLI keeps Agent Skills, Hooks, Subagents and Extensions (now "Antigravity plugins"). It is a closed binary. — [letsdatascience](https://letsdatascience.com/news/google-replaces-gemini-cli-with-antigravity-cli-574b2056) (snippet)
- It supports multiple models (Gemini, Claude, GPT-OSS). — [buildfastwithai](https://www.buildfastwithai.com/blogs/google-antigravity-2-0-developer-guide-2026) (snippet)
- The command is `agy`. — [discussion comments](https://github.com/google-gemini/gemini-cli/discussions/27274); [search summary](https://lilachbullock.com/google-antigravity) (snippet)
- **Historical Gemini CLI quota table** (page last updated 18 Jun 2026; consumer rows now superseded), requests per user per day:

  | Plan | Requests/day |
  |---|---|
  | Code Assist Individual (free Google login) | 1,000 |
  | Google AI Pro | 1,500 |
  | Google AI Ultra | 2,000 |
  | Gemini API key, free tier (Flash model only) | 250 |
  | Code Assist Standard | 1,500 |
  | Code Assist Enterprise / Workspace AI Ultra | 2,000 |

  - "Tiers not listed, including Google AI Plus, are not supported."
  - Requests span the Gemini model family "as determined by Gemini CLI".
  - Vertex AI Express mode is free for 90 days with no billing.
  - Banner: "Gemini CLI was replaced by Antigravity CLI on June 18th, 2026" for unpaid-tier and Google One users.
  - Gemini CLI "has not been optimized" for Gemini 4 Argon. — [geminicli.com quota-and-pricing](https://geminicli.com/docs/resources/quota-and-pricing/) (fetched)

**Antigravity (app, IDE, CLI, SDK) quotas**
- Antigravity 2.0 launched at I/O on 19 May 2026: desktop app with parallel agents and scheduled background tasks, a new CLI, and an SDK for custom agents. It is powered largely by Gemini 3.5 Flash. A new $100 AI Ultra tier gives "5x higher AI limits in Antigravity than the Pro plan". The top Ultra tier dropped from $250 to $200 with "20x higher limits than the Pro plan". — [TechCrunch, 19 May 2026](https://techcrunch.com/2026/05/19/google-launches-antigravity-2-0-with-an-updated-desktop-app-and-cli-tool/) (fetched)
- Official plans page:
  - Ultra: "The highest, most generous quota, refreshed every five hours", "highest weekly rate limits", third-party models.
  - Pro: "A high, generous quota, refreshed every five hours until the weekly limit is reached", "A higher weekly rate limit".
  - Plans other than Pro and Ultra: "A meaningful quota, refreshed weekly" and "A weekly rate limit".
  - The core agent model is "Gemini 3.8 Flash and other offered Gemini Enterprise models".
  - All plans get "Access to all product features, such as scheduled tasks and the CLI".
  - Pro and Ultra can spend purchased AI credits beyond quota (overage setting: Never or Always), at Gemini Enterprise consumption pricing. — [antigravity.google/docs/plans](https://antigravity.google/docs/plans) (fetched)
- Pricing page:
  - Individual: $0, "basic weekly rate limits", unlimited Tab completions and Command requests.
  - AI Pro: more limits plus a "Flexible AI credit pool".
  - AI Ultra: "Even more rate limits".
  - Business via Google Cloud: from $30/seat/mo, or pay-as-you-go.
  - Google AI Plus is not mentioned. — [antigravity.google/pricing](https://antigravity.google/pricing) (fetched)
- Third-party sources place AI Plus in the free bucket for Antigravity (no added quota). — [engineerfoodie](https://engineerfoodie.in/blog/antigravity-plus-vs-pro/) and [axonbuild](https://axonbuild.com/blog/google-antigravity-pricing/) (snippets)

**Google AI Plus vs AI Pro (plan page)**
- AI Plus: Gemini 3.6 Flash and Deep Research, "double the access to Gemini", "Includes Gemini 4 Argon" (limits unstated), more media generation, 400 GB storage, family sharing. No developer-tool items are listed.
- AI Pro: "four times more access to Gemini", Gemini 4 Argon, expanded Gemini 3.1 Pro, expanded Deep Research, Tasks in Gemini. Developer items:
  - "Expanded access to Gemini 4 Argon in Google Antigravity and Google AI Studio".
  - Expanded Android Studio limits.
  - Colab "200 compute units per month".
  - "$10 in monthly Google Cloud credits from Google Developer Program".
  - 1,000 Flow credits per month and 5 TB storage.
- Ultra: 5x Pro usage in Gemini, Colab 2,000 CU/mo, $40/mo GCP credits, Gemini Agent (US, English).
- Prices were blank in the extracted page. The extraction also listed "Ultra 20x tier at $100", which conflicts with TechCrunch ($100 = 5x, $200 = 20x); treat it as an extraction error. — [one.google.com/about/google-ai-plans](https://one.google.com/about/google-ai-plans/) (fetched); [TechCrunch](https://techcrunch.com/2026/05/19/google-launches-antigravity-2-0-with-an-updated-desktop-app-and-cli-tool/)

**Jules (async cloud coding agent)**
- Official limits:

  | Tier | Tasks per rolling 24 h | Concurrent | Model |
  |---|---|---|---|
  | Jules (base) | 15 | 3 | Gemini 2.5 Pro |
  | Jules in Pro (with Google AI Pro) | 100 | 15 | "higher access to the newest model, starting with Gemini 3 Pro" |
  | Jules in Ultra | 300 | 60 | priority access |

  - Paid tiers come only through Google AI Plans and are "currently available only for individual Google Accounts (ending in @gmail.com)". Users must be 18+.
  - Google AI Plus is not listed.
  - The docs nav includes REST API, Jules Tools (CLI), Integrations and Scheduled tasks. — [jules.google/docs/usage-limits](https://jules.google/docs/usage-limits) (fetched; no last-updated date; model names may be stale)
- Jules Tools CLI (launched 2 Oct 2025):
  - Install with `npm install -g @google/jules` (or run `npx @google/jules`), then `jules login` with a Google account.
  - `jules remote new --repo <owner/repo> --session "<task>"` creates a task; `jules remote list` monitors.
  - It accepts piped stdin, e.g. `gh issue list ... | jq ... | jules remote new --repo .`
  - A Jules API early preview launched the same day. — [Google Developers Blog](https://developers.googleblog.com/meet-jules-tools-a-command-line-companion-for-googles-async-coding-agent/); [Jules changelog 2025-10-02](https://jules.google/docs/changelog/2025-10-02) (snippets; 2025 sources, so the CLI and API may have changed)

**Free Gemini API (AI Studio) tier**
- In Gemini CLI, a free API key gets 250 requests/day, Flash only. — [geminicli.com](https://geminicli.com/docs/resources/quota-and-pricing/) (fetched)
- Sources conflict on the broader free API tier:
  - Pro models reportedly became paid-only from 1 Apr 2026, with the free tier Flash-only. — [agentdeals.dev](https://agentdeals.dev/gemini-api-pricing-changes) (snippet)
  - Another source says Flash, Flash-Lite and 2.5 Pro are free as of 30 Sep 2026. — [yingtu.ai](https://yingtu.ai/en/blog/google-gemini-api-free-tier-limits-2026) (snippet)
  - Figures of roughly 15 RPM / 1,500 RPD for Flash also circulate, alongside reports of cuts to 20–100 RPD. — [snippets](https://tinkerllm.com/blog/gemini-api-free-tier-limits-rate-quotas/)
  - Limits are per Google Cloud project and reset at midnight Pacific. A Google AI Pro or Ultra subscription does not add API-key quota. — (snippets)

### Inferences
- **AI Plus is close to useless for orchestration.** It is not a supported Gemini CLI tier, is not named for Jules, and is not named for Antigravity. Expect free-tier Antigravity (weekly "meaningful" quota) and base Jules (15 tasks/day). Use it only as a chat-app or Deep Research seat.
- **Each AI Pro account gives:**
  - Jules at 100 tasks/24 h with 15 concurrent, the strongest numeric async-agent quota in this set.
  - Antigravity Pro quota (5-hour refresh plus a weekly cap).
  - 200 Colab CU per month.
  - $10/mo of GCP credit (usable for paid Gemini API keys, which still work with open-source Gemini CLI).

  Two AI Pro accounts give two independent pools of each. Jules needs @gmail.com accounts.
- For headless Google work, `jules remote new` (CLI or API) is the documented scriptable path. Antigravity CLI headless flags were not confirmed, and quota numbers are opaque and reportedly low.
- Open-source Gemini CLI plus a free AI Studio key is a "cheap worker" path: 250 requests/day, Flash only, per project.

### Gaps
- Antigravity CLI non-interactive or headless flags, output formats, and whether quota is pooled across app, IDE, CLI and SDK were not found (the getting-started page redirected and was not read).
- No numeric Antigravity quotas for any tier.
- The status of the Gemini CLI GitHub Action (`run-gemini-cli`) after 18 Jun 2026 was not checked. It presumably works with paid or free API keys or Vertex, but that is unverified.
- Current Jules API version, auth (API key vs OAuth) and rate limits were not verified beyond the 2025 launch posts.
- Whether AI Plus subscribers get any Jules uplift is unconfirmed. The official page lists only base, Pro and Ultra.

---

## 4. Meta: "Muse" free tier (Muse agent, Muse Spark models, developer access)

### Takeaway
"Muse" refers to two different things:
- **Muse**, Meta's consumer personal agent launched 8 Sep 2026. It runs on its own cloud computer with a real browser and connectors, is available in the Muse/Meta AI app, web and WhatsApp, and is limited to US adults 18+. The free tier reportedly allows about 100M tokens/week.
- **Muse Spark**, the model family behind it.

No API, CLI or programmatic interface to the consumer Muse agent or its free quota was found. Developer access exists only through the pay-per-token Meta Model API: `muse-spark-1.1` at `api.meta.ai/v1`, OpenAI- and Anthropic-compatible, US only.

### Cited Findings
- Muse is "the personal agent" and Muse Spark is "the model powering it". It launched on 8 Sep 2026 and runs on its own cloud computer, keeping working after you close the app. Availability:
  - U.S. adults 18 and older.
  - iOS, Android and web; AI glasses later.
  - Usable through the Muse app or WhatsApp.
  - Connectors with per-service permission controls (e.g. read vs send email).
  - Muse Spark 1.3 was released 2 Sep 2026. — [The Neuron explainer](https://theneuron.ai/explainer-articles/how-to-get-started-with-meta-muse/) (fetched); [Axios, 8 Sep 2026](https://axios.com/2026/09/08/meta-debuts-muse-personal-ai-agent) (snippet)
- Free-tier cap: "up to 100 million tokens per week", via Alex Heath's interview and attributed to Zuckerberg's launch post. Meta's newsroom says only that Muse is "free for most of what people need". — [The Neuron](https://theneuron.ai/explainer-articles/how-to-get-started-with-meta-muse/); [eesel](https://www.eesel.ai/blog/meta-muse-agent-pricing) and [mindwiredai](https://mindwiredai.com/2026/09/21/meta-muse-free-tier-limits/) (snippets)
- Paid tiers: Power at $20/mo (reportedly 500M tokens/week) and Maximum at $100/mo (reportedly 3B tokens/week). The paid allowances are not verified on a Meta page. — [Tech Insider](https://tech-insider.org/meta-muse-personal-ai-agent-launch-2026/) (snippet); [The Neuron](https://theneuron.ai/explainer-articles/how-to-get-started-with-meta-muse/) (says paid allowances couldn't be verified)
- The free tier reportedly includes the agent's own cloud computer, a browser it drives, connectors, and shopping or booking with approval. "Signup requires a payment card", even for free access (attributed to TechCrunch). — [The Neuron](https://theneuron.ai/explainer-articles/how-to-get-started-with-meta-muse/); [cellcog](https://cellcog.ai/blog/what-is-muse/) (snippet)
- Muse Spark 1.1 was announced 9 Jul 2026 as "a multimodal reasoning model built for agentic tasks" with a "1 million tokens" context window. It is available in "Thinking" mode in the Meta AI app. Meta reports major gains in tool use, computer use, coding and multi-agent orchestration. It names harnesses OpenCode, OpenClaw, Cline and Replit. — [ai.meta.com blog](https://ai.meta.com/blog/introducing-muse-spark-meta-model-api/) (fetched)
- Meta Model API (dev blog, 8 Jul 2026):
  - Base URL `api.meta.ai/v1`, model `muse-spark-1.1` (`meta/muse-spark-1.1` in OpenCode).
  - "self-serve and OpenAI SDK–compatible" (Chat Completions and Responses formats), plus the Anthropic Messages format.
  - "Pay-as-you-go pricing starts at $1.25 input/$4.25 output per million tokens"; reasoning tokens are billed as output.
  - "accessible in public preview … to developers in the US".
  - The nav also lists Muse Spark 1.2 and 1.3, and a "Muse Code" overview page. — [dev.meta.ai blog](https://dev.meta.ai/resources/blog/build-with-muse-spark/) (fetched)
- New developers reportedly get $20 in free credits, and keys come from dev.meta.ai. — [Yahoo Tech](https://tech.yahoo.com/ai/article/meta-opens-up-its-muse-spark-ai-model-to-developers-171724178.html) and [DataCamp](https://www.datacamp.com/blog/muse-spark-1-1) (snippets; not on the official page I read)
- Muse Code is described as a terminal coding agent that can run multiple sub-agents concurrently. — [search summary](https://www.developersdigest.tech/blog/meta-muse-spark-1-1-developer-guide-2026) (snippet). Its install, auth and pricing were not found on official pages.

### Inferences
- The free Muse agent is an *interactive consumer* worker: browser tasks, errands, shopping, connector-driven chores. An orchestrator can reach it only by a human (or UI automation) messaging it through the app or WhatsApp. That is fragile, and policy risk is out of scope here.
- The cheapest programmatic Meta path is the Meta Model API's starter credits (reportedly $20), then low per-token prices. That is API billing, not subscription quota.

### Gaps
- No official Meta page confirming the free-tier token cap, paid allowances or the card requirement was read. The figures rest on interviews and secondary coverage.
- No evidence of any API, webhook, MCP or CLI to submit tasks to the consumer Muse agent.
- Muse Code's auth model (Meta account vs API key) and whether it can use consumer Muse quota are unknown.
- Meta Model API rate limits were not found, nor any free tier beyond the starter credits. The Llama API's current status was not determined.

---

## 5. Other free tiers usable as cheap workers (brief)

### Takeaway
Several free API tiers are viable for low-stakes, high-volume tasks: OpenRouter `:free` models, Groq, Cerebras, the Mistral "Experiment" plan, and the Gemini API free key. All are rate-limited and change often. GitHub Copilot Free is IDE-oriented and has no meaningful agent quota. Numbers below come mostly from aggregators dated May–Oct 2026.

### Cited Findings
- **OpenRouter `:free` models:** 20 requests/minute; 50 requests/day until the account has bought $10 of credits lifetime, then 1,000/day. — [OpenRouter blog](https://openrouter.ai/blog/tutorials/free-llm-apis-compared/) (snippet)
- **Groq:** about 30 RPM and 14,400 requests/day on some models. The per-minute token ceiling is usually the binding limit, and the figures have changed since. — [costbench](https://costbench.com/best/best-llm-api-with-free-tier/) and [ianlpaterson](https://ianlpaterson.com/blog/free-llm-api-2026/) (snippets)
- **Cerebras:** about 1M tokens/day free, with an 8K context cap on the free tier. One source mentions a $5 credit after adding a card (conflicting). — [snippets](https://benchlm.ai/md/free-tier.md)
- **Mistral "Experiment" plan:** free, no card, about 1B tokens/month, around 1 request/second. Prompts may be used to improve models. — [snippets](https://github.com/mnfst/awesome-free-llm-apis/wiki)
- **GitHub Copilot Free:** 2,000 code completions/month and auto model selection only. GitHub replaced Premium Request Units with "GitHub AI Credits" on 1 Jun 2026. The Free plan now lists "limited chat and agent usage" without a number (previously 50 premium requests/month). The coding agent is a Pro-tier feature. — [costbench Copilot Free](https://costbench.com/software/ai-coding-assistants/github-copilot/free-plan/) and [agentdeals](https://agentdeals.dev/vendor/github-copilot) (snippets)
- **Gemini API free key:** see section 3 (250/day Flash-only in Gemini CLI; broader limits conflicting).
- **Vertex AI Express mode:** free for 90 days with no billing; quotas vary by account. — [geminicli.com](https://geminicli.com/docs/resources/quota-and-pricing/) (fetched)

### Inferences
- For an orchestration plane, these suit classification, summarisation, routing decisions and drafting. Throttle per-minute and keep a fallback chain, because one source warns none is "one you can plan a workload on".
- Copilot Free adds little beyond IDE completions.

### Gaps
- None of these numbers was verified on the providers' own limits pages in this pass. Recheck each provider's current page before hard-coding quotas.
