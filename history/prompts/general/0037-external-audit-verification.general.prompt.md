---
id: 0037
title: External audit verification
stage: general
date: 2026-10-13
surface: agent
model: claude-opus-5-5
feature: none
branch: master
user: asmaiqbal01
command: free-form audit prompt (no /sp command), followed by two approval prompts
labels: ["audit", "verification", "claim-set", "correction-note", "feature-ids", "secrets", "live-evidence", "g1-pending"]
links:
  spec: null
  ticket: null
  adr: adrs/ADR-005-runtime-integration-strategy.md (Proposed, DRAFT)
  pr: null
files:
 - history/prompts/general/0037-external-audit-verification.general.prompt.md (this file, new)
 - adrs/ADR-005-runtime-integration-strategy.md (new, DRAFT)
 - (outside repo, not tracked) ~/dealclaw-private/rotation-checklist.md
tests:
 - uv run --with pytest --with jsonschema pytest -q  (run 1: 106 passed in 1.08s, exit 0)
 - uv run --with pytest --with jsonschema pytest -q  (run 2: 106 passed in 0.88s, exit 0)
---

## Prompt

### Prompt 1 (audit request, verbatim)

Outcome: Verify the findings of an external audit of DealClaw against the
actual repo and machine, then tell me what to work on next and in what order.
This is a read-only analysis. No implementation.

Context: An outside review (read-only access to the GitHub repos) found:
1. 106 tests pass, but pipeline tests run against tests/pipeline_sim.py, a
   re-implementation of the rules in agents/*/SOUL.md and skills/*.md. The
   repo has no Gmail, Gemini, HubSpot or Discord client code. Live behaviour
   is unproven in the repo.
2. README says F013 is "in build". My notes say "specced, not built". No
   specs/006 folder was found.
3. Feature IDs conflict: README uses F001-F005 for specs 001-005, but
   history/prompts label them F008-F011. The README row "F008-F012: 15 tests"
   may double-count.
4. All 152 tasks in specs/*/tasks.md are ticked [X]. That is self-reported.
5. pipeline_sim.py sets Gemini daily limit 20, halt at 18. Possible scaling
   blocker for pilots.
6. dashboard/server.py binds 127.0.0.1 and no auth was found, but
   skills/remote-dashboard.md describes remote access.
7. ADR-001: Discord token regeneration and .openclaw gitignore are unchecked.
   The Gmail OAuth client secret from the v1 repo (DS1-OPEN-001) may still
   need rotating.
8. No runtime Checker exists. constitution-checker and spec-scorer are
   dev-time only.
9. The landing page (dealclaw repo) shows "$0 / Month", a "Watch Pipeline
   Demo" CTA and "social proof". I have no signed pilots and no revenue.

Do NOT trust these. Confirm or refute each one yourself, with evidence.

Constraints:
- Read-only. Do not edit, create, delete or commit files. No git writes,
  no deploys, no network calls to HubSpot, Discord, Gemini or Gmail.
- Never print secrets. Report only that a secret exists or is exposed,
  never its value. Lead content never appears in output, only IDs.
- Do not mark anything verified unless you ran something or read the exact
  line. Otherwise write UNVERIFIED or UNKNOWN.
- Local machine is allowed: check ~/openclaw-deal-scout (v1, branch
  001-gmail-intake), ~/.openclaw config presence and permissions (not
  contents), and whether any OpenClaw process or logs exist.

Authority: You may read any file under ~/openclaw-deal-scout-realestate,
~/openclaw-deal-scout and ~/AsmaIqbal01, run pytest, and run
git log / status / diff / grep. You may not install packages or start servers
without asking me first.

Deliverable (print in the terminal, do not write files):
A. Finding-by-finding verdict for items 1-9: CONFIRMED, REFUTED or UNVERIFIED,
   with file:line or command output as evidence.
B. Where the real Gmail/Gemini/HubSpot/Discord logic lives (which repo, which
   file, or "not found"), and the state of branch 001-gmail-intake
   (T001-T033 done vs not).
C. Any run logs, traces or screenshots that prove a live end-to-end run.
   If none exist, say so plainly.
D. A canonical feature table: spec folder -> F-ID -> test count -> status.
E. A prioritized next-work list (P0-P3). Each item: what, why, files,
   dependency, acceptance criteria, how to verify.
