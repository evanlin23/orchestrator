# Hosting and Phone Access for a Personal AI-Agent Orchestrator (no home server, as of Nov 2026)

Scope: the compute/runtime layer and the phone-access layer for a supervisor process that drives the official subscription CLIs (Claude Code, OpenAI Codex CLI, Gemini CLI / its successor) logged into the user's own consumer subscriptions. Software design and ToS/policy questions are covered by other researchers. Research date: 2026-11-03. Prices are USD unless noted; EUR prices exclude VAT.

Important cross-cutting change found during this research: Google's own Gemini CLI docs say Gemini CLI "was replaced by Antigravity CLI on June 18th, 2026", and secondary sources say Gemini CLI stopped serving Google AI Pro / Ultra individual subscribers on that date. Anything below about "Gemini CLI" on a subscription login should be read as "Antigravity CLI (`agy`)". See the vendor-cloud and security sections. — [Gemini CLI auth docs](https://geminicli.com/docs/get-started/authentication/); [letsdatascience](https://letsdatascience.com/news/google-replaces-gemini-cli-with-antigravity-cli-574b2056); [inventivehq](https://inventivehq.com/blog/gemini-cli-deprecated-antigravity-cli-migration)

---

## 1. Cheap VPS: what a 2-4 vCPU / 4-8 GB always-on box costs in Nov 2026 (Hetzner, Oracle Always Free, DigitalOcean, Vultr, Linode, Lightsail, GCP e2-micro)

### Takeaway
A always-on VPS is still the simplest host that keeps CLI logins and long-running tmux sessions, but 2026 made it pricier. Hetzner raised prices twice (April and June 2026), and its cheap Cost-Optimized line showed as "currently unavailable" when checked on 2026-11-03. Oracle's free ARM tier was halved to 2 OCPU / 12 GB. A US-region 4 GB box from DigitalOcean, Lightsail or Linode costs about $24/mo. If you can still order one, Hetzner EU is about €9-11/mo for 4 vCPU / 8 GB. The GCP free e2-micro is too small for these CLIs.

