# ADR-005: Runtime Integration Strategy — How DealClaw Reaches Gmail, Gemini, HubSpot and the Notification Channel

> **Scope**: Decision cluster covering where the live integration logic for
> the v2 (real-estate) pipeline lives and who executes it at runtime: the
> OpenClaw runtime interpreting `agents/*/SOUL.md` and `skills/*.md`, Python
> client code ported from v1 (`~/openclaw-deal-scout`), or a hybrid in which
> OpenClaw orchestrates and v1's clients run as tenant-aware MCP tools.
> Includes three coupled open questions (notification channel for the first
> live run, OpenClaw MCP capability, Gemini quota plan) because each changes
> which option is viable. Excludes the remote-dashboard authentication design
> (G4) and the runtime Checker design, which depend on this decision.

- **Status:** Proposed (DRAFT, decision pending owner)
- **Date:** 2026-10-13
- **Feature:** none (cross-cutting; blocks the first live end-to-end run for F008)
- **Context:** The 2026-10-13 audit (PHR `history/prompts/general/0037-external-audit-verification.general.prompt.md`)
  verified that v2's 106 passing tests exercise `tests/pipeline_sim.py`, a
  re-implementation of the agent rules (`tests/pipeline_sim.py:1-18`; ADR-002
  records why), and that v2 contains no Gmail, Gemini, HubSpot, WhatsApp or
  Discord client code. v2 has never been deployed into the installed OpenClaw
  runtime (`~/.openclaw/agents/` holds only `main`; no `tenants/` directory in
  `~/.openclaw/workspace`). The only real client code for these services is in
  v1, which is single-tenant (0 case-insensitive matches for "tenant" in v1
  `src/`, grep) and whose live runs are recorded only in agent-written PHRs
  (self-reported, unverified). Before the first live lead can run, the owner
  must decide what executes the integration calls at runtime.

<!-- Significance checklist (ALL must be true to justify this ADR)
     1) Impact: Long-term consequence for architecture/platform/security?  YES — defines the runtime, the test strategy and the credential surface.
     2) Alternatives: Multiple viable options considered with tradeoffs?   YES — A, B, C below.
     3) Scope: Cross-cutting concern (not an isolated detail)?             YES — touches every agent, every external service, tests and the constitution. -->

## Decision

**Not made. G1 is the owner's decision.** This draft lays out three options,
the open questions that gate them, and a recommendation (see
"Recommendation"), which the owner may accept, amend or reject.

Evidence labels used below:
- **verified**: a command was run or the exact line was read on 2026-10-13.
- **self-reported, unverified**: stated in an agent-written PHR; no raw log,
  trace or screenshot exists in either repo (find, 2026-10-13).
- **unverified**: not checked; must be resolved before relying on it.

### Constraints any option must satisfy (verified, `CONSTITUTION.md`)

- Hard Rule 7 (`CONSTITUTION.md:161-163`): OpenClaw remains the sole runtime
  orchestrator.
- Hard Rule 8 (`CONSTITUTION.md:164-165`): PK-mode notifications use WhatsApp
  only; UK-mode use Discord only; neither substitutes the other.
- Section 5 (`CONSTITUTION.md:131-134`): zero-cost; no paid APIs.
- Tool priority rule (`CONSTITUTION.md:145-147`): existing ClawHub skill
  before custom code; OpenClaw built-in channel before an external API.
- Hard Rule 3 (`CONSTITUTION.md:153-154`): pipeline halts at
  `gemini_today_count` ≥ 18.
- Phase 1 gate (`CONSTITUTION.md:50-52`): 3 PK agencies confirmed before
  UK-market work begins.

### v1 client interfaces available for reuse (verified, `~/openclaw-deal-scout`, branch `main`)

