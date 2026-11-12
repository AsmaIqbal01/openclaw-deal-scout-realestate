---
id: 0039
title: F013 spec interview, scoring, split
stage: general
date: 2026-10-13
surface: agent
model: claude-opus-5-5
feature: none (F013 spec drafted outside the repo; spec folder not yet created)
branch: master
user: asmaiqbal01
command: free-form brief "specify F013" (no /sp.specify run; spec drafted by hand from .specify/templates/spec-template.md)
labels: ["f013", "whatsapp", "spec", "clarify-interview", "spec-scorer", "split", "npm-view", "read-only"]
links:
  spec: null (draft in session scratchpad; backup at ~/dealclaw-private/f013-draft-spec.md, outside the repo)
  ticket: null
  adr: adrs/ADR-005-runtime-integration-strategy.md
  pr: null
files:
 - history/prompts/general/0039-f013-spec-interview-scoring-split.general.prompt.md (this file, new)
 - (outside repo) ~/dealclaw-private/f013-draft-spec.md (backup of F013 draft v3)
tests:
 - none run (spec work only); spec-scorer rubric applied in 3 rounds by independent read-only agents
---

## Prompt

### Prompt 6 (verbatim)

Brief: specify F013 (PK WhatsApp notification).

Outcome: a spec for F013 under the repo's next spec number, scored
at or above the threshold in CONSTITUTION.md (cite the line). Spec only.

Context: WhatsApp in OpenClaw is the external plugin @openclaw/whatsapp,
over WhatsApp Web (Baileys), not built-in as CONSTITUTION.md and TOOLS.md
say. It is not installed. See PHR 0038 and ADR-005 Amendment 1. Route 1
is chosen. I am deciding G1 as Option C, conditional on the MCP probe.

The spec must cover:
- Free WhatsApp for the pilot only, behind a swappable adapter, with a
  fallback (click-to-chat link or Telegram) that works if the number is
  banned or the session drops.
- Agent opt-in: the agent messages the DealClaw number first.
- A "Mark contacted" link in each notification.
- Logs hold IDs and error types only. Never lead content.
- Open questions to resolve: one sending number versus a number per
  tenant; always-on hosting (the session lives in the gateway, and today
  that is my WSL machine); QR re-link when the session drops; ban risk
  (unverified, list it as a risk); and whether installing
  @openclaw/whatsapp is allowed under the "no third-party dependencies"
  rule in my project summary (who publishes it is unverified).
- Acceptance tests written first, with a 9.6+ spec score.

Constraints: write only the new spec folder (spec.md, and plan/tasks only
if I approve the spec first). Do not edit CONSTITUTION.md, TOOLS.md,
ADR-005 or any existing spec. List needed corrections as proposals in the
terminal. Do not install the plugin. Do not run the OpenClaw binary or
any probe. No network calls. Never open config or credential files.
No git add or commit.

Authority: ask me the clarifying questions first, one at a time. Then
draft the spec. Run spec-scorer at most 3 rounds, then stop and report
the score and the gaps, even if it is below the threshold.

Verification: every claim cites file:line. Mark anything unconfirmed
as "unverified".

Name the Agent Factory concept you use now and the ones used before.
Show your plan and wait for my approval before writing any file.

### Prompt 6a (answer given through the question tool, verbatim)

Question 1 (fallback) answered: "Telegram to agent". Superseded by Prompt 7, Q1.

### Prompt 7 (verbatim, sent after interrupting the second question)

Q1: Option 1, owner relay link. Queue the lead as notify_pending and alert
    me with a wa.me link. No Telegram, no amendment to Hard Rule 8.
Q2: Not a web link. The agent replies with a short keyword plus the lead ID
    in WhatsApp. Mark this as unverified (inbound routing is documented but
    not proven). A web link can come later, after dashboard auth is specced.
Q3: The rule is from dealclaw-summary.md (Architecture notes), not the repo.
    Intent: no paid or untrusted third-party skills. Allow @openclaw/whatsapp
    only if the publisher is verified as OpenClaw. List the conflict with
    CONSTITUTION.md:145-147 as an open question.
