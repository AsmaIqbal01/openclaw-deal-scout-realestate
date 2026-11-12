# Implementation Plan: PK WhatsApp Lead Alert Adapter + Owner Relay (F013)

**Branch**: `006-pk-whatsapp-notify` | **Date**: 2026-11-12 | **Spec**: [spec.md](./spec.md)
**Status**: DRAFT, BLOCKED. `spec.md` scored 5.5 and 5.6 in two rounds and
the current version is unscored. This plan MUST NOT be used for
implementation until `spec.md` scores ≥ 9.6 (`CONSTITUTION.md:73`, `:81`)
and the owner approves it. Nothing in this plan installs, connects or
configures a transport.

## Summary

Add one adapter interface, `send_lead_alert`, between Delivery and every
lead-alert transport (spec §Interfaces). Pilot transport is T1,
`openclaw_whatsapp_web`: OpenClaw's WhatsApp plugin (unofficial WhatsApp Web)
on the dedicated number, free, pilot-only (spec OD-1). If ADR-005 G1 decides
the runtime is not OpenClaw, T5 `selfrun_whatsapp_web` replaces it (OD-1a).
At the first paying client, `agent_transport` switches to
`whatsapp_cloud_api` by configuration (OD-2, FR-016). Fallback is owner
relay: one email to `OWNER_RELAY_EMAIL` with a `wa.me` link (OD-3). The agent
only ever receives WhatsApp (Hard Rule 8, `CONSTITUTION.md:164-165`).

## Technical Context

- **Language**: Python 3.11+, standard library for the contract, state,
  messages and fake adapters (matches `CLAUDE.md` Active Technologies).
  The T1 adapter calls the `openclaw` CLI as a subprocess (no new Python
  dependency). The relay adapter calls the Gmail API over HTTPS with the
  standard library (`urllib.request`); no Google client library is added.
- **Runtime placement**: depends on ADR-005 G1, still open.
  - Option C (hybrid, recommended in ADR-005): the adapters run inside a
    tenant-aware Python MCP tool that the OpenClaw Delivery agent calls.
  - Option A (OpenClaw only): Delivery calls OpenClaw's own message action;
    the Python module is used for tests and state rules only. **Unverified**
    whether this split is workable.
  - Option B (v1 Python): T5 replaces T1 and the spec returns to spec
    (S1-T5).
- **Testing**: pytest, fake adapters, faked clock, faked subprocess runner,
  no network, no OpenClaw process.
- **Storage**: per-tenant `notify-records.json`, shared
  `whatsapp-channel-state.json`, `owner.json`, `notify-run.log`, all under
  `~/.openclaw/workspace/` (spec §Key Entities). No database.
- **Constraints**: zero cost for the pilot (`CONSTITUTION.md:131-134`);
  ID-only logs (FR-012); `WHATSAPP_BUSINESS_NUMBER` and `OWNER_RELAY_EMAIL`
  as env vars only.

## Constitution Check (draft; re-run against the implementation)

| Gate | Status | Note |
|---|---|---|
| I1 one market | PASS (design) | PK only |
| I2 zero cost | PASS for pilot | T1 free. The OD-2 Cloud API switch needs its own I2 approval (spec RISK-3) |
| I3 WhatsApp for PK | PASS (design) | agent gets WhatsApp only; owner relay is operator communication (OD-6) |
| I5 OpenClaw orchestrates | DEPENDS | on G1; A and C keep it |
| A2 CRM before notify | PASS (design) | `send_lead_alert` is called only after a confirmed CRM write |
| A4 / A8 isolation | PASS (design) | spec E2, §Multi-tenant isolation |
| A7 approval for client email | PASS (design) | relay email goes only to the owner (OD-6 ruling 2026-11-12) |
| Q4 no secrets | PASS (design) | env-var names only; AT-22 grep |
| B1 milestone | PASS | F013 (`CONSTITUTION.md:209`) |

## Key Decisions