| Service | Interface | Notes |
|---|---|---|
| Gmail read | `build_service(credentials_path)` `src/gmail_intake/gmail_client.py:17`; `poll_inbox(service, since_ts, max_messages)` `:54` | Read-only scope `:14`. |
| Gmail send | `src/email_scheduler/auth.py:1` (`gmail.send` scope) | Separate OAuth use; feature 007 has an operator-approval queue (`src/email_scheduler/queue_store.py`). |
| Gemini | `classify(request, api_key)` `src/gmail_intake/classifier.py:121` | Model `gemini-2.5-flash` `:18`; retries 10s/30s/60s `:19`; prompt is a business-deal classifier parameterised by `{target_segment}` `:39`, not the v2 PK real-estate rubric. |
| HubSpot | `HubSpotClient(token)` `src/crm_logger/client.py:19,26`; `search_contact` `:90`, `upsert_contact` `:118`, `create_deal` `:152` | Rate guard `:16`. Deal mapping in `src/crm_logger/log_deal.py:78`. |
| Discord | `notify_deal(deal, adapter, state_path)` `src/discord_notifier/notifier.py:17`; `NotifierContract` protocol `src/discord_notifier/adapter.py:11`; `DiscordAdapter(webhook_url)` `:18`; `NoopAdapter` `:71`; `get_adapter` `:77` | Webhook, i.e. an external API, not OpenClaw's built-in channel. No WhatsApp adapter exists. |
| Orchestration | `run_cycle(config, cycle_logger)` `src/pipeline_orchestrator/runner.py:122` | Python orchestrator. |
| MCP surface | `check_new_deals` `src/gmail_intake/server.py:256`; CRM tool `src/crm_logger/server.py:17`; six gateway tools `src/openclaw_gateway/server.py:17-50` (`get_gateway_status`, `get_health`, `run_cycle`, `get_pipeline_cycles`, `get_deals`, `get_quota_usage`) | FastMCP (`pyproject.toml` dependencies). |
| Scheduling | `deploy/openclaw.service:9` runs `python3.12 -m openclaw_gateway`; `deploy/openclaw.timer:7` every 15 min; in-process scheduler `src/openclaw_gateway/__main__.py:20-21` | v1's production path was Python under systemd, not OpenClaw agents calling MCP tools. |

Size: 4,971 lines of Python in v1 `src/`; 83 commits on `main` (`git rev-list --count main`).

### v2 agent rules that define the required behaviour (verified)

- Intake classifies with Gemini 2.5 Flash: `agents/intake/SOUL.md:5`, `:26`; quota bookkeeping `:72`.
- Delivery validates, writes HubSpot, notifies: `agents/delivery/SOUL.md:12-25`, WhatsApp `:32`, Discord `:37`.
- Orchestrator routing by market: `agents/orchestrator/SOUL.md:17-18`; HubSpot pre-flight `:51`.
- Tooling assumptions: Gmail via `agentmail-integration` skill (`workspace/TOOLS.md:10`, **not installed**; installed plugin skills are `browser-automation`, `canvas`, `discord`); HubSpot and Gemini via env vars (`workspace/TOOLS.md:32-33`); WhatsApp and Discord via OpenClaw channel config (`workspace/TOOLS.md:35-36`).
- Per-tenant config: `workspace/tenants/_template/USER.md:5-6` (WhatsApp number, Discord channel), `:9-10` (env-var names for HubSpot and Gemini keys).

---

### Option A — OpenClaw Markdown agents only

**What exists today**
- v2 agent and skill Markdown (above). Installed runtime OpenClaw 2026.6.11
  with a running gateway process (`ps`, `package.json`, 2026-10-13), only the
  default `main` agent registered.
- No code path in the repo states *how* an agent performs the HubSpot HTTP
  calls or the Gemini call (env var names only, `workspace/TOOLS.md:32-33`).

**What must be built**
1. Install and verify a Gmail skill (`agentmail-integration` or equivalent).
2. Establish how agents call HubSpot and Gemini: an existing ClawHub skill, a
   built-in HTTP tool, or custom skills (unverified which exist for 2026.6.11).
3. Deploy Orchestrator/Intake/Delivery and one tenant into the runtime.
4. Configure the WhatsApp channel (PK) per Hard Rule 8.
5. A trace format; the runtime's own logging is unexamined.

**Cost**: zero infrastructure. Unknown LLM cost for OpenClaw's own reasoning;
if the runtime's driving model is Gemini on the same free key, orchestration
steps would also consume the 20/day budget (**unverified**; the model
configuration was not read).

**Risk**
- No live evidence at all for this path.
- Thresholds (0.7/0.9, halt at 18, dedup) are enforced by an LLM reading prose; the sim tests (ADR-002) cannot detect runtime drift.
- Hardest option to trace deterministically.

**Live-behaviour evidence**: none (verified absence: no v2 logs, traces,
state files or tenant directory).

---

### Option B — Port v1's Python clients into v2 (Python runs the pipeline)

> **Amendment 1 (2026-10-13):** Option B is withdrawn as a primary option. It
> is kept only as a fallback if Option C fails its MCP probe. See Amendment 1
> at the end of this ADR.

**What exists today**: all v1 interfaces in the table above, plus v1's
orchestrator, scheduler, state stores and tests (test files exist, e.g.
`tests/unit/test_classifier.py`, `tests/integration/test_check_new_deals.py`;
pass/fail **unverified**, not run).