Q4: No pilot has been told yet. Assume one dedicated DealClaw number, and
    the agent messages it first. Add a task to confirm in writing with
    Inhabitant Construction.

### Prompt 8 (verbatim, mid-turn)

Q2: Option 3. Ship the reply keyword in F013, WhatsApp only, no Telegram
(Hard Rule 8). Specify the tappable link as a later phase, blocked until
dashboard auth is specced. Keyword edge cases to cover in the spec: lower
and upper case, extra spaces, an unknown lead ID, a duplicate DONE for a
lead already marked, and a reply from a number not on the tenant allowlist.
The lead ID is an ID only, never lead content.

### Prompt 9 (verbatim, mid-turn)

1. Copy the current draft to ~/dealclaw-private/f013-draft-spec.md as a
   backup. Not into any repo.
2. OQ-6: answer (a). Owner alerts are operator communication outside
   Hard Rule 8. I will write the dated clarification myself. Do not edit
   CONSTITUTION.md.
3. After round 3, stop and report the score, the remaining gaps and the
   cost of another round. Do not run round 4 until I say so.
4. Make sure Telegram appears in the spec only as a rejected option.

### Prompt 10 (verbatim)

Decisions:
1. Option 2. Split F013 into F013a (outbound + owner relay) and F013b
   (inbound DONE keyword). Trim F013a to what the first pilot test needs
   and move crash recovery, at-most-once delivery and corrupt-file
   handling to a short "later" section.
2. Fix in F013a: E4 must detect a WSL session drop (no-listener and
   timeout errors must count toward degradation, and retries must stop
   sooner than 90 seconds per lead); and state explicitly any change to
   what unauthorized_reply_source means versus spec 001, or revert it.
3. Approved, network, read-only: run `npm view @openclaw/whatsapp` and
   report the maintainers, repository and version. Do NOT install it. If
   the publisher is OpenClaw, ask me before the next step.
4. Write PHR 0039 now (this session's prompts verbatim, no secrets, no
   long IDs, no phone digits).
5. Maximum 2 scoring rounds on F013a, then stop and report. Nothing goes
   into the repo until I approve the spec. No git add or commit.
6. Name the Agent Factory concept you use now and the ones used before.

## Response snapshot

### Clarifying interview outcome (owner decisions)

- Q1 fallback: owner relay (`notify_pending` + owner alert with a `wa.me`
  link). Telegram rejected (would breach Hard Rule 8, `CONSTITUTION.md:164-165`).
- Q2 "Mark contacted": WhatsApp reply keyword `DONE <lead_id>`; inbound
  routing unverified; tappable link deferred until dashboard auth is specced.
- Q3 dependency rule: from the owner's `dealclaw-summary.md` (outside the
  repo, not read); plugin allowed only if publisher is verified as OpenClaw;
  conflict with `CONSTITUTION.md:145-147` recorded as an open question.
- Q4: one dedicated DealClaw number; agent messages it first; written
  confirmation from the pilot agency Inhabitant Construction as a pre-plan task.
- OQ-6: answered (a): owner alerts on Discord are operator communication
  outside Hard Rule 8; owner writes the constitution clarification.

### Grounding facts used (verified)

- Spec-score threshold ≥ 9.6/10: `CONSTITUTION.md:73`, `:81`;
  `agents/spec-scorer/spec-scorer.md:5`, `:10`. Rubric: `spec-scorer.md:12-80`.
- Next spec number 006 (specs 001–005 exist); F013 within F008–F017
  (`CONSTITUTION.md:209`).
- No "no third-party dependencies" rule exists in the repo; nearest are
  per-feature "standard library only" notes (`CLAUDE.md:215`, `:217`).
- OpenClaw DM policies and pairing (`channels/whatsapp.md:255-261`;
  `channels/pairing.md:20`, `:32-33`, `:42-45`). Pairing can bootstrap the
  command owner, so the draft chose `allowlist`.