F. The single best next task, with a plan for me to approve. Do not start it.
   Candidate to evaluate first: one live end-to-end lead with a trace
   (no lead text in logs), before the Checker step.
G. Decisions I must make, as a short list.

Verification: Re-run pytest and report the exact result. For every claim in
your report, state the source. Mark anything you could not confirm as
"unverified".

Also: name the Agent Factory concepts (via the Zia Tutor connector) you are
using now and those used earlier in this project (4Ds loop, Delegation Brief,
Delegation loop, Verify). Show your plan and wait for my approval before any
consequential action.

### Prompt 2 (first approval, verbatim)

APPROVED
(a) Run pytest via: uv run --with pytest --with jsonschema pytest -q
    Report the exact result. Do not install anything into the repo.
(b) Write the PHR for the audit. No secrets, no lead content, IDs only.
    Record the audit as a claim set with the verdicts and the evidence you cited.

NOT APPROVED YET
(c) The live end-to-end run. It stays blocked until credentials are
    rotated (by me, in the provider portals) and decision G1 is made.
    Do not install skills, start OpenClaw agents, start any server or
    tunnel, or make network calls to Gmail, Gemini, HubSpot or Discord.

MY DECISIONS
G2: F-IDs are F008-F012, as in the specs and CONSTITUTION.md.
G3: Gemini plan is undecided. Do not change HEARTBEAT.md or the sim.
G4: Remote dashboard is off. Do not start cloudflared or server.py until
    auth is specced.
G5: Correct records going forward only. Do not rewrite past PHRs or tasks.md.
    Add a dated correction note instead.
G1: I have NOT decided. Prepare the decision, do not make it.

TASKS (in this order, then stop and wait for me)
1. Run pytest as above.
2. Write the audit PHR (task b).
3. Draft ADR options for G1 (OpenClaw markdown agents vs porting v1's
   Python clients into v2 vs a hybrid), as a draft only. For each option:
   what exists today (cite file and line from ~/openclaw-deal-scout/src and
   the v2 agents), what must be built, cost, risk, and what evidence of
   live behaviour supports it. Use the v1 clients' real interfaces
   (gmail_client, classifier, crm_logger/client, discord_notifier,
   pipeline_orchestrator/runner). Mark anything unverified. Give a
   recommendation with reasons, but I decide. Use /sp.adr for the format.
4. Write a credential-rotation checklist for me (steps, where, how to
   confirm the old one fails). It must not read or print config contents.
   List only file names, and the provider portal steps.

CONSTRAINTS
- Write only the PHR, the ADR draft and the checklist. No other file edits.
  No git commits or pushes. Ask me before the first write.
- Never print secrets. Do not open openclaw.json or its backups.
- Mark every claim with its source. If you cannot confirm it, write "unverified".

AGENT FACTORY: name the concept you use now (via Zia Tutor) and the ones
used earlier in this project. Show your plan, then wait for my approval.

### Prompt 3 (second approval, verbatim)

Approved, with changes. Do tasks 2-4 now, then stop and wait.

CHANGES TO YOUR PLAN
1. Credential checklist: write it OUTSIDE the repo, in
   ~/dealclaw-private/rotation-checklist.md. Not in adrs/. Do not call it
   ADR-001-anything. File names and portal steps only. Never open
   openclaw.json or its backups.
2. PHR claim C6: write only "remote dashboard lacks authentication;
   remote access disabled (G4)". No endpoint details or tenant-listing
   behaviour in any file inside the repo.
3. No git add, commit or push. I will review and commit.
4. ADR-005: Status stays "Proposed (DRAFT, decision pending owner)".
   Keep the pytest/__pycache__ folders.

EXTRA EVIDENCE TO USE IN ADR-005 (verify each yourself before citing)
- v1 (~/openclaw-deal-scout, main) has about 5,000 lines in src/ with 61
  commits. It has zero mentions of "tenant" in src/ (single-tenant).
- v1 has 7 live-run PHRs under history/prompts: gmail-intake/017, 018,
  020, 021; discord-notification/028; 004-pipeline-orchestration/0002;
  general/014. They are agent-written narratives. No raw logs, traces or
  screenshots exist in the repo. Label them "self-reported, unverified".
  Do not copy HubSpot deal IDs into any new file.
- PHR 0002 records Gemini 429 bursts on a 22-message batch (about 6.5
  minutes per cycle). Treat as evidence of a per-minute limit, separate
  from v2's daily cap of 18.