**What must be built**
1. Tenant-awareness throughout (v1 has none): per-tenant credentials resolved from `USER.md` env-var names, per-tenant state, per-tenant quota.
2. A PK real-estate classifier prompt matching v2's rubric (v1 prompt is a generic deal classifier, `classifier.py:39`).
3. A WhatsApp adapter behind `NotifierContract` (`adapter.py:11`), or OpenClaw's built-in channel.
4. Map v1 state to the v2 `MEMORY.md` spine and `dashboard-state.json` contract.
5. Repoint v2 tests from `pipeline_sim.py` to real functions.
6. **A constitution amendment**: Python `run_cycle` under systemd as the runtime orchestrator violates Hard Rule 7 (`CONSTITUTION.md:161-163`) and gate I5.

**Cost**: largest code effort of the three; zero infrastructure (systemd timer). Adds v1's dependency set to v2 (`pyproject.toml`: fastmcp, google-api-python-client, google-auth-oauthlib, google-genai, portalocker, requests).

**Risk**
- Requires amending a non-negotiable rule.
- Makes the "OpenClaw orchestrator" positioning inaccurate (`README.md:51`).
- Porting single-tenant code invites cross-tenant leakage (Hard Rule 5).

**Live-behaviour evidence**: v1 PHRs, all **self-reported, unverified**:
`gmail-intake/017`, `018` (2026-07-15), `020`, `021` (2026-07-29);
`discord-notification/028` (2026-07-18, synthetic state-store entry);
`004-pipeline-orchestration/0002` (2026-07-23, all four services);
`general/014` (2026-07-28, a skill-install check, not a pipeline run). No raw
logs, traces or screenshots exist in v1 (find; `data/` holds only `.gitkeep`).

---

### Option C — Hybrid: OpenClaw orchestrates; v1 clients run as tenant-aware MCP tools

**What exists today**
- v1 already wraps its clients as FastMCP tools (`src/gmail_intake/server.py:256`, `src/crm_logger/server.py:17`, `src/openclaw_gateway/server.py:17-50`).
- v2 agents already separate Maker (Intake) from Checker (Delivery).
- Whether the OpenClaw runtime has ever called these MCP tools: **unverified**. v1's production path ran Python's own scheduler (`deploy/openclaw.service:9`), so v1 is not evidence of OpenClaw→MCP calls.

**What must be built**
1. Confirm OpenClaw 2026.6.11 can register and call MCP tools (open question b). Hard prerequisite.
2. Add `tenant_id` to every tool; resolve credentials and state per tenant from `USER.md`; reject unknown tenants (Hard Rule 5).
3. Move deterministic rules into tool code: score thresholds, the halt at 18, dedup and CRM-before-notify. SOUL.md then calls tools instead of describing HTTP behaviour.
4. Notifications through OpenClaw's built-in channel (tool priority rule, `CONSTITUTION.md:145-147`), not v1's webhook adapter.
5. Repoint v2 tests from `pipeline_sim.py` to the real tool functions.
6. An ID-only trace emitted by the tools.
7. Resolve the port collision: v1's gateway default (`clawhub/deal-scout/SKILL.md:32`) equals v2's dashboard port (`dashboard/server.py:24`).

**Cost**: medium. Zero infrastructure. Two runtimes to operate: OpenClaw (Node) plus a Python MCP process.

**Risk**
- Entirely dependent on open question b.
- Two processes to supervise.
- Tool-call errors cross a process boundary and must surface in the trace.

Upside: deterministic code becomes an independent check on the LLM's
decisions, the maker-checker separation the runtime currently lacks (audit
claim C8).

**Live-behaviour evidence**: same v1 PHRs as Option B for the clients
(**self-reported, unverified**); **none** for the OpenClaw↔MCP link.

---

### Open questions (owner decides; each option below lists its constitutional effect)

**(a) First live run's notification channel.** PK mode requires WhatsApp
(`agents/orchestrator/SOUL.md:17`; Hard Rule 8), and Discord is UK-only.
- a1. UK-mode live run through Discord. Conflicts with the Phase 1 gate
  (`CONSTITUTION.md:50-52`, no UK work before 3 PK agencies).
- a2. PK-mode run with a documented, test-tenant-only Discord deviation.
  Conflicts with Hard Rule 8 and gate I3 (`CONSTITUTION.md:184-185`); needs a
  dated, scoped exception recorded before the run.
- a3. Hold the live run until F013 (WhatsApp) is specced. Constitution-compliant; delays the first live evidence.