### Draft and scoring

The draft was written in the session scratchpad, not in the repo. Each round
was scored by a fresh, read-only agent applying the spec-scorer rubric.

| Round | Score | Main gaps |
|---|---|---|
| 1 | 5.4/10 | error paths 0.0/2.0 and multi-tenant 0.0/1.0: owner-alert failure, reply failure, invalid agent number, re-send failure, no exit from degraded, tenant mismatch, storage location |
| 2 | 5.8/10 | 12 gaps + 6 contradictions: retry rule vs no-retry cases, M4 vs blocked leads, `available()` vs no-listener, English-only M3d, phone digits in the alert queue, state-file failures, crash duplicates |
| 3 | 6.4/10 | 10 gaps: unverifiable plugin contract (S1–S3), rate limits, pipeline-run scope, isolation wording, `tenant_mismatch` storage, E10 vs one-M4, E12 check, 4 untested branches; E4 misses a WSL session drop; `unauthorized_reply_source` silently widened vs spec 001 |

Citations spot-checked by the scorers were correct after round 1 fixes
(K10, K12 and K16 wording corrected in rounds 1–2). Telegram appears in the
draft only as a rejected option (owner instruction).

### Decision after round 3

The owner chose option 2: split F013 into **F013a** (outbound notification +
owner relay, trimmed to the first pilot test; crash recovery, at-most-once
delivery and corrupt-file handling moved to a "later" section) and **F013b**
(inbound DONE keyword). Required fixes in F013a: E4 detects a WSL session
drop (no-listener and timeout count toward degradation; retries stop in
under 90 seconds per lead), and `unauthorized_reply_source` is either
reverted to spec 001's meaning or its change is stated explicitly. At most 2
scoring rounds on F013a.

### `npm view @openclaw/whatsapp` (approved, read-only network; nothing installed)

- Repository `git+https://github.com/openclaw/openclaw.git`, the same
  repository as the installed core (`~/.npm-global/lib/node_modules/openclaw/package.json:12-14`).
- Maintainers: npm users `steipete`, `vincentkoc`.
- Versions: latest 2026.10.2; extended-stable 2026.8.35; beta
  2026.10.5-beta.2; created 2026-05-02; a 2026.6.11 version exists.
- Compatibility: latest declares peer `openclaw >=2026.10.2` and plugin API
  `>=2026.10.2`; the installed core is 2026.6.11, so the latest plugin does
  not match the installed core.
- Publisher identity: consistent with the OpenClaw project (same repository,
  `@openclaw` scope). That the two maintainer accounts are OpenClaw's
  official publishers is **not independently verified**. Per owner
  instruction, the next plugin step waits for the owner.

## Outcome

- ✅ Impact: F013 requirements clarified and drafted. Three independent scoring rounds showed the single spec cannot reach 9.6 offline, so the owner split it. Plugin provenance checked; a core/plugin version mismatch surfaced.
- 🧪 Tests: none run; spec-scorer applied 3 times (5.4, 5.8, 6.4).
- 📁 Files: this PHR; backup draft outside the repo. No spec folder yet.
- 🔁 Next prompts: owner decides the plugin version path (pin 2026.6.11, or upgrade core to ≥ 2026.10.2); F013a draft + up to 2 scoring rounds; owner approval before the spec enters `specs/006-*`.
- 🧠 Reflection: a long single spec grows new seams each round; scoping to the first pilot test, and separating the unverified inbound path, is the faster route to a passing spec.

## Evaluation notes (flywheel)

- Failure modes observed: author-side contradictions caught only by an independent scorer; an unverifiable external contract caps interface marks; a question-tool answer later superseded by the owner's typed answer.
- Graders run and results (PASS/FAIL): spec-scorer round 1 BLOCKED 5.4; round 2 BLOCKED 5.8; round 3 BLOCKED 6.4.
- Prompt variant (if applicable): none
- Next experiment (smallest change to try): F013a limited to outbound + relay with the E4 session-drop fix, scored once before any further expansion.