- v1's tasks.md shows 9 done and 24 open although the code exists. v1's
  README says "Production" for all 7 features. Note both as unreconciled.
- v1 feature branches (001-gmail-intake, 002-hubspot-crm-logger) are
  named in PHR frontmatter but do not exist on the remote now.

ADR-005 MUST INCLUDE
- Options A (OpenClaw markdown only), B (port v1 Python clients into v2),
  C (hybrid: OpenClaw orchestrates, v1 clients run as MCP tools made
  tenant-aware). For each: what exists (file:line), what must be built,
  cost, risk, live-behaviour evidence.
- Open questions, as options for me to decide:
  a) PK=WhatsApp and Discord=UK-only conflict: UK-mode live run through
     Discord, PK-mode with a documented Discord deviation, or hold the
     live run for the F013 spec.
  b) Whether OpenClaw 2026.6.11 can call MCP tools (unverified; the README,
     ADR-001 and the install name three different versions).
  c) Gemini plan (G3, undecided): key per tenant, paid tier, or pre-filter.
- Your recommendation with reasons. I decide G1.

PHR
- Same plan as before (ID 0037, claims C1-C9, G2-G5 recorded, G1 pending,
  dated correction note 2026-10-13). Add: pytest result 106 passed
  (verified by two separate runs) and the v1 evidence above.

AFTER WRITING
- Check for leftover {{placeholders}}, grep the new files for anything
  shaped like a token, and run git status. Expect 2 new untracked files
  in the repo (PHR, ADR-005) and 1 file outside it. Then stop.

AGENT FACTORY
Name the concept you use now (via Zia Tutor) and the ones used earlier.

## Response snapshot

Read-only audit of the v2 repo (this repo), the v1 repo (`~/openclaw-deal-scout`),
and local machine state (`~/.openclaw` presence and permissions only; config
files were never opened). The audit is recorded below as a claim set: each
claim carries a verdict and the evidence it rests on. "Unverified" means no
command was run and no exact line was read that settles it.

### Test result (verified)

`uv run --with pytest --with jsonschema pytest -q` was run twice in this
session: **106 passed** both times (1.08s and 0.88s, exit 0). Ephemeral uv
environment; no `.venv`, `pyproject.toml` or lock file was created in the repo.
Gitignored `.pytest_cache/` and `__pycache__/` folders were left in place on
owner instruction. These tests exercise `tests/pipeline_sim.py`, not live
services (see C1).

### Claim set C1–C9

