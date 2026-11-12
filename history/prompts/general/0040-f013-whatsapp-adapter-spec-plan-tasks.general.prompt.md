---
id: 0040
title: F013 WhatsApp adapter spec plan tasks
stage: general
date: 2026-11-12
surface: agent
model: claude-opus-5-5
feature: 006-pk-whatsapp-notify (F013)
branch: master
user: asmaiqbal01
command: free-form delegation brief "F013 WhatsApp adapter (spec + plan only)"
labels: ["f013", "whatsapp", "adapter", "owner-relay", "spec", "plan", "tasks", "draft"]
links:
  spec: specs/006-pk-whatsapp-notify/spec.md
  ticket: null
  adr: adrs/ADR-005-runtime-integration-strategy.md
  pr: null
files:
 - specs/006-pk-whatsapp-notify/spec.md (new)
 - specs/006-pk-whatsapp-notify/plan.md (new, draft, blocked)
 - specs/006-pk-whatsapp-notify/tasks.md (new, draft, blocked)
 - history/prompts/general/0040-f013-whatsapp-adapter-spec-plan-tasks.general.prompt.md (this file, new)
tests:
 - none run (spec work only); spec not scored
---

## Prompt

Delegation Brief: F013 WhatsApp adapter (spec + plan only)

Outcome: A Spec Kit set for F013 (spec.md, plan.md, tasks.md) for a
free, swappable WhatsApp notification adapter with a fallback channel.
Nothing is built or connected in this task.

Context:
- Specs live in this repo, following CONSTITUTION.md and F008-F012.
- Pipeline code is in ~/openclaw-deal-scout (v1). Read it to see how
  the Discord notifier works, but do not edit it.
- A dedicated WhatsApp Business number exists. Refer to it ONLY as the
  env var WHATSAPP_BUSINESS_NUMBER. Never write the real number in any file.
- Gating rule (put it in the spec): go-live only after (1) one live
  Discord lead is proven with an ID-only trace, (2) this spec is
  approved, (3) the pilot client confirms how they receive leads.

Constraints:
- Plan mode. Read-only except the new files under the F013 spec folder.
- Compare free options (WhatsApp Cloud API free tier, click-to-chat
  link, others). Give pros, cons and limits. Do not choose. List them
  as options for me to decide.
- Define one adapter interface (send_lead_alert) so WhatsApp can be
  swapped out, with Discord/email as the fallback.
- No secrets, tokens, or real phone numbers in files. No git
  add/commit/push. Do not describe security weaknesses in repo files.

Authority: Create the F013 spec files only. Ask before anything else.

Deliverable: The 3 files, plus a short summary listing open questions
for me and the pilot client.

Verification: Show me the file list and the diff. Confirm that grep
finds no phone number or token in the repo. Mark anything unverified
as unverified.

### Answers given through the question tool (verbatim choices)

- Fallback framing: "Owner relay (Recommended)"
- Scope: "Outbound only, reuse F013a (Recommended)"
- PHR: "Yes, write PHR 0040"

## Response snapshot

- Created `specs/006-pk-whatsapp-notify/` with spec.md, plan.md, tasks.md.
  Plan and tasks are marked DRAFT, BLOCKED until the spec scores ≥ 9.6 and
  is approved (`CONSTITUTION.md:81`).
- Spec reuses the F013a draft (states, outcome table, ID-only logs) with
  transport-neutral error types, the `send_lead_alert` interface, the
  three-part go-live gate, and five transport options (T1 OpenClaw WhatsApp
  plugin, T2 Cloud API, T3 click-to-chat, T4 provider trials, T5 self-run
  library) with pros/cons/limits and no choice made.
- Fallback = owner relay over Discord or email to the owner; the agent only
  receives WhatsApp (Hard Rule 8).
- Web search (read-only) on Cloud API pricing and limits: third-party sources
  say in-window utility templates became billable on 2026-10-01; not
  confirmed on Meta's live page, marked unverified.
- Flagged conflict: go-live G-1 (live Discord lead) vs Hard Rule 8 / Phase 1
  gate, recorded as OQ-1.

## Outcome

- ✅ Impact: F013 spec set drafted; transport choice left to the owner.
- 🧪 Tests: none run; spec not yet scored.
- 📁 Files: 3 spec files + this PHR.
- 🔁 Next prompts: owner answers OQ-1 to OQ-4; spec-scorer run; pilot client answers OQ-5.
- 🧠 Reflection: the brief's "Discord/email fallback" collided with Hard Rule 8; asking before writing kept the agent-facing channel WhatsApp-only.

## Evaluation notes (flywheel)

- Failure modes observed: brief conflicted with earlier owner decisions (split, fallback, plan-before-score); resolved by asking.
- Graders run and results (PASS/FAIL): none.
- Prompt variant (if applicable): none
- Next experiment (smallest change to try): score spec.md once before editing plan.md further.