**(b) Can OpenClaw 2026.6.11 call MCP tools?** Unverified. Three different
versions are named: README `2026.7.1` (`README.md:51`), ADR-001 `2026.6.33`
(`adrs/ADR-001-secrets-hardening.md:7`), installed `2026.6.11` (global
`package.json`).
- b1. Read the installed runtime's own docs and `--help` offline (no network,
  no agents started). This needs owner approval, because the global npm
  directory is outside the audit's read authority.
- b2. Upgrade OpenClaw first, then check. Changes the runtime under test; needs its own decision.
- b3. Treat as unknown; decide G1 between A and B only.

**(c) Gemini plan (G3, undecided).** Two limits are in play: the v2 daily
cap (halt at 18, `CONSTITUTION.md:153-154`), and a per-minute burst limit
recorded in v1 PHR `004-pipeline-orchestration/0002` lines 34, 36, 50, 54
(429 bursts across a 22-message batch, about 6.5 minutes per cycle;
self-reported, unverified).
- c1. One key per tenant. The template already supports a per-tenant env-var
  name (`workspace/tenants/_template/USER.md:10`). Whether Google's limits
  apply per key or per project: **unverified**.
- c2. Paid tier. Conflicts with Section 5 and gate I2 (`CONSTITUTION.md:131-134`, `:183`).
- c3. Deterministic pre-filter before Gemini (for example sender/format rules
  in `skills/zameen-parser.md`). Constitution-compliant. Reduces both daily
  and per-minute pressure; can be combined with c1.

## Recommendation (for owner review; not a decision)

**Option C, conditional on (b) being answered "yes" by b1. If the answer is
"no", choose between A and B with that evidence in hand.** For (a), a3
unless the owner records a scoped exception for a2. For (c), c3, optionally
with c1.