| ID | Claim (external audit) | Verdict | Evidence |
|---|---|---|---|
| C1 | Pipeline tests run against a re-implementation; no Gmail/Gemini/HubSpot/Discord client code; live behaviour unproven | CONFIRMED | `tests/pipeline_sim.py:1-18` states it is "NOT production agent logic" and re-implements thresholds; 14 test files import it (grep). Only non-test Python: `dashboard/server.py`, `discovery/create_form.py`, `discovery/create_form_uk.py` (Google Forms scope only, `discovery/create_form.py:26`). Grep for Gmail/genai/HubSpot/Discord client imports: no hits. |
| C2 | README says F013 "in build"; no specs/006 | CONFIRMED | `README.md:40`, `README.md:55`. `ls specs` shows 001–005 only; `git log --all` has no F013 commit; only branch is `master`. |
| C3 | Feature-ID conflict; README "F008–F012: 15 tests" may double-count | CONFIRMED (ID conflict) / REFUTED (double-count of total) | Specs declare F008–F012: `specs/001-pk-lead-intake-notify/spec.md:8`, `specs/002-pk-client-dashboard/spec.md:9`, `specs/003-pk-email-approval-gate/spec.md:9`, `specs/004-pk-dashboard-email-queue/spec.md:9`, `specs/005-pk-pilot-tracking/spec.md:9`. Total of 106 is correct (static count = 106; pytest = 106). Per-spec counts by creating commit (no test file modified afterwards, per `git log --stat -- tests`): 001=28, 002=17, 003=29, 004=11, 005=21. README lists 002=12 and 003=19; the "F008–F012: 15" row equals the 5+10 shortfall, not separate tests. |
| C4 | All 152 tasks ticked; self-reported | CONFIRMED | grep of `- [x]` / `- [ ]` per `specs/*/tasks.md`: 37+26+32+27+30 = 152 ticked, 0 open. Test-producing tasks have their test files; behavioural tasks cannot be confirmed because nothing ran live (C1). |
| C5 | Gemini daily limit 20, halt at 18; scaling risk | CONFIRMED | `tests/pipeline_sim.py:27-29`, `workspace/HEARTBEAT.md:29-30`, `CONSTITUTION.md:196`. Quota is tracked "per tenant" (`workspace/MEMORY.md:9`) but the tenant template points every tenant at the same env var (`workspace/tenants/_template/USER.md:10`), so the cap is effectively shared (inference from those two lines). Google's actual free-tier limits: unverified (no network). |
| C6 | Dashboard / remote access | CONFIRMED | remote dashboard lacks authentication; remote access disabled (G4) |
| C7 | ADR-001 action items unchecked; Gmail OAuth secret (DS1-OPEN-001) may need rotating | CONFIRMED (unchecked) / UNVERIFIED (rotation) | `adrs/ADR-001-secrets-hardening.md:18-20` unchecked. `~/.openclaw` is `drwx------`; config files are mode 600 (`stat`, `find -printf`). Six config-file copies exist by name (`openclaw.json` plus five backups); contents not opened. ADR-001:7-8 states the config holds plaintext secrets (claim from ADR, not re-verified). The string `DS1-OPEN-001` is not present in v2, v1 or `~/AsmaIqbal01` (grep). v1 has no credential/token files in its WSL working tree and none were ever added to its git history (`git log --all --diff-filter=A`). Whether any credential was rotated: unknown. |
| C8 | No runtime Checker exists | PARTLY REFUTED | A runtime Checker is designed: `agents/delivery/SOUL.md:1-10`. It is not independent (the same agent writes CRM and notifies, `agents/delivery/SOUL.md:21-37`) and not deployed (`~/.openclaw/agents/` contains only `main`). constitution-checker and spec-scorer are dev-time (`agents/constitution-checker/constitution-checker.md:9-11`). |
| C9 | Landing page shows "$0 / Month", demo CTA, social proof; no pilots or revenue | UNVERIFIED (landing page) / CONFIRMED (no pilots) | dealclaw repo is not on this machine; no web fetch was authorized. `PILOTS.md`: 4 slots, all `"status": "not_started"`, all `"active": false` (grep). |

### Additional findings (verified unless marked)

