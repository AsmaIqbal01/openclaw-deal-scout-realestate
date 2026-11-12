---
id: 0041
title: F013 transport decision, scoring, fixes
stage: spec
date: 2026-11-12
surface: agent
model: claude-opus-5-5
feature: 006-pk-whatsapp-notify (F013)
branch: master
user: asmaiqbal01
command: free-form follow-ups to the F013 delegation brief (PHR 0040)
labels: ["f013", "whatsapp", "transport", "owner-relay", "spec-scorer", "openclaw-docs", "handoff"]
links:
  spec: specs/006-pk-whatsapp-notify/spec.md
  ticket: null
  adr: adrs/ADR-005-runtime-integration-strategy.md
  pr: null
files:
 - specs/006-pk-whatsapp-notify/spec.md (revised)
 - specs/006-pk-whatsapp-notify/plan.md (rewritten, draft, blocked)
 - specs/006-pk-whatsapp-notify/tasks.md (rewritten, draft, blocked)
 - history/prompts/006-pk-whatsapp-notify/0041-f013-transport-decision-scoring-fixes.spec.prompt.md (this file, new)
 - (outside repo) ~/dealclaw-private/DC-20261012T191406Z-2AFD97-v2.md (handoff summary, new)
tests:
 - none run (spec work only)
 - spec-scorer round 1 = 5.5/10 BLOCKED; round 2 = 5.6/10 BLOCKED; third round skipped by owner
---

## Prompt

### Prompt 1 (verbatim; the relay inbox address the owner typed is replaced by its env-var name)

Update spec.md first: transport must be free, using an unofficial WhatsApp Web library on the dedicated number for the pilot, behind the swappable adapter. Owner relay with wa.me link is the fallback. Record the ban risk in the spec. Then run spec-scorer on spec.md, max 2 rounds.Note in the spec: free transport is pilot-only; switch to the official paid WhatsApp Cloud API when the first paying client signs. The adapter must make this a config swap, not a rewrite.pilot email address is [OWNER_RELAY_EMAIL — literal address withheld from the repo per owner choice]

### Question-tool answers (verbatim choices)

- Pilot email use: "Owner relay inbox (R2)"
- Address in files: "Env var only (Recommended)"
- Library: "T1 OpenClaw plugin (Recommended)"

### Prompt 2 (verbatim, sent mid-turn)

1Record T1 OpenClaw plugin as the pilot transport only, behind send_lead_alert. Note that the runtime decision (OpenClaw / v1 / hybrid) is still open, and the self-run library is the alternate transport if the runtime is not OpenClaw.

### Prompt 3 (verbatim)

Yes: fix gaps 3-14 now. Update plan.md and tasks.md to match the spec. Approved: read-only look at the installed OpenClaw plugin docs to close gaps 1-2, no installs or changes. Write PHR 0041. For OD-6, record my ruling as a dated note: owner-relay emails sent only to the owner's own inbox are operator communication, same as Discord owner alerts, so they fall outside Hard Rule 4. Skip the third scoring round. Then write the handoff summary: ~/dealclaw-private/DC-20261012T191406Z-2AFD97-v2.md with today's F013 decisions, done vs remaining, and the exact next build step. No secrets or real numbers.

## Response snapshot

- Spec revised: OD-1 T1 `openclaw_whatsapp_web` pilot-only; OD-1a G1 open,
  T5 `selfrun_whatsapp_web` alternate if runtime is not OpenClaw; OD-2
  config-only switch to `whatsapp_cloud_api` at first paying client (FR-015
  template-ready alerts, FR-016); OD-3 owner relay by email to
  `OWNER_RELAY_EMAIL`; OD-5 ban risk accepted (RISK-1); OD-6 dated ruling
  2026-11-12; OD-7 one recipient per tenant; OD-8 only `delivered` counts
  toward the Phase 1 gate.
- Scoring: round 1 5.5 (interface 0.6, errors 0.1, ambiguity 0.3); round 2
  5.6 (interface 0.1, errors 0.1, ambiguity 0.9). Market, tests,
  multi-tenant and business gate at full marks both rounds.
- Read-only docs (installed OpenClaw 2026.6.11): `docs/cli/message.md`
  (send command, flags, WhatsApp target = E.164, fail-closed on unresolved
  credential refs), `docs/cli/channels.md` (status --probe, works / probe
  failed, config-only when gateway unreachable), `docs/channels/whatsapp.md`
  (no active listener, not linked, message-id acceptance rule). JSON output
  fields for send and status are not documented: marked unverified, mapped
  in tasks T-009.
- Post-round-2 fixes: S1/S2/S4 commands, S1-T5 deferral, S3a token form and
  refresh, M5/M6/M8 subjects, LeadAlert null rules, E2(d) corrupt USER.md,
  E1 invalid date, E14 crash-recovered not eligible, E15 write failures,
  state transitions, AT-21 range, AT-26 to AT-29. Unscored.
- plan.md and tasks.md rewritten to match.

## Outcome

- ✅ Impact: F013 transport decided for the pilot; fallback, ban risk and the paid switch recorded; spec gaps addressed but unscored.
- 🧪 Tests: none; 2 scoring rounds (5.5, 5.6).
- 📁 Files: spec, plan, tasks revised; this PHR; private handoff file.
- 🔁 Next prompts: re-score spec (T-001); owner decides G1 (T-002); T-009 field mapping.
- 🧠 Reflection: interface marks stay capped until the CLI's JSON output fields are read from the installed source.

## Evaluation notes (flywheel)

- Failure modes observed: scorer found new contradictions introduced by round-1 fixes (null tenant for E4 M8, crash-recovered lead still eligible).
- Graders run and results (PASS/FAIL): spec-scorer round 1 BLOCKED 5.5; round 2 BLOCKED 5.6.
- Prompt variant (if applicable): none
- Next experiment (smallest change to try): fill T-009 field mapping, then one scoring round.