Reasons:
1. **Constitution fit.** C keeps OpenClaw as sole orchestrator (Hard Rule 7). B requires amending a non-negotiable rule.
2. **Evidence reuse.** C and B reuse the only client code with any live-run record, even if self-reported. A starts from zero evidence and a missing Gmail skill.
3. **Deterministic rules.** Thresholds, the quota halt and dedup become code. The existing 106 tests can then target real functions instead of a parallel simulation (ADR-002's stated limitation).
4. **An independent check.** Tools that refuse invalid actions give the runtime the independent checker that audit claim C8 found missing.
5. **Smallest decisive next step.** b1 is offline, read-only and cheap, and it settles whether C is viable before any build.

## Consequences

### Positive

- (C) Deterministic enforcement of non-negotiables; real-function tests; traceable tool calls; OpenClaw positioning preserved.
- (Any) Ends reliance on `pipeline_sim.py` as the only evidence of behaviour.

### Negative

- (C) Two runtimes; dependence on an unverified OpenClaw capability; tenant-aware refactor of v1 clients; port collision to resolve.
- (B) Constitution amendment; largest port effort.
- (A) No evidence; LLM-enforced thresholds; possible hidden Gemini consumption by the runtime.
- (Any) The first live run stays blocked until credentials are rotated (owner) and (a) is resolved.

## Alternatives Considered

- **A, B, C** as above.
- **Run v1 unchanged for a first trace.** Considered and not listed as an
  option. v1 is single-tenant, uses a generic deal classifier and a Discord
  webhook, so a v1 run would not show v2 working.

## References

- Feature Spec: `specs/001-pk-lead-intake-notify/spec.md` (F008)
- Implementation Plan: none yet (pending G1)
- Related ADRs: ADR-001 (secrets hardening, open items), ADR-002 (test simulation pattern), ADR-003 (dashboard server architecture)
- Evaluator Evidence: `history/prompts/general/0037-external-audit-verification.general.prompt.md`
- Related conflict noted for the owner: G4 (remote dashboard off) vs gate I4 (`CONSTITUTION.md:186`, dashboard client-accessible).

---

## Amendment 1 — 2026-10-13: open question (b) answered; constitution conflicts; route chosen for (a)

Status of this ADR is unchanged: **Proposed (DRAFT, decision pending owner)**
for G1. This amendment records the (b) check, the constitution conflicts, and
the owner's choice on open question (a).

### (b) result: documented yes; not yet proven on this machine

Read-only, offline check of the installed package
(`~/.npm-global/lib/node_modules/openclaw/`, `package.json:3` = `2026.6.11`):

- MCP SDK bundled: `package.json:1965` (`@modelcontextprotocol/sdk` 1.29.0).
- OpenClaw acts as an MCP client registry; saved servers are consumed by
  embedded OpenClaw runtimes: `docs/cli/mcp.md:11-19`, `:376`.
- Embedded OpenClaw exposes configured MCP tools in the `coding` and
  `messaging` tool profiles; `minimal` hides them: `docs/cli/mcp.md:394`.
- Config shape `mcp.servers`, stdio or remote:
  `docs/gateway/configuration-reference.md:89-94`, `:137-138`.
- Tools appear under plugin id `bundle-mcp`; sandboxed sessions need an extra
  allowlist entry: `docs/gateway/config-tools.md:46-77`.
- Stdio fix shipped in this release: `CHANGELOG.md:71` (PR #95102), inside
  the 2026.6.11 section (`CHANGELOG.md:5`).
- Code present, not just docs: 53 files under `dist/` reference `bundle-mcp` (grep).

**Not yet proven:**
- The active tool profile and sandbox mode, which live in config (not opened).
- Whether DealClaw's Intake/Delivery agents receive the tools.
- A live connection to v1's FastMCP server via `openclaw mcp probe`. Deferred
  by the owner until credential rotation is confirmed.

Option C is therefore **viable, pending a probe**.

Constraints for Option C, from the docs:
- Launch v1 from its own venv Python. `PYTHONPATH` in a stdio server's `env`
  is rejected (`docs/cli/mcp.md:680`).
- Pass secrets as env references, not literal values (`docs/cli/mcp.md`, `doctor` checks).
- The tool profile must not be `minimal`.

### Constitution conflicts (verified)

- **Option B** conflicts with Hard Rule 7 / gate I5 (`CONSTITUTION.md:161-163`).
  **Option B is withdrawn as a primary option and kept only as a fallback if
  Option C fails its probe.**
- **(a1)** UK-mode live run conflicts with the Phase 1 gate (`CONSTITUTION.md:50-52`).
- **(a2)** PK-mode run on Discord conflicts with Hard Rule 8 / gate I3
  (`CONSTITUTION.md:164-165`, `:184-185`).
- **(c2)** paid Gemini tier conflicts with Section 5 / gate I2
  (`CONSTITUTION.md:131-134`, `:183`).
- **G4** (remote dashboard off) conflicts with gate I4 (`CONSTITUTION.md:186`).
  Noted only; not decided here.

### Open question (a): owner decision — Route 1 chosen

**Route 1 — Spec F013 first, no constitution change (CHOSEN by owner, 2026-10-13).**
1. `/sp.specify` F013, PK WhatsApp notification through OpenClaw's built-in
   WhatsApp channel. The spec must score **≥ 9.6/10** before `/sp.plan`
   (`CONSTITUTION.md:73`, `:81`; `agents/spec-scorer/spec-scorer.md:5`, `:10`).
2. Build the Option C tools: tenant-aware, with an ID-only trace.
3. First live run: PK mode, test tenant, WhatsApp, synthetic lead.

- Pros: fully compliant; the first live evidence is the real product path.
- Cons: live evidence waits for F013's spec, plan and build.

Findings relevant to the F013 spec (read-only check of
`docs/channels/whatsapp.md` in the installed package):
- The channel is WhatsApp Web via Baileys, not an official business API (`:8`, `:155-158`).
- Login is a QR scan by a phone (`:63-70`, `:113-117`).
- The WhatsApp runtime is an external plugin, `@openclaw/whatsapp`, not in
  the core package (`:10-20`). It is not installed on this machine (only a
  Discord plugin project exists under `~/.openclaw/npm/projects/`, names only).
- A dedicated number is recommended (`:109`, `:122-142`).
- Multiple accounts are supported (`:591-605`); outbound sends need an
  active listener for the sending account (`:168`, `:703-708`).
- The docs contain no ban or terms-of-service warning (grep for
  ban/suspend/terms/unofficial: no hits). Ban risk for unofficial WhatsApp
  Web clients is therefore **unverified** and must be assessed in the F013
  spec.

**Route 2 — Owner amends the constitution for one labelled test run
(CONSIDERED, NOT CHOSEN).** The owner would add a dated, scoped exception to
`CONSTITUTION.md` permitting one PK-mode live run on Discord with these limits:
- test tenant only;
- labelled `TEST-RUN-DISCORD-EXCEPTION`;
- synthetic lead only;
- not counted toward the Phase 1 gate;
- auto-expiring.

Not chosen because it sets a precedent for exceptions to a non-negotiable
rule, and it proves Discord rather than WhatsApp, so F013 would still need
its own live run.

### Prerequisites before any live run (unchanged)

- Credential rotation confirmed by the owner.
- Owner approval for `openclaw mcp probe` against v1's server.
- G1 decided.
- F013 specced (≥ 9.6/10), planned and built (Route 1).

### Recommendation (unchanged)

Option C, with Option B as the fallback only if C fails its probe. G1 remains
the owner's decision.