- Live-run evidence for v2: **none**. No logs, traces, screenshots,
  `dashboard-state.json` or `approval-queue.json` in the repo (find).
  `~/.openclaw/workspace` has no `tenants/` directory; its files are dated
  2026-07-07 (before v2's first commit on 2026-08-01) and differ in size from
  v2's `workspace/` files, so v2 has not been deployed into the runtime
  (inference from names, sizes and dates; contents not read).
- An OpenClaw gateway process was running during the audit (`ps`). Installed
  version 2026.6.11 (`package.json` of the global install). README says
  2026.7.1 (`README.md:51`); ADR-001 says 2026.6.33 (`adrs/ADR-001-secrets-hardening.md:7`).
- `workspace/TOOLS.md:10` expects an `agentmail-integration` skill for Gmail;
  installed plugin skills are `browser-automation`, `canvas`, `discord` (ls).
- Channel mismatch: PK mode notifies via WhatsApp and Discord is UK-only
  (`agents/orchestrator/SOUL.md:17-18`, `workspace/TOOLS.md:35-36`); README
  calls Discord "current" (`README.md:55`).

### v1 evidence (`~/openclaw-deal-scout`, branch `main`)

- Size: 4,971 lines of Python in `src/` (5,683 lines across all `src/`
  files). Commit count: **83** on `main` (`git rev-list --count main`); the
  "61 commits" figure supplied with the prompt is refuted.
- Single-tenant: 0 matches for "tenant" in `src/` (case-insensitive grep).
- Real clients: `src/gmail_intake/gmail_client.py:17,54`,
  `src/gmail_intake/classifier.py:121`, `src/crm_logger/client.py:19-152`,
  `src/discord_notifier/notifier.py:17`, `src/discord_notifier/adapter.py:11-77`,
  `src/pipeline_orchestrator/runner.py:122`.
- Seven live-run PHRs exist, all agent-written narratives, **self-reported,
  unverified**: `gmail-intake/017` (2026-07-15), `gmail-intake/018`
  (2026-07-15), `gmail-intake/020` and `021` (2026-07-29),
  `discord-notification/028` (2026-07-18; used a synthetic entry injected
  into the state store), `004-pipeline-orchestration/0002` (2026-07-23; all
  four services), `general/014` (2026-07-28; a skill-install check, not a
  pipeline run). No raw logs, traces or screenshots in the v1 repo (find);
  v1 `data/` holds only `.gitkeep`.
- PHR `004-pipeline-orchestration/0002` lines 34, 36, 50, 54 record Gemini
  429 bursts across a 22-message batch, about 6.5 minutes for one cycle,
  attributed there to a per-minute limit. This is separate from v2's daily
  cap of 18 (C5).
- `specs/001-gmail-intake/tasks.md`: 9 ticked (T001–T007, T032, T033), 24
  open (T008–T031), yet the files those tasks create exist and commits
  `4cbd861` and `b0d7adc` are titled with those task IDs. v1 `README.md:5`
  and `README.md:31-37` say "Production" for all 7 features. **Unreconciled.**
- PHR frontmatter names seven feature branches (`001-gmail-intake` through
  `007-email-scheduling`); local remote-tracking refs show only `origin/main`
  (`git branch -a`; no fetch was run, so the live remote was not queried).
  `001-gmail-intake` exists only as a spec folder on `main`.

### Owner decisions recorded

- **G1** (runtime integration strategy): **pending**. Options drafted in
  ADR-005 (Proposed, DRAFT).
- **G2**: canonical feature IDs are F008–F012, as in the specs and `CONSTITUTION.md:209`.
- **G3**: Gemini plan undecided; `workspace/HEARTBEAT.md` and the sim stay unchanged.
- **G4**: remote dashboard off until authentication is specced.
- **G5**: correct records going forward only; no rewrites of past PHRs or
  tasks.md; dated correction note below.

### Correction note — 2026-10-13

This note corrects earlier records without editing them (G5). Source for each
item is the claim or evidence line cited above.

1. `README.md:40,55` "F013 — in build": F013 is not specced (C2).
2. `README.md` Build Progress table: spec 002 (F009) has 17 tests, not 12;
   spec 003 (F010) has 29, not 19; the "F008–F012: 15" row is not a separate
   set of tests (C3). Canonical IDs are F008–F012 (G2), not F001–F005.
3. `README.md:55` "Discord (current)": the PK spec path notifies via
   WhatsApp; Discord is UK-only in the agent rules.
4. `README.md:51` OpenClaw 2026.7.1: installed runtime is 2026.6.11.
5. v2 `specs/*/tasks.md` (152/152 ticked): ticks reflect simulated tests,
   not live behaviour (C1, C4).
6. v1 `specs/001-gmail-intake/tasks.md` under-reports (24 open with code
   present), while v1 README reports all features "Production". Unreconciled.
7. "106 passing" is now verified by two pytest runs on 2026-10-13, against
   the simulation only.

## Outcome

- ✅ Impact: The external audit is now a verified claim set. Live behaviour of v2 is confirmed unproven; v1 holds the only real client code, with self-reported (unverified) live runs. Next work is gated on credential rotation (owner) and G1.
- 🧪 Tests: pytest run twice via ephemeral uv env, 106 passed each time; no live tests run.
- 📁 Files: this PHR; ADR-005 draft; one private checklist outside the repo.
- 🔁 Next prompts: owner rotates credentials using the private checklist; owner decides G1 and open questions in ADR-005; then plan the single live end-to-end lead with an ID-only trace.
- 🧠 Reflection: Both repos' task lists disagree with their code in opposite directions; ticks and agent-written PHRs are claims, not evidence. Only a command output or an exact line counts.

## Evaluation notes (flywheel)

- Failure modes observed: self-reported completion (v2 tasks.md, v1 README) diverging from evidence; feature-ID drift between README and specs; version drift across README, ADR-001 and the install; a supplied figure (61 commits) that did not match the repo.
- Graders run and results (PASS/FAIL): pytest 106/106 PASS (x2). Static test count 106 PASS. Live end-to-end: NOT RUN (blocked by owner).
- Prompt variant (if applicable): none
- Next experiment (smallest change to try): one live lead through a single test tenant with an ID-only trace, after rotation and G1.