- **P1 One interface for agent and owner messages** (`send_lead_alert` with
  `recipient_kind`). Rejected: separate agent and owner contracts (two
  error vocabularies).
- **P2 Pilot transport T1 via the `openclaw message send` CLI** (spec S1,
  S2, S4). Alternate T5 only under G1 = B. Rejected for the pilot: Cloud
  API (cost), click-to-chat as primary (manual), provider trials (expire).
- **P3 Owner relay by Gmail API email** (spec S3, S3a), credential as an
  OAuth token file whose path is held in an env var, as in v1
  (`~/openclaw-deal-scout/src/email_scheduler/auth.py:15-20`).
- **P4 Transport selected once, in owner config** (`agent_transport`), not
  per tenant, because all tenants share one number (FR-002).
- **P5 Template-ready alerts** (FR-015): every agent alert carries both
  `text` and `template`, so the Cloud API adapter needs no Delivery change.
- **P6 Adapters never raise** (K9; FR-013).
- **P7 At-most-once automatic send** via `attempt_in_flight` (E14).

📋 Architectural decision detected: pilot lead-alert transport (unofficial
WhatsApp Web via OpenClaw), template-ready adapter interface, and the
config-only switch to the paid Cloud API. Document reasoning and tradeoffs?
Run `/sp.adr pk-lead-alert-transport`.

## Components

```text
lead_alert/
├── contract.py      # LeadAlert, SendResult, error_type values, Adapter protocol
├── registry.py      # agent_transport -> adapter; unknown/unregistered -> transport_invalid
├── fake.py          # FakeAgentAdapter, FakeOwnerAdapter, FakeCloudApiAdapter (AT-14, AT-17)
├── openclaw_web.py  # T1: S1 send + S2 mapping + S4 health via subprocess
├── gmail_relay.py   # owner relay: S3 send + S3a token refresh
├── messages.py      # M2/M4/M5/M6/M8, E9 truncation, wa.me encoding, template params
├── state.py         # notify records, channel state, owner config (E13, E15)
├── delivery.py      # gate check, opt-in, retries, degrade/probe, caps, relay, expiry, crash recovery
└── log.py           # ID-only writer with a field allow-list (FR-012), stderr fallback (E15 b)
```

Path: repository-root `src/` under Option C (mirrors v1's layout).
**Unverified** until G1.

## Field mapping to fix before build (spec says unverified)

| Item | Source to read (read-only) | Fills |
|---|---|---|
| `message send --json` success field holding the provider message id | installed OpenClaw source or a `--dry-run` output, with owner approval | S1 |
| Error text for no listener / not linked / rate limit | installed OpenClaw source | S2 |
| `channels status --probe --json` field holding `works` | installed OpenClaw source | S4 |
| Gmail send and token refresh shapes | Google API reference, dated | S3, S3a |

If any row cannot be filled, F013 returns to spec (spec §External calls).

## Non-functional budgets

- Per lead: ≤ 40 s worst case (FR-007). Health check ≤ 10 s per run (S4).
- Per run: ≤ 30 sends with ≥ 3 s spacing (FR-010). Worst case if every send
  times out with no degradation is 30 × 43 s ≈ 21.5 min, over the 15-minute
  heartbeat; FR-008 degrades after 2 failed leads, so the real worst case is
  2 × 43 s + 10 s health check ≈ 96 s. T-024 asserts this.
- Logs: allow-list of fields, so a new field cannot leak lead content.

## Risks and guardrails

- Real send before go-live → FR-003 forces `fake` while any G date is null.
- Number ban (RISK-1, accepted) → degrade on the first `logged_out`, relay
  every lead, M5 to the owner.
- Paid switch (RISK-3) → `whatsapp_cloud_api` is not registered in F013;
  selecting it gives `transport_invalid` until its own feature lands.

## Validation

Definition of done: spec AT-01 to AT-29 pass; full suite passes (Q1);
Constitution Checker PASS; grep finds no phone number, email address or
token in the F013 files.