### Cited Findings
**Hetzner Cloud (official pages, fetched 2026-11-03)**
- Cost-Optimized plans, prices "incl. IPv4", EUR excl. VAT: CX23 (2 vCPU x86, 4 GB, 40 GB NVMe) €5.99/mo (€0.0096/h); CAX11 (2 vCPU Ampere ARM64, 4 GB, 40 GB) €6.49/mo; CX33 (4 vCPU, 8 GB, 80 GB) €8.99/mo; CAX21 (4 vCPU ARM, 8 GB, 80 GB) €10.99/mo; CX43 (8 vCPU, 16 GB) €16.49; CAX31 (8 ARM, 16 GB) €21.49. 20 TB traffic included in EU locations. — [Hetzner Cost-Optimized](https://www.hetzner.com/cloud/cost-optimized/)
- On the same page every Cost-Optimized plan was marked "not available" / "currently unavailable. Please check back later". Servers are billed until deleted, even when powered off, and the monthly price is a cap. — [Hetzner Cost-Optimized](https://www.hetzner.com/cloud/cost-optimized/)
- The Hetzner cloud overview lists Cost-Optimized from €5.99, Regular Performance (shared) from €6.49 and General Purpose (dedicated vCPU) from €43.49. Locations: Falkenstein, Nuremberg, Helsinki (EU), Hillsboro OR and Ashburn VA (US), and Singapore. — [Hetzner Cloud](https://www.hetzner.com/cloud)
- 2026 price history (secondary sources): April 1, 2026 increase, then a June 15, 2026 adjustment for new orders and rescales. CX23 went €3.99 → €5.49 and CAX11 €4.49 → €5.99, both excluding IPv4. Existing instances reportedly keep their old price. — [wz-it](https://wz-it.com/en/blog/hetzner-price-increase-june-2026-cpx-ccx-alternatives/); [Vincent Schmalbach](https://www.vincentschmalbach.com/hetzner-cheap-cloud-unavailable-price-increases/); [Northflank](https://northflank.com/blog/hetzner-cloud-server-price-increases). These figures match the official "incl. IPv4" prices above once you add the roughly €0.50/mo IPv4 charge that [Bitdoze](https://www.bitdoze.com/md/es/hetzner-cloud-precios-2026.md) reports.
- Cost-Optimized plans reportedly showed a disabled "Create" button on several dates in September and October 2026. — [Bitdoze](https://www.bitdoze.com/md/es/hetzner-cloud-precios-2026.md); [agentdeals digest](https://agentdeals.dev/digest/2026-w25)
- US regions (Ashburn/Hillsboro): the CX line is reportedly not offered in the US. CPX11 (2 vCPU / 2 GB) reportedly went from $6.99 to $20.49/mo. Dedicated/AMD lines rose by up to +176% (e.g., CPX22 €7.99 → €19.49). — [Bitdoze](https://www.bitdoze.com/md/es/hetzner-cloud-precios-2026.md); [agentdeals digest](https://agentdeals.dev/digest/2026-w25); [wz-it](https://wz-it.com/en/blog/hetzner-price-increase-june-2026-cpx-ccx-alternatives/)

**Oracle Cloud Always Free (official docs, fetched 2026-11-03)**
- Ampere A1 Always Free is now 1,500 OCPU-hours and 9,000 GB-hours per month, which equals 2 OCPU and 12 GB of memory (one instance, or two 1-OCPU instances). It must run in the home region. You get 200 GB of block storage shared between boot and block volumes, and 10 TB/month outbound data. — [Oracle Always Free Resources](https://docs.oracle.com/en-us/iaas/Content/FreeTier/resourceref.htm)
- Idle reclamation: Oracle may reclaim an Always Free instance when, over 7 days, 95th-percentile CPU is below 20%, network is below 20%, and (A1 only) memory is below 20%. The doc gives no exemption for Pay-As-You-Go accounts. — [Oracle Always Free Resources](https://docs.oracle.com/en-us/iaas/Content/FreeTier/resourceref.htm)
- The cut from 4 OCPU / 24 GB to 2 / 12 was effective June 15, 2026 per InfoQ. Community reports describe A1 instances being "administratively disabled" after an August 18, 2026 enforcement. A trial account that ends while over 2/12 reportedly has its A1 instances disabled and deleted after 30 days. — [InfoQ](https://infoq.com/news/2026/07/oracle-cloud-free-tier-limits/); [Linuxiac](https://linuxiac.com/oracle-quietly-cuts-free-tier-ampere-a1-resources-in-half/); [terminalbytes](https://terminalbytes.com/oracle-cloud-free-tier-changes-2026); [issoh.co.jp](https://www.issoh.co.jp/tech/details/17531/). Sources conflict on the enforcement date, and on whether PAYG instances at 4/24 are grandfathered.

**DigitalOcean (official)**
- Basic Droplets: 2 vCPU / 4 GiB / 80 GiB / 4,000 GiB transfer at $24/mo; 4 vCPU / 8 GiB / 160 GiB at $48/mo; 1 vCPU / 2 GiB at $12/mo. Per-second billing (60 s minimum) starting January 1, 2026. Backups cost 20% (weekly) or 30% (daily) of the Droplet price. US regions: SFO, NYC, ATL, RIC, KC; EU regions: AMS, LON, FRA. — [DigitalOcean Droplet pricing](https://www.digitalocean.com/pricing/droplets)

**AWS Lightsail (official)**
- Linux with public IPv4: 2 GB / 2 vCPU / 60 GB at $12; 4 GB / 2 vCPU / 80 GB at $24; 8 GB / 2 vCPU / 160 GB at $44. IPv6-only bundles cost $10 / $20 / $40. — [Lightsail pricing](https://aws.amazon.com/lightsail/pricing/)

**Linode / Vultr (secondary only)**
- Linode 4 GB (2 vCPU, ~80 GB) is about $24/mo. Vultr's 4 GB is reported at $20/mo by some sources and $24/mo by others, so the sources conflict. Vultr's official pricing page returned HTTP 403 to my fetch. — [Deploynix](https://deploynix.io/blog/digitalocean-vs-vultr-vs-linode-for-laravel); [SSD Nodes](https://www.ssdnodes.com/blog/digitalocean-vs-linode-vs-vultr/)

**GCP free tier (official)**
- One non-preemptible e2-micro per month in us-west1, us-central1 or us-east1, with 30 GB-month standard PD and 1 GB/month egress from North America. New accounts get a $300 credit for 90 days. The page does not list e2-micro's vCPU/RAM. — [Google Cloud Free features](https://docs.cloud.google.com/free/docs/free-cloud-features)

**Workload requirement**
- Claude Code's stated hardware requirement is "4 GB+ RAM, x64 or ARM64 processor". Supported Linux: Ubuntu 20.04+, Debian 10+, Alpine 3.19+. — [Claude Code setup](https://code.claude.com/docs/en/setup)

### Inferences
- Rough monthly cost for the target box:

  | Host | 2 vCPU / 4 GB | 4 vCPU / 8 GB |
  |---|---|---|
  | Hetzner EU (if orderable) | about €6-6.50 | €9-11 |
  | DigitalOcean | $24 | $48 |
  | Lightsail | $24 | $44 (still only 2 vCPU) |
  | Linode | about $24 | n/a |
  | Vultr | $20-24 | n/a |
  | Oracle A1 | n/a | $0, but capped at 2 OCPU / 12 GB ARM with reclamation and capacity risk |

- Running Claude Code, Codex and the Google CLI side by side, with several concurrent agent sessions, makes 8 GB the comfortable target, given the 4 GB+ requirement for Claude Code alone. Oracle's 2 OCPU / 12 GB is enough RAM. Its 20% idle rule is an actual risk for an orchestrator that mostly waits on network, because CPU, network and memory must all stay under 20% for reclamation to trigger. A 12 GB box with agents resident may stay above the memory threshold, but that is unverified.
- A US user on Hetzner would realistically use an EU location, since the US now has only the expensive CPX line. Transatlantic latency of roughly 100 ms is fine with mosh. Without Hetzner, the cheapest US option at the target size is DO/Lightsail/Linode at about $24, or Vultr at about $20.
- GCP's e2-micro falls below Claude Code's 4 GB RAM requirement, so it is not viable as the orchestrator host. At most it could serve as a tiny relay (ntfy server, bot), and its 1 GB egress is small.

### Gaps
- No official per-plan Hetzner Regular Performance (CPX) EU price list, and no official US CPX prices. The "unavailable" status may be temporary or region-specific. Check the Hetzner Console.
- Vultr's official price was not retrievable (403). The Linode price comes from secondary sources only.
- No official Oracle statement on whether upgrading to PAYG exempts A1 from idle reclamation or the 2/12 cut. Current A1 "out of capacity" frequency in US regions was not measured.
- e2-micro specs (commonly 2 shared vCPU / 1 GB) are not confirmed on the fetched page.

---

## 2. Dev-environment-as-a-service: can Codespaces, Ona/Gitpod, Coder, Daytona, DevPod, Firebase Studio or Google Cloud Shell host the orchestrator?

### Takeaway
None of these can be an always-on host in 2026. Codespaces has a hard 12-hour lifetime and at most a 4-hour idle timeout, and it stops processes. Cloud Shell caps sessions at 12 hours and 50 hours/week. Firebase Studio is shutting down. Ona (ex-Gitpod) dropped its free plan. Codespaces (free hours, plus more with GitHub Pro via the Student Pack) and Cloud Shell are useful only as free on-demand interactive terminals whose home directory keeps CLI login files.

### Cited Findings
**GitHub Codespaces**
- Included per month: GitHub Free gets 120 "hours of compute time" and 15 GB-month storage; GitHub Pro gets 180 hours and 20 GB-month. Usage is weighted by a core multiplier (2 for a 2-core machine, 4 for 4-core). Overage costs $0.18/h (2-core), $0.36/h (4-core), $0.72/h (8-core), plus $0.07/GB-month storage. — [GitHub Codespaces billing](https://docs.github.com/en/billing/concepts/product-billing/github-codespaces)
- Idle timeout defaults to 30 minutes and can be set from 5 to 240 minutes. "Terminal activity, either input or output, also resets the idle timeout period." — [Codespaces timeout](https://docs.github.com/en/codespaces/setting-your-user-preferences/setting-your-timeout-period-for-github-codespaces)
- "A codespace has a maximum lifetime of 12 hours, regardless of your idle timeout policy or settings." Stopping a codespace stops running processes. Inactive codespaces are deleted after 30 days by default. Changes inside `/workspaces` survive a rebuild, but changes outside it are cleared on rebuild. — [Codespace lifecycle](https://docs.github.com/en/codespaces/about-codespaces/understanding-the-codespace-lifecycle). (A secondary search could not find this 12-hour cap in older docs, so it may be a recent change. The official lifecycle page fetched on 2026-11-03 states it.)

**Student offers**
- The GitHub Student Developer Pack includes GitHub Pro, which gives 180 Codespaces hours and $18 of Actions compute. Secondary sources list $200 DigitalOcean credit and about $100 Azure credit, but one 2026 source refers to a "DigitalOcean exit", so the DO credit is unconfirmed. GitHub paused new Copilot Student sign-ups in April 2026. — [Capital & Compute](https://capitalandcompute.net/blog/github-student-developer-pack-2026/); [perkstack](https://perkstack.co/blog/github-student-pack-guide); [buildmvpfast](https://www.buildmvpfast.com/blog/student-developer-perks-free-ai-tools-copilot-cursor-2026)

**Google Cloud Shell**
- 50 hours/week default quota and 5 GB persistent `$HOME` per user. Sessions are capped at 12 hours, and non-interactive sessions end automatically after 40 minutes. `$HOME` is deleted after 120 days of inactivity. Google points heavier users to Cloud Workstations. — [Cloud Shell quotas and limits](https://docs.cloud.google.com/shell/docs/quotas-limits); [Cloud Shell limitations](https://docs.cloud.google.com/shell/docs/limitations)

**Firebase Studio**
- Sunset announced March 19, 2026. New workspace creation and sign-ups were disabled June 22, 2026, and final shutdown is March 22, 2027. — [Firebase Studio release notes](https://firebase.google.com/support/release-notes/firebase-studio); [storyboard18](https://www.storyboard18.com/digital/google-to-shut-firebase-studio-in-2027-shifts-focus-to-ai-studio-and-antigravity-92922.htm)

**Ona (formerly Gitpod)**
- Free plan removed in 2026 (PricingSaaS change log: "Free plan removed; Core adds Codex subscription"). Core is reported from $20/month with Ona Compute Units (OCUs), and add-ons cost $10 per 40 OCUs. gitpod.io/pricing redirects to ona.com/pricing. — [PricingSaaS Q2 2026](https://pricingsaas.com/companies/gitpod/diffs/2026Q2); [agentdeals Ona](https://agentdeals.dev/vendor/ona). One stale aggregator still lists a free tier. — [StackPick](https://stackpick.net/tools/gitpod/)

### Inferences
- Codespaces on Free: 120 weighted hours is about 60 wall-clock hours/month on a 2-core machine. GitHub Pro (Student Pack) raises that to about 90 hours. That covers on-demand interactive sessions from a phone browser, but not a 24/7 supervisor, given the 12-hour lifetime and stopped processes. CLI login files in `~` survive stop/start but may be wiped on rebuild unless they live under `/workspaces` or a dotfiles/secret mechanism.
- Cloud Shell can serve as a free emergency terminal, and its persistent `$HOME` can hold logins. Its 12-hour cap and 50 h/week quota rule it out as a host.
- For this workload, these products lose to a $5-25/mo VPS or the persistent-sandbox products in section 3.

### Gaps
- Coder, Daytona and DevPod were not researched within the tool budget, so no 2026 pricing or persistence data was gathered. The commonly understood shape (Coder and DevPod are self-hosted or bring-your-own-cloud control planes and still need a VM; Daytona pivoted to agent sandboxes) is unverified here.
- Whether the GitHub Mobile app can open or start a Codespace was not verified. The browser at github.com/codespaces is the assumed route.
- Ona's official pricing page was not fetched directly.

---

## 3. Agent-oriented sandboxes and on-demand machines (Fly.io, Sprites, exe.dev, Modal, E2B, Cloudflare, Railway, Render): intermittent pricing and disk persistence

### Takeaway
Two products fit "persistent Linux box for coding agents that sleeps when idle". Sprites (Fly.io) is pay-per-use with a persistent 100 GB disk, wake-on-request and checkpoints. exe.dev is a flat $15/mo for a 2 vCPU / 4 GiB pool, with persistent VMs over SSH and HTTPS. Both keep CLI login state on disk. Fly Machines and Railway work but cost more for always-on. Modal, E2B and Cloudflare Sandbox are built for short-lived execution, with session caps or inactivity stops, and are poor homes for long-lived tmux and OAuth state.

### Cited Findings
**Sprites (fly.io/sprites; sprites.dev redirects there)**
- Sprites are "full Linux computers designed for agents, not sandboxes", with a persistent disk (100 GB volume, pay for used storage), checkpoints in about 1 s (automatic on idle and shutdown), a per-Sprite HTTPS URL, and auto-sleep (running → warm → cold). An incoming request to the URL wakes it with the filesystem intact. Restoring a checkpoint rewinds files but does not resume a paused process. — [Fly.io Sprites](https://fly.io/sprites)
- Pricing: $0.0385/CPU-hour and $0.021875/GB-hour while running. Hot storage costs about $0.50/GB-month and cold storage about $0.02/GB-month, with no compute billing while warm or cold. Fly's own examples: a 4-hour Claude Code session bursting to full CPU and 8 GB costs about $0.23; a web app with 30 wake-hours/month at 10% of 2 CPUs and 1 GB costs about $1.05/month. Access is via `sprite console`, `sprite exec`, HTTP API and SDKs. Harnesses like Claude and Codex "find everything where they left it." Free trial is $1 for new users and $30 for new orgs. The page does not mention SSH. — [Fly.io Sprites](https://fly.io/sprites)

**exe.dev**
- "Computers for developers and agents": persistent Linux VMs with root, `apt` and `systemd`, reachable at `ssh exe.dev` and over HTTPS. VMs are private by default and shareable. A built-in web coding agent ("Shelley") is included. Disk changes are streamed to other machines for regional durability (local NVMe). — [exe.dev](https://exe.dev); [exe.dev docs](https://exe.dev/docs)
- Plans: Personal $15/mo for a 2 vCPU / 4 GiB pool (expandable to 16 vCPU / 32 GiB), 100 GiB disk, up to 50 VMs per pool, and a 7-day trial "with a paid ChatGPT subscription". Work is $75/mo ($37.50 intro). The pricing page does not say whether VMs run continuously or sleep. — [exe.dev pricing](https://exe.dev/pricing)

**Fly.io Machines**
- shared-cpu-1x 1 GB costs $6.70/mo and shared-cpu-2x 2 GB $13.39/mo. Extra RAM is $6.00/GB-month. performance-1x 2 GB is $33/mo. Stopped Machines pay only $0.15/GB-month for rootfs, and volumes cost $0.15/GB-month provisioned. Dedicated IPv4 is $2/mo. There is no free tier: the trial is 2 machine-hours or 7 days. — [Fly.io pricing](https://docs.fly.io/about/pricing). A secondary source says Fly RAM pricing rose 20% in 2026. — [buildmvpfast](https://www.buildmvpfast.com/blog/fly-io-pricing-2026-ram-increase-vs-hetzner)

**Modal**
- The Starter plan includes $30/month in free credits. Sandboxes bill at 3x list: $0.00003942 per physical-core-second (1 physical core = 2 vCPU) and $0.00000667 per GiB-second. Volumes cost $0.09/GiB-month with 1 TiB/month free. The pricing page does not state a maximum sandbox lifetime. — [Modal pricing](https://modal.com/pricing)

**E2B**
- Hobby: $0 with a one-time $100 credit and a 1-hour max session. Pro: $150/mo with a 24-hour max session. Rates are $0.000014/vCPU-s and $0.0000045/GiB-s. — [E2B pricing](https://e2b.dev/pricing)

**Cloudflare Sandbox SDK / Containers**
- Sandbox SDK is billed as Containers (vCPU, memory, disk, egress), plus Workers requests, plus a Durable Object per sandbox. — [Cloudflare Sandbox pricing](https://developers.cloudflare.com/sandbox/platform/pricing/)
- A Cloudflare product page lists $0.000020/vCPU-s and $0.0000025/GiB-s, with free amounts of 25 GiB-hours and 375 vCPU-minutes. — [workers.cloudflare.com sandboxes](https://workers.cloudflare.com/product/sandboxes)
- A third-party summary says the instance stops at its inactivity timeout even with work running, and that snapshots (public beta) save only the writable rootfs, not processes. — [developersdigest](https://www.developersdigest.tech/agent-sandboxes/cloudflare-sandbox) (secondary, unverified)

**Railway**
- Plans: Free ($1/mo usage, 1 vCPU / 0.5 GB max), Hobby ($5/mo including $5 usage), Pro ($20/mo). Rates are about $20 per vCPU-month, about $10 per GB-RAM-month and about $0.15/GB-month for volumes. Stopped services cost nothing. Railway "Sandboxes" have idle timeouts (5 min on Free, 30 min on Hobby/Pro) that destroy them. — [Railway pricing](https://railway.com/pricing)

### Inferences
- **Monthly cost for an always-awake 2 vCPU / 4 GB box**, computed from the list rates. Real bills are lower if billing follows actual use rather than allocation, which the Sprites examples suggest.

  | Host | Always-awake, 2 vCPU / 4 GB |
  |---|---|
  | exe.dev | $15 flat |
  | Fly Machines | about $25 (shared-cpu-2x plus 2 GB extra) |
  | Railway | about $80 if fully used; an idle supervisor near 0.1 vCPU / 1 GB is about $12 |
  | Sprites | about $120 at full allocation, but cents-to-dollars if mostly asleep |
  | Modal | about $170 (sandbox rates) |

- **Fit:**
  - Sprites suits an on-demand pattern: wake the box from the phone, run a burst of agent work, let it sleep, and keep logins on disk.
  - exe.dev's flat $15 suits an always-on supervisor and is competitive with a US $24 VPS. Its persistence model is close to a normal VPS (SSH, systemd), so tmux, mosh-style use and CLI auth files should work. Whether VMs are ever suspended when idle is unverified.
- **Weak fits:** E2B's 1-hour or 24-hour caps and Cloudflare's inactivity stop make them unsuitable for long-lived tmux and OAuth state. They suit only fire-and-forget jobs, and they would need the credentials injected on every start.

### Gaps
- Render was not researched (no pricing fetched).
- Sprites: no SSH confirmation, and no statement on whether a running tmux or supervisor process survives warm/cold transitions. The docs imply processes stop on sleep, since restore "doesn't resume a paused process".
- exe.dev: always-on vs. idle-suspend behaviour, and whether the 7-day ChatGPT-linked trial needs Plus or Pro, are not stated.
- Cloudflare: official 2026 instance-type table and max sleep/keepalive settings were not retrieved.

---

## 4. GitHub Actions as a serverless orchestration runtime: limits, triggers, and running the CLIs with subscription tokens

### Takeaway
Actions works as an event-driven, on-demand runtime: 6 h per job, cron down to 5 minutes, and issue-comment/label/workflow_dispatch triggers. A private repo gets about 2,000 (Free) or 3,000 (Pro) 2-core Linux minutes per month, and public repos run free. Claude Code officially supports subscription auth in Actions via a `claude setup-token` OAuth secret. Codex officially recommends API keys for CI, and subscription use requires shuttling `auth.json`. Google's CLI action uses API keys or Vertex/WIF, and subscription OAuth is not a documented path. For subscription tokens, keep the repo private.

### Cited Findings
**Limits and pricing**
- Free accounts get $12 of included Actions compute per month and Pro gets $18. Standard runners in public repos are free and don't consume the allowance. Linux 2-core x64 is $0.006/min, 2-core arm64 $0.005/min and 1-core $0.002/min. Artifact storage is 500 MB (Free) or 1 GB (Pro). — [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)
- Max job execution time is 6 hours, and max workflow run time is 35 days including waits and approvals. Concurrency is 20 jobs on Free and 40 on Pro. `GITHUB_TOKEN` API limit is 1,000 requests/hour/repo. — [Actions limits](https://docs.github.com/en/actions/reference/limits)
- Schedule: shortest interval is every 5 minutes. Runs may be delayed or dropped under load, especially at the top of the hour. Schedules run on the default branch only. In public repos, scheduled workflows are disabled after 60 days without repository activity. — [Events that trigger workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows)
- `workflow_dispatch`: up to 25 inputs and a 65,535-character payload. It can be triggered from the "Run workflow" UI button, the REST API, or `gh workflow run -f ...`. Events triggered by `GITHUB_TOKEN` do not create new workflow runs, except `workflow_dispatch` and `repository_dispatch`. — [Events that trigger workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows)

**Claude Code in Actions**
- `CLAUDE_CODE_OAUTH_TOKEN` is "an OAuth token that authenticates with your Claude subscription, available on Pro, Max, Ultra, Team, and Enterprise plans". Generate it with `claude setup-token` and pass it as `claude_code_oauth_token`. "If you authenticate with an OAuth token, runs use your Claude subscription instead of API billing." — [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions)
- Interactive mode answers `@claude` in issue and PR comments. Automation mode with a `prompt` runs on any event, including `schedule`. The triggering user must have write access, and bot actors are rejected unless listed in `allowed_bots`. Docs suggest `--max-turns`, workflow timeouts and concurrency controls. — [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions)
- A third-party guide says the setup-token is valid for one year and shown once. A GitHub issue reports `claude-code-action@v1` failing with "Could not resolve authentication credentials" after a Pro → Max upgrade (single report, no confirmed fix). — [alexcloudstar](https://alexcloudstar.com/blog/claude-code-github-actions-2026/); [claude-code-action #1281](https://github.com/anthropics/claude-code-action/issues/1281)
- Claude also offers cloud scheduled tasks ("routines"), which run as cloud sessions on Anthropic infrastructure. This is an alternative to Actions cron for the Claude side. — [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web)

**Codex in Actions**
- OpenAI's auth docs say "API keys are still the recommended default for automation". Codex access tokens for non-interactive use are a ChatGPT Enterprise feature. A separate CI/CD guide covers keeping an updated `auth.json` between jobs, and the docs say not to run Codex "in untrusted or public environments". — [Codex authentication](https://learn.chatgpt.com/docs/auth.md)
- The official `openai/codex-action` appears to take an API key. A community fork (Chinchill-AI/codex-action) accepts a base64 `auth.json` to use a ChatGPT subscription. — [Chinchill-AI/codex-action](https://github.com/Chinchill-AI/codex-action)

**Gemini / Antigravity in Actions**
- `google-github-actions/run-gemini-cli` supports a Gemini API key, Workload Identity Federation (Vertex), or Gemini Code Assist (`GOOGLE_GENAI_USE_GCA`). No source documents a Google AI Pro personal-OAuth path in CI. — [run-gemini-cli docs mirror](https://docsearch.algolia.com/mcp/docs/repo/google-github-actions/run-gemini-cli); [classmethod](https://dev.classmethod.jp/en/articles/gemini-cli-github-actions-google-cloud)
- One source warns that scripts, cron jobs or GitHub workflows calling `gemini` on Google AI Pro/Ultra stopped working on June 18, 2026. — [inventivehq](https://inventivehq.com/blog/gemini-cli-deprecated-antigravity-cli-migration)

### Inferences
- **Event-driven orchestration:**
  - Trigger work by commenting on or labelling issues, or by dispatching a workflow.
  - Results come back as PRs, comments, or a ntfy push sent from the job.
  - The CLIs' subscription quotas, not Actions minutes, are the real limit. A private repo's roughly 2,000-3,000 free minutes covers about 33-50 hours of runner time per month.
- **Don't use public repos for subscription work.** Public repos give unlimited free minutes, but run logs and workflow files are public, and Codex's docs explicitly warn against public environments. Fork-PR runs also don't get secrets.
- **The Codex `auth.json` must be written back.** Codex auto-refreshes tokens, so a CI flow that reuses `auth.json` has to store the refreshed file after each job (e.g., as an updated secret or artifact), or the token goes stale. This is presumably what OpenAI's CI/CD auth guide covers.
- **No interactive approvals.** Actions jobs are stateless, with no persistent tmux and no human approve/deny mid-run except via PR review. This suits "run task to completion → open PR", not a long-lived supervisor.

### Gaps
- I did not verify that the GitHub Mobile app exposes the "Run workflow" (workflow_dispatch) button. The REST API, `gh`, or a mobile browser on github.com are the confirmed routes.
- OpenAI's CI/CD auth guide (refreshing `auth.json` between jobs) was not fetched.
- Whether Antigravity CLI supports any headless or CI subscription auth is undocumented in the sources found.
- The exact current lifetime of `claude setup-token` tokens comes from a third-party source only.

---

## 5. Using a vendor cloud session as the host (Claude Code on the web, Codex cloud)

### Takeaway
Claude Code cloud sessions are included with Max, at no extra VM charge, and run on 4 vCPU / 16 GB / 30 GB VMs that you can steer from the Claude mobile app. That makes them an excellent free host for Claude work. They are a poor host for the other CLIs or a long-lived supervisor: VMs pause when idle and can be reclaimed (background processes are lost), the default "Trusted" allowlist has no OpenAI domains, and there is no persistent home for OAuth files. Codex cloud strips secrets before the agent phase and caches containers for at most 12 hours. Nested CLIs are technically possible with Custom or Full network access, but they are ephemeral, and the ToS question belongs to the policy researchers.

### Cited Findings
**Claude Code cloud sessions**
- Available on Pro, Max, Ultra and Team. A session "keeps running after you close your laptop". It "counts toward your plan's usage limits… and there's no separate charge for the cloud VM". You can start or steer it from the browser, the Claude mobile app's Code tab, the Desktop app, `claude --cloud`, or cloud scheduled tasks. — [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web)
- Resource ceilings are about 4 vCPUs, 16 GB RAM and 30 GB disk. The setup script runs as root on Ubuntu 24.04 (`apt install` works). It is cached as a filesystem snapshot if it finishes in about 5 minutes, and rebuilt about every 7 days. The snapshot keeps files, not running processes. — [Cloud environments](https://code.claude.com/docs/en/cloud-environments)
- "After a few minutes without activity, a session's VM pauses with its files saved, and a paused VM can later be reclaimed." On reclaim, the conversation history is restored, but shell commands, subagents and session-scoped scheduled tasks are not. Bash commands default to a 2-minute timeout (10 max). Commands that time out move to the background for up to 30 more minutes, adjustable with `BASH_DEFAULT_TIMEOUT_MS`. — [Cloud environments](https://code.claude.com/docs/en/cloud-environments); [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web)
- Network levels are None, Trusted (default), Full (any domain) and Custom (your own list, optionally plus the defaults). Trusted includes api.anthropic.com, GitHub, npm, PyPI, Docker registries, `accounts.google.com`, `*.googleapis.com` and cloud.google.com. It includes no openai.com or chatgpt.com entries. "Network secrets" are injected by a proxy into requests to matching hosts and never enter the VM. GitHub credentials also stay outside the VM. — [Cloud environments](https://code.claude.com/docs/en/cloud-environments)
- `claude --teleport` pulls a cloud session into a local terminal. `claude -p "msg" --cloud <session-id>` queues follow-ups from any logged-in CLI, which makes it scriptable from a supervisor. — [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web)

**Codex cloud (now documented as "Codex Cloud (Legacy) environments")**
- Default `universal` image. The setup script runs with internet access. A maintenance script runs on cached-container resume, and containers are cached up to 12 hours. Agent-phase internet is off by default, with limited or unrestricted options. Secrets are available only to setup scripts and removed before the agent phase, while environment variables persist for the whole task. Extra tools can be installed through the setup script. — [Codex cloud environment](https://learn.chatgpt.com/docs/environments/cloud-environment)

### Inferences
- **Claude work:** Claude Code cloud sessions (plus Remote Control for self-hosted sessions) are effectively a free, phone-controllable compute layer for the Claude part of the orchestration. Two Max 20x accounts means two separate session pools.
- **Codex CLI inside a Claude cloud session:**
  - Requires Custom or Full network access to reach OpenAI auth and API hosts.
  - Requires putting `auth.json` into the VM. Network-secret header injection doesn't fit Codex's refreshable OAuth file, and storing it as a plaintext environment variable would be a security regression.
  - Refreshed tokens would be lost when the VM is reclaimed, so this is fragile.
- **Google's CLI inside a Claude cloud session:** the Google domains are already in Trusted, but Antigravity reportedly stores credentials in the OS keyring (see section 8), which a headless cloud VM may lack.
- **Codex cloud as a host for other CLIs:** removing secrets before the agent phase means another vendor's CLI could authenticate only via environment variables (visible for the whole task). This is a weak fit.
- **Not a supervisor host:** neither vendor cloud suits a 24/7 supervisor. They suit dispatching discrete tasks to the cloud from a supervisor that lives elsewhere.

### Gaps
- No vendor docs explicitly address installing a competitor's CLI inside their cloud session. Feasibility here is inferred from network and secret mechanics. ToS implications are out of scope (policy researcher).
- Codex cloud's agent-phase domain allowlist presets and per-task time limits were not on the fetched page.

---

## 6. An old Android phone or tablet as a zero-cost home server via Termux

### Takeaway
It's possible but fragile. Claude Code has no official Android support, and plain Termux has had no working build since about v2.1.113. Community reports show the official Linux ARM64 installer working inside `proot-distro` Ubuntu on Termux. Codex has only community Termux ports. No source confirms Antigravity CLI on Android. Treat this as a hobby option, not a reliable orchestrator host.

### Cited Findings
- Claude Code's official supported OSes are macOS, Windows, Ubuntu, Debian and Alpine. Hardware: 4 GB+ RAM, x64 or ARM64. The npm package supports `linux-arm64` and `linux-arm64-musl`, but no Android target. — [Claude Code setup](https://code.claude.com/docs/en/setup)
- Community reports:
  - Plain Termux runs on Android's Bionic libc and has had no Android build since v2.1.113 (tracking issue anthropics/claude-code #50270). `claude` installs but fails because the native binary is missing.
  - `pkg install proot-distro && proot-distro install ubuntu && proot-distro login ubuntu`, then the official `curl -fsSL https://claude.ai/install.sh | bash`, works with auto-updates.
  - Reported working on a Pixel 10 Pro and a Galaxy S26 Ultra.
  — [jonathansblog](https://jonathansblog.co.uk/?p=30143); [ferrumclaudepilgrim/claude-code-android](https://github.com/ferrumclaudepilgrim/claude-code-android); [Rathodpd99/claude-code-android](https://github.com/Rathodpd99/claude-code-android)
- A rooted-device tool (root-aicli) pins bare-Termux Claude Code to 2.1.112 and needs Magisk, KernelSU or APatch. — [cosyra](https://cosyra.com/guides/claude-code-on-android.html); [root-aicli](https://github.com/Crypt0Shmipt0/root-aicli/releases)
- Codex CLI has no official Android build. Community options are `@aryanxispe/codex-termux` (built from upstream rust-v0.121.0, may lag) and `codex-cli-voice-android` (alpha cross-compile). — [codex-termux](https://github.com/aryanispe/codex-termux); [codex-cli-voice-android](https://github.com/camdoherty/codex-cli-voice-android); [cosyra](https://cosyra.com/guides/codex-cli-on-android.html)

### Inferences
- An old phone needs at least 4 GB RAM to meet Claude Code's stated requirement, and three CLIs in proot add overhead. Most "old" phones (3-4 GB) will be marginal.
- **Reliability risks:**
  - Android's background-process killing, battery optimisation and thermal throttling.
  - proot's syscall-translation overhead.
  - Community ports lagging upstream (Codex pinned to an older release).
- The cost is zero, but a $5-15/mo VPS or exe.dev box is far more dependable. A phone works best as a client (section 7), or at most a Tailscale-reachable fallback host.

### Gaps
- No source on Antigravity CLI (or legacy Gemini CLI) running on Termux in 2026.
- No measured data on keeping Termux and proot sessions alive for days (wake locks, Android 14/15 phantom-process killer). The general knowledge suggests it is a known problem, but it was not sourced here.

---

## 7. Recommended phone access stack: Tailscale, mosh + tmux, mobile SSH clients, web terminals, push notifications and approve/deny flows

### Takeaway
The stack used in practice:
- Tailscale (free Personal plan) for a private network with no open ports.
- OpenSSH plus mosh plus tmux on the host.
- Termius (iOS and Android) or Blink (iOS only) on the phone.
- For Claude specifically, `claude remote-control` server mode on the VPS, which uses outbound-only HTTPS and gives native phone steering, push notifications and permission approvals in the Claude app.
- ntfy-based hooks for push alerts and approve/deny buttons for the other CLIs.

### Cited Findings
**Network: Tailscale**
- Free Personal plan ("free forever"): up to 6 users, unlimited user devices, 50 tagged resources, 3 ACL groups, and "nearly all of Tailscale's offerings". Basic Tailscale SSH is limited to 5 hosts. Non-commercial use only. Paid plans are Standard ($8/user/mo) and Premium ($18). — [Tailscale pricing](https://tailscale.com/pricing)

**Session layer: mosh + tmux**
- The "canonical free stack" is a Linux box, Tailscale, OpenSSH plus mosh, and tmux. Mosh does not work with Tailscale SSH (Tailscale's own SSH server), so keep OpenSSH running for mosh-server. The guide also recommends tmux mouse mode for phone use. — [OpenReplay: remote box agentic coding setup](https://blog.openreplay.com/remote-box-agentic-coding-setup/)
- A practitioner runs six Claude Code agents from an iPhone using Termius, mosh, Tailscale, a Vultr VM and tmux, with phone-friendly tmux keybindings. — [granda.org via scour.ing](https://scour.ing/p/https://granda.org/en/2026/01/02/claude-code-on-the-go)

**Claude Remote Control**
- `claude remote-control` (server mode) or `claude --remote-control` / `/remote-control` (interactive) connects claude.ai/code or the Claude iOS/Android app to a session running on your machine. "Your local Claude Code session makes outbound HTTPS requests only and never opens inbound ports." — [Remote Control](https://code.claude.com/docs/en/remote-control)
- Server mode options:
  - `--spawn same-dir|worktree|session`
  - `--capacity N` (default 32 concurrent sessions)
  - `--permission-mode`
  - A QR code to connect from the phone.
  — [Remote Control](https://code.claude.com/docs/en/remote-control)
- Requires Pro, Max, Ultra, Team or Enterprise, and API keys are not supported. On a remote machine, "start it inside tmux or screen". Server mode exits after about 10 minutes of network outage, so it needs a supervisor or systemd restart. Permission prompts and `AskUserQuestion` stay open until answered. The transcript is stored on Anthropic servers while connected. — [Remote Control](https://code.claude.com/docs/en/remote-control)
- Mobile push: enable "Push when actions required" (permission prompts and questions) and/or "Push when Claude decides" in `/config`. — [Remote Control](https://code.claude.com/docs/en/remote-control)
- Optional "Require trusted devices" on Pro/Max ties Remote Control access to a verified device and a recent authentication. — [Remote Control](https://code.claude.com/docs/en/remote-control)

**Push and approval for other CLIs (ntfy)**
- `claude-remote-approver` sends each Claude Code permission prompt to the phone via ntfy with Approve / Always Approve / Deny buttons, using a PermissionRequest hook and a reply topic. — [claude-remote-approver](https://github.com/yuuichieguchi/claude-remote-approver)
- `claude-push` uses a PermissionRequest hook with Allow / Deny and a 90 s timeout that falls back to the terminal prompt. — [claude-push](https://github.com/coa00/claude-push)
- `claude-pager` uses the Notification hook and sends keystrokes back into tmux, outbound connections only. — [claude-pager](https://github.com/FilipCondac/claude-pager)
- Pushary is a paid ($9.99/mo) two-way approval service. — [Pushary](https://pushary.com/fr/claude-code-notifications)
- Anyone who knows a public ntfy topic name can read or post to it. — [claude-remote-approver](https://github.com/yuuichieguchi/claude-remote-approver) and the search summary of these projects; use a self-hosted server or authenticated topics for approvals.

**Mobile SSH clients (secondary, partly vendor-authored)**
- Blink Shell is iOS/iPadOS-only and mosh-native, about $19.99/yr with a 14-day trial. Termius is cross-platform (including Android) and listed as supporting mosh. Termius Pro pricing is reported inconsistently ($10/mo billed annually, $9.99/mo or $19.99/mo). — [termai.sh](https://termai.sh/blog/blink-shell-vs-termius); [getmoshi](https://getmoshi.app/articles/blink-vs-termius). These comparison pages come from competing app makers.

### Inferences
**Recommended phone stack (inference):**
1. VPS (or exe.dev) joined to Tailscale. Bind SSH to the tailnet only, or firewall the public SSH port; keep OpenSSH for mosh.
2. tmux as the persistent layer, with one window per agent and one for the supervisor, all under systemd or tmux-resurrect.
3. Clients: Termius on Android, or Blink or Termius on iOS, using mosh over Tailscale.
4. For Claude: run `claude remote-control --spawn worktree` inside tmux under systemd, so you steer, approve and get push notifications natively in the Claude app with no SSH typing.
5. For Codex and the Google CLI, plus supervisor events: send ntfy pushes (authenticated or self-hosted topics) from hooks or the supervisor. Approve/deny buttons need a reply channel (an ntfy reply topic or a small Telegram bot).
- **Option to drop SSH from the phone:** Remote Control's outbound-only design means the VPS needs no inbound port at all for the Claude side. SSH over Tailscale is only needed for the other CLIs and maintenance.

### Gaps
- Web terminals (ttyd, Wetty) were not researched. A web terminal behind Tailscale Serve, or authenticated, would avoid an SSH app, but no 2026 sources were gathered.
- Pushover and Telegram-bot approval flows were not separately sourced. No 2026 Pushover price was found.
- JuiceSSH's 2026 maintenance status, and Termius's free-tier mosh support, were not verified.
- Whether Tailscale Funnel and Serve are on the free Personal plan was not shown on the pricing page.

---

## 8. Security: protecting long-lived OAuth tokens on a VPS, firewalling, not exposing ports

### Takeaway
The highest-value secrets on the box are three subscription credentials:
- Claude's long-lived `setup-token` / OAuth credentials.
- Codex's plaintext `~/.codex/auth.json`, which auto-refreshes.
- The Google CLI's cached OAuth.

Keep the box reachable only over Tailscale, prefer outbound-only control paths (Remote Control, ntfy), restrict file permissions or use a keyring, and never put these tokens in public repos or Actions logs.

### Cited Findings
**Codex credentials**
- Codex caches login in a plaintext `~/.codex/auth.json` or the OS credential store, set by `cli_auth_credentials_store` = `file` | `keyring` | `auto` | `ephemeral`. "For sign in with ChatGPT sessions, Codex refreshes tokens automatically during use." Treat `auth.json` like a password: don't commit it, paste it into tickets, or share it in chat. — [Codex authentication](https://learn.chatgpt.com/docs/auth.md)
- Headless login options:
  - `codex login --device-auth` (beta; must be enabled in ChatGPT security settings).
  - Copy `auth.json` from a machine with a browser.
  - SSH-forward port 1455.
  — [Codex authentication](https://learn.chatgpt.com/docs/auth.md)

**Claude credentials**
- `claude setup-token` creates a long-lived subscription token. Store it as a secret and never commit it. An OAuth token is tied to the subscription of the person who generated it. — [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions)
- Remote Control is outbound HTTPS only (no inbound ports). Trusted Devices can require a verified device. — [Remote Control](https://code.claude.com/docs/en/remote-control)
- Claude cloud sessions keep GitHub credentials and network secrets outside the VM via a proxy. — [Cloud environments](https://code.claude.com/docs/en/cloud-environments)

**Google credentials**
- Gemini CLI caches OAuth tokens at `~/.gemini/oauth_creds.json` (macOS may use Keychain). — [agent-safehouse](https://www.agent-safehouse.dev/docs/agent-investigations/gemini-cli)
- Antigravity CLI reportedly stores its credential in the system keyring, and its migration onboarding can move Gemini CLI session tokens into the OS keyring. — [Antigravity CLI migration](https://antigravity.google/docs/cli/gcli-migration); [agentpedia](https://agentpedia.codes/blog/gemini-cli-to-antigravity-cli-migration)
- Official Gemini CLI docs say Google sign-in "requires a web browser on a machine that can communicate with the terminal". Headless mode reuses cached credentials. For CI, use a Vertex service-account key or API key. — [Gemini CLI auth](https://geminicli.com/docs/get-started/authentication/)

**Network exposure**
- Mosh needs OpenSSH (not Tailscale SSH). Tailscale provides private networking so ports needn't be public. — [OpenReplay](https://blog.openreplay.com/remote-box-agentic-coding-setup/)
- Public ntfy topics are readable and writable by anyone who knows the name. — [claude-remote-approver](https://github.com/yuuichieguchi/claude-remote-approver)
- In public repos, secrets are withheld from fork-PR runs. — [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions)
- Codex says never run in untrusted or public environments. — [Codex authentication](https://learn.chatgpt.com/docs/auth.md)

### Inferences
**Practical hardening for the VPS (inference):**
1. Expose no public inbound ports. Use the provider firewall to deny all inbound except, optionally, the Tailscale UDP port, and run SSH and mosh only over the tailnet.
2. Use a dedicated non-root user per subscription account, so that 2 Claude Max, ChatGPT Plus and 2 Google accounts don't share `~/.claude`, `~/.codex` and `~/.gemini`. Files should be 0600, on full-disk-encrypted storage if the provider offers it.
3. Enable `cli_auth_credentials_store=keyring` where a headless keyring (e.g., gnome-keyring/libsecret) is practical. Otherwise accept 0600 files and keep backups encrypted.
4. Turn on Remote Control Trusted Devices.
5. Use authenticated or self-hosted ntfy topics for any approval channel.
6. Know the recovery path: rotate credentials (log out, re-login, regenerate setup-token) if the box is compromised.
- **Copying `auth.json` from a phone:** the phone has no desktop browser session to copy from, so the realistic logins are Codex `--device-auth` and Claude's copy-paste login URL. For the Google CLI on a headless VPS, the keyring requirement and browser-callback flow may be the hardest part. An SSH-forwarded callback port from a phone client is awkward, so do the first login once from any laptop or browser-capable device and copy the cached credentials.

### Gaps
- No official Antigravity CLI documentation was found on headless/SSH sign-in, keyring fallback on servers without a desktop keyring, or Linux ARM64 support.
- Not researched: whether Google AI Plus (as opposed to Pro/Ultra) includes any CLI or agent access.
- Not researched: provider-specific firewall features (Hetzner Cloud Firewall, Oracle security lists, DO Cloud Firewall) and their costs.
- Not documented officially: Claude `setup-token` revocation UI and exact lifetime.

---

## 9. Overall comparison and fit for this workload

### Takeaway
The best value is a hybrid:
- **Supervisor:** one small always-on Linux box. Options are Hetzner EU 4 vCPU / 8 GB at about €9-11 if orderable, exe.dev at $15/mo, a US $20-24 4 GB VPS, or Oracle's free 2 OCPU / 12 GB ARM if you accept reclamation risk. It holds the CLI logins in tmux and runs `claude remote-control`.
- **Overflow:** Claude cloud sessions (free with Max) for Claude work, and GitHub Actions (private repo) for event-driven jobs.
- **Phone access:** Tailscale, mosh, tmux, Termius/Blink, the Claude app, and ntfy.

### Cited Findings
- See sections 1-8 for the per-option sources. Key anchors:
  - Hetzner prices and availability: [Hetzner Cost-Optimized](https://www.hetzner.com/cloud/cost-optimized/)
  - Oracle 2/12 and idle rule: [Oracle Always Free](https://docs.oracle.com/en-us/iaas/Content/FreeTier/resourceref.htm)
  - DO $24/$48: [DigitalOcean](https://www.digitalocean.com/pricing/droplets)
  - Lightsail $24/$44: [Lightsail](https://aws.amazon.com/lightsail/pricing/)
  - exe.dev $15: [exe.dev pricing](https://exe.dev/pricing)
  - Sprites per-use: [Fly.io Sprites](https://fly.io/sprites)
  - Codespaces 12 h cap: [Codespace lifecycle](https://docs.github.com/en/codespaces/about-codespaces/understanding-the-codespace-lifecycle)
  - Actions 6 h and $12/$18: [Actions limits](https://docs.github.com/en/actions/reference/limits), [Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)
  - Claude cloud VM 4 vCPU / 16 GB, no extra charge: [Cloud environments](https://code.claude.com/docs/en/cloud-environments), [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web)
  - Remote Control outbound-only: [Remote Control](https://code.claude.com/docs/en/remote-control)

### Inferences
Comparison matrix (costs from the cited pages; "persistence" means keeping CLI logins on disk and keeping tmux processes running):

| Option | Approx. cost for this use | Keeps logins on disk | Keeps tmux / long processes | Phone setup ease | Network / security | Fit |
|---|---|---|---|---|---|---|
| Hetzner EU CX33/CAX21 (4c/8 GB) | €8.99-10.99/mo (if orderable) | Yes | Yes, 24/7 | Medium: SSH bootstrap, then Tailscale | Firewall + Tailscale, no open ports | Excellent, if available |
| DO / Lightsail / Linode 4 GB (US) | about $24/mo; Vultr about $20 | Yes | Yes | Medium | Same | Good, pricier |
| Oracle A1 Always Free | $0 (2 OCPU / 12 GB ARM) | Yes | Yes, but reclaimable if idle | Hard (signup and capacity) | Same | Good if obtainable; plan for reclamation |
| exe.dev Personal | $15/mo flat (2 vCPU / 4 GiB pool, 100 GiB) | Yes (persistent VMs) | Likely; idle policy unverified | Easy (`ssh exe.dev`, HTTPS, web agent) | Private by default | Very good |
| Sprites (Fly) | Cents to a few dollars/mo if mostly asleep; about $120/mo if pinned awake at 2c/4 GB | Yes (persistent 100 GB disk, checkpoints) | No across sleep (processes stop) | Medium (CLI/API, HTTPS URL) | Private or public URL | Best on-demand option |
| Fly Machines | about $25/mo for 2c/4 GB always-on; $0.15/GB-month when stopped | With volume | Yes while running | Medium | Fly proxy | OK |
| Railway Hobby | $5 + usage (about $12+ for an idle supervisor) | With volume | Yes | Medium | — | OK |
| Modal / E2B / Cloudflare Sandbox | Pay per second; E2B 1 h (free) or 24 h (Pro) cap | Volumes only | No (short-lived) | Hard | — | Poor for a supervisor; OK for burst jobs |
| GitHub Codespaces | Free 120 weighted h (about 60 h on 2-core); Pro 180 | Mostly (`/workspaces`) | No (12 h cap, idle stop ≤ 4 h) | Easy (browser) | GitHub-managed | On-demand only |
| Google Cloud Shell | Free (50 h/week) | 5 GB `$HOME` | No (12 h cap) | Easy (browser) | Google-managed | Emergency only |
| GitHub Actions (private repo) | Free $12/$18 compute (about 2,000-3,000 min) | No (stateless; secrets per run) | No (6 h/job) | Easy (comment, label, dispatch) | Secrets; avoid public repos | Good for event-driven tasks |
| Claude Code cloud sessions | Included with Max (no VM charge) | No for other CLIs | No (pause / reclaim) | Excellent (Claude app) | Proxy-held secrets, allowlist | Excellent for Claude-only tasks |
| Codex cloud | Included with ChatGPT plan (not verified here) | No | No (12 h cache) | Good (ChatGPT app) | Secrets setup-only | Codex-only tasks |
| Old Android + Termux | $0 | Yes | Fragile | Medium | Tailscale | Hobby / fallback only |
| Firebase Studio / Ona | Shutting down / no free plan | — | — | — | — | Not viable |

- **Suggested build order for this user:**
  1. Try Oracle A1 (free) or Hetzner CAX21/CX33. Otherwise use exe.dev ($15) or a $20-24 US VPS.
  2. Install Tailscale, OpenSSH, mosh and tmux.
  3. Log in each CLI once, using device-code or copy-paste flows, under separate Unix users per account.
  4. Run `claude remote-control` under systemd inside tmux, for each Max account.
  5. Add ntfy hooks for the Codex and Google CLIs.
  6. Use Claude cloud sessions and GitHub Actions as overflow capacity. Use Sprites if you want an on-demand second box.
- **Budget range:** $0 (Oracle + free tiers) to about $15-25/mo (exe.dev or a US VPS), plus optional pennies for Sprites.

### Gaps
- Codex cloud task limits and inclusion in ChatGPT Plus were not verified in this pass. Another researcher may cover them.
- Render, Coder, Daytona, DevPod, ttyd/Wetty and Pushover/Telegram were not researched within the tool budget.
- Real-world RAM use of Claude Code, Codex and Antigravity running concurrently was not measured or sourced. The 8 GB recommendation is an inference from Claude Code's 4 GB+ requirement.
